---
inclusion: always
---

# AWS アーキテクチャガイドライン

## MCP による情報収集

AWS 実装時は MCP で AWS ドキュメントを確認する。事前学習の知識に頼らない。

実装前に確認:
1. `mcp_aws_docs_search_documentation` で最新ドキュメントを検索
2. 複数の実現方法がある場合はメリット・デメリットを比較して提示
3. CloudFormation/SAM テンプレート作成時は `aws-cloudformation-guidelines.md` を参照

IAM 権限調査の検索パターン:
- 基本: `[サービス名] IAM permissions`
- 機能固有: `[サービス名] [機能名] IAM permissions`
- 外部連携: `[サービス名] [外部サービス名] connection permissions`

## リソース作成の原則

| ルール | 説明 |
|--------|------|
| 同時作成 | Lambda 作成時に IAM ロール・CloudWatch Logs も同時に作成 |
| 同時削除 | 機能削除時に関連サポートリソースも削除 |
| 禁止 | 「将来使うかも」でリソースを先行作成しない |

## AWS CLI 実行

ページネーション対策:
```bash
aws [command] --no-paginate --output json
aws [command] --no-paginate --max-items 100 --output json  # 大量データ時
```

## Lambda 関数

ディレクトリ構造:
```
src/[function-name]/
├── app.py
├── requirements.txt
└── tests/
```

実装ルール:
- 環境変数で設定値管理
- structured logging（JSON 形式）でログ出力
- タイムアウト設定は処理内容に応じて適切に設定

## Lambda Layer

サードパーティ Layer は使用禁止（アクセス権限エラーの可能性）。自アカウントの Layer を使用する。

```yaml
# ❌ 禁止: サードパーティ Layer
Layers:
  - arn:aws:lambda:ap-northeast-1:123456789012:layer:some-layer:1

# ✅ 推奨: 自アカウントの Layer
Layers:
  - !Sub 'arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:layer:my-layer:1'
```

自前 Layer 作成手順（FFmpeg 等の外部バイナリ用）:
1. 静的ビルドのバイナリをダウンロード
2. `bin/` ディレクトリに配置
3. ZIP 化して S3 にアップロード
4. `aws lambda publish-layer-version` で発行

```bash
# Layer 構造
layer/
└── bin/
    ├── ffmpeg
    └── ffprobe

# ZIP 作成・発行
cd layer && zip -r9 ../layer.zip bin/
aws lambda publish-layer-version \
  --layer-name my-layer \
  --content S3Bucket=my-bucket,S3Key=layers/layer.zip \
  --compatible-runtimes <runtime> \
  --compatible-architectures x86_64
```

Layer vs Docker 選択基準:

| 方法 | 選択条件 |
|------|----------|
| Layer | 250MB 以下、まずこちらを試す |
| Docker | 250MB 超過時のみ検討 |

## CloudWatch Alarms

監視対象の決定:
1. `.kiro/specs/[project-name]/requirements.md` の監視要件を確認
2. MCP で AWS ベストプラクティスを調査
3. 要件に明記された項目のみ実装

追加不要: 要件に記載がない、「念のため」「将来使うかも」という理由のみ

## データベース選択

| サービス | ユースケース |
|----------|-------------|
| DynamoDB | NoSQL、高スケーラビリティ |
| RDS | RDBMS、複雑なクエリ |
| Aurora Serverless | 可変ワークロード |

## 監視・ログ・コスト

- CloudWatch Logs: 構造化ログ、適切な保持期間設定
- X-Ray: 分散トレーシング有効化
- コスト: タグベースのコスト配分、Budgets での監視
