---
inclusion: always
---

# 品質保証ガイドライン

## AI アシスタントの責務

コード変更時に以下を自動実行する。ユーザーの指示を待たない。

### 実行項目

| 項目 | コマンド | 実行タイミング |
|------|---------|---------------|
| フォーマット | `python3.11 -m black src/` | コード変更後 |
| Lint | `python3.11 -m ruff check src/` | コード変更後 |
| 型チェック | `python3.11 -m mypy src/` | コード変更後 |
| 単体テスト | `python3.11 -m pytest tests/unit/ -v` | 機能変更後 |
| 診断確認 | `getDiagnostics` ツール | ファイル編集後 |
| カバレッジ検証 | `python3.11 -m coverage run -m pytest tests/ && python3.11 -m coverage report` | Phase 完了時・リリース前 |

### カバレッジ目標

Phase 完了時・リリース前に以下を満たすこと。未達成の場合はテストを追加する。

| 優先度 | 対象 | 目標 | 未達成時のアクション |
|--------|------|------|---------------------|
| 高 | ビジネスロジック、データ変換、エラーハンドリング | 70%+ | テスト追加 |
| 中 | ユーティリティ、設定管理、ファイル I/O | 50%+ | テスト追加 |
| 低 | UI 表示、ログ出力、初期化処理 | 30%+ | テスト追加推奨 |

カバレッジ目標未達成のままリリース・Phase 完了としない。

### 検証失敗時の対応

1. エラーを修正する（謝罪のみで終わらせない）
2. 再度検証を実行
3. 全て成功するまで繰り返す

## 確認の責任分担

### AI が実施（自動）

- コードフォーマット・Lint・型チェック
- テスト実行・カバレッジ確認
- README の手順・コード例の整合性
- 設定ファイルの構文検証
- 依存関係の整合性（pyproject.toml）
- CFN Lint（SAM テンプレート変更時）

### 人間が実施（手動）

- アプリケーション起動・主要機能の動作確認
- UI/UX の視覚確認
- 要件充足・ビジネスロジックの妥当性
- セキュリティレビュー

## dataclass/モデル変更時

フィールド追加・変更時は以下を確認する。

### 更新必須箇所

```python
# 1. フィールド定義
@dataclass
class Config:
    new_field: str = ""  # 追加

# 2. シリアライズ
def to_dict(self) -> dict:
    return {
        "new_field": self.new_field,  # 追加
        ...
    }

# 3. デシリアライズ
@classmethod
def from_dict(cls, data: dict) -> "Config":
    return cls(
        new_field=data.get("new_field", ""),  # 追加（デフォルト値必須）
        ...
    )
```

### 検証手順

```bash
# 往復テスト実行
python3.11 -m pytest tests/unit/test_config.py -v -k roundtrip

# CLI 動作確認
vco config show
```

## チェックリスト

### コミット前（AI 実行）

```bash
# 全て exit code 0 であること
python3.11 -m black src/ --check
python3.11 -m ruff check src/
python3.11 -m pytest tests/unit/ -v
```

### プルリクエスト前（AI + 人間）

```bash
# AI: 全テスト実行 + カバレッジ検証
python3.11 -m coverage run -m pytest tests/ -v
python3.11 -m coverage report

# カバレッジ目標確認（未達成なら PR 作成前にテスト追加）
# - ビジネスロジック: 70%+
# - 設定管理・ファイル I/O: 50%+
# - UI: 30%+

# 人間: 動作確認
# - アプリケーション起動
# - 主要機能テスト
```

### リリース前（AI + 人間）

```bash
# AI: クリーン環境テスト
python3.11 -m pip install -e .
python3.11 -m pytest tests/ -v

# 人間: 最終確認
# - 全機能動作確認
# - ドキュメント確認
# - CHANGELOG 確認
```

## 緊急時対応

問題発生時の優先順序：

1. 影響範囲を特定（どの機能が影響を受けるか）
2. ロールバック可否を判断
3. 修正 → テスト → 再デプロイ

## 関連ガイドライン

- テスト戦略・カバレッジ目標: `testing-guidelines.md`
- 開発ワークフロー: `development-guidelines.md`
