---
inclusion: always
---

# Git 操作ガイドライン

## 破壊的操作の禁止

コミットされていないローカルの変更を含むディレクトリやファイルを削除してはならない。

### 禁止コマンド

| コマンド | 理由 |
|---------|------|
| `rm -rf [directory]` | Git 管理下のディレクトリの強制削除 |
| `git reset --hard` | 作業ディレクトリの変更を破棄 |
| `git clean -fd` | 未追跡ファイルの削除 |
| `git reset --hard origin/main` | ローカル変更の上書き |

`git reset --hard` と `git clean -fd` は `git status` で状態確認後のみ実行可。

## 変更を元に戻す手順

誤った操作をした場合、以下の順序で対応する：

1. `git status` で現在の状態を確認
2. ローカルの変更を `git stash` または別ブランチに保存
3. 誤った変更のみを取り消す（`git revert` または `strReplace` で修正）
4. 必要に応じて `git stash pop` で復元

不明な場合はユーザーに確認する。慌てて作業しない。

## サブモジュール操作

### 操作前の確認手順

```bash
cd [submodule-path]
git status
git remote -v
```

未コミットの変更がある場合は、コミットしてリモートに Push してから操作する。

### 禁止事項

- `rm -rf` でサブモジュールディレクトリを削除してはならない
- ローカルの変更が完全に失われる

## 操作前チェックリスト

### 破壊的操作前

- [ ] `git status` で状態確認
- [ ] 未コミットの変更がないか確認
- [ ] 操作の影響範囲を理解している

### サブモジュール操作前

- [ ] サブモジュール内: `cd [submodule] && git status`
- [ ] 親リポジトリ: `cd .. && git status`
- [ ] `.gitmodules` の内容確認
- [ ] 変更はリモートに Push 済み

## 関連ガイドライン

ブランチ戦略・ワークフローは `development-guidelines.md` を参照。
