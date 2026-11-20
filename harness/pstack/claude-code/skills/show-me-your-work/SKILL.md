---
name: show-me-your-work
description: "長時間の作業や無人の作業について、レビューできる決定の記録を残す。決定 1 つにつき 1 行の TSV のログに、何を、なぜ、証拠、結果を書く。既定ではローカルに置き、レビュアーが結果を信頼するのに記録が必要なときはコミットする。/show-me-your-work、自律的な実行や複数フェーズの実行、人間が席を外したあとでレビューする作業に使う。"
disable-model-invocation: true
---

# 作業の過程を見せる（Show me your work）

In Claude Code, read [the Claude Code adapter](../../docs/harness/claude-code.md) once per session before you follow this skill. It maps the Cursor names in pstack skills to Claude Code.

正となるログを 1 つだけ持つ。

## 書式（The format）

1 つの TSV ファイルに、決定 1 つにつき 1 行を書く。セルは 1 行に収める。証拠は文章ではなく、参照先を指すものにする。

`references/decision-log-template.tsv`（ヘッダー行）をコピーして、新しいログを始める。列は次のとおりである。

- **ts.** ISO8601 のタイムスタンプ。
- **phase.** フェーズか、作業の流れ（workstream）。
- **decision.** 選んだことや行ったこと。1 行で書く。
- **why.** 理由を平易な言葉で書く。原則に従って決めたなら、専門用語のタグではなく、平易に言う。
- **evidence.** それを証明するリンクかパス。コミットの SHA、PR 番号、`file:line`、成果物・トレース・スクリーンショットのパスなど。決して段落を書かない。
- **result.** 結果か、判定条件（predicate）の状態。`tests green`、`reverted`、`pixel-diff 0`、`INCONCLUSIVE`、`open` など。

例を示す。レビュアーが一目で読めるよう、平易な言葉で書いてある。

```
ts	phase	decision	why	evidence	result
2026-05-24T09:02:00Z	frame	counted the work first, about 100 components and roughly 75 hours	wanted to know the size before starting a long run	commit 3a9f1c2	found 5 things to sort out before starting
2026-05-24T09:40:00Z	harness	took screenshots of the old version before changing anything	so we can compare old against new and catch any visual change	scripts/snapshot.sh, baseline/	saved 120 reference screenshots
2026-05-24T11:15:00Z	widget	moved the widget styles over without changing how it looks	keep the change small and the result identical	commit 7c21e0a, pixel-diff 0	looks identical, tests pass
2026-05-24T12:30:00Z	widget	threw out a helper's work because its screenshots were blank	checked the real files instead of trusting its summary	worktree reset	reverted, tightened the instructions for next time
```

## 行を記録する（Logging a row）

各エントリーは、自分がしたことをチームメイトに伝えるように書く。平易な言葉と具体的な行動で書き、AI っぽい言い回しや抽象的な専門用語は使わない（ログの文面にも **unslop** スキルを適用する）。

ヘルパーの `scripts/log.sh <logfile> <phase> <decision> <why> <evidence> <result>` を使う。このスクリプトは `ts` を記入し、初回にヘッダーを書き、紛れ込んだタブや改行を取り除き、`=`、`+`、`-`、`@` で始まるセルの先頭にシングルクォートを付ける。素の `printf` で行を追記してもよいが、セルが生成された文章やユーザーが渡した文章から来るなら、同じバイトに注意する。

記録するのは、すべての行動ではなく、決定の時点とチェックポイントである。たとえば、分かれ道での選択、検証の結果を伴う単位の完了、きっかけを伴う方針転換やリバート、見つかった障害、直したゲートである。ループで回す実行では、反復 1 回につき 1 行にする。些細なことや自明なことは省く。

1 回の実行（run）とは、エージェントとの 1 つの会話のことで、その後のターンや会話の要約も含む。作業の引き継ぎ、交代したエージェント、新しいチャットは、新しい実行を始める。すでに行があるログに実行が追記するときは、その最初の行の phase を `start` にする。別の実行の `start` 行のあとに書く最初の行も同様である。そのため、後のターンでログに戻ってきた実行は、まずログの最後の数行を読み、そのあいだに別の実行が書いていないかを確かめる。`start` 行には、その前にある、この実行が書いていない行の `ts` の範囲を書き、evidence にはエージェントの id など、この実行を示すものを書く。phase の `start` は、ほかの用途には使わない。

## 置き場所（Where it lives）

既定では、ログはコミットしない作業用の成果物である。作業ディレクトリの `decisions.tsv` に置くか、複数の作業が同時に進むときは `.audit/<task-slug>.tsv` に置き、git には入れない。

コミットするのは、作業の規模が大きく、レビュアーが結果を信頼するのに記録が必要なときだけである。

## 規則（Rules）

- 追記のみ。誤った判断には、それを置き換える新しい行を足す。履歴は決して編集も削除もしない。
- 手作りの使い捨てより、コミット済みのスクリプトが生成する証拠を優先する（**encode-lessons-in-structure** 原則スキル。教訓を構造に埋め込む）。

## トランスクリプトと照らしてログを監査する（Audit the log against the transcript）

実行の終わりに、作業を引き渡す前に、ログが事実を書いているかを確かめる。アクティブなワークスペースの `agent-transcripts/` ディレクトリにある、この実行のトランスクリプトを読む（パスはシステムプロンプトに書いてある）。`~/.cursor/projects/*/` をまたいで glob しない。関係のない非公開のチャットまで読んでしまう。この実行の行を、実際に起きたことと照らし合わせていく。この実行の行のまとまりは、それぞれ、この実行の `start` 行のどれか（この実行がログを作ったなら最初の行）から始まり、別の実行の次の `start` 行で終わる。

- どの行も、実際の決定か行動に対応していることを確かめる。
- 各行の証拠がたどれて、その行の主張を示していることを確かめる。
- 作業を方向づけたのに記録されていない分かれ道、方針転換、放棄した方法は、抜けである。それを追記する。

直すのは話の筋ではなく、ログである。監査では、作り話の行であっても、決して行を編集も削除もしない。実際の決定も実際の行動も記録していない行や、主張や証拠が間違っている行には、実際に起きたことと、たどれる参照先を書いた行を足して置き換える。この監査は、この実行のまとまりの外にある行は確かめない。この実行自身の作業からそのどれかが間違っていると分かったら、ほかの誤った判断と同じように、新しい行で置き換える。

## 別のモデルによる記録のレビュー（Cross-model review of the trail）

作業を引き渡す前に、作業をしたモデルとは別のベンダーのモデルでサブエージェントを起動する。セルフレビューでは代わりにならない。サブエージェントは監査の記録と実行のトランスクリプトを読み、ユーザーが注意すべき点を指摘する。作業のやり直しではなく、最善でない点や危うい点を探す走査である。各サブエージェントの推論のレベルは、**poteto-mode** スキルのサブエージェント（Subagents）の節に従って、仕事に合わせて選ぶ。

- 証拠が弱い、あるいは無いまま記録された決定。
- 省かれた検証の手順や、トランスクリプトに証明が無いのに行ったと主張された検証の手順。
- あとから見ると危うい選択（時期尚早、範囲の広がり、症状を取り繕っただけのもの）。
- ユーザーがざっと目を通しただけでは見落とす抜け。

記録を残した実行の返答は、どれも「Attention」（注意点）の節で終える。最初に、レビュアーのモデルを 1 行で書き（`reviewed by <model>`）、そのあと、具体的な行や時点を指す指摘を 1 つずつ並べる。指摘については「No flags」（指摘なし）と書いてよいが、モデル名はそうはいかない。

## 記録をレビューする（Reviewing the trail）

上から下へ読み、証拠の参照先をたどり、抜き取りで確かめる。コミットした TSV は、GitHub が表として表示する。ターミナルでは `column -s$'\t' -t decisions.tsv` で表示できる。

## このスキルを組み合わせる（Composing this skill）

ほかのスキルは、監査の記録を独自に作らず、このスキルで残す。名前で参照し、書式の決定はこのスキルに任せる。列の説明を繰り返さない。
