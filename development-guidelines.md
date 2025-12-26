---
inclusion: always
---

# 開発ガイドライン

## 言語規則

- ドキュメント・チャット: 日本語
- コードコメント: 英語可
- 変数名・関数名: 英語

## Git ワークフロー

### ブランチ戦略

main への直接コミット禁止。

| ブランチ種別 | 命名規則 |
|-------------|---------|
| 新機能 | `feature/<機能名>` |
| バグ修正 | `fix/<修正内容>` |
| 緊急修正 | `hotfix/<バージョン>-<修正内容>` |

### バージョニング

セマンティックバージョニング（MAJOR.MINOR.PATCH）を使用。

## タスク実行ルール

### 実装開始前（必須）

1. タスクに「〜に従って」の記載がある場合 → 参照ファイルを先に読む
2. 参照ファイルの要件をチェックリスト化

### 実装完了前（必須）

1. タスクの全指示項目を確認
2. 検証項目（CFN Lint 等）を実行
3. ガイドライン準拠を確認

### CloudFormation/SAM 作成時

1. `aws-cloudformation-guidelines.md` を参照
2. MCP で AWS ドキュメントを確認
3. CFN Lint で検証（exit code 0 必須）

## 外部サービス・ライブラリの制約調査

実装前に制約を調査・文書化する。

### 調査すべき制約

- 入力: フォーマット、コーデック、ファイルサイズ上限
- 出力: 生成可能フォーマット、解像度制限
- 処理: 最大処理時間、同時実行数、レート制限
- リソース: メモリ、ストレージ、ネットワーク帯域
- コスト: 料金体系、無料枠、課金単位

### 調査手順

1. 公式ドキュメントで「Quotas」「Limits」「Supported formats」を検索
2. AWS の場合は MCP で最新情報を確認
3. requirements.md に制約を追記
4. design.md に対応方針を記載

## 手動作業の自動化

手動で実行した作業は、再現性のあるスクリプトにする。

### スクリプト化すべき作業

- AWS リソースの作成（Lambda Layer、S3 バケット等）
- 環境セットアップ
- デプロイ手順
- データ移行

### スクリプト作成の必須要素

```bash
#!/bin/bash
set -e  # エラー時に停止

# ヘルプ表示
show_help() { ... }

# 引数解析（--help, --dry-run 必須）
while [[ $# -gt 0 ]]; do
    case $1 in
        --dry-run) DRY_RUN=true; shift ;;
        --help) show_help; exit 0 ;;
        *) echo "エラー: 不明なオプション: $1"; exit 1 ;;
    esac
done

# 一時ファイルのクリーンアップ
WORK_DIR=$(mktemp -d)
trap "rm -rf $WORK_DIR" EXIT

# 進捗表示
echo "[1/N] ステップ 1..."
```

## ドキュメント更新ルール

コード変更完了直後に README.md を更新する。「後でまとめて」は禁止。

### 更新順序

1. コード実装
2. README.md（即時更新）
3. 仕様書（requirements.md → design.md → tasks.md）
4. CHANGELOG.md

### README.md 更新トリガー

以下の変更時は必ず更新:

- 機能追加・削除
- UI/メニュー変更
- コマンドオプション変更
- 設定ファイル形式変更
- 依存関係変更
- デフォルト動作変更

## GitHub 操作

GitHub MCP サーバーを使用する。`gh` コマンドは使用禁止。

### プルリクエスト作成

```bash
# 1. コミットとプッシュ
git add -A
git commit -m "Descriptive commit message"
git push origin feature/branch-name
```

```python
# 2. MCP でプルリクエスト作成
mcp_github_create_pull_request(
    owner="owner-name",
    repo="repo-name",
    title="PR title",
    head="feature/branch-name",
    base="main",
    body="## 変更内容\n- 変更点"
)
```

## 関連ガイドライン

| ガイドライン | 参照タイミング |
|-------------|---------------|
| `quality-assurance-guidelines.md` | 品質保証、リリース前確認時 |
| `testing-guidelines.md` | テスト作成・実行時 |
| `aws-cloudformation-guidelines.md` | CFN/SAM テンプレート作成時 |
| `aws-architecture.md` | AWS サービス設計時 |
