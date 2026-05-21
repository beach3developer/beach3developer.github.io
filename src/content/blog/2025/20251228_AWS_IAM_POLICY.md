---
title: "【実務向け】AWS運用でよく使うIAMポリシーの種類と設計パターンまとめ"
publishDate: 2025-12-28 00:00:00
img: /assets/blog/2025/AWS_Cloud.svg
img_alt: Template
description: |
  AWS運用で避けて通れないIAMポリシー。本記事では、AWS管理ポリシーからカスタマー管理ポリシー、SCP、Permission Boundaryまで、実務で本当によく使うIAMポリシーの種類と設計パターンを海外のベストプラクティスを踏まえて解説します。
tags:
  - AWS
  - IAM
  - セキュリティ
  - クラウド運用
---
## ●AWSを運用するために絶対必要なIAMに関する運用

AWS を運用していると、ほぼ確実に直面するのが **IAM（Identity and Access Management）** の設計です。  
「IAMは難しい」と言われがちですが、実際には**種類が多く整理されていないこと**が混乱の原因であるケースがほとんどです。

海外の多くの事故事例やセキュリティインシデントを見ても、

- 権限が広すぎる
- 誰が何をできるのか分からない
- root や管理者権限が常用されている

といった **IAM設計の問題** が原因になっていることが分かります。

本記事では、AWS運用で**実際によく使われるIAMポリシーの種類**を整理し、  
それぞれの役割・使いどころ・注意点を実務目線で解説します。

---

## ●IAMポリシーの全体像

IAMポリシーは一言で言っても、用途ごとに明確な役割分担があります。

|種類|主な用途|
|:----|:----|
|AWS管理ポリシー|すぐに使える既製品|
|カスタマー管理ポリシー|実運用の主役|
|インラインポリシー|例外・一時用途|
|リソースベースポリシー|S3/KMS/SQSなど|
|SCP(Service Control Policy)|組織統制|
|Permission Boundary|権限の上限設定|
|Session Policy|一時セッション制御|

AWSのポリシーというと上記の種類に分類できます。
まず初めに触れていくのは「AWS管理ポリシー」「カスタマー管理ポリシー」「インラインポリシー」だと思います。
個人的な使い分けは慣れるまでもしくは、開発で少しポリシーに余裕を持たせたい場合は「AWS管理ポリシー」で実運用時は「カスタマー管理ポリシー」で最低限のポリシーのみを設定します。一時的または緊急的に利用する場合はインラインポリシーで利用しています。

---

## ●IAMポリシーの記載ルール（基本構造）

IAMポリシーは **JSON形式** で記述され、すべて同じ基本構造を持っています。  
まずは「どの項目が何を意味しているか」を理解することが重要です。

### IAMポリシーの基本形

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "service:Operation",
      "Resource": "*"
    }
  ]
}
```

#### 各要素の意味
##### Version

```json
"Version": "2012-10-17"
```

- IAMポリシーの **構文バージョン**
- 現在は **必ず `2012-10-17` を指定**
- 日付だが意味的な更新日は気にしなくてよい

##### Statement

```json
"Statement": [ { ... } ]
```

- ポリシーの本体
- 配列なので **複数ルールを定義可能**
- 1つのポリシーに複数の許可・拒否条件を書ける

##### Effect（許可 or 拒否）

```json
"Effect": "Allow"
```

|値|意味|
|:----|:----|
|Allow|操作を許可|
|Deny|操作を明示的に拒否|

**⚠️ DenyはAllowより常に優先されます**
（これはIAM設計で最も重要なルールの1つです）

##### Action（何をしてよいか）

```json
"Action": "s3:GetObject"
```

- 実行可能なAPI操作
- サービス名:操作名 の形式

##### ワイルドカード例

```json
"Action": "ec2:Describe*"
```

- `DescribeInstances`
- `DescribeVolumes`
- などすべて含まれる

海外のベストプラクティスでは
**👉 Read系（Describe / List）と Write系を分ける** 設計が推奨されています。

---

## ●AWS管理ポリシー（AWS Managed Policies）

AWS が公式に提供しているポリシーです。

代表例：

- `AdministratorAccess`
- `PowerUserAccess`
- `ReadOnlyAccess`

### メリット
- すぐに使える
- 学習用途・PoCに向いている

### デメリット
- 権限が広すぎる
- 本番運用では過剰権限になりやすい

```json
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}
```

「本番では極力使わない」 と明言されることが多く、あくまで一時的・検証用 と考えるのが安全です。

---

## ●カスタマー管理ポリシー（最重要）
実運用で最も重要なのが **カスタマー管理ポリシー** です。
海外では “Custom policies are the only sane choice for production” と言われるほどです。


- EC2運用担当者向けポリシー例

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*",
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

- EC2運用担当者向けポリシー例（制限を設ける場合）

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DisplayEC2",
            "Effect": "Allow",
            "Action": [
                "ec2:Describe*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "ControlEC2",
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Resource": "*",
            "Condition": {
                "StringNotLike": {
                    "ec2:ResourceTag/Name": "【触らせたくないEC2名】"
                }
            }
        }
    ]
}
```

- S3バケット限定アクセス例

```json
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": [
    "arn:aws:s3:::mybucket",
    "arn:aws:s3:::mybucket/*"
  ]
}
```

- MFAポリシーを強制する

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowIfNoMfa",
            "Effect": "Allow",
            "Action": [
                "iam:CreateVirtualMFADevice",
                "iam:DeactivateMFADevice",
                "iam:DeleteVirtualMFADevice",
                "iam:EnableMFADevice",
                "iam:GetMFADevice",
                "iam:ListMFADevices",
                "iam:ListUsers",
                "iam:ListVirtualMFADevices",
                "iam:ResyncMFADevice"
            ],
            "Resource": [
                "arn:aws:iam::*:user/${aws:username}",
                "arn:aws:iam::*:mfa/*"
            ]
        },
        {
            "Sid": "DenyIfNoMfa",
            "Effect": "Deny",
            "NotAction": [
                "iam:CreateVirtualMFADevice",
                "iam:DeactivateMFADevice",
                "iam:DeleteVirtualMFADevice",
                "iam:EnableMFADevice",
                "iam:GetMFADevice",
                "iam:ListMFADevices",
                "iam:ListUsers",
                "iam:ListVirtualMFADevices",
                "iam:ResyncMFADevice"
            ],
            "Resource": "*",
            "Condition": {
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresent": "false"
                }
            }
        }
    ]
}
```


### ベストプラクティス
- 職務（Role）単位で設計する
- 最小権限（Least Privilege）を守る
- サービスごとにポリシーを分割する

---

## ●インラインポリシー
IAMユーザーやロールに直接紐づくポリシーです。

- 使いどころ
  - 一時的な例外対応
  - 緊急対応
- 注意点
  - 再利用できない
  - 管理が属人化しやすい
  - 削除漏れが発生しやすい

海外では「Last Resort（最後の手段）」として扱われます。


---

## ●リソースベースポリシー

S3、KMS、SQS、SNS などで利用されるポリシーです。
IAM主体ではなく **リソース側でアクセス制御** を行います。

- S3 バケットポリシー（IP制限例）

```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": [
    "arn:aws:s3:::mybucket",
    "arn:aws:s3:::mybucket/*"
  ],
  "Condition": {
    "NotIpAddress": {
      "aws:SourceIp": ["1.2.3.4/32"]
    }
  }
}
```

---

## ●SCP（Service Control Policy）
AWS Organizations で利用する **強力な統制ポリシー** です。

- 特徴
  - 権限を「付与」するものではない
  - 絶対に禁止するルールを定義する
- よくある例
  - 東京リージョン以外禁止
  - CloudTrail削除禁止
  - IAM削除禁止

```json
{
  "Effect": "Deny",
  "Action": "ec2:*",
  "Resource": "*",
  "Condition": {
    "StringNotEquals": {
      "aws:RequestedRegion": "ap-northeast-1"
    }
  }
}
```

---

## ●Permission Boundary
IAMロールやユーザーが **持てる最大権限の上限** を定義します。

- 利用シーン
  - 開発者にIAM作成権限を委譲する
  - マルチチーム運用

Delegated Admin（権限委譲）設計では非常によく使われます。

---

## ●Session Policy（STS利用時）
AssumeRole 時に **一時的に権限を絞る** ポリシーです。

- 利用例
  - CI/CD
  - 外部IdP連携
  - 一時的な管理作業

```bash
aws sts assume-role --policy file://session-policy.json
```

---

## ●IAM設計でよくあるアンチパターン
海外記事で特に人気のある失敗例です。

- Action: "*" を常用
- IAMユーザーを量産
- rootユーザーを日常利用
- ポリシーを1枚に集約
- CloudTrail未設定

**IAM設計＝セキュリティ設計** と考えることが重要です。
またGoogle CloudやAzureではMFAを必須にしており、AWSでの利用でもMFAを強制するような運用をおすすめします。

---

## ●まとめ

- 本番運用では カスタマー管理ポリシーが基本
- SCPで「絶対に守るルール」を定義する
- 権限は「付与」より「制限」を意識
- IAM設計は後回しにしない

IAMを正しく設計することは、AWS運用全体の安定性と安全性を大きく左右します。

