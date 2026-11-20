# 自分のやり方に合わせる（Make it yours）

poteto-mode は、ある 1 人のやり方を表したものである。その下にあるプレイブック、振り分け、モデルのロールは、読者のやり方でも同じように使える。このページでは、自分用のモードの生成、セッションから得た教訓の記録、エージェントが同じ間違いを繰り返さないためのリポジトリの修正、目的を絞ったスキルの作成、スキルの変更を信頼する前のテストを扱う。

思っているより小さく始める。初日からたくさんのスキルは要らないし、このプラグイン全体すら要らない。素直にプロンプトを書き、エージェントがどこで失敗するかを観察し、同じ失敗が 2 回起きたらスキルかチェックを足す。

## `/automate-me` で自分のモードを生成する（Generate your own mode with `/automate-me`）

```text
/automate-me
```

自分のやり方を説明する必要はない。[`/automate-me`](../../skills/automate-me/SKILL.md) が履歴から読み取るからである。開いているワークスペースの最近のトランスクリプトを調べ、返答、委任、検証、コード、文章、進め方について、繰り返し現れる好みを拾い出し、どのパターンが本当に自分のものかを読者に尋ねる。そのうえで、Cursor に組み込まれた `create-skill` の流れで `.cursor/skills/<your-name>-mode/SKILL.md` の下書きを作り、その下書きを [`/unslop`](../../skills/unslop/SKILL.md) にかけ、worktree から PR を開く。読者はそれを、ほかの変更と同じようにレビューする。

自分の習慣が変わってきたら、いつでももう一度実行する。

```text
/automate-me update my mode skill with everything since its last edit
```

更新モードでは、スキルが最後に変わってからの履歴だけを調べる。読者の行動と食い違っていないルールは残し、新しい証拠があるルールは改め、新しいパターンについてだけ節を追加する。

## `/reflect` でセッションの教訓を記録する（Capture a session's lessons with `/reflect`）

何かを学んだタスクの直後に、次を実行する。

```text
/reflect that took way too long. capture what we learned so the next run doesn't repeat it.
```

[`/reflect`](../../skills/reflect/SKILL.md) はトランスクリプトを読み、今後のエージェントの作業を変えるような教訓だけを残し、提案を `Accepted`、`Rejected`、`Backlog` に分ける。スキルを変える前には、読者の承認を待つ。提案を承認するのは、それが今後の判断を変える場合だけにする。1 回の変わったセッションは逸話であって、ルールではない。

## `/correct` で環境を直す（Fix the environment with `/correct`）

同じ間違いについて何度もエージェントを正しているなら、直すべき場所は次のプロンプトではなくリポジトリである。選択肢を、どれだけ確実に間違いを防ぎ続けられるかの順に並べる。

1. アーキテクチャや、より良いデータ構造で、間違いを起こせないようにする。
2. 型で、あるいはエラーメッセージに直し方が書かれた lint や CI のチェックで防ぐ。
3. テストで捕まえる。
4. ドキュメントやエージェント向けのルールとして書き残す。エージェントがルールを読み飛ばしても何も失敗しないので、これは最後になる。

人間のレビューはこの一覧にない。PR のたびに同じ間違いを指摘しなければならないレビュアーこそ、これで解消したい問題だからである。この作業は [`/correct`](../../skills/correct/SKILL.md) が行う。

```text
/correct agents keep calling the database client directly instead of going through the repository layer
```

`/correct` は、最近のコミット、リバート、レビューコメント、回避策を説明しているコメントを読み、間違いを種類ごとにまとめる。2 回起きたら、1 つの種類として数える。頻度の高い種類から、1 種類につき 1 コミットで、通用する中でいちばん上の段階の方法で直し、新しいチェックが過去の実際の間違いで失敗することを証明する。また、エージェント向けの指示ファイルに、各ルールとそれを強制する仕組みを対応づけた表を置く。そのため、何にも強制されていないルールは、間違いの繰り返しとして見えてくる。返答には、種類ごとに、その証拠、選んだ段階、より上の段階が使えなかった理由が並ぶ。

引数なしで実行すると、履歴から間違いの種類を自分で見つける。`/reflect` と `/correct` は役割を分担している。`/reflect` は 1 つのセッションからスキルを改善する。`/correct` はリポジトリを変えて、ある種類の間違いが二度と起きないようにする。修正が新しい境界を設けることになるときは、`/architect` と組み合わせる。[多くのプロジェクトを並行して動かす（Run many projects in parallel）](./07-overnight.md#多くのプロジェクトを並行して動かすrun-many-projects-in-parallel)に、リポジトリ全体でこの両方を行うプロンプトがある。

## 目的を絞ったスキルを書く（Author a focused skill）

残しておきたい作業の流れがすでに分かっているときは、次のようにする。

```text
/poteto-mode write a skill for verifying database migrations in this repo
```

スキルを書く依頼は、[スキルの作成・変更（Authoring or modifying a skill）プレイブック](../../skills/poteto-mode/playbooks/authoring-a-skill.md)に当てはまる。このプレイブックは Cursor に組み込まれた `create-skill` を通し、frontmatter とリンクを検証し、PR を開く（Opening a PR）プレイブックを通して結果を出す。エージェント向けの文章には、人間向けの文章より高い基準が求められる。役に立たない一文が、将来どこかのエージェントが従う指示になってしまうからだ。`SKILL.md` を自己流で書かず、その基準はプレイブックに守らせる。

2 つの特別な場合には、専用の生成スキルがある。アプリを操作して振る舞いを証明しなければならないスキルは検証スキルなので、代わりに [`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md) と [`/maintain-verification-skill`](../../skills/maintain-verification-skill/SKILL.md) を使う。モデル検査器、証明、その他の検証器でコードの性質を確かめるスキルは形式検証スキルなので、[`/create-formal-verification-skill`](../../skills/create-formal-verification-skill/SKILL.md) と [`/maintain-formal-verification-skill`](../../skills/maintain-formal-verification-skill/SKILL.md) を使う。どちらも [検証してリリースする（Verify and ship）](./06-verify-and-ship.md#形式検証スキルで性質を証明するprove-a-property-with-a-formal-verification-skill)で扱っている。

## `/technical-writing` で基準に沿ったドキュメントを書く（Write docs to a standard with `/technical-writing`）

読者が出す文章はスキルだけではない。ドキュメント、RFC、README、PR の説明、コミットメッセージには、次のようにする。

```text
/technical-writing review the readme changes
```

[`/technical-writing`](../../skills/technical-writing/SKILL.md) は、何層かからなる基準を適用する。目指すのは、疲れたエンジニアが一度読んで理解できる文章である。まず文書の種類（チュートリアル、ハウツー、リファレンス、解説）を決め、そのあと 1 文ずつ見ていく。誰が何をするか、1 文に 1 つの考え、2 通りに読める箇所を残さない、という観点である。自分やエージェントが書いたばかりのものをレビューするのに使うか、ドキュメントを頼むときに最初から名指しする。

## スキルの変更をブラインドでテストする（Test a skill change blind）

スキルの編集は、この先のすべてのセッションに影響する。だから、実験として扱ってテストする。ほかの人のスキルを取り入れるときも同じである。手元に残す前に、自分の作業でエージェントが良くなることを確かめる。

```text
/poteto-mode run the eval playbook on this skill change. same task for both variants, candidates stay blind.
```

スキルが何度も期待を外し、どうすべきかが分かっているときは、変更とテストを 1 つのタスクで行う。

```text
/poteto-mode update the review skill so it flags missing migrations, and eval the change.
```

最初から評価を頼んでおけば、編集が都合のよいものにならない。1 回のまずいセッションから書いた修正は、そのセッションに過剰に合わせたものになりがちで、編集を重ねるうちにスキルが本来の形からずれていく。評価を挟めば、そのずれを出す前に見つけられる。

[評価（Eval）プレイブック](../../skills/poteto-mode/playbooks/eval.md)は、1 つの失敗の型、つまり観察者効果を中心に組み立てられている。評価されていると知っているエージェントは、振る舞いが変わる。そのため、候補のエージェントには、手がかりを取り除いたディレクトリで、自然に見えるタスクを与える。「eval」や「candidate」という語は決して見せず、ほかの候補がいることも決して知らせない。1 つのジャッジが、中立なラベルを付けたすべての出力を採点する。参照の連鎖をたどったかどうかは、候補が主張する内容ではなく、各候補が実際に読んだファイルから採点する。

判定を受け入れる前に、すべての出力を自分で読む。ジャッジと意見が合わないときは、自分の判断を疑う前に評価基準を疑う。

## `/make-bot-ui` でボットの UI を作る（Build a bot UI with `/make-bot-ui`）

状況を選ぶスキルの 1 つに、Grok Bot の利用者向けのものがある。[`/make-bot-ui`](../../skills/make-bot-ui/SKILL.md) は、ボタンを押すと webhook のルーチン経由でボットを起こす小さなページを作る。たとえば、レビューのキューをスワイプしていき、スワイプするたびに、その項目への対応をボットに頼む、といった使い方ができる。webhook の送信用の鍵は読者のマシン上のサーバーが持つので、鍵がブラウザやチャットに渡ることはない。このスキルは、ページを Tailscale で公開する方法も扱う。

**落とし穴：** スキルの挙動がおかしいからといって、タスクの途中でスキルを編集しない。スキルは専用の PR で直し、タスクは先に進める。機能開発の作業に紛れ込んで出たスキルの編集は、レビューで見えず、評価もできない。

次: [レシピと落とし穴](./10-recipes-and-pitfalls.md)
