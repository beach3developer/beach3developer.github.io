---
title: "古いPC上に構築したUbuntuにVirtualBoxで再生！手軽に使える検証用サーバー環境を構築する手順まとめ"
publishDate: 2025-11-20 00:00:00
img: /assets/blog/SOLO_IT_PERSONNEL.png
img_alt: 古いPCをUbuntu＋VirtualBoxで再生する手順まとめ
description: 
  古いWindowsマシンをUbuntuに変え、VirtualBoxで複数の検証用サーバーを構築する手順をまとめました。開発・検証環境を自宅に作りたいエンジニア向け。
tags:
  - Ubuntu
  - Virtualbox
  - 検証環境
  - 自宅サーバー
---

## ●Ubuntu上に複数システムを運用できるように（もしくは社内検証用に）環境を整える

### 不要になった古いPCにUbuntuをインストールする。

こちらの手順については別記事で紹介しています。  
今回は、すでにUbuntuをインストールした環境にVirtualBoxを導入し、**手軽に検証用サーバーを構築する方法**を紹介します。

実際に手を動かしてみることで理解が深まります。  
「百聞は一見にしかず」という言葉のとおり、古いPCを検証用マシンに変えて、思い切って試してみましょう。

👉 [古いWindows PCをUbuntuサーバーに再生！自宅で検証環境をつくる手順まとめ](https://www.beach3blog.com/blog/20251106_setup_ubuntu_server/)

---

## そもそもVirtualboxとは何か？

**VirtualBox（バーチャルボックス）**とは、  
**1台のPC上で複数の仮想マシン（VM）を動かすためのソフトウェア**です。  
Oracle社が開発・提供しており、無料で利用できます。

### ● 仮想化とは？

仮想化とは、物理的なハードウェアをソフトウェア的に分割・再現し、  
複数のOSを同時に動かせるようにする技術のことです。

たとえば：
- Ubuntu上でWindowsを動かす  
- 1台のPC上で複数のUbuntuサーバーを同時に起動する  
といったことが可能になります。

### ● VirtualBoxを使うメリット

- **GUIベースで使いやすい**（初心者でも理解しやすい）
- **無料・オープンソース**
- **UbuntuやWindowsでも動作**
- **VMごとにスナップショットで環境を保存可能**

DockerやKVMと違い、**OSレベルの完全仮想化**を行うため、  
本番サーバーのような動作検証が可能です。

---

### VirtualBox構築手順

Ubuntuのシステムを最新化してからVirtualBoxをインストールします。


### 1. パッケージの更新とアップグレード

```bash
sudo apt update
sudo apt upgrade -y
```

### 2. VirtualBox公式リポジトリを追加してインストール

以下は VirtualBox 7.1（執筆時点の最新版）の例です。
Ubuntu 24.04（Noble）環境を想定しています。

```bash
wget -O- https://www.virtualbox.org/download/oracle_vbox_2016.asc | sudo gpg --dearmor --yes --output /usr/share/keyrings/oracle-virtualbox-2016.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/oracle-virtualbox-2016.gpg] http://download.virtualbox.org/virtualbox/debian noble contrib" | sudo tee /etc/apt/sources.list.d/virtualbox.list
sudo apt update
sudo apt install virtualbox-7.1
```

### 3. 現在のユーザーを vboxusers グループに追加

```bash
sudo usermod -aG vboxusers $USER
```

この設定は再ログイン後に反映されます。
GUI版VirtualBoxを使う場合、この操作を忘れるとVMが起動できないことがあるため注意。

---

### （必要に応じて）KVMとの競合回避設定

Ubuntuにはデフォルトで **KVM（別の仮想化モジュール）**が有効になっている場合があります。
VirtualBoxを使用する際に競合する場合は、以下を設定して無効化します。


```bash
sudo vi /etc/modprobe.d/disable-kvm.conf
```

[/etc/modprobe.d/disable-kvm.conf]に記載する内容は以下になります。

以下を追記：
```conf
install kvm /bin/true
install kvm_intel /bin/true
install kvm_amd /bin/true
```

変更を反映：
```bash
sudo update-initramfs -u
sudo reboot -n
```

---

### VirtualBoxの起動と仮想マシン作成

再起動後、VirtualBox を起動します。
（GUIランチャーまたはターミナルで virtualbox と入力）

#### OSイメージのダウンロード

今回はUbuntu Serverを使用するので、以下から入手します。

👉 [https://jp.ubuntu.com/download](https://jp.ubuntu.com/download)

#### 新しい仮想マシンを作成

1. 「新規」をクリック
2. 名前：test-server
3. OSタイプ：先ほどダウンロードしたファイル
4. メモリ：1〜2GB
5. 仮想ディスク：20GB（VDI, 動的割り当て）

#### 作成したVMをホスト側のPC立ち上げ時に自動起動にする

- VM名を確認する

```bash
VBoxManage list vms
```

- systemd ユニットを作成

```bash
sudo nano /etc/systemd/system/vboxvm.service
```

```ini
[Unit]
Description=Start VirtualBox VM
After=network.target vboxdrv.service

[Service]
User=[USER名]
Group=[GROUP名]
Type=simple
ExecStart=/usr/bin/VBoxManage startvm "【VM名】" --type headless
ExecStop=/usr/bin/VBoxManage controlvm "【VM名】" acpipowerbutton
TimeoutSec=30
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```
- Systemdを有効化

```bash
sudo systemctl daemon-reload
sudo systemctl enable vboxvm
sudo systemctl start vboxvm
```

再起動して作成したVMが問題なく立ち上がっていることを確認してください。
実行中になっていれば問題なくVMが起動していることになります。



## まとめ

古いPCでも、Ubuntu + VirtualBox を組み合わせることで
複数の仮想サーバーを同時に立ち上げられる 自宅検証環境 を構築できます。

- VirtualBoxは無料で扱いやすい仮想化ツール
- Ubuntuと組み合わせることで低コスト運用が可能
- ネットワーク構成の勉強やテスト環境構築に最適

「古いPCを再利用して自分だけのラボをつくる」
そんな体験を、ぜひ試してみてください。

