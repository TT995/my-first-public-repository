# my-first-public-repository

# Git 強制同期系コマンド集

## origin/main の状態へ完全リセット

ローカル変更をすべて破棄し、GitHub 上の `origin/main` と完全一致させる。

```bash
git fetch origin
git reset --hard origin/main
git clean -fd
```

- staged (`Changes to be committed`) を破棄
- unstaged (`Changes not staged for commit`) を破棄
- untracked file も削除
- ローカルを `origin/main` と完全一致

---

## 作業ブランチを local main に強制追従

現在の作業ブランチへ `main` を取り込み、コンフリクト時は `main` 側を優先する。

```bash
git merge main -X theirs
```

- コンフリクト時に `main` の内容を採用
- 作業ブランチを `main` 寄りに同期

---

## GitHub に push した commit を消す（履歴改変）

直前の commit を消し、その状態を GitHub に強制反映する。

```bash
git reset --hard HEAD~1
git push --force origin main
```

- `HEAD~1`
  = 1 commit 前へ戻る
- GitHub 側の履歴も上書きされる
- 共同開発では危険

### 安全版（履歴を壊さない）

```bash
git revert HEAD
git push origin main
```

- 打ち消し commit を追加
- 履歴は保持される
- チーム開発向け
```
