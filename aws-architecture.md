# AWS アーキテクチャガイドライン

## AWS CLIコマンド実行ガイドライン

### ページネーション対策（必須）
**AWS CLIコマンド実行時は、出力が途中で止まることを防ぐため、以下の方法を使用する**

#### 推奨方法
```bash
# --no-paginate オプション使用（最も推奨）
aws [command] --no-paginate --output json

# 例：SNS Topic属性確認
aws sns get-topic-attributes --topic-arn [ARN] --no-paginate --output json

# 例：Secrets Manager確認
aws secretsmanager describe-secret --secret-id [SECRET_ID] --no-paginate --output json

# 例：CloudFormationスタック確認
aws cloudformation describe-stacks --stack-name [STACK_NAME] --no-paginate --output json

# 例：Glue Triggers確認
aws glue get-triggers --no-paginate --output json

# 例：CloudFormationイベント確認
aws cloudformation describe-stack-events --stack-name [STACK_NAME] --no-paginate --output json
```

#### その他の方法
```bash
# 大量データの場合：制限付きで取得
aws cloudformation describe-stack-events --stack-name [STACK_NAME] --no-paginate --max-items 100 --output json

# 環境変数でページャー無効化（スクリプト内で複数コマンド実行時）
export AWS_PAGER=""
aws [command] --output json

# 設定ファイルでの永続化（~/.aws/config）
[default]
cli_pager=
output=json
```

#### 適用場面
- **検証・テスト時**: 全出力を確認する必要がある場合
- **デバッグ時**: エラー詳細や設定内容を完全に把握したい場合
- **レポート作成時**: 正確な情報を取得する必要がある場合

## MCP 統合要件（必須）

### AWS ドキュメント確認の原則

**AWS の仕様のことを聞かれたり、AWS の実装方法を決めるときは、MCP で AWS ドキュメントを確認する**

### CloudFormation/SAM テンプレート作成時の MCP 使用（必須）

**CloudFormation/SAM テンプレートを作成・修正する際は、事前学習の知識に頼らず、必ず MCP で AWS ドキュメントを確認する**

#### 必須確認項目
1. **リソーススキーマ情報**: `get_resource_schema_information` でリソースの正確なプロパティを確認
2. **最新の設定オプション**: 各 AWS サービスの最新機能と設定項目を確認
3. **プロパティの制約**: 必須プロパティ、値の範囲、形式制限を確認
4. **ベストプラクティス**: セキュリティ、パフォーマンス、コスト最適化の観点

#### 対象リソース
- **全ての AWS リソース**: S3、Lambda、IAM、SNS、Secrets Manager、CloudWatch 等
- **SAM 固有リソース**: AWS::Serverless::Function、AWS::Serverless::Api 等
- **複雑な設定**: VPC、セキュリティグループ、IAM ポリシー等

#### 実装手順
1. **リソース仕様確認**: MCP でリソーススキーマを確認
2. **プロパティ選択**: 要件に適したプロパティを選択
3. **設定値決定**: 制約条件を満たす設定値を決定
4. **実装**: AWS CloudFormation Guidelines 準拠でテンプレート作成
5. **検証**: CFN Lint + SAM validate で検証

#### MCP 活用例
```bash
# リソース作成前の仕様確認
get_resource_schema_information(resource_type="AWS::SecretsManager::Secret")

# 既存リソースの設定確認
get_resource(resource_type="AWS::S3::Bucket", identifier="bucket-name")

# リソース一覧確認
list_resources(resource_type="AWS::SNS::Topic")
```

### 実装方法の選択

**複数の実現方法があればそのメリットとデメリットを提示して確認を求める**

#### 手順
1. AWS ドキュメントで各実装方法を調査
2. それぞれのメリット・デメリットを整理
3. コスト、パフォーマンス、運用性の観点で比較
4. ユーザーに選択肢を提示し、確認を求める

#### 比較観点
- **コスト**: 初期費用、運用費用
- **パフォーマンス**: 処理速度、スループット
- **運用性**: 管理の複雑さ、監視の容易さ
- **拡張性**: 将来の要件変更への対応
- **セキュリティ**: セキュリティ機能、コンプライアンス対応

## IAMロール作成の原則（必須）

**IAMロール・ポリシー作成時は、必ずMCPでAWSドキュメントを確認してから作成する**

### 段階的ドキュメント確認プロセス（必須）

#### 1. 基本サービス権限の確認
- **一般的なサービス権限**: 対象サービスの基本IAMドキュメントを確認
- **マネージドポリシー**: AWS管理ポリシーの内容と制限を確認

#### 2. 具体的な機能・コネクター権限の確認（重要）
- **機能固有の権限**: 使用する具体的な機能のドキュメントを確認
- **コネクター固有の権限**: 外部サービス連携時の専用ドキュメントを確認
- **例**: Glue + Google Analytics 4 → "Configuring Google Analytics 4 connections"

#### 3. 依存サービス権限の確認
- **暗号化サービス**: KMS、Secrets Manager等の権限
- **ネットワーク**: VPC、EC2ネットワークインターフェース権限
- **ログ・監視**: CloudWatch、X-Ray権限

#### 4. 検証とテスト
- **権限不足エラー**: エラー発生時は再度ドキュメント確認
- **段階的権限追加**: 不足権限を特定して追加

### 信頼性確保のための必須ルール

#### ドキュメント検索の徹底
1. **汎用検索**: "[サービス名] IAM permissions"
2. **機能固有検索**: "[サービス名] [機能名] IAM permissions"
3. **コネクター検索**: "[サービス名] [外部サービス名] connection permissions"
4. **エラー対応検索**: "[エラーメッセージ] IAM permissions"

#### 確認すべきドキュメント例
- **AWS Glue基本**: "Create an IAM role for AWS Glue"
- **Glue + Google Analytics**: "Configuring Google Analytics 4 connections"
- **Lambda**: "AWS Lambda execution role"
- **Step Functions**: "IAM policies for Step Functions"

### 失敗パターンの回避

#### ❌ 避けるべき行動
- 事前学習データのみに依存
- 一般的な権限のみで済ませる
- エラーが出てから調査する
- 推測で権限を設定する

#### ✅ 正しい行動
- 必ず複数のドキュメントを確認
- 機能固有のドキュメントを優先
- 実装前に徹底的に調査
- エラー時は根本原因を特定

## リソース作成の原則（必須）

**IAMロール、CloudWatch Logs、その他のサポートリソースは、対象リソースが作られてはじめて作成する**

### リソース作成ルール
1. **必要時作成**: 対象リソース（Lambda、Glue、Step Functions等）を作成する時に、関連するIAMロールやログリソースも同時に作成
2. **未使用リソース禁止**: 将来使用予定であっても、現在使用されていないリソースは作成しない
3. **段階的追加**: 新機能追加時に必要なサポートリソースを追加
4. **クリーンアップ**: 機能削除時は関連するサポートリソースも削除

### 対象リソース例
- **IAMロール**: Lambda、Glue、Step Functions等の実行ロール
- **CloudWatch Logs**: Lambda関数、Glue Job等のログ出力先
- **VPCエンドポイント**: VPC内リソースからのサービスアクセス用
- **セキュリティグループ**: EC2、RDS等のネットワークアクセス制御

### 避けるべきパターン
- 「将来使用予定」でのリソース事前作成
- 使用されていないIAMロールの放置
- 参照されていないCloudWatch Logsグループの作成
- 未使用のVPCエンドポイントの作成

### 正しいパターン
- Lambda関数作成時にIAMロールとCloudWatch Logsを同時作成
- Glue Job作成時に実行ロールとログ設定を同時作成
- 機能削除時に関連リソースも同時削除

## ランタイム選択の原則

**SAM/Lambdaテンプレート作成時は、MCPでAWSドキュメントを確認して最新のサポートされているランタイムを選択する**

### ランタイム選択ルール
1. **最新優先**: 技術的な障壁がない限り、最新のサポートされているランタイムを使用
2. **非推奨期限確認**: 非推奨まで12ヶ月以上あるランタイムを選択
3. **MCP確認必須**: テンプレート作成前にMCPでAWS Lambda runtimesドキュメントを確認
4. **長期サポート重視**: 同等の機能であれば、より長期サポートが予定されているランタイムを選択

### 適用場面

1. **AWS サービスの仕様確認時**
   - サービスの機能や制限事項
   - APIの使用方法
   - 設定パラメータの詳細

2. **実装方法の選択時**
   - 複数の実現方法がある場合
   - ベストプラクティスの確認
   - パフォーマンスや制限事項の比較

3. **新しいAWSサービスや機能の調査時**
   - 最新の機能追加
   - 既存サービスとの統合方法
   - 料金体系の確認

4. **CloudFormation/SAMテンプレート作成時（必須）**
   - AWSリソースのプロパティ仕様確認
   - 最新のリソース設定オプション確認
   - リソース間の依存関係確認
   - ベストプラクティスの確認



## アーキテクチャ原則
- **サーバーレスファースト**: Lambda、Step Functions、EventBridge等を優先
- **マネージドサービス活用**: RDS、DynamoDB、S3等のフルマネージドサービスを選択
- **コスト最適化**: 使用量ベースの課金モデルを活用
- **セキュリティ**: IAMロール・ポリシーによる最小権限の原則



## Lambda関数

### 構造
```
src/
├── function-name/
│   ├── app.py          # メイン処理
│   ├── requirements.txt # 依存関係
│   └── tests/          # テストコード
```

### ベストプラクティス
- 環境変数で設定値管理
- ログ出力はstructured logging
- エラーハンドリングの徹底
- タイムアウト設定の適切な設定

## データベース

### 選択基準
- **DynamoDB**: NoSQL、高スケーラビリティが必要
- **RDS**: RDBMS、複雑なクエリが必要
- **Aurora Serverless**: 可変ワークロード

### 設計原則
- パーティションキーの適切な設計
- インデックス戦略の検討
- バックアップ・復旧戦略

## 監視・ログ

### CloudWatch
- カスタムメトリクスの活用
- アラーム設定による異常検知
- ダッシュボードでの可視化

### X-Ray
- 分散トレーシングの有効化
- パフォーマンス分析
- エラー原因の特定

## コスト管理

### 基本方針
- **使用量ベース課金**の活用
- **予約インスタンス**の検討（長期利用の場合）
- **スポットインスタンス**の活用（バッチ処理等）

### 監視
- Cost Explorerでのコスト分析
- Budgetsでの予算管理
- タグベースのコスト配分