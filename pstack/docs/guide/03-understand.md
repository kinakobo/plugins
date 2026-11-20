# 変更する前にコードを理解する（Understand the code before changing it）

気づきにくいリグレッションが出てしまうのは、理解していないコードを編集したときである。これはユーザーにもエージェントにも同じように当てはまる。エージェントの失敗は、たいてい 2 つのどちらかである。ユーザーの望みを読み違えるか、作業を正しく行うためのコンテキストを持っていないかである。前者は [プロンプトに何を入れるか（What goes in a prompt）](./02-poteto-mode.md#プロンプトに何を入れるかwhat-goes-in-a-prompt) で扱った。このページでは後者を扱う。

pstack には、そのための入口が 4 つある。`/how` は、コードがいま何をしているかを説明する。`/why` は、コードがその形になった理由を探し出す。`/teach` は、その両方を 1 つの説明にまとめる。`/recall` は、あるトピックについてのユーザー自身の最近のコンテキストを組み立て直す。どれも、ユーザーが確かめられる言葉でエージェントに説明させる。そうすることで、ユーザーよりコードに詳しいかもしれないエージェントを監督できる。

![ロボットたちが事件ファイルを取ってくるあいだ、探偵が虫眼鏡で機械の設計図を調べている。背後の証拠ボードでは、/how と /why の下に手がかりが結ばれている。](./images/understanding.jpg)

## 読み取り専用の調査から始める（Start with a read-only investigation）

原因がはっきりしないときは、修正ではなく調査結果を求める。

```text
/poteto-mode investigate why background jobs time out every few hours. give me what we know, what data you used, and your best hypotheses. don't change any code yet.
```

「don't change any code yet」（まだコードを変えない）があるので、このプロンプトは [調査（Investigation）のプレイブック](../../skills/poteto-mode/playbooks/investigation.md) に振り分けられる。このプレイブックはコードを追い、動機についての問いには `/why` を加え、出典を示した説明を返す。選択肢から選ぶ問いなら、トレードオフの表を付けた推奨を返す。「what data you used」（どのデータを使ったか）と尋ねると、エージェントは証拠と推測を分けて示す。調査結果が修正を指し示していたら、その修正は新しいタスクとして始める。

## `/how` で振る舞いを追う（Trace behavior with `/how`）

```text
/how do we dedupe notifications? is there an n+1 when we look up subscribers?
```

実際に抱えている疑問をそのまま尋ねる。[`/how`](../../skills/how/SKILL.md) はコードを読み、そのサブシステムを初めて担当する人にシニアエンジニアが説明するときの水準で、実行時の流れ、主要な型、わかりにくい箇所を答える。コードを調べて説明を書く読み取り専用のサブエージェントを 1 つ起動する。

## `/why` で経緯を調べる（Dig up history with `/why`）

```text
/why was the retry limit set to five? does the reason still hold?
```

[`/why`](../../skills/why/SKILL.md) は、コードではなく記録から答える。そのコードを導入したコミットと PR の議論、およびその後の変更ごとのコミットと PR の議論を読み、そうした記録が乏しければほかの情報源も調べる。回答は、作者が書いたことを引用し、エージェント自身の推論には推論であると明記し、記録からは答えが出ないことも述べる。「なぜそうしたのか誰も書き残していない」ということ自体が答えになるからである。

この 2 つは自然に組み合わせられる。コードが込み入っている理由が経緯で説明できそうなときは、`do why first then how` がよいプロンプトになる。

## `/teach` で本当に理解する（Actually understand it with `/teach`）

```text
/teach me how this PR changes retries. convince me it fixes the cause and not the symptom.
```

[`/teach`](../../skills/teach/SKILL.md) は、要約では足りないときに使う。`/how` と `/why` を実行し（小さな変更ならどちらか一方だけのこともある）、その結果を、図を 1 枚ずつ積み上げていく平易な説明にまとめる。「convince me」（納得させて）という頼み方は、ほかでも使う価値がある。説明が、ひととおり案内して終わるものではなく、検証できる論証になる。

エージェント自身の選択にも使える。

```text
/teach me why you implemented it this way and not with a queue. what did you trade off, and why?
```

教えることは、ユーザーと同じくらいエージェントにも役立つ。自分の作業を説明しなければならないエージェントは、主張を自信ありげに述べて先へ進むのではなく、コードを読み、主張の 1 つ 1 つを証拠で裏付けなければならない。

## `/recall` で自分のコンテキストを組み立て直す（Rebuild your own context with `/recall`）

```text
/recall catch me up on the export work from last week
```

[`/recall`](../../skills/recall/SKILL.md) は、ユーザー自身の最近のチャットと共有の記録（issue、過去の修正、まだ発生し続けているエラー）を調べ、現状と次にやることをまとめた概要を返す。古いチャットには、新しいエージェントが持っていないコンテキストがある。そのため、古いトピックで新しい作業を始めるときは、まずそのコンテキストを読み込ませてから、新しい入力を渡す。

```text
/recall my work on the virtualized list from yesterday, then read this bug report.
```

特定の 1 つのチャットを再開したいなら、使うのは `/recall` ではなく、下で説明するセッションの引き継ぎ（Session pickup）のプレイブックである。

## 前の作業をセッションの引き継ぎで受け継ぐ（Take over prior work with Session pickup）

ほかのエージェント（あるいは先週のユーザー自身）が、ブランチを作業の途中で残していったときは、次のように頼む。

```text
/poteto-mode take over this branch. read the decision log, figure out what's done, and continue from there. don't redo finished work.
```

[セッションの引き継ぎ（Session pickup）のプレイブック](../../skills/poteto-mode/playbooks/session-pickup.md) は、前の作業の記録を正として扱う。ブランチの状態と決定を復元し、再開する地点を特定し、すべてを一から導き直すのではなく、引き継いだ主張を元の目標に照らして確かめる。

**落とし穴：** 「どうせエージェントがコードを読む」からといって、このページのスキルを飛ばさない。コードを追って動きを把握しないまま編集を始めたエージェントは、最初にもっともらしく見えた箇所で症状だけを直しがちである。先に `/how` を実行するほうが、2 つ目のバグを出すより安くつく。

次は [変更を設計する（Design the change）](./04-design.md)。
