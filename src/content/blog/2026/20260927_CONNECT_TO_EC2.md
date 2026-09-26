---
title: "改めてEC2への接続ってどの方法でできるのかまとめてみる"
publishDate: 2026-09-27 00:00:00
img: /assets/blog/2026/EC2.svg
img_alt: EC2
description: |
  VPC上のEC2へ接続する8つの方法を網羅的に解説。パブリックIP・踏み台・EIC Endpoint・Session Manager・Fleet Manager・Client VPN・Site-to-Site VPN・Direct Connectの仕組み・手順・比較を一記事にまとめました。
tags:
  - Amazon Web Service
  - AWS
  - EC2
  - SSM
  - VPN
---

# VPC上のEC2に接続する方法を全部まとめてみた ― 8つのアプローチを徹底比較

## はじめに

AWS上のVPCに配置したEC2インスタンスに「どうやって接続するか」は、AWSを使い始めると必ずぶつかるテーマです。検索すると個別の接続方法を解説した記事はたくさん見つかりますが、**すべての方法を横並びで比較した記事**はなかなかありません。

この記事では、EC2への接続方法を **8つ** 網羅的に取り上げ、それぞれの仕組み・ユースケース・セットアップの主要手順・メリット/デメリットを解説します。「結局うちのケースではどれを使えばいいの？」という疑問に答えられる内容を目指しました。

---

## 全体像：8つの接続方法マップ

まず、この記事で扱う8つの方法を俯瞰します。大きく分けると「インターネット経由」「AWSネットワーク内で完結」「オンプレミスとの専用接続」の3カテゴリに分類できます。

```
┌─────────────────────────────────────────────────────────┐
│                     接続方法の分類                         │
├───────────────────┬──────────────────┬──────────────────┤
│ インターネット経由  │ AWSネットワーク内 │ オンプレ専用接続   │
├───────────────────┼──────────────────┼──────────────────┤
│ ① パブリックIP     │ ④ Session Manager│ ⑦ Site-to-Site VPN│
│ ② 踏み台サーバー   │ ⑤ Fleet Manager  │ ⑧ Direct Connect  │
│ ③ EC2 Instance    │ ⑥ Client VPN     │                  │
│    Connect Endpoint│                  │                  │
└───────────────────┴──────────────────┴──────────────────┘
```

---

## ① パブリックIPを付与してインターネット経由で接続する

### 仕組み

最もシンプルな方法です。EC2インスタンスにパブリックIPアドレス（またはElastic IP）を割り当て、インターネットゲートウェイ経由でSSHやRDP接続します。

```
[ローカルPC] ──── インターネット ──── [IGW] ──── [パブリックサブネットのEC2]
```

### ユースケース

検証用や個人開発など、素早く接続したいケース。本番環境では基本的に非推奨です。

### 主要セットアップ手順

**1. VPCにインターネットゲートウェイ（IGW）をアタッチ**

VPCコンソール → 「インターネットゲートウェイ」 → 「インターネットゲートウェイの作成」で作成し、対象VPCにアタッチします。

**2. パブリックサブネットのルートテーブルにIGWへのルートを追加**

```
送信先: 0.0.0.0/0
ターゲット: igw-xxxxxxxx（作成したIGW）
```

**3. EC2にパブリックIPまたはElastic IPを付与**

EC2起動時に「パブリックIPの自動割り当て」を有効にするか、Elastic IPを作成してインスタンスに関連付けます。

```bash
# Elastic IPの割り当てと関連付け（AWS CLI）
aws ec2 allocate-address --domain vpc
aws ec2 associate-address --instance-id i-xxxxxxxxx --allocation-id eipalloc-xxxxxxxxx
```

**4. セキュリティグループでSSH/RDPを許可**

```bash
# SSH（Linux）の場合：ポート22
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxxx \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/32  # 自分のIPアドレスに限定する

# RDP（Windows）の場合：ポート3389
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxxx \
  --protocol tcp \
  --port 3389 \
  --cidr 203.0.113.0/32
```

**5. 接続**

```bash
# SSH（Linux）の場合
ssh -i ~/.ssh/my-key.pem ec2-user@<パブリックIP>

# RDP（Windows）の場合
# Macなら「Microsoft Remote Desktop」アプリを使用
# 接続先: <パブリックIP>:3389
# ユーザー名: Administrator
# パスワード: EC2コンソールで「Windows パスワードを取得」から復号
```

### メリット / デメリット

| メリット | デメリット |
|---|---|
| 設定が最もシンプル | EC2がインターネットに直接公開される |
| 追加コストが少ない（EIPは固定費あり） | セキュリティグループの設定ミスが重大事故に直結 |
| 馴染みのあるSSH/RDP接続 | 鍵ファイルの管理が必要 |

> **⚠️ 注意：** セキュリティグループのインバウンドルールで `0.0.0.0/0`（全世界）からSSHやRDPを許可するのは絶対に避けてください。必ず接続元IPを限定しましょう。

---

## ② 踏み台サーバー（Bastion Host）経由で接続する

### 仕組み

パブリックサブネットに「踏み台サーバー」と呼ばれる中継用のEC2を配置し、そこを経由してプライベートサブネットのEC2にアクセスします。接続したい本番サーバーを直接インターネットに公開せずに済む、古くからある定番パターンです。

```
[ローカルPC] ── インターネット ── [IGW] ── [踏み台EC2（パブリック）] ── [本番EC2（プライベート）]
```

### ユースケース

プライベートサブネットのEC2にSSH/RDPでアクセスしたいが、SSMエージェントの導入が難しい環境や、従来型の運用フローを維持したいケース。

### 主要セットアップ手順

**1. パブリックサブネットに踏み台用EC2を起動**

小さいインスタンスタイプ（t3.micro 等）で十分です。パブリックIPを付与し、セキュリティグループでSSH（ポート22）を接続元IPのみに許可します。

**2. プライベートサブネットのEC2のセキュリティグループを設定**

踏み台サーバーのセキュリティグループ、またはプライベートIPからのSSHのみを許可します。

```
インバウンドルール:
  タイプ: SSH（またはRDP）
  ポート: 22（またはRDPなら3389）
  ソース: sg-xxxxx（踏み台のセキュリティグループ）
```

**3. SSHのProxyJump（多段接続）で接続**

ローカルPCの `~/.ssh/config` に以下を記述すると、1コマンドで踏み台経由の接続ができます。

```
Host bastion
  HostName <踏み台のパブリックIP>
  User ec2-user
  IdentityFile ~/.ssh/bastion-key.pem

Host private-ec2
  HostName <プライベートEC2のプライベートIP>
  User ec2-user
  IdentityFile ~/.ssh/private-key.pem
  ProxyJump bastion
```

```bash
# これだけで踏み台経由でプライベートEC2に接続できる
ssh private-ec2
```

> **Windows EC2（RDP）への踏み台接続：** SSHポートフォワーディングを使います。まず踏み台経由でRDPポートをローカルに転送し、RDPクライアントから `localhost:13389` に接続します。
>
> ```bash
> ssh -i ~/.ssh/bastion-key.pem -L 13389:<プライベートEC2のIP>:3389 ec2-user@<踏み台のパブリックIP>
> ```

### メリット / デメリット

| メリット | デメリット |
|---|---|
| 本番EC2をインターネットに公開しなくてよい | 踏み台サーバー自体の管理・パッチ適用が必要 |
| SSHの慣れた操作感 | 踏み台が単一障害点になりうる |
| 追加AWSサービスの契約不要 | 鍵ファイルの管理が煩雑（2組必要） |

---

## ③ EC2 Instance Connect Endpoint で接続する

### 仕組み

2023年に登場した比較的新しいサービスです。VPC内に「EC2 Instance Connect Endpoint（EIC Endpoint）」を作成すると、**パブリックIPも踏み台サーバーもIGWも不要**で、AWSの内部ネットワークを経由してプライベートサブネットのEC2にSSH/RDP接続できます。

```
[ローカルPC] ── インターネット ── [AWS API] ── [EIC Endpoint（VPC内）] ── [プライベートEC2]
```

IAMの認証情報を使ってAWS APIを呼び出し、EIC Endpointがトンネルを張ってくれるイメージです。

### ユースケース

プライベートサブネットのEC2にSSH/RDPしたいが、踏み台サーバーの管理コストをなくしたいケース。SSMのエージェントが入れられない場合にも有効です。

### 主要セットアップ手順

**1. EC2 Instance Connect Endpointの作成**

VPCコンソール → 「エンドポイント」 → 「エンドポイントの作成」で、サービスカテゴリから**「EC2 Instance Connect Endpoint」**を選択します。

```bash
# AWS CLIで作成する場合
aws ec2 create-instance-connect-endpoint \
  --subnet-id subnet-xxxxxxxxx \
  --security-group-ids sg-xxxxxxxxx
```

> **注意：** EIC Endpointの作成には数分かかります。ステータスが `create-complete` になるまで待ちましょう。

**2. セキュリティグループの設定**

EIC Endpointのセキュリティグループ：アウトバウンドでプライベートEC2へのSSH（22）/RDP（3389）を許可。

プライベートEC2のセキュリティグループ：EIC Endpointのセキュリティグループからのインバウンドを許可。

**3. IAMポリシーの設定**

接続するユーザーに以下の権限を付与します。

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2-instance-connect:OpenTunnel"
      ],
      "Resource": "arn:aws:ec2:ap-northeast-1:123456789012:instance-connect-endpoint/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstanceConnectEndpoints",
        "ec2:DescribeInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

**4. 接続**

```bash
# SSH接続（Linux EC2）
aws ec2-instance-connect ssh \
  --instance-id i-xxxxxxxxx \
  --connection-type eice

# RDP接続（Windows EC2）のためのトンネルを作成
aws ec2-instance-connect open-tunnel \
  --instance-id i-xxxxxxxxx \
  --remote-port 3389 \
  --local-port 13389
# 別のターミナルや「Microsoft Remote Desktop」から localhost:13389 に接続
```

### メリット / デメリット

| メリット | デメリット |
|---|---|
| パブリックIP・IGW・踏み台が一切不要 | EIC Endpointの作成に数分かかる |
| IAMで接続を制御でき、監査ログも残る | 同時接続数に制限がある |
| 追加のエージェントインストール不要 | 比較的新しいサービスのため情報が少なめ |
| SSHだけでなくRDPトンネルにも対応 | VPCあたり1つしかEIC Endpointを作成できない |

---

## ④ Systems Manager Session Manager で接続する

### 仕組み

AWS Systems Manager（SSM）のSession Manager機能を使う方法です。EC2にインストールされた**SSMエージェント**がSSMサービスと常時通信しており、AWSマネジメントコンソールやCLIからシェルセッションを開始できます。

```
[ローカルPC] ── インターネット ── [AWS API / SSMサービス] ←→ [SSMエージェント on EC2（プライベート）]
                                                                    │
                                                          [VPCエンドポイント or NATゲートウェイ]
```

EC2からSSMサービスへの通信経路として、**VPCエンドポイント（PrivateLink）**またはNATゲートウェイが必要です。

### ユースケース

SSHの鍵管理をなくしたい、ポート22を開けたくない、IAMで接続を一元管理したいケース。多くの企業で標準的な接続方法として採用されています。

### 主要セットアップ手順

**1. SSMエージェントの確認**

Amazon Linux 2/2023やWindows Server 2016以降のAMIにはSSMエージェントがプリインストールされています。インストール状況を確認します。

```bash
# Linux（Amazon Linux）の場合
sudo systemctl status amazon-ssm-agent

# Windows（PowerShell）の場合
Get-Service AmazonSSMAgent
```

**2. EC2にIAMロール（インスタンスプロファイル）をアタッチ**

EC2がSSMサービスと通信するためのIAMロールが必要です。AWS管理ポリシー `AmazonSSMManagedInstanceCore` をアタッチしたロールを作成し、EC2に関連付けます。

IAMコンソール → 「ロール」 → 「ロールを作成」 → 信頼されたエンティティで「AWSのサービス」→「EC2」を選択 → `AmazonSSMManagedInstanceCore` ポリシーをアタッチ → ロール名を入力して作成

作成後、EC2コンソールで対象インスタンスを選択 → 「アクション」 → 「セキュリティ」 → 「IAMロールを変更」から関連付けます。

**3. VPCエンドポイントの作成（プライベートサブネットの場合）**

EC2がNATゲートウェイ経由でインターネットに出られない場合、以下の3つのVPCエンドポイント（Interface型）を作成します。

```
com.amazonaws.<リージョン>.ssm
com.amazonaws.<リージョン>.ssmmessages
com.amazonaws.<リージョン>.ec2messages
```

VPCコンソール → 「エンドポイント」 → 「エンドポイントの作成」から、それぞれのサービス名を選択して作成します。各エンドポイントのセキュリティグループでは、EC2からのHTTPS（ポート443）インバウンドを許可してください。

> **コスト補足：** Interface型VPCエンドポイントは1つあたり約$0.014/時間（東京リージョン）+ データ処理料金がかかります。3つ作成すると月額約$30程度になるため、NATゲートウェイとのコスト比較も検討しましょう。

**4. 接続**

```bash
# AWSマネジメントコンソールから：
# EC2 → インスタンスを選択 → 「接続」 → 「Session Manager」タブ → 「接続」

# AWS CLIから（Linux/Windows共通）：
aws ssm start-session --target i-xxxxxxxxx

# ポートフォワーディングでRDP接続する場合：
aws ssm start-session \
  --target i-xxxxxxxxx \
  --document-name AWS-StartPortForwardingSession \
  --parameters '{"portNumber":["3389"],"localPortNumber":["13389"]}'
# その後、RDPクライアントから localhost:13389 に接続
```

### メリット / デメリット

| メリット | デメリット |
|---|---|
| SSH鍵の管理が不要 | SSMエージェントのインストールが必要 |
| ポート22/3389を開ける必要がない | VPCエンドポイント3つ分の費用がかかる |
| IAMで接続を一元管理、CloudTrailで監査可能 | ファイル転送はひと手間かかる |
| コンソールからワンクリックで接続 | ネットワーク構成によっては初期設定がやや複雑 |

---

## ⑤ Systems Manager Fleet Manager で接続する

### 仕組み

Fleet Managerは④のSession Managerと同じSSMの仕組みを基盤にしていますが、**AWSマネジメントコンソール上のGUIでリモートデスクトップ（RDP）接続**ができる点が特徴です。Windowsサーバーの管理に特に便利です。

```
[ブラウザ] ── AWS マネジメントコンソール ── [SSMサービス] ←→ [SSMエージェント on Windows EC2]
```

### ユースケース

Windows EC2をブラウザだけでリモートデスクトップ操作したいケース。RDPクライアントのインストールやVPN接続が不要になります。

### 主要セットアップ手順

前提条件は④のSession Managerと同様です（SSMエージェント、IAMロール、VPCエンドポイント）。これらが設定済みであれば、追加の設定なしにFleet Managerが利用できます。

**1. Fleet Managerを開く**

AWSマネジメントコンソール → Systems Manager → 左メニュー「Fleet Manager」を開きます。SSMエージェントが正常に動作しているEC2インスタンスが一覧に表示されます。

> **インスタンスが表示されない場合：** SSMエージェントが起動していないか、IAMロールが正しくアタッチされていない可能性があります。④のSession Managerのセットアップ手順を確認してください。

**2. リモートデスクトップ接続**

1. 対象のインスタンスを選択（チェックボックスにチェック）
2. 右上の **「ノードアクション」** → **「リモートデスクトップとの接続」** をクリック
3. 認証方法を選択
    - **ユーザー認証情報**：Windowsのユーザー名とパスワードを直接入力
    - **キーペア**：EC2のキーペア（.pemファイル）をアップロードしてパスワードを自動復号
4. **「接続」** をクリックすると、ブラウザ上にWindowsのデスクトップ画面が表示される

### メリット / デメリット

| メリット | デメリット |
|---|---|
| ブラウザだけでRDP接続できる | Windowsインスタンスのみ対応（RDP） |
| RDPクライアントのインストール不要 | 操作のレスポンスはネイティブRDPより劣る |
| ④と同じくIAM管理＋監査ログ対応 | 解像度やクリップボード連携に制限あり |
| ポートやSGの追加設定が不要 | 長時間のGUI操作には向かない |

---

## ⑥ AWS Client VPN で接続する

### 仕組み

AWSが提供するマネージドVPNサービスです。ローカルPCにOpenVPNクライアントをインストールし、**Client VPNエンドポイント**に接続することで、PCがVPC内のネットワークに参加したような状態になります。プライベートIPで直接EC2にアクセスできます。

```
[ローカルPC + OpenVPNクライアント] ── VPNトンネル ── [Client VPN Endpoint] ── [プライベートEC2]
```

### ユースケース

リモートワークの社員がVPC内の複数リソース（EC2だけでなくRDSなども）にプライベートアクセスしたいケース。1台だけでなく複数台のEC2に接続する場合に便利です。

### 主要セットアップ手順

**1. ACM（AWS Certificate Manager）で証明書を準備**

相互認証を使う場合、サーバー証明書とクライアント証明書を ACM にインポートします。

```bash
# easy-rsaで証明書を生成する例
git clone https://github.com/OpenVPN/easy-rsa.git
cd easy-rsa/easyrsa3
./easyrsa init-pki
./easyrsa build-ca nopass
./easyrsa build-server-full server nopass
./easyrsa build-client-full client1 nopass
```

生成した証明書（CA証明書、サーバー証明書、サーバー秘密鍵）をACMにインポートします。

```bash
aws acm import-certificate \
  --certificate fileb://pki/issued/server.crt \
  --private-key fileb://pki/private/server.key \
  --certificate-chain fileb://pki/ca.crt
```

**2. Client VPNエンドポイントの作成**

VPCコンソール → 「Client VPNエンドポイント」 → 「Client VPNエンドポイントの作成」

主な設定項目：

```
クライアントIPv4 CIDR: 10.100.0.0/16（VPCのCIDRと重複しない範囲）
サーバー証明書ARN: ACMにインポートした証明書
認証オプション: 相互認証 or Active Directory認証 or SAML認証
```

> **認証オプションの選び方：** 個人利用や小規模なら「相互認証（証明書ベース）」が手軽です。企業で既にActive DirectoryやOkta/Azure ADなどのIdPを運用している場合は「SAML認証」にすると、既存のID管理と統合でき運用が楽になります。

**3. ターゲットネットワークの関連付け**

Client VPNエンドポイントにサブネットを関連付けます（EC2が配置されているサブネット）。可用性を高めるには複数AZのサブネットを関連付けましょう。

**4. 承認ルールの追加**

接続を許可するCIDR範囲を設定します（例：VPC全体の `10.0.0.0/16`）。

**5. クライアント設定ファイルのダウンロードと接続**

Client VPNエンドポイントの画面から「クライアント設定のダウンロード」で `.ovpn` ファイルを取得し、クライアント証明書と鍵の情報を追記します。

`.ovpn` ファイルの末尾に以下を追加：

```
<cert>
（client1.crt の内容を貼り付け）
</cert>

<key>
（client1.key の内容を貼り付け）
</key>
```

```bash
# AWS公式VPNクライアント or OpenVPNクライアント（Tunnelblick等）で .ovpn を読み込んで接続
# 接続後はプライベートIPで直接SSH/RDP可能
ssh -i ~/.ssh/my-key.pem ec2-user@10.0.1.100
# RDPの場合は Microsoft Remote Desktop から 10.0.1.100:3389 へ接続
```

### メリット / デメリット

| メリット | デメリット |
|---|---|
| VPC内のすべてのリソースにプライベートアクセス | 証明書管理がやや複雑 |
| SAML/AD連携で企業のID基盤と統合可能 | エンドポイントのサブネット関連付けに費用が発生（約$0.15/時間/関連付け） |
| スプリットトンネル設定でVPC向け通信のみVPN経由にできる | クライアントソフトのインストールが必要 |
| EC2以外のリソース（RDS等）にもアクセス可能 | 同時接続ユーザー数に応じてコスト増 |

---

## ⑦ Site-to-Site VPN で接続する

### 仕組み

オンプレミスのネットワークとAWS VPCをIPsec VPNトンネルで接続する方法です。オフィスのVPN対応ルーターと、AWS側のVirtual Private Gateway（VGW）またはTransit Gatewayの間にトンネルを張ります。

```
[オフィスPC] ── [社内ネットワーク] ── [VPN装置/ルーター] ══ IPsecトンネル ══ [VGW/TGW] ── [プライベートEC2]
```

### ユースケース

オフィスのネットワーク全体からVPC内のリソースにアクセスしたいケース。オンプレミスとAWSのハイブリッド構成で定番の方法です。

### 主要セットアップ手順

**1. カスタマーゲートウェイ（CGW）の作成**

オンプレミス側のVPN装置のパブリックIPを登録します。

```
VPCコンソール → 「カスタマーゲートウェイ」 → 「カスタマーゲートウェイの作成」
  名前: office-vpn-router
  BGP ASN: 65000（オンプレ側のASN。静的ルーティングなら任意の値でOK）
  IPアドレス: <VPN装置のパブリックIP>
```

**2. Virtual Private Gateway（VGW）の作成とVPCへのアタッチ**

```
VPCコンソール → 「仮想プライベートゲートウェイ」 → 「仮想プライベートゲートウェイの作成」
→ 作成後、対象VPCにアタッチ
```

**3. Site-to-Site VPN接続の作成**

```
VPCコンソール → 「Site-to-Site VPN接続」 → 「VPN接続の作成」
  ターゲットゲートウェイ: 作成したVGW
  カスタマーゲートウェイ: 作成したCGW
  ルーティングオプション: 静的 or 動的（BGP）
  静的IPプレフィックス: オンプレ側のCIDR（例: 192.168.0.0/16）
```

> **静的ルーティング vs 動的（BGP）ルーティング：** 小規模なら静的ルーティングで十分です。拠点が多い場合や、ルート情報を自動的にやり取りしたい場合はBGPを選択しましょう。

**4. VPN装置の設定**

VPN接続作成後、**「設定のダウンロード」**から対応するルーター/ファイアウォールの設定テンプレートをダウンロードできます。Yamaha、Cisco、Fortinet（FortiGate）、Juniper、Palo Altoなど主要機器のテンプレートが用意されているため、テンプレートに沿ってオンプレ側の設定を適用します。

**5. ルートテーブルの更新**

VPC側のルートテーブルに、オンプレミス宛のルートを追加します。

```
VPCコンソール → 「ルートテーブル」 → 該当のルートテーブルを選択 → 「ルート伝搬」タブ
→ VGWからのルート伝搬を「有効」にする
```

静的ルーティングの場合は「ルート」タブから手動でオンプレCIDR → VGWのルートを追加します。

**6. 接続確認**

VPN接続のステータスが **「使用可能」（available）** になり、トンネルのステータスが **「UP」** になっていることを確認します。

```bash
# オフィスのPCからプライベートEC2にpingして疎通確認
ping 10.0.1.100
```

### メリット / デメリット

| メリット | デメリット |
|---|---|
| オフィス全体からVPCにアクセス可能 | オンプレ側にVPN対応機器が必要 |
| 暗号化されたトンネルで安全 | 帯域幅は最大1.25Gbps/トンネル |
| ルーター設定テンプレートが充実 | インターネット品質に依存（遅延・パケロス） |
| コスト比較的安価（VPN接続あたり約$36/月） | 初期設定にネットワーク知識が必要 |

---

## ⑧ AWS Direct Connect で接続する

### 仕組み

オンプレミスのデータセンターとAWSを**専用線（物理的な光ファイバー）**で接続する方法です。インターネットを経由しないため、安定した帯域と低遅延を実現できます。

```
[オフィス/DC] ── [Direct Connectロケーション（相互接続ポイント）] ══ 専用線 ══ [AWS] ── [VPC] ── [EC2]
```

### ユースケース

大量のデータ転送がある場合や、低遅延が要求される業務システム。金融・医療などミッションクリティカルな環境で採用されることが多いです。

### 主要セットアップ手順

**1. Direct Connect接続の申請**

AWSコンソール → Direct Connect → 「接続の作成」

```
帯域幅: 1Gbps or 10Gbps（専用接続）/ 50Mbps〜10Gbps（ホスト接続）
ロケーション: 最寄りのDirect Connectロケーション（日本ではEquinix TY2等）
```

> **専用接続 vs ホスト接続：** 専用接続はAWSとの物理ポートを占有するため大容量向き。ホスト接続はAWSパートナー経由で共有ポートの一部帯域を利用するため、小〜中規模や導入スピードを重視する場合に適しています。

> **注意：** 専用接続の場合、物理的な回線の手配が必要なため、利用開始まで数週間〜数ヶ月かかります。パートナー経由のホスト接続であればより短期間で利用開始できます。

**2. 仮想インターフェイス（VIF）の作成**

プライベートVIF（VPCへの接続用）を作成し、VGWまたはDirect Connect Gatewayに関連付けます。

```
VIFタイプ: プライベート
接続先: Virtual Private Gateway or Direct Connect Gateway
VLAN: （自動割り当て or 指定）
BGP ASN: オンプレ側のASN
```

> **Direct Connect Gateway を使うと：** 複数リージョンの複数VPCに1つのDirect Connect接続から到達できるようになります。マルチリージョン構成の場合は検討しましょう。

**3. オンプレ側のルーター設定**

BGPピアリングの設定を行い、AWS側とルート情報を交換できるようにします。AWSから提示されるBGPピアIP、認証キーをルーターに設定します。

**4. ルートテーブルの確認**

VPC側のルートテーブルでVGWへのルート伝搬を有効にします。

### メリット / デメリット

| メリット | デメリット |
|---|---|
| 安定した帯域・低遅延 | 初期コストが高い（回線費用＋ポート費用） |
| インターネットを経由しない高いセキュリティ | 利用開始まで数週間〜数ヶ月 |
| 大容量データ転送に最適 | Direct Connectロケーションへの物理的な接続が必要 |
| 月額固定でデータ転送量が予測しやすい | 冗長化には2回線以上の契約が必要 |

> **ベストプラクティス：** AWS公式ではDirect Connectの冗長構成として、**Site-to-Site VPNをバックアップ回線**として併用することを推奨しています。Direct Connectの障害時にVPN経由で通信を継続できます。

---

## 比較一覧表

8つの方法を横並びで比較します。

| 方法 | パブリックIP | 踏み台/エージェント | セキュリティ | コスト | 導入難易度 | 主な対象 |
|---|---|---|---|---|---|---|
| ① パブリックIP | 必要 | 不要 | △ | ◎ 安い | ◎ 簡単 | 検証・個人開発 |
| ② 踏み台サーバー | 踏み台のみ | 踏み台EC2 | ○ | ○ | ○ | 従来型運用 |
| ③ EIC Endpoint | 不要 | 不要 | ◎ | ◎ 安い | ○ | プライベートEC2 |
| ④ Session Manager | 不要 | SSMエージェント | ◎ | ○ | ○ | 企業の標準運用 |
| ⑤ Fleet Manager | 不要 | SSMエージェント | ◎ | ○ | ○ | Windows管理 |
| ⑥ Client VPN | 不要 | VPNクライアント | ◎ | △ やや高い | △ | リモートワーク |
| ⑦ Site-to-Site VPN | 不要 | VPN装置 | ◎ | ○ | △ | オフィス全体 |
| ⑧ Direct Connect | 不要 | 専用線 | ◎ | × 高い | × 難しい | エンタープライズ |

---

## どれを選べばいい？ フローチャート

迷ったときは、以下の判断基準で絞り込んでみてください。

```
Q1. 検証・一時利用？
  → Yes → ① パブリックIP or ③ EIC Endpoint
  → No → Q2へ

Q2. オンプレミスとの接続が必要？
  → Yes → Q3へ
  → No → Q4へ

Q3. 安定した帯域・低遅延が必須？
  → Yes → ⑧ Direct Connect（+ ⑦ Site-to-Site VPNをバックアップ）
  → No → ⑦ Site-to-Site VPN

Q4. 個人のPCから複数のVPCリソースにアクセスしたい？
  → Yes → ⑥ Client VPN
  → No → Q5へ

Q5. Windows EC2をブラウザからGUI操作したい？
  → Yes → ⑤ Fleet Manager
  → No → ④ Session Manager or ③ EIC Endpoint
```

---

## まとめ

この記事では、VPC上のEC2に接続する8つの方法を解説しました。

「とりあえず試したい」なら**①パブリックIP**か**③EIC Endpoint**、企業で標準運用するなら**④Session Manager**、オンプレミスとの接続が必要なら**⑦Site-to-Site VPN**や**⑧Direct Connect**、リモートワーク環境なら**⑥Client VPN**が候補になります。

大切なのは、自分の環境・要件に合った方法を選ぶことです。この記事が「結局どれを使えばいいの？」という疑問の解消に役立てば幸いです。

---

## 参考URL

- [Amazon EC2 インスタンスへの接続 - AWS公式ドキュメント](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/connect-to-linux-instance.html)
- [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/session-manager.html)
- [EC2 Instance Connect Endpoint](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/connect-using-eice.html)
- [AWS Client VPN 管理者ガイド](https://docs.aws.amazon.com/ja_jp/vpn/latest/clientvpn-admin/what-is.html)
- [AWS Site-to-Site VPN](https://docs.aws.amazon.com/ja_jp/vpn/latest/s2svpn/VPC_VPN.html)
- [AWS Direct Connect](https://docs.aws.amazon.com/ja_jp/directconnect/latest/UserGuide/Welcome.html)
- [AWS Systems Manager Fleet Manager](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/fleet.html)
