# 開発ガイドライン

## 基本方針

### 言語とドキュメント
- **ドキュメントの記載とチャットの会話は日本語**
- コード内のコメントは英語でも可
- 変数名、関数名は英語を使用

### 開発プロセス
- **Infrastructure as Code**: CloudFormation/SAMテンプレートでインフラ管理
- **CI/CD**: AWS CodePipeline等による自動デプロイ
- **監視・ログ**: CloudWatch、X-Rayによる可観測性確保

## 技術スタック

### Core Technologies
- **Infrastructure as Code**: AWS CloudFormation (YAML format)
- **Serverless Framework**: AWS SAM (Serverless Application Model)
- **Runtime**: Python 3.x for Lambda functions
- **Configuration Management**: YAML configuration files
- **Linting**: CFN Lint (.cfnlintrc configuration)
- **Version Control**: Git with pre-commit hooks

### Build System
- **SAM CLI**: For serverless application building and deployment
- **Pre-commit**: Code quality and linting automation
- **CFN Lint**: CloudFormation template validation

## Common Commands

### SAM Applications
```bash
# Build SAM application
sam build

# Deploy SAM application (check samconfig.toml first)
sam deploy

# Deploy with guided setup (only if no samconfig.toml)
sam deploy --guided

# Local testing
sam local start-api
sam local invoke
```

### CloudFormation
```bash
# Validate template
aws cloudformation validate-template --template-body file://template.yaml

# Deploy stack
aws cloudformation deploy --template-file template.yaml --stack-name my-stack

# Lint templates
cfn-lint template.yaml
```

## Dependencies
- AWS CLI configured with appropriate permissions
- SAM CLI for serverless applications
- Python 3.x and pip for Lambda dependencies
- CFN Lint for template validation

## タスク実行時の必須プロセス

### 実装開始前の必須確認
- **ガイドライン参照指示の確認**: タスクに「〜に従って」「〜を遵守」「〜.mdに従って」と記載がある場合、**必ずそのファイルを最初に読み込む**
- **参照ファイルの理解**: 参照ファイルの内容を理解してから実装を開始する
- **要件のチェックリスト化**: 参照ファイルの要件をチェックリスト化して実装中に確認する

### 実装完了前の必須検証
- **全指示項目の確認**: タスクに記載された全ての指示項目を一つずつ確認
- **検証項目の実行**: 特に「CFN Lint実行（exit code 0必須）」等の検証項目は必ず実行
- **ガイドライン準拠確認**: ガイドライン準拠が要求されている場合は準拠状況を確認

### CloudFormation/SAMテンプレート作成/更新時の必須手順
1. **CloudFormation Template Guidelines確認**: ステアリングファイルのcloudformation-template-guidelines.mdを参照
2. **必須セクション順序の遵守**: AWSTemplateFormatVersion → Transform → Description → Metadata → Parameters → Resources → Outputs
3. **Description形式の適用**: `aws-cloudformation-templates/[service]/[template] [action] [service].`
4. **Metadata構造の実装**: `AWS::CloudFormation::Interface`を使用
5. **標準パラメータの追加**: AlarmLevel, SNSForAlertArn, SNSForDeploymentArn, TagKey, TagValue
6. **CFN Lint検証**: exit code 0を確保

## SAMデプロイメント原則

### デプロイ前の必須確認手順（全プロジェクト共通）
1. **プロジェクト固有の設計書確認**: デプロイ前に必ず該当プロジェクトの設計書（design.md）を確認する
2. **設定ファイル管理方式の確認**: 設定ファイルの配置場所と管理方式を設計書で確認する
   - プロジェクトディレクトリ内（標準的な配置）
   - プロジェクトディレクトリ外（プライベートリポジトリ等）
   - 機密情報分離の有無
3. **設定ファイル取得**: 設計書の指示に従って必要な設定ファイルを取得・配置する
4. **デプロイ手順の確認**: プロジェクト固有のデプロイ手順があるかを設計書で確認する

### 必須デプロイ手順
1. **sam build実行**: デプロイ前に必ず`sam build`を実行する
2. **samconfig.tomlの存在確認**: プロジェクトディレクトリにsamconfig.tomlがあるかチェック
3. **設定ファイルがある場合**: `sam deploy` を使用（--guidedは不要）
4. **設定ファイルがない場合**: `sam deploy --guided` を使用して初回設定

### CloudFormationスタック障害対応原則
1. **スタック削除禁止**: UPDATE_FAILEDやUPDATE_ROLLBACK_FAILEDの場合でもスタックを削除しない
2. **更新ロールバック続行**: `aws cloudformation continue-update-rollback`を使用してロールバックを続行
3. **根本原因調査**: CloudWatch Logsでエラーの詳細を確認
4. **問題修正後再デプロイ**: 原因を修正してから再度デプロイを実行

### パラメータ設定ガイドライン

#### 1. 新たなパラメータが必要な場合
- **必ず実際の値を確認してから設定する**
- 推測や仮定でテスト値を設定しない
- 値が不明な場合は明確に質問する
- ユーザーから値を取得するまでデプロイを実行しない

#### 2. デプロイ方法の決定
- `samconfig.toml`の存在を確認
- 存在する場合：既存設定を活用し、不足パラメータのみ追加
- 存在しない場合：`sam deploy --guided`を使用
- パラメータ追加時は`--parameter-overrides`で指定

#### 3. 機密情報の取り扱い
- Client SecretなどはNoEcho: trueで保護
- samconfig.tomlには機密情報を直接記載しない
- デプロイ時の手動指定を推奨

### デプロイコマンド例
```bash
# 1. 設計書確認（必須）
# プロジェクトの .kiro/specs/[project-name]/design.md を確認

# 2. 設定ファイル取得（設計書の指示に従う）
# 例：プライベートリポジトリから取得する場合
cp ../../../aws-cfn-conf-[account]/[project]/samconfig.toml .

# 3. 必須：デプロイ前に必ずビルドを実行
sam build

# 4. デプロイ実行
# 設定ファイルがある場合
sam deploy

# 新しいパラメータを追加する場合
sam deploy --parameter-overrides \
  NewParameter="USER_PROVIDED_VALUE"

# 設定ファイルがない場合のみ
sam deploy --guided
```

## CloudFormationスタック障害対応

### UPDATE_ROLLBACK_FAILED状態の対処法

**重要**: スタックがUPDATE_ROLLBACK_FAILED状態になった場合、スタックを削除せずに更新ロールバックを続行する

#### 対処手順
1. **CloudWatch Logsで根本原因を調査**
   ```bash
   # Lambda関数のログを確認
   AWS_PAGER="" aws logs describe-log-groups --log-group-name-prefix "/aws/lambda/[function-name]"
   
   # CloudFormationイベントを確認
   AWS_PAGER="" aws cloudformation describe-stack-events --stack-name [stack-name]
   ```

2. **更新ロールバックの続行**
   ```bash
   # 更新ロールバックを続行（スタックを削除しない）
   aws cloudformation continue-update-rollback --stack-name [stack-name]
   ```

3. **問題修正後の再デプロイ**
   ```bash
   # SAMの場合：必須：修正後は必ずビルドしてからデプロイ
   sam build
   sam deploy
   
   # CloudFormationの場合：直接デプロイ
   aws cloudformation deploy --template-file template.yaml --stack-name [stack-name]
   ```

#### 避けるべき行動
- ❌ スタックの削除
- ❌ 原因調査なしでの再作成
- ❌ sam buildを忘れてのデプロイ
- ❌ 設計書を確認せずにデプロイ
- ❌ 一般的なパターンを前提とした作業

#### 正しい行動
- ✅ デプロイ前の設計書確認
- ✅ プロジェクト固有の設定ファイル管理方式の確認
- ✅ CloudWatch Logsでの根本原因調査
- ✅ continue-update-rollbackでの復旧
- ✅ 問題修正後の再デプロイ（SAM: sam build → sam deploy、CloudFormation: aws cloudformation deploy）

### パラメータ値確認の必須フロー
1. **新しいパラメータの検出**: テンプレートに新しいパラメータが追加された場合
2. **ユーザーへの確認**: 「[パラメータ名]の値を教えてください」と明確に質問
3. **値の取得待ち**: ユーザーから実際の値を取得するまで待機
4. **デプロイ実行**: 実際の値を使用してデプロイを実行

## 品質保証
- CFN Lintでexit code 0必須
- テンプレート構造の統一
- 命名規則の遵守
- **CloudFormation Template Guidelines完全準拠**