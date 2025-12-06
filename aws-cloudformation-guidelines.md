---
inclusion: fileMatch
fileMatchPattern: '*.yaml'
---

# AWS CloudFormation ガイドライン

このドキュメントは、aws-cloudformation-templates プロジェクトにおける CloudFormation YAML テンプレートの包括的な標準形式を定義します。全てのサービステンプレート、ファイル構成、命名規則の一貫性を維持するための完全なリファレンスとして機能します。

## テンプレート構造と順序

### 必須セクション順序
全ての CloudFormation テンプレートは、この正確な構造と順序に従う必要があります：

1. **AWSTemplateFormatVersion** (MANDATORY)
2. **Transform** (MANDATORY for SAM templates)
3. **Description** (MANDATORY)
4. **Globals** (OPTIONAL - SAM templates only)
5. **Metadata** (MANDATORY)
6. **Parameters** (MANDATORY)
7. **Conditions** (OPTIONAL)
8. **Resources** (MANDATORY)
9. **Outputs** (OPTIONAL)

### セクション間隔ルール
- **1行の空白行**で各主要セクションを区切る必要があります
- パラメータ定義やリソースプロパティ内には**空白行を入れない**
- Resources セクション内の異なるリソースタイプ間には**1行の空白行**を入れる

## ヘッダーセクション標準

### テンプレート形式宣言
```yaml
AWSTemplateFormatVersion: 2010-09-09
Transform: AWS::Serverless-2016-10-31
Description: aws-cloudformation-templates/[service-name]/[template-name] [brief description].
```

### Description 形式ルール
Description フィールドは、この正確なパターンに従う必要があります：
- **形式**: `aws-cloudformation-templates/[service-name]/[template-name] [action] [AWS service/resource].`
- **例**:
  - `aws-cloudformation-templates/security/iam sets IAM.`
  - `aws-cloudformation-templates/network/az creates a VPC Subnet and related resources.`
  - `aws-cloudformation-templates/edge/cloudfront creates an Amazon CloudFront.`
  - `aws-cloudformation-templates/media/medialive sets Elemental MediaLive.`

### Description アクション動詞
- **sets**: 設定/セットアップテンプレート用
- **creates**: リソース作成テンプレート用
- **builds**: 複雑な複数リソーステンプレート用

### SAM Globals セクション（SAM テンプレートのみ）
```yaml
Globals:
  Function:
    Architectures:
      - arm64
    Handler: lambda_function.lambda_handler
    Runtime: python3.13  # Use latest supported Python runtime - check AWS Lambda documentation
    Tracing: Active
```

#### ランタイム選択ガイドライン
- **常に最新のサポートされている Python ランタイム**を AWS Lambda で使用する
- テンプレート作成前に **AWS Lambda ドキュメントを確認**して、現在の最新バージョンを特定する
- **非推奨タイムラインを考慮**: 非推奨まで12ヶ月未満のランタイムは避ける
- テンプレート作成時に **MCP AWS ドキュメントツールを使用**して、現在サポートされているランタイムを確認する

## Metadata セクション標準

### AWS::CloudFormation::Interface Structure
```yaml
Metadata: 
  AWS::CloudFormation::Interface:
    ParameterGroups: 
      - Label: 
          default: '[Service Name] Configuration'
        Parameters: 
          - Parameter1
          - Parameter2
      - Label: 
          default: 'Notification Configuration'
        Parameters: 
          - AlarmLevel
          - SNSForAlertArn
          - SNSForDeploymentArn
      - Label: 
          default: 'Tag Configuration'
        Parameters:
          - Environment
          - TagKey
          - TagValue
```

### パラメータグループ順序ルール
1. **サービス固有の設定グループ**（主要機能）
2. **セカンダリ設定グループ**（サポート機能）
3. **Notification Configuration**（常に最後から2番目）
4. **Tag Configuration**（常に最後）

### ラベル命名規則
ラベルは以下の正確なパターンに従う必要があります：
- **サービス設定**: `'[ServiceName] Configuration'`
  - 例: `'CloudFront Configuration'`, `'FSx Configuration'`, `'IVS Configuration'`
- **機能設定**: `'[FeatureName] Configuration'`
  - 例: `'Route53 Configuration'`, `'WAF Configuration'`, `'Logging Configuration'`
- **標準ラベル**（常にこれらの正確な名前を使用）:
  - `'Notification Configuration'`
  - `'Tag Configuration'`

### ラベル書式ルール
- ラベルテキストには常にシングルクォートを使用する
- 各主要単語の最初の文字を常に大文字にする
- 常に 'Configuration' で終わる
- 公式の AWS サービス名を使用する（例: 'Route53'、'Route 53' ではない）

## Parameters セクション標準

### パラメータ順序
Parameters セクション内では、Metadata での論理的なグループ化に関係なく、パラメータは**アルファベット順**に並べる必要があります。

### 標準パラメータ定義

#### 共通パラメータ（ほとんどのテンプレートに登場）
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

#### パラメータ命名規則
- **サービス固有のパラメータ**: サービス名で始まる説明的な名前を使用（例: `CloudFrontDefaultTTL`）
- **リソース識別子**: 明確な命名を使用（例: `VPCId`, `SubnetIds`）
- **ブール型のようなパラメータ**: true/false の代わりに ENABLED/DISABLED 値を使用
- **必須パラメータ**: 説明に `[required]` を付ける

#### パラメータプロパティ順序
```yaml
ParameterName:
  Type: String                    # Always first
  Default: value                  # Second (if applicable)
  AllowedValues:                  # Third (if applicable)
    - value1
    - value2
  AllowedPattern: .+             # Fourth (if applicable)
  MinValue: 0                    # Fifth (for numbers)
  MaxValue: 100                  # Sixth (for numbers)
  Description: Description text   # Always last
```

#### パラメータ説明標準

##### 説明形式ルール
- **冠詞で始める**: 特定の項目には "The"、一般的な項目には "A" を使用
- **コンテキストで終わる**: 必須パラメータには `[required]`、条件付きパラメータには `[SERVICE_NAME]` を追加
- **現在形を使用**: パラメータが何をするかではなく、何を表すかを説明

##### 説明パターン
```yaml
# Resource Identifiers
VPCId:
  Description: The VPC id [required]
SubnetIds:
  Description: Specifies the IDs of the subnets that the file system will be accessible from

# Configuration Values  
CloudFrontDefaultTTL:
  Description: CloudFront Default TTL [required]
StorageCapacity:
  Description: Sets the storage capacity of the file system that you're creating

# Service-Specific Parameters
MediaPackageChannelId:
  Description: The MediaPackage channel id [MEDIA_PACKAGE]
ElementalLinkId1:
  Description: The unique ID for the Elemental Link device [ELEMENTAL_LINK]

# Boolean-like Parameters
AutoInputFailover:
  Description: Enable or disable automatic input failover [required]
NetworkAddressTranslation:
  Description: Enable or disable NetworkAddressTranslation (NAT) [required]

# Standard Parameters
AlarmLevel:
  Description: The alarm level of CloudWatch alarms
SNSForAlertArn:
  Description: The Amazon SNS topic ARN for alert
Environment:
  Description: (No description needed - standard parameter)
```

##### 説明コンテキストタグ
- **[required]**: テンプレート機能に必須のパラメータ
- **[SERVICE_NAME]**: 特定のサービス/機能が有効な場合のみ適用されるパラメータ
- **[CONDITIONAL]**: 他のパラメータ値に依存するパラメータ
- **タグなし**: オプションのパラメータ

## Conditions セクション標準

### Condition 命名
- アクション動詞で始まる説明的な名前を使用: `Create`, `Enable`, `Has`
- 例: `CreateSNSForAlert`, `EnableLogging`, `HasCustomDomain`

### Condition 順序
Condition をアルファベット順に並べる。

## Resources セクション標準

### リソース順序ルール
リソースは以下の優先順位で並べる必要があります：

1. **ネストされたスタック**（`AWS::Serverless::Application`, `AWS::CloudFormation::Stack`）
2. **IAM リソース**（`AWS::IAM::Role`, `AWS::IAM::Policy` など）
3. **その他全てのリソース**（論理的な関係でグループ化し、その後アルファベット順）

### リソース命名規則

#### 一般的な命名ルール
- リソース名はリソースタイプと目的を明確に示す必要があります
- 全てのリソース名に PascalCase を使用
- パターンに従う: **ResourceTypeFor[Purpose]** で明確に識別
- 広く理解されている場合を除き、略語を避ける

#### 特定のリソースタイプ命名
```yaml
# S3 Buckets - Always use S3BucketFor[Purpose] format
S3BucketForIcebergData:    # For Iceberg data storage
S3BucketForQueryResults:   # For Athena query results
S3BucketForRawData:        # For raw data staging
S3BucketForScripts:        # For Glue scripts storage
S3BucketForLogs:           # For log storage

# Secrets Manager - Always use SecretsManagerFor[Purpose] format
SecretsManagerForGoogleAnalytics:  # For Google Analytics credentials
SecretsManagerForDatabase:         # For database credentials
SecretsManagerForAPIKeys:          # For API key storage

# Route Tables
RouteTableForPublic:       # Public route table
RouteTableForPrivate:      # Private route table
RouteTableForTransit:      # Transit gateway route table

# IAM Roles - Always use IAMRoleFor[Service/Purpose] format
IAMRoleForLambda:          # For Lambda execution
IAMRoleForEC2:             # For EC2 instances
IAMRoleForCodeBuild:       # For CodeBuild projects
IAMRoleForGlue:            # For AWS Glue jobs

#### IAM ロール命名と説明標準

##### 説明形式
IAM ロールの説明は以下の形式を使用する必要があります：
```yaml
Description: !Sub IAM role for [Service/Purpose] in ${AWS::StackName}
```

Examples:
- `Description: !Sub IAM role for Lambda functions in ${AWS::StackName}`
- `Description: !Sub IAM role for AWS Glue jobs in ${AWS::StackName}`
- `Description: !Sub IAM role for Step Functions state machines in ${AWS::StackName}`

##### RoleName 形式
IAM ロール名は、一意性を確保し64文字制限を回避するために、このパターンに従う必要があります：

**標準形式（AWS::StackName が短い場合）：**
```yaml
RoleName: !Sub ${AWS::StackName}-[Service]-${AWS::Region}
```

**LogicalName 形式（AWS::StackName が長い場合）：**
```yaml
RoleName: !Sub ${LogicalName}-[Service]-${AWS::Region}
```

**サービス名の例：**
- `Lambda` - Lambda 実行ロール用
- `Glue` - AWS Glue ジョブロール用
- `StepFunctions` - Step Functions 実行ロール用
- `EC2` - EC2 インスタンスロール用
- `CodeBuild` - CodeBuild プロジェクトロール用

**完全な例：**
```yaml
# Lambda role
RoleName: !Sub ${LogicalName}-Lambda-${AWS::Region}

# Glue role
RoleName: !Sub ${LogicalName}-Glue-${AWS::Region}

# Step Functions role (longer service name)
RoleName: !Sub ${LogicalName}-stepfunctions-execution-role-${AWS::Region}
```

**文字数制限の考慮事項：**
- IAM ロール名には64文字の制限があります
- スタック名が長い場合は `${AWS::StackName}` の代わりに `${LogicalName}` を使用
- LogicalName パラメータは3-50文字である必要があります（AllowedPattern で強制）
- これにより、最終的なロール名が64文字制限内に収まることを保証

# Security Groups - Use SecurityGroupFor[Purpose] format
SecurityGroupForRDS:       # For RDS database access
SecurityGroupForELB:       # For load balancer
SecurityGroupForLambda:    # For Lambda function

# Subnets - Use SubnetFor[Purpose] format
SubnetForPublic:           # Public subnet
SubnetForPrivate:          # Private subnet
SubnetForDatabase:         # Database subnet

# Lambda Functions - Use LambdaFunctionFor[Purpose] format
LambdaFunctionForDataProcessor:    # For data processing
LambdaFunctionForNotification:     # For notifications
LambdaFunctionForETL:              # For ETL operations

# CloudWatch Alarms - Use AlarmFor[Metric/Resource] format
AlarmForCPUUtilization:    # For CPU monitoring
AlarmForDiskSpace:         # For disk space monitoring
AlarmForLambdaErrors:      # For Lambda error monitor

### リソースプロパティ標準

#### タグ構造
タグをサポートする全てのリソースは、この構造を使用する必要があります：
```yaml
Tags:
  - Key: Name
    Value: !Sub resource-${LogicalName}-${Purpose}
  - Key: environment
    Value: !Ref Environment
  - Key: !Ref TagKey
    Value: !Ref TagValue
```

#### プロパティ順序
1. **UpdateReplacePolicy**（該当する場合）
2. **DeletionPolicy**（該当する場合）
3. **Condition**（該当する場合）
4. **DependsOn**（該当する場合）
5. **Type**
6. **Properties**（AWS ドキュメントに合わせてサブプロパティをアルファベット順に）

## Outputs セクション標準

### Output 命名
- 出力内容を明確に示す説明的な名前を使用
- PascalCase を使用
- 一般的なパターン: `Id`, `Arn`, `URL`, `Name`

### Output 構造
```yaml
Outputs:
  ResourceId:
    Description: Clear description of the output
    Value: !Ref ResourceName
  ResourceArn:
    Condition: ConditionalOutput  # If applicable
    Description: Clear description of the output
    Value: !GetAtt ResourceName.Arn
```

### Output 順序
Output をアルファベット順に並べる。

## 文字列引用符標準

### 引用符使用ルール
YAML での一貫した文字列引用符のために、これらのルールに従います：

#### 常にシングルクォートを使用
- **Metadata ラベル**: 全てのラベル値はシングルクォートを使用する必要があります
- **スペースを含む文字列値**: スペースを含む全ての文字列
- **特殊文字を含む文字列値**: コロン、ブラケット、または YAML 予約文字を含む文字列
- **バージョン番号**: CloudFormation バージョン、API バージョン
- **複雑な文字列パターン**: 正規表現、JSON 文字列

```yaml
# Metadata labels (ALWAYS quoted)
Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: 'Service Configuration'    # ALWAYS single quotes

# String values with spaces (ALWAYS quoted)
Description: 'A comprehensive CloudFormation template'

# Version numbers (ALWAYS quoted)
AWSTemplateFormatVersion: '2010-09-09'
PythonVersion: '3.11'
GlueVersion: '4.0'

# JSON strings (ALWAYS quoted)
SecretString: '{"key":"value"}'
```

#### 引用符を使用しない（プレーン文字列）
- **シンプルなパラメータ説明**: 特殊文字のない単一行の説明
- **リソース名**: シンプルな英数字のリソース識別子
- **ブール値**: true, false（YAML ネイティブ）
- **数値**: 引用符なしの数値
- **ENABLED/DISABLED**: CloudFormation 規約値
- **AWS サービス名**: シンプルなサービス識別子

```yaml
# Parameter descriptions (NO quotes)
Parameters:
  VPCId:
    Type: String
    Description: The VPC identifier for the deployment

# Simple string values (NO quotes)
Type: String
Default: production
BucketName: my-bucket-name

# Boolean and numeric values (NO quotes)
Enabled: true
Count: 5
Timeout: 300

# CloudFormation conventions (NO quotes)
AllowedValues:
  - ENABLED
  - DISABLED
```

#### 特殊なケース
- **空文字列**: 明示的な空の値には空の引用符 `''` を使用
- **数字で始まる文字列**: 数値として解釈される可能性がある場合は引用符を付ける
- **コロンを含む文字列**: コロンを含む文字列には常に引用符を付ける
- **複数行文字列**: YAML リテラルブロックスカラー `|` または折りたたみスカラー `>` を使用

```yaml
# Empty strings
Default: ''

# Strings that might be interpreted as numbers
Version: '2.0'

# Strings with colons
URL: 'https://example.com:8080'

# Multi-line strings
ScriptContent: |
  #!/bin/bash
  echo "Multi-line script"
```

### 埋め込みコード引用符ルール
YAML 文字列内に他の言語（Python、JavaScript など）のコードを埋め込む場合、YAML の規約ではなく、対象言語の引用符規約に従います：

#### YAML 内の Python コード
- **PEP 8 に従う**: 文字列にはダブルクォート、一貫性のある辞書キーにはシングルクォートを使用
- **文字列リテラル**: 可読性のためにダブルクォート `"string"` を使用
- **辞書キー**: Python 規約との一貫性のためにシングルクォート `'key'` を使用
- **混在使用**: Python のベストプラクティスに従う限り、Python コード内で許容

```yaml
# Python code embedded in YAML - follow Python conventions
ScriptContent: |
  import sys
  
  # Python string literals - use double quotes
  connection_type = "googleanalytics4"
  format_type = "glueparquet"
  
  # Python dictionary - can use single quotes for keys
  connection_options = {
      'PARTITION_FIELD': "date",
      'API_VERSION': "v1beta"
  }
  
  # Function calls with string parameters
  print(f"Processing data from {source}")
```

#### YAML 内の JavaScript コード
```yaml
# JavaScript code embedded in YAML - follow JavaScript conventions
ScriptContent: |
  const config = {
    "apiVersion": "v1",
    "connectionType": "database"
  };
  
  console.log("Script executed successfully");
```

#### YAML 内のシェルスクリプトコード
```yaml
# Shell script embedded in YAML - follow shell conventions
ScriptContent: |
  #!/bin/bash
  echo "Starting deployment"
  export API_URL="https://api.example.com"
  curl -H "Content-Type: application/json" "$API_URL"
```

## 組み込み関数標準

### 関数使用の優先順位
- `Ref:` よりも `!Ref` を優先
- `Fn::GetAtt:` よりも `!GetAtt` を優先
- `Fn::Sub:` よりも `!Sub` を優先
- 全ての組み込み関数で短縮形を使用

### Sub 関数の使用
```yaml
# Preferred
Value: !Sub '${ResourceName}-${Environment}'

# For complex substitutions
Value: !Sub 
  - '${Resource}-${Env}'
  - Resource: !Ref ResourceName
    Env: !Ref Environment
```

## コメントとドキュメント

### コメントの使用
- 主要なリソースグループを区切るためにコメントを使用
- 複雑なロジックを説明するためにコメントを使用
- 形式: `# コメントテキスト`

### リソースグループコメント
```yaml
Resources:
  # Nested Stack
  SNSForAlert:
    Type: AWS::Serverless::Application
    
  # IAM Roles
  IAMRoleForLambda:
    Type: AWS::IAM::Role
    
  # Lambda Functions
  LambdaFunction:
    Type: AWS::Lambda::Function
```

### 標準 SNS リソース形式

#### SNS トピック標準実装
全ての SNS トピックは AWS Serverless Repository アプリケーション形式を使用する必要があります：

```yaml
SNSForAlert:
  Condition: CreateSNSForAlert
  Type: AWS::Serverless::Application
  Properties:
    Location:
      ApplicationId: arn:aws:serverlessrepo:us-east-1:172664222583:applications/sns-topic
      SemanticVersion: 2.2.13
    NotificationARNs:
      - !If
        - CreateSNSForDeployment
        - !GetAtt SNSForDeployment.Outputs.SNSTopicArn
        - !Ref SNSForDeploymentArn
    Parameters:
      TopicName: !Sub Alert-createdby-${AWS::StackName}
    Tags:
      environment: !Ref Environment
      createdby: !Ref TagValue

SNSForDeployment:
  Condition: CreateSNSForDeployment
  Type: AWS::Serverless::Application
  Properties:
    Location:
      ApplicationId: arn:aws:serverlessrepo:us-east-1:172664222583:applications/sns-topic
      SemanticVersion: 2.2.13
    Parameters:
      TopicName: !Sub Deployment-createdby-${AWS::StackName}
    Tags:
      environment: !Ref Environment
      createdby: !Ref TagValue
```

#### SNS リソース命名規則
- **アラート SNS**: `SNSForAlert`
- **デプロイ SNS**: `SNSForDeployment`
- **トピック名パターン**: `Alert-createdby-${AWS::StackName}` または `Deployment-createdby-${AWS::StackName}`

#### SNS リソースに必要な Conditions
```yaml
Conditions:
  CreateSNSForAlert: !Equals [!Ref SNSForAlertArn, '']
  CreateSNSForDeployment: !Equals [!Ref SNSForDeploymentArn, '']
```

#### SNS リソースアンチパターン
❌ **直接的な AWS::SNS::Topic 作成を避ける**：
```yaml
# DO NOT USE THIS FORMAT
SNSForAlert:
  Type: AWS::SNS::Topic
  Properties:
    TopicName: !Sub "${LogicalName}-alerts-${AWS::Region}"
    DisplayName: Analytics Platform Alerts
    Tags:
      - Key: Name
        Value: !Sub "${LogicalName}-alerts-${AWS::Region}"
      - Key: environment
        Value: !Ref Environment
      - Key: !Ref TagKey
        Value: !Ref TagValue
```

✅ **上記の標準実装に示されているように AWS::Serverless::Application 形式を使用**。

## Outputs セクション標準

### Output 定義ルール
Outputs セクションは、一貫性と保守性のために以下の包括的なルールに従う必要があります：

#### 必須 Outputs（含める必要がある）
1. **クロススタック参照リソース**: 親または他のテンプレートによって参照される全てのリソース

#### オプション Outputs（含めても良い）
1. **主要サービスリソース**: テンプレートの目的を定義する主要リソース
2. **IAM ロールとポリシー**: セキュリティとアクセス管理のための IAM リソース（外部参照が必要な場合のみ）
3. **リソース識別子**: 統合に必要な ID、ARN、名前
4. **デバッグリソース**: トラブルシューティングに役立つが機能には不要
5. **便利な出力**: オペレーターにとって便利な情報
6. **監視リソース**: 可観測性のための CloudWatch リソース

#### Output 命名規則
- **リソース識別子**: リソース名 + タイプサフィックスを使用
  - 例: `VPCId`, `SubnetIds`, `SecurityGroupId`
- **ARN**: リソース名 + `Arn` サフィックスを使用
  - 例: `IAMRoleForLambdaArn`, `S3BucketArn`, `SecretsManagerArn`
- **名前**: リソース名 + `Name` サフィックスを使用
  - 例: `LambdaFunctionName`, `GlueConnectionName`
- **URL とエンドポイント**: 説明的な名前を使用
  - 例: `APIGatewayURL`, `CloudFrontDistributionURL`

#### Output 構造標準
```yaml
Outputs:
  # Resource IDs (alphabetical order)
  ResourceId:
    Description: Clear description of the resource identifier
    Value: !Ref ResourceName
    
  # Resource ARNs (alphabetical order)
  ResourceArn:
    Description: ARN of the [resource type] for [purpose]
    Value: !GetAtt ResourceName.Arn
    
  # Resource Names (alphabetical order)
  ResourceName:
    Description: Name of the [resource type] for [purpose]
    Value: !Ref ResourceName
    
  # Conditional outputs (if applicable)
  ConditionalResource:
    Condition: CreateResource
    Description: Description for conditional resource
    Value: !Ref ConditionalResourceName
```

#### Output 順序ルール
Outputs セクション内では、以下の順序で出力を並べます：
1. **タイプ優先順位**: IDs → ARNs → Names → URLs → その他
2. **タイプ内でアルファベット順**: 各タイプグループ内でアルファベット順にソート
3. **条件付き出力**: 最後に配置し、同様にアルファベット順にソート

#### Output 説明標準
- **形式**: リソースタイプで始まり、その後に目的を記述
- **例**:
  - `Description: ID of the VPC for the analytics environment`
  - `Description: ARN of the IAM role for AWS Glue Google Analytics connector`
  - `Description: Name of the Google Analytics 4 ETL job`
  - `Description: URL of the CloudFront distribution for static content`

#### クロススタック参照要件
ネストされたスタックとクロススタック参照の場合：

##### 子テンプレート要件
- **親テンプレートによって参照される全てのリソースを出力する必要があります**
- **主要サービスリソースを出力しても良い**（外部参照が必要な場合のみ）
- **IAM ロールとポリシーを出力しても良い**（外部参照が必要な場合のみ）
- **デバッグ用のリソース識別子を出力しても良い**

##### 親テンプレート要件
- **他のスタックが参照する可能性のあるリソースを出力する必要があります**
- **子スタックからの集約情報を出力すべきです**
- **オペレーター向けの便利な情報を出力しても良い**

#### テンプレートタイプ固有のルール

##### ネストされたスタックテンプレート（子テンプレート）
```yaml
Outputs:
  # Cross-referenced resources (MANDATORY)
  CrossReferencedResourceArn:
    Description: ARN of the resource referenced by parent template
    Value: !GetAtt CrossReferencedResource.Arn
    
  # Primary service resources (OPTIONAL)
  PrimaryServiceId:
    Description: ID of the primary service resource
    Value: !Ref PrimaryServiceResource
    
  # IAM resources (OPTIONAL - only if needed externally)
  IAMRoleArn:
    Description: ARN of the IAM role for [service]
    Value: !GetAtt IAMRole.Arn
    
  # Supporting resources (OPTIONAL)
  SupportingResourceName:
    Description: Name of the supporting resource for debugging
    Value: !Ref SupportingResource
```

##### メイン/親テンプレート
```yaml
Outputs:
  # Aggregated outputs from child stacks (MANDATORY)
  ChildStackOutput:
    Description: Output from child stack for external reference
    Value: !GetAtt ChildStack.Outputs.ResourceArn
    
  # Primary template resources (MANDATORY)
  MainResourceId:
    Description: ID of the main resource created by this template
    Value: !Ref MainResource
```

##### スタンドアロンテンプレート
```yaml
Outputs:
  # Integration points (OPTIONAL - only if needed for external reference)
  IntegrationEndpoint:
    Description: Endpoint for integration with other services
    Value: !GetAtt Resource.Endpoint
    
  # Primary resources (OPTIONAL - only if needed for external reference)
  PrimaryResourceId:
    Description: ID of the primary resource
    Value: !Ref PrimaryResource
    
  # IAM resources (OPTIONAL - only if needed for external reference)
  IAMRoleArn:
    Description: ARN of the IAM role
    Value: !GetAtt IAMRole.Arn
```

### Output 検証チェックリスト
- [ ] 全てのクロス参照リソースが出力されている（必須）
- [ ] Output 名が命名規則に従っている
- [ ] 説明が明確で一貫している
- [ ] Outputs が正しく順序付けられている（タイプ優先順位、その後アルファベット順）
- [ ] 条件付き出力が適切に条件付けられている
- [ ] 未使用の出力がない（どこからも参照されていない出力）
- [ ] 必要な出力のみが含まれている（出力の肥大化を避ける）
- [ ] **cfn-lint が終了コード 0 で合格する（必須）**

## SAM 固有のガイドライン

### SAM テンプレート識別
- 常に `Transform: AWS::Serverless-2016-10-31` を含める
- 共通の関数プロパティには `Globals` セクションを使用
- 利用可能な場合は CloudFormation 同等物よりも SAM リソースタイプを優先

### SAM リソースタイプ
- ネストされた SAM アプリケーションには `AWS::Serverless::Application` を使用
- Lambda 関数には `AWS::Serverless::Function` を使用
- API Gateway には `AWS::Serverless::Api` を使用

## MCP 統合要件

**CloudFormation/SAM テンプレート作成時の MCP 使用方法については、`aws-architecture.md` の「MCP 統合要件」セクションを参照してください。**

### 重要な原則
- テンプレート作成前に必ず MCP で AWS ドキュメントを確認
- リソーススキーマ情報を `get_resource_schema_information` で確認
- 実装方法が複数ある場合は、メリット・デメリットを比較してユーザーに確認を求める

詳細は `aws-architecture.md` を参照。

## ネストされたスタックアーキテクチャ

### 基本原則
**CloudFormation テンプレートの場合は `AWS::CloudFormation::Stack`、SAM テンプレートの場合は `AWS::Serverless::Application` を使用する**

### SAM テンプレートでのネストされたスタック
```yaml
# 親スタック内で子スタックを呼び出す
ChildStack:
  Type: AWS::Serverless::Application
  Properties:
    Location: ./child-module.yaml
    Parameters:
      LogicalName: !Ref LogicalName
      Parameter1: !Ref Parameter1
      Parameter2: !Ref Parameter2
    Tags:
      environment: !Ref Environment
```

### CloudFormation テンプレートでのネストされたスタック
```yaml
# 親スタック内で子スタックを呼び出す
ChildStack:
  Type: AWS::CloudFormation::Stack
  Properties:
    TemplateURL: https://s3.amazonaws.com/bucket/child-module.yaml
    Parameters:
      LogicalName: !Ref LogicalName
      Parameter1: !Ref Parameter1
      Parameter2: !Ref Parameter2
    Tags:
      - Key: environment
        Value: !Ref Environment
```

### 親子スタック間の連携方法

#### パラメータ経由での値渡し（推奨）
```yaml
# 親スタック - 子スタック呼び出し時にパラメータ渡し
ChildStack:
  Type: AWS::Serverless::Application
  Properties:
    Location: ./child-stack.yaml
    Parameters:
      SharedResourceArn: !GetAtt SharedResource.Arn

# 子スタック - パラメータで受け取り
Parameters:
  LogicalName:
    Type: String
    Description: Logical name for resource naming and cross-stack references
  SharedResourceArn:
    Type: String
    Description: ARN of shared resource from parent stack
```

#### Export/Import（非推奨）
```yaml
# 親スタック - 出力値の定義
Outputs:
  SharedResourceArn:
    Description: ARN of shared resource
    Value: !GetAtt SharedResource.Arn
    Export:
      Name: !Sub ${AWS::StackName}-SharedResourceArn

# 子スタック - 出力値の参照
Resources:
  ChildResource:
    Type: AWS::SomeService::Resource
    Properties:
      SharedResourceArn:
        Fn::ImportValue: !Sub ${LogicalName}-SharedResourceArn
```

### 設計ガイドライン

#### 使い分けの基準
- **AWS::Serverless::Application**: SAM テンプレート、ローカルファイル参照
- **AWS::CloudFormation::Stack**: CloudFormation テンプレート、S3 URL 参照

#### 依存関係管理
1. **親スタック → 子スタック**: 子スタックは親スタックのリソースを参照可能
2. **子スタック間**: 直接的な依存関係は避ける（独立性確保）
3. **削除順序**: 子スタック → 親スタックの順で削除

#### 命名規則
- **子スタックファイル名**: `{機能名}.yaml` または `{モジュール名}.yaml`
  - ✅ 良い例: `google-analytics.yaml`, `cloudfront-logs.yaml`, `s3-access-logs.yaml`
  - ❌ 悪い例: `google-analytics-stack.yaml`, `cloudfront-logs-stack.yaml`
- **理由**: シンプルで機能が明確、冗長な `-stack` サフィックスを避ける

#### パラメータ命名規則
- **親スタック識別子**: `LogicalName` パラメータを使用（`ParentStackName` ではない）
  - ✅ 良い例: `LogicalName`
  - ❌ 悪い例: `ParentStackName`, `ParentStack`, `StackName`
- **理由**: 既存のテンプレートとの一貫性、リソース命名の論理名として使用
- **用途**: パラメータ経由での値渡し、リソース名の生成に使用

#### 親子スタック間の値渡し方法
- **パラメータ経由を優先**: `Fn::ImportValue` は使用せず、パラメータで全ての値を親スタックから子スタックに渡す
- **理由**: 複雑性回避、依存関係の明確化、デバッグの容易さ
- **実装方法**: 親スタックで子スタック呼び出し時に必要な全ての値を Parameters で渡す

#### ベストプラクティス
- **モジュール化**: 機能別・データソース別に子スタックを分離
- **再利用性**: 子スタックテンプレートの汎用化
- **障害分離**: 1つの子スタックの問題が他に影響しない設計
- **パラメータ管理**: 親スタックで一元的にパラメータを管理

## 検証と品質保証

### 包括的なデプロイ前チェックリスト

#### 必須検証（最初に合格する必要がある）
- [ ] **cfn-lint 検証が終了コード 0 で合格**
- [ ] **全ての cfn-lint 警告とエラーが解決済み**
- [ ] **テンプレート構文が有効**

#### ファイル構成
- [ ] サービスディレクトリが命名規則に従っている（小文字、ハイフン区切り）
- [ ] テンプレートファイル名が確立されたパターンに従っている
- [ ] SAM Lambda 関数が正しいディレクトリ構造を使用している
- [ ] README ファイルが存在する（README.md と README_JP.md）

#### テンプレート構造
- [ ] テンプレートが必須セクション順序に従っている
- [ ] 各主要セクションが1行の空白行で区切られている
- [ ] Description が正確な形式パターンに従っている
- [ ] SAM テンプレートに Transform が含まれている
- [ ] SAM テンプレートに Globals セクションが存在する（必要な場合）

#### Metadata セクション
- [ ] パラメータグループが正しい順序に従っている
- [ ] ラベルが正確な命名規則を使用している
- [ ] 全てのラベルが 'Configuration' で終わっている
- [ ] Notification と Tag セクションに標準ラベルが使用されている

#### Parameters セクション
- [ ] パラメータがアルファベット順に並んでいる
- [ ] パラメータプロパティが正しい順序になっている
- [ ] 説明が形式標準に従っている
- [ ] コンテキストタグが適切に使用されている（[required]、[SERVICE_NAME]）
- [ ] 標準パラメータが正確な定義を使用している
- [ ] ブールパラメータが ENABLED/DISABLED 値を使用している
- [ ] AllowedPattern が入力を正しく検証している（例: S3 命名規則）

#### Resources セクション
- [ ] リソースがタイプ優先順位で並んでいる（Nested → IAM → その他）
- [ ] リソース名が命名規則に従っている
- [ ] タグ可能な全てのリソースが一貫したタグ構造を持っている
- [ ] コメントが主要なリソースグループを区切っている
- [ ] 組み込み関数が短縮形を使用している

#### Outputs セクション
- [ ] Outputs がアルファベット順に並んでいる
- [ ] Output 名が内容を明確に示している
- [ ] 全ての出力に説明がある
- [ ] クロススタック参照リソースのみが出力されている（必須）
- [ ] 不要な出力がない（出力の肥大化を避ける）

### 避けるべき一般的なアンチパターン

#### ファイル構成アンチパターン
- ❌ ディレクトリ名に CamelCase またはアンダースコアを使用
- ❌ 一貫性のないテンプレートファイル命名
- ❌ Lambda 関数ファイルが `lambda_function.py` という名前でない
- ❌ README ファイルが欠落

#### テンプレート構造アンチパターン
- ❌ 不正なセクション順序
- ❌ セクション間の空白行が欠落
- ❌ 間違った Description 形式
- ❌ SAM テンプレートに Transform が欠落

#### パラメータアンチパターン
- ❌ パラメータがアルファベット順でない
- ❌ 一貫性のないパラメータ説明
- ❌ 必須パラメータに [required] タグが欠落
- ❌ ENABLED/DISABLED の代わりに true/false を使用
- ❌ 汎用的なパラメータ名

#### リソースアンチパターン
- ❌ 汎用的なリソース名（例: `Resource1`, `MyResource`）
- ❌ 不正なリソース順序
- ❌ 一貫性のないタグ構造
- ❌ リソースタイププレフィックスが欠落（RouteTable vs RouteTablePublic）
- ❌ 長い形式の組み込み関数を使用

#### ラベルと説明のアンチパターン
- ❌ ラベルが 'Configuration' で終わっていない
- ❌ 一貫性のないラベルの大文字化
- ❌ ラベルの周りにシングルクォートが欠落
- ❌ 説明が冠詞（The/A）で始まっていない
- ❌ 説明にコンテキストタグが欠落

### 品質保証ツール

#### 必須検証（必須）
全ての CloudFormation テンプレートは、デプロイまたはコミット前に cfn-lint 検証に合格する必要があります：

```bash
# CloudFormation Linter (MANDATORY)
cfn-lint template.yaml

# Must return exit code 0 (no errors or warnings)
# Fix all warnings and errors before proceeding
```

#### 追加検証（推奨）
```bash
# Template syntax validation
aws cloudformation validate-template --template-body file://template.yaml

# SAM template validation (for SAM templates)
sam validate
```

#### cfn-lint 要件
- **インストール**: pip 経由で cfn-lint をインストール: `pip install cfn-lint`
- **実行**: テンプレート変更毎に cfn-lint を実行
- **ゼロトレランス**: 全ての警告とエラーを解決する必要がある
- **終了コード**: テンプレートが完成したと見なされる前に 0（成功）を返す必要がある

#### 手動レビューチェックリスト
1. **cfn-lint 検証（必須の最初のステップ）**
2. **ファイル命名の一貫性チェック**
3. **テンプレート構造の検証**
4. **パラメータ順序の検証**
5. **リソース命名規則のチェック**
6. **Description 形式の検証**
7. **タグ構造の一貫性**

## ファイル構成とディレクトリ構造

### プロジェクトディレクトリ構造
```
aws-cloudformation-templates/
├── [service-name]/                    # Service directory (lowercase, hyphenated)
│   ├── README.md                      # English documentation
│   ├── README_JP.md                   # Japanese documentation
│   ├── templates/                     # CloudFormation templates
│   │   ├── template.yaml              # Main service template
│   │   ├── [service-feature].yaml    # Individual feature templates
│   │   └── [aws-service].yaml        # AWS service-specific templates
│   ├── sam-app/                       # SAM applications (if applicable)
│   │   ├── template.yaml              # Main SAM template
│   │   ├── [feature].yaml            # Feature-specific SAM templates
│   │   └── [function-name]/           # Lambda function directories
│   │       ├── lambda_function.py     # Lambda code
│   │       └── requirements.txt       # Dependencies
│   ├── readme/                        # Additional documentation (optional)
│   └── scripts/                       # Helper scripts (optional)
└── images/                            # Shared images for documentation
```

### サービスディレクトリ命名
- **形式**: 単語の区切りにハイフンを使用した小文字
- **例**: `security`, `network`, `static-website-hosting`, `security-config-rules`
- **避ける**: CamelCase、アンダースコア、またはスペース

### テンプレートファイル命名規則

#### メインテンプレート
- **サービスメインテンプレート**: `template.yaml`（複数のサービスを集約）
- **個別サービステンプレート**: `[aws-service-name].yaml`
  - 例: `cloudfront.yaml`, `iam.yaml`, `fsx.yaml`
- **機能固有テンプレート**: `[feature-name].yaml`
  - 例: `autoscaling.yaml`, `realtime-dashboard.yaml`

#### SAM アプリケーションテンプレート
- **メイン SAM テンプレート**: `template.yaml`
- **機能テンプレート**: `[feature].yaml`
  - 例: `events.yaml`, `sns.yaml`

#### 特殊なテンプレート
- **ハイフン区切りの名前**: 複数単語のテンプレートにはハイフンを使用
  - 例: `centralized-logging.yaml`, `synthetics-heartbeat.yaml`
- **サービスの組み合わせ**: `[service1]-[service2].yaml`
  - 例: `ecs-codedeploy.yaml`, `application-elb.yaml`

### Lambda 関数ディレクトリ構造（SAM）
```
sam-app/
├── template.yaml                      # Main SAM template
├── [function-name]/                   # One directory per function
│   ├── lambda_function.py             # Always this exact name
│   └── requirements.txt               # Python dependencies
└── [additional-templates].yaml        # Supporting templates
```

#### Lambda 関数命名
- **ディレクトリ名**: 説明的なハイフン区切りの名前を使用
  - 例: `sendNotificationToSlack`, `analyzeUnauthorizedApiCalls`
- **ファイル名**: 常に `lambda_function.py` を使用（これを変更しない）

### テンプレートサイズと複雑性のガイドライン

#### テンプレート構成ルール
- **単一責任**: 主要な AWS サービスまたは機能ごとに1つのテンプレート
- **サイズ制限**: テンプレートあたり最大500行（推奨）
- **複雑性制限**: テンプレートあたり最大50パラメータ
- **ネストされたスタック**: 複雑な複数サービスデプロイに使用

#### テンプレート分割ガイドライン
以下の場合にテンプレートを分割：
- テンプレートが500行を超える
- 50以上のパラメータが必要
- 1つのテンプレートに複数の無関係な AWS サービス
- 異なるデプロイライフサイクルが必要

#### 分割テンプレートのファイル命名
```yaml
# Original large template
template.yaml

# Split into:
vpc.yaml              # VPC and networking
ec2.yaml              # EC2 instances and related
elb.yaml              # Load balancers
monitoring.yaml       # CloudWatch and alarms
```

## バージョン管理とメンテナンス

### テンプレートバージョニング
- 重要な変更を行う際はテンプレートの説明を更新
- ネストされたスタックアプリケーションにはセマンティックバージョニングを使用
- コミットメッセージに破壊的変更を文書化

### メンテナンスガイドライン
- AWS サービス更新のために四半期ごとにテンプレートをレビュー
- 必要に応じてパラメータのデフォルト値を更新
- 最新の CloudFormation 機能との互換性を確保
- 本番デプロイ前に開発環境でテンプレートをテスト