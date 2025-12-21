# テストガイドライン

## テスト戦略

テストは実装と同時に進行し、品質を継続的に確保する

## テスト作成の段階的アプローチ

テストは以下の段階で作成する。各段階は独立して価値があるが、上位の段階まで完了して初めてテストが完成する。

### 段階 1: ユーティリティ/ヘルパー関数の単体テスト

独立した小さな関数を先にテストする。

```python
# ユーティリティ関数のテスト
def test_format_bytes():
    assert format_bytes(1024) == "1.0 KB"
    assert format_bytes(1048576) == "1.0 MB"
```

この段階のテストは有効だが、これだけでは不十分。

### 段階 2: クラスメソッドの単体テスト

クラスの各メソッドを個別にテストする。外部依存はモック化する。

```python
# クラスメソッドのテスト（外部依存をモック）
def test_manager_get_videos_by_date_range():
    manager = PhotosAccessManager()
    with patch.object(manager, 'get_all_videos', return_value=mock_videos):
        result = manager.get_videos_by_date_range(from_date, to_date)
        assert len(result) == expected_count
```

### 段階 3: クラス統合テスト

クラス全体の動作をテストする。ヘルパー関数との連携を含む。

```python
# クラス統合テスト（内部ヘルパー関数との連携）
def test_convert_service_generates_correct_filename():
    service = ConvertService(...)
    result = service.convert_batch([candidate], ...)
    # ファイル命名ロジック（内部ヘルパー）が正しく動作することを確認
    assert result.converted_path.name == "video_h265.mp4"
```

### 段階 4: サービス間統合テスト

複数クラスの連携をテストする。

```python
# サービス間統合テスト
def test_scan_and_convert_workflow():
    scan_service = ScanService(...)
    convert_service = ConvertService(...)
    
    candidates = scan_service.scan(from_date, to_date)
    result = convert_service.convert_batch(candidates, ...)
```

### 完了基準

テストは段階 2（クラスメソッドの単体テスト）まで完了して初めて「完了」とする。

- ❌ 段階 1 のみ: ヘルパー関数のテストだけでは不完全
- ✅ 段階 2 以上: クラスメソッドのテストがあれば完了
- ✅✅ 段階 3 以上: より堅牢なテスト

### 避けるべき状態

ヘルパー関数のみをテストして、それを使用する本体クラスをテストしない状態は避ける。

```python
# ❌ 不完全: ヘルパー関数のみテスト
def test_filter_videos_by_date_range():
    result = filter_videos_by_date_range(videos, from_date, to_date)
# → PhotosAccessManager.get_videos_by_date_range() のテストがない

# ✅ 完全: 本体クラスもテスト
def test_manager_get_videos_by_date_range():
    manager = PhotosAccessManager()
    with patch.object(manager, 'get_all_videos', return_value=mock_videos):
        result = manager.get_videos_by_date_range(from_date, to_date)
```

## tasks.md でのテストタスク記述

### タスク記述の原則

tasks.md にテストタスクを記述する際は、段階的アプローチに従って記述する。

### 記述形式

```markdown
- [ ] X.Y [コンポーネント名]のテスト
  - [ ] 段階 1: ユーティリティ関数のテスト（該当する場合）
  - [ ] 段階 2: クラスメソッドの単体テスト
  - [ ] 段階 3: クラス統合テスト
  - _要件: [関連要件]_
```

### 記述例

```markdown
- [ ] 3.3 PhotosAccessManager のテスト
  - [ ] 段階 1: filter_videos_by_date_range ヘルパー関数のテスト
  - [ ] 段階 2: PhotosAccessManager.get_videos_by_date_range のテスト
  - [ ] 段階 3: PhotosAccessManager 全体の統合テスト
  - _要件: 1.1.1, 1.1.2, 1.1.3_

- [ ] 5.2 CompressionAnalyzer のテスト
  - [ ] 段階 2: CompressionAnalyzer.analyze_video のテスト
  - [ ] 段階 2: CompressionAnalyzer.should_skip のテスト
  - [ ] 段階 3: コーデック分類からスキップ判定までの統合テスト
  - _要件: 1.4, 1.5, 10.3_
```

### 完了判定

- 段階 2 のタスクがすべて完了: テストタスクは「完了」
- 段階 1 のみ完了: 「未完了」
- 段階 3 まで完了: より堅牢なテスト

## テスト作成時の原則

### テスト対象の特定

1. アプリケーションで使用されるクラス・メソッドを特定
2. 呼び出し経路の確認: ユーザーやシステムがどのようにそのコードを呼び出すかを確認
3. モックの範囲: 外部依存のみをモック化する

### テストデータの現実性

テストデータは現実的な値を使用する。

1. 日付範囲: 現在の年を含む広い範囲を使用（例：2015-2025）
2. データの多様性: 複数のパターン、エッジケースを含める
3. 境界値: 範囲の開始・終了の境界値をテスト

```python
# ❌ 特定の狭い日付範囲のみ（現在は 2025 年）
video = create_video(capture_date=datetime(2008, 6, 15))

# ✅ 現実的で広い日付範囲
videos = [
    create_video(capture_date=datetime(2015, 6, 15)),
    create_video(capture_date=datetime(2024, 11, 10)),
]
```

### 外部ライブラリの戻り値を確認する

外部ライブラリをモック化する前に、実際の戻り値の型・構造を確認する。

1. ドキュメント確認: ライブラリのドキュメントで戻り値の型を確認
2. 実際の動作確認: 可能であれば実際にライブラリを呼び出して戻り値を確認
3. モックの正確性: モックオブジェクトが実際の戻り値と同じ構造を持つことを確認

### テスト作成のチェックリスト

#### 段階的アプローチの確認
- [ ] 段階 1: ユーティリティ/ヘルパー関数のテストを作成したか
- [ ] 段階 2: クラスメソッドの単体テストを作成したか
- [ ] 段階 3: クラス統合テストを作成したか
- [ ] ヘルパー関数のみをテストして終わっていないか

#### テスト品質の確認
- [ ] テストデータは現実的な値を使用しているか
- [ ] 日付範囲は現在の年を含む広い範囲か
- [ ] 外部ライブラリの戻り値の型・構造を確認したか
- [ ] 境界値・エッジケースをテストしているか

## テスト実行タイミング

### 概要

テストは作成するだけでなく、適切なタイミングで実行する。

### 実行タイミングの分類

#### 1. 実装中のテスト実行
タイミング: コンポーネント実装直後
範囲: 実装したコンポーネントの単体テストのみ
目的: 実装が正しく動作することを即座に確認

```bash
# 例：新しいコンポーネントのテストのみ実行
pytest tests/test_new_component.py -v
```

#### 2. コミット前のテスト実行
タイミング: git commit 実行前
範囲: 変更に関連するテストのみ
目的: 変更が既存機能を壊していないことを確認

```bash
# 例：変更したファイルに関連するテストのみ実行
pytest tests/test_modified_*.py -v
```

所要時間目標: 数秒〜1分以内

#### 3. プルリクエスト作成前のテスト実行（リリース前テスト 1回目）
タイミング: featureブランチでの作業完了後、プルリクエスト作成前
範囲: 全テスト（単体・統合・E2E）
目的: mainブランチに壊れたコードをマージしない

```bash
# featureブランチで実行
pytest -v                    # 全テスト実行
coverage run -m pytest       # カバレッジ測定
coverage report              # カバレッジ確認
```

追加確認:
- GUI アプリケーションの場合：実際に起動して動作確認
- ドキュメントの更新確認
- コードフォーマット確認

所要時間目標: 5分以内

#### 4. タグプッシュ前のテスト実行（リリース前テスト 2回目）
タイミング: mainブランチでバージョン更新後、タグプッシュ前
範囲: 全テスト（単体・統合・E2E）
目的: ユーザーに壊れたバージョンをリリースしない

```bash
# mainブランチで実行
git checkout main
git pull origin main
pytest -v                    # 全テスト実行
coverage run -m pytest       # カバレッジ測定
coverage report              # カバレッジ確認
```

追加確認:
- GUI アプリケーションの場合：実際に起動して全機能確認
- CHANGELOG.md の更新確認
- バージョン番号の確認
- README.md の最新性確認

所要時間目標: 5分以内

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

### リリース前テストは2回実行する

| タイミング | 実行場所 | 目的 | 失敗時の対応 |
|-----------|---------|------|-------------|
| 1回目 | featureブランチ | mainへのマージ前確認 | featureブランチで修正 |
| 2回目 | mainブランチ | ユーザーへのリリース前確認 | リリース中止、修正 |

### テスト実行の順序

#### Phase 1: コア機能の単体テスト
1. 基盤コンポーネント: 依存関係が少ないコンポーネントから
2. ビジネスロジック: 主要な処理ロジック
3. ユーティリティ: 共通処理

#### Phase 2: 統合テスト
1. コンポーネント連携: 2-3個のコンポーネントの組み合わせ
2. 外部依存: ファイルシステム、外部コマンド
3. エラーハンドリング: 異常系の処理

#### Phase 3: E2Eテスト
1. 主要フロー: ユーザーが最も使う機能
2. エッジケース: 特殊な状況での動作
3. エラーリカバリ: エラー発生時の復旧

### テストカバレッジの目標

#### カバレッジ基準
- 最低ライン: 全体で30%以上
- 中間ライン: 全体で50%以上
- 目標ライン: コアロジックで70%以上

#### 優先順位
1. 高優先度（70%以上目標）:
   - ビジネスロジック
   - データ変換処理
   - エラーハンドリング

2. 中優先度（50%以上目標）:
   - ユーティリティ関数
   - 設定管理
   - ファイルI/O

3. 低優先度（30%以上目標）:
   - UI表示ロジック
   - ログ出力
   - 初期化処理

### テスト完了の判断基準

- [ ] 全テストが成功
- [ ] カバレッジが最低基準（30%）を超えている
- [ ] CI/CDパイプラインでテストが自動実行される

### GUI アプリケーションのテスト戦略

### メニューバーアプリケーション等の GUI テスト

GUI コンポーネントは、モックだけでなく実際の初期化もテストする

#### テストの種類

1. 単体テスト（モック使用）
   - コンポーネントのロジックをテスト
   - 外部依存をモック化
   - 高速実行

2. 統合テスト（実際の初期化）
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

#### GUI テスト

1. 実際の初期化テストを含める
   - モックだけでは実際の動作を検証できない
   - パラメータの設定ミスを検出できない

2. 環境依存を明記
   - macOS 専用の場合は明記
   - CI/CD で実行可能か確認

3. 視覚的な確認も実施
   - 自動テストだけでなく、実際に起動して確認
   - スクリーンショットを撮って記録

4. 統合テストを CI/CD に含める
   - GitHub Actions で macOS ランナーを使用
   - 実際の GUI 初期化をテスト

#### テストファイル構成例

```
tests/
├── test_menubar_app.py              # 単体テスト（モック使用）
├── test_menubar_initialization.py   # 統合テスト（実際の初期化）
└── test_menubar_integration.py      # E2E テスト
```



### Homebrew Formula テスト

Homebrew でインストール可能なプロジェクトは、プルリクエスト時に Homebrew Formula のテストを必ず実行する

#### テストの目的
- 本番リリース前に Homebrew インストールの問題を検出
- すべてのエントリーポイント（コマンド）が正しく作成されることを確認
- 依存関係が正しくインストールされることを確認
- ユーザーに影響が出る前にバグを修正

#### GitHub Actions での自動テスト

プルリクエスト時に自動的に以下のテストが実行される：

1. Formula 作成: 現在のソースコードから一時的な Homebrew Formula を作成
2. ローカルインストール: `brew install --build-from-source` でインストール
3. コマンド確認: すべてのエントリーポイントが作成されているか確認
4. 基本機能テスト: `--version`, `--help` などの基本コマンドをテスト
5. クリーンアップ: テスト後にアンインストール

#### ローカルでの Homebrew テスト手順

プルリクエストを作成する前に、ローカルで Homebrew インストールをテストする：

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
