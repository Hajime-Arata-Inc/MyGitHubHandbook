# ０５　　小技・便利設定など
> 本番環境でも安心して使える便利ワザをいくつかまとめました。

1. よく使うコマンドを短縮（エイリアス設定）
- ターミナルの ~/.gitconfig に設定すると、長いコマンドを省略できます。
- 例: `git st` → `git status`
```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.cm "commit -m"
```

2. プッシュ前の差分確認
- `git diff --staged` でステージ済みの変更点を確認できます。
- 安全にコミット・プッシュするための最終チェックにおすすめ。

3. 誤コミットを取り消す
- 直前のコミットメッセージを修正したい場合：
`git commit --amend` すでにリモートに push 済みの履歴は書き換えないのが安全。

4. コミットログを見やすくする
- 履歴を1行で表示する便利オプション：
`git log --oneline --graph --decorate`

5. `.gitignore`の活用
- ログや環境ファイルを誤ってコミットしないために必須。
例： Python / Django の例
```bash
__pycache__/
*.pyc
.env
db.sqlite3
6. デフォルトブランチ名を統一
新しいGitでは main が標準ですが、プロジェクトによっては master もあります。
git config --global init.defaultBranch main
7. SSH接続を簡単に
毎回パス入力せずに済むように、ssh-agent に鍵を登録。
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```


---

*注意書き*
- このハンドブックの中のメールアドレス・鍵は例です。実際の情報は絶対に公開しないでください
- 本ハンドブックの内容を利用したことによるいかなるトラブル・損害についても、作者は一切の責任を負いません。
*Note*
- This handbook is for learning purposes only. Do not expose your real emails, keys, or secrets.
*Disclaimer*
- The author assumes no responsibility for any trouble or damage caused by using this handbook.

