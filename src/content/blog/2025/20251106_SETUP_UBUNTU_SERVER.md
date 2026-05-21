---
title: "古いWindows PCをUbuntuサーバーに再生！自宅で検証環境をつくる手順まとめ"
publishDate: 2025-11-06 00:00:00
img: /assets/blog/2025/Ubuntu.svg
img_alt: Template
description: |
  古いInspironなどのWindows PCをUbuntuに入れ替えて、自宅でActive DirectoryやWebサーバーなどを検証できる環境を構築する手順を紹介します。
tags:
  - Ubuntu
  - 自宅サーバー
  - 検証環境
  - Linux
  - リユース
---
### はじめに：古いPCを「もう一度」活かす

「もう動作が重くて使わなくなったWindows機、どうしよう…」  
そんなPC、実は**Ubuntuを入れるだけで“軽いサーバー”として再利用**できます。

筆者の家でも、かつてWindows 10が入っていた **Dell Inspiron** を使い、  
社内向けシステムの検証用サーバーとして再生しました。  

本記事では、そのセットアップ手順と注意点をまとめます。  
Active Directory検証やDockerの練習にも使えるので、ぜひ挑戦してみてください。

---

### ステップ1：PCのスペック確認と準備

Ubuntuサーバーとして動かすには、以下のような最低スペックが目安です。

| 項目 | 推奨値 |
|------|--------|
| CPU | 2コア以上（Core i3 以降ならOK） |
| メモリ | 4GB以上（8GBあると快適） |
| ストレージ | 64GB以上（SSD推奨） |

#### 🔧 事前にやっておくこと
- **重要データのバックアップ**
- **BIOSでSecure Bootを無効化**
- **USBメモリ（8GB以上）を用意**

---

### ステップ2：Ubuntuのダウンロードとインストールメディア作成

1. 公式サイトからUbuntuをダウンロード  
   👉 [https://ubuntu.com/download](https://ubuntu.com/download)

2. **Ubuntu Desktop** または **Ubuntu Server** のどちらかを選択  
   - GUI操作をしたい場合：**Ubuntu Desktop** →今回はこちらをインストールします。
   - サーバー用途に特化したい場合：**Ubuntu Server**

3. **Rufus**（Windows）や **[balenaEtcher](https://etcher.balena.io/#download-etcher)**（macOS/Linux）を使い、  
   ダウンロードしたISOをUSBメモリに書き込みます。

---

### ステップ3：PCにUbuntuをインストール

1. USBを挿してPCを起動し、BIOSでUSBブートを選択。  
2. 「Install Ubuntu」を選択してインストール開始。  
3. 言語・キーボード・ネットワーク・パーティションを設定。  
   ※他OSを残したい場合は「Ubuntu alongside Windows」を選択可能。  
4. インストール完了後、再起動。

---

### ステップ4：初期設定とSSHの有効化

サーバーとして使うなら、まずリモートで接続できるように設定します。

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
```

同じLAN内の別のPCから接続してみましょう：

```bash
ssh ユーザー名@<UbuntuのIPアドレス>
```

これでディスプレイやキーボードを外しても、ネットワーク経由で操作できます。


### ステップ5：固定IPアドレスの設定（重要）

ルーターでIPが変わると接続できなくなるため、固定IPにしておきます。

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

```yaml
network:
  version: 2
  ethernets:
    enp3s0:
      dhcp4: no
      addresses: [192.168.0.50/24]
      gateway4: 192.168.0.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```

保存して反映：

```bash
sudo netplan apply
```

### ステップ6：便利なサーバー構成にしてみる

| 用途 | コマンド例 |
|------|--------|
| Sambaでファイル共有 | `sudo apt install samba` |
| Dockerで検証環境構築 | `sudo apt install docker.io` |
| Webサーバー構築 | `sudo apt install nginx` |
| Active Directory検証	| `sudo apt install samba-ad-dc` |

これらを入れておけば、社内システムのテスト環境として十分活用できます。

### ステップ7：自宅サーバー運用のポイント

- 定期的に `sudo apt upgrade` を実行して更新
- UPS（無停電電源装置）を使うと安全
- ファイアウォール（ufw）の設定も忘れずに：

```bash
sudo ufw allow ssh
sudo ufw enable
```
### ステップ8：メインPCでリモートデスクトップができるようにする

Ubuntuのデフォルトの設定で画面共有やリモートデスクトップができますが、
ログインしていないと使えなかったりと不便なため別にインストールを進めていきます。

```bash
sudo apt install xrdp -y
sudo systemctl enable xrdp
sudo systemctl start xrdp
sudo ufw allow 3389
```

上記で設定は問題ないためログイン確認をします。
Macを利用している場合はApp Storeから「Windows App」を使ってログインをしていきます。
mDNSを使ってローカルネットワークであれば名前解決ができため、コンピュータ名で接続をしていきます。

- PC name: [コンピュータ名].local:3389


### まとめ：古いPCが学びと検証の基地になる

Inspironなどの古いWindows機でも、Ubuntuを入れることで
**軽量で安定した検証用サーバー**としてよみがえります。

会社で使うActive Directoryやファイルサーバーの練習、
Webアプリのデプロイ検証など、自宅でも安全に試せます。

この環境を使って **Samba ADを構築し、Windowsクライアントをドメイン参加させる手順** を紹介します。