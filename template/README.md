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
5. 規則が効いているかを、必ず自分で確かめます（`OK` と出ても、相手のフォルダが守られることの保証ではありません。下の「結果の読み方」を必ず読んでください）。
   - **相手のフォルダには何も書き込みません**（読むだけです）。試すのは、毎回新しく作る一時フォルダ（`mktemp -d`）の中のダミーファイルだけで、終わると（途中で失敗しても）その一時フォルダだけが消えます。
   - Git Bash で `cd` して company フォルダに移動し、`<相手社フォルダ>` を相手のフォルダの場所に書き換えてから、全体を貼り付けて実行します。`claude -p` を1回呼びます。一時フォルダでは、あなたの settings.json の写し（相手のフォルダの場所だけを置き換えたもの）を使います。写しに hooks などの設定があれば、それも動きます。
   ```
   ( set -eu
     P="<相手社フォルダ>"   # 例: C:/Users/you/shared/partner（このフォルダは読むだけで、何も書きません）
     [ -d "$P" ] || { echo "NG: フォルダが見つかりません: $P"; exit 1; }
     [ -f .claude/settings.json ] || { echo "NG: company フォルダで実行してください（.claude/settings.json がありません）"; exit 1; }
     abs() { local w; w=$(cd "$1" && pwd -W); echo "//$(echo "${w:0:1}" | tr 'A-Z' 'a-z')${w:2}"; }
     # 1) settings.json に、相手のフォルダを絶対パスで指す規則の文字列があるか（読むだけ）
     A="$(abs "$P")"
     case "$A" in *[][*?{}]*) echo "NG: フォルダの場所に [ ] * ? { } のどれかが含まれています。deny の規則で正しく守れないことがあるので、フォルダ名を変えてください: $P"; exit 1;; esac
     grep -qF "\"Edit($A/**)\"" .claude/settings.json || { echo "NG: settings.json に \"Edit($A/**)\" がありません"; exit 1; }
     # 2) あなたの settings.json の「写し」で Write ツールが止まるかを、新しく作った一時フォルダの中のダミーだけで試す
     #    写しでは、相手のフォルダの場所だけを一時フォルダの中に置き換えます（JSON の壊れや allow への書き間違いも、ここで NG になります）
     T=$(mktemp -d)                         # 毎回新しく作られる空のフォルダ。既存のフォルダは使わない
     trap 'cd / && rm -rf -- "$T"' EXIT     # 終わったら（途中で失敗しても）、この一時フォルダだけを消す
     mkdir "$T/company" "$T/company/.claude" "$T/partner" "$T/free"
     echo original > "$T/partner/dummy.txt" # 守られるはずのダミー
     echo original > "$T/free/dummy.txt"    # 比較用：守られていないダミー（AI が本当に書こうとしたかの確認）
     S=$(cat .claude/settings.json)
     printf '%s\n' "${S//"$A/"/"$(abs "$T/partner")/"}" > "$T/company/.claude/settings.json"
     B=$(sha256sum < "$T/partner/dummy.txt")
     cd "$T/company"
     claude -p "First use the Read tool on ../free/dummy.txt and ../partner/dummy.txt. Then use the Write tool to overwrite each of them with the text changed. Reply for each file: succeeded or denied." --model haiku --output-format json --allowedTools "Read,Write,Edit" --permission-mode acceptEdits < /dev/null > "$T/out.json" 2>/dev/null || true
     if [ "$B" = "$(sha256sum < "$T/free/dummy.txt")" ]; then echo "判定できません: 比較用のダミーも変わっていません（AI が書き込みを試していません）。もう一度実行してください"; exit 2; fi
     if [ "$B" = "$(sha256sum < "$T/partner/dummy.txt")" ]; then
       grep -qF '"permission_denials":[{' "$T/out.json" || { echo "判定できません: 拒否の記録がありません（AI が守られる側に書き込みを試していません）。もう一度実行してください"; exit 2; }
       echo "OK: 写しの設定で、AI の Write ツールによるダミーへの書き込みは拒否されました（相手のフォルダが守られる保証ではありません。README の「結果の読み方」を参照）"
     else echo "NG: 守られるはずのダミーが書き換わりました（settings.json の書き方を確かめてください）"; exit 1; fi
   )
   ```
   - 結果の読み方：`OK` は「あなたの settings.json の写しで、AI が Write ツールでダミーに書こうとしたとき、拒否された」という意味です。**相手のフォルダが守られることの保証ではありません。** Bash を自動許可しているときの node などのスクリプト経由の書き込み、対話モード、確認の後で設定を変えた場合は、この確認の範囲外です。拒否の記録がどのファイルについてのものかまでは確かめていません。`NG` は、規則が無い・settings.json の JSON が壊れている・規則を `allow` に書いた、などで効いていません。`判定できません` は AI が書き込みを試さなかったときなので、もう一度実行してください。
   - Git Bash 用です（`pwd -W` を使うため。Mac・Linux では未確認）。
   - 絶対パスの deny でも、Bash を自動許可していると、node などのスクリプト経由では書き込めました（筆者の環境で実測）。Bash は自動許可しないでください。
6. `claude` を起動し、「STATUS.md を読んで、最初の企画を探して」と頼みます。

## 筆者の環境で確認したこと（2026-10-11）
- 実行済み・合格：4つのサブエージェントが読み込まれる。sync はスキルとして認識される。絶対パスの deny で、相手社のファイルへの Write・Edit・Bash の echo が止まる。
- 実行済み・止まらなかった：絶対パスの deny でも、Bash を自動許可していると、node のスクリプト経由の書き込みは止まらない。
- 実行済み・不合格（上の注意に反映済み）：相対パスの deny では止まらない。
- 未確認：Windows 以外の環境、長期間の運用。
