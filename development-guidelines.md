---
inclusion: always
---

# 開発ガイドライン

## 言語規則

- ドキュメント・チャット: 日本語
- コードコメント: 英語可
- 変数名・関数名: 英語

## ブランチ戦略

main ブランチへの直接コミットは禁止。

| ブランチ種類 | 命名規則 | 用途 |
|-------------|---------|------|
| main | - | 直接コミット禁止 |
| feature | `feature/<機能名>` | 新機能開発 |
| fix | `fix/<修正内容>` | バグ修正 |
| hotfix | `hotfix/<バージョン>-<修正内容>` | 緊急修正 |

## バージョニング

セマンティックバージョニング（MAJOR.MINOR.PATCH）を採用:
- MAJOR: 互換性のない変更
- MINOR: 後方互換性を保った機能追加
- PATCH: 後方互換性を保ったバグ修正

## タスク実行プロセス

### 実装開始前

1. タスクに「〜に従って」「〜.md に従って」の記載がある場合、そのファイルを先に読む
2. 参照ファイルの要件をチェックリスト化する

### 実装完了前

1. タスクの全指示項目を確認
2. 検証項目（CFN Lint 等）を実行
3. ガイドライン準拠を確認

### CloudFormation/SAM テンプレート作成時

1. `aws-cloudformation-guidelines.md` を参照
2. MCP で AWS ドキュメントを確認
3. CFN Lint で検証（exit code 0）

## 実装変更時のドキュメント更新

コード変更完了直後に README.md を更新する。「後でまとめて」は禁止。

### 更新順序

1. コード実装
2. README.md（即時）
3. 仕様書（requirements.md → design.md → tasks.md）
4. CHANGELOG.md
5. 外部システム（GitHub Actions、Homebrew Formula 等）

### README.md 更新が必要なケース

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

### その他の GitHub 操作

| 操作 | ツール |
|-----|-------|
| Issue 作成 | `mcp_github_issue_write` |
| Issue コメント | `mcp_github_add_issue_comment` |
| PR 更新 | `mcp_github_update_pull_request` |

## 関連ガイドライン

| ガイドライン | 内容 |
|-------------|------|
| `quality-assurance-guidelines.md` | 品質保証、Pre-commit、リリース前確認 |
| `testing-guidelines.md` | テスト戦略、カバレッジ目標 |
| `aws-cloudformation-guidelines.md` | CFN/SAM テンプレート作成 |
| `aws-architecture.md` | AWS サービス設計 |
