---
name: principle-type-system-discipline
description: "型を設計するとき、関数のシグネチャをレビューするとき、静的型付け言語でコードを書くときに適用する。不正な状態を表現できなくし、意味を持つプリミティブにブランド型を付け、外部のデータを境界でパースし、バリアントを網羅し、正となるスキーマから型を導く。"
disable-model-invocation: true
---

# 型システムの規律（Type System Discipline）

In Claude Code, read [the Claude Code adapter](../../docs/harness/claude-code.md) once per session before you follow this skill. It maps the Cursor names in pstack skills to Claude Code.

ありえない状態、取り違えたプリミティブ、扱っていないバリアントを、コンパイラが拒否するようにする。

- **意味を持つプリミティブにブランド型を付ける。** 2 つの引数のプリミティブ型が同じでも意味が違うなら、それぞれに専用の型を与える。検証は生成時に 1 度だけ行う。
- **外部のデータは、パースするまで型がない。** JSON、RPC のペイロード、CLI の引数、設定、環境変数、データベースの行は、境界で、型のついたモデルを返すパース関数を通す。
- **正となるスキーマから型を導く。** データの形をスキーマファイルが定めているなら、並行する型を手書きせず、そのスキーマから型を導く。

TypeScript の書き方は、**typescript-best-practices** スキルを参照する。
