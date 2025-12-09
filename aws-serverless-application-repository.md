---
inclusion: fileMatch
fileMatchPattern: 'aws-cloudformation-templates/monitoring/**'
---

# AWS Serverless Application Repository 管理ガイドライン

## 適用スコープ

このガイドラインは `aws-cloudformation-templates/monitoring/` プロジェクトにのみ適用されます。

## 基本方針

**再利用可能なリソースは、AWS Serverless Application Repository に登録可能であれば、必ず登録する。**

### 対象リソースの判断基準

#### 必須確認事項

新しいリソースを作成する際は、以下の手順で AWS Serverless Application Repository への登録可否を判断する：

1. **MCP で AWS ドキュメントを確認**
   - 検索キーワード：「AWS Serverless Application Repository supported resources [リソースタイプ]」
   - 公式ドキュメント：https://docs.aws.amazon.com/serverlessrepo/latest/devguide/list-supported-resources.html

2. **登録可能な場合は必ず登録**
   - サポートリソースリストに含まれている場合は、必ず AWS Serverless Application Repository に登録
   - 個別のテンプレートとして作成せず、再利用可能なアプリケーションとして管理

3. **登録不可能な場合のみ個別実装**
   - サポートリソースリストに含まれていない場合のみ、個別のテンプレートとして実装

#### 主なサポート対象リソース

AWS Serverless Application Repository は以下のリソースタイプをサポートしています（一部抜粋）：

- **監視・ログ**: `AWS::CloudWatch::Alarm`, `AWS::CloudWatch::Dashboard`, `AWS::Logs::LogGroup`
- **通知**: `AWS::SNS::Topic`, `AWS::SNS::Subscription`
- **Lambda**: `AWS::Lambda::Function`, `AWS::Lambda::LayerVersion`
- **IAM**: `AWS::IAM::Role`, `AWS::IAM::Policy`
- **S3**: `AWS::S3::Bucket`, `AWS::S3::BucketPolicy`
- **その他多数**: API Gateway, DynamoDB, Glue, Step Functions, Secrets Manager 等

完全なリストは MCP で確認してください。

### 理由

1. **再利用性**: 同じ設定を複数のプロジェクトで使い回せる
2. **バージョン管理**: ApplicationId でバージョンを管理できる
3. **保守性**: 更新が一箇所で済む
4. **標準化**: 組織全体で統一された設定を適用できる

### 現在の登録状況

このプロジェクトでは、以下のリソースを AWS Serverless Application Repository で管理しています：

- **CloudWatch Alarm**: Lambda、Glue、ACM、その他 AWS サービス用
- **SNS Topic**: アラート通知用

## 監視実装の二段階プロセス（必須）

**監視リソース（CloudWatch Alarms等）を実装する際は、必ず以下の二段階プロセスに従う。**

### Phase 1: AWS Serverless Application Repository の確認と登録

#### 1.1 MCP でサポート対象リソースタイプを確認

```
検索キーワード：「AWS Serverless Application Repository supported resources CloudWatch Alarm」
```

#### 1.2 既存アプリケーションの確認

以下の監視アプリケーションが AWS Serverless Application Repository に存在するか確認：

- Visual ETL Job（Glue Job）監視アプリケーション
- Lambda 関数監視アプリケーション
- コスト監視アプリケーション
- その他必要な監視アプリケーション

#### 1.3 存在しない場合は作成・登録

存在しない場合は、`aws-cloudformation-templates/monitoring` プロジェクトで作成・登録：

1. **SAM テンプレート作成**
   - `sam-app/[service].yaml` を作成
   - cloudformation-template-guidelines.md に記載の制約を守る
   - CFN Lint 実行（exit code 0 必須）

2. **README 作成**
   - `readme/cloudwatch-alarm-about-[service].md` を作成

3. **Metadata セクション追加**
   - テンプレートに `Metadata: AWS::ServerlessRepo::Application` セクションを追加
   - `SemanticVersion` は含めない（`--semantic-version` オプションで渡すため）

4. **buildspec 更新**
   - `cicd/codebuild/buildspec-upload-artifacts-serverlessrepo.yml` の `for` ループに追加

5. **Git タグでリリース**
   - `v[version]-rc` パターンのタグを作成
   - CodeBuild が自動的に `sam publish` で Serverless Application Repository に登録

6. **ApplicationId を記録**
   - 登録後に発行される ApplicationId を記録

**重要な変更点**:
- テンプレートに `Metadata` セクションが必須
- `SemanticVersion` はテンプレートに含めない（`--semantic-version` オプションで渡す）
- `sam publish` を使用（S3 URL の手動管理が不要）

詳細は `aws-cloudformation-templates/monitoring/CONTRIBUTING.md` を参照。

### Phase 2: CloudFormation テンプレートへの追加

#### 2.1 AWS::Serverless::Application で参照

Phase 1 で確認・登録した Serverless Application Repository アプリケーションを参照：

```yaml
Resources:
  CloudWatchAlarmGlue:
    Type: AWS::Serverless::Application
    Properties:
      Location:
        ApplicationId: arn:aws:serverlessrepo:us-east-1:172664222583:applications/cloudwatch-alarm-about-glue
        SemanticVersion: 1.0.0
      Parameters:
        ResourceName: !Ref VisualETLJob
        AlarmEmail: !Ref AlarmEmail
```

#### 2.2 CFN Lint 実行

```bash
cfn-lint template.yaml
```

**exit code 0 であることを確認**

### タスク分割の原則

**tasks.md で監視実装タスクを記載する際は、必ず以下の 2 つのタスクに分割する：**

#### Task X.1a: AWS Serverless Application Repository の確認と登録

- MCP でサポート対象リソースタイプを確認
- 必要な監視アプリケーションが Serverless Application Repository に存在するか確認
- 存在しない場合は、aws-cloudformation-templates/monitoring プロジェクトで作成・登録
- ApplicationId を記録
- **人間チェック**: Serverless Application Repository 登録内容と ApplicationId を確認

#### Task X.1b: CloudWatch 監視の実装

- Task X.1a で確認・登録した Serverless Application Repository アプリケーションを AWS::Serverless::Application で参照
- CloudFormation テンプレートに監視リソースを追加
- CFN Lint 実行（exit code 0 必須）
- **人間チェック**: 監視設定とアラーム設定を確認

### 避けるべきパターン

- ❌ 直接 CloudWatch Alarms を CloudFormation テンプレートに実装
- ❌ Serverless Application Repository の確認をスキップ
- ❌ 未登録のアプリケーションを参照
- ❌ 監視実装を 1 つのタスクにまとめる

### 正しいパターン

- ✅ Phase 1: Serverless Application Repository 確認・登録
- ✅ Phase 2: CloudFormation テンプレートで参照
- ✅ タスクを 2 つに分割（Task X.1a と Task X.1b）

## 重要ルール（必須）

### リソース命名規則

**AWS::Serverless::Application リソースの命名は、監視対象サービス名を使用する。**

#### CloudWatch Alarm 用の命名パターン

```yaml
# 標準パターン: CloudWatchAlarm[ServiceName]
CloudWatchAlarmLambda:
  Type: AWS::Serverless::Application
  Properties:
    Location:
      ApplicationId: arn:aws:serverlessrepo:us-east-1:172664222583:applications/cloudwatch-alarm-about-lambda

CloudWatchAlarmGlue:
  Type: AWS::Serverless::Application
  Properties:
    Location:
      ApplicationId: arn:aws:serverlessrepo:us-east-1:172664222583:applications/cloudwatch-alarm-about-glue

CloudWatchAlarmEC2:
  Type: AWS::Serverless::Application
  Properties:
    Location:
      ApplicationId: arn:aws:serverlessrepo:us-east-1:172664222583:applications/cloudwatch-alarm-about-ec2
```

#### 複数の同一サービスを監視する場合

```yaml
# 数字サフィックスを追加
CloudWatchAlarmMediaLive0:
  Type: AWS::Serverless::Application

CloudWatchAlarmMediaLive1:
  Type: AWS::Serverless::Application
```

#### 避けるべき命名

- ❌ `LambdaMonitoringStack` - "Stack" サフィックスは不要
- ❌ `MonitoringLambda` - "CloudWatchAlarm" プレフィックスが必要
- ❌ `LambdaAlarm` - "CloudWatch" が欠けている

### 実装順序の厳守

**親テンプレートを作成する前に、必ず Serverless Application Repository への登録を完了させる。**

1. ✅ SAM テンプレート作成
2. ✅ README 作成
3. ✅ buildspec 更新（重要）
4. ✅ Git タグでリリース
5. ✅ ApplicationId 取得
6. ✅ 親テンプレートで参照

この順序を守らないと、親テンプレートのデプロイ時に ApplicationId が存在せずエラーになります。

### 必須チェック項目

- [ ] CFN Lint が exit code 0 で成功している
- [ ] テンプレートに `Metadata: AWS::ServerlessRepo::Application` セクションが追加されている
- [ ] `SemanticVersion` がテンプレートに含まれていない（`--semantic-version` オプションで渡すため）
- [ ] buildspec（`cicd/codebuild/buildspec-upload-artifacts-serverlessrepo.yml`）に追加されている
  - [ ] `build` フェーズの `for` ループにテンプレートパスを追加
  - [ ] `post_build` フェーズの `for` ループにアプリケーション ID を追加
- [ ] `v[version]-rc` パターンのタグを作成している
- [ ] パブリック公開ポリシーを設定している（`aws serverlessrepo put-application-policy`）
- [ ] デプロイして動作確認している

### buildspec 更新の重要性

**buildspec を更新せずにタグをプッシュしても、新しいテンプレートは Serverless Application Repository に登録されません。**

新しいテンプレートを追加する際は、必ず buildspec ファイルの `build` フェーズと `post_build` フェーズの両方の `for` ループに追加してください。

### sam publish 方式の利点

- ✅ S3 URL の手動管理が不要
- ✅ テンプレートの Metadata セクションから自動的に情報を取得
- ✅ `--semantic-version` オプションでバージョンを動的に指定可能
- ✅ 新規作成と更新を同じコマンドで実行可能

## 詳細な実装手順

具体的な実装手順、コマンド例、トラブルシューティングについては、以下のドキュメントを参照してください：

**#[[file:aws-cloudformation-templates/monitoring/CONTRIBUTING.md]]**

このドキュメントには以下が含まれています：

- 新しい CloudWatch Alarm テンプレートの追加手順（Phase 1〜4）
- 具体的なコマンド例（コピペ可能）
- バージョン更新手順
- パブリック公開手順
- 参考リンク
