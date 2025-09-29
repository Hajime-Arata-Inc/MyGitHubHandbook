# 04 よくあるエラーと対処法

1. `fatal: not a git repository (or any of the parent directories): .git`
- 意味: ここは Git 管理下ではない。
- 対処: `git init` でリポジトリを作成する。または `.git` がある正しいディレクトリに移動する。

2. `error: failed to push some refs to ...`
- 意味: リモートとローカルの履歴が食い違っている。
- 対処: `git pull --rebase origin main` でローカルを更新してから再プッシュ。

3. `Merge conflict in ...`
- 意味: マージ時に同じ部分を別々に変更 → 自動で統合できない。
- 対処: 対象ファイルを開いて `<<<<<<< 〜 >>>>>>>` を手動で解決 → `git add → git commit` で修正を確定。

4. `Updates were rejected because the remote contains work that you do not have locally.`
- 意味: リモートにあるコミットがローカルに取り込まれていない。
- 対処: `git pull --rebase origin main` 競合があれば解決 → `git push`

5. `detached HEAD` 状態
- 意味: ブランチではなく特定のコミットに直接チェックアウトしている状態。
- 対処: 新しいブランチを作成：`git switch -c 修正用ブランチ名`

6. `fatal: remote origin already exists`
- 意味: すでに`origin` が登録されている。
- 対処: 確認：`git remote -v` → 修正： `git remote set-url origin git@github.com:USER/REPO.git`

7. `Permission denied (publickey).`
- 意味: SSH鍵の設定が GitHub と一致していない
- 対処: `ssh -T git@github.com` で確認 → 必要なら新しいSSH鍵を作成してGitHubに登録。

8. `Repository not found`
- 原因: リポジトリURLのスペルミス。アクセス権限不足（特に組織リポジトリ）。
- 対処: URLを確認：`git remote -v` → 必要なら正しいURLに修正 → `git remote set-url origin git@github.com:USER/REPO.git` → 組織リポジトリならアクセス権を確認

9. `non-fast-forward`
- 意味: pushしようとしたが、リモートにローカルが持っていない更新がある。
- 対処: 先にリモートの更新を取り込んでからpush。
```bash
git fetch origin
git rebase origin/main     # 例：main を取り込む場合
# 競合があれば解消して続行
git push
```

*注意書き*
- このハンドブックの中のメールアドレス・鍵は例です。実際の情報は絶対に公開しないでください
- 本ハンドブックの内容を利用したことによるいかなるトラブル・損害についても、作者は一切の責任を負いません。
*Note*
- This handbook is for learning purposes only. Do not expose your real emails, keys, or secrets.
*Disclaimer*
- The author assumes no responsibility for any trouble or damage caused by using this handbook.

