# 01_commit_push 着実なコミットと安全なプッシュ
- 目的: Git と GitHub を安全に使い、間違ったブランチへのコミットや機密情報の漏洩を防ぎながら、見通しよく開発・プッシュするための実践メモ。
- 対象: macOS / ターミナル前提。既存リポジトリが main で運用されている想定しています。
- branchを作成した場合は[1-1](いまどこのブランチ？)を参照してください。

**プッシュ前の5秒チェック**
***毎回チェックすることなのでここに書いておきます。***
- ブランチ名 → `git branch`
- ステージ内容 → `git status`
- 差分 → `git diff --staged`
- 秘密ファイル → チェック済み？
- リモート先 → `origin`/`branch`

## 0-0. (重要)情報漏洩防止のために
*機密の非公開化：.env、APIキー、個人情報、秘密鍵はコミットしない*
*例えばDjangoなどで自動生成されるSEACRET_KEYは`.env`で管理して公開リポジトリにpushしないこと。*
- .gitignore に最低限以下を追加
```
.DS_Store
.env
*.env
.vscode/
.idea/
*.log
```
*コミット前に確認すると安心*
```
git status
git ls-files | grep -E '\.env|id_rsa|id_ed25519|\.pem' || echo "OK: 機密ファイルの追跡なし"
```
*誤ってコミットした場合*
- 履歴からの完全削除を検討する（`git filter-repo` `git filter-branch`）
- SECRET_KEYを即時ローテーションする。

## 0-1. よくあるトラブル
`Permission denied (publickey)` → SSH 鍵の登録漏れ／`ssh-add` 未実行。 → ステップ1を再確認。
`Repository not found` → リポジトリURLや権限を確認（組織内権限／スペルミス）。
`non-fast-forward` push時にエラー。これはつまりリモートに別の更新があってローカルに取り込んでないということ。→ 先にローカルにリモートの更新を取り込んでその後ローカルをpushする
```
git fetch origin
git rebase origin/main     # 例：main を取り込む場合
# 競合解消→続行
git push
```

## 0-2. 設定（初回だけです）
`git config --global user.name  "Your User Name"`
`git config --global user.email "you@example.com"`
`git config --global init.defaultBranch main`
`git config --global core.editor "code --wait"`
**vim・nanoの選択**
`git config --global core.editor "nano"`

## 1. ブランチ開発の基本フロー
### 1-1. 今どのブランチ？(これは結構重要です。)
`git branch`
`git status`
## 1-2. 作業ブランチへ移動
`git switch study-github`
**新規作成の場合**
`git switch -c study-github`
### 1-3. 変更→ステージ→コミット
`git add path/to/file1 path/to/file2`
`git add -p`
`git commit -m "feat: ◯◯を追加 / fix: △△を修正"`
### 1-4. GitHubへプッシュ
`git push -u origin study-github`
**2回目以降**
`git push`

## 2. git status チェックリスト
|表示|意味|やること|
|:---------|:----------:|:------------:|
|`Untracked files`	|新規ファイル（未追跡）	|`git add ファイル名` or `.gitignore`|
|`Changes not staged`	|変更したが未ステージ	|`git add`|
|`Changes to be committed`	|ステージ済み	|`git commit`|
|混在	|意図違いの変更が混ざる	|コミットを分ける|
|誤ブランチ	|枝違いで変更	|`git stash` → 正しいブランチへ|

## 3. よくあるミスと対処
- 誤ブランチで編集 → `git stash -u → git switch → git stash pop`
- 入れすぎた → `git restore --staged ファイル`
- 直前コミットを修正 → `git commit --amend`
- 直前コミットを取り消し → `git reset --soft HEAD~1`

## 4. マージと Pull Request
- CLIでマージ
`git switch main`
`git pull --ff-only`
`git merge study-github`
`git push origin main`
- GitHub画面でのpull request
`Compare & pull request`
`Merge pull request` → Confirm
`Delete branch`　→ mainにマージされたのでbranchは不要になります。（任意）

## 5. セキュリティ & .gitignore
- プッシュ前チェック
`git ls-files | grep -E '^\\.env$' || echo "OK"` → これで「OK」なら.envは追跡されていないということです。（pushされないということ。）
`git diff --staged | egrep -i "secret|password|api[_-]?key|token|PRIVATE KEY" || echo "OK"`　→　これで「OK」ならとくに問題ありません。（pushしないように設定しているものが含まれていないということ）
- Django用 .gitignore
```
.DS_Store
__pycache__/
*.py[cod]
venv*/
.env
*.sqlite3
```

## 6. ケーススタディ（`git status`が混在していた場合）
- `git diff` で削除/変更内容を確認
- 誤削除なら `git restore`
- 意図通りなら `git add -A`
- `Untracked` → 必要なら `git add`、不要なら `.gitignore` + `git clean -fdn`
- 意味ごとに分割コミット推奨

## 7. 便利コマンド早見表
```
git add -p
git restore --staged file
git restore --worktree file
git reset --soft HEAD~1
git push --force-with-lease
```

## 8. ブランチ命名 & コミット規約
- feature/◯◯ 新機能
- fix/◯◯ バグ修正
- refactor/◯◯ 整理
- study/◯◯ 学習用

## 9. vim が開いたとき(初心者の時はいきなり開いて焦ります)
- 保存終了: Esc → :wq
- 中断: Esc → :q!

---

# Github（リモート）とパソコンのフォルダ（ローカル）をつなぐ方法
- リモートとローカルをつなげる場合、以下の二つのバターンが考えられます。その２パターンのケースに分けてリモートとローカルをつなげる方法を記載します。
１、Github（リモート）のリポジトリを先に作った。
２、パソコンでプロジェクトなどのフォルダ（ローカル）を先につくった。

## １、Github（リモート）のリポジトリを先に作った時にローカルのフォルダと繋ぐ方法

### 目的
- リモート（GitHub）からローカルへクローンし、ブランチを作って安全に push できる状態にする。

### 前提
- macOS / ターミナルが使える
- Git が入っている（`git --version` で確認）
- GitHubのアカウントがある

### ステップ1：SSH を準備する(推奨です。)

#### 1-1. 鍵があるか確認
`ls -al ~/.ssh`　鍵が無ければ鍵を作成
#### 1-2. 鍵を作成（無い場合）
`ssh-keygen -t ed25519 -C "（GitHubのメール or 任意ラベル）"`
- 保存先は Enter（デフォルトの鍵名でOK）
- パスフレーズは設定推奨（盗難対策）
#### 1-3. `ssh-agent`を起動し鍵を登録
- agent起動
`eval "$(ssh-agent -s)"`
- macOS 推奨設定（自動登録＆キーチェーン連携）
`ls -al ~/.ssh`
`printf "%s\n" "Host *" "  AddKeysToAgent yes" "  UseKeychain yes" "  IdentityFile ~/.ssh/<秘密鍵ファイル名>" >> ~/.ssh/config`
- 鍵を agent に読み込み（<秘密鍵ファイル名>を実名に置換）
`ssh-add ~/.ssh/<秘密鍵ファイル名>`

#### 1-4. 公開鍵を GitHub に登録
- もっとも新しい公開鍵をクリップボードへ（.pub を自動検出）
`PUBLIC_KEY="$(ls -t ~/.ssh/*.pub | head -n1)"; echo "Using: $PUBLIC_KEY"; pbcopy < "$PUBLIC_KEY"`
- GitHub(Web) → 右上アイコン → Settings → SSH and GPG keys → New SSH key → 貼り付け → Add SSH key

#### 1-5. 接続テスト
`ssh -T git@github.com` "Hi <ユーザー名>! You've successfully authenticated..." が出ればOK

#### 1-6. リモートURLがHTTPSならSSHへ切替
```
git remote -v
git remote set-url origin git@github.com:<USERNAME>/<REPOSITRY>.git
```

### ステップ2：リポジトリをクローン
```
mkdir -p ~/Projects && cd ~/Projects
git clone git@github.com:YOURGITHUBUSERNAME/REPOSITRYNAME.git
cd REPOSITRYNAME
git remote -v   #originのURL確認
```

### ステップ3：Git の本人情報(このリポジトリの設定)
```
git config user.name  "YOUR NAME"
git config user.email "YOUR_GitHub_EMAIL"
git config init.defaultBranch main
git config --local -l  # 設定の確認
```

### ステップ4：ブランチを作って作業(もしブランチをつくって作業するならこちら) & 初回プッシュ
```
git checkout -b BRANCH_NAME(例_study-github　← 学習用ブランチ)
git status
git add .　
git commit -m "コメントをいれるならココに入力。例_初回プッシュ"

# 初回プッシュは -u で upstream 設定
git push -u origin study-github
```
- 2回目以降のプッシュはこちら
```
git add . （「.」は変更されたファイルをすべてコミットする時に使用する。例えばREADME.mdだけコミットしたいなら「.」ではなく「README.md」と記入する）
git commit -m "ココに更新内容を記入しておく（例_Update README.md）" 
git push
```

## ２、ローカルのフォルダを作成した後にGithub（リモート）とつなぐ方法

### 目的
- ローカルの作業ディレクトリから GitHubに新規リポジトリを作成して接続し、`main` ブランチを初回 push できる状態にする。

### 前提
- macOS / ターミナル使用可
- Git インストール済み（`git --version`）
- GitHub アカウント（組織：`Hajime-Arata-Inc`）

### ステップ1：ローカルを Git リポジトリ化 & 初回コミット

#### 1-1. 対象フォルダへ移動 & 初期化
```
cd ~/YOURFOLDER #作業フォルダへ移動   
git init #初期化
```

#### 1-2 `.gitignore` を先に用意（機密/不要ファイルを除外）
```
cat <<'EOF' >> .gitignore
# macOS
.DS_Store

# 機密・環境ファイル
.env
*.env
*.pem
*.key
*.p12

# エディタ/IDE
.vscode/
.idea/

# 一時ファイル/ログ
*.log
*.tmp
~$*
EOF
```

#### 1-3. 最初のコミット
```
git add .
git commit -m "first commit(任意のコメントを記入)"
```

#### 1-4. 既定ブランチ名を main に（必要な場合）
- 既に main ならスキップ可
'git branch -M main' 

### ステップ2：GitHubで空のリポジトリを作成（Web UI）
- GitHub 右上 + → New repository
- Owner：YOURGITHUBNAME を選択
- Repository nameを作成
- 公開範囲：Private or Pubric
- Initialize のチェック
- Create repository をクリック

### ステップ3：ローカルにリモート `origin` を紐づけ
#### 3-1. SSH（推奨）
- `ls -al ~/.ssh` SSH鍵未設定なら（1回だけ）
- 無ければ鍵を作成する
```
# 鍵を作成（Ed25519方式）
ssh-keygen -t ed25519 -C "YOURGITHUBEMAIL"

# ssh-agent を起動して鍵を登録
eval "$(ssh-agent -s)"; ssh-add ~/.ssh/id_ed25519

# 公開鍵をクリップボードへコピー（macOS）
pbcopy < ~/.ssh/id_ed25519.pub
# GitHub > Settings > SSH and GPG keys > New SSH key > 貼り付け

# リモート追加
git remote add origin git@github.com:YOURNAME/YOURRIPOSITRY.git
git remote -v
```

#### 3-2 HTTPS（代替）
```
git remote add origin https://github.com/YOURGITHUBNAME/YOURREPOSITRY.git
git remote -v
# push 時にパスワードの代わりに Personal Access Token を入力（macOS はキーチェーンに保存可）
```

### ステップ4：初回 push（upstream 設定を一度で）
`git push -u origin main` 
- -u は「このブランチの上流＝origin/main」を記憶。以後は `git push` だけでOK

*注意書き*
- このハンドブックの中のメールアドレス・鍵は例です。実際の情報は絶対に公開しないでください
- 本ハンドブックの内容を利用したことによるいかなるトラブル・損害についても、作者は一切の責任を負いません。
*Note*
- This handbook is for learning purposes only. Do not expose your real emails, keys, or secrets.
*Disclaimer*
- The author assumes no responsibility for any trouble or damage caused by using this handbook.


