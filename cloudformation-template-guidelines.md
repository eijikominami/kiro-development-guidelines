---
inclusion: fileMatch
fileMatchPattern: '*.yaml'
---

# CloudFormation Template Guidelines

This document defines the comprehensive standard format for CloudFormation YAML templates in the aws-cloudformation-templates project. It serves as a complete reference for maintaining consistency across all service templates, file organization, and naming conventions.

## Template Structure and Ordering

### Mandatory Section Order
All CloudFormation templates MUST follow this exact structure and ordering:

1. **AWSTemplateFormatVersion** (MANDATORY)
2. **Transform** (MANDATORY for SAM templates)
3. **Description** (MANDATORY)
4. **Globals** (OPTIONAL - SAM templates only)
5. **Metadata** (MANDATORY)
6. **Parameters** (MANDATORY)
7. **Conditions** (OPTIONAL)
8. **Resources** (MANDATORY)
9. **Outputs** (OPTIONAL)

### Section Spacing Rules
- **One blank line** must separate each major section
- **No blank lines** within parameter definitions or resource properties
- **One blank line** between different resource types in Resources section

## Header Section Standards

### Template Format Declaration
```yaml
AWSTemplateFormatVersion: 2010-09-09
Transform: AWS::Serverless-2016-10-31
Description: aws-cloudformation-templates/[service-name]/[template-name] [brief description].
```

### Description Format Rules
The Description field MUST follow this exact pattern:
- **Format**: `aws-cloudformation-templates/[service-name]/[template-name] [action] [AWS service/resource].`
- **Examples**:
  - `aws-cloudformation-templates/security/iam sets IAM.`
  - `aws-cloudformation-templates/network/az creates a VPC Subnet and related resources.`
  - `aws-cloudformation-templates/edge/cloudfront creates an Amazon CloudFront.`
  - `aws-cloudformation-templates/media/medialive sets Elemental MediaLive.`

### Description Action Verbs
- **sets**: For configuration/setup templates
- **creates**: For resource creation templates  
- **builds**: For complex multi-resource templates

### SAM Globals Section (SAM templates only)
```yaml
Globals:
  Function:
    Architectures:
      - arm64
    Handler: lambda_function.lambda_handler
    Runtime: python3.13  # Use latest supported Python runtime - check AWS Lambda documentation
    Tracing: Active
```

#### Runtime Selection Guidelines
- **Always use the latest supported Python runtime** available in AWS Lambda
- **Check AWS Lambda documentation** before template creation to identify the current latest version
- **Consider deprecation timeline**: Avoid runtimes with less than 12 months until deprecation
- **Use MCP AWS documentation tools** to verify current supported runtimes when creating templates

## Metadata Section Standards

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

### Parameter Group Ordering Rules
1. **Service-specific configuration groups** (primary functionality)
2. **Secondary configuration groups** (supporting features)  
3. **Notification Configuration** (always second-to-last)
4. **Tag Configuration** (always last)

### Label Naming Conventions
Labels MUST follow these exact patterns:
- **Service Configuration**: `'[ServiceName] Configuration'`
  - Examples: `'CloudFront Configuration'`, `'FSx Configuration'`, `'IVS Configuration'`
- **Feature Configuration**: `'[FeatureName] Configuration'`
  - Examples: `'Route53 Configuration'`, `'WAF Configuration'`, `'Logging Configuration'`
- **Standard Labels** (always use these exact names):
  - `'Notification Configuration'`
  - `'Tag Configuration'`

### Label Formatting Rules
- Always use single quotes around label text
- Always capitalize the first letter of each major word
- Always end with 'Configuration'
- Use the official AWS service name (e.g., 'Route53', not 'Route 53')

## Parameters Section Standards

### Parameter Ordering
Parameters MUST be ordered **alphabetically** within the Parameters section, regardless of their logical grouping in Metadata.

### Standard Parameter Definitions

#### Common Parameters (appear in most templates)
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

#### Parameter Naming Conventions
- **Service-specific parameters**: Use descriptive names starting with service name (e.g., `CloudFrontDefaultTTL`)
- **Resource identifiers**: Use clear naming (e.g., `VPCId`, `SubnetIds`)
- **Boolean-like parameters**: Use ENABLED/DISABLED values instead of true/false
- **Required parameters**: Mark with `[required]` in description

#### Parameter Properties Order
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

#### Parameter Description Standards

##### Description Format Rules
- **Start with article**: Use "The" for specific items, "A" for general items
- **End with context**: Add `[required]` for mandatory parameters, `[SERVICE_NAME]` for conditional parameters
- **Use present tense**: Describe what the parameter represents, not what it will do

##### Description Patterns
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

##### Description Context Tags
- **[required]**: Parameter is mandatory for template function
- **[SERVICE_NAME]**: Parameter only applies when specific service/feature is enabled
- **[CONDITIONAL]**: Parameter depends on other parameter values
- **No tag**: Parameter is optional

## Conditions Section Standards

### Condition Naming
- Use descriptive names starting with action verb: `Create`, `Enable`, `Has`
- Examples: `CreateSNSForAlert`, `EnableLogging`, `HasCustomDomain`

### Condition Ordering
Order conditions alphabetically by name.

## Resources Section Standards

### Resource Ordering Rules
Resources MUST be ordered in the following priority:

1. **Nested Stacks** (`AWS::Serverless::Application`, `AWS::CloudFormation::Stack`)
2. **IAM Resources** (`AWS::IAM::Role`, `AWS::IAM::Policy`, etc.)
3. **All other resources** (grouped by logical relationship, then alphabetically)

### Resource Naming Conventions

#### General Naming Rules
- Resource names should clearly indicate the resource type and purpose
- Use PascalCase for all resource names
- Follow the pattern: **ResourceTypeFor[Purpose]** for clear identification
- Avoid abbreviations unless they are widely understood

#### Specific Resource Type Naming
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

#### IAM Role Naming and Description Standards

##### Description Format
IAM Role descriptions MUST use the following format:
```yaml
Description: !Sub IAM role for [Service/Purpose] in ${AWS::StackName}
```

Examples:
- `Description: !Sub IAM role for Lambda functions in ${AWS::StackName}`
- `Description: !Sub IAM role for AWS Glue jobs in ${AWS::StackName}`
- `Description: !Sub IAM role for Step Functions state machines in ${AWS::StackName}`

##### RoleName Format
IAM Role names MUST follow this pattern to ensure uniqueness and avoid the 64-character limit:

**Standard Format (when AWS::StackName is short):**
```yaml
RoleName: !Sub ${AWS::StackName}-[Service]-${AWS::Region}
```

**LogicalName Format (when AWS::StackName is long):**
```yaml
RoleName: !Sub ${LogicalName}-[Service]-${AWS::Region}
```

**Service Name Examples:**
- `Lambda` - For Lambda execution roles
- `Glue` - For AWS Glue job roles  
- `StepFunctions` - For Step Functions execution roles
- `EC2` - For EC2 instance roles
- `CodeBuild` - For CodeBuild project roles

**Complete Examples:**
```yaml
# Lambda role
RoleName: !Sub ${LogicalName}-Lambda-${AWS::Region}

# Glue role
RoleName: !Sub ${LogicalName}-Glue-${AWS::Region}

# Step Functions role (longer service name)
RoleName: !Sub ${LogicalName}-stepfunctions-execution-role-${AWS::Region}
```

**Character Limit Considerations:**
- IAM Role names have a 64-character limit
- Use `${LogicalName}` instead of `${AWS::StackName}` when the stack name is long
- LogicalName parameter should be 3-50 characters (enforced by AllowedPattern)
- This ensures the final role name stays within the 64-character limit

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

### Resource Properties Standards

#### Tags Structure
All resources that support tags MUST use this structure:
```yaml
Tags:
  - Key: Name
    Value: !Sub resource-${LogicalName}-${Purpose}
  - Key: environment
    Value: !Ref Environment
  - Key: !Ref TagKey
    Value: !Ref TagValue
```

#### Properties Ordering
1. **UpdateReplacePolicy** (if applicable)
2. **DeletionPolicy** (if applicable)
3. **Condition** (if applicable)
4. **DependsOn** (if applicable)
5. **Type**
6. **Properties** (with alphabetical ordering of sub-properties to match AWS documentation)

## Outputs Section Standards

### Output Naming
- Use descriptive names that clearly indicate what is being output
- Use PascalCase
- Common patterns: `Id`, `Arn`, `URL`, `Name`

### Output Structure
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

### Output Ordering
Order outputs alphabetically by name.

## String Quotation Standards

### Quotation Usage Rules
Follow these rules for consistent string quotation in YAML:

#### ALWAYS Use Single Quotes
- **Metadata labels**: All label values must use single quotes
- **String values with spaces**: Any string containing spaces
- **String values with special characters**: Strings with colons, brackets, or YAML-reserved characters
- **Version numbers**: CloudFormation versions, API versions
- **Complex string patterns**: Regular expressions, JSON strings

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

#### NEVER Use Quotes (Plain Strings)
- **Simple parameter descriptions**: Single-line descriptions without special characters
- **Resource names**: Simple alphanumeric resource identifiers
- **Boolean values**: true, false (YAML native)
- **Numeric values**: Numbers without quotes
- **ENABLED/DISABLED**: CloudFormation convention values
- **AWS service names**: Simple service identifiers

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

#### Special Cases
- **Empty strings**: Use empty quotes `''` for explicit empty values
- **Strings starting with numbers**: Quote if they might be interpreted as numbers
- **Strings with colons**: Always quote strings containing colons
- **Multi-line strings**: Use YAML literal block scalars `|` or folded scalars `>`

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

### Embedded Code Quotation Rules
When embedding code in other languages (Python, JavaScript, etc.) within YAML strings, follow the target language's quotation conventions, NOT YAML conventions:

#### Python Code in YAML
- **Follow PEP 8**: Use double quotes for strings, single quotes for dictionary keys when consistent
- **String literals**: Use double quotes `"string"` for readability
- **Dictionary keys**: Use single quotes `'key'` for consistency with Python conventions
- **Mixed usage**: Acceptable within Python code as long as it follows Python best practices

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

#### JavaScript Code in YAML
```yaml
# JavaScript code embedded in YAML - follow JavaScript conventions
ScriptContent: |
  const config = {
    "apiVersion": "v1",
    "connectionType": "database"
  };
  
  console.log("Script executed successfully");
```

#### Shell Script Code in YAML
```yaml
# Shell script embedded in YAML - follow shell conventions
ScriptContent: |
  #!/bin/bash
  echo "Starting deployment"
  export API_URL="https://api.example.com"
  curl -H "Content-Type: application/json" "$API_URL"
```

## Intrinsic Functions Standards

### Function Usage Preferences
- Prefer `!Ref` over `Ref:`
- Prefer `!GetAtt` over `Fn::GetAtt:`
- Prefer `!Sub` over `Fn::Sub:`
- Use short form for all intrinsic functions

### Sub Function Usage
```yaml
# Preferred
Value: !Sub '${ResourceName}-${Environment}'

# For complex substitutions
Value: !Sub 
  - '${Resource}-${Env}'
  - Resource: !Ref ResourceName
    Env: !Ref Environment
```

## Comments and Documentation

### Comment Usage
- Use comments to separate major resource groups
- Use comments to explain complex logic
- Format: `# Comment text`

### Resource Group Comments
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

### Standard SNS Resource Format

#### SNS Topic Standard Implementation
All SNS topics MUST use the AWS Serverless Repository application format:

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

#### SNS Resource Naming Conventions
- **Alert SNS**: `SNSForAlert`
- **Deployment SNS**: `SNSForDeployment`
- **Topic Name Pattern**: `Alert-createdby-${AWS::StackName}` or `Deployment-createdby-${AWS::StackName}`

#### Required Conditions for SNS Resources
```yaml
Conditions:
  CreateSNSForAlert: !Equals [!Ref SNSForAlertArn, '']
  CreateSNSForDeployment: !Equals [!Ref SNSForDeploymentArn, '']
```

#### SNS Resource Anti-Patterns
❌ **Avoid direct AWS::SNS::Topic creation**:
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

✅ **Use AWS::Serverless::Application format** as shown in the standard implementation above.

## Outputs Section Standards

### Output Definition Rules
The Outputs section MUST follow these comprehensive rules for consistency and maintainability:

#### Mandatory Outputs (MUST include)
1. **Cross-stack referenced resources**: Any resource referenced by parent or other templates

#### Optional Outputs (MAY include)
1. **Primary service resources**: Main resources that define the template's purpose
2. **IAM roles and policies**: IAM resources for security and access management (only if needed for external reference)
3. **Resource identifiers**: IDs, ARNs, and names needed for integration
4. **Debugging resources**: Helpful for troubleshooting but not required for functionality
5. **Convenience outputs**: Nice-to-have information for operators
6. **Monitoring resources**: CloudWatch resources for observability

#### Output Naming Conventions
- **Resource identifiers**: Use the resource name + type suffix
  - Examples: `VPCId`, `SubnetIds`, `SecurityGroupId`
- **ARNs**: Use resource name + `Arn` suffix
  - Examples: `IAMRoleForLambdaArn`, `S3BucketArn`, `SecretsManagerArn`
- **Names**: Use resource name + `Name` suffix
  - Examples: `LambdaFunctionName`, `GlueConnectionName`
- **URLs and endpoints**: Use descriptive names
  - Examples: `APIGatewayURL`, `CloudFrontDistributionURL`

#### Output Structure Standards
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

#### Output Ordering Rules
Within the Outputs section, order outputs by:
1. **Type priority**: IDs → ARNs → Names → URLs → Other
2. **Alphabetical within type**: Sort alphabetically within each type group
3. **Conditional outputs**: Place at the end, also alphabetically sorted

#### Output Description Standards
- **Format**: Start with the resource type, followed by purpose
- **Examples**:
  - `Description: ID of the VPC for the analytics environment`
  - `Description: ARN of the IAM role for AWS Glue Google Analytics connector`
  - `Description: Name of the Google Analytics 4 ETL job`
  - `Description: URL of the CloudFront distribution for static content`

#### Cross-Stack Reference Requirements
For nested stacks and cross-stack references:

##### Child Template Requirements
- **MUST output all resources referenced by parent templates**
- **MAY output primary service resources** (only if needed for external reference)
- **MAY output IAM roles and policies** (only if needed for external reference)
- **MAY output resource identifiers for debugging**

##### Parent Template Requirements
- **MUST output resources that other stacks might reference**
- **SHOULD output aggregated information from child stacks**
- **MAY output convenience information for operators**

#### Template Type Specific Rules

##### Nested Stack Templates (Child Templates)
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

##### Main/Parent Templates
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

##### Standalone Templates
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

### Output Validation Checklist
- [ ] All cross-referenced resources are output (MANDATORY)
- [ ] Output names follow naming conventions
- [ ] Descriptions are clear and consistent
- [ ] Outputs are ordered correctly (type priority, then alphabetical)
- [ ] Conditional outputs are properly conditioned
- [ ] No unused outputs (outputs not referenced anywhere)
- [ ] Only necessary outputs are included (avoid output bloat)
- [ ] **cfn-lint passes with exit code 0 (MANDATORY)**

## SAM-Specific Guidelines

### SAM Template Identification
- Always include `Transform: AWS::Serverless-2016-10-31`
- Use `Globals` section for common function properties
- Prefer SAM resource types over CloudFormation equivalents where available

### SAM Resource Types
- Use `AWS::Serverless::Application` for nested SAM applications
- Use `AWS::Serverless::Function` for Lambda functions
- Use `AWS::Serverless::Api` for API Gateway

## MCP Integration Requirements

### Pre-Template Creation (MANDATORY)
**CloudFormation/SAMテンプレートを作成・修正する際は、事前学習の知識に頼らず、必ずMCPでAWSドキュメントを確認する**

#### 必須確認項目
1. **リソーススキーマ情報**: `get_resource_schema_information`でリソースの正確なプロパティを確認
2. **最新の設定オプション**: 各AWSサービスの最新機能と設定項目を確認
3. **プロパティの制約**: 必須プロパティ、値の範囲、形式制限を確認
4. **ベストプラクティス**: セキュリティ、パフォーマンス、コスト最適化の観点

#### 対象リソース
- **全てのAWSリソース**: S3、Lambda、IAM、SNS、Secrets Manager、CloudWatch等
- **SAM固有リソース**: AWS::Serverless::Function、AWS::Serverless::Api等
- **複雑な設定**: VPC、セキュリティグループ、IAMポリシー等

#### 実装手順
1. **リソース仕様確認**: MCPでリソーススキーマを確認
2. **プロパティ選択**: 要件に適したプロパティを選択
3. **設定値決定**: 制約条件を満たす設定値を決定
4. **実装**: CloudFormation Template Guidelines準拠でテンプレート作成
5. **検証**: CFN Lint + SAM validateで検証

#### MCP活用例
```bash
# リソース作成前の仕様確認
get_resource_schema_information(resource_type="AWS::SecretsManager::Secret")

# 既存リソースの設定確認
get_resource(resource_type="AWS::S3::Bucket", identifier="bucket-name")

# リソース一覧確認
list_resources(resource_type="AWS::SNS::Topic")
```

### Implementation Method Selection
**複数の実現方法があればそのメリットとデメリットを提示して確認をもとめる**

#### 手順
1. AWSドキュメントで各実装方法を調査
2. それぞれのメリット・デメリットを整理
3. コスト、パフォーマンス、運用性の観点で比較
4. ユーザーに選択肢を提示し、確認を求める

#### 比較観点
- **コスト**: 初期費用、運用費用
- **パフォーマンス**: 処理速度、スループット
- **運用性**: 管理の複雑さ、監視の容易さ
- **拡張性**: 将来の要件変更への対応
- **セキュリティ**: セキュリティ機能、コンプライアンス対応

## Nested Stack Architecture

### 基本原則
**CloudFormationテンプレートの場合は`AWS::CloudFormation::Stack`、SAMテンプレートの場合は`AWS::Serverless::Application`を使用する**

### SAMテンプレートでのネストされたスタック
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

### CloudFormationテンプレートでのネストされたスタック
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
- **AWS::Serverless::Application**: SAMテンプレート、ローカルファイル参照
- **AWS::CloudFormation::Stack**: CloudFormationテンプレート、S3 URL参照

#### 依存関係管理
1. **親スタック → 子スタック**: 子スタックは親スタックのリソースを参照可能
2. **子スタック間**: 直接的な依存関係は避ける（独立性確保）
3. **削除順序**: 子スタック → 親スタックの順で削除

#### 命名規則
- **子スタックファイル名**: `{機能名}.yaml` または `{モジュール名}.yaml`
  - ✅ 良い例: `google-analytics.yaml`, `cloudfront-logs.yaml`, `s3-access-logs.yaml`
  - ❌ 悪い例: `google-analytics-stack.yaml`, `cloudfront-logs-stack.yaml`
- **理由**: シンプルで機能が明確、冗長な`-stack`サフィックスを避ける

#### パラメータ命名規則
- **親スタック識別子**: `LogicalName`パラメータを使用（`ParentStackName`ではない）
  - ✅ 良い例: `LogicalName`
  - ❌ 悪い例: `ParentStackName`, `ParentStack`, `StackName`
- **理由**: 既存のテンプレートとの一貫性、リソース命名の論理名として使用
- **用途**: パラメータ経由での値渡し、リソース名の生成に使用

#### 親子スタック間の値渡し方法
- **パラメータ経由を優先**: `Fn::ImportValue`は使用せず、パラメータで全ての値を親スタックから子スタックに渡す
- **理由**: 複雑性回避、依存関係の明確化、デバッグの容易さ
- **実装方法**: 親スタックで子スタック呼び出し時に必要な全ての値をParametersで渡す

#### ベストプラクティス
- **モジュール化**: 機能別・データソース別に子スタックを分離
- **再利用性**: 子スタックテンプレートの汎用化
- **障害分離**: 1つの子スタックの問題が他に影響しない設計
- **パラメータ管理**: 親スタックで一元的にパラメータを管理

## Validation and Quality Assurance

### Comprehensive Pre-Deployment Checklist

#### Mandatory Validation (MUST PASS FIRST)
- [ ] **cfn-lint validation passes with exit code 0**
- [ ] **All cfn-lint warnings and errors resolved**
- [ ] **Template syntax is valid**

#### File Organization
- [ ] Service directory follows naming convention (lowercase, hyphenated)
- [ ] Template file names follow established patterns
- [ ] SAM Lambda functions use correct directory structure
- [ ] README files exist (README.md and README_JP.md)

#### Template Structure
- [ ] Template follows mandatory section ordering
- [ ] One blank line separates each major section
- [ ] Description follows exact format pattern
- [ ] Transform included for SAM templates
- [ ] Globals section present for SAM templates (if needed)

#### Metadata Section
- [ ] Parameter groups follow correct ordering
- [ ] Labels use exact naming conventions
- [ ] All labels end with 'Configuration'
- [ ] Standard labels used for Notification and Tag sections

#### Parameters Section
- [ ] Parameters ordered alphabetically
- [ ] Parameter properties in correct order
- [ ] Descriptions follow format standards
- [ ] Context tags used appropriately ([required], [SERVICE_NAME])
- [ ] Standard parameters use exact definitions
- [ ] Boolean parameters use ENABLED/DISABLED values
- [ ] AllowedPattern validates input correctly (e.g., S3 naming rules)

#### Resources Section
- [ ] Resources ordered by type priority (Nested → IAM → Others)
- [ ] Resource names follow naming conventions
- [ ] All taggable resources have consistent tag structure
- [ ] Comments separate major resource groups
- [ ] Intrinsic functions use short form

#### Outputs Section
- [ ] Outputs ordered alphabetically
- [ ] Output names clearly indicate content
- [ ] All outputs have descriptions
- [ ] Only cross-stack referenced resources are output (mandatory)
- [ ] No unnecessary outputs (avoid output bloat)

### Common Anti-Patterns to Avoid

#### File Organization Anti-Patterns
- ❌ Using CamelCase or underscores in directory names
- ❌ Inconsistent template file naming
- ❌ Lambda function files not named `lambda_function.py`
- ❌ Missing README files

#### Template Structure Anti-Patterns
- ❌ Incorrect section ordering
- ❌ Missing blank lines between sections
- ❌ Wrong Description format
- ❌ Missing Transform for SAM templates

#### Parameter Anti-Patterns
- ❌ Parameters not in alphabetical order
- ❌ Inconsistent parameter descriptions
- ❌ Missing [required] tags for mandatory parameters
- ❌ Using true/false instead of ENABLED/DISABLED
- ❌ Generic parameter names

#### Resource Anti-Patterns
- ❌ Generic resource names (e.g., `Resource1`, `MyResource`)
- ❌ Incorrect resource ordering
- ❌ Inconsistent tag structure
- ❌ Missing resource type prefixes (RouteTable vs RouteTablePublic)
- ❌ Using long form intrinsic functions

#### Label and Description Anti-Patterns
- ❌ Labels not ending with 'Configuration'
- ❌ Inconsistent label capitalization
- ❌ Missing single quotes around labels
- ❌ Descriptions not starting with articles (The/A)
- ❌ Missing context tags in descriptions

### Quality Assurance Tools

#### Mandatory Validation (REQUIRED)
All CloudFormation templates MUST pass cfn-lint validation before deployment or commit:

```bash
# CloudFormation Linter (MANDATORY)
cfn-lint template.yaml

# Must return exit code 0 (no errors or warnings)
# Fix all warnings and errors before proceeding
```

#### Additional Validation (RECOMMENDED)
```bash
# Template syntax validation
aws cloudformation validate-template --template-body file://template.yaml

# SAM template validation (for SAM templates)
sam validate
```

#### cfn-lint Requirements
- **Installation**: Install cfn-lint via pip: `pip install cfn-lint`
- **Execution**: Run cfn-lint on every template modification
- **Zero tolerance**: All warnings and errors must be resolved
- **Exit code**: Must return 0 (success) before template is considered complete

#### Manual Review Checklist
1. **cfn-lint validation (MANDATORY FIRST STEP)**
2. **File naming consistency check**
3. **Template structure verification**
4. **Parameter ordering validation**
5. **Resource naming convention check**
6. **Description format verification**
7. **Tag structure consistency**

## File Organization and Directory Structure

### Project Directory Structure
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

### Service Directory Naming
- **Format**: Lowercase with hyphens for word separation
- **Examples**: `security`, `network`, `static-website-hosting`, `security-config-rules`
- **Avoid**: CamelCase, underscores, or spaces

### Template File Naming Conventions

#### Main Templates
- **Service main template**: `template.yaml` (aggregates multiple services)
- **Individual service template**: `[aws-service-name].yaml`
  - Examples: `cloudfront.yaml`, `iam.yaml`, `fsx.yaml`
- **Feature-specific template**: `[feature-name].yaml`
  - Examples: `autoscaling.yaml`, `realtime-dashboard.yaml`

#### SAM Application Templates
- **Main SAM template**: `template.yaml`
- **Feature templates**: `[feature].yaml`
  - Examples: `events.yaml`, `sns.yaml`

#### Specialized Templates
- **Hyphenated names**: Use hyphens for multi-word templates
  - Examples: `centralized-logging.yaml`, `synthetics-heartbeat.yaml`
- **Service combinations**: `[service1]-[service2].yaml`
  - Examples: `ecs-codedeploy.yaml`, `application-elb.yaml`

### Lambda Function Directory Structure (SAM)
```
sam-app/
├── template.yaml                      # Main SAM template
├── [function-name]/                   # One directory per function
│   ├── lambda_function.py             # Always this exact name
│   └── requirements.txt               # Python dependencies
└── [additional-templates].yaml        # Supporting templates
```

#### Lambda Function Naming
- **Directory names**: Use descriptive, hyphenated names
  - Examples: `sendNotificationToSlack`, `analyzeUnauthorizedApiCalls`
- **File names**: Always use `lambda_function.py` (never vary this)

### Template Size and Complexity Guidelines

#### Template Organization Rules
- **Single responsibility**: One template per major AWS service or feature
- **Size limits**: Maximum 500 lines per template (recommended)
- **Complexity limits**: Maximum 50 parameters per template
- **Nested stacks**: Use for complex multi-service deployments

#### Template Splitting Guidelines
Split templates when:
- Template exceeds 500 lines
- More than 50 parameters required
- Multiple unrelated AWS services in one template
- Different deployment lifecycles needed

#### File Naming for Split Templates
```yaml
# Original large template
template.yaml

# Split into:
vpc.yaml              # VPC and networking
ec2.yaml              # EC2 instances and related
elb.yaml              # Load balancers
monitoring.yaml       # CloudWatch and alarms
```

## Version Control and Maintenance

### Template Versioning
- Update template descriptions when making significant changes
- Use semantic versioning for nested stack applications
- Document breaking changes in commit messages

### Maintenance Guidelines
- Review templates quarterly for AWS service updates
- Update parameter defaults as needed
- Ensure compatibility with latest CloudFormation features
- Test templates in development environment before production deployment