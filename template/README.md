# Claude Code で「AI会社」を運営する設定一式

Claude Code のメインセッションを Root（責任者）にし、企画・マーケティング・商品開発・品質管理の4つのサブエージェントと、人間（Owner）の承認ゲートで動かすための設定です。
筆者が、副業の実験として2026年10月から運用している設定から、個人情報と固有の事情を抜きました。

**注意**
- これは運用を始めて間もない設定です。収益の実績はありません。「これで稼げる」仕組みではありません。
- 承認ゲートはルール（文章）で AI に守らせるもので、技術的に止められるのは settings.json の deny だけです。外部への公開・課金・送信は、必ず人間が自分で行ってください。

## 中身
| ファイル | 役割 |
|---|---|
| CLAUDE.md | Root の運用規約（目標、承認ゲート、成果物の5段階、報告の形） |
| .claude/agents/*.md | 企画・マーケティング・商品開発・品質管理の4担当。担当ごとに渡すツールを絞り（企画・マーケ・品質管理は Write・Edit なし）、書ける範囲はルールで指定 |
| .claude/skills/sync/SKILL.md | 2社（別のAI同士）で運用するときの同期手順 |
| .claude/settings.json | 相手社のフォルダへの書き込みを禁止する設定（2社運用の場合） |
| docs/PLANNING_CRITERIA.md | 企画の審査基準と、小さく市場で試す枠 |
| docs/ORGANIZATION.md | 部門の責任、品質ゲート、実行ルート |
| docs/APPROVALS.md | 人間の承認を記録する形式 |
| docs/BUSINESS_IDEAS.md | 未採用の案の置き場 |
| STATUS.md / OWNER_INBOX.md | 再開の入口と、人間にしか決められないことの一覧 |
| experiments/_TEMPLATE.md、ledger/、owner-guides/ | 案件記録、収支・学び、人間向け手順書の置き場 |

## 導入の手順（6つ。時間は実測していません）
1. このフォルダを、作業場所（例：`company/`）としてコピーします。
2. CLAUDE.md の「目標と制約」の `{{ }}` を、自分の目標と制約に書き換えます。
3. 2社で運用しない場合は、`.claude/skills/sync/` と `.claude/settings.json` の deny を消して構いません。
4. 2社で運用する場合：
   - 共有フォルダを作り、settings.json の `//c/CHANGE_ME/shared/partner/**` を、相手社フォルダの**絶対パス**に書き換えます。Windows の `C:\Users\you\shared\partner` なら `//c/Users/you/shared/partner/**` です。
   - `.claude/skills/sync/SKILL.md` の共有フォルダの場所（`../shared/` の行）も、実際の場所に書き換えます。
   - **相対パス（`../shared/partner/**`）では止まりませんでした**（筆者の環境で実測。Claude Code 2026年10月時点）。`Write(...)` の規則も効きません。`Edit(...)` で書いてください。
5. 書き込みが止まることを、**ダミーのファイルで**必ず自分で確かめます。相手のフォルダの本物のファイルでは試さないでください。
   ```
   echo original > "<相手社フォルダ>/_deny_test.txt"
   sha256sum "<相手社フォルダ>/_deny_test.txt"
   claude -p "First use the Read tool on <相手社フォルダ>/_deny_test.txt, then use the Write tool to overwrite it with the text changed. Reply: succeeded or denied." --allowedTools "Read,Write,Edit" --permission-mode acceptEdits
   sha256sum "<相手社フォルダ>/_deny_test.txt"   # 最初と同じなら止まっている
   rm "<相手社フォルダ>/_deny_test.txt"
   ```
   - 絶対パスの deny でも、Bash を自動許可していると、node などのスクリプト経由では書き込めました（筆者の環境で実測）。Bash は自動許可しないでください。
6. `claude` を起動し、「STATUS.md を読んで、最初の企画を探して」と頼みます。

## 筆者の環境で確認したこと（2026-10-11）
- 実行済み・合格：4つのサブエージェントが読み込まれる。sync はスキルとして認識される。絶対パスの deny で、相手社のファイルへの Write・Edit・Bash の echo が止まる。
- 実行済み・止まらなかった：絶対パスの deny でも、Bash を自動許可していると、node のスクリプト経由の書き込みは止まらない。
- 実行済み・不合格（上の注意に反映済み）：相対パスの deny では止まらない。
- 未確認：Windows 以外の環境、長期間の運用。
