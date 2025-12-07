# AWS アーキテクチャガイドライン

## アーキテクチャ・設計の原則（必須）

### 基本方針
- **サーバーレスファースト**: Lambda、Step Functions、EventBridge等を優先
- **マネージドサービス活用**: RDS、DynamoDB、S3等のフルマネージドサービスを選択
- **コスト最適化**: 使用量ベースの課金モデルを活用
- **セキュリティ**: IAMロール・ポリシーによる最小権限の原則

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

### リソース作成の原則

**「将来使うかもしれない」という理由で、今使わないリソースを作成しない**

#### 基本ルール

リソースは、それを使う対象リソースと同時に作成します。

**正しい順序：**
1. Lambda関数を作る → その時にIAMロールとCloudWatch Logsも作る
2. Glue Jobを作る → その時に実行ロールとログ設定も作る
3. 機能を削除する → 関連するサポートリソースも削除する

**避けるべきパターン：**
- ❌ 「いつか使うかも」でIAMロールだけ先に作る
- ❌ Lambda関数がないのにLambda用のCloudWatch Logsを作る
- ❌ 使わなくなったIAMロールを放置する

**対象となるサポートリソース：**
- IAMロール（Lambda、Glue、Step Functions等の実行ロール）
- CloudWatch Logs（ログ出力先）
- VPCエンドポイント（VPC内からのサービスアクセス用）
- セキュリティグループ（ネットワークアクセス制御）

**理由：**
1. **無駄なリソースを防ぐ**: 計画が変わって結局使わないことがある
2. **コスト削減**: 使わないリソースでも課金される場合がある
3. **管理の簡素化**: 何が使われているか明確になる

## 実装時の必須手順

### MCP によるドキュメント確認（必須）

**AWS の仕様のことを聞かれたり、AWS の実装方法を決めるときは、MCP で AWS ドキュメントを確認する**

#### CloudFormation/SAM テンプレート作成時

**CloudFormation/SAM テンプレートを作成・修正する際は、事前学習の知識に頼らず、必ず MCP で AWS ドキュメントを確認する**

**必須確認項目：**
1. **リソーススキーマ情報**: `get_resource_schema_information` でリソースの正確なプロパティを確認
2. **最新の設定オプション**: 各 AWS サービスの最新機能と設定項目を確認
3. **プロパティの制約**: 必須プロパティ、値の範囲、形式制限を確認
4. **ベストプラクティス**: セキュリティ、パフォーマンス、コスト最適化の観点

**対象リソース：**
CloudFormation/SAM テンプレートで定義する全ての AWS と SAM リソース

例：AWS::S3::Bucket、AWS::Lambda::Function、AWS::IAM::Role、AWS::SNS::Topic、AWS::SecretsManager::Secret、AWS::Serverless::Function、AWS::Serverless::Api 等

**実装手順：**
1. **リソース仕様確認**: MCP でリソーススキーマを確認
2. **プロパティ選択**: 要件に適したプロパティを選択
3. **設定値決定**: 制約条件を満たす設定値を決定
4. **実装**: AWS CloudFormation Guidelines 準拠でテンプレート作成
5. **検証**: CFN Lint + SAM validate で検証

**MCP 活用例：**
```bash
# リソース作成前の仕様確認
get_resource_schema_information(resource_type="AWS::SecretsManager::Secret")

# 既存リソースの設定確認
get_resource(resource_type="AWS::S3::Bucket", identifier="bucket-name")

# リソース一覧確認
list_resources(resource_type="AWS::SNS::Topic")
```

#### IAMロール作成時

**IAMロール・ポリシー作成時は、必ずMCPでAWSドキュメントを確認してから作成する**

IAM権限は、一般的なサービスドキュメントだけでなく、**使用する具体的な機能のドキュメント**まで確認する必要があります。

**確認の流れ：**
1. **サービスの基本IAM権限**を確認
   - 例：「AWS Glue IAM permissions」「AWS Lambda execution role」
2. **使用する具体的な機能の権限**を確認（重要）
   - 例：Glue + Google Analytics 4 → 「Configuring Google Analytics 4 connections」
   - 例：Lambda + VPC → 「Lambda VPC permissions」
3. **依存サービスの権限**を確認
   - 暗号化：Secrets Manager、KMS
   - ネットワーク：VPC、EC2ネットワークインターフェース
   - ログ・監視：CloudWatch Logs、X-Ray

**ドキュメント検索のパターン：**
- 基本：「[サービス名] IAM permissions」
- 機能固有：「[サービス名] [機能名] IAM permissions」
- 外部連携：「[サービス名] [外部サービス名] connection permissions」

**重要な注意点：**
- ❌ 一般的な権限ドキュメントだけで済ませる
- ❌ 事前学習データのみに依存する
- ❌ エラーが出てから調査する
- ✅ 使用する具体的な機能のドキュメントまで確認する
- ✅ 実装前に徹底的に調査する
- ✅ 権限不足エラー時は根本原因を特定する

#### ランタイム選択時

**SAM/Lambdaテンプレート作成時は、MCPでAWSドキュメントを確認して最新のサポートされているランタイムを選択する**

**ランタイム選択ルール：**
1. **最新優先**: 技術的な障壁がない限り、最新のサポートされているランタイムを使用
2. **非推奨期限確認**: 非推奨まで12ヶ月以上あるランタイムを選択
3. **MCP確認必須**: テンプレート作成前にMCPでAWS Lambda runtimesドキュメントを確認
4. **長期サポート重視**: 同等の機能であれば、より長期サポートが予定されているランタイムを選択

### AWS CLI 実行時の注意点

#### ページネーション対策（必須）

**AWS CLIコマンド実行時は、出力が途中で止まることを防ぐため、以下の方法を使用する**

**推奨方法：**
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

**その他の方法：**
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

**適用場面：**
- **検証・テスト時**: 全出力を確認する必要がある場合
- **デバッグ時**: エラー詳細や設定内容を完全に把握したい場合
- **レポート作成時**: 正確な情報を取得する必要がある場合

## サービス別ガイドライン

### Lambda関数

#### 構造
```
src/
├── function-name/
│   ├── app.py          # メイン処理
│   ├── requirements.txt # 依存関係
│   └── tests/          # テストコード
```

#### ベストプラクティス
- 環境変数で設定値管理
- ログ出力はstructured logging
- エラーハンドリングの徹底
- タイムアウト設定の適切な設定

### データベース

#### 選択基準
- **DynamoDB**: NoSQL、高スケーラビリティが必要
- **RDS**: RDBMS、複雑なクエリが必要
- **Aurora Serverless**: 可変ワークロード

#### 設計原則
- パーティションキーの適切な設計
- インデックス戦略の検討
- バックアップ・復旧戦略

### 監視・ログ

#### CloudWatch
- カスタムメトリクスの活用
- アラーム設定による異常検知
- ダッシュボードでの可視化

#### X-Ray
- 分散トレーシングの有効化
- パフォーマンス分析
- エラー原因の特定

### コスト管理

#### 基本方針
- **使用量ベース課金**の活用
- **予約インスタンス**の検討（長期利用の場合）
- **スポットインスタンス**の活用（バッチ処理等）

#### 監視
- Cost Explorerでのコスト分析
- Budgetsでの予算管理
- タグベースのコスト配分
