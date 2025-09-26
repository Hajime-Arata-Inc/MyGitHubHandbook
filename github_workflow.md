# GitHub学習メモ：ブランチを使った基本の開発〜プッシュ手順など

GitとGitHubの基本的な使い方を学ぶための実践メモです。  
安全で見通しの良い開発を行うために、ブランチとGit操作を明確に分けて記録していきます。

## ✅ ブランチ開発の基本フロー

### 1. 今どのブランチか確認
```bash
git branch
```
### ２、作業したいブランチに切り替える（例：study-github）
```bash
git checkout study-github
```

### ３、状況を確認
```bash
git status
```
### ４、ファイルをステージへ追加
```bash
git add .
```
### ５、コミット（メッセージ付きで記録）
```bash
git commit -m "作業内容"
```
### ６、Githubへプッシュ
```bash
git push origin ブランチ名  ＊初回は-uオプションをつけると便利
```
---

### ✅ git status で確認すべきこととその対応一覧

| 状況 | 表示例 | 意味 | 対応方法 |

| 🟥 Untracked files（未追跡ファイル） | `Untracked files: github_workflow.md` | まだGitに登録されていない新規ファイル | `git add ファイル名` でステージに追加 |

| 🟦 Changes not staged for commit（未ステージの変更） | `modified: views.py` | 既存のファイルが変更されたが、まだステージに入っていない | `git add ファイル名` でステージに追加 |

| 🟩 Changes to be committed（コミット予定） | `new file: github_workflow.md` | ステージに追加済みで、次のコミット対象 | `git commit -m "コメント"` でコミット |

| 🔁 混在している場合（変更が複数のブランチにまたがる） | modified: 複数ファイル | 目的が違うファイルが一緒に変更されている | コミットを分ける or ブランチを分ける |

| ❌ 意図しないファイルが変更されている | modified: settings.py（studyブランチなのに） | 間違ったブランチで作業してしまった | `git stash` で一時保存 or `checkout` で正しいブランチに移動 |

---
📌 **解説：**
- `git branch`：どこにいるかを確認
- `git checkout study-github`：必要ならブランチを切り替える
- それからステータスを確認して作業開始！

❌ **もし `main` や `study` になっていたら？**

そのまま作業すると違うブランチに記録されてしまいます。
なので必ず：`git checkout study-github`で切り替えてから作業しましょう。

✅ ブランチの確認が必要になる主な場面

| タイミング                        | 理由  

| 🔄 **作業を始める前**              | 間違ったブランチで編集しないため（`main`に直接書いてしまうのを防ぐ）   

| 💾 **コミットする前**              | どのブランチに変更を記録するか確認するため                   

| 🚀 **プッシュする前**              | 違うブランチに送ってしまうのを防ぐため（GitHubの管理をわかりやすく保つ） 

| 🔀 **マージやPull Requestを作る前** | どのブランチ同士を結合するのか把握するため    


✅ 仕組みの理解

|操作 |結果  |

| `git checkout main` → `git commit` → `git push origin main`                 | `main`だけに変更が記録される         |

| `git checkout study-github` → `git commit` → `git push origin study-github` | `study-github`にだけ変更が反映される |

| ✅ **`git merge study-github` を実行しない限り、mainに影響しない**                  |                                    |



---
## ✅ マージについて
Gitでは、マージ（`merge`）しない限り、別のブランチの変更内容は反映されません。

❌ `.gitignore` の役割とは違う
`.gitignore` は「`Git`で管理したくないファイル・フォルダを指定する仕組み」です。

つまり：
`.env` や `__pycache__`、ログファイル など、そもそもGitに入れたくないファイルを無視するためのもの
特定のブランチの変更を除外するものではない
なので、ブランチ間の変更の反映・除外のコントロールは、マージで管理します。

✍️ マージの実行
たとえば「`study-github`ブランチで実験して、よかったら`main`に取り込む」という場合は：
`git checkout main`
`git merge study-github`
逆に「実験結果を`main`には入れたくない」ときは、そのまま放置でOKです。

'git branch'          # 今いるブランチ確認
'git switch main'     # main に移動（すでに main なら不要）
'git pull origin main'  # 念のため最新を取得
'git merge refactor/template-structure'  # マージ
'git push origin main'  # リモートに反映

vimの操作
操作方法（vimの場合）
そのままメッセージで良い場合：
'Esc' キーを押す
':wq' と入力して Enter
→ 保存して終了（マージ完了）
マージをやめたい場合：
'Esc' キーを押す
':q!' と入力して Enter
→ 保存せず終了（マージ中断）

## ６月２４日（火）１１：１３コミットとプッシュの実行の履歴
- 状況：`git status`をすると`main`関連のファイルと`study-github`関連のファイルが混在していた（下部に主力結果添付）
- 対応：  ２つのブランチ（`main`,`study-github`）に分けてコミットプッシュ

🥇 ステップ①：`study-github` に `github_workflow.md` だけコミット
```bash
            git checkout study-github
            git status
            git add github_workflow.md
            git commit -m "Git学習用メモを追加"
            git push -u origin study-github（初回だけ　このブランチをGitHub（origin）の study-github と紐づける）
```
🥈 ステップ②：残りの変更（本編コード）は`main`ブランチへ切り替えて反映
```bash
            git checkout main
            （任意）git merge study-github  # 必要なら（※メモなので不要なことが多い）
            git status
            git add .
            git commit -m "viewsやテンプレートなどの修正"
            git push origin main
```

- （出力結果）
```bash
(venv_private_diary) HJ-MacBook-Pro:private_diary takedahajime$ git status
On branch study-github
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   config/settings.py
	modified:   config/urls.py
	modified:   diary/models.py
	modified:   diary/templates/base.html
	modified:   diary/views.py

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	diary/migrations/0002_alter_diary_content_alter_diary_title.py
	diary/migrations/0003_diary_user.py
	diary/templates/registration/
	github_workflow.md
```

- 実行結果１：　
```bash
(venv_private_diary) HJ-MacBook-Pro:private_diary takedahajime$ git checkout study-github
M	config/settings.py
M	config/urls.py
M	diary/models.py
M	diary/templates/base.html
M	diary/views.py
Already on 'study-github'
```
    🔄 今のたけださんの状態を日本語でまとめると：

    たけださんは現在 study-github ブランチにいます。
    そのブランチ上で、以下のファイルに変更がありますが、まだステージには追加されていません（git add していない）。
        🧠 状態の意味まとめ
        | 表示         | 状態            | 説明                                |
        | ---------- | ------------- | --------------------------------- |
        | `M`（左側に出る） | **Modified**  | 変更されたがまだ `git add` されていない（＝ステージ外） |
        | `M`（右側に出る） | **Staged**    | 変更されて、`git add` 済み（＝ステージ内）        |
        | `A`        | **Added**     | 新しく追加されたファイル                      |
        | `D`        | **Deleted**   | 削除されたファイル                         |
        | `??`       | **Untracked** | Gitにまだ登録されていない新ファイル（`git add` 必要） |

- 実行結果２：　
```bash
(venv_private_diary) HJ-MacBook-Pro:private_diary takedahajime$ git add github_workflow.md
```
- 実行結果３：　
```bash
(venv_private_diary) HJ-MacBook-Pro:private_diary takedahajime$ git commit -m "study-githubブランチの作成と学習開始"
[study-github 8530bac] study-githubブランチの作成と学習開始
 1 file changed, 126 insertions(+)
 create mode 100644 github_workflow.md
```
- 実行結果４：　
```bash
(venv_private_diary) HJ-MacBook-Pro:private_diary takedahajime$ git push -u origin study-github　省略
```
- 実行結果５：　
```bash
(venv_private_diary) HJ-MacBook-Pro:private_diary takedahajime$ git push
Everything up-to-date
```
- 実行結果６：　
```bash
(venv_private_diary) HJ-MacBook-Pro:private_diary takedahajime$ git push
Everything up-to-date
```
- 実行結果７：　
```bash
(venv_private_diary) HJ-MacBook-Pro:private_diary takedahajime$ git checkout main
M	config/settings.py
M	config/urls.py
M	diary/models.py
M	diary/templates/base.html
M	diary/views.py
Switched to branch 'main'
Your branch is up to date with 'origin/main'.
```
- 実行結果８：　
```bash
(venv_private_diary) HJ-MacBook-Pro:private_diary takedahajime$ git add .
```
- 実行結果９：　
```bash
(venv_private_diary) HJ-MacBook-Pro:private_diary takedahajime$ git commit -m "設定ファイルやビューを修正"
[main 58d657d] 設定ファイルやビューを修正
 11 files changed, 86 insertions(+), 25 deletions(-)
 create mode 100644 diary/migrations/0002_alter_diary_content_alter_diary_title.py
 create mode 100644 diary/migrations/0003_diary_user.py
 create mode 100644 diary/templates/registration/logged_out.html
 create mode 100644 diary/templates/registration/login.html
 create mode 100644 diary/templates/registration/password_change_done.html
 create mode 100644 diary/templates/registration/password_change_form.html
 ```
- 実行結果10：　
```bash
(venv_private_diary) HJ-MacBook-Pro:private_diary takedahajime$ git push origin main
Enumerating objects: 26, done.
Counting objects: 100% (26/26), done.
Delta compression using up to 8 threads
Compressing objects: 100% (16/16), done.
Writing objects: 100% (16/16), 2.86 KiB | 1.43 MiB/s, done.
Total 16 (delta 7), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (7/7), completed with 7 local objects.
To https://github.com/Hajime-Arata-Inc/private_diary.git
   5c15516..58d657d  main -> main
```

---
## 🎯 UI整理のヒント（5つのポイント）

**① ✅ 見出しを整理（階層的に）**
- `#` はタイトル（1つだけ）
- `##` は大きな章（例：基本フロー / git status の確認）
- `###` で小さな手順

**② 🗂️ ステップは番号と箇条書きを組み合わせて明確に**
- `### 3. ファイルをステージへ追加`
- コマンド：`git add ファイル名`
- すべて追加：`git add .`


**③ 📊 表は枠線を付けて整える**

| 状況 | 表示例 | 意味 | 対応方法 |
|------|--------|------|-----------|


**④ 🎨 コードブロックの中も整える**
`git status`
`git add .`


## ✅ bash 表記をつけるとGitHubで色が付きます（これはオムニも使ってます）


### ⑤ 💡 見出しの順番を見直す（おすすめ構成）

```markdown
# GitHubワークフロー学習メモ

## 1. はじめに（このファイルの目的）

## 2. 基本の開発〜プッシュ手順

## 3. git status の使い方と対応パターン

## 4. よくあるミスとその対処法（←新しく追加しても良い）

## 5. その他Tips（stashの使い方などを追加可能）
```
✨ 追加アイデア（オプション）

**✅ や ❌ の絵文字でチェックリスト感を出す**
**重要な部分** は太字にする
最後に「参考リンク（あれば）」を付けるのもプロっぽい


✅ たけださんへのアドバイス

Markdown学習メモでは：

実行手順や複数コマンドは ```bash を使ってブロックでまとめる
説明文の中で軽く触れるときは コード を使って補足する
この使い分けをするだけで、📘メモの見やすさ・伝わりやすさがグンと上がります！

```bash
git status

On branch refactor/template-structure
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	deleted:    CREATE
	modified:   config/settings.py
	modified:   config/urls.py
	modified:   diary/migrations/0001_initial.py
	deleted:    diary/migrations/0002_alter_diary_content_alter_diary_title.py
	deleted:    diary/migrations/0003_diary_user.py
	modified:   diary/models.py
	deleted:    diary/templates/registration/logged_out.html
	deleted:    diary/templates/registration/login.html
	deleted:    diary/templates/registration/password_change_done.html
	deleted:    diary/templates/registration/password_change_form.html
	modified:   diary/views.py

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	diary/migrations/0002_diary_updated_at.py
	diary/templates/diary/signup.html
	private_diary/

no changes added to commit (use "git add" and/or "git commit -a")


#上記の場合どのようにプッシュしていくか手順を示して欲しい
```
✅ 状況整理
① 変更はあるがステージングしていないファイル

例：config/settings.py, diary/models.py（modified）
例：CREATE, 各種テンプレートなど（deleted）
② 新しく作ったファイル（Untracked files）

例：diary/migrations/0002_diary_updated_at.py
例：diary/templates/diary/signup.html
③ まだコミットもされていないし、当然プッシュもしていない状態

🔧 プッシュまでの手順

🔹ステップ1：内容を確認する（※慎重に）
`git diff`
※ どのような変更をしたか全体を確認できます。

🔹ステップ2：現在のブランチを確認する
`git branch`

🔹ステップ3：必要なファイルをステージングする
`git add .`
（add .は全ての変更を一括ステージングします）

🔹ステップ4：コミットする
`git commit -m "テンプレート構造のリファクタリングとマイグレーション修正"`

🔹ステップ5：現在のブランチ（refactor/template-structure）にプッシュ
`git push origin refactor/template-structure`


















