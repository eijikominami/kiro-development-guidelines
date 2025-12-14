# AWS アーキテクチャガイドライン

## アーキテクチャ・設計の原則

### 情報収集・調査の原則

AWS実装時は、事前学習の知識に頼らず、MCPでAWSドキュメントを確認する

#### 基本ルール
- 実装方法を決める前に、MCPでAWSドキュメントを確認
- 複数の実現方法がある場合は、それぞれ調査してメリット・デメリットを比較
- CloudFormation/SAMテンプレート作成時の具体的な手順は `aws-cloudformation-guidelines.md` を参照

#### IAMロール・ポリシー作成時の調査原則

**IAMロール・ポリシー作成時は、一般的なサービスドキュメントだけでなく、使用する具体的な機能のドキュメントまで確認する**

**確認の流れ：**
1. **サービスの基本IAM権限**を確認
   - 例：「AWS Glue IAM permissions」「AWS Lambda execution role」
2. **使用する具体的な機能の権限**を確認
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

**注意点：**
- ❌ 一般的な権限ドキュメントだけで済ませる
- ❌ 事前学習データのみに依存する
- ❌ エラーが出てから調査する
- ✅ 使用する具体的な機能のドキュメントまで確認する
- ✅ 実装前に徹底的に調査する
- ✅ 権限不足エラー時は根本原因を特定する

### 実装方法の選択

複数の実現方法があればそのメリットとデメリットを提示して確認を求める

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

### ネストされたスタックの設計原則

複雑なシステムは、機能単位で子スタックに分割して管理する。詳細は `aws-cloudformation-guidelines.md` の「ネストされたスタック実装」を参照。

## 実装時の手順

### AWS CLI 実行時の注意点

#### ページネーション対策

AWS CLIコマンド実行時は、出力が途中で止まることを防ぐため、以下の方法を使用する

**方法：**
```bash
# --no-paginate オプション使用
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

#### CloudWatch Alarms 設定の原則

CloudWatch Alarms は、要件・設計書に記載された監視要件と AWS ベストプラクティスを基に決定する

##### 監視対象の決定プロセス

1. **要件・設計書の確認**
   - `.kiro/specs/[project-name]/requirements.md` の監視要件を確認
   - `.kiro/specs/[project-name]/design.md` のエラーハンドリング・監視セクションを確認
   - 要件に明記された監視項目を特定

2. **AWS ベストプラクティスの調査**
   - MCP で AWS ドキュメントを検索：「[サービス名] monitoring CloudWatch alarms best practices」
   - AWS Well-Architected Framework の事項を確認
   - 各サービスの公式ドキュメントで監視メトリクスを確認

3. **監視項目の特定**
   - **要件・設計書に記載されている監視項目**：実装
   - **AWS ベストプラクティスで項目**：実装
   - **優先度が低く、要件・設計書に記載されていない項目**：実装不要

##### 実装時の確認事項

- **CFN Lint 検証**
   - 実装後は `cfn-lint` で検証（exit code 0）

##### 監視項目追加の判断基準

**追加すべき場合：**
- ✅ 要件・設計書に明記されている
- ✅ AWS ベストプラクティスで
- ✅ システムの健全性に直接影響する

**追加不要な場合：**
- ❌ 要件・設計書に記載がなく、優先度が低い
- ❌ 上位サービスの監視で間接的に検出可能
- ❌ 「念のため」「将来使うかも」という理由のみ

#### CloudWatch Logs
- カスタムメトリクスの活用
- ログ保持期間の適切な設定
- 構造化ログ

#### CloudWatch Dashboards
- 主要メトリクスの可視化
- リアルタイム監視
- 異常検知の迅速化

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
