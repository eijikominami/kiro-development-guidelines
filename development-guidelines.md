# 開発ガイドライン

## 基本方針

### 言語とドキュメント
- **ドキュメントの記載とチャットの会話は日本語**
- コード内のコメントは英語でも可
- 変数名、関数名は英語を使用

### 開発プロセス
- **Infrastructure as Code**: CloudFormation/SAMテンプレートでインフラ管理
- **CI/CD**: AWS で作る場合は AWS CodePipeline、それ以外は GitHub Actions 等でも可
- **監視・ログ**: CloudWatch、X-Rayによる可観測性確保
- **ブランチ戦略**: 詳細は「ブランチ戦略（必須）」セクションを参照

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
- **命名規則**: `feature/<機能名>`
- **用途**: 新機能開発、リファクタリング
- **作成元**: main ブランチから分岐
- **マージ先**: main ブランチへマージ

#### 3. fix ブランチ
- **命名規則**: `fix/<修正内容>`
- **用途**: バグ修正
- **作成元**: main ブランチから分岐
- **マージ先**: main ブランチへマージ

#### 4. hotfix ブランチ
- **命名規則**: `hotfix/<バージョン>-<修正内容>`
- **用途**: 本番環境の緊急修正
- **作成元**: main ブランチから分岐
- **マージ先**: main ブランチへマージ後、即座にタグ作成

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
# main ブランチで直接作業
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

### main ブランチ保護の重要性

**絶対ルール**: main ブランチへの直接コミットは禁止

#### main ブランチで作業してしまった場合の対処法

もし誤って main ブランチで作業してしまった場合：

```bash
# 1. 現在の変更を確認
git status

# 2. まだコミットしていない場合
git stash                              # 変更を一時保存
git checkout -b feature/fix-something  # feature ブランチ作成
git stash pop                          # 変更を復元
git add .
git commit -m "Fix something"

# 3. すでにコミットしてしまった場合（まだプッシュしていない）
git log --oneline -5                   # コミット履歴確認
git branch feature/fix-something       # 現在位置で feature ブランチ作成
git reset --hard HEAD~1                # main を1つ前に戻す
git checkout feature/fix-something     # feature ブランチに移動

# 4. すでにプッシュしてしまった場合
# → 次回から必ず feature ブランチを使用
# → 今回のコミットは履歴に残るが、以降は正しい手順を守る
```

#### 作業開始前の必須チェック

```bash
# 現在のブランチを確認
git branch

# main ブランチにいる場合は即座に feature ブランチを作成
git checkout -b feature/your-feature-name
```

### ブランチ管理のベストプラクティス

#### 基本ルール
1. **main ブランチでの直接作業禁止**: 必ず適切なブランチ（feature/fix/hotfix）を作成してから作業を開始
2. **テスト完了後にマージ**: 全てのテストが成功してから main ブランチへマージ
3. **リリース時のタグ作成**: main ブランチでバージョンタグを作成してリリース

#### 日常的な運用
1. **小さく頻繁にコミット**: 大きな変更を避け、小さな単位でコミット
2. **明確なコミットメッセージ**: 何を変更したか、なぜ変更したかを明記
3. **定期的なマージ**: feature ブランチが長期化しないよう、定期的に main へマージ
4. **コンフリクトの早期解決**: main の変更を定期的に feature ブランチへマージ
5. **ブランチの削除**: マージ後は不要な feature ブランチを削除

## 技術スタック

### コア技術
- **Infrastructure as Code**: AWS CloudFormation/SAM（詳細は `aws-cloudformation-guidelines.md` 参照）
- **ランタイム**: プロジェクトに応じて選択（Python、Node.js、Java、Go 等）
- **設定管理**: YAML 設定ファイル
- **バージョン管理**: Git と pre-commit フック

### ビルドシステム
- **Pre-commit**: コード品質と Lint の自動化
- **SAM CLI**: サーバーレスアプリケーションのビルドとデプロイ（詳細は `aws-cloudformation-guidelines.md` 参照）

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
**詳細は `aws-cloudformation-guidelines.md` を参照**

1. **AWS CloudFormation Guidelines確認**: `aws-cloudformation-guidelines.md` を参照
2. **MCP でAWSドキュメント確認**: リソーススキーマ情報を確認（`aws-architecture.md` 参照）
3. **テンプレート構造遵守**: 必須セクション順序、命名規則、標準パラメータ
4. **CFN Lint検証**: exit code 0を確保（必須）

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

#### 4. 更新作業の実行順序と確認（厳守）

##### 実行順序
1. **コード実装**: 機能の実装・変更
2. **README.md の即時更新**: 実装変更の直後に必ず更新（最優先・必須）
3. **仕様書の更新**: requirements.md → design.md → tasks.md の順で更新
4. **その他ドキュメントの更新**: CHANGELOG.md → その他ドキュメント
5. **外部システムの更新**: GitHub Actions → Homebrew Formula → その他設定
6. **整合性の最終確認**: 全体を通して矛盾がないか確認

**重要**: README.md の更新は「後でまとめて」ではなく、「実装変更の直後に即座に」行う

##### 実装変更時の必須チェック項目
- [ ] **README.md を即座に更新**（最優先・必須）
- [ ] 要件定義書の受け入れ基準との整合性を確認
- [ ] 設計書のアーキテクチャ図・設計方針との整合性を確認
- [ ] CHANGELOG.md に変更内容を記録
- [ ] 設定ファイル例やデフォルト値を更新
- [ ] GitHub Actions 等の自動化スクリプトを更新
- [ ] 依存関係の変更をパッケージ管理ファイルに反映

##### README.md 更新が必須となる変更パターン
- **UI/メニュー変更**: メニュー項目の追加・削除・変更 → メニュー構成図を更新
- **機能追加**: 新機能の実装 → 使用方法を追加
- **機能削除**: 機能の削除 → 該当機能の記載を削除
- **コマンドオプション変更**: オプションの追加・削除・変更 → 使用例を更新
- **設定ファイル形式の変更**: 例、サンプル、ドキュメントをすべて更新
- **依存関係の追加・削除**: requirements.txt、Formula、インストール手順を更新
- **デフォルト動作の変更**: 要件定義、設計書、ユーザーガイドを確認・更新
- **エラーメッセージの変更**: トラブルシューティングガイドを更新
- **通知機能の変更**: 通知の追加・削除・変更 → 機能説明を更新

#### 5. 更新漏れ防止のための原則
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

## 品質保証

**品質保証の詳細は `quality-assurance-guidelines.md` を参照してください。**

### 概要
- Pre-commit フックによるコミット前の自動品質チェック
- CI/CD との連携による段階的な品質保証
- コミット前の必須検証プロセス（import文整理、コードフォーマット等）
- 段階的リリース戦略（Pre-release、ブランチ戦略）
- 問題発生時の迅速対応メカニズム（ロールバック、ホットフィックス）
- 検証チェックリスト（コミット前、リリース前、緊急時）
- 継続的改善（問題パターンの記録と対策）

詳細は `quality-assurance-guidelines.md` を参照。

## テスト戦略

**テスト戦略の詳細は `testing-guidelines.md` を参照してください。**

### 概要
- テストは実装と同時に進行し、品質を継続的に確保
- 単体テスト → 統合テスト → E2Eテスト の順で実装
- カバレッジ目標：最低30%、推奨50%、理想70%（コアロジック）
- CI/CD でのテスト自動化必須
- Homebrew パッケージの場合は Formula テスト必須

詳細は `testing-guidelines.md` を参照。

## GitHub 操作ガイドライン（必須）

### プルリクエスト作成の原則

**重要**: プルリクエストの作成は GitHub MCP サーバーを使用する。`gh` コマンドや `git` コマンドは使用しない。

#### プルリクエスト作成手順

1. **変更のコミットとプッシュ**:
   ```bash
   git add -A
   git commit -m "Descriptive commit message"
   git push origin feature/branch-name
   ```

2. **GitHub MCP サーバーでプルリクエスト作成**:
   - `mcp_github_create_pull_request` ツールを使用
   - 必須パラメータ:
     - `owner`: リポジトリオーナー名
     - `repo`: リポジトリ名
     - `title`: プルリクエストのタイトル
     - `head`: ソースブランチ名（例: `feature/enhance-documentation`）
     - `base`: ターゲットブランチ名（通常は `main`）
   - オプションパラメータ:
     - `body`: プルリクエストの説明
     - `draft`: ドラフトPRとして作成する場合は `true`

#### プルリクエストのタイトルと説明

**タイトル形式**:
- 簡潔で内容が分かりやすいタイトル
- 例: "Add documentation internationalization (English/Japanese)"
- 例: "Fix Homebrew Formula resource dependencies"

**説明（body）の推奨内容**:
```markdown
## 変更内容
- 主要な変更点を箇条書きで記載

## 目的
- なぜこの変更が必要か

## テスト
- 実施したテスト内容
- 確認した動作

## 関連Issue
- Closes #123（該当する場合）
```

#### 例：プルリクエスト作成

```python
# GitHub MCP サーバーを使用
mcp_github_create_pull_request(
    owner="eijikominami",
    repo="display-layout-manager",
    title="Add documentation internationalization (English/Japanese)",
    head="feature/enhance-documentation",
    base="main",
    body="""## 変更内容
- 英語版ドキュメントの作成（README.md, ARCHITECTURE.md, docs/configuration.md, docs/troubleshooting.md）
- 日本語版ドキュメントの作成（_JP サフィックス）
- 各ドキュメントに言語切り替えリンクを追加
- 内部リンクを各言語版に対応
- CONTRIBUTING.md を削除（不要）

## 目的
- 国際的なユーザーに対応するため、英語と日本語の両方でドキュメントを提供
- ユーザーが母国語でアプリケーションの使い方を理解できるようにする

## テスト
- すべての言語切り替えリンクが機能することを確認
- 内部リンクが正しい言語版を参照していることを確認
- ドキュメントの内容が両言語で一致していることを確認
"""
)
```

### 避けるべき操作

#### ❌ 使用禁止
- `gh pr create` コマンド
- `gh` コマンド全般（GitHub CLI）
- Web ブラウザでの手動プルリクエスト作成の指示

#### ✅ 正しい方法
- GitHub MCP サーバーの `mcp_github_create_pull_request` ツールを使用
- プログラマティックにプルリクエストを作成

### その他の GitHub 操作

#### Issue 作成
- `mcp_github_issue_write` ツールを使用（method="create"）

#### Issue コメント追加
- `mcp_github_add_issue_comment` ツールを使用

#### プルリクエスト更新
- `mcp_github_update_pull_request` ツールを使用

#### リポジトリ情報取得
- `mcp_github_search_repositories` ツールを使用
- `mcp_github_list_branches` ツールを使用
- `mcp_github_list_commits` ツールを使用

### GitHub MCP サーバーの利点

1. **自動化**: コマンドラインツールを使わずにプログラマティックに操作
2. **一貫性**: 常に同じ方法でGitHub操作を実行
3. **エラーハンドリング**: 適切なエラーメッセージとリトライ機能
4. **統合**: 他のツールとシームレスに連携

## 品質保証の必須要件
- **実装変更時の仕様書・ドキュメント整合性確認必須**
- **品質保証メカニズム必須実行**: `quality-assurance-guidelines.md` 参照
- **テスト戦略に基づく包括的なテスト実施必須**: `testing-guidelines.md` 参照
- **GitHub 操作は GitHub MCP サーバーを使用**（gh コマンド禁止）
- **CloudFormation/SAM**: `aws-cloudformation-guidelines.md` 参照（CFN Lint exit code 0必須）