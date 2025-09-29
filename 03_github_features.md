# 03. GitHubの主な機能ガイド（Code / Issues / Pull Requests / Actions / Projects / Security / Insight）
> 目的：**GitHub の画面のどこで何ができるか**を理解しましょう。

## 1. Codeとは
> GitHubリポジトリの中身（ソースコードやフォルダ構成）を表示する場所です。プロジェクト全体を見たり取得したりするための基本画面です。
**できること**
1. ファイル一覧を確認
- ソースコード、README.md、ディレクトリ構成などがブラウザで見られる。
- ファイルをクリックすると中身を閲覧可能。
2. ブランチの切り替え
- 左上にある 「main ▼」 から他のブランチへ切り替え。
- ここで新しいブランチも作成可能。
3. コードをダウンロード / クローン
- 緑色の 「Code」ボタン を押すと表示されるメニューで：
    - HTTPS や SSH のURLをコピーして git clone に使う
    - Download ZIP を押してそのままPCに保存する
4. 最新のコミット情報を確認
- ファイル一覧の上部に「最新のコミットメッセージ」が表示される。
- クリックすると詳細（誰が・いつ・何を変更したか）が見られる。

## 2. Issuesとは
> GitHub上で 「課題・タスク・アイデア」 を記録・共有するための仕組みです。いわば 「やることリスト」 をチーム全員が見える場所に置くイメージです。
**使用例**
- バグ報告　例：ログインできない。
- 新機能の提案　例：検索機能を追加したい。
- 改善リクエスト　例：UIをもっと見やすく。
- 学習ログ　例：Djangoのフォーム勉強用Issueを作る。
**入力する主な項目**
- `Title`：一行で目的を明確にする。例：`ログイン画面：フォームバリデーション追加`。
- `Description`：詳細を記入。再現手順・期待結果・スクショ・チェックリスト。
- `Assignees`：担当者。自分や他の人を指定。
- `Labels`：分類（`bug` / `feature` / `docs` / `urgent`など）
- `Projects`：カンバンボードに紐づけしたい時。
**便利な使い方**
* コミットメッセージ/PR本文で `Fixes #12` と書くと、マージ時にIssue #12が自動でClose。
* 検索バーで `is:issue is:open assignee:@me` など絞り込み可能。

## ３. Pull requestsとは
> GitHubで 「自分の変更を他の人に提案して、確認・レビューしてもらうための仕組み」 です。
**作り方（基本）**
1. `main` からブランチを切って作業（例：`feature/login-form`）
2. 編集した内容をコミット → GitHubにプッシュ
3. `Pull requests` タブで New pull request
    - base（取り込み先）：main
    - compare（作業ブランチ）：feature/login-form
4. 差分（何を変えたか）を確認 → タイトル・説明を入力
5. レビュアーを指定 → 承認されたら Merge
**Pull requests本文に書くこと**
- 背景 / 目的（なぜ変更したのか）
- 変更内容（何を追加・修正したか）
- 確認方法（どの画面を開けば確認できるか）
- 関連Issue（例：Fixes #12）
**レビューの観点**
- 影響範囲（他機能を壊さないか）
- セキュリティ（秘密情報の混入、認可、入力検証）
- 可読性（命名・コメント・テストの有無）
- Draft pull request：作業途中で相談したいときは Create draft pull request。

## 3. Actionsとは
> GitHub上で 自動的に処理を実行してくれる仕組み です。CI/CD（継続的インテグレーション / 継続的デリバリー）を簡単に実現できます。
**Actionsの目的**
- コードをプッシュするたびに自動でテスト → 壊れたコードがmainに入らない。
- チーム開発で「レビュー前に必ずCIを通す」というルールの作成が可能。
- 本番運用前に品質を確認できる。
**具体例**
- コードをプッシュしたら → 自動でテストが走る（`pytest` や `npm test` を自動で実行）
- Pull Requestが作られたら → Lint/フォーマットチェック：`flake8` / `eslint`
- mainにマージしたら → AWS・Vercel・Herokuなどに自動デプロイ
- セキュリティ：依存パッケージの脆弱性チェック（Dependabotと連携）
- 実例（pythonのテスト）　
> Actionsのワークフローファイル（ci.yml）。GitHubにプッシュやPull requestをしたときに自動でPython環境を作ってテストを実行する設定になってます。
```
name: CI
on: [push, pull_request] # このワークフローが動くタイミング
jobs:
  test:
    runs-on: ubuntu-latest # 実行環境(GitHubが用意するubuntu環境)
    steps:
      - uses: actions/checkout@v4 # ソースコードをチェックアウト
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12' # Python 3.12 の環境をインストールしてセットアップ
      - name: Install deps
        run: pip install -r requirements.txt # 依存ライブラリをインストール
      - name: Run tests
        run: pytest -q # quiet（出力を最小化）
```
**用語**
- `Workflow`：自動化の単位（YAML）。`.github/workflows/*.yml`に定義する。
- `Job` :Workflowの中の大きな処理単位。
- `Step` :Jobの中の細かい処理。シェルコマンドやアクションを実行。
- `Runner`：Workflowを実行するマシン。GitHub提供のLinux/Windows/Macか、自前のサーバー。
**セキュリティのポイント**
- 秘密情報（APIキーやパスワード）はリポジトリに直書きせず、Settings > Secrets and variables > Actions に登録する.
- YAMLファイルからは `${{ secrets.MY_SECRET }}` のように呼び出す。

## 4. Projectsとは
> GitHub上で使える プロジェクト管理ツール（カンバン方式やタスクボード） です。チームや個人がやること（Issues / Pull request）を見える化して整理できます。
**基本操作**
1. New Project をクリック
2. テンプレートを選択（Board / Table / Roadmap）
    - Board：Todo → In Progress → Done のようなカンバン形式
    - Table：スプレッドシート風に並べ替え可能
    - Roadmap：スケジュールやタイムラインで管理
3. IssueやPRをカード化してドラッグ
    - 例：Issue「ログイン画面を作る」を In Progress に移動
    - 作業完了したら Done に移す
4. フィールドを追加して管理
    - 期日（Due date）
    - 優先度（Priority）
    - 見積り（Estimate）
**運用例**
- 個人開発（学習用）
    - Todo = 「次に学ぶこと」
    - In Progress = 「今やっている」
    - Done = 「完了した学習」
- チーム開発
    - Issueを全部Projectsに紐づけ → 毎朝スタンドアップで確認
    - 進捗をボードで可視化

## 5. Securityとは
> GitHubリポジトリのセキュリティ機能をまとめたタブです。依存ライブラリの脆弱性や、秘密情報の漏洩リスクを早期に検知して教えてくれます。
**主な機能**
- Dependabot alerts
    - 依存パッケージ（例：DjangoやFlask）が脆弱なバージョンだと通知してくれる
    - 自動で「修正PR」を作成することも可能
-  Code scanning（コード解析）
    - 静的解析ツール（CodeQLなど）でソースコードをチェック
    - SQLインジェクションや入力チェック不足などのセキュリティ問題を検出
-  Secret scanning（秘密情報検出）
    - APIキーやパスワードを誤ってコミットすると警告
    - 一部の有名サービス（AWS, Google, GitHub自身）は即検知してアラート
-  Dependency review
    - PRで新しい依存ライブラリを追加すると、セキュリティリスクを可視化
-  ブランチ保護ルールと連携
    - `main` に「必ずテストが通ること」「レビュー必須」を強制できる
**使い方**
1. Securityタブを開く
    - リポジトリの上部にある → Security をクリック。
    - 最初は「Enable」や「Set up」ボタンが並んでいます。
2. Dependabot（依存関係チェック）の有効化
    - Dependabot alerts を ON にすると、脆弱なライブラリがあると通知されます。
    - Dependabot security updates を ON にすると、自動で修正版PRを作ってくれます。
    - 例：`Django 4.2.1 → 4.2.7` に自動アップデートPRを提案。　→　学習中のDjangoやPostgreSQL関連パッケージを常に最新に保てる。
3. Secret scanning（秘密情報検出）の設定
    - Settings > Code security and analysis → Secret scanning を有効化。
    - 誤って .env をコミットした場合、アラートが出ます。
    - GitHubはAWS/GCP/Slackなど主要サービスのキーを自動検出。 → Djangoの SECRET_KEY やAWSキーを守るために必須。
4. Code scanning（コード脆弱性チェック）
    - Set up code scanning → CodeQL Analysis を選ぶのが基本。
    - `.github/workflows/codeql.yml` が自動生成され、Pull requestやpushの際にコード解析が走る。
    - SQLインジェクションや入力チェック漏れなどを検知。
5. Branch protection（ブランチ保護）と組み合わせ
    - Securityタブと合わせて使うと「安全な開発の流れ」が作れる。
    - 例：ブランチを守るルールづくり
        - Settings > Branches > Add rule
        - `main` に直コミット禁止
        - Pull request必須
        - テスト（Actions）が通らないとマージできない
6. 運用のコツ
- Dependabotの自動PRをONにして、依存ライブラリを常に最新へ
- Secret scanning を有効化して「うっかり漏洩」を未然に防止
- Code scanning は最初はGitHub提供のテンプレートで十分
7. 日常的な使い方
- 毎朝 → Securityタブを確認（アラートが出ていないか）
- Pull requestを作成したら → Dependabot / CodeQLチェックが通るか確認
- アラートが出たら → 対応するIssueを立てる or 自動PRをマージ
- 秘密情報が漏れた場合 → 即座にキーを再発行・履歴から削除
* `main` に 必須レビュー / 必須ステータスチェック / 直コミット禁止を設定

## 6.Insightとは
> GitHubリポジトリの 活動状況や貢献度を可視化するタブ です。「どのくらい開発が進んでいるか」「誰がどんな貢献をしているか」を見える化できます。

### 主な機能
- Contributors: 誰がどのくらいコードを追加・修正しているかを可視化。貢献度を把握できます。
- Commits: 時系列でのコミット数を表示し、活発に更新されているかを確認可能。
- Code frequency: 追加・削除されたコード行数の推移を確認できます。
- Pull requests: PR（プルリクエスト）の作成・マージ状況を分析できます。
- Traffic: リポジトリへのアクセス数やクローン数を確認可能。

### 活用ポイント
- 個人学習リポジトリでは「どのくらい作業してきたか」の振り返りに便利。
- チーム開発では「誰がどの部分を担当しているか」「開発が停滞していないか」を客観的に把握できる。
- プロフィールや取引先へのアピールにも使える（定期的にコミットしていることを示せる）。

---

*注意書き*
- このハンドブックの中のメールアドレス・鍵は例です。実際の情報は絶対に公開しないでください
- 本ハンドブックの内容を利用したことによるいかなるトラブル・損害についても、作者は一切の責任を負いません。
*Note*
- This handbook is for learning purposes only. Do not expose your real emails, keys, or secrets.
*Disclaimer*
- The author assumes no responsibility for any trouble or damage caused by using this handbook.

