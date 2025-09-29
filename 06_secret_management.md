# 06 情報漏洩防止を徹底しましょう。
> GitHubを活用しながら開発する場合、情報漏洩には最大限の注意をはらわなければなりません。その防止策をまとめます。

## 1. 基本
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


### 2. 仕組みで守る
- `.gitignore` に `.env`、鍵、DB ダンプ、`/media` 等を追加
- `pre-commit` で **secret scan**（例：`gitleaks`）と **lint/test** を自動化
- `.env.example` を配布用に整備（必要変数のみ、ダミー値）
- ログと CI のマスキング設定（GitHub Actions の `secrets.*`）

### ３. うっかりコミットした時の緊急手順
1. **直後**なら：`git reset HEAD~` で取り消し、`.gitignore` 追加
2. **push 済み**なら：キー無効化／ローテーション → 履歴から削除（`git filter-repo`）→ 影響範囲通知
3. Issue/PR で**事後対策を記録**（再発防止）


