---
title: "サーバーなしでメールを送る！AstroサイトにEmailJSを組み込む方法"
publishDate: 2025-10-27 00:00:00
img: /assets/blog/2025/EmailJS.svg
img_alt: EMAILJS
description: |
  バックエンドの実装が不要なEmailjsを使ってフロントエンドだけで、メール送付ができる問い合わせページを実装
tags:
  - Emailjs
  - 問い合わせページ
  - メール送付
---

本ブログは**Astroのみ**で構築し、**GitHub Pages**上で公開しています。
元々のテンプレートでは「メールリンクによる直接問い合わせ（mailto）」が実装されていましたが、
第三者から直接メールを受け取るのはセキュリティ的にやや不安があるため、
今回は**問い合わせフォーム**を実装し、フォーム送信後に**自分宛へ通知メールを受け取れる**ようにしました。

ただし、Astroは静的サイトジェネレーターのため、**バックエンド機能**がありません。
そのため、フロントエンドだけで完結する方法を検討する必要があります。

### 🔍 採用検討したサービス

検討段階では、以下の3つのサービスを比較しました。
特別な知見があったわけではありませんが、最終的にEmailJSを採用しています。

- [SendGrid](https://sendgrid.kke.co.jp/)
  - 無料枠での利用を検討しましたが、**法人向けアカウント限定**と記載があり断念
- [SmtpJS](https://smtpjs.com/)
  - 検索で上位に出てきましたが、**指定SMTPサービスへの登録が必要**で手順が不明だったため保留
- [EmailJS](https://www.emailjs.com/)
  - 無料で**月200リクエスト**まで利用可能。登録も簡単で、**ドキュメント**が充実していたため採用

### 🤖EmailJSってどういうサービスなのかを知る
**EmailJS**は、バックエンドやサーバー構築を一切せずに**メール送信機能を実装できるサービス**です。
問い合わせフォーム、フィードバック送信、通知など、フロントエンドから直接メールを送ることが可能です。

対応しているメールサービスは以下の通りです：

- トランザクション系：Mailgun / Mailjet / Mailtrap / SendinBlue / SendGrid / Amazon SES / Postmark
- 個人メール系：Gmail / Outlook / Yahoo / Zoho など

料金プランは「Free / Personal / Professional / Business」があり、
無料プランでは**月200リクエスト・テンプレート2つまで**利用できます。

ドキュメント（[EmailJS Docs](https://www.emailjs.com/docs/tutorial/overview/)）も整備されており、
サンプルフォーム付きのチュートリアルも掲載されています。
英語のみの記載ですが、ブラウザ翻訳で十分理解できるレベルです。

### 🔧EmailJSの初期設定

ここでは**Gmailを使った例**で説明します。
他サービスでも大きな違いはありませんが、対応メールは事前に確認しておきましょう。

1. EmailJSのアカウント作成

[Sign Up Free](https://dashboard.emailjs.com/sign-up)から新規登録します。
入力項目は以下の通りです：

- Your Name
- Your Email
- Password
- Bot Protection（ボット対策）

登録後、<mark>support@emailjs.com</mark> から確認メールが届きます。
本文中の 「**Verify email**」 をクリックすると認証完了です。
届かない場合は迷惑メールフォルダも確認しましょう。

2. Gmailアカウントとの連携

    1. ナビゲーションバーから「Email Services」→「Add New Service」をクリック
    2. 「Personal Services」内の「Gmail」を選択
    3. 「Service ID」は自動生成されるため、そのままでOK
    4. 「Connect Account」をクリックし、Google認証でアクセスを許可
    5. 「ユーザー本人に代わってメールを送信」にチェックを入れ、「続行」をクリック

最後に「Send test email to verify configuration」にチェックを入れたまま
「Create Service」を押すと完了です。

完了後、<mark>service_</mark> から始まる**Service ID**をメモしておきましょう。

3. テンプレートの作成

    1. 「Email Templates」→「Create New Template」へ進む
    2. テンプレート名は任意（例：「Contact Us」）
    3. 以下のように設定します：
      - To Email:[Gmailのメールアドレス]
      - From Name:[Gmailのメールアドレス]
      - From Email:「Use Default Email Address」にチェック
      - Subject:<mark>Beach3Blogからの問い合わせ</mark>
      - Content
```
Beach3のサイト経由で問い合わせがきました。
内容を確認して対応してください。

名前：{{name}}
メールアドレス：{{contact_email}}
件名：{{subject}}
本文：
{{message}}
```

作成後、<mark>template_</mark>**から始まるTemplate ID**をメモします。

4. パブリックキーの確認

「Account」→「API Keys」から **Public Key** を確認します。
これも後でAstro側で使うためメモしておきましょう。


### 🚀Astroでの問い合わせページの実装

Astroで作成した問い合わせページは以下の構成です。

    - お名前
    - メールアドレス
    - 件名
    - メッセージ本文

環境変数は *.github/workflows/deploy.yml* と **GitHub Secrets** に登録します。

```yaml
env:
  PUBLIC_G_TAG: ${{secrets.PUBLIC_G_TAG}}
  PUBLIC_NEWRELIC_ACCOUNT_ID: ${{secrets.PUBLIC_NEWRELIC_ACCOUNT_ID}}
  PUBLIC_NEWRELIC_APPLICATION_ID: ${{secrets.PUBLIC_NEWRELIC_APPLICATION_ID}}
  PUBLIC_NEWRELIC_LICENSE_KEY: ${{secrets.PUBLIC_NEWRELIC_LICENSE_KEY}}
  PUBLIC_EMAILJS_SERVICE_ID: ${{secrets.PUBLIC_EMAILJS_SERVICE_ID}}
  PUBLIC_EMAILJS_TEMPLATE_ID: ${{secrets.PUBLIC_EMAILJS_TEMPLATE_ID}}
  PUBLIC_EMAILJS_PUBLIC_KEY: ${{secrets.PUBLIC_EMAILJS_PUBLIC_KEY}}
  BUILD_PATH: "." # default value when not using subfolders
```

*src/pages/contact.astro* にフォームを実装し、
`<SendContactMessage />` コンポーネントでEmailJSを呼び出します。

src/pages/contact.astroを追加

```astro
---
import ContactCTA from '../components/ContactCTA.astro';
import Hero from '../components/Hero.astro';
import BaseLayout from '../layouts/BaseLayout.astro';
import SendContactMessage from '../components/SendContactMessage.astro';
---

<BaseLayout
  title="Contact | Beach3"
  description="Contact Beach3 Engineer"
>
  <div class="stack gap-20">
    <main class="wrapper py-12">
      <div class="stack gap-15">
        <header>
          <Hero title="Contact" align="start">
            <p class="description">
              ブログや個人に関するご質問・ご相談などがございましたら、以下のフォームよりお気軽にお問い合わせください。
            </p>
          </Hero>
        </header>
          <div class="stack gap-4">
            <label for="name">お名前</label>
            <input type="text" id="name" name="name" required />
          </div>
          <div class="stack gap-4">
            <label for="contact_email">メールアドレス</label>
            <input type="email" id="contact_email" name="contact_email" required />
          </div>
          <div class="stack gap-4">
            <label for="subject">件名</label>
            <input id="subject" name="subject" required />
          </div>
          <div class="stack gap-4">
            <label for="message">メッセージ</label>
            <textarea id="message" name="message" rows="6" required></textarea>
          </div>
          <SendContactMessage />
      </div>
    </main>
    <ContactCTA />
  </div>
</BaseLayout>

<style>
  label {
    font-weight: 600;
  }

  input,
  textarea {
    width: 100%;
    padding: 0.75rem;
    border: 1px solid var(--gray-0);
    border-radius: 0.375rem;
    font-size: 1rem;
    font-family: inherit;
    max-width: 100%;
    min-width: 100%;
  }

  input:focus,
  textarea:focus {
    outline: none;
    border-color: var(--color-primary);
    box-shadow: 0 0 0 3px var(--color-primary-light);
  }
</style>
```

src/components/SendContactMessage.astroを追加

```astro
---
import ContactCTA from '../components/ContactCTA.astro';
import Hero from '../components/Hero.astro';
import BaseLayout from '../layouts/BaseLayout.astro';
import SendContactMessage from '../components/SendContactMessage.astro';
---

<BaseLayout
  title="Contact | Beach3"
  description="Contact Beach3 Engineer"
>
  <div class="stack gap-20">
    <main class="wrapper py-12">
      <div class="stack gap-15">
        <header>
          <Hero title="Contact" align="start">
            <p class="description">
              ブログや個人に関するご質問・ご相談などがございましたら、以下のフォームよりお気軽にお問い合わせください。
            </p>
          </Hero>
        </header>
          <div class="stack gap-4">
            <label for="name">お名前</label>
            <input type="text" id="name" name="name" required />
          </div>
          <div class="stack gap-4">
            <label for="contact_email">メールアドレス</label>
            <input type="email" id="contact_email" name="contact_email" required />
          </div>
          <div class="stack gap-4">
            <label for="subject">件名</label>
            <input id="subject" name="subject" required />
          </div>
          <div class="stack gap-4">
            <label for="message">メッセージ</label>
            <textarea id="message" name="message" rows="6" required></textarea>
          </div>
          <SendContactMessage />
      </div>
    </main>
    <ContactCTA />
  </div>
</BaseLayout>

<style>
  label {
    font-weight: 600;
  }

  input,
  textarea {
    width: 100%;
    padding: 0.75rem;
    border: 1px solid var(--gray-0);
    border-radius: 0.375rem;
    font-size: 1rem;
    font-family: inherit;
    max-width: 100%;
    min-width: 100%;
  }

  input:focus,
  textarea:focus {
    outline: none;
    border-color: var(--color-primary);
    box-shadow: 0 0 0 3px var(--color-primary-light);
  }
</style>
```

これらのファイルの追加が完了したら、これまでメールへリンクしていたリンク先を/contactへ変更します。

src/components/ContactCTA.astroの一部変更

```diff
- <CallToAction href="mailto:me@example.com">
+ <CallToAction href="/contact/">
```

これで「Send Me a Message」ボタンから問い合わせフォームに遷移できます。
送信後、メールが届けば設定完了です 🎉

### ⚠️注意点と改善ポイント

EmailJSの **パブリックキー・サービスID・テンプレートID** は
静的サイトの特性上、完全に隠すことはできません。
そのため、大規模システムでは**サーバーサイド処理**を検討する必要があります。

個人サイトや小規模企業（〜100名程度）であれば、
この方法でも十分に実用的だと考えています。

#### 💡セキュリティ対策例

- パブリックキーを定期的に変更する
- Google reCAPTCHAを導入する
- サーバー側APIでProxy送信を検討する

#### 💬 改善アイデア（UX面）

- 送信確認や完了メッセージをモーダル表示に変更
- 入力値のバリデーション強化（メール形式や必須チェック）
- 入力ミス時のエラーメッセージ表示

こうした改善により、フォーム利用時の **ユーザー体験（UX）** が向上します。

### ✌️まとめ
Astro + EmailJSを組み合わせることで、サーバーレスでも安全・簡単に問い合わせフォームを実装できます。

GitHub Pagesなど静的ホスティング環境でも動作するため、個人ブログや中小企業サイトには最適な選択肢です。

今後は改善アイデアの反映をできていけたらいいなと思っています。