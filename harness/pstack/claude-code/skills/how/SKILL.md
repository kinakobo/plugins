---
name: how
description: "「X はどう動くのか」、何かを変える前のコードの読み解き、配置・所有・レイヤーについての問い（「これはどこに置くべきか」「どのパッケージがこれを持つのか」「これは正しいレイヤーか」）に使う。サブシステムのアーキテクチャ、実行時の流れ、新しく加わる人向けのメンタルモデルを説明する。動機については why を使う。"
disable-model-invocation: true
---

# どう動くか（How）

In Claude Code, read [the Claude Code adapter](../../docs/harness/claude-code.md) once per session before you follow this skill. It maps the Cursor names in pstack skills to Claude Code.

コードベースを調べて、「X はどう動くのか」という問いに答える。

範囲があいまいなら、自分の解釈を述べてから調べる。方向が違えば、ユーザーが修正できる。

## 1. 説明する（Explain）

`subagent_type: generalPurpose` と `readonly: true` を付けた `Task` サブエージェントを 1 つ起動し、調査と説明を一度に行わせる。ユーザーの問いと、次の依頼文を渡す。

> あなたは、シニアエンジニアに向けてアーキテクチャの説明を書いている。コードベースを調べ、この領域に詳しくないシニアエンジニアが読み終えたときに、確かなメンタルモデルを得られる説明を書く。読み手は、自信を持ってこの領域で作業を始められる程度に、アーキテクチャを理解できなければならない。コードベースへのアクセスは読み取り専用である。

## 2. 提示する（Present）

サブエージェントの出力を提示する。わかりやすくするための軽い編集や、会話の文脈を補う程度の編集はよい。大幅に書き直さない。
