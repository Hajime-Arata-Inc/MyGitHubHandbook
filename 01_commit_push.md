# 01_commit_push 着実なコミットと安全なプッシュ
>目的: Git と GitHub を安全に使い、間違ったブランチへのコミットや機密情報の漏洩を防ぎながら、見通しよく開発・プッシュするための実践メモ。

---

# 1 コミット

## 1-1 コミットとは
- コミットとはローカルの変更を「スナップショット」として保存する操作のことです。  
- 一定の作業が終わったら、コメント（メッセージ）を付けて記録します。  
- コミットはあくまで 自分のPC内（ローカルリポジトリ）に保存されるだけでGitHubにはまだ反映されません。

## 1-2 コミットの方法
1. 状況を確認  
`git status`
- `nothing to commit, working tree clean` → 問題なし（次へ進む）
- `Untracked files` → ファイルが追跡対象になってない → `git add <ファイル名>`で追跡開始。
- `Changes not staged` → 変更はあるが未ステージ（＝`git add`をしてステージにのせてコミットの準備をする）
- `Changes to be committed` → コミット準備OK。コミットに進む。
2. ファイルをステージへ追加
`git add ファイル名` ファイルを限定して追加　
`git add .` カレント配下すべてのファイル（.gitignoreに登録されているファイルは除く）。
3. コミットする（コメント付きで記録する）
`git commit -m "作業内容を簡潔に書く"`
**コメントについて**
- 小さく、意味単位で。
    - 1 コミット = 1 目的。混在は `git add -p` で分割。
    - `feat:` 機能追加 / `fix:` バグ修正 / `docs:` ドキュメント / `refactor:` リファクタ
    - `chore:` 付帯 / `test:` テスト / `perf:` 性能 / `revert:` 差し戻し
    - 例：`fix: prevent double-submit on signup`

---

# 2 プッシュ

## 2-1 プッシュとは
- プッシュとは、ローカルリポジトリのコミットを GitHub（リモートリポジトリ） に送信して反映させる操作のことです。
- プッシュしないと、GitHub側には自分の変更が見えません。
- チーム開発や公開用では必須のステップです。
- GitHubのリポジトリが**Public**の時は情報漏洩の防止策を施してください。初心者はPreivateを推奨。

## 2-2 プッシュの方法
1. ブランチの確認（結構重要です）
`git branch`
2. プッシュする
- 初回だけ
`git push -u origin ブランチ名`
- 2回目以降
`git push origin ブランチ名`

## 2-3 プッシュ前の５秒チェック
- ブランチ名 → `git branch`
- ステージ内容 → `git status`
- 差分 → `git diff --staged`
- 秘密ファイル → チェック済み？
- リモート先 → `origin`/`branch`

---


*注意書き*
- このハンドブックの中のメールアドレス・鍵は例です。実際の情報は絶対に公開しないでください
- 本ハンドブックの内容を利用したことによるいかなるトラブル・損害についても、作者は一切の責任を負いません。
*Note*
- This handbook is for learning purposes only. Do not expose your real emails, keys, or secrets.
*Disclaimer*
- The author assumes no responsibility for any trouble or damage caused by using this handbook.
