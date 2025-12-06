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
- **ブランチ戦略**: 機能開発は feature ブランチで行い、main ブランチへの直接コミットは避ける

## ブランチ戦略（必須）

### 基本原則

**重要**: main ブランチへの直接コミットは避け、必ず feature ブランチを使用する

### ブランチの種類

#### 1. main ブランチ
- **用途**: 本番環境にデプロイ可能な安定版コード
- **保護**: 直接コミット禁止
- **マージ**: feature ブランチからのみマージ可能
- **タグ**: リリース時にバージョンタグを付与

#### 2. feature ブランチ
- **命名規則**: `feature/<機能名>` または `fix/<修正内容>`
- **用途**: 新機能開発、バグ修正、リファクタリング
- **作成元**: main ブランチから分岐
- **マージ先**: main ブランチへマージ

### ワークフロー

#### 機能開発の流れ
```bash
# 1. main ブランチを最新化
git checkout main
git pull origin main

# 2. feature ブランチを作成
git checkout -b feature/remove-daemon-functionality

# 3. 開発作業
# - コードの変更
# - テストの実行
# - コミット

git add .
git commit -m "Remove daemon functionality"

# 4. リモートにプッシュ
git push origin feature/remove-daemon-functionality

# 5. main ブランチへマージ
git checkout main
git merge feature/remove-daemon-functionality

# 6. リモートの main を更新
git push origin main

# 7. feature ブランチを削除（オプション）
git branch -d feature/remove-daemon-functionality
git push origin --delete feature/remove-daemon-functionality
```

#### バグ修正の流れ
```bash
# 1. fix ブランチを作成
git checkout -b fix/github-actions-pyobjc-issue

# 2. 修正作業
git add .
git commit -m "Fix GitHub Actions workflow to remove PyObjC resources"

# 3. main へマージ
git checkout main
git merge fix/github-actions-pyobjc-issue
git push origin main
```

#### ホットフィックスの流れ（緊急修正）
```bash
# 1. hotfix ブランチを作成
git checkout -b hotfix/v1.2.1-integration-test-fix

# 2. 最小限の修正
git add .
git commit -m "Fix integration test imports"

# 3. main へマージ
git checkout main
git merge hotfix/v1.2.1-integration-test-fix
git push origin main

# 4. 緊急リリースタグ
git tag v1.2.1
git push origin v1.2.1
```

### リリースプロセス

#### 通常リリース
```bash
# 1. main ブランチが安定していることを確認
git checkout main
git pull origin main

# 2. バージョンを更新（__init__.py, CHANGELOG.md など）
# 3. コミット
git add .
git commit -m "Bump version to 1.3.0"
git push origin main

# 4. タグを作成
git tag v1.3.0
git push origin v1.3.0
```

#### プレリリース（ベータ版）
```bash
# 1. beta ブランチを作成
git checkout -b beta/v1.3.0-beta.1

# 2. ベータ版としてタグ付け
git tag v1.3.0-beta.1
git push origin v1.3.0-beta.1

# 3. テスト後、問題なければ main へマージ
git checkout main
git merge beta/v1.3.0-beta.1
git tag v1.3.0
git push origin v1.3.0
```

### 避けるべきパターン

#### ❌ 悪い例
```bash
# main ブランチで直接作業（今回のケース）
git checkout main
# 変更作業...
git add .
git commit -m "Fix something"
git push origin main
```

#### ✅ 良い例
```bash
# feature ブランチで作業
git checkout -b feature/fix-something
# 変更作業...
git add .
git commit -m "Fix something"
git push origin feature/fix-something

# main へマージ
git checkout main
git merge feature/fix-something
git push origin main
```

### ブランチ管理のベストプラクティス

1. **小さく頻繁にコミット**: 大きな変更を避け、小さな単位でコミット
2. **明確なコミットメッセージ**: 何を変更したか、なぜ変更したかを明記
3. **定期的なマージ**: feature ブランチが長期化しないよう、定期的に main へマージ
4. **コンフリクトの早期解決**: main の変更を定期的に feature ブランチへマージ
5. **ブランチの削除**: マージ後は不要な feature ブランチを削除

### 今後の対応

今回は main ブランチで直接作業してしまいましたが、今後は以下のように対応します：

1. **新機能開発**: `feature/` ブランチを作成
2. **バグ修正**: `fix/` ブランチを作成
3. **緊急修正**: `hotfix/` ブランチを作成
4. **テスト完了後**: main ブランチへマージ
5. **リリース**: main ブランチでタグを作成

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
1. **AWS CloudFormation Guidelines確認**: ステアリングファイルのaws-cloudformation-guidelines.mdを参照
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

## 実装変更時の仕様書・ドキュメント整合性確認（必須）

### 実装変更後の必須確認プロセス

**重要**: 試行錯誤やエラー解消のために実装を変更した場合、必ず以下の確認を行う

**絶対ルール**: コード実装を変更したら、その変更が完了した直後に必ず README.md を更新する。実装とドキュメントの更新は一体の作業として扱う。

#### 1. 仕様書への影響確認（必須）
- **要件定義書 (requirements.md)**: 変更が要件の受け入れ基準に影響しないか確認
- **設計書 (design.md)**: アーキテクチャや設計方針との整合性を確認
- **実装計画書 (tasks.md)**: タスクの前提条件や依存関係に影響しないか確認

#### 2. ドキュメントへの影響確認（必須・即時実行）

**README.md 更新の必須タイミング**:
- ✅ **実装変更直後**: コード変更が完了したら、その場で README.md を更新
- ✅ **機能追加時**: 新機能の使用方法を README.md に追加
- ✅ **機能削除時**: 削除された機能の記載を README.md から削除
- ✅ **UI/メニュー変更時**: メニュー構成や操作方法の変更を README.md に反映
- ✅ **コマンドオプション変更時**: 使用例とオプション説明を README.md で更新

**更新対象ドキュメント**:
- **README.md**: インストール手順、使用方法、設定例を即座に更新（必須）
- **CHANGELOG.md**: 変更内容の記録が必要か確認
- **API仕様書**: インターフェース変更がある場合は更新
- **設定ファイル例**: デフォルト値や設定項目の変更を反映

#### 3. 外部システムへの影響確認
- **GitHub Actions**: ワークフローファイルの更新が必要か確認
- **Homebrew Formula**: 依存関係や設定の変更を反映
- **Docker設定**: コンテナ設定やビルドプロセスへの影響を確認

#### 4. 確認チェックリスト

##### 実装変更時の必須チェック項目（実装直後に実行）
- [ ] **README.md を即座に更新**（最優先・必須）
- [ ] 要件定義書の受け入れ基準との整合性を確認
- [ ] 設計書のアーキテクチャ図・設計方針との整合性を確認
- [ ] CHANGELOG.md に変更内容を記録
- [ ] 設定ファイル例やデフォルト値を更新
- [ ] GitHub Actions 等の自動化スクリプトを更新
- [ ] 依存関係の変更をパッケージ管理ファイルに反映

##### README.md 更新が必須となる変更パターン
- **UI/メニュー変更**: メニュー項目の追加・削除・変更 → README.md のメニュー構成図を更新
- **機能追加**: 新機能の実装 → README.md に使用方法を追加
- **機能削除**: 機能の削除 → README.md から該当機能の記載を削除
- **コマンドオプション変更**: オプションの追加・削除・変更 → README.md の使用例を更新
- **設定ファイル形式の変更**: 例、サンプル、ドキュメントをすべて更新
- **依存関係の追加・削除**: requirements.txt、Formula、インストール手順を更新
- **デフォルト動作の変更**: 要件定義、設計書、ユーザーガイドを確認・更新
- **エラーメッセージの変更**: トラブルシューティングガイドを更新
- **通知機能の変更**: 通知の追加・削除・変更 → README.md の機能説明を更新

#### 5. 更新作業の実行順序（厳守）
1. **コード実装**: 機能の実装・変更
2. **README.md の即時更新**: 実装変更の直後に必ず更新（最優先）
3. **仕様書の更新**: requirements.md → design.md → tasks.md の順で更新
4. **その他ドキュメントの更新**: CHANGELOG.md → その他ドキュメント
5. **外部システムの更新**: GitHub Actions → Homebrew Formula → その他設定
6. **整合性の最終確認**: 全体を通して矛盾がないか確認

**重要**: README.md の更新は「後でまとめて」ではなく、「実装変更の直後に即座に」行う

#### 6. 更新漏れ防止のための原則
- **変更の影響範囲を事前に特定**: 変更前に影響を受ける可能性のあるファイルをリストアップ
- **段階的な更新**: 一度にすべてを変更せず、段階的に更新して整合性を確認
- **レビューの実施**: 可能であれば第三者による整合性レビューを実施
- **テストの実行**: 更新後は必ず動作テストを実行して問題がないことを確認

### 実装変更の記録

#### 変更ログの記録項目
- **変更理由**: なぜ変更が必要だったか（エラー解消、機能改善等）
- **変更内容**: 具体的に何を変更したか
- **影響範囲**: どのファイル・システムに影響があったか
- **更新したドキュメント**: 整合性確保のために更新したファイル一覧

#### 例：変更記録テンプレート
```markdown
## 実装変更記録

### 変更理由
- macOS セキュリティ制限により Homebrew post_install での LaunchAgent 作成が失敗

### 変更内容
- Homebrew Formula から自動有効化処理を削除
- ユーザー手動有効化方式に変更

### 影響範囲
- homebrew-display-layout-manager/Formula/display-layout-manager.rb
- .github/workflows/release.yml

### 更新したドキュメント
- .kiro/specs/display-layout-manager/requirements.md (要件16)
- display-layout-manager/README.md (常駐機能セクション)
```

## コミット前品質保証メカニズム（必須）

### コミット前の必須検証プロセス

**重要**: コミット・プッシュ前に以下の検証を必ず実行し、問題を事前に発見・修正する

#### 1. ローカル統合テスト（必須）

##### Python プロジェクトの場合
```bash
# 1. 依存関係の確認
pip install -e .
python -c "import [main_module]; print('✓ Import successful')"

# 2. 基本機能テスト
[main_command] --version
[main_command] --help
[main_command] --run-tests

# 3. 重要機能の動作確認
[main_command] [key_functionality]
```

##### Homebrew Formula の場合
```bash
# 1. Formula 構文チェック
brew audit --strict [formula_name]

# 2. ローカルビルドテスト
brew install --build-from-source [formula_name]

# 3. インストール後テスト
[installed_command] --version
[installed_command] [basic_test]

# 4. アンインストールテスト
brew uninstall [formula_name]
```

#### 2. 環境別テスト（推奨）

##### 仮想環境でのテスト
```bash
# Python 仮想環境でのクリーンテスト
python -m venv test_env
source test_env/bin/activate
pip install .
[test_commands]
deactivate
rm -rf test_env
```

##### Docker でのテスト（可能な場合）
```bash
# クリーンな macOS 環境でのテスト
docker run --rm -v $(pwd):/workspace macos-test:latest \
  /bin/bash -c "cd /workspace && [test_commands]"
```

#### 3. 依存関係検証（必須）

##### 依存関係の完全性確認
```bash
# Python: requirements.txt と pyproject.toml の整合性
pip-compile --dry-run pyproject.toml
pip check

# Homebrew: リソースの存在確認
curl -I [resource_url]  # 各リソース URL の存在確認
```

##### 外部サービス依存の確認
```bash
# GitHub Actions ワークフローの構文チェック
act --dry-run  # GitHub Actions のローカル実行ツール

# または GitHub CLI での確認
gh workflow view [workflow_name]
```

#### 4. 設定ファイル・ドキュメント検証（必須）

##### 設定ファイルの妥当性確認
```bash
# JSON 設定ファイルの構文チェック
python -m json.tool config.json

# YAML 設定ファイルの構文チェック
python -c "import yaml; yaml.safe_load(open('config.yaml'))"
```

##### ドキュメントの整合性確認
```bash
# README の手順を実際に実行
# インストール手順を一つずつ実行して確認

# リンクの有効性確認（可能な場合）
markdown-link-check README.md
```

### 5. 段階的リリース戦略

#### Pre-release での検証
```bash
# 1. プレリリース版の作成
git tag v1.x.x-beta.1
git push origin v1.x.x-beta.1

# 2. プレリリース版での検証
brew install [tap]/[formula]@beta
[comprehensive_tests]

# 3. 問題なければ正式リリース
git tag v1.x.x
git push origin v1.x.x
```

#### ブランチ戦略
```bash
# 1. 機能ブランチでの開発
git checkout -b feature/fix-pyobjc-issue

# 2. 機能ブランチでの完全テスト
[all_verification_steps]

# 3. main ブランチへのマージ
git checkout main
git merge feature/fix-pyobjc-issue

# 4. main ブランチでの最終確認
[final_verification_steps]

# 5. リリース
git tag v1.x.x
```

### 6. 自動化による品質保証

#### Pre-commit フックの設定
```bash
# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: local-tests
        name: Local Integration Tests
        entry: ./scripts/pre-commit-tests.sh
        language: script
        pass_filenames: false
```

#### CI/CD での段階的検証
```yaml
# GitHub Actions での多段階テスト
jobs:
  test:
    runs-on: macos-latest
    steps:
      - name: Unit Tests
      - name: Integration Tests
      - name: Homebrew Formula Test
      - name: End-to-End Test
  
  pre-release:
    needs: test
    if: contains(github.ref, 'beta')
    steps:
      - name: Create Pre-release
  
  release:
    needs: test
    if: startsWith(github.ref, 'refs/tags/v')
    steps:
      - name: Create Release
```

### 7. 問題発生時の迅速対応メカニズム

#### ロールバック戦略
```bash
# 1. 問題のあるリリースの特定
git log --oneline

# 2. 前のバージョンへのロールバック
git revert [problematic_commit]
git tag v1.x.x-hotfix.1

# 3. Homebrew Formula の緊急修正
# Formula を前のバージョンに戻す
```

#### ホットフィックス手順
```bash
# 1. ホットフィックスブランチの作成
git checkout -b hotfix/v1.x.x-fix

# 2. 最小限の修正
[minimal_fix]

# 3. 緊急テスト
[critical_tests_only]

# 4. 緊急リリース
git tag v1.x.x-hotfix.1
```

### 8. 検証チェックリスト

#### コミット前必須チェック項目
- [ ] ローカルでの基本機能テスト完了
- [ ] 依存関係の完全性確認完了
- [ ] 設定ファイルの構文チェック完了
- [ ] ドキュメントの整合性確認完了
- [ ] 重要な使用ケースのテスト完了

#### リリース前必須チェック項目
- [ ] クリーン環境でのインストールテスト完了
- [ ] 全機能の動作確認完了
- [ ] アンインストールテスト完了
- [ ] ドキュメントの手順確認完了
- [ ] GitHub Actions の動作確認完了

#### 緊急時対応チェック項目
- [ ] 問題の影響範囲特定完了
- [ ] ロールバック手順確認完了
- [ ] ユーザーへの通知準備完了
- [ ] 修正版のテスト完了

### 9. 継続的改善

#### 問題パターンの記録と対策
```markdown
## 品質問題記録

### 問題: PyObjC 依存関係不足
- **発生日**: 2025-12-06
- **原因**: Homebrew Formula にリソース未記載
- **対策**: Formula 作成時の依存関係チェックリスト追加
- **予防策**: ローカル Homebrew インストールテストの必須化

### 問題: GitHub Actions での自動削除
- **発生日**: 2025-12-06
- **原因**: ワークフローテンプレートの不備
- **対策**: ワークフローファイルの構文チェック追加
- **予防策**: GitHub Actions のローカル実行テスト導入
```

## テスト戦略とベストプラクティス（必須）

### テストの基本原則

**重要**: テストは実装と同時に進行し、品質を継続的に確保する

#### テストの目的
1. **バグの早期発見**: 実装段階で問題を検出
2. **リファクタリングの安全性**: 変更時の影響を検証
3. **仕様の文書化**: テストコードが仕様の証明となる
4. **保守性の向上**: 将来の変更時の安全網

### テストの種類と実施タイミング

#### 1. 単体テスト（Unit Tests）
**タイミング**: 各コンポーネント実装直後

**対象**:
- 個別のクラス・関数
- ビジネスロジック
- データ変換処理
- バリデーション処理

**実装方針**:
- 1つのコンポーネントにつき1つのテストファイル
- モックを活用して外部依存を排除
- エッジケースを含む複数のテストケース
- テストケース名は「何をテストしているか」を明確に

**例**:
```python
# test_display_manager.py
def test_extract_screen_ids_success():
    """正常なdisplayplacer出力からScreen IDを抽出できる"""
    
def test_extract_screen_ids_empty_output():
    """空の出力を適切に処理できる"""
    
def test_extract_screen_ids_invalid_format():
    """不正なフォーマットでエラーを返す"""
```

#### 2. 統合テスト（Integration Tests）
**タイミング**: 複数コンポーネントの連携実装後

**対象**:
- コンポーネント間の連携
- 外部コマンド実行
- ファイルI/O
- 設定ファイル読み込み

**実装方針**:
- 実際の環境に近い状態でテスト
- 一時ファイル・ディレクトリを使用
- テスト後のクリーンアップを確実に実施
- エラーケースも含めて検証

**例**:
```python
# test_menubar_integration.py
def test_apply_layout_end_to_end():
    """レイアウト適用の全フローが正常に動作する"""
    # 設定ファイル作成 → ディスプレイ検出 → パターンマッチ → コマンド実行
```

#### 3. エンドツーエンドテスト（E2E Tests）
**タイミング**: 主要機能の実装完了後

**対象**:
- ユーザーの実際の使用フロー
- CLIコマンドの実行
- メニューバーアプリの操作

**実装方針**:
- 実際のユーザー操作をシミュレート
- 本番環境に近い設定で実行
- 成功パスとエラーパスの両方をテスト

### テスト実装の順序（推奨）

#### Phase 1: コア機能の単体テスト
1. **基盤コンポーネント**: 依存関係が少ないコンポーネントから
2. **ビジネスロジック**: 主要な処理ロジック
3. **ユーティリティ**: 共通処理

#### Phase 2: 統合テスト
1. **コンポーネント連携**: 2-3個のコンポーネントの組み合わせ
2. **外部依存**: ファイルシステム、外部コマンド
3. **エラーハンドリング**: 異常系の処理

#### Phase 3: E2Eテスト
1. **主要フロー**: ユーザーが最も使う機能
2. **エッジケース**: 特殊な状況での動作
3. **エラーリカバリ**: エラー発生時の復旧

### テストカバレッジの目標

#### カバレッジ基準
- **最低ライン**: 全体で30%以上
- **推奨ライン**: 全体で50%以上
- **理想ライン**: コアロジックで70%以上

#### 優先順位
1. **高優先度（70%以上目標）**:
   - ビジネスロジック
   - データ変換処理
   - エラーハンドリング

2. **中優先度（50%以上目標）**:
   - ユーティリティ関数
   - 設定管理
   - ファイルI/O

3. **低優先度（30%以上目標）**:
   - UI表示ロジック
   - ログ出力
   - 初期化処理

#### カバレッジ測定
```bash
# Python の場合
pip install coverage
coverage run -m pytest
coverage report
coverage html  # HTMLレポート生成
```

### CI/CDでのテスト自動化（必須）

#### GitHub Actions 設定例
```yaml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: macos-latest
    strategy:
      matrix:
        python-version: ['3.9', '3.10', '3.11']
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      
      - name: Install dependencies
        run: |
          pip install -e .
          pip install pytest coverage
      
      - name: Run unit tests
        run: pytest tests/test_*.py -v
      
      - name: Run integration tests
        run: pytest tests/integration_*.py -v
      
      - name: Generate coverage report
        run: |
          coverage run -m pytest
          coverage report
          coverage html
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

#### 必須テスト項目
- [ ] 単体テスト実行
- [ ] 統合テスト実行
- [ ] カバレッジレポート生成
- [ ] Lintチェック（flake8、black、isort等）
- [ ] 型チェック（mypy等、該当する場合）

### テスト実装のベストプラクティス

#### 1. テストの独立性
```python
# ✅ 良い例：各テストが独立
def test_create_config():
    config = create_temp_config()
    # テスト実行
    cleanup_temp_config(config)

# ❌ 悪い例：テスト間で状態を共有
global_config = None
def test_create_config():
    global global_config
    global_config = create_config()
```

#### 2. テストデータの管理
```python
# ✅ 良い例：テストごとに新しいデータ
@pytest.fixture
def sample_display_output():
    return """
    Persistent screen id: 1234-5678
    Resolution: 1920x1080
    """

# ❌ 悪い例：ハードコードされた外部ファイル依存
def test_parse():
    with open('/tmp/test_data.txt') as f:
        data = f.read()
```

#### 3. モックの適切な使用
```python
# ✅ 良い例：外部依存をモック化
@patch('subprocess.run')
def test_execute_command(mock_run):
    mock_run.return_value = MagicMock(returncode=0, stdout="success")
    result = execute_command("test")
    assert result.success

# ❌ 悪い例：実際のコマンドを実行
def test_execute_command():
    result = execute_command("rm -rf /")  # 危険！
```

#### 4. テストの可読性
```python
# ✅ 良い例：AAA パターン（Arrange-Act-Assert）
def test_pattern_matching():
    # Arrange: テストデータの準備
    config = {"patterns": [{"name": "dual", "screen_ids": ["123", "456"]}]}
    detected_ids = ["123", "456"]
    
    # Act: テスト対象の実行
    result = match_pattern(config, detected_ids)
    
    # Assert: 結果の検証
    assert result.matched
    assert result.pattern_name == "dual"

# ❌ 悪い例：準備・実行・検証が混在
def test_pattern_matching():
    result = match_pattern({"patterns": [{"name": "dual", "screen_ids": ["123", "456"]}]}, ["123", "456"])
    assert result.matched and result.pattern_name == "dual"
```

### テスト完了の判断基準

#### 必須条件
- [ ] 全ての単体テストが成功
- [ ] 全ての統合テストが成功
- [ ] カバレッジが最低基準（30%）を超えている
- [ ] CI/CDパイプラインでテストが自動実行される
- [ ] 主要な使用ケースがE2Eテストでカバーされている

#### 推奨条件
- [ ] カバレッジが推奨基準（50%）を超えている
- [ ] エラーケースのテストが充実している
- [ ] エッジケースのテストが含まれている
- [ ] テストドキュメントが整備されている
- [ ] テスト実行時間が適切（全体で5分以内が目安）

#### 理想条件
- [ ] コアロジックのカバレッジが70%を超えている
- [ ] 複数のPythonバージョンでテストが成功
- [ ] パフォーマンステストが実施されている
- [ ] セキュリティテストが実施されている

### テスト保守のガイドライン

#### テストコードの品質
- テストコードも本番コードと同じ品質基準を適用
- テストコードのリファクタリングも定期的に実施
- 重複するテストコードは共通化
- テストユーティリティ・フィクスチャの活用

#### テストの更新タイミング
- **機能追加時**: 新機能のテストを追加
- **バグ修正時**: バグを再現するテストを追加してから修正
- **リファクタリング時**: テストが失敗しないことを確認
- **仕様変更時**: 関連するテストを更新

#### テスト失敗時の対応
1. **原因特定**: どのテストが、なぜ失敗したか
2. **影響範囲確認**: 他のテストへの影響
3. **修正**: コードまたはテストの修正
4. **再実行**: 全テストの再実行で確認
5. **記録**: 失敗原因と対策を記録

### プロジェクトタイプ別のテスト戦略

#### CLIアプリケーション
- コマンドライン引数のパース
- 標準出力・標準エラー出力の検証
- 終了コードの確認
- 外部コマンド実行のモック化

#### GUIアプリケーション（メニューバーアプリ等）
- UI要素の存在確認
- ユーザーアクション（クリック等）のシミュレート
- 状態変化の検証
- バックグラウンド処理のテスト

#### サーバーレスアプリケーション（Lambda等）
- イベントハンドラのテスト
- 環境変数の設定
- 外部サービス（S3、DynamoDB等）のモック化
- タイムアウト・メモリ制限の考慮

#### ライブラリ・SDK
- 公開APIの全メソッドをテスト
- 互換性テスト（複数バージョン）
- ドキュメントの例がテストとして機能
- エラーメッセージの明確性

### テスト環境の管理

#### ローカル開発環境
```bash
# 仮想環境の作成
python -m venv venv
source venv/bin/activate

# 開発依存関係のインストール
pip install -e ".[dev]"

# テスト実行
pytest -v
```

#### CI/CD環境
- クリーンな環境で毎回実行
- 複数のOS・バージョンでテスト
- 並列実行でテスト時間を短縮
- テスト結果の可視化

#### テストデータの管理
- テストデータはリポジトリに含める
- 機密情報は含めない
- 大きなファイルは外部ストレージ
- テストデータ生成スクリプトの提供

## 品質保証
- CFN Lintでexit code 0必須
- テンプレート構造の統一
- 命名規則の遵守
- **AWS CloudFormation Guidelines完全準拠**
- **実装変更時の仕様書・ドキュメント整合性確認必須**
- **コミット前品質保証メカニズム必須実行**
- **テスト戦略に基づく包括的なテスト実施必須**