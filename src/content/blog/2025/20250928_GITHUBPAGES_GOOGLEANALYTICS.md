---
title: "GitHub Pagesを使って無料で個人サイトを公開する方法"
publishDate: 2025-09-28 00:00:00
img: /assets/blog/2025/GitHub_Logo.png
img_alt: GitHub
description: |
  静的サイトジェネレーターAstroで作成したブログをGitHub Pagesで無料公開する手順を完全解説。Node.jsやGitの設定からActionsでの自動デプロイ、Analytics導入まで詳しく説明。
tags:
  - GitHub Pages
  - Astro
  - Google Analytics
---
#### 1.導入
この記事では、**Astroで作成したブログをGitHub Pagesで公開する方法**を解説します。

- **この記事のゴール**
  - Astroで作ったブログをGitHub Pagesで公開できるようになること
- **対象読者**
  - 初めて自分のサイトを公開してみたい方
  - 無料で公開したい方
- **完成イメージ**
  - [https://beach3developer.github.io/](https://beach3developer.github.io/)

#### 2.前提条件
以下の環境・アカウントが準備済みであることを前提とします。

- **Googleアカウントを持っていること**  
  持っていない場合は[こちら](http://google.com/intl/ja/account/about/)から「アカウントを作成する」を選んで作成してください。
- **Node.js / npm がインストール済み（v22.20.0以上）**  
  未インストールの場合は[Node.js公式サイト](https://nodejs.org/ja/download)からダウンロード・インストールしてください。
- **Gitがインストール済みであること**  
  未インストールの場合は[Git公式サイト](https://git-scm.com/downloads)からインストールしてください。


#### 3.個人サイト用のAstroプロジェクトの準備
ここでは、このブログでも使用しているサンプルテーマ「[Portfolio](https://astro.build/themes/details/portfolio/)」を使用します。  
以下のコマンドをターミナルで実行してください。

```
npm create astro@latest -- --template portfolio
```

インストール時にヒアリング事項があるので、今回は以下の内容で回答しています。

>Where should we create your new project?

こちらはプロジェクトをどこで作成するかなので、作成したいPathを指定し、フォルダ名もここで記載します。
例） ./MySite

>Install dependencies?

プロジェクトの依存関係のインストールを聞かれるのでここではYesを選択してください。

>Initialize a new git repository?

新しく Git リポジトリを初期化するか聞かれるのでYesで初期化しておきます。
※本手順では GitHubへコミットする際は一旦削除してからコミットすることにします。


問題なく実行できた場合に対話形式で回答が必要なので記載通りにすると以下のような表示になるかと思います。

```
> npx
> create-astro --template portfolio


 astro   Launch sequence initiated.

   dir   Where should we create your new project?
         ./MySite
      ◼  tmpl Using portfolio as project template

  deps   Install dependencies?
         Yes

   git   Initialize a new git repository?
         Yes

      ✔  Project initialized!
         ■ Template copied
         ■ Dependencies installed
         ■ Git initialized

  next   Liftoff confirmed. Explore your project!

         Enter your project directory using cd ./MySite 
         Run npm run dev to start the dev server. CTRL+C to stop.
         Add frameworks like react or tailwind using astro add.

         Stuck? Join us at https://astro.build/chat

╭─────╮  Houston:
│ ◠ ◡ ◠  Good luck out there, astronaut! 🚀
╰─────╯ 
```

ここまで終わったら、まずはサイトをローカルで表示してみましょう。

```
% cd MySite 
% npm run dev

> mysite@0.0.1 dev
> astro dev

13:51:42 [types] Generated 0ms
13:51:42 [content] Syncing content
13:51:42 [content] Synced content

 astro  v5.13.11 ready in 167 ms

┃ Local    http://localhost:4321/
┃ Network  use --host to expose

```
ブラウザで http://localhost:4321/ を開き、READMEと同じ画像が表示されれば成功です。
画像や背景、配色などは自由に変更できます。まずは動作確認をしてから、自分好みにカスタマイズしてみてください🙌

#### 4.GitHub Pages用の設定

1. __GitHubリポジトリを作成__<br />

[GitHub](https://github.com/login)にログインします。
Googleアカウントを使う場合は「Continue with Google」をクリックし、使用するGoogleアカウントで認証してください。
ログイン後、画面右上の緑色ボタン「Create Repository」をクリックし、新しいリポジトリを作成します。
リポジトリ名は【[USERNAME].github.io】とし、その他はデフォルトのまま「Create repository」をクリックしてください。

2. __ブランチ保護ルールを設定する（第三者が更新できないようにする）__<br />

[Settings] → [New ruleset] → [New branch ruleset] から以下のように設定します。

- Ruleset Name
  - main
- Enforcement status
  - Active
- Bypass list
  - Repository admin
- Target branches
  - All branches
- Rules
  - Restrict creations:true
  - Restrict updates:true
  - Restrict deletions:true
  - Require linear history:true
  - Require deployments to succeed:false
  - Require signed commits:true
  - Require a pull request before merging:false
  - Require status checks to pass:false
  - Block force pushes:true
  - Require code scanning results:false
  - Automatically request Copilot code review:false

「Create」をクリックし、[Ruleset created]と表示されれば完了です。

3. __GitHub Actionsで自動デプロイを設定する__

MySite/.github/workflows フォルダを作成し、deploy.yml を置きます。
（Google Analyticsの測定IDを環境変数で扱う設定も含みます）

```
# Sample workflow for building and deploying an Astro site to GitHub Pages
#
# To get started with Astro see: https://docs.astro.build/en/getting-started/
#
name: Deploy Astro site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["main"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

env:
  PUBLIC_G_TAG: ${{secrets.PUBLIC_G_TAG}}
  BUILD_PATH: "." # default value when not using subfolders
  # BUILD_PATH: subfolder

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Detect package manager
        id: detect-package-manager
        run: |
          if [ -f "${{ github.workspace }}/yarn.lock" ]; then
            echo "manager=yarn" >> $GITHUB_OUTPUT
            echo "command=install" >> $GITHUB_OUTPUT
            echo "runner=yarn" >> $GITHUB_OUTPUT
            echo "lockfile=yarn.lock" >> $GITHUB_OUTPUT
            exit 0
          elif [ -f "${{ github.workspace }}/package.json" ]; then
            echo "manager=npm" >> $GITHUB_OUTPUT
            echo "command=ci" >> $GITHUB_OUTPUT
            echo "runner=npx --no-install" >> $GITHUB_OUTPUT
            echo "lockfile=package-lock.json" >> $GITHUB_OUTPUT
            exit 0
          else
            echo "Unable to determine package manager"
            exit 1
          fi
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: ${{ steps.detect-package-manager.outputs.manager }}
          cache-dependency-path: ${{ env.BUILD_PATH }}/${{ steps.detect-package-manager.outputs.lockfile }}
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install dependencies
        run: ${{ steps.detect-package-manager.outputs.manager }} ${{ steps.detect-package-manager.outputs.command }}
        working-directory: ${{ env.BUILD_PATH }}
      - name: Check env
        run: echo "PUBLIC_G_TAG=${PUBLIC_G_TAG}"
      - name: Build with Astro
        run: |
          ${{ steps.detect-package-manager.outputs.runner }} astro build \
            --site "${{ steps.pages.outputs.origin }}" \
            --base "${{ steps.pages.outputs.base_path }}"
        working-directory: ${{ env.BUILD_PATH }}
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ${{ env.BUILD_PATH }}/dist

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    needs: build
    runs-on: ubuntu-latest
    name: Deploy
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

#### 5.Google アナリティクスの設定

Googleアカウントでログインした状態で[アナリティクス](https://developers.google.com/analytics?hl=ja)を開き、以下の手順で設定します。
Googleアナリティクスについては全くの知見がないので、個人サイトをやりながら学んでいこうと思います。
学んだ内容も投稿していけたらと思います。

1. アカウントの作成

アカウント名は必須なので、本個人サイト用を見分けるための名前を好きにつけてください。
アカウントのデータ共有設定は任意のまま進めます。

2. プロパティの作成

プロパティ名をこちらもつけていきます。
レポートのタイムゾーンと通貨を日本で設定して「次へ」を押下します。

3. お店やサービスの詳細

業種とビジネスの規模を選択していきます。
今回は、業種を「ビジネス、産業」でビジネス規模は、「小規模 - 従業員数 1～10 名」を選択し「次へ」を押下します。

4. ビジネス目標

ビジネス目標を選択するために、2つ以下にすることを推奨しているので、
今回の用途であれば以下の二つにしておきます。

  - ウェブ/アプリのトラフィックの分析
  - ユーザーエンゲージメントとユーザー維持率の把握

選択後に「作成」をクリック後にGoogleアナリティクス利用規約の確認が表示されますので、同意してください。

5. データの収集

「データ収集を開始する」が表示されますので、「ウェブ」を選択しウェブサイトのURLとストリーム名を入力する。入力後に右上の「作成して続行」をクリックする。<br />
・ウェブサイトのURL：https://[GitHubのユーザー名].github.io
<br />
・ストリーム名：MySite

これでGoogleアナリティクスの方は設定が完了になります。次項でAstroにタグ設定を行っていきます。

#### 6.AstroにGoogle Analyticsタグを追加する
./src/components/GoogleAnalytics.astro を作成し、以下を記載します。

- ./src/components/GoogleAnalytics.astro

```
---
const gtagId = import.meta.env.PUBLIC_G_TAG;
---

<!-- Google tag (gtag.js) -->
<script type="text/partytown" async src={`https://www.googletagmanager.com/gtag/js?id=${gtagId}`}></script>
<script is:inline define:vars={{ gtagId: gtagId }} type="text/partytown">
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', gtagId);
</script>
```

- ./src/layouts/BaseLayout.astro

```
---
// Learn about using Astro layouts:
// https://docs.astro.build/en/core-concepts/layouts/

import Footer from '../components/Footer.astro';
// Component Imports
import MainHead from '../components/MainHead.astro';
import Nav from '../components/Nav.astro';
import GoogleAnalytics from '../components/GoogleAnalytics.astro';

interface Props {
  title?: string | undefined;
  description?: string | undefined;
}

const { title, description } = Astro.props;
---

<html lang="en">
  <head>
    <MainHead title={title} description={description} />
    <GoogleAnalytics />
  </head>
  <body>
    <div class="stack backgrounds">
      <Nav />
      <slot />
      <Footer />
    </div>

-----以下は変更がないため省略-----
```

./src/layouts/BaseLayout.astro に <GoogleAnalytics /> を追加します。
GitHubのSecretsに PUBLIC_G_TAG を登録しておくことで、環境変数経由でタグIDを渡せます。

- GitHubでの設定方法
  - GitHubへログインし、リポジトリから【[USERNAME].github.io】を開く
  - [Settings]を開き、[Secrets and variables/Actions]を開く
  - Repository secretsの「New repository secret」を押下する
  - New secretで以下の設定し、「Add secret」を押下して登録する
    - Name:PUBLIC_G_TAG
    - Secret:【Googleアナリティクスの測定ID】

これで公開に向けた設定が整いました。

#### 7.GitHub Pagesで公開

GitHub上のリポジトリで [Settings] → [Pages] を開き、Build and deploymentのSourceで[GitHub Actions]を選択します。

ローカルからGitHubへコードをプッシュする場合は以下のコマンドを実行してください。

```
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin 【GitHub上のURL】
git push -u origin main
```

#### 8.公開後の確認

プッシュ後、GitHub Actionsが自動でデプロイを行います。
以下のURLでサイトが表示されるか確認してください。

URL：https://beach3developer.github.io/

Googleアナリティクスのリアルタイムレポートでアクティブユーザーが表示されるかも併せて確認しましょう。

#### 9.まとめ

以上の手順で、Astroを使った個人サイトを無料で公開できます。
これからもAstroの学習を進めつつ、サイトのデザインやコンテンツを充実させていきたいと思います。
画像はGitHub Pagesの容量制限（1GB）を考慮し、1記事につき1枚までとするなど運用にも工夫していきます。
