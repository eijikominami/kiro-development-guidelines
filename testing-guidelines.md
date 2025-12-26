---
inclusion: always
---

# テストガイドライン

テストは実装と同時に進行し、品質を継続的に確保する。

## 段階的テストアプローチ

テストは段階 2（クラスメソッドの単体テスト）まで完了して初めて「完了」とする。

| 段階 | 内容 | 完了判定 |
|------|------|----------|
| 1 | ユーティリティ/ヘルパー関数の単体テスト | ❌ これだけでは不完全 |
| 2 | クラスメソッドの単体テスト（外部依存はモック化） | ✅ 完了 |
| 3 | クラス統合テスト（ヘルパー関数との連携） | ✅✅ より堅牢 |
| 4 | サービス間統合テスト | ✅✅✅ 最も堅牢 |

### 避けるべきパターン

```python
# ❌ ヘルパー関数のみテストして本体クラスをテストしない
def test_filter_videos_by_date_range():
    result = filter_videos_by_date_range(videos, from_date, to_date)

# ✅ 本体クラスもテスト
def test_manager_get_videos_by_date_range():
    manager = PhotosAccessManager()
    with patch.object(manager, 'get_all_videos', return_value=mock_videos):
        result = manager.get_videos_by_date_range(from_date, to_date)
```

## テストディレクトリ構造

```
tests/
├── conftest.py              # 共通フィクスチャ
├── unit/                    # 単体テスト（段階 1-2）
├── properties/              # Property-based テスト（Hypothesis）
├── integration/             # 統合テスト（段階 3-4）
│   └── aws/                 # デプロイ後テスト
└── e2e/                     # E2E テスト
```

## テスト実行コマンド

プロジェクトで指定された Python バージョンとテストランナーを使用する。

```bash
# 単体テスト（例: pytest）
pytest tests/unit/ -v

# Property-based テスト
pytest tests/properties/ -v

# デプロイ後テスト
pytest tests/integration/ -v -m deployed

# 全テスト + カバレッジ
coverage run -m pytest tests/ -v
coverage report
```

## テスト実行タイミング

| タイミング | 範囲 | 目的 |
|-----------|------|------|
| 実装直後 | 該当コンポーネントのみ | 実装の即時検証 |
| コミット前 | 変更関連テスト | 既存機能の破壊防止 |
| PR 作成前 | 全テスト | main への品質保証 |
| タグプッシュ前 | 全テスト | リリース品質保証 |

## デプロイ後テスト

モックでは検出できない問題を検証する。

| 問題 | 例 |
|-----|---|
| 設定の誤り | 環境変数の未設定、IAM 権限不足 |
| 依存関係の不足 | Lambda Layer の欠落、バイナリの PATH 設定ミス |
| API パラメータの誤り | パラメータの欠落、値の制約違反 |

```python
import pytest

pytestmark = pytest.mark.deployed

class TestDeployedLambda:
    def test_lambda_invocation(self, lambda_client):
        response = lambda_client.invoke(
            FunctionName="my-function",
            Payload=json.dumps({"key": "value"})
        )
        assert response["StatusCode"] == 200
```

## Property-based テスト

Hypothesis を使用してランダム入力に対する不変条件を検証する。

Property-based テストは時間がかかっても完了まで実行する。途中でスキップしない。

### 実行ルール

| ルール | 説明 |
|--------|------|
| 完了まで実行 | Property-based テストは時間がかかっても最後まで実行する |
| タイムアウト | 単体テストより長いタイムアウトを設定（5-10 分） |
| スキップ禁止 | 時間を理由にスキップしない |

### AI アシスタントのタイムアウト設定

テスト実行時のタイムアウト目安：

| テスト種別 | タイムアウト |
|-----------|-------------|
| 単体テスト | 120 秒（2 分） |
| Property-based テスト | 600 秒（10 分） |
| 統合テスト | 300 秒（5 分） |
| 全テスト | 900 秒（15 分） |

```python
from hypothesis import given, strategies as st

class TestCodecClassification:
    """検証対象: 要件 1.4, 1.5, 10.3"""
    
    @given(codec=st.sampled_from(["h264", "H264", "avc1", "mpeg2video"]))
    def test_inefficient_codecs_marked_as_pending(self, codec):
        video = create_video(codec=codec)
        result = analyzer.analyze_video(video)
        assert result.status == CandidateStatus.PENDING
```

## テストデータのバリエーション

データモデルの各フィールドについて、取りうる値のバリエーションをテストする。

| フィールド型 | テストすべき値 |
|-------------|---------------|
| Boolean | True, False |
| Optional | None, 値あり |
| Enum | すべての値 |
| 数値 | 0, 最大値, 負数（境界値） |
| 文字列 | 空文字, 通常値, 特殊文字 |

Boolean フィールドが複数ある場合は全組み合わせをテスト:

```python
@pytest.mark.parametrize("is_local,is_in_icloud,expected", [
    (True, False, "use_local"),
    (False, True, "download_from_icloud"),
    (True, True, "use_local"),
    (False, False, "error"),
])
def test_file_access_combinations(is_local, is_in_icloud, expected):
    video = create_video(is_local=is_local, is_in_icloud=is_in_icloud)
    ...
```

## テストデータの現実性

```python
# ❌ 特定の狭い日付範囲のみ
video = create_video(capture_date=datetime(2008, 6, 15))

# ✅ 現実的で広い日付範囲
videos = [
    create_video(capture_date=datetime(2015, 6, 15)),
    create_video(capture_date=datetime(2024, 11, 10)),
]
```

## テストデータの独立性

テストデータは実装コードから独立して定義する。実装の定数やリストを直接参照すると、実装の漏れをテストが検出できない。

```python
# ❌ 実装を参照（実装の漏れを検出できない）
from myapp.analyzer import OPTIMIZED_CODECS

@given(codec=st.sampled_from(list(OPTIMIZED_CODECS)))
def test_optimized_codecs(self, codec):
    assert analyzer.classify(codec) == "optimized"

# ✅ テスト側で独立して定義（外部仕様から）
# Source: FFmpeg documentation, Apple Technical Note TN2224
EXPECTED_H265_CODECS = ["hevc", "hev1", "hvc1", "h265", "x265"]

def test_implementation_covers_all_h265_variants(self):
    """実装が仕様で定義された全バリアントをカバーしていることを検証"""
    from myapp.analyzer import OPTIMIZED_CODECS
    for codec in EXPECTED_H265_CODECS:
        assert codec in OPTIMIZED_CODECS, f"Missing: {codec}"
```

### 原則

| ルール | 説明 |
|--------|------|
| 独立定義 | テストデータは外部仕様（ドキュメント、RFC 等）から定義 |
| 出典明記 | テストデータの出典をコメントで記載 |
| カバレッジ検証 | 実装が仕様の全項目をカバーしていることをテスト |

### 適用対象

- コーデック名、ファイル形式などの分類リスト
- エラーコード、ステータスコードの定義
- 外部サービスの制約値（サイズ上限、タイムアウト等）

## 要件ベーステスト

テストの docstring に対応する要件番号を記載する:

```python
class TestCodecClassification:
    """検証対象: 要件 1.4, 1.5, 10.3
    - 1.4: 非効率コーデックを変換候補としてマーク
    - 1.5: H.265 は「最適化済み」としてスキップ
    """
    
    def test_inefficient_codecs_marked_as_pending(self):
        """1.4: 非効率コーデックは変換候補としてマークされる"""
        ...
```

## dataclass/モデル変更時

フィールド追加・変更時は以下を確認:

1. フィールド定義を更新
2. `to_dict` / `from_dict` を更新
3. 往復テストを追加

```python
def test_config_roundtrip():
    original = Config(field1="test", field2="value")
    dict_data = config_to_dict(original)
    restored = config_from_dict(dict_data)
    assert restored.field1 == original.field1
    assert restored.field2 == original.field2
```

### 状態の一貫性確認

```python
# ❌ 個別フィールドのみテスト
def test_skip_reason_is_set():
    result = analyzer.analyze(item)
    assert result.skip_reason == "Duration too short"

# ✅ 関連フィールドの一貫性をテスト
def test_skip_reason_and_status_consistency():
    result = analyzer.analyze(item)
    assert result.skip_reason == "Duration too short"
    assert result.status == Status.SKIPPED
```

## カバレッジ目標

Phase 完了時・リリース前に以下を満たすこと。未達成のままリリースしない。

| 優先度 | 対象 | 目標 | 例 |
|--------|------|------|-----|
| 高 | ビジネスロジック、データ変換、エラーハンドリング | 70%+ | analyzer, services, quality |
| 中 | ユーティリティ、設定管理、ファイル I/O | 50%+ | config, metadata, utils |
| 低 | UI 表示、ログ出力、初期化処理 | 30%+ | cli |

### カバレッジ検証タイミング

| タイミング | アクション |
|-----------|-----------|
| Phase 完了時 | カバレッジレポート実行、目標未達成ならテスト追加 |
| PR 作成前 | カバレッジ目標達成を確認 |
| リリース前 | 全モジュールがカバレッジ目標を満たすことを確認 |

### 検証コマンド

```bash
# カバレッジ計測（プロジェクトのテストランナーを使用）
coverage run -m pytest tests/ -v

# レポート表示（モジュール別）
coverage report --show-missing

# 目標未達成モジュールの特定
# - 70% 未満のビジネスロジック → テスト追加
# - 50% 未満の設定管理 → テスト追加
# - 30% 未満の UI → テスト追加を推奨
```

## tasks.md でのテストタスク記述

```markdown
- [ ] X.Y [コンポーネント名]のテスト
  - [ ] 段階 2: クラスメソッドの単体テスト
  - [ ] 段階 3: クラス統合テスト
  - _要件: [関連要件]_
```

## 外部ライブラリのモック

モック化する前に実際の戻り値の型・構造を確認する:

1. ライブラリのドキュメントで戻り値の型を確認
2. 可能であれば実際に呼び出して戻り値を確認
3. モックが実際の戻り値と同じ構造を持つことを確認

## GUI テスト（該当する場合）

モックだけでなく実際の初期化もテストする:

```python
# ❌ モックのみ
def test_app_initialization():
    with patch('framework.App.__init__') as mock_init:
        mock_init.return_value = None
        app = MyApp()

# ✅ 実際の初期化
def test_app_title_initialization():
    app = MyApp()
    assert app.title == "Expected Title"
```

## Homebrew Formula テスト（該当する場合）

PR 作成前にローカルで Homebrew インストールをテスト:

```bash
tar czf /tmp/test-source.tar.gz --exclude='.git' .
SHA256=$(shasum -a 256 /tmp/test-source.tar.gz | cut -d' ' -f1)
# Formula 作成 → brew install --build-from-source → 動作確認
```

確認項目:
- すべてのエントリーポイントが作成される
- `--version`, `--help` が動作する
- 主要機能が動作する
