# ０５　　小技・便利設定など
> 本番環境でも安心して使える便利ワザをいくつかまとめました。

<h1>設定エイリアス系</h1>

1. よく使うコマンドを短縮（エイリアス設定）
ターミナルの `~/.gitconfig` に設定するとコマンドを短縮できる。
例: `git st` → `git status`
```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.cm "commit -m"
```

2. デフォルトブランチ名を統一
Gitでは `main` が標準ですが、プロジェクトによっては `master`もあります。混乱を避けたい時はブランチを統一する。
`git config --global init.defaultBranch main`

3. SSH接続を簡単に
毎回パス入力せずに済むように、`ssh-agent` に鍵を登録する。
これで毎回パスワードを入力せずにpush/pullが実行できる。
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

<h1>ブランチ・リモート管理系</h1>

1. ローカルで不要なブランチを一掃する。
```bash
git fetch --prune
git branch --merged | grep -v 'main' | xargs git branch -d
```

2. リモートとローカルを整理整頓（整理・同期）するコマンド一覧
- リモートで削除されたブランチをローカルでも削除する。
`git fetch --prune`
- ローカルとリモートの全ブランチ一覧を確認する.
`git branch -a`
- ローカルブランチをリモートの最新状態に更新する.
`git pull` 
ちなみに`git pull`＝`git fetch + git merge`のショートカットです。
`main`ブランチでは`git pull --ff-only`を使うとより安全です。

<h1>履歴整理・コミット系</h1>

1. プッシュ前の差分確認
`git diff --staged` でステージ済みの変更点を確認できます。
安全にコミット・プッシュするための最終チェックにおすすめ。

2. コミットログを見やすくする
履歴を1行で表示することができる。
`git log --oneline --graph --decorate`

3. 複数の細かいコミットをひとつにまとめ履歴をきれいに保つ。
- 例：直接３コミットを対象に編集
`git rebase -i HEAD~3`
 - 整理前
```bash
    pick a1b2c3 修正A  
    pick d4e5f6 修正B
    pick g7h8i9 修正C
```
 - ２行目以降の`pick`を`squash`に変更する。
```bash
    pick a1b2c3 修正A
    squash d4e5f6 修正B
    squash g7h8i9 修正C
```
 - エディタが切り替わるのでコミットメッセージをまとめる。
  - まずは以下のようなこの画面が表示される
```txt
# This is a combination of 3 commits.
# The first commit's message is:
修正A

# The following commit messages will also be included:
修正B
修正C

# Please enter the commit message for your changes.
```
  - 残したいメッセージへ編集
```txt
機能Xの追加と細かい修正をまとめた
```
  - 完了後ログを見る。`git log --oneline`
```txt
h1i2j3 機能Xの追加と細かい修正をまとめた
```

<h1>ファイル・作業系</h1>

1. 過去のある時点に戻る方法
- 直前のコミットメッセージを修正したいとき。（ファイルの追加を忘れた時など）
`git commit --amend` 
メッセージ修正も修正できます。
⚠️**注意**: すでにリモートに`push`済みの履歴は書き換えないのが安全。
- 一時的に過去の状態で確認したいとき。
`git checkout <コミットID>`
作業が終了したら`git switch ブランチ名`で戻ることができます。
- 間違えて消したファイルを復元する。
`git checkout HEAD -- ファイル名`
これで直前のコミット時点の状態に戻せます。
- 間違えて`add .`をした場合（ステージから一部を外す）
`git reset HEAD ファイル名`
- 作業を一時退避（`stash`）
作業中に別ブランチへ移動したい時に便利です。
```bash
git stash
git switch main
# 用が済んだら戻す
git stash pop
```
- コミットとリモートの差分を確認する。
`git diff origin/main`
- 誰がどの行を変更したが調べる。
`git blame ファイル名`













---

*注意書き*
- このハンドブックの中のメールアドレス・鍵は例です。実際の情報は絶対に公開しないでください
- 本ハンドブックの内容を利用したことによるいかなるトラブル・損害についても、作者は一切の責任を負いません。
*Note*
- This handbook is for learning purposes only. Do not expose your real emails, keys, or secrets.
*Disclaimer*
- The author assumes no responsibility for any trouble or damage caused by using this handbook.

