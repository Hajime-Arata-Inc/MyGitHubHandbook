# 02_branch_workflow.md — ブランチ/マージ/プルリクエスト
> 目的：安全で見通しの良い開発のために、ブランチ、マージ、プルリクエストについて解説します。

---

## 1. ブランチ（Branch）

### 1-1. ブランチとは
- Git における「開発の分岐点（作業用の枝）」のことで,1つのリポジトリの中で、複数の開発ラインを同時に進められる仕組みです。
- 例えば、開発のある時点において開発を成功させるために２パターンの開発ラインができたらブランチを作ってそれぞれの開発ラインで開発を進めることができます。そして成功した方を`main`にマージすればいいわけです。
- 本筋である`main`の他のブランチを`topic`ブランチといいます。

### 1-2. ブランチの命名について
- `main` # 本番運用ブランチ。開発用のブランチ以下のようなブランチを`main`とは別で作ってマージする方法を推奨。
- 以下用途別ブランチ例（命名は自由にできます。）
  - `develop`　# 統合・リリース前検証
  - `feature/<要約>` # 機能追加（例：feature/signup-form）
  - `fix/<要約>` # バグ修正（例：fix/login-redirect）
  - `hotfix/<要約>` # 本番緊急修正（例：hotfix/env-misconfig）
  - `refactor/<要約>` # 仕様不変の改良（例：refactor/template-structure）
  - `docs/<要約>` # ドキュメントのみ（例：docs/readme-ja）
  - `chore/<要約>` # 付帯作業（CI, lint, 依存更新 等）
- 命名規則
 - 区切りはハイフン `-`、日本語は避ける（URLやCLIで扱いやすくするため）。
 - Issue 番号があれば先頭に付与：`feature/123-signup-form`。

### 1-3. 作業の基本フロー（main 運用）
1. 最新化：`git switch main && git pull`
2. 枝分け：`git switch -c feature/signup-form`
3. 小さくコミット（意味の塊ごと）：`git add -p && git commit -m "feat: add signup form skeleton"`
4. こまめに push：`git push -u origin feature/signup-form`
5. WIP（Work In Progress（作業中））はDraft Pull Requestで早めに共有（レビュー依頼前でも可）:`WIP: ログイン機能の途中実装`
6. CI（Continuous Integration）が緑 かつ レビューOK になったら`main`へマージ。
7. 削除：マージ後にリモート・ローカルのトピックブランチを削除。理由：使い終わったブランチを残すとどのブランチが最新か分かりにくくなるため。  

### 1-4. 例：`develop`を使う場合
- トピック → `develop`に段階的に集約 → リリース直前に`main`へマージ。
- ホットフィックス（本番環境で見つかった緊急の不具合を、最優先で修正して即時リリースする対応）だけは`main`へ直接プッシュ → その後で`develop`に逆マージして整合性を保つ。

---

## 2. マージ（Merge）

### 2-1.マージとは
- Gitで別々のブランチで進めた変更を、1つに統合する操作のことです。
- マージの種類
  - Fast-forwardマージ: main に他の変更がなく、枝をそのまま先に進めるだけで統合できる。履歴がきれい。
  - 通常のマージ（マージコミットあり）: 両方のブランチで変更があった場合、統合専用のコミットが作られる。
  - コンフリクト（衝突）: 同じファイルの同じ行を別々に変更していたら、自動で統合できない。開発者が手動で解決する必要あり。
- Merge戦略一覧

| Merge              | 使いどころ                         | 長所                         | 短所               |
| ------------------ | ----------------------------- | -------------------------- | ---------------- |
| **Merge commit**   | 複数ブランチをそのまま統合したい時             | 履歴が実態に忠実、並行作業の合流点が見える      | 履歴が枝分かれで肥大化しがち   |
| **Squash & merge** | 小粒なコミットを 1 つに要約したい時（UI 微修正など） | main 履歴がスッキリ、レビュー単位でまとまる   | 元コミット単位の粒度は失われる  |
| **Rebase & merge** | 直線的な履歴を維持したい時                 | 歴史がきれい、`git bisect` 等がしやすい | 履歴を書き換えるため、運用に注意 |

### 2-2. マージの判断の目安
- 履歴を要約したい？ → *Yes* = `Squash`／*No* → 次へ
- 直線履歴を厳格維持？ → *Yes* = `Rebase`／*No* = **Merge commit**
  - **公開リポジトリ＋共同開発で`rebase`を使う際の注意**：共有済みブランチの履歴書き換えは原則禁止。どうしても必要ならPull request上で実行し、強制push方針はドキュメント化。
- チーム合意があればリポジトリ設定で1〜2種類に固定すると混乱が減る。

### 2-3. 実務ルール例（個人〜小規模）
- 既存コードへの小さな改善・UI 修正：**Squash & merge**
- 大規模機能や長期ブランチの統合：**Merge commit**（合流点を残す）
- 完全直線を好む場合：**Rebase & merge**（ただし強制pushは避け、Pull request上で`rebase`）

## 2-4. よくある分岐（ケース別レシピ）
- **長生きする大規模機能**：こまめに `main` を取り込み（`git pull --rebase origin main`）、衝突を小さく保つ。
- **デザイン小修正が連発**：1 PR = 1 テーマでまとめ、**Squash** で履歴をクリーンに。
- **急ぎの本番障害**：`hotfix/*` → テスト最小で main マージ → すぐ戻せるように **変更範囲を極小** に。
- **学習・検証用**：`study/*` を採番。main へは入れず、README にリンク。

### 2-5. マージのコマンド
```bash
git switch main              # main ブランチに移動
git merge feature/login      # feature/login の変更を main に統合
```

---

## 3. Pull request

### 3-1. Pull requestとは
- GitHub上で「他のブランチで行った変更を、main などの別ブランチに取り込んでください」と依頼する仕組みです。略してPRと呼ばれます。以下の目的で活用されます。
  - コードの変更内容を可視化
  - レビューや`main`への取り込みを依頼（チームメンバーが内容を確認できる）
    - 作業例：　ローカルで `feature/login` ブランチを作業　→ `git push origin feature/login`で GitHubにアップ → GitHub上でPull Requestを作成する → `feature/login` を `main`に取り込メルかどうかの確認を依頼 → 責任者や担当者が確認 → レビューやCIが通れば`Merge pull request`で統合
#### ローカルで新しいブランチ(featur/login)を作成し、pushまでのコマンド
```bash
# mainブランチを最新にしてから作業
git switch main
git pull origin main

# 新しいブランチを作成して移動
git switch -c feature/login

# ファイルを編集したらステージング
git add <YOURFILENAME>   # ファイルを指定。全ファイルなら git add .

# コミット
git commit -m "COMMENT"

# リモートにブランチをプッシュ
git push -u origin feature/login

# pushした後はGitHub（リモート）を開き、Codeのタブをクリック。内容を確認しPull requestする。
```

### ３-２.Pull requestの作り方
1. ブランチを push → 「Compare & pull request」
2. テンプレート（このページ下部にサンプルあり）に沿って記載：
  - 目的 / 背景
  - 変更点（スクショ・GIF が有効）
  - テスト観点・手順
  - 影響範囲 / 逆影響（破壊的変更の有無）
  - 秘密情報の混入チェック
3. Draft pull requestで早期共有 → フィードバック反映 → Ready for review

### 3-2. レビューの観点（チェックリスト）
- 仕様満たす？ UX 問題は？
- 変更は最小？（YAGNI / KISS）
- 命名・責務分離・重複排除（DRY）
- 例外処理・ログ・バリデーション
- テスト網羅 / CI 緑
- `.env.example` 更新・ドキュメント更新
- 秘密情報や個人情報が差分に無い（鍵・証明書・ダンプ・画像 EXIF）

### 3-3. マージ後
- 自動デプロイ（あれば）→ モニタリング確認
- ブランチ削除(ブランチ整理するするために必要なら削除する。使うなら残す)
```bash
git switch main              # mainに戻る
git pull origin main         # 最新を取得
git branch -d feature/update-readme
```
- リリースノートへ要約を転記（Conventional Commits なら自動生成も可）


> Pull request テンプレート（サンプル）

```
### 目的 / 背景
-

### 変更点
-

### テスト観点・手順
- [ ] ローカルで起動し、XXX を確認
- [ ] テストが通過

### 影響範囲 / 逆影響（破壊的変更の有無）
-

### 秘密情報の混入チェック
- [ ] 秘密情報や個人情報は含まれていない
- [ ] `.env.example` を更新済み

### スクリーンショット
(任意)
```

---


*注意書き*
- このハンドブックの中のメールアドレス・鍵は例です。実際の情報は絶対に公開しないでください
- 本ハンドブックの内容を利用したことによるいかなるトラブル・損害についても、作者は一切の責任を負いません。
*Note*
- This handbook is for learning purposes only. Do not expose your real emails, keys, or secrets.
*Disclaimer*
- The author assumes no responsibility for any trouble or damage caused by using this handbook.


