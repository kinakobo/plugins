# 原則の名前で舵を取る（Steer with principle names）

pstack は 24 の原則を、それぞれ独立したスキルとして同梱している。`/poteto-mode` は、複数の手順からなるタスクを始めるたびに原則の索引を読み、タスクに当てはまる原則を適用し、返答の中で、適用した原則の名前を、それによって変わった決定とともに挙げる。

原則は呼び出すものではない。名前を使って舵を取るものである。それぞれの名前は、エージェントがすでに読んだ完全なルールを指しているので、一言のほうが、一段落の指示よりも正確に作業の向きを変えられる。

## 実際の舵取り（Steering in practice）

エージェントが、既存の 3 つのアダプターに、新しいアダプターを継ぎ足そうとしているとする。

```text
use subtract before you add. delete the obsolete adapters first, then design what's left.
```

ビルドが通ったことを理由に、成功したと言っているとする。

```text
apply prove it works. run the real import flow and show me the written records.
```

並行する 2 つの試みが、同じブランチに書き込もうとしているとする。

```text
separate before serializing shared state. give each attempt its own worktree, no locks.
```

どの一言も、背後のルールが具体的だから伝わる。それでもエージェントは、そのルールがどの決定を変えたかを返答で述べなければならない。変わった決定を伴わずに原則を挙げているなら、エージェントはルールの名前を出しただけで、適用はしていない。

## 24 の原則の概要（The 24, briefly）

中核の原則で、どれだけ作るか、いつ設計を考え直すかを決める。

- [怠惰のプロトコル（Laziness Protocol）](../../skills/principle-laziness-protocol/SKILL.md)は、削除と、問題を解決する最小の変更を優先する。
- [基礎から考える（Foundational Thinking）](../../skills/principle-foundational-thinking/SKILL.md)は、ロジックを書く前に、中心となるデータ構造を選ぶ。
- [第一原理から設計し直す（Redesign from First Principles）](../../skills/principle-redesign-from-first-principles/SKILL.md)は、新しい要件を、最初からあったかのように組み込む。
- [前提を疑う（Attack the Premise）](../../skills/principle-attack-the-premise/SKILL.md)は、2 回以上失敗した修正が共有していた前提を書き出し、次の修正の前にその前提を直接検証する。
- [足す前に引く（Subtract Before You Add）](../../skills/principle-subtract-before-you-add/SKILL.md)は、その上に何かを作る前に、不要なものを取り除く。
- [読み手の負荷を最小にする（Minimize Reader Load）](../../skills/principle-minimize-reader-load/SKILL.md)は、読み手が頭の中に保持しなければならない層や隠れた状態をまとめて減らす。
- [成果志向で実行する（Outcome-Oriented Execution）](../../skills/principle-outcome-oriented-execution/SKILL.md)は、書き直しを目標の設計へ収束させ、使い捨ての互換性のための中間状態を保たない。
- [体験から始める（Experience First）](../../skills/principle-experience-first/SKILL.md)は、実装の都合よりも、ユーザーが得る結果を選ぶ。
- [設計空間を探り尽くす（Exhaust the Design Space）](../../skills/principle-exhaust-the-design-space/SKILL.md)は、前例がないときに、競合する 2〜3 個のプロトタイプを作る。
- [てこを作る（Build the Lever）](../../skills/principle-build-the-lever/SKILL.md)は、作業を行う、または作業を証明するスクリプトを作り、レビュアーが再実行できるようにする。エージェントが同じことを手作業で繰り返しているなら、エージェント自身があればよいと思うツールやスキルを書かせる。ある手順をスクリプトで毎回同じようにこなせるなら、スクリプトを使い、エージェントは判断が要る場面に回す。

アーキテクチャの原則で、状態、検証、互換性をどこに置くかを決める。

- [ドメインをモデル化する（Model the Domain）](../../skills/principle-model-the-domain/SKILL.md)は、繰り返し出てくるルールを、散らばった条件分岐ではなく 1 つの構造に表す。
- [境界の規律（Boundary Discipline）](../../skills/principle-boundary-discipline/SKILL.md)は、境界で検証し、内部の型は信頼する。
- [型システムの規律（Type System Discipline）](../../skills/principle-type-system-discipline/SKILL.md)は、不正な状態を表現できないようにする。
- [操作を冪等にする（Make Operations Idempotent）](../../skills/principle-make-operations-idempotent/SKILL.md)は、リトライしても同じ最終状態に収束するようにする。
- [呼び出し元を移してからレガシー API を消す（Migrate Callers Then Delete Legacy APIs）](../../skills/principle-migrate-callers-then-delete-legacy-apis/SKILL.md)は、移行と削除を同じ一連の作業の中で行う。
- [共有状態は直列化する前に分離する（Separate Before Serializing Shared State）](../../skills/principle-separate-before-serializing-shared-state/SKILL.md)は、調停の仕組みを足す前に、共有そのものをなくす。

検証の原則で、何を証明とみなすかを定める。

- [動くことを証明する（Prove It Works）](../../skills/principle-prove-it-works/SKILL.md)は、代わりの指標ではなく、実際の成果物を検証する。
- [根本原因を直す（Fix Root Causes）](../../skills/principle-fix-root-causes/SKILL.md)は、コードを変える前に、問題を再現して原因までたどる。
- [検証できる単位で順に進める（Sequence Work into Verifiable Units）](../../skills/principle-sequence-verifiable-units/SKILL.md)は、小さな単位をそれぞれチェックで締めくくってから、次の単位を始める。
- [実装ではなく振る舞いをテストする（Test Behavior, Not Implementation）](../../skills/principle-test-behavior-not-implementation/SKILL.md)は、コードの利用者が観察できることを検証する。そうすれば、テストはその振る舞いが壊れたときに失敗し、内部を変えても通り続ける。
- [数値を説明する（Explain the Number）](../../skills/principle-explain-the-number/SKILL.md)は、誰かが測った数値を信頼したり報告したりする前に、その数値を制限しているものを特定し、別のものを測っていた可能性を排除する。[`/benchmark-checklist`](../../skills/benchmark-checklist/SKILL.md) は、これを、実際の実行から答える 7 つの問いにしたものである。

委任の原則で、並行作業が混乱しないようにする。

- [コンテキストウィンドウを守る（Guard the Context Window）](../../skills/principle-guard-the-context-window/SKILL.md)は、大量の読み込みをサブエージェントに振り分け、メインのチャットには発見だけを残す。
- [人間を待って止まらない（Never Block on the Human）](../../skills/principle-never-block-on-the-human/SKILL.md)は、やり直しのきく作業は先に進め、結果を示す。

そして、メタな原則が 1 つある。

- [教訓を構造に埋め込む（Encode Lessons in Structure）](../../skills/principle-encode-lessons-in-structure/SKILL.md)は、2 回繰り返した助言を、lint、チェック、スクリプトに変える。[`/correct`](../../skills/correct/SKILL.md) は、これをリポジトリ全体に適用する。その使い方は [自分のやり方に合わせる（Make it yours）](./09-make-it-yours.md#correct-で環境を直すfix-the-environment-with-correct)で示す。

この一覧を覚える必要はない。いまはざっと目を通し、ここにある名前を出していれば防げたはずのことをエージェントがしているのに気付いたら、ここへ戻ってくる。そうすれば語彙が身につく。

次: [自分のやり方に合わせる](./09-make-it-yours.md)
