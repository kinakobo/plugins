---
name: swarm
description: "N 個のワーカーを並列にファンアウトし、すべて終わるまで待って結果を集め、1 つの報告を返す。/swarm、「swarm this」、または並列の網羅、競争（race）、連続した試験（gauntlet）、探索に使う。"
disable-model-invocation: true
---

# 並列のワーカー群（Swarm）

In Claude Code, read [the Claude Code adapter](../../docs/harness/claude-code.md) once per session before you follow this skill. It maps the Cursor names in pstack skills to Claude Code.

## 開始（Start）

何かを起動する前に、フェーズごとに 1 項目ずつの todolist を開く。

1. 枠組みを決める（Frame）
2. ファンアウト（Fan out）
3. 集約（Aggregate）
4. 報告（Report）

## フェーズ A: 枠組みを決める（Phase A: Frame）

1. 完了の条件と、swarm が返すべき成果物または報告を述べる。
2. 形を選ぶ。作業をスライスに分割するか、同一の依頼文で N 個のワーカーを競わせるか、両方を組み合わせる。競争か組み合わせの形なら、起動する前に `first pass`、`rank all`、`best-of` のどれにするかを宣言する。
3. ワーカーのモデルは、`~/.cursor/rules/pstack-models.mdc` の `swarm workers` の行から選ぶ。ルールかその行がなければ、`claude-opus-5-5-high-fast` を使う。`auto` か `inherit-parent` なら、ワーカーが親のモデルで動くように `model` を省く。`Task` ツールが slug を拒否したら、既定のモデルを使い、そうしたことを伝える。既定のモデルも拒否されたら、エラーメッセージにある同じファミリーの有効な slug のうち、最も近いものを使う。モデルを競わせるときは、各アームのモデルを最初に明示する。各サブエージェントの推論のレベルは、**poteto-mode** スキルのサブエージェント（Subagents）の節に従って、作業に合わせて選ぶ。
4. ワーカーが書き込むときは、ワーカーごとに専用の書き込み先を与える。ワーカーがコミットを検証または測定するときは、各依頼文で対象の SHA を正確に指定する。測定の依頼文では、方法（サンプル数、1 サンプルが何か、順序）も指定する。ワーカーは、その両方を結果に記録する。

## フェーズ B: ファンアウト（Phase B: Fan out）

N 個のワーカーすべてを 1 つのメッセージの中で起動し、`subagent_type: generalPurpose`、`environment: "cloud"`、`run_in_background: true`、手順 3 のモデル（`auto` か `inherit-parent` なら指定しない）を付ける。`environment: "local"` は、ワーカーがユーザーのコンピューター上の何かにアクセスする必要があるときだけ使う。

ワーカーが既定以外の push 済みのブランチから始めなければならないときは、`cloud_base_branch` を渡す。

どの依頼文も、それだけで完結させる。目標、範囲、担当するスライスまたは競争のアームの正確な指定、検証の方法、報告する内容を含める。報告には、証拠とともに `PASS`、`ISSUES`、`BLOCKED` のいずれかを使う。欠陥を証明できるワーカーは `ISSUES` を報告し、最初の 1 件だけでなく、証明できる問題をすべて挙げる。

ワーカーが脱落したら、N-1 個で進め、そのことを記録する。

## フェーズ C: 集約（Phase C: Aggregate）

終了したワーカーの結果を読む。依頼文が指定した SHA と方法を記録していない結果は捨て、そのワーカーを 1 回だけ起動し直す。2 回目も記録がなければ、欠落（gap）として記録する。欠落は成功とは数えない。網羅が目的なら、必要なスライスすべてに結果が必要である。競争なら、最初に宣言した選択の規則を適用する。first pass、rank all、best-of のいずれかを使う。ワーカーの生の出力を貼り付けない。

## フェーズ D: 報告（Phase D: Report）

チャットの中で、1 つにまとめた報告を返す。結果の表、問題ごとの 1 行の要約、欠落や脱落、競争をした場合はその規則を含める。
