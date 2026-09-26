---
title: "Claude DesktopをAmazon Bedrock経由（フリーアカウントプラン）でMacbookにセットアップ"
publishDate: 2026-09-26 00:00:00
img: /assets/blog/2026/anthropic.svg
img_alt: anthropic
description: |
  Claude DesktopをAWS Bedrockの従量課金で使う！Macでのセットアップ手順をHomebrewからトラブルシューティングまで丁寧に解説します。
tags:
  - Claude Code
  - Claude Cowork
  - AWS Bedrock
  - Anthropic
---

## 💻 PC作業が劇的に変わる！「Claude Desktop」の魅力とできること完全解説

「AIツールを使っているけれど、いちいちブラウザを開くのが面倒…」「もっとPCの作業とシームレスに連携してくれたらいいのに」そんな方におすすめなのが、AIコミュニティで今もっとも注目されている生成AI「Claude（クロード）」のデスクトップアプリ版（Claude Desktop） です。

この記事では、初めてClaudeに触れる方にもわかりやすく「Claudeとは何か」という基本から、ブラウザ版にはない「Claude Desktopだからこそできる神機能」まで、詳しく徹底解説します！

## 💡 そもそも「Claude（クロード）」とは？ 3つの特徴

Claude（クロード）は、元OpenAIのメンバーらが設立した米国のAIスタートアップ企業Anthropic（アンソロピック）社が開発した、最高峰の対話型AIサービスです。ChatGPTの強力なライバルとして、世界中のビジネスパーソンやクリエイターから愛用されています。

Claudeがこれほど支持されている理由は、主に次の3つの特徴にあります。

- **人間らしい自然で美しい日本語**
AI特有の「機械っぽさ」が少なく、小説の執筆、メール文面の作成、ブログ記事の添削などを、まるで優秀なライターが書いたような自然な表現で出力してくれます。

- **大量の情報を一瞬で読み込む能力**
一般的な本数冊分に相当する膨大なテキスト（最大100万トークン）を一度に読み込めます。長大な論文や、何十ページもある規約PDFなどを丸ごと読み込ませて要約・分析させることが大得意です。

- **「Artifacts（アーティファクト）」機能が超便利**
チャット中にClaudeが作成したソースコード、HTMLのデザイン、図表、長文テキストなどを、チャット画面の右側に「独立した画面」として表示・プレビューしてくれる機能です。いちいちコードをコピーして別のソフトで開く必要がありません。

## 🚀 デスクトップアプリ版「Claude Desktop」でできること

「ブラウザ版（Webサイト）があるなら、わざわざアプリをインストールしなくてよくない？」と思うかもしれません。しかし、Claude DesktopはPC作業に完全に密着した設計になっており、ブラウザ版を遥かに凌ぐ快適さと強力な機能を持っています。

具体的にできる「4つの神機能」をご紹介します。

1. **グローバルショートカットで「いつでも瞬時に呼び出し」**

    ブラウザ版のように「タブを探す」「お気に入りから開く」といった手間は一切不要です。PCで別の作業（ExcelやWordの編集、ブラウジングなど）をしていても、専用のショートカットキー（例：Macなら `Option + Space`）を押すだけで、画面中央に小さな入力窓（クイックエントリー機能）がポップアップします。思いついたときにすぐAIに質問できるため、作業の集中力が途切れません。

2. **PC内のファイルやアプリとの高度な連携（MCP対応）**

    Claude Desktopの最大の強みは、「MCP（Model Context Protocol）」と呼ばれる外部連携機能に対応している点です。これにより、あなたのパソコン内のローカルフォルダにあるファイルを直接参照させたり、開発環境や他のデスクトップアプリと繋ぎ込んで、ファイル操作やデータ連携などの複雑なタスクをClaudeに任せることができるようになります。

3. **PC作業を自動化する強力なエージェント「Claude Cowork / Code」**

    アプリ上では、通常のチャット以外に、特定の業務に特化したエージェント機能をシームレスに利用できます。

    - **Claude Cowork**： 非エンジニア向けの業務自動化エージェント。Excelのデータ整理やVLOOKUP関数の自動設定、毎朝のニュース自動収集（Routines機能）など、オフィスの定型業務を身代わりになって実行してくれます。
    - **Claude Code**： 開発者向けのコーディングエージェント。PC上の指定したフォルダ内で自律的にコードを書き進めたり、エラーをデバッグしたりしてくれるため、開発スピードが爆発的に向上します。

4. **画面が広く、作業スペースとして快適**

    ブラウザの他のタブに邪魔されることなく、全画面を使ってClaudeと1対1で向き合えます。特に前述の「Artifacts（プレビュー画面）」を開きながら作業をする際、デスクトップアプリの広いウィンドウは抜群に使いやすく、ドキュメント作成やプログラミングの効率が大きく上がります。

## 💭 Amazon Bedrock経由でClaude Desktopを使うメリット

一番ポピュラーな利用方法は利用するユーザー分のライセンスを契約して月額や年契約での支払いになることが多いかと思います。しかし、組織によっては個人によって差があったり部署によって利用頻度にばらつきがあると思います。また、他のAIと比べて使いやすさや機能差を確認するために使ってみたい。といった方にも使った分だけの費用が請求できるような従量課金という使い方がいいのではないでしょうか。

個人的には従量課金で利用できる点がこのやり方の一番のメリットになるかと思いますが、AWSがブログでメリットについてまとめているので、詳細はAWSブログを見てみてください。

- [Claude Code / Claude Cowork を Amazon Bedrock 経由で組織利用する４つのメリット](https://aws.amazon.com/jp/blogs/startup/claude-code-cowork-amazon-bedrock-benefits/)

## 👨‍💻 利用方法

ここからは具体的なセットアップ手順を、**AWS管理者が行う作業**と**Mac利用者が行う作業**の2パートに分けて解説します。

Windows版のセットアップ記事はすでに公開されていますので、この記事では**Mac（macOS）固有のターミナル操作やHomebrew**を使った手順に焦点を当てて解説します。

---

### 🔧 STEP 1：AWS管理者作業（IAMの準備）

Claude DesktopがBedrock APIを呼び出すために、適切な権限を持ったIAMユーザーが必要です。AWS管理者が以下の手順で準備してください。

#### 1-1. IAMポリシーの作成

AWSマネジメントコンソールにログインし、**IAM** サービスに移動します。

左メニューから **「ポリシー」** → **「ポリシーを作成」** をクリックし、**「JSON」タブ**に切り替えて以下のポリシーを貼り付けます。

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCoworkBedrockBearerToken",
            "Effect": "Allow",
            "Action": [
                "bedrock:CallWithBearerToken",
                "bedrock:InvokeModel",
                "bedrock:ListFoundationModels",
                "bedrock:ListInferenceProfiles",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": "*"
        },
        {
            "Sid": "AllowMarketplaceModelSubscription",
            "Effect": "Allow",
            "Action": [
                "aws-marketplace:Subscribe",
                "aws-marketplace:Unsubscribe",
                "aws-marketplace:ViewSubscriptions"
            ],
            "Resource": "*"
        }
    ]
}
```

**「次へ」** をクリックし、ポリシー名を入力します（例：`ClaudeDesktopBedrockPolicy`）。わかりやすい説明を添えて **「ポリシーの作成」** をクリックすれば完了です。

> **ポイント：** `AllowCoworkBedrockBearerToken` はClaude DesktopがBedrockのモデルを呼び出すための権限、`AllowMarketplaceModelSubscription` はBedrockのモデルアクセスを有効化する際に必要な権限です。

#### 1-2. IAMユーザーの作成

続けてIAMユーザーを作成します。

1. IAMの左メニューから **「ユーザー」** → **「ユーザーを作成」** をクリック
2. ユーザー名を入力します（例：`claude-desktop-user`）
3. **「AWS マネジメントコンソールへのユーザーアクセスを提供する」のチェックは外したまま**にして「次へ」
4. **「ポリシーを直接アタッチする」** を選択し、先ほど作成した `ClaudeDesktopBedrockPolicy` を検索してチェックを入れます
5. **「次へ」** → 内容を確認して **「ユーザーの作成」** をクリック

#### 1-3. アクセスキーの発行

作成したIAMユーザーのアクセスキーを発行します。

1. 作成したユーザー（例：`claude-desktop-user`）をクリックしてユーザー詳細画面を開く
2. **「セキュリティ認証情報」** タブをクリック
3. 「アクセスキー」セクションの **「アクセスキーを作成」** をクリック
4. ユースケースは **「コマンドラインインターフェイス (CLI)」** を選択
5. 確認のチェックボックスにチェックを入れて **「次へ」** → **「アクセスキーを作成」**
6. **アクセスキーID**と**シークレットアクセスキー**が表示されるので、**必ずこの画面でメモしてください**（シークレットアクセスキーは二度と表示されません）

> **⚠️ 注意：** アクセスキーは絶対に外部に漏らさないでください。万が一漏洩した場合は、すぐにIAMコンソールから該当キーを無効化・削除してください。

---

### 🍎 STEP 2：PCセットアップ作業（Mac利用者個人）

ここからはMacを使う利用者自身の作業です。ターミナル（Terminal.app）を使った操作が中心になりますが、一つずつ丁寧に進めていきましょう。

#### 2-1. Homebrewのインストール（未導入の場合）

MacでCLIツールを管理するためのパッケージマネージャ「Homebrew」を使います。すでにインストール済みの方はスキップしてください。

ターミナルを開いて以下のコマンドを実行します。

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

インストール完了後、Homebrewが使えることを確認します。

```bash
brew --version
```

バージョン番号が表示されればOKです。

> **Apple Silicon（M1/M2/M3/M4）のMacをお使いの方へ：** Homebrewのインストール先が `/opt/homebrew` になります。インストール完了時に表示される指示に従って、以下のコマンドでPATHを通してください。
>
> ```bash
> echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
> eval "$(/opt/homebrew/bin/brew shellenv)"
> ```

#### 2-2. AWS CLIのインストール

AWS CLIをHomebrewでインストールします。

```bash
brew install awscli
```

インストール完了後、バージョンを確認します。

```bash
aws --version
```

`aws-cli/2.x.x` のようにバージョンが表示されれば成功です。

> **Homebrew以外の方法：** AWS公式のpkgインストーラを使う方法もあります。詳しくは [AWS CLI公式ドキュメント](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html) を参照してください。

#### 2-3. AWS CLIの初期設定（プロファイル作成）

管理者から受け取ったアクセスキーを使って、AWS CLIのプロファイルを設定します。ここでは `claude-bedrock` という名前付きプロファイルを作成します（デフォルトプロファイルを汚さないためです）。

```bash
aws configure --profile claude-bedrock
```

対話形式で以下の4項目を聞かれるので、順番に入力します。

```
AWS Access Key ID [None]: （管理者から受け取ったアクセスキーIDを入力）
AWS Secret Access Key [None]: （管理者から受け取ったシークレットアクセスキーを入力）
Default region name [None]: us-east-1
Default output format [None]: json
```

> **リージョンについて：** Claudeモデルが利用可能なリージョンを指定してください。`us-east-1`（バージニア北部）が最も多くのモデルに対応しています。東京リージョン（`ap-northeast-1`）でも一部モデルが利用可能です。

設定が正しく保存されたか確認するには、以下のコマンドを実行します。

```bash
aws sts get-caller-identity --profile claude-bedrock
```

以下のようなJSON出力が返ってくれば、認証情報は正しく設定されています。

```json
{
    "UserId": "AIDAXXXXXXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/claude-desktop-user"
}
```

#### 2-4. Bedrockモデルアクセスの有効化

AWSマネジメントコンソールにログインし、リージョンを `us-east-1`（または設定したリージョン）に切り替えます。

1. サービス検索で **「Amazon Bedrock」** を開く
2. 左メニューから **「モデルアクセス」** をクリック
3. **「モデルアクセスを変更」** をクリック
4. **Anthropic** のセクションから、利用したいClaudeモデル（例：Claude Sonnet 4, Claude Opus 4 など）にチェックを入れる
5. **「変更を保存」** をクリック

> **注意：** モデルアクセスの有効化には数分かかる場合があります。ステータスが「アクセスが付与されました」に変わるまで待ちましょう。

#### 2-5. Claude Desktopのインストール

Anthropicの公式サイトからClaude Desktopをダウンロードします。

1. [https://claude.ai/download](https://claude.ai/download) にアクセス
2. **「macOS」** 版をダウンロード
3. ダウンロードされた `.dmg` ファイルを開く
4. Claude のアイコンを **Applications** フォルダにドラッグ＆ドロップ
5. Applicationsフォルダから **Claude** を起動

初回起動時にAnthropicアカウントへのサインインを求められますが、**フリーアカウント（無料）でOK**です。Anthropicアカウントを持っていない場合はメールアドレスで新規登録してください。

#### 2-6. Claude DesktopのBedrock接続設定

ここが最も重要なステップです。Claude Desktopの設定ファイルを編集して、Bedrock経由でモデルを利用するよう設定します。

##### 設定ファイルの場所

macOSでのClaude Desktopの設定ファイルは以下のパスにあります。

```
~/Library/Application Support/Claude/settings.json
```

> **補足：** `~` はホームディレクトリ（`/Users/あなたのユーザー名`）を意味します。Finderからは見えにくい場所にあるため、ターミナルで操作するのが確実です。

##### 設定ファイルの編集

まず、設定ファイルが存在するディレクトリに移動します（初回の場合、ディレクトリやファイルがまだ存在しない可能性があるため、作成も行います）。

```bash
# ディレクトリの作成（すでにある場合はスキップされます）
mkdir -p ~/Library/Application\ Support/Claude

# 設定ファイルを開く（viエディタの場合）
vi ~/Library/Application\ Support/Claude/settings.json
```

> **viエディタに不慣れな方へ：** `nano` や VS Code を使うこともできます。
>
> ```bash
> # nanoエディタの場合
> nano ~/Library/Application\ Support/Claude/settings.json
>
> # VS Codeの場合（VS Codeがインストール済みなら）
> code ~/Library/Application\ Support/Claude/settings.json
> ```

以下の内容を記述して保存します。

```json
{
  "primaryProviderConfig": {
    "type": "bedrock",
    "awsRegion": "us-east-1",
    "awsProfile": "claude-bedrock"
  }
}
```

各項目の説明：

| キー | 値 | 説明 |
|---|---|---|
| `type` | `"bedrock"` | プロバイダとしてAmazon Bedrockを使用する指定 |
| `awsRegion` | `"us-east-1"` | Bedrockモデルを利用するAWSリージョン |
| `awsProfile` | `"claude-bedrock"` | STEP 2-3で作成したAWS CLIプロファイル名 |

> **環境変数で指定する方法もあります：** プロファイルではなく環境変数 `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` / `AWS_DEFAULT_REGION` をセットする方法もありますが、プロファイル方式のほうが認証情報の管理がしやすくおすすめです。

##### 設定の反映

設定ファイルを保存したら、**Claude Desktopを完全に終了して再起動**します。

```bash
# メニューバーのClaudeアイコンを右クリック → 「Quit Claude」
# または、ターミナルから強制終了する場合：
killall Claude
```

その後、ApplicationsフォルダやSpotlight（`Cmd + Space` → 「Claude」と入力）からClaude Desktopを再度起動します。

#### 2-7. 動作確認

Claude Desktopが起動したら、Bedrock経由で正常に動作しているか確認しましょう。

1. チャット画面で何かメッセージを送信してみます（例：「こんにちは！」）
2. 正常にClaudeから応答が返ってくれば成功です
3. 画面左下や設定画面でプロバイダが **「Amazon Bedrock」** になっていることを確認できます

---

## 🔥 トラブルシューティング

セットアップ中によくあるエラーと対処法をまとめました。

### ❌ 「認証エラー」が表示される場合

**原因：** AWS CLIのプロファイル設定が正しくないか、アクセスキーが無効になっている可能性があります。

**対処法：**

```bash
# プロファイルの設定内容を確認
aws configure list --profile claude-bedrock

# 認証が通るかテスト
aws sts get-caller-identity --profile claude-bedrock
```

エラーが返る場合は、アクセスキーIDとシークレットアクセスキーを再確認し、`aws configure --profile claude-bedrock` で再設定してください。

### ❌ 「モデルにアクセスできません」と表示される場合

**原因：** Bedrockのモデルアクセスが有効化されていない、またはリージョンが一致していない可能性があります。

**対処法：**

1. AWSコンソールでBedrockの「モデルアクセス」画面を開き、Claudeモデルが「アクセスが付与されました」になっているか確認
2. `settings.json` の `awsRegion` が、モデルアクセスを有効化したリージョンと一致しているか確認

### ❌ `settings.json` を編集してもClaude Desktopに反映されない場合

**原因：** Claude Desktopが完全に終了していない可能性があります。macOSではウィンドウを閉じてもアプリがバックグラウンドで動作し続けることがあります。

**対処法：**

```bash
# Claude Desktopのプロセスを確認
ps aux | grep -i claude

# プロセスが残っていれば強制終了
killall Claude
```

その後、再度Claude Desktopを起動してください。

### ❌ Homebrewのインストールで `Permission denied` が出る場合

**原因：** macOSのセキュリティ設定やディレクトリ権限の問題です。

**対処法：**

```bash
# Xcodeコマンドラインツールを先にインストール
xcode-select --install

# その後、Homebrewのインストールを再実行
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### ❌ `aws: command not found` と表示される場合

**原因：** AWS CLIへのPATHが通っていない可能性があります。

**対処法：**

```bash
# Homebrewのパスが通っているか確認
which brew

# brewのパスが表示されない場合（Apple Silicon Mac）
eval "$(/opt/homebrew/bin/brew shellenv)"

# AWS CLIを再度確認
which aws
aws --version
```

---

## 💰 料金について

Amazon Bedrock経由の場合、Anthropicへのサブスクリプション料金は不要で、**AWSの従量課金のみ**で利用できます。料金はモデルごとに異なり、入力トークンと出力トークンそれぞれに課金されます。

最新の料金は [Amazon Bedrock の料金ページ](https://aws.amazon.com/jp/bedrock/pricing/) で確認してください。

> **コスト管理のコツ：** AWS Budgetsを使って月額の上限アラートを設定しておくと、想定外の費用が発生した際にすぐ気付けて安心です。

---

## 📝 まとめ

この記事では、MacbookにClaude DesktopをAmazon Bedrock経由でセットアップする手順を解説しました。手順をまとめると以下のとおりです。

| ステップ | 作業内容 | 担当 |
|---|---|---|
| 1-1 | IAMポリシーの作成 | AWS管理者 |
| 1-2 | IAMユーザーの作成 | AWS管理者 |
| 1-3 | アクセスキーの発行 | AWS管理者 |
| 2-1 | Homebrewのインストール | Mac利用者 |
| 2-2 | AWS CLIのインストール | Mac利用者 |
| 2-3 | AWS CLIプロファイルの設定 | Mac利用者 |
| 2-4 | Bedrockモデルアクセスの有効化 | AWS管理者 or Mac利用者 |
| 2-5 | Claude Desktopのインストール | Mac利用者 |
| 2-6 | settings.jsonの編集 | Mac利用者 |
| 2-7 | 動作確認 | Mac利用者 |

従量課金で利用できるため、まずは小さく試してみて、チームへの展開を検討するのも良いのではないでしょうか。

---

## ●参考URL

- [Claude Desktop (Cowork) を Amazon Bedrock 経由で Windows にセットアップ](https://zenn.dev/aws_japan/articles/aws-bedrock-claude-cowork-setup)
- [Claude Desktop (Cowork) を Amazon Bedrock 経由で利用してみた](https://dev.classmethod.jp/articles/amazon-bedrock-claude-desktop-cowork-3p-inference/)
- [Claude Code / Claude Cowork を Amazon Bedrock 経由で組織利用する４つのメリット](https://aws.amazon.com/jp/blogs/startup/claude-code-cowork-amazon-bedrock-benefits/)
- [AWS CLI のインストール](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html)
- [Amazon Bedrock の料金](https://aws.amazon.com/jp/bedrock/pricing/)
