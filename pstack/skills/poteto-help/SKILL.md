---
name: poteto-help
description: pstack のセットアップ、/poteto-mode の使い方、タスクに合うスキル・プレイブック・原則の選び方を案内する。/poteto-help に続けて質問を入力する。
disable-model-invocation: true
---

# Poteto のヘルプ（Poteto help）

pstack についてのユーザーの質問に答え、そのまま送れるプロンプトを渡し、答えの出どころのファイルへのリンクを示す。ヘルプの質問では、作業を始めない。ユーザーはやり方を尋ねたのであり、pstack の実行は実際にトークンを消費するので、プロンプトはユーザー自身に送ってもらう。

「pstack でこのバグを直して」のように作業を頼むメッセージは、ヘルプの質問ではない。[`poteto-mode`](../poteto-mode/SKILL.md) を読み、それに従って作業し、Custom Mode にすれば有効なまま保てることを一度だけ伝える。

このファイルは、質問を、答えが書いてあるスキルとガイドのページに対応づける。詳細はそれらのファイルが受け持つ。案内する先のファイルは、引用する前に読み、この対応表と食い違うときはそのファイルを信じる。ここのリンクはインストールされたプラグインの中を指しており、ユーザーには開けないことがあるので、ユーザーにはそのファイルの公開版を渡す。公開版の URL は、`https://github.com/cursor/plugins/blob/main/pstack/` にそのパスを続けたものである。

## 何が必要かを見極める（Find out what they need）

メッセージと会話から、必要なものを推し量る。「PR をレビューするのはどのスキル？」のように状況がはっきりしていれば、その節に直行する。それでも分からなければ、次の選択肢で選択式の質問を 1 つだけし、選ばれた節だけに答える。

- セットアップする
- `/poteto-mode` でタスクを始める
- 状況に合うスキルを選ぶ
- うまくいかなかった実行を直す
- pstack を自分用にする

答えを左右する状態を確かめ、答えが変わるときだけそれを伝える。

- `~/.cursor/rules/pstack-models.mdc` がなければ、このユーザーはまだ `/setup-pstack` を実行していないので、どのロールも既定のモデルを使う。
- プロジェクトに `verify-*` スキルやほかのアプリ用のハーネスがなければ、エージェントにはアプリをスクリプトで操作する手段がない。質問が、変更が動くことの証明についてなら、`/create-verification-skill` を紹介する。
- プロジェクトに形式検証のスキルがなければ、テストが届く範囲を超えて検査される性質はない。質問が競合状態、不変条件、そのほかテストではめったに当たらないケースについてなら、`/create-formal-verification-skill` を紹介する。形式検証のスキルがあり、最後に同期したと記録しているコミットより後に対象のコードが変わっていて、質問がそのコードに関わるなら、`/maintain-formal-verification-skill` を紹介する。

モデルのルールがなく、それが答えに関わるときは、ロールごとのモデルと推論の予算をいま選びたいかをユーザーに尋ねる。関わるのは、ユーザーが初めて使うとき、質問がセットアップか費用についてのとき、どのモデルが動くかで答えが変わるときである。尋ねるのは 1 つのチャットにつき 1 回までにする。必要なものも分からないときは、2 つの質問をまとめて尋ねる。選択肢は 2 つ示す。

- 今すぐ：入力する `/setup-pstack` を渡し、質問にも答える。
- 後で：質問に答え、`/setup-pstack` を実行するまではどのロールも既定のモデルのままだと 1 行添える。

## セットアップする（Get set up）

1. チャットで `/add-plugin pstack` を実行するか、サイドバーの Customize からインストールする。
2. [`/setup-pstack`](../setup-pstack/SKILL.md) を実行する。推論の予算を尋ね、ロールごとにモデルを割り当て、ルールを書く。ルールは新しいチャットから適用される。その後、プロジェクトに検証スキルと形式検証のスキルがまだなければ、それぞれを生成するかを一度ずつ提案する。
3. `/poteto-mode` に、目標と、合否のつく確認を付けて、実際のタスクを始める。

インストールしただけでは、ユーザーがスキルを呼び出すまで何も変わらない。ユーザーの言葉から読み込まれるのは `/setup-pstack` だけである。詳しくは [README](../../README.md) と [ガイドの 1 ページ目](../../docs/guide/01-setup.md) にある。[`references/prompting.md`](references/prompting.md) に従い、最初のプロンプトの文面を一緒に考えることを申し出る。

費用が心配なら、トークンが何に使われ、どうすれば減らせるかを伝える。pstack は、サブエージェントとレビューのパネルに余分なトークンを使う。`/setup-pstack` をもう一度実行し、小さい予算か安いモデルを選ぶ。`auto` か `inherit-parent` を設定したロールはチャットのモデルで動くので、チャットが Auto か安いモデルで動いていればトークンを節約できる。パネルの一覧を短くすれば、起動するサブエージェントが減る（1 項目につき 1 つ）。`/poteto-mode` は、厳密さが必要な作業のためにとっておく。

pstack は Cursor 向けに作られているが、Claude Code にも `/plugin marketplace add cursor/plugins` の後に `/plugin install pstack@cursor-plugins` でインストールできる。Claude Code では、各コマンドに `pstack:` の接頭辞が付く（`/pstack:poteto-mode` など）。[Claude Code のアダプター](../../docs/harness/claude-code.md) が Cursor の名前を対応づけ、Bugbot や Automations のような Cursor だけの機能が必要な手順は、1 行の注記を残して飛ばされる。

スキルは Agent Skills の形式なので、ほかのツールでも読める。ただし、`/poteto-mode`、`/how`、`/teach` を含むほとんどのワークフローのスキルは Cursor のサブエージェントを起動し、その一部はロールごとのモデルを使う。また、Custom Mode と `/loop` は Cursor の機能である。そのため、これらの部分はほかのツールでは動かない可能性がある。

## `/poteto-mode` でタスクを始める（Start a task with `/poteto-mode`）

`/poteto-mode` は、タスクに合うプレイブックを選び、その手順を todo list に写し、手順が必要とするときにほかのスキルを実行する。良いプロンプトは、目標と、完了をどう判定するかを書く。スキルは列挙しない。手で書いた順序では、プレイブックなら守るはずの手順が抜けたり入れ替わったりしやすいからである。プロンプトの文面づくりを手伝う前に、[`references/prompting.md`](references/prompting.md) を読む。例は [ガイドの 2 ページ目](../../docs/guide/02-poteto-mode.md) にある。

`/poteto-mode` が有効なまま続くかどうかは、ユーザーがどう始めたかで決まる。

- `/poteto-mode` で Enter を押すと、スキルはその 1 つのメッセージに付く。チャットが進むにつれて、その影響は薄れていく。
- Mac で Option+Enter、Windows で Alt+Enter を押すか、スキルの項目から Use as Mode を選ぶと、Custom Mode になる。ユーザーがモードを抜けるまで毎ターンのコンテキストに残り、気軽なやりとりのターンには関わらない。
- Cursor のドキュメントでは、Custom Mode が使えるのは Agents Window と CLI とされている。それ以外では、新しいタスクのたびに `/poteto-mode` で始める。

この話題になったら、[Cursor のスキルのドキュメント](https://cursor.com/docs/skills) へのリンクを示す。チャットの途中で「new task」（新しいタスク）と言うと、モードはプレイブックを選び直す。`/poteto-mode` は、プレイブックの手順が起動するサブエージェントに、すでに `poteto-agent` を使っている。自分で起動するサブエージェントでも同じスタイルにしたいなら、`subagent_type: "poteto-agent"` を指定して起動する。

## スキルを選ぶ（Pick a skill）

既定の答えは `/poteto-mode` である。手順が必要とするときに、ほかのスキルの大半を実行するからである。ユーザーが何かをプレイブックより多く、または少なく求めるときは、スキルを直接名指しする。勧める前にそのスキルを読み、プロンプトの例を 1 つ示す。

| ユーザーがしたいこと | スキル |
|---|---|
| 自明でないタスクを厳密に進める | [`/poteto-mode`](../poteto-mode/SKILL.md) |
| 始める前に、エージェントが目標と問題を理解したか確かめる | [`/in-your-own-words`](../in-your-own-words/SKILL.md) |
| 送る前に、プロンプトを大きくではなく深く書き直してもらう。一流の専門家が足すもの、その素材と手法、外部からの確認を加える。返すのはプロンプトで、タスクは実行しない | [`/sharpen`](../sharpen/SKILL.md) |
| コードがいまどう動いているか、新しいコードをどこに置くべきかを知る | [`/how`](../how/SKILL.md) |
| コードがなぜこの形なのか、ある数値がどこから来たのかを知る | [`/why`](../why/SKILL.md) |
| 変更やサブシステムを、平易な説明で理解する | [`/teach`](../teach/SKILL.md) |
| あるトピックについて、自分の最近の作業を振り返る | [`/recall`](../recall/SKILL.md) |
| 自分の前回のメッセージ以降にこのチャットで何が変わったかを、平易な言葉と小さな図で把握する。自分の判断を待っている項目はすべて列挙され、返答だけを読んで決められる | [`/ketchup`](../ketchup/SKILL.md) |
| `/ketchup` と同じように、スライドの短い字幕付き動画で把握する。判断待ちの事項は動画の下に文章で示される | [`/ketchup-video`](../ketchup-video/SKILL.md) |
| 小さな差分が、その外側で何を壊しうるかを知る | [`/blast-radius`](../blast-radius/SKILL.md) |
| 関数の境界をまたぐコードを書く前に、型とモジュールの形を決める | [`/architect`](../architect/SKILL.md) |
| 1 つの依頼文に対する複数の試みを得て、最良のものにまとめる | [`/arena`](../arena/SKILL.md) |
| 切り分けた範囲ごとの並列のチェックや、ワーカー同士の競争を、cloud agent で実行する | [`/swarm`](../swarm/SKILL.md) |
| 異なるモデルに差分をレビューさせ、壊そうとさせる | [`/interrogate`](../interrogate/SKILL.md) |
| 手元で安くテストできるとき、テストを先に書いてバグを直す | [`/tdd`](../tdd/SKILL.md) |
| `.ts` や `.tsx` の作業に TypeScript の規則を適用する | [`/typescript-best-practices`](../typescript-best-practices/SKILL.md) |
| レビューの前に、コメントを書いていないレビュアーを使ってコメントを取り除く | [`/no-comments`](../no-comments/SKILL.md) |
| 文章から AI が書いた痕跡を取り除く | [`/unslop`](../unslop/SKILL.md) |
| ドキュメント、RFC、README、PR の説明、コミットメッセージを一定の水準で書く | [`/technical-writing`](../technical-writing/SKILL.md) |
| 直前の返答を、平易な言葉で言い直してもらう | [`/bro`](../bro/SKILL.md) |
| エージェントに、アプリをスクリプトで操作して振る舞いを証明する手段を与える | [`/create-verification-skill`](../create-verification-skill/SKILL.md) |
| 検証スキルとその機能マップを、アプリの現状に合わせ直す | [`/maintain-verification-skill`](../maintain-verification-skill/SKILL.md) |
| 競合状態、不変条件など、テストではめったに届かない性質を形式検証する | [`/create-formal-verification-skill`](../create-formal-verification-skill/SKILL.md) |
| 対象のコードが変わった後に形式検証を合わせ直し、失敗がずれなのかバグなのかを知る | [`/maintain-formal-verification-skill`](../maintain-formal-verification-skill/SKILL.md) |
| ユーザーに見えるものを、TLA+ のモデルで検査した図として PM やデザイナーに見せる。規則を破る各状態への最短の道筋も示す | [`/statechart`](../statechart/SKILL.md) |
| 性能の数値を、報告したり行動の根拠にしたりする前に吟味する | [`/benchmark-checklist`](../benchmark-checklist/SKILL.md) |
| 大きな変更や横断的な変更、または離席後にレビューする変更を進める | [`/figure-it-out`](../figure-it-out/SKILL.md) |
| 実行中に決定ログを残し、後で見直す | [`/show-me-your-work`](../show-me-your-work/SKILL.md) |
| ロールごとのモデルと推論の予算を選ぶ | [`/setup-pstack`](../setup-pstack/SKILL.md) |
| 自分の作業の習慣を、個人用のモードのスキルにする | [`/automate-me`](../automate-me/SKILL.md) |
| 終わったタスクから得た教訓を、スキルの編集に変える | [`/reflect`](../reflect/SKILL.md) |
| このリポジトリでエージェントが同じ間違いを繰り返さないようにする | [`/correct`](../correct/SKILL.md) |
| ボタンを押すと webhook 経由で Grok Bot を起動するページを作る | [`/make-bot-ui`](../make-bot-ui/SKILL.md) |
| pstack の中で迷わないよう案内してもらう | `/poteto-help` |

このディレクトリと同じ階層にあるスキルのディレクトリが表にないときは、その frontmatter を読み、description に従って振り分ける。`principle-*` のディレクトリは、後の原則の節で扱う。

紛らわしい組み合わせ：

- `/how` はコードが何をするかを説明する。`/why` はその理由を説明する。`/teach` はどちらか一方か両方を実行し、結果を平易に説明する。
- `/arena` はすべてのワーカーに同じ依頼文を渡し、最も良い部分をまとめる。`/swarm` は作業を切り分けるか競争させ、1 つの報告を返す。
- `/interrogate` は差分をレビューする。`/blast-radius` は差分の外で壊れるものを探し、変更を安全にしている 1 つの事実を証明する。
- `/recall` は、最近の複数のチャットにまたがって文脈を組み立て直す。`/ketchup` が扱うのは、このチャットのうち、ユーザーの前回のメッセージ以降だけである。特定の 1 つのチャットやブランチを再開するのは、セッションの引き継ぎ（Session pickup）のプレイブックである。
- `/figure-it-out` は、厳密な実行を 1 つ設計する。自律実行（Autonomous run）のプレイブックは、1 つのタスクを終了条件まで進める。

pstack に含まれないもの：

- `/deslop`、`control-cli`、`control-ui` は `cursor-team-kit` プラグインに同梱されている。
- `/loop` と `/create-skill` は Cursor の組み込み機能である。
- pstack に `/orchestrate` スキルはない。スラッシュメニューに `/orchestrate` が出るなら、ほかのプラグインが提供している。

## プレイブックと原則（Playbooks and principles）

プレイブックは `/poteto-mode` の中にある手順の一覧であり、スキルではないので、スラッシュコマンドはない。`/poteto-mode` の中でタスクを説明すればプレイブックが 1 つ選ばれ、次のフレーズはプレイブックを直接指定する。

- 「babysit this pr」（この PR を見守って）や「check on pr 123」（PR 123 の様子を見て）は、見守り（Babysit）を実行する。PR をマージできる状態まで進め、そこで止まる。ユーザーがマージ（merge、land、ship）を頼まない限り、マージはしない。
- 「land the stack」（スタックをマージして）は、リリース（Shipping）を実行する。
- 「take over this branch」（このブランチを引き継いで）は、セッションの引き継ぎ（Session pickup）を実行する。
- 「pause safely」（安全に一時停止して）は、安全な一時停止（Pause safely）を実行する。
- 「full autopilot on this queue」（このキューを完全に自動で進めて）は、マージまでの自動実行（Autopilot-full）を実行する。「autopilot, don't merge」（自動で進めて、ただしマージはしないで）は、マージなしの自動実行（Autopilot-stack）を実行する。
- 「run the eval playbook」（評価のプレイブックを実行して）は、評価（Eval）を実行する。

`/poteto-mode` なしでは、「babysit this pr」のようなフレーズが、代わりに同じ役割の Cursor 自身のスキルを起動することがある。すべてのプレイブックの一覧は、[`poteto-mode`](../poteto-mode/SKILL.md) のプレイブック（Playbooks）の節にある。PR を開く、見守る、マージするまでの流れは、[ガイドの 6 ページ目](../../docs/guide/06-verify-and-ship.md) で扱う。

pstack に計画用のスキルはない。Cursor の Plan Mode と併用できる。複数のフェーズや積み重ねた PR にまたがる作業では、`/poteto-mode` に計画を頼むと [複数フェーズの計画（Multi-phase plan）のプレイブック](../poteto-mode/playbooks/multi-phase-plan.md) が実行され、計画を書くが実装はしない。設計の問いは、プロトタイプ（Prototype）のプレイブックか `/architect` が、まずコードで決着させる。

原則は、規則を 1 つだけ持つスキルで、`/poteto-mode` が読み、返答で引用する。ユーザーが原則を直接呼び出すことはまれである。代わりに、「apply prove it works. show me the real output.」（動くことを証明する（prove it works）を適用して。実際の出力を見せて）のように、名前で方向を指示する。それでも `/principle-<name>` と入力すれば、必要なときに読み込める。一覧は [ガイドの 8 ページ目](../../docs/guide/08-principles.md) にある。

## うまくいかなかった実行を直す（Fix a run that went wrong）

| 症状 | 対処 |
|---|---|
| モードが数ターンで適用されなくなった | Enter で始めたためである。Custom Mode として始めるか、タスクごとに `/poteto-mode` で始める。 |
| 質問が、直前のタスクの次の手順として扱われた | 「new task」と言うか、そのターンにはモードが不要だと伝える。 |
| 新しく選んだモデルが反映されない | `/setup-pstack` のルールは新しいチャットから適用される。新しいチャットを始める。 |
| 実行の費用が想定より高い | セットアップする（Get set up）の節にある、費用の段落を参照する。 |
| スキルが自動で読み込まれなかった | ユーザーの言葉から読み込まれるのは `/setup-pstack` だけである。ほかのスキルは、ユーザーが入力したときか、`/poteto-mode` が実行したときに読み込まれる。`/poteto-mode` もすべてのスキルを実行するわけではない。 |
| 並列のエージェントが互いの変更を上書きした | エージェントごとに worktree を分けるか、cloud agent として実行する。cloud agent はそれぞれ専用のマシンで動く。 |
| 夜間の無人実行で、作業は進んだが何も完了しなかった | `/loop` には、期間ではなく、合否のつく確認が必要である。[ガイドの 7 ページ目](../../docs/guide/07-overnight.md) を参照する。 |
| 返答が、ビルドが通ったことを根拠に成功を主張している | 実際のコマンド、操作の流れ、保存された値、プロファイルを求める。これが動くことを証明する（prove-it-works）原則である。 |

実行が逸れていくときは、[`references/prompting.md`](references/prompting.md) に 1 行で方向を正す指示がある。[ガイドの 10 ページ目](../../docs/guide/10-recipes-and-pitfalls.md) には、ほかの落とし穴と、そのまま使えるレシピがある。

## pstack を自分用にする（Make pstack my own）

- [`/automate-me`](../automate-me/SKILL.md) は、ユーザー自身の履歴から、`/poteto-mode` と併用する個人用のモードのスキルを下書きする。
- セッションの後に [`/reflect`](../reflect/SKILL.md) を使うと、そのセッションの教訓を、ユーザーが承認するスキルの編集に変える。
- `/poteto-mode write a skill for <workflow>` は、スキルの作成（Authoring a skill）のプレイブックを実行する。評価（Eval）のプレイブックは、スキルの変更を盲検でテストする。
- おかしな振る舞いをするスキルは、問題が起きた機能開発の作業の中ではなく、専用の PR で直す。

それぞれの詳細は [ガイドの 9 ページ目](../../docs/guide/09-make-it-yours.md) にある。

## 返答（Reply）

答えから書く。プロンプトの例はコードブロックで 1 つまで示し、合うものがあれば [`references/recipes.md`](references/recipes.md) から手直しして使い、そのあとにそのファイルへのリンクを示す。ユーザーが全体の案内を求めない限り、短くまとめる。
