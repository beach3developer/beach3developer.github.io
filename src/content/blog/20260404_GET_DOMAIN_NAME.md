---
title: "【2026年版】お名前.comで独自ドメイン取得→GitHub Pagesに設定する完全手順（SEO対策あり）"
publishDate: 2026-04-04 00:00:00
img: /assets/blog/2026/ONAMAE.png
img_alt: お名前.com GitHub Pages ドメイン設定
description: |
  お名前.comで取得した独自ドメインをGitHub Pagesに設定する方法を初心者向けに解説。
  DNS設定・Aレコード・CNAME・HTTPS化・Search Console対策まで完全網羅。
tags:
  - Github Pages
  - お名前.com
  - ドメイン取得
  - SEO
  - サイトマップ
---

## ●Google Search Consoleでサイトマップが登録されない問題

GitHub Pagesでブログを公開していましたが、  
Google Search Consoleにサイトマップを登録しても、

- 検出されたページ数：0
- サイトマップを読み込めませんでした

という状態が続いていました。

色々調べると、

> GitHub Pagesはインデックスが遅いことがある

という情報もありましたが、  
せっかくなのでこの機会に**独自ドメイン化**することにしました。

---

## ●結論：独自ドメインにするとSEO的にも有利

独自ドメインにするメリット👇

- Googleに「独立したサイト」として認識されやすい
- インデックスされやすくなる
- ブランド化できる
- URLが短くなる

👉 **ブログを続けるならほぼ必須レベルです**

---

## ●お名前.comでドメインを取得

[お名前.com](https://www.onamae.com/)にアクセスしてドメインを取得します。

### 手順

1. 好きなドメインを検索（例：`beach3blog.com`）
2. 「次へ進む」をクリック
3. 使用用途 → 「ブログ・ホームページ」
4. レンタルサーバー → **削除（重要）**
5. メールアドレス＆パスワード登録
6. クレジットカード登録（必須）

👉 初年度無料の場合もありますが、**自動更新に注意**

---

## ●Whois情報について（重要）

登録後に不安になるポイント👇

> 「個人情報公開されてない？」

結論：

👉 **Whois公開代行がONなら問題なし**

設定確認：
- Whois情報公開代行 → 有効

※触らなくてOK

---

## ●DNS設定（ここが一番重要）

お名前.com管理画面で設定します。

### ① CNAME設定（wwwあり）

| 項目 | 値 |
|------|----|
| ホスト名 | www |
| TYPE | CNAME |
| VALUE | beach3developer.github.io |

---

### ② Aレコード設定（wwwなし）

GitHub Pages指定のIPを設定👇

[GitHub ページ サイトのカスタム ドメインの管理](https://docs.github.com/ja/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)

- 185.199.108.153
- 185.199.109.153
- 185.199.110.153
- 185.199.111.153

設定はレコード追加で以下の画像のように設定をします。

| TYPE | VALUE |
|------|------|
| A | 上記4つすべて |

👉 これをやらないと「ルートドメイン」でアクセスできない

![Screenshot_Onamae.png](/assets/blog/2026/Screenshot_Onamae.png)

---

## ●GitHub Pages側の設定

GitHubリポジトリ → Settings → Pages

**Custom domainに入力**

`www.beach3blog.com`


👉 **ここ超重要**

wwwなし（example.com）にするとハマることが多い

---

## ●HTTPS対応（超重要）

設定後しばらくすると👇

- Enforce HTTPS → 有効化可能

もしダメな場合👇

- DNS反映待ち（最大24時間）
- www設定ミス


！！注意！！
ここのドメインを[`www.XXXXX.com`]にしないといつまで経ってもDNS Check in Progressのままになります。
Progressのままだとその下にある「Enforce HTTPS」にチェックがつけれらないため、HTTPでのアクセスのみになります。
1日経っても「Unavailable for your site because your domain is not properly configured to support HTTPS」となり、チェックを入れることができません。

![Custom domwin](/assets/blog/2026/Screenshot_GithubPagesCustomDomain.png)

[Enforce HTTPS]にチェックを入れることでHTTPSでアクセスができるようになり問題ない接続が可能になります。

---


## ●Astro側の設定（SEO的に必須）

`astro.config.mjs` を修正👇

```js
export default defineConfig({
// @ts-check
import { defineConfig } from 'astro/config';
import mdx from '@astrojs/mdx';
import partytown from "@astrojs/partytown";
import sitemap from '@astrojs/sitemap';

// https://astro.build/config
export default defineConfig({
  site: 'https://beach3blog.com',
  base: '/',
  trailingSlash: 'ignore',
  integrations: [
    partytown({
      config: {
        forward: ['dataLayer.push'],
      },
    }),
    sitemap(),
    mdx()
  ],
});
```

👉 これをやらないと

- sitemapが旧URLになる
- SEO評価が分散する


## ●おわりに

正直、

「サイトマップ登録されない問題」

はかなりハマりました。

ただ今回の対応で、

DNS
SEO
GitHub Pages

の理解がかなり深まりました。

👉 こういうトラブルが一番成長する

同じところで詰まっている方の参考になれば嬉しいです。