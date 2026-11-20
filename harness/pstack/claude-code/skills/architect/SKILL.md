---
name: architect
description: "コードを書く前に、型、シグネチャ、モジュールの構造を素描し、実装が埋まっていくあいだも関わり続ける。/architect、「architect this」「design this」、またはいきなりコードを書くと誤った形で固まってしまう自明でない作業に使う。"
disable-model-invocation: true
---

# 設計する（Architect）

In Claude Code, read [the Claude Code adapter](../../docs/harness/claude-code.md) once per session before you follow this skill. It maps the Cursor names in pstack skills to Claude Code.

コードを書く前に、変更が触れるシステムをすべてたどる。そのうえで、変更の形を 2 通り素描し（型、シグネチャ、モジュールの境界）、状態ごとに持ち主が 1 つで、タスクごとにやり方が 1 つで、呼び出し元が知るべきことが最も少ないほうを選ぶ。回避策が繰り返し必要になるなら、継ぎはぎせずに設計し直す。

大きな設計や意見の割れる設計では、1 人で素描するのではなく **arena** スキルを実行する。
