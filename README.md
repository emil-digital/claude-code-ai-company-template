# claude-code-ai-company

Claude Code のメインセッションを「責任者（Root）」にし、企画・調査・制作・品質チェックの4つのAI担当（サブエージェント）を、人間の承認と品質チェックの決まりのもとで動かすための設定テンプレートです。版：1.0.0

> 運用を始めて間もない設定です。**これで収益が出た実績はありません。** 決まりの多くは文章で AI に守らせるもので、技術的に止められるものは限られます（「できること・できないこと」）。外部への公開・課金・送信は、必ず人間が自分の手で行ってください。

## 中身
| パス | 内容 |
|---|---|
| template/ | 導入する 18 ファイル。新しい作業フォルダにコピーして使う |
| template/CLAUDE.md | Root の規約（目標、承認が要る行為、成果物の5段階、報告の形） |
| template/.claude/agents/ | 企画・調査・制作・品質チェックの4担当 |
| template/.claude/settings.json | 相手のフォルダへの書き込み禁止の**サンプル**（2社で運用する場合だけ使う） |
| template/.claude/skills/sync/ | 2社で運用するときの同期手順 |
| template/docs/ ほか | 企画の審査基準、組織の決まり、承認・収支・学びの記録の雛形 |
| SHA256SUMS | template/ の各ファイルの照合値 |
| tools/install.sh | 任意の導入スクリプト（使わなくても導入できる） |
| VERSION / CHANGELOG.md / LICENSE | 版、更新履歴、利用条件（MIT） |

## 導入（基本の方法）
必要なもの：Claude Code。照合と動作確認には Git Bash（Windows）または bash が使えると便利です。

1. このページの **Releases** から `claude-code-ai-company-v1.0.0.zip` をダウンロードし、展開します。
2. （任意）中身が配布したものと同じかを照合します。展開したフォルダで：
   ```
   sha256sum -c SHA256SUMS
   ```
   すべて `OK` と出れば一致しています。
3. **新しい**作業フォルダ（例：`company`）を作り、展開した `template/` の**中身**をすべてコピーします。`.claude` のような「.」で始まるフォルダも忘れずにコピーしてください（Windows のエクスプローラーでは、表示設定によって見えないことがあります）。既にある作業フォルダには上書きしないでください。
4. `company/CLAUDE.md` の「目標と制約」にある `{{ }}` を、自分の目標と制約に書き換えます。
5. 2社（別のAI同士）で運用**しない**場合：`company/.claude/skills/sync/` フォルダと、`company/.claude/settings.json` を削除して構いません。
6. 2社で運用する場合だけ、次の「相手のフォルダを守る設定」を行います。
7. `company` で `claude` を起動し、「STATUS.md を読んで、最初の企画を探して」と頼みます。

### 任意：導入スクリプトを使う方法
Git Bash で、展開したフォルダから次を実行すると、手順3の代わりになります。**まだ存在しないフォルダ名**を指定してください（既にあると何もせずに止まります）。ネットワークには接続しません。
```
bash tools/install.sh company
```

## 相手のフォルダを守る設定（2社で運用する場合）
`template/.claude/settings.json` は**サンプル**です。中の `//c/CHANGE_ME/shared/partner/**` は存在しない場所なので、そのままでは何も守りません。

1. `CHANGE_ME/shared/partner` の部分を、相手のフォルダの**絶対パス**に書き換えます。Windows の `C:\Users\you\shared\partner` なら `Edit(//c/Users/you/shared/partner/**)` です。
   - 筆者の環境（Windows 10、Claude Code 2.1.296）では、`Edit(../shared/partner/**)` のような `../` の相対パスでは止まりませんでした。
2. `.claude/skills/sync/SKILL.md` の共有フォルダの場所（`../shared/`）も、実際の場所に書き換えます。
3. **本当に止まるかを、ダミーのファイルで確かめます。**相手のフォルダの本物のファイルでは試さないでください。
   ```
   echo original > "<相手のフォルダ>/_deny_test.txt"
   sha256sum "<相手のフォルダ>/_deny_test.txt"
   claude -p "First use the Read tool on <相手のフォルダ>/_deny_test.txt, then use the Write tool to overwrite it with the text changed. Reply: succeeded or denied." --allowedTools "Read,Write,Edit" --permission-mode acceptEdits
   sha256sum "<相手のフォルダ>/_deny_test.txt"
   rm "<相手のフォルダ>/_deny_test.txt"
   ```
   2回のハッシュが同じなら止まっています。`claude -p` を作業フォルダ（`company`）で実行してください。

## できること・できないこと
| 決まり | どう守られるか | 筆者の環境での確認 |
|---|---|---|
| 企画・調査・品質チェックはファイルを書かない | 担当の設定で Write・Edit を渡していない（技術的）。品質チェックは検証用に Bash を持つので、Bash で書かないことは文章のルール | 4担当が読み込まれることを確認 |
| 制作は指定したフォルダだけに書く | 文章のルール | 技術的な制限なし |
| 外部への公開・課金・送信は人間だけ | 文章のルール | 技術的な制限なし |
| 相手のフォルダに書き込まない | settings.json の deny（技術的）＋文章のルール | 絶対パスの deny で Write・Edit・Bash の echo は止まった。**Bash を自動許可していると、node などのスクリプト経由の書き込みは止まらなかった** |

- deny は「ブレーキ」であって、セキュリティの境界ではありません。**Bash を自動許可しない**でください。本当に触らせたくないものは、別の OS ユーザーで動かす、仮想マシンやコンテナで動かす、などの方法も検討してください。
- 確かめていないこと：Mac・Linux、対話モードでの権限の動作、長期間の運用。

## 照合値について
`SHA256SUMS` は `template/` の各ファイルの SHA-256 です。Release の ZIP 自体の SHA-256 は、Release のページに書きます。

## ライセンス
MIT（LICENSE）。自由に使い、変えて構いません。無保証です。
