---
inclusion: fileMatch
fileMatchPattern: 'aws-cloudformation-templates/**/*.md'
---

# aws-cloudformation-templates ガイドライン

このドキュメントは、aws-cloudformation-templates プロジェクトの構造と README ファイルの標準形式を定義します。

## 1. プロジェクト概要

### リポジトリ概要

本番環境対応の AWS CloudFormation テンプレート集。サービスカテゴリ別に整理されています。

#### 主な特徴
- **本番環境対応**: セキュリティ設定、監視、アラートを含む
- **モジュラー設計**: 単独または組み合わせて使用可能
- **包括的なカバレッジ**: セキュリティ、ネットワーク、監視、分析、メディア、CI/CD など
- **多言語対応**: 英語と日本語のドキュメント
- **SAM 統合**: Lambda ベースのソリューション向け

#### 主なユースケース
- 集中ログと監視による安全な AWS 環境のセットアップ
- AWS アカウント全体の標準化されたセキュリティ設定
- 分析パイプラインとデータ処理ワークフロー
- CI/CD パイプラインを使用した Web アプリケーションのデプロイ
- マルチアカウントガバナンスとコンプライアンス

#### アーキテクチャパターン
- AWS Organizations によるマルチアカウントセキュリティ
- 集中ログ（S3 + OpenSearch）
- CloudFront による静的 Web サイトホスティング
- ECS によるコンテナアプリケーション
- AWS Glue による分析パイプライン

### ディレクトリ構造

#### トップレベル構造
```
aws-cloudformation-templates/
├── {service-category}/          # サービス固有のテンプレート
│   ├── README.md               # カテゴリドキュメント（英語）
│   ├── README_JP.md           # カテゴリドキュメント（日本語）
│   ├── templates/             # CloudFormation テンプレート
│   ├── sam-app/              # SAM アプリケーション（該当する場合）
│   └── readme/               # 追加ドキュメント
├── images/                   # アーキテクチャ図とスクリーンショット
└── shared/                  # 再利用可能なコンポーネントとユーティリティ
```

#### サービスカテゴリ一覧
| カテゴリ | 説明 |
|---------|------|
| aiml/ | AI/ML サービス（Bedrock、Kendra） |
| analytics/ | データ分析（Glue、分析パイプライン） |
| cicd/ | CI/CD パイプライン（CodeBuild、CodePipeline） |
| cloudops/ | 運用（Systems Manager、監視） |
| edge/ | エッジサービス（CloudFront、WAF） |
| identity/ | ID 管理（IAM、Identity Center） |
| media/ | メディアサービス（MediaLive、MediaConnect） |
| monitoring/ | CloudWatch アラームとダッシュボード |
| network/ | ネットワーク（VPC、Transit Gateway、Route53） |
| notification/ | メッセージング（SNS、EventBridge、Chatbot） |
| security/ | セキュリティサービス（GuardDuty、Security Hub、Config） |
| storage/ | ストレージサービス（FSx） |
| web-servers/ | Web アプリケーションホスティング（ECS、Auto Scaling） |

#### サービスフォルダ構造
```
aws-cloudformation-templates/
├── [service-name]/
│   ├── README.md (英語)
│   ├── README_JP.md (日本語)
│   └── templates/
│       ├── template.yaml (メインテンプレート)
│       └── [other-templates].yaml
```

### ファイル命名規則
- **テンプレート**: `{service-name}.yaml` または メインテンプレートは `template.yaml`
- **ドキュメント**: `README.md`（英語）、`README_JP.md`（日本語）
- **SAM アプリ**: `sam-app/` 内に `template.yaml` で整理
- **Lambda コード**: `sam-app/` 内に関数ごとに別ディレクトリ

### テンプレート構造標準
- すべての CloudFormation テンプレートに YAML 形式を使用
- 包括的なパラメータ説明を含める
- 意味のある出力値を提供
- AWS リソース命名規則に従う
- 適切なタグとメタデータを含める

### テンプレート URL の違い
- **デプロイボタン**: `/templates/` パスなしの S3 URL
  - 例: `https://eijikominami.s3-ap-northeast-1.amazonaws.com/aws-cloudformation-templates/edge/cloudfront.yaml`
- **CLI コマンド**: `templates/` フォルダ付きのローカルパス
  - 例: `aws cloudformation deploy --template-file templates/cloudfront.yaml`

※ CI/CD が `[service]/templates/` から S3 の `aws-cloudformation-templates/[service]/` にフラット化してアップロードするため

---

## 2. 全体構成ルール

### 必須セクション順序

1. **Header Section**（必須）
2. **Prerequisites Section**（必須）
3. **TL;DR Section**（必須）
4. **Architecture Section**（推奨）
5. **Deployment Section**（推奨）
6. **Service-Specific Sections**（オプション）

### セクション命名規則

| セクション | 英語タイトル | 日本語タイトル |
|-----------|-------------|---------------|
| Prerequisites | `## Prerequisites` | `## 前提条件` |
| TL;DR | `## TL;DR` | `## TL;DR` |
| Architecture | `## Architecture` | `## アーキテクチャ` |
| Deployment | `## Deployment` | `## デプロイ` |

**禁止**: サービス名をセクションタイトルにしない（例: `## Security`, `## Network`）

---

## 3. 日本語固有のルール

### スペーシングルール
日本語テキストと英語単語・略語の間に必ずスペースを追加する。

| ❌ 悪い例 | ✅ 良い例 |
|----------|----------|
| `CloudFrontディストリビューション` | `CloudFront ディストリビューション` |
| `VPCとサブネット` | `VPC とサブネット` |
| `OAuth 2.0認証情報` | `OAuth 2.0 認証情報` |

### AWS サービス名の表記
英語のまま保持し、適切なスペーシングを適用する。

- ✅ `Amazon S3 バケット`
- ✅ `AWS Lambda 関数`
- ✅ `Amazon CloudWatch アラーム`

### 技術用語の翻訳

| 英語 | 日本語 |
|------|--------|
| deployment | デプロイ |
| template | テンプレート |
| parameter | パラメータ |

---

## 4. 各セクションの詳細

### 4.1 Header Section

#### 英語版
```markdown
English / [**日本語**](README_JP.md)

# AWSCloudFormationTemplates/[service-name]
![Build Status](https://codebuild.ap-northeast-1.amazonaws.com/badges?uuid=...)
![GitHub](https://img.shields.io/github/license/eijikominami/aws-cloudformation-templates)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/eijikominami/aws-cloudformation-templates)

``AWSCloudFormationTemplates/[service-name]`` [brief description].
```

#### 日本語版
```markdown
[**English**](README.md) / 日本語

# AWSCloudFormationTemplates/[service-name]
![Build Status](https://codebuild.ap-northeast-1.amazonaws.com/badges?uuid=...)
![GitHub](https://img.shields.io/github/license/eijikominami/aws-cloudformation-templates)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/eijikominami/aws-cloudformation-templates)

``AWSCloudFormationTemplates/[service-name]`` は、[簡潔な説明]。
```

#### 必須要素
- 言語ナビゲーションリンク
- サービスタイトル: `# AWSCloudFormationTemplates/[service-name]`
- 3つのバッジ（Build Status、License、Release）
- 簡潔なサービス説明（1-2文）

---

### 4.2 Prerequisites Section

#### 必要な条件
以下のいずれかを満たす場合に必須：
- TL;DR セクションにデプロイボタンがある
- Deployment セクションがある
- `templates/` フォルダに YAML ファイルが存在する

#### 不要な場合
- ドキュメントのみの README
- プロジェクトルートの README
- CONTRIBUTING.md や LICENSE などの補助ファイル

#### 英語版
```markdown
## Prerequisites

Before deploying this template, ensure you have:

- [サービス固有の要件 1]
- [サービス固有の要件 2]
```

#### 日本語版
```markdown
## 前提条件

デプロイの前に以下を準備してください。

- [サービス固有の要件 1]
- [サービス固有の要件 2]
```

#### コンテンツルール
- ✅ 含める: サービス固有の要件（証明書、ドメイン名、既存リソース、他テンプレートの事前デプロイ）
- ❌ 除外: 一般的な要件（AWS CLI、基本的な IAM 権限、一般的な AWS 知識）

#### バリエーション
- 監視サービスの場合（英語）: `Before deploying these monitoring templates, ensure you have:`
- 監視サービスの場合（日本語）: `これらの監視テンプレートをデプロイする前に、以下を準備してください。`

---

### 4.3 TL;DR Section

#### パターン一覧

| パターン | 説明 |
|---------|------|
| A | メインテンプレート + 個別サービステンプレート |
| B | メインテンプレートのみ |
| C | 個別サービステンプレートのみ |
| D | 事前ステップが必要（番号付きリスト） |
| E | 複数のデプロイ方法（CloudFormation + SAR） |

#### パターン A: メインテンプレート + 個別サービステンプレート

**英語版**
```markdown
## TL;DR

If you just want to deploy the stack, click the button below.

| US East (Virginia) | Asia Pacific (Tokyo) |
| --- | --- |
| [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) | [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) |

If you want to deploy each service individually, click the button below.

| Services | US East (Virginia) | Asia Pacific (Tokyo) |
| --- | --- | --- |
| [Service Name] | [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) | [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) |
```

**日本語版**
```markdown
## TL;DR

以下のボタンをクリックすることで、CloudFormation をデプロイすることが可能です。

| 米国東部 (バージニア北部) | アジアパシフィック (東京) |
| --- | --- |
| [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) | [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) |

個別のサービスをデプロイする場合は、以下のボタンをクリックしてください。

| サービス | 米国東部 (バージニア北部) | アジアパシフィック (東京) |
| --- | --- | --- |
| [サービス名] | [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) | [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) |
```

#### パターン B: メインテンプレートのみ

**英語版**
```markdown
## TL;DR

If you just want to deploy the stack, click the button below.

| US East (Virginia) | Asia Pacific (Tokyo) |
| --- | --- |
| [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) | [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) |
```

**日本語版**
```markdown
## TL;DR

以下のボタンをクリックすることで、CloudFormation をデプロイすることが可能です。

| 米国東部 (バージニア北部) | アジアパシフィック (東京) |
| --- | --- |
| [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) | [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) |
```

#### パターン C: 個別サービステンプレートのみ

**英語版**
```markdown
## TL;DR

If you want to deploy each service individually, click the button below.

| Services | US East (Virginia) | Asia Pacific (Tokyo) |
| --- | --- | --- |
| [Service Name] | [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) | [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) |
```

**日本語版**
```markdown
## TL;DR

個別のサービスをデプロイする場合は、以下のボタンをクリックしてください。

| サービス | 米国東部 (バージニア北部) | アジアパシフィック (東京) |
| --- | --- | --- |
| [サービス名] | [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) | [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) |
```

#### パターン D: 事前ステップが必要

**英語版**
```markdown
## TL;DR

1. Before running this CloudFormation template, run the [**Security template**](../security/README.md) in this project. (Optional)

[![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL)

2. **Click the button below to deploy this template.**

[![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL)
```

**日本語版**
```markdown
## TL;DR

1. この CloudFormation テンプレートを実行する前に、このプロジェクトの [**Security テンプレート**](../security/README_JP.md) を実行してください。（オプション）

[![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL)

2. **以下のボタンをクリックして、このテンプレートをデプロイしてください。**

[![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL)
```

#### パターン E: 複数のデプロイ方法

**英語版**
```markdown
## TL;DR

If you just want to deploy the stack, click one of the two buttons below.

[application-name - AWS Serverless Application Repository](SAR_URL)

| US East (Virginia) | Asia Pacific (Tokyo) |
| --- | --- |
| [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) | [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) |
```

**日本語版**
```markdown
## TL;DR

以下のいずれかのボタンをクリックすることで、CloudFormation をデプロイすることが可能です。

[application-name - AWS Serverless Application Repository](SAR_URL)

| 米国東部 (バージニア北部) | アジアパシフィック (東京) |
| --- | --- |
| [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) | [![cloudformation-launch-stack](../images/cloudformation-launch-stack.png)](URL) |
```

#### TL;DR フレーズルール

| パターン | 英語フレーズ | 日本語フレーズ |
|---------|-------------|---------------|
| A, B | `If you just want to deploy the stack, click the button below.` | `以下のボタンをクリックすることで、CloudFormation をデプロイすることが可能です。` |
| A, C | `If you want to deploy each service individually, click the button below.` | `個別のサービスをデプロイする場合は、以下のボタンをクリックしてください。` |
| E | `If you just want to deploy the stack, click one of the two buttons below.` | `以下のいずれかのボタンをクリックすることで、CloudFormation をデプロイすることが可能です。` |
| D | 番号付きリストで手順を示す | 番号付きリストで手順を示す |

**禁止フレーズ（パターン A, B, C）：**
- ❌ `follow these steps`
- ❌ `click the buttons below`（複数形）

---

### 4.4 Architecture Section（推奨）

アーキテクチャ図が存在する場合に含める。

**英語版**
```markdown
## Architecture

The following sections describe the individual components of the architecture.

![](../images/architecture-[service-name].png)

### [Component Name]

[Component description]
```

**日本語版**
```markdown
## アーキテクチャ

以下のセクションでは、アーキテクチャの各コンポーネントについて説明します。

![](../images/architecture-[service-name].png)

### [コンポーネント名]

[コンポーネントの説明]
```

---

### 4.5 Deployment Section（推奨）

複雑なデプロイ手順が存在する場合に含める。

**英語版**
```markdown
## Deployment

Execute the command to deploy with required parameters.

\`\`\`bash
aws cloudformation deploy --template-file templates/template.yaml --stack-name StackName --parameter-overrides Param1=Value1
\`\`\`

You can provide optional parameters as follows.

| Name | Type | Default | Required | Details |
| --- | --- | --- | --- | --- |
| ParamName | String | | ○ | Parameter description |
```

**日本語版**
```markdown
## デプロイ

必要なパラメータを指定してコマンドを実行します。

\`\`\`bash
aws cloudformation deploy --template-file templates/template.yaml --stack-name StackName --parameter-overrides Param1=Value1
\`\`\`

オプションのパラメータは以下の通りです。

| 名前 | タイプ | デフォルト値 | 必須 | 詳細 |
| --- | --- | --- | --- | --- |
| ParamName | String | | ○ | パラメータの説明 |
```

---

## 5. 品質保証チェックリスト

### 形式チェック
- [ ] 言語ナビゲーションリンクが正しい
- [ ] サービスタイトルが `# AWSCloudFormationTemplates/[service-name]` 形式
- [ ] 3つのバッジが存在（Build Status、License、Release）
- [ ] セクション順序が正しい（Prerequisites → TL;DR → Architecture → Deployment）
- [ ] セクションタイトルが正確（サービス名をタイトルにしていない）

### コンテンツチェック
- [ ] Prerequisites がサービス固有の要件のみを含む
- [ ] TL;DR が適切なパターンのフレーズを使用
- [ ] デプロイボタンの URL が `/templates/` パスなし
- [ ] CLI コマンドが `templates/` フォルダ付きパス

### 日本語チェック
- [ ] 日本語と英語の間にスペースがある
- [ ] AWS サービス名が英語のまま
- [ ] 技術用語の翻訳が一貫している

### 技術的正確性
- [ ] YAML ファイルとの整合性を確認
- [ ] 全ての情報を AWS ドキュメントで検証
- [ ] 推測や仮定なし

---

## 6. 実装ガイドライン

### 作業アプローチ
- 1つのサービスを完全に完了してから次へ
- フォルダ名のアルファベット順で作業
- README.md と README_JP.md の両方を更新

### 情報の正確性
- サービス固有の要件は AWS MCP または公式ドキュメントで検証必須
- 推測禁止
- 含める前に全ての技術的詳細を確認

### 特殊ケース
- **テンプレートがないサービス**: 基本構造で README を作成し、テンプレートの不在を記載
- **複数のサブサービス**: TL;DR で個別デプロイテーブルを使用
- **複雑なアーキテクチャ**: 詳細な Architecture セクションを含める
- **シンプルなデプロイ**: TL;DR で十分な場合は Deployment セクションを省略可能

---

## 7. 一般的な問題と解決策

| 問題 | 解決策 |
|------|--------|
| デプロイボタンが 404 エラー | S3 URL が `/templates/` パスを除外しているか確認 |
| 日本語が読みにくい | 日本語と英語の間にスペースを追加 |
| Prerequisites が一般的すぎる | サービス固有の要件のみに絞る |
