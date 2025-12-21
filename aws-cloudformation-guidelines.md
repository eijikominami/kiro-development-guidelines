---
inclusion: fileMatch
fileMatchPattern: ['**/*.yaml', '**/template.yaml', '**/sam-app/**']
---

# AWS CloudFormation ガイドライン

CloudFormation/SAM テンプレートの標準形式を定義する。

## テンプレート作成前の確認事項

### MCP によるドキュメント確認

CloudFormation/SAM テンプレートを作成・修正する前に MCP で以下を確認する：

1. リソーススキーマ情報（プロパティ、必須項目、制約）
2. 最新の設定オプション

### ランタイム選択

Lambda ランタイムは MCP で最新バージョンを確認し、非推奨まで12ヶ月以上あるものを選択する。

## テンプレート構造

### セクション順序

以下の順序で記述する：

1. AWSTemplateFormatVersion
2. Transform（SAM テンプレートのみ）
3. Description
4. Globals（SAM のみ）
5. Metadata
6. Parameters
7. Conditions
8. Resources
9. Outputs

セクション間は1行の空白行で区切る。

## ヘッダーセクション

### Description 形式

```yaml
AWSTemplateFormatVersion: 2010-09-09
Transform: AWS::Serverless-2016-10-31
Description: aws-cloudformation-templates/[service-name]/[template-name] [action] [AWS service/resource].
```

アクション動詞: `sets`（設定）、`creates`（作成）、`builds`（複合リソース）

### SAM Globals

```yaml
Globals:
  Function:
    Architectures:
      - arm64
    Handler: lambda_function.lambda_handler
    Runtime: python3.13
    Tracing: Active
```

## Metadata セクション

### パラメータグループ順序

1. サービス固有の設定グループ
2. セカンダリ設定グループ
3. Notification Configuration
4. Tag Configuration

ラベルはシングルクォートで囲み、`'Configuration'` で終わる。

```yaml
Metadata: 
  AWS::CloudFormation::Interface:
    ParameterGroups: 
      - Label: 
          default: 'CloudFront Configuration'
        Parameters: 
          - CloudFrontDefaultTTL
      - Label: 
          default: 'Notification Configuration'
        Parameters: 
          - SNSForAlertArn
      - Label: 
          default: 'Tag Configuration'
        Parameters:
          - Environment
          - TagKey
          - TagValue
```

## Parameters セクション

### パラメータ順序

アルファベット順に並べる。

### パラメータプロパティ順序

Type → Default → AllowedValues → AllowedPattern → MinValue → MaxValue → Description

```yaml
ParameterName:
  Type: String
  Default: value
  AllowedValues:
    - value1
    - value2
  AllowedPattern: .+
  Description: Description text
```

### 標準パラメータ

```yaml
Parameters:
  AlarmLevel:
    Type: String
    Default: NOTICE
    AllowedValues:
      - NOTICE
      - WARNING
    Description: The alarm level of CloudWatch alarms
  Environment:
    Type: String
    Default: production
    AllowedValues:
      - production
      - test
      - development
  SNSForAlertArn:
    Type: String
    Default: ''
    Description: The Amazon SNS topic ARN for alert
  SNSForDeploymentArn:
    Type: String
    Default: ''
    Description: The Amazon SNS topic ARN for deployment information
  TagKey:
    Type: String
    Default: createdby
    AllowedPattern: .+
  TagValue:
    Type: String
    Default: aws-cloudformation-templates
    AllowedPattern: .+
```

### 命名規則

- サービス固有: サービス名で始まる（例: `CloudFrontDefaultTTL`）
- ブール型: `ENABLED/DISABLED` を使用（`true/false` ではない）
- 必須パラメータ: 説明に `[required]` を付ける

## Conditions セクション

アクション動詞で始まる名前を使用し、アルファベット順に並べる。

例: `CreateSNSForAlert`, `EnableLogging`, `HasCustomDomain`

## Resources セクション

### リソース順序

1. ネストされたスタック（`AWS::Serverless::Application`, `AWS::CloudFormation::Stack`）
2. IAM リソース
3. その他（論理的な関係でグループ化、その後アルファベット順）

### リソース命名規則

PascalCase で **ResourceTypeFor[Purpose]** パターンを使用する。

```yaml
S3BucketForIcebergData:
SecretsManagerForGoogleAnalytics:
IAMRoleForLambda:
SecurityGroupForRDS:
LambdaFunctionForProcessor:
AlarmForLambdaErrors:
```

### IAM ロール

```yaml
Description: !Sub IAM role for [Service/Purpose] in ${AWS::StackName}
RoleName: !Sub ${AWS::StackName}-[Service]-${AWS::Region}  # 64文字制限に注意
```

### リソースプロパティ順序

Condition → DependsOn → Type → Properties（サブプロパティはアルファベット順）

### タグ構造

```yaml
Tags:
  - Key: Name
    Value: !Sub resource-${LogicalName}-${Purpose}
  - Key: environment
    Value: !Ref Environment
  - Key: !Ref TagKey
    Value: !Ref TagValue
```

## SNS トピック

AWS Serverless Repository アプリケーションを使用する。直接 `AWS::SNS::Topic` を作成しない。

```yaml
SNSForAlert:
  Condition: CreateSNSForAlert
  Type: AWS::Serverless::Application
  Properties:
    Location:
      ApplicationId: arn:aws:serverlessrepo:us-east-1:172664222583:applications/sns-topic
      SemanticVersion: 2.2.24
    Parameters:
      TopicName: !Sub Alert-createdby-${AWS::StackName}
    Tags:
      environment: !Ref Environment
      createdby: !Ref TagValue

Conditions:
  CreateSNSForAlert: !Equals [!Ref SNSForAlertArn, '']
```

## CloudWatch Alarm

https://github.com/eijikominami/aws-cloudformation-templates/tree/master/monitoring のアプリケーションを使用する。

## Outputs セクション

### 含めるべき Outputs

- クロススタック参照リソース（必須）
- 主要サービスリソース（オプション）
- IAM ロールとポリシー（外部参照が必要な場合のみ）
- リソース識別子（統合に必要な場合）

### Output 命名規則

- リソース識別子: `VPCId`, `SubnetIds`, `SecurityGroupId`
- ARN: `IAMRoleForLambdaArn`, `S3BucketArn`
- 名前: `LambdaFunctionName`, `GlueConnectionName`
- URL: `APIGatewayURL`, `CloudFrontDistributionURL`

### Output 順序

1. タイプ優先順位: IDs → ARNs → Names → URLs → その他
2. タイプ内でアルファベット順
3. 条件付き出力は最後

```yaml
Outputs:
  ResourceId:
    Description: Clear description of the resource identifier
    Value: !Ref ResourceName
    
  ResourceArn:
    Description: ARN of the [resource type] for [purpose]
    Value: !GetAtt ResourceName.Arn
    
  ConditionalResource:
    Condition: CreateResource
    Description: Description for conditional resource
    Value: !Ref ConditionalResourceName
```

## 文字列引用符

### シングルクォートを使用

- Metadata ラベル
- スペースを含む文字列
- 特殊文字を含む文字列
- バージョン番号
- JSON 文字列

### 引用符なし

- シンプルなパラメータ説明
- リソース名
- ブール値、数値
- ENABLED/DISABLED

### 埋め込みコード

YAML 内の Python/JavaScript/Shell コードは対象言語の引用符規約に従う。

## 組み込み関数

短縮形を使用する：
- `!Ref` （`Ref:` ではなく）
- `!GetAtt` （`Fn::GetAtt:` ではなく）
- `!Sub` （`Fn::Sub:` ではなく）

```yaml
# 推奨
Value: !Sub '${ResourceName}-${Environment}'

# 複雑な置換
Value: !Sub 
  - '${Resource}-${Env}'
  - Resource: !Ref ResourceName
    Env: !Ref Environment
```

## ネストされたスタック

### 基本実装

```yaml
# SAM テンプレート
ChildStack:
  Type: AWS::Serverless::Application
  Properties:
    Location: ./child-module.yaml
    Parameters:
      LogicalName: !Ref LogicalName
      Parameter1: !Ref Parameter1
    Tags:
      environment: !Ref Environment

# CloudFormation テンプレート
ChildStack:
  Type: AWS::CloudFormation::Stack
  Properties:
    TemplateURL: https://s3.amazonaws.com/bucket/child-module.yaml
    Parameters:
      LogicalName: !Ref LogicalName
```

### 命名規則

- 子スタックファイル名: `{機能名}.yaml`（例: `google-analytics.yaml`）
- 親スタック識別子: `LogicalName` パラメータを使用

### 値の受け渡し

パラメータ経由で全ての値を渡す。`Fn::ImportValue` は使用しない。

## ファイル構成

### ディレクトリ構造

```
aws-cloudformation-templates/
├── [service-name]/                    # 小文字、ハイフン区切り
│   ├── README.md
│   ├── README_JP.md
│   ├── templates/
│   │   ├── template.yaml              # メインテンプレート
│   │   └── [feature].yaml
│   └── sam-app/
│       ├── template.yaml
│       └── [function-name]/
│           ├── lambda_function.py     # 常にこの名前
│           └── requirements.txt
```

### テンプレートファイル命名

- メインテンプレート: `template.yaml`
- 個別サービス: `[aws-service-name].yaml`
- 機能固有: `[feature-name].yaml`

## 検証

### cfn-lint（必須）

```bash
cfn-lint template.yaml
```

終了コード 0 を確保する。全ての警告とエラーを解決する。

### 追加検証

```bash
aws cloudformation validate-template --template-body file://template.yaml
sam validate
```

## デプロイ

### デプロイ前確認

1. プロジェクトの設計書（design.md）を確認
2. 設定ファイル管理方式を確認
3. 必要な設定ファイルを取得・配置

### SAM デプロイ

```bash
# ビルド
sam build

# デプロイ（samconfig.toml がある場合）
sam deploy

# 初回設定（samconfig.toml がない場合）
sam deploy --guided

# 新しいパラメータを追加する場合
sam deploy --parameter-overrides NewParameter="VALUE"
```

### パラメータ設定

- 実際の値を確認してから設定する
- 推測や仮定でテスト値を設定しない
- 値が不明な場合はユーザーに確認する
- 機密情報は `NoEcho: true` で保護

## トラブルシューティング

### UPDATE_ROLLBACK_FAILED 状態

スタックを削除せずに更新ロールバックを続行する。

```bash
# 根本原因を調査
AWS_PAGER="" aws cloudformation describe-stack-events --stack-name [stack-name]

# 更新ロールバックを続行
aws cloudformation continue-update-rollback --stack-name [stack-name]

# 問題修正後の再デプロイ
sam build
sam deploy
```

❌ スタックの削除、原因調査なしでの再作成は避ける。

## チェックリスト

### デプロイ前

- [ ] cfn-lint が終了コード 0 で合格
- [ ] テンプレートがセクション順序に従っている
- [ ] Description が正確な形式パターンに従っている
- [ ] パラメータがアルファベット順
- [ ] リソースがタイプ優先順位で並んでいる
- [ ] リソース名が命名規則に従っている
- [ ] タグ構造が一貫している
- [ ] 組み込み関数が短縮形を使用している
- [ ] クロススタック参照リソースが出力されている
- [ ] 不要な出力がない

### アンチパターン

- ❌ 不正なセクション順序
- ❌ パラメータがアルファベット順でない
- ❌ 汎用的なリソース名（`Resource1`, `MyResource`）
- ❌ 直接的な `AWS::SNS::Topic` 作成
- ❌ 長い形式の組み込み関数
- ❌ `Fn::ImportValue` の使用
- ❌ 出力の肥大化
