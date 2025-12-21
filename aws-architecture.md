---
inclusion: always
---

# AWS アーキテクチャガイドライン

## 情報収集の原則

AWS 実装時は MCP で AWS ドキュメントを確認する。事前学習の知識に頼らない。

### 調査手順

1. 実装方法を決める前に MCP で AWS ドキュメントを確認
2. 複数の実現方法がある場合はメリット・デメリットを比較して提示
3. CloudFormation/SAM テンプレート作成時は `aws-cloudformation-guidelines.md` を参照

### IAM ロール・ポリシー作成時の調査

確認の流れ：
1. サービスの基本 IAM 権限（例：「AWS Lambda execution role」）
2. 使用する具体的な機能の権限（例：「Lambda VPC permissions」）
3. 依存サービスの権限（Secrets Manager、KMS、CloudWatch Logs 等）

ドキュメント検索パターン：
- 基本：`[サービス名] IAM permissions`
- 機能固有：`[サービス名] [機能名] IAM permissions`
- 外部連携：`[サービス名] [外部サービス名] connection permissions`

## リソース作成の原則

リソースは使用する対象リソースと同時に作成する。「将来使うかもしれない」という理由で作成しない。

正しい順序：
- Lambda 関数を作る → IAM ロールと CloudWatch Logs も同時に作る
- 機能を削除する → 関連するサポートリソースも削除する

避けるべきパターン：
- 「いつか使うかも」で IAM ロールだけ先に作る
- 使わなくなった IAM ロールを放置する

## AWS CLI 実行時の注意点

ページネーション対策として `--no-paginate --output json` を使用する。

```bash
aws [command] --no-paginate --output json
```

大量データの場合：
```bash
aws [command] --no-paginate --max-items 100 --output json
```

## サービス別ガイドライン

### Lambda 関数

ディレクトリ構造：
```
src/
├── function-name/
│   ├── app.py
│   ├── requirements.txt
│   └── tests/
```

実装ルール：
- 環境変数で設定値管理
- structured logging でログ出力
- 適切なタイムアウト設定

### データベース選択基準

- DynamoDB: NoSQL、高スケーラビリティ
- RDS: RDBMS、複雑なクエリ
- Aurora Serverless: 可変ワークロード

### CloudWatch Alarms

監視対象の決定プロセス：
1. `.kiro/specs/[project-name]/requirements.md` の監視要件を確認
2. MCP で AWS ベストプラクティスを調査
3. 要件に明記された項目とベストプラクティスの項目を実装

追加すべき場合：
- 要件・設計書に明記されている
- AWS ベストプラクティスで推奨
- システムの健全性に直接影響する

追加不要な場合：
- 要件・設計書に記載がなく優先度が低い
- 「念のため」「将来使うかも」という理由のみ

### 監視・ログ

- CloudWatch Logs: 構造化ログ、適切な保持期間
- X-Ray: 分散トレーシング、パフォーマンス分析

### コスト管理

- 使用量ベース課金の活用
- タグベースのコスト配分
- Cost Explorer と Budgets での監視
