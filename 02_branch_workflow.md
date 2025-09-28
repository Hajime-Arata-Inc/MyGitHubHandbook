# 02_branch_workflow.md — ブランチ運用とマージ戦略

> 目的：安全で見通しの良い開発のために、**ブランチの切り方・育て方・戻し方**と、**状況に応じたマージ戦略**を定義する。

---

## 0. 用語と前提

* **main**：常にデプロイ可能（＝壊れていない）状態。本番の源流。
* **develop（任意）**：チームや中規模以上で採用。main 直前の統合先。
* **トピックブランチ**：機能追加・修正など作業単位のブランチ（例：`feature/*`, `fix/*`）。
* **PR（Pull Request）**：レビューと検証の場。マージは常に PR 経由。

> 🔒 **機密対策の大原則**：API キーや `SECRET_KEY` は **コードに書かない**。`.env` に分離し、`.gitignore` で除外。PR テンプレートに「秘密情報が含まれていないか」チェック項目を置く。

---

## 1. ブランチの使い方（命名・ライフサイクル）

### 1-1. 命名規則（推奨）

```
main                      # 本番運用ブランチ
(任意) develop            # 統合・リリース前検証
feature/<要約>            # 機能追加（例：feature/signup-form）
fix/<要約>                # バグ修正（例：fix/login-redirect)
hotfix/<要約>             # 本番緊急修正（例：hotfix/env-misconfig）
refactor/<要約>           # 仕様不変の改良（例：refactor/template-structure）
docs/<要約>               # ドキュメントのみ（例：docs/readme-ja）
chore/<要約>              # 付帯作業（CI, lint, 依存更新 等）
```

* 区切りはハイフン `-`、**日本語は避ける**（URL や CLI で扱いやすくするため）。
* Issue 番号があれば先頭に付与：`feature/123-signup-form`。

### 1-2. 作業の基本フロー（main 運用）

1. **最新化**：`git switch main && git pull`
2. **枝分け**：`git switch -c feature/signup-form`
3. **小さくコミット**（意味の塊ごと）：`git add -p && git commit -m "feat: add signup form skeleton"`
4. **こまめに push**：`git push -u origin feature/signup-form`
5. **WIP は Draft PR** で早めに共有（レビュー依頼前でも可）。
6. **CI が緑**かつレビュー OK になったら **main へマージ**。
7. **削除**：マージ後にリモート・ローカルのトピックブランチを削除。

> 🧪 **テスト最優先**：UI 変更でも最低限のユニット/スモークテストを用意。CI で秘密情報が出力ログに出ないことも確認。

### 1-3. develop を使う場合（任意）

* トピック → **develop** に段階的に集約 → リリース直前に **main へ**。
* ホットフィックスだけは **main** へ直接 → その後 **develop に逆マージ** して整合性を保つ。

---


---

## 3. マージ戦略（3 つの選択肢）

| 戦略                 | 使いどころ                         | 長所                         | 短所               |
| ------------------ | ----------------------------- | -------------------------- | ---------------- |
| **Merge commit**   | 複数ブランチをそのまま統合したい時             | 履歴が実態に忠実、並行作業の合流点が見える      | 履歴が枝分かれで肥大化しがち   |
| **Squash & merge** | 小粒なコミットを 1 つに要約したい時（UI 微修正など） | main 履歴がスッキリ、レビュー単位でまとまる   | 元コミット単位の粒度は失われる  |
| **Rebase & merge** | 直線的な履歴を維持したい時                 | 歴史がきれい、`git bisect` 等がしやすい | 履歴を書き換えるため、運用に注意 |

### 3-1. 判断の目安（簡易ツリー）

* **履歴を要約したい？** → *Yes* = **Squash**／*No* → 次へ
* **直線履歴を厳格維持？** → *Yes* = **Rebase**／*No* = **Merge commit**
* チーム合意があれば **リポジトリ設定で 1〜2 種類に固定** すると混乱が減る。

### 3-2. 実務ルール例（個人〜小規模）

* 既存コードへの小さな改善・UI 修正：**Squash & merge**
* 大規模機能や長期ブランチの統合：**Merge commit**（合流点を残す）
* 完全直線を好む場合：**Rebase & merge**（ただし強制 push は避け、PR 上で rebase）

> ⚠️ **公開リポ＋共同開発で rebase を使う際の注意**：共有済みブランチの履歴書き換えは原則禁止。どうしても必要なら PR 上で実行し、強制 push 方針はドキュメント化。

---

## 4. PR（レビュー）フロー

### 4-1. PR の作り方

1. ブランチを push → 「Compare & pull request」
2. **テンプレート**（例）に沿って記載：

   * 目的 / 背景
   * 変更点（スクショ・GIF が有効）
   * テスト観点・手順
   * 影響範囲 / 逆影響（破壊的変更の有無）
   * **秘密情報の混入チェック ✅**
3. **Draft PR** で早期共有 → フィードバック反映 → Ready for review

### 4-2. レビューの観点（チェックリスト）

* 仕様満たす？ UX 問題は？
* 変更は最小？（YAGNI / KISS）
* 命名・責務分離・重複排除（DRY）
* 例外処理・ログ・バリデーション
* テスト網羅 / CI 緑
* **`.env.example` 更新・ドキュメント更新**
* **秘密情報や個人情報が差分に無い**（鍵・証明書・ダンプ・画像 EXIF）

### 4-3. マージ後

* 自動デプロイ（あれば）→ モニタリング確認
* ブランチ削除
* リリースノートへ要約を転記（Conventional Commits なら自動生成も可）

---

## 5. リリースと保守

### 5-1. タグ運用

* リリース時にタグ：`v1.2.3`（SemVer）
* 重要修正は **hotfix/** ブランチ → main へ即時反映 → develop に逆マージ

### 5-2. ブランチ保護ルール（GitHub Settings）

* `main`（必要なら `develop`）に**保護ルール**：

  * ① PR レビュー必須（1 名〜）
  * ② ステータスチェック必須（CI 緑）
  * ③ 直 push 禁止、強制 push 禁止
  * ④ 必要なら「線形履歴（linear history）」を有効化

---

## 6. 機密情報の漏洩防止（恒常対策）

### 6-1. 仕組みで守る

* `.gitignore` に `.env`、鍵、DB ダンプ、`/media` 等を追加
* `pre-commit` で **secret scan**（例：`gitleaks`）と **lint/test** を自動化
* `.env.example` を配布用に整備（必要変数のみ、ダミー値）
* ログと CI のマスキング設定（GitHub Actions の `secrets.*`）

### 6-2. うっかりコミットした時の緊急手順

1. **直後**なら：`git reset HEAD~` で取り消し、`.gitignore` 追加
2. **push 済み**なら：キー無効化／ローテーション → 履歴から削除（`git filter-repo`）→ 影響範囲通知
3. Issue/PR で**事後対策を記録**（再発防止）

---

## 7. よくある分岐（ケース別レシピ）

* **長生きする大規模機能**：こまめに `main` を取り込み（`git pull --rebase origin main`）、衝突を小さく保つ。
* **デザイン小修正が連発**：1 PR = 1 テーマでまとめ、**Squash** で履歴をクリーンに。
* **急ぎの本番障害**：`hotfix/*` → テスト最小で main マージ → すぐ戻せるように **変更範囲を極小** に。
* **学習・検証用**：`study/*` を採番。main へは入れず、README にリンク。

---

## 8. コマンド・チートシート

```bash
# 最新化と枝分け
git switch main && git pull
git switch -c feature/signup-form

# 粒度を保ってコミット
git add -p
git commit -m "feat: add signup form"

# リモート追跡＆PR 作成
git push -u origin feature/signup-form

# main を取り込む（衝突を早めに解消）
git pull --rebase origin main

# マージ後の掃除
git branch -d feature/signup-form
git push origin --delete feature/signup-form
```

---

## 9. PR テンプレ（サンプル）

```
### 目的
-

### 変更点
-

### 動作確認
- [ ] ローカルで起動し、XXX を確認
- [ ] テストが通過

### 影響範囲
-

### セキュリティ
- [ ] 秘密情報や個人情報は含まれていない
- [ ] `.env.example` を更新済み

### スクリーンショット
(任意)
```

---

## 10. まとめ

* **ブランチは目的別・短命に、PR は小さく早く**。
* **状況でマージ戦略を選ぶ（Squash / Merge / Rebase）**。
* **保護ルール＋pre-commit＋secret scan** で漏洩を未然に防ぐ。

*注意書き*
- このハンドブックの中のメールアドレス・鍵は例です。実際の情報は絶対に公開しないでください
- 本ハンドブックの内容を利用したことによるいかなるトラブル・損害についても、作者は一切の責任を負いません。
*Note*
- This handbook is for learning purposes only. Do not expose your real emails, keys, or secrets.
*Disclaimer*
- The author assumes no responsibility for any trouble or damage caused by using this handbook.

