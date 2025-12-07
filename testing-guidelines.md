# テストガイドライン

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

## テスト実行タイミング（必須）

### 概要

テストは**作成するだけでなく、適切なタイミングで実行する**ことが重要です。

### 実行タイミングの分類

#### 1. 実装中のテスト実行
**タイミング**: コンポーネント実装直後
**範囲**: 実装したコンポーネントの単体テストのみ
**目的**: 実装が正しく動作することを即座に確認

```bash
# 例：新しいコンポーネントのテストのみ実行
pytest tests/test_new_component.py -v
```

#### 2. コミット前のテスト実行
**タイミング**: git commit 実行前
**範囲**: 変更に関連するテストのみ
**目的**: 変更が既存機能を壊していないことを確認

```bash
# 例：変更したファイルに関連するテストのみ実行
pytest tests/test_modified_*.py -v
```

**所要時間目標**: 数秒〜1分以内

#### 3. プルリクエスト作成前のテスト実行（リリース前テスト 1回目）
**タイミング**: featureブランチでの作業完了後、プルリクエスト作成前
**範囲**: 全テスト（単体・統合・E2E）
**目的**: mainブランチに壊れたコードをマージしない

```bash
# featureブランチで実行
pytest -v                    # 全テスト実行
coverage run -m pytest       # カバレッジ測定
coverage report              # カバレッジ確認
```

**追加確認**:
- GUI アプリケーションの場合：実際に起動して動作確認
- ドキュメントの更新確認
- コードフォーマット確認

**所要時間目標**: 5分以内

#### 4. タグプッシュ前のテスト実行（リリース前テスト 2回目）
**タイミング**: mainブランチでバージョン更新後、タグプッシュ前
**範囲**: 全テスト（単体・統合・E2E）
**目的**: ユーザーに壊れたバージョンをリリースしない

```bash
# mainブランチで実行
git checkout main
git pull origin main
pytest -v                    # 全テスト実行
coverage run -m pytest       # カバレッジ測定
coverage report              # カバレッジ確認
```

**追加確認**:
- GUI アプリケーションの場合：実際に起動して全機能確認
- CHANGELOG.md の更新確認
- バージョン番号の確認
- README.md の最新性確認

**所要時間目標**: 5分以内

### テスト実行の完全なワークフロー

```
1. 実装開始
   └─ featureブランチ作成

2. 実装中
   ├─ コンポーネント実装
   ├─ 単体テスト作成
   └─ 単体テスト実行 ✅ (実装中のテスト)

3. コミット前
   ├─ 変更に関連するテスト実行 ✅ (コミット前のテスト)
   ├─ コードフォーマット
   └─ git commit

4. プルリクエスト作成前
   ├─ 全テスト実行 ✅✅✅ (リリース前テスト 1回目)
   ├─ カバレッジ確認
   ├─ GUI手動確認（該当する場合）
   └─ プルリクエスト作成

5. レビュー・マージ
   └─ mainブランチへマージ

6. リリース準備
   ├─ mainブランチで最新化
   ├─ バージョン更新
   └─ CHANGELOG更新

7. タグプッシュ前
   ├─ 全テスト実行 ✅✅✅ (リリース前テスト 2回目)
   ├─ カバレッジ確認
   ├─ GUI手動確認（該当する場合）
   └─ タグプッシュ

8. リリース
   └─ GitHub Actions が自動実行
```

### リリース前テストは2回実行する（必須）

| タイミング | 実行場所 | 目的 | 失敗時の対応 |
|-----------|---------|------|-------------|
| **1回目** | featureブランチ | mainへのマージ前確認 | featureブランチで修正 |
| **2回目** | mainブランチ | ユーザーへのリリース前確認 | リリース中止、修正 |

**重要**: 両方のタイミングで全テストを実行することで、二重の安全網を確保します。

### テスト実行の順序（推奨）

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

### GUI アプリケーションのテスト戦略（必須）

### メニューバーアプリケーション等の GUI テスト

**重要**: GUI コンポーネントは、モックだけでなく実際の初期化もテストする

#### テストの種類

1. **単体テスト（モック使用）**
   - コンポーネントのロジックをテスト
   - 外部依存をモック化
   - 高速実行

2. **統合テスト（実際の初期化）**
   - 実際のライブラリ初期化をテスト
   - パラメータが正しく設定されているか確認
   - GUI フレームワークの動作を検証

#### メニューバーアプリのテスト例

```python
# ❌ 悪い例：モックのみでテスト
def test_menubar_initialization():
    with patch('rumps.App.__init__') as mock_init:
        mock_init.return_value = None
        app = DisplayLayoutMenuBar()
        # title が正しく設定されているか確認できない！

# ✅ 良い例：実際の初期化もテスト
def test_menubar_title_initialization():
    """実際の rumps 初期化をテストして title を確認"""
    app = DisplayLayoutMenuBar()
    
    # title が設定されているか確認
    assert app.title is not None, "app.title が None であってはならない"
    assert app.title == "⧈", "app.title が期待値と一致すべき"
    
    # name が設定されているか確認
    assert app.name is not None, "app.name が None であってはならない"
```

#### GUI テストのベストプラクティス

1. **実際の初期化テストを必ず含める**
   - モックだけでは実際の動作を検証できない
   - パラメータの設定ミスを検出できない

2. **環境依存を明記**
   - macOS 専用の場合は明記
   - CI/CD で実行可能か確認

3. **視覚的な確認も実施**
   - 自動テストだけでなく、実際に起動して確認
   - スクリーンショットを撮って記録

4. **統合テストを CI/CD に含める**
   - GitHub Actions で macOS ランナーを使用
   - 実際の GUI 初期化をテスト

#### テストファイル構成例

```
tests/
├── test_menubar_app.py              # 単体テスト（モック使用）
├── test_menubar_initialization.py   # 統合テスト（実際の初期化）
└── test_menubar_integration.py      # E2E テスト
```

## プロジェクトタイプ別のテスト戦略

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

### Homebrew Formula テスト（必須）

**重要**: Homebrew でインストール可能なプロジェクトは、プルリクエスト時に Homebrew Formula のテストを必ず実行する

#### テストの目的
- 本番リリース前に Homebrew インストールの問題を検出
- すべてのエントリーポイント（コマンド）が正しく作成されることを確認
- 依存関係が正しくインストールされることを確認
- ユーザーに影響が出る前にバグを修正

#### GitHub Actions での自動テスト

プルリクエスト時に自動的に以下のテストが実行されます：

1. **Formula 作成**: 現在のソースコードから一時的な Homebrew Formula を作成
2. **ローカルインストール**: `brew install --build-from-source` でインストール
3. **コマンド確認**: すべてのエントリーポイントが作成されているか確認
4. **基本機能テスト**: `--version`, `--help` などの基本コマンドをテスト
5. **クリーンアップ**: テスト後にアンインストール

#### ローカルでの Homebrew テスト手順

プルリクエストを作成する前に、ローカルで Homebrew インストールをテストすることを推奨します：

```bash
# 1. 現在のソースコードをアーカイブ
tar czf /tmp/test-source.tar.gz --exclude='.git' .

# 2. SHA256 を計算
SHA256=$(shasum -a 256 /tmp/test-source.tar.gz | cut -d' ' -f1)
echo "SHA256: $SHA256"

# 3. テスト用 Formula を作成
mkdir -p /tmp/homebrew-test/Formula
cat > /tmp/homebrew-test/Formula/your-package.rb << EOF
class YourPackage < Formula
  include Language::Python::Virtualenv

  desc "Your package description"
  homepage "https://github.com/your-org/your-package"
  url "file:///tmp/test-source.tar.gz"
  sha256 "$SHA256"
  license "MIT"

  depends_on "python@3.11"
  # その他の依存関係

  # リソース（Python パッケージ）を追加
  resource "package-name" do
    url "https://files.pythonhosted.org/packages/.../package.tar.gz"
    sha256 "..."
  end

  def install
    virtualenv_install_with_resources
  end

  test do
    assert_match "Version", shell_output("#{bin}/your-command --version")
  end
end
EOF

# 4. ローカルインストール
brew install --build-from-source /tmp/homebrew-test/Formula/your-package.rb

# 5. インストール確認
which your-command
your-command --version

# 6. すべてのコマンドが作成されているか確認
ls -la $(brew --prefix)/bin/ | grep your-package

# 7. Cellar 構造を確認
CELLAR_PATH=$(brew --cellar your-package)/$(your-command --version | grep -oE '[0-9]+\.[0-9]+\.[0-9]+')
ls -la $CELLAR_PATH/bin/

# 8. クリーンアップ
brew uninstall your-package
rm -rf /tmp/homebrew-test /tmp/test-source.tar.gz
```

#### よくある問題と解決策

##### 1. エントリーポイントが作成されない
**原因**: Python 依存関係のインストールが失敗している
**解決策**: 
- すべての依存関係を Formula の `resource` セクションに追加
- 特に `pyobjc-core` などの基盤パッケージを忘れずに追加

##### 2. モジュールが見つからない（ModuleNotFoundError）
**原因**: 依存関係の依存関係が不足している
**解決策**:
- `pip show package-name` で依存関係を確認
- 不足している依存関係を Formula に追加

##### 3. インストールは成功するがコマンドが動作しない
**原因**: ランタイム依存関係が不足している
**解決策**:
- `brew install` 後に実際にコマンドを実行してテスト
- エラーメッセージから不足している依存関係を特定

#### チェックリスト

プルリクエスト作成前：
- [ ] ローカルで Homebrew インストールテストを実行
- [ ] すべてのエントリーポイント（コマンド）が作成されることを確認
- [ ] 基本的なコマンド（`--version`, `--help`）が動作することを確認
- [ ] 主要機能が動作することを確認

プルリクエスト作成後：
- [ ] GitHub Actions の `homebrew-test` ジョブが成功することを確認
- [ ] テスト失敗時は原因を特定して修正
- [ ] すべてのテストが成功してから main にマージ
