# pstack のガイド（The pstack guide）

pstack が最もうまく働くのは、エージェントを細かく管理するのをやめたときである。ユーザーは、何をしてほしいかと、終わったことをどう確かめるかを書く。`/poteto-mode` はプレイブックを選び、手順が必要とするときにほかのスキルを実行し、証拠を示す。このガイドは、現実的なプロンプトを例に、その習慣を教える。

このガイドで学ぶことは次のとおりである。

1. [pstack をセットアップする](./01-setup.md)。プラグインをインストールし、モデルを選ぶ。
2. [`/poteto-mode` を通して作業を進める](./02-poteto-mode.md)。目標を与え、プレイブックを選ぶ様子を見る。
3. [コードを理解する](./03-understand.md)。何かを編集する前に、読み取りだけの調査を行い、続いて `/how`、`/why`、`/teach`、`/recall` を使う。
4. [変更を設計する](./04-design.md)。コードで形が固まる前に、`/architect`、`/arena`、`/swarm`、`/interrogate`、プロトタイプ、計画を使う。
5. [変更を作り、整える](./05-build-and-clean.md)。作るためのプレイブック、`/tdd`、`/unslop`、`/no-comments`。
6. [検証してリリースする](./06-verify-and-ship.md)。実際のアプリで振る舞いを証明し、テストではめったに届かない部分を形式検証し、`/benchmark-checklist` で数値を吟味してから、焦点を絞った PR を開いてマージまで進める。
7. [寝ているあいだに作業を進める](./07-overnight.md)。ループの前に信頼を得ること、夜間の無人実行の取り決め、監査できる決定ログ、そして 1 つのエージェントを超えて規模を広げる Projects とオートメーション。
8. [原則の名前でエージェントを導く](./08-principles.md)。作業の途中でエージェントの向きを変える 24 の名前。
9. [自分のものにする](./09-make-it-yours.md)。自分専用のモード、繰り返される間違いのための `/correct`、スキルの変更を試す方法。
10. [レシピと落とし穴](./10-recipes-and-pitfalls.md)。そのまま使えるプロンプトと、避けるべき間違い。

最初はページを順番に読む。その後は、どのページも単独で読める。

行き詰まったときや、どのスキルが合うかわからないときは、[`/poteto-help`](../../skills/poteto-help/SKILL.md) に続けて質問を入力する。

```text
/poteto-help which skill should i use to review this branch?
```

このスキルは質問に答え、送るためのプロンプトを渡し、答えの出典となったスキルかガイドのページへのリンクを示す。pstack の実行は実際にトークンを消費するので、このスキルは作業を始めない。準備ができたら、ユーザーがそのプロンプトを送る。このスキルは、入力したときだけ動く。

## 1 つだけ覚えるなら（If you only remember one thing）

目標と、それを確かめる方法を、自分の言葉でエージェントに伝える。

```text
/poteto-mode the export writes duplicate rows when a retry lands mid-run. repro first, then fix and verify.
```

プレイブックの名前を挙げたり、スキルを列挙したりする必要はない。`/poteto-mode` が振り分けに必要とする手がかりは、「repro first」（まず再現する）と、確かめられる結果だけである。`/poteto-mode` はバグ修正（Bug fix）のプレイブックを選び、手順を todo list に写し、各手順の実行に合わせて適切なスキルを呼ぶ。

次: [pstack をセットアップする](./01-setup.md)。
