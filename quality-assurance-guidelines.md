# 品質保証ガイドライン

## コミット前品質保証メカニズム（必須）

### Pre-commit フックによる自動化（推奨）

**重要**: pre-commit フックを使用することで、コミット前の品質チェックを自動化できる

#### Pre-commit のセットアップ

##### 1. 開発依存関係のインストール
```bash
# setup.py に extras_require["dev"] が定義されている場合
pip install -e ".[dev]"

# または個別にインストール
pip install pre-commit black isort flake8
```

##### 2. Pre-commit フックのインストール
```bash
# プロジェクトルートで実行
pre-commit install
```

##### 3. 設定ファイルの作成（.pre-commit-config.yaml）
```yaml
repos:
  # Black フォーマッター
  - repo: https://github.com/psf/black
    rev: 23.12.1
    hooks:
      - id: black
        language_version: python3
        args: [--line-length=88]

  # isort（import文の整理）
  - repo: https://github.com/PyCQA/isort
    rev: 5.13.2
    hooks:
      - id: isort
        args: [--profile=black]

  # flake8（Lintチェック）
  - repo: https://github.com/PyCQA/flake8
    rev: 7.0.0
    hooks:
      - id: flake8
        args: [--max-line-length=88, --extend-ignore=E203]

  # 基本的なチェック
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
```

##### 4. 手動実行（オプション）
```bash
# 全ファイルに対して実行
pre-commit run --all-files

# 特定のフックのみ実行
pre-commit run black --all-files
pre-commit run isort --all-files
```

#### Pre-commit の利点
- **自動実行**: git commit 時に自動的にチェックが実行される
- **問題の早期発見**: コミット前に問題を検出できる
- **一貫性**: チーム全体で同じ品質基準を適用できる
- **CI/CD の負荷軽減**: ローカルで問題を解決してからプッシュできる

### リリース前の必須手動確認（GUI アプリケーション）

**重要**: 自動テストだけでは検出できない問題があるため、リリース前に必ず手動確認を実施する

#### GUI アプリケーションの手動確認項目

1. **実際の起動確認**
   ```bash
   # アプリケーションを実際に起動
   display-layout-menubar
   
   # 確認項目：
   # - メニューバーにアイコンが表示されるか
   # - アイコンをクリックしてメニューが表示されるか
   # - 各メニュー項目が正しく動作するか
   ```

2. **視覚的な確認**
   - アイコンが正しく表示されているか
   - メニュー項目のテキストが正しいか
   - チェックマークが正しく表示されるか

3. **機能確認**
   - 各メニュー項目をクリックして動作確認
   - エラーが発生しないか確認
   - ログファイルにエラーが記録されていないか確認

4. **スクリーンショット記録**
   - 正常動作時のスクリーンショットを撮影
   - リリースノートやドキュメントに添付

#### 手動確認のタイミング

- ✅ **プルリクエスト作成前**: feature ブランチで実装完了後
- ✅ **main マージ前**: レビュー完了後
- ✅ **リリース前**: タグプッシュ前に最終確認

#### 手動確認チェックリスト

```markdown
## GUI アプリケーション手動確認

- [ ] アプリケーションが起動する
- [ ] メニューバーにアイコンが表示される
- [ ] アイコンをクリックしてメニューが表示される
- [ ] 「レイアウトを適用」が動作する
- [ ] 「現在の設定を保存」が動作する
- [ ] 「ログイン時に起動」のチェックマークが切り替わる
- [ ] 「終了」でアプリが終了する
- [ ] エラーログに問題がない
- [ ] スクリーンショットを撮影した
```

### Pre-commit と CI/CD の連携ガイドライン（必須）

**重要原則**: GitHub Actions 等の CI/CD で実行される品質チェックは、可能な限り pre-commit でもローカル実行できるようにする

#### 1. CI/CD チェック項目の Pre-commit 対応

##### 対応すべき項目
- ✅ **コードフォーマット**: Black, isort → pre-commit で自動修正
- ✅ **Lint チェック**: flake8, pylint → pre-commit で検出
- ✅ **型チェック**: mypy → pre-commit で検出可能
- ✅ **セキュリティチェック**: bandit → pre-commit で検出可能
- ✅ **基本検証**: 末尾空白、EOF、YAML 構文 → pre-commit で自動修正
- ⚠️ **単体テスト**: pytest → pre-commit で実行可能だが重い場合は CI のみ
- ⚠️ **統合テスト**: → 通常は CI のみ（環境依存のため）
- ⚠️ **ビルドテスト**: Homebrew インストール等 → CI のみ

##### Pre-commit 設定例（包括的）
```yaml
repos:
  # コードフォーマット
  - repo: https://github.com/psf/black
    rev: 23.12.1
    hooks:
      - id: black
        language_version: python3
        args: [--line-length=88]

  # import 文整理
  - repo: https://github.com/PyCQA/isort
    rev: 5.13.2
    hooks:
      - id: isort
        args: [--profile=black]

  # Lint チェック
  - repo: https://github.com/PyCQA/flake8
    rev: 7.0.0
    hooks:
      - id: flake8
        args: [--max-line-length=88, --extend-ignore=E203]

  # 型チェック（オプション）
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.8.0
    hooks:
      - id: mypy
        args: [--ignore-missing-imports]

  # セキュリティチェック（オプション）
  - repo: https://github.com/PyCQA/bandit
    rev: 1.7.6
    hooks:
      - id: bandit
        args: [-c, pyproject.toml]

  # 基本チェック
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
      - id: check-merge-conflict
      - id: check-json

  # 単体テスト（軽量な場合のみ）
  # - repo: local
  #   hooks:
  #     - id: pytest-fast
  #       name: Fast Unit Tests
  #       entry: pytest tests/unit -v --tb=short
  #       language: system
  #       pass_filenames: false
```

#### 2. CI/CD と Pre-commit の役割分担

##### Pre-commit で実行すべきもの
- **高速なチェック**: 数秒以内に完了するもの
- **自動修正可能**: Black, isort 等の自動フォーマット
- **開発体験向上**: 即座にフィードバックが得られるもの
- **必須品質基準**: コミット前に必ず満たすべき基準

##### CI/CD でのみ実行すべきもの
- **時間のかかるテスト**: 統合テスト、E2E テスト
- **環境依存のテスト**: Homebrew インストール、Docker ビルド
- **複数環境テスト**: 複数の Python バージョン、OS
- **デプロイ関連**: リリース作成、パッケージ公開

#### 3. 段階的な品質チェック戦略

```
開発者ローカル環境
├── Pre-commit フック（必須）
│   ├── コードフォーマット（自動修正）
│   ├── Lint チェック（エラー検出）
│   └── 基本検証（自動修正）
│
├── 手動テスト（推奨）
│   ├── 単体テスト実行
│   └── 基本機能確認
│
└── Git Push
    ↓
GitHub Actions CI/CD
├── Pull Request 時
│   ├── 全品質チェック再実行
│   ├── 単体テスト + 統合テスト
│   ├── 複数環境テスト
│   └── Homebrew インストールテスト
│
└── Tag Push 時（リリース）
    ├── 全テスト実行
    ├── ビルド + パッケージ作成
    └── GitHub Release 作成
```

#### 4. Pre-commit スキップのガイドライン

##### スキップが許容される場合
- **緊急修正**: ホットフィックスで即座のコミットが必要
- **ドキュメントのみ変更**: コードに影響しない変更
- **WIP コミット**: 作業中の一時保存（後で修正前提）

##### スキップ方法
```bash
# 全フックをスキップ
git commit --no-verify -m "WIP: work in progress"

# 特定のフックのみスキップ
SKIP=flake8 git commit -m "Fix formatting, flake8 errors to be addressed"

# 複数のフックをスキップ
SKIP=flake8,mypy git commit -m "Quick fix"
```

##### スキップ時の注意事項
- **CI で必ず確認**: スキップしても CI でチェックされることを確認
- **後で修正**: WIP コミットは後で必ず修正してから PR 作成
- **記録**: コミットメッセージにスキップ理由を記載

#### 5. チーム開発での運用ルール

##### 必須ルール
1. **Pre-commit インストール必須**: 全開発者が `pre-commit install` を実行
2. **CI 失敗時の対応**: CI で失敗したら必ずローカルで再現・修正
3. **Pre-commit 更新**: `.pre-commit-config.yaml` 更新時は `pre-commit autoupdate` 実行

##### 推奨ルール
1. **定期的な更新**: 月1回程度 pre-commit フックのバージョンを更新
2. **新規フック追加**: チーム合意の上で段階的に追加
3. **パフォーマンス監視**: Pre-commit が遅い場合は設定を見直し

#### 6. トラブルシューティング

##### Pre-commit が遅い場合
```bash
# 特定のフックのみ実行
pre-commit run black --all-files

# 変更ファイルのみチェック
git add -A
pre-commit run

# キャッシュクリア
pre-commit clean
```

##### Pre-commit が失敗する場合
```bash
# 詳細ログ表示
pre-commit run --verbose --all-files

# 環境再構築
pre-commit clean
pre-commit install --install-hooks
```

##### CI と Pre-commit の結果が異なる場合
```bash
# CI と同じ環境で実行
pre-commit run --all-files --show-diff-on-failure

# Pre-commit 設定の更新
pre-commit autoupdate
```

### コミット前の必須検証プロセス

**重要**: Pre-commit フックを使用しない場合は、以下の検証を手動で実行する

#### 1. ローカル統合テスト（必須）

##### Python プロジェクトの場合
```bash
# 1. import文の整理（必須）
python3 -m isort --profile black src/ tests/

# 2. コードフォーマット（必須）
python3 -m black src/ tests/
# または特定のディレクトリのみ
python3 -m black [target_directory]

# 3. フォーマットチェック（確認用）
python3 -m isort --check-only --profile black src/ tests/
python3 -m black --check src/ tests/

# 4. 依存関係の確認
pip install -e .
python -c "import [main_module]; print('✓ Import successful')"

# 5. 基本機能テスト
[main_command] --version
[main_command] --help
[main_command] --run-tests

# 6. 重要機能の動作確認
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

## 段階的リリース戦略

### Pre-release での検証
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

### ブランチ戦略
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

## 自動化による品質保証

### Pre-commit フックの設定

#### Python プロジェクト向け設定例
```yaml
# .pre-commit-config.yaml
repos:
  # Black フォーマッター
  - repo: https://github.com/psf/black
    rev: 23.12.1
    hooks:
      - id: black
        language_version: python3
        args: [--line-length=88]
  
  # isort（import文の整理）
  - repo: https://github.com/PyCQA/isort
    rev: 5.13.2
    hooks:
      - id: isort
        args: [--profile=black]
  
  # flake8（Lintチェック）
  - repo: https://github.com/PyCQA/flake8
    rev: 7.0.0
    hooks:
      - id: flake8
        args: [--max-line-length=88, --extend-ignore=E203]
  
  # ローカルテスト
  - repo: local
    hooks:
      - id: local-tests
        name: Local Integration Tests
        entry: ./scripts/pre-commit-tests.sh
        language: script
        pass_filenames: false
```

#### Pre-commit のインストールと有効化
```bash
# pre-commit のインストール
pip install pre-commit

# プロジェクトに pre-commit フックを設定
pre-commit install

# 全ファイルに対して手動実行（初回）
pre-commit run --all-files

# 以降、git commit 時に自動実行される
```

### CI/CD での段階的検証
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

## 問題発生時の迅速対応メカニズム

### ロールバック戦略
```bash
# 1. 問題のあるリリースの特定
git log --oneline

# 2. 前のバージョンへのロールバック
git revert [problematic_commit]
git tag v1.x.x-hotfix.1

# 3. Homebrew Formula の緊急修正
# Formula を前のバージョンに戻す
```

### ホットフィックス手順
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

## 検証チェックリスト

### コミット前必須チェック項目
- [ ] **import文の整理実行完了**（Python: isort）
- [ ] **コードフォーマット実行完了**（Python: black、JavaScript: prettier等）
- [ ] ローカルでの基本機能テスト完了
- [ ] 依存関係の完全性確認完了
- [ ] 設定ファイルの構文チェック完了
- [ ] ドキュメントの整合性確認完了
- [ ] 重要な使用ケースのテスト完了

### リリース前必須チェック項目
- [ ] クリーン環境でのインストールテスト完了
- [ ] 全機能の動作確認完了
- [ ] アンインストールテスト完了
- [ ] ドキュメントの手順確認完了
- [ ] GitHub Actions の動作確認完了

### 緊急時対応チェック項目
- [ ] 問題の影響範囲特定完了
- [ ] ロールバック手順確認完了
- [ ] ユーザーへの通知準備完了
- [ ] 修正版のテスト完了

## 継続的改善

### 問題パターンの記録と対策
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

### 問題: Black フォーマッターエラー
- **発生日**: 2025-12-06
- **原因**: コミット前にコードフォーマットを実行していなかった
- **対策**: コミット前に `python3 -m black src/ tests/` を実行
- **予防策**: Pre-commit フックで Black を自動実行する設定を追加

### 問題: isort import文整理エラー
- **発生日**: 2025-12-06
- **原因**: コミット前に import 文の整理を実行していなかった
- **対策**: コミット前に `python3 -m isort --profile black src/ tests/` を実行
- **予防策**: Pre-commit フックで isort を自動実行する設定を追加（Black互換のため --profile black を使用）
```
