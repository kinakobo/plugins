# pstack

私は [poteto](https://x.com/poteto) である。社長でも CEO でもないが、Meta、Netflix、Cursor で数百万行のコードに携わってきた。React のコアチームにも所属し、React Compiler の開発と保守を手伝っている。

AI は slop なコード（冗長で質の低いコード）を書きすぎる、という感覚が広がっている。私も同感である。slop 職人 20 人のチームのような出し方はしたくない。品質を伴わないスループットは、私の目指すところではない。速く進みたいなら、まず深く取り組む。

**pstack はそれに対する私の答えである。** ここにあるのは、私が Cursor で品質の高いコードを出すために毎日使っているスキルそのものである。これで Cursor は本物のエンジニアリングチームになる。目的はコードの行数（LOC）を最大にすることではなく、むしろその逆である。pstack を使うと、書くコードは減り、その質は上がる。

**pstack があれば、恐れずに並列化できる。** 1 つのエージェントに深く取り組ませ、検証できる良いコードを書くと信頼できるなら、自信を持って本当の意味で並列化できる。`poteto-mode` で複数のエージェントを起動し、どのエージェントも作業に厳密なエンジニアリングの原則を適用すると信頼して任せればよい。

**Cursor なら、すべてのモデルのいいとこ取りができる。** フロンティアモデルには、それぞれ長所と短所がある。pstack はどのモデルでも使える。実際、私のスキルの多くは、各モデル固有の長所を活かすために、複数のモデルを使うワークフローを採っている。

フォークして、改良して、自分のものにしてほしい。PR も歓迎する！

## インストール（install）

```bash
/add-plugin pstack
```

Claude Code では次のようにする。

```bash
/plugin marketplace add cursor/plugins
/plugin install pstack@cursor-plugins
```

Claude Code は、サードパーティのマーケットプレイスでは自動更新をオフにしているので、pstack はインストールしたバージョンのまま変わらない。自動更新をオンにするには、`/plugin` を実行し、**marketplaces** タブを開いて `cursor-plugins` を選び、**enable auto-update** を選ぶ。手動で 1 回だけ更新するには、`claude plugin update pstack@cursor-plugins` を実行する。

Claude Code はプラグインのコマンドとエージェントに接頭辞を付けるので、`/poteto-mode` は `/pstack:poteto-mode` になる。そのほかの Cursor の名前は、[Claude Code のアダプター](./docs/harness/claude-code.md)が対応づけている。Claude Code では、`/pstack:setup-pstack` がモデルの選択を `~/.claude/rules/pstack-models.md` に書き込む。Bugbot や Automations のような Cursor だけの機能を必要とする手順は、1 行の注記を残して飛ばす。

Claude Code は、この README とガイドを含む pstack の生成済みのコピーを、リポジトリのルートにある [`harness/pstack/`](https://github.com/cursor/plugins/tree/main/harness/pstack) からインストールする。pstack を変更したら、リポジトリのルートで `node scripts/generate-pstack-harness-trees.mjs` を実行し、その結果をコミットする。生成されたコピーが古いあいだは、CI が失敗する。

## はじめかた（get started）

手順は 2 つである。

1. [`/setup-pstack`](./skills/setup-pstack/SKILL.md) を実行し、推論の予算を選び、使いたいモデルを選ぶ。
2. 厳密さが求められる作業では、いつでも [`/poteto-mode`](./skills/poteto-mode/SKILL.md) を使う。

初めて使うなら、[pstack のガイド](./docs/guide/README.md)が、セットアップとプロンプトの書き方から検証、夜間の無人実行まで、最初の実際のタスクを順を追って説明している。行き詰まったときや、どのスキルが合うかわからないときは、[`/poteto-help`](./skills/poteto-help/SKILL.md) に聞く。

これだけである。ほかのスキルは状況に応じて使うもので、必要なときはモードのスキルが代わりに使う。初期設定では、コードの委任先（feature、refactoring、bug fix、perf、hillclimb）、文章、判断は Opus 5.5 の xhigh に割り当てられ、xhigh より上で動くロールはない。レビューのパネル以外で Fable 5.2 が動くのは、ユーザーが名前を挙げたときと、同じ依頼文での Opus 5.5 の実行が 2 回失敗したときだけである。既定のパネルは Fable 5.2 / Opus 5.5 / Gemini 4 Neon である。[`/setup-pstack`](./skills/setup-pstack/SKILL.md) で、これらはどれでも変えられる。

## 使い方（usage）

タスクの最初に [`/poteto-mode`](./skills/poteto-mode/SKILL.md) を使う。このスキルは依頼を読み、用意されたプレイブックから 1 つを選び、手順が必要とするときにほかのスキルを実行する。

タスクを入力する前に、`/` メニューから `/poteto-mode` を選ぶ。そうすると、Cursor がスキルをメッセージに添付する。

### とにかく [`/poteto-mode`](./skills/poteto-mode/SKILL.md) を使う（just use `/poteto-mode`）

このスキルが主な近道である。エージェントに厳密なエンジニアリングの作業をさせたいときは、私はいつもこれを使う。このスキルには 22 のプレイブックが付いている。

```
/poteto-mode this pr has a subtle bug where the scroll drifts every 750ms even when idle. repro
first, then fix and verify.
```

```
/poteto-mode i'm going to bed. land the stack even if ci flakes. i want everything merged by
morning.
```

<details>
<summary>22 のプレイブック</summary>

| プレイブック | 用途 |
|---|---|
| [調査（investigation）](./skills/poteto-mode/playbooks/investigation.md) | 読み取りだけの質問。x はどう動くのか、y はなぜこう作られたのか、本当に確かか。 |
| [バグ修正（bug fix）](./skills/poteto-mode/playbooks/bug-fix.md) | 不具合を再現し、根本原因を突き止め、実行時の証拠をもとに直す。 |
| [性能問題（perf）](./skills/poteto-mode/playbooks/perf-issue.md) | 計測で確かめた遅さをトレースし、ベースラインと比べて改善する。 |
| [山登り（hillclimb）](./skills/poteto-mode/playbooks/hillclimb.md) | 1 つの指標を、目標に向けて継続的かつ科学的に改善する。仮説を立てては前後の計測で確かめることを繰り返し、採用した改善ごとに 1 コミットにする。 |
| [実行時の解析（runtime forensics）](./skills/poteto-mode/playbooks/runtime-forensics.md) | 動作中に起きている症状（リーク、アイドル時の CPU の空回り、グリッチ）を、計装をもとに診断する。 |
| [トレースの解析（trace forensics）](./skills/poteto-mode/playbooks/trace-forensics.md) | 取得済みのプロファイリングの成果物（cpuprofile、トレース、spindump、ヒープスナップショット）から診断する。 |
| [機能開発（feature）](./skills/poteto-mode/playbooks/feature.md) | 新しい振る舞い、または変更した振る舞いを、名前を付けたデータの形から組み立てる。 |
| [リファクタリング（refactoring）](./skills/poteto-mode/playbooks/refactoring.md) | 振る舞いを保ったまま、構造や形を変える。 |
| [プロトタイプ（prototype）](./skills/poteto-mode/playbooks/prototype.md) | 設計や振る舞いの判断を安く下すための、または実際に観察して経験的な分岐に決着をつけるための、使い捨ての試作。 |
| [見た目の一致（visual parity）](./skills/poteto-mode/playbooks/visual-parity.md) | 2 つの実装のあいだで、UI をピクセル単位で一致させる。 |
| [スキルの作成（authoring a skill）](./skills/poteto-mode/playbooks/authoring-a-skill.md) | SKILL.md を書く、または編集する。 |
| [評価（eval）](./skills/poteto-mode/playbooks/eval.md) | スキルやプロンプトの変更がエージェントの振る舞いにどう影響するかを、ブラインドで試す。 |
| [見守り（babysit）](./skills/poteto-mode/playbooks/babysit.md) | PR やスタックを、マージできる状態まで進める（コンフリクト、レビューのスレッド、CI）。 |
| [リリース（shipping）](./skills/poteto-mode/playbooks/shipping.md) | CI が通ったスタックを独立に検証し、検証済みの連続した PR を下から順にマージする。既定では GitHub を通し、使えるときは Origin を通す。 |
| [自律実行（autonomous run）](./skills/poteto-mode/playbooks/autonomous-run.md) | 長いタスクを、止まらずに最後までやり遂げる。 |
| [autopilot-full](./skills/poteto-mode/playbooks/autopilot-full.md) | 独立した PR を、PR ごとに 1 つの担当を置いてマージまで進める。コードの準備ができた各 head は swarm が検証し、その後パッチを変えるたびに改めて検査する。 |
| [autopilot-stack](./skills/poteto-mode/playbooks/autopilot-stack.md) | マージしない autopilot-full。オペレーターがレビューしてマージするための PR を作って検証する。PR は、それが土台にする PR の上にだけ積む。 |
| [セッションの引き継ぎ（session pickup）](./skills/poteto-mode/playbooks/session-pickup.md) | 前のエージェントの進行中の作業を再開する、または引き継ぐ。 |
| [安全な一時停止（pause safely）](./skills/poteto-mode/playbooks/pause-safely.md) | あとで再開できるように、進行中の作業をきれいに中断する。 |
| [複数フェーズの計画（multi-phase plan）](./skills/poteto-mode/playbooks/multi-phase-plan.md) | 複数のフェーズや、積み重ねた PR にまたがる作業。 |
| [worktree の片付け（worktree cleanup）](./skills/poteto-mode/playbooks/worktree-cleanup.md) | マージ済みや放棄された worktree と、古い iOS シミュレーターを削除して、ディスクを空ける。安全のための確認を挟む。 |
| [PR を開く（opening a pr）](./skills/poteto-mode/playbooks/opening-a-pr.md) | 順序立てた小さなコミットから、Conventional Commits 形式のタイトルとブリーフィング形式の本文を付けて、ready の PR を開く。ほかのすべてのプレイブックの最後に呼ばれる。 |

</details>



呼び出されると、このスキルは次のように動く。

1. タスクに合う[プレイブック](./skills/poteto-mode/playbooks/)を選び、todo list を開く。todo list の最初の項目は、そのプレイブックの手順をそのまま写したものである。
2. 手順の実行に合わせて、ほかのスキルに振り分ける。
3. slop を取り除いた返答を、利用者とメンテナーに向けた形で書く。

ルールとプレイブックの全体は [`skills/poteto-mode/SKILL.md`](./skills/poteto-mode/SKILL.md) にある。

[`/poteto-mode`](./skills/poteto-mode/SKILL.md) を複数のターンにわたって有効にしておくには、`/` メニューから選び、enter の代わりに option+enter（Mac）か alt+enter（Windows）を押す。そうすると[カスタムモード](https://cursor.com/docs/skills#using-a-skill-as-a-custom-mode)になる。カスタムモードは、Cursor の Agents ウィンドウと CLI で使える。モードを抜けるまで、毎ターン、コンテキストに含まれ続ける。ただの enter では、1 つのメッセージにしか添付されない。

[`/poteto-mode`](./skills/poteto-mode/SKILL.md) は、Cursor の `/loop` コマンドととても相性がよい。厳密さを犠牲にせずに、Cursor を何時間も働かせられる。

## スキル（skills）

これらのスキルのほとんど（`why`、`architect`、`arena`、`swarm`、`interrogate`、`unslop`、`no-comments`、`technical-writing`、`tdd`、そして原則）は、手順が必要とするときに [`/poteto-mode`](./skills/poteto-mode/SKILL.md) が代わりに実行する。下の表は、スキルを直接使いたいときのためのものである。

```
/how do we cancel runs? do we have an n+1 when we look up every run to cancel?
```

```
/interrogate review this pr.
```

<details>
<summary>すべてのスキル</summary>

| スキル | 使う場面 |
|---|---|
| [`/poteto-mode`](./skills/poteto-mode/SKILL.md) | 自明でないタスクすべての、既定の入口。 |
| [`/poteto-help`](./skills/poteto-help/SKILL.md) | pstack を初めて使うとき、またはどのスキル、プレイブック、原則が合うかわからないとき。ユーザーが何をしようとしているかを確かめてその部分に答え、入力するプロンプトを渡す。`/poteto-help` と入力したときだけ動く。 |
| [`/in-your-own-words`](./skills/in-your-own-words/SKILL.md) | 作業を始める前に、ユーザーの目標と問題をエージェントに言い直させたいとき。`/in-your-own-words` と入力したときだけ動く。 |
| [`/sharpen`](./skills/sharpen/SKILL.md) | 送る前に、プロンプトを大きくするのではなく深くしたいとき：一流の専門家ならユーザーが頼まなくてもやること、その専門家が出発点にする素材、その手法と道具、結果が正しいことを外から確かめる方法。書き直したプロンプトを返す。そのプロンプトは、実験で決着がつく選択についてはエージェントにプロトタイプを作らせ、製品や好みに関わる選択はユーザーに委ねる。タスク自体は行わない。`/sharpen` と入力したときだけ動く。 |
| [`/how`](./skills/how/SKILL.md) | サブシステムがどう動くかを、順を追って説明してほしいとき。 |
| [`/why`](./skills/why/SKILL.md) | 何かがなぜこう作られたのかを知りたいとき。コードではなく記録から答える：それを導入したコミットと PR の議論、その後の各変更、そしてその手がかりが乏しいときはほかの情報源。作者が書いたことを引用し、自分の推測にはそうと印を付け、記録からは答えが出ないことを明示する。 |
| [`/recall`](./skills/recall/SKILL.md) | 作業を始める、または再開するときに、あるトピックについての最近の文脈を自分のチャット履歴と共有の記録から組み立て直し、現状を簡潔にまとめた要約として受け取りたいとき。 |
| [`/ketchup`](./skills/ketchup/SKILL.md) | 前回のメッセージ以降にこのチャットで何が変わったかを、平易な言葉と小さな図で知りたいとき。ユーザーの判断を待っている項目はすべて挙げ、返答だけを読んで決められるようにする。そのせいで返答が長くなってもそうする。 |
| [`/ketchup-video`](./skills/ketchup-video/SKILL.md) | `/ketchup` の要約を、スライドでできた字幕付きの短い動画で受け取りたいとき。ユーザーの判断を待っている事項は、動画の下にテキストで付く。 |
| [`/blast-radius`](./skills/blast-radius/SKILL.md) | 小さく見える変更があり、それがほかに何を壊しうるかを知りたいとき。その変更が安全だと言える根拠の事実 1 つを、主張するだけでなく、コードを実行して証明する。 |
| [`/architect`](./skills/architect/SKILL.md) | 関数の境界をまたぐコードを書こうとしていて、呼び出し元での使い方、型、モジュールの形を先に決めておきたいとき。 |
| [`/arena`](./skills/arena/SKILL.md) | 同じことを N 通り並列に試し、それぞれのよい部分を取り出したいとき。 |
| [`/swarm`](./skills/swarm/SKILL.md) | 別々の担当範囲や競争に N 個のワーカーを並列に割り当て、結果を 1 つのレポートにまとめたいとき。 |
| [`/interrogate`](./skills/interrogate/SKILL.md) | 差分があり、別々のモデルにそれを壊そうと試させたいとき。実行して証明された指摘だけを残す。 |
| [`/automate-me`](./skills/automate-me/SKILL.md) | 自分の `-mode` スキルを、実際の自分の働き方をもとに下書きしてほしいとき。 |
| [`/make-bot-ui`](./skills/make-bot-ui/SKILL.md) | ボタンを押すと webhook 経由で Grok Bot を起動するページやダッシュボードがほしいとき。sender key の受け渡しと Tailscale も含む。 |
| [`/setup-pstack`](./skills/setup-pstack/SKILL.md) | pstack がロールごとにどのモデルを使うかを選びたいとき。使えるモデルを検出し、設定のルールを書き出す。 |
| [`/reflect`](./skills/reflect/SKILL.md) | 長いタスクが完了し、そのやり方をスキルの編集として残したいとき。 |
| [`/correct`](./skills/correct/SKILL.md) | エージェントの同じ間違いを何度も直しているとき。履歴から間違いの種類を洗い出し、それぞれを直せる最も高いレベルで直し（アーキテクチャ、次に型・lint・CI、次にテスト、最後にドキュメント）、各ルールとそれを強制する仕組みを対にした表を保つ。 |
| [`/teach`](./skills/teach/SKILL.md) | 変更やサブシステムを、要約してもらうだけでなく、本当に理解したいとき。how と why を実行し、図を 1 枚ずつ積み上げながら、1 つの平易な説明にまとめる。 |
| [`/tdd`](./skills/tdd/SKILL.md) | バグを直していて、ローカルで手軽にテストできる方法があるとき。失敗するテストを先に書き、それから修正する。 |
| [`/benchmark-checklist`](./skills/benchmark-checklist/SKILL.md) | ベンチマークを実行した、または高速化や性能の劣化を計測したとき。数値を報告したり、それをもとに動いたりする前に、その数値を吟味する（律速要因、チューニング、エラー、繰り返しの実行、エンドツーエンドでの意味）。 |
| [`/no-comments`](./skills/no-comments/SKILL.md) | レビューの前にコメントを取り除く。Comment Sicko を起動し、受け入れた指摘を直し、コメントが主張する制約をコードで表す方法を提案する。 |
| [`/typescript-best-practices`](./skills/typescript-best-practices/SKILL.md) | TypeScript を読んだり編集したりしているとき。型システムの規律（type-system-discipline）の原則を、具体的な構文に落とし込む。 |
| [`/figure-it-out`](./skills/figure-it-out/SKILL.md) | 同梱のプレイブックがどれも合わないとき。そのタスクのために、厳密で監査できるプレイブックを設計する。 |
| [`/show-me-your-work`](./skills/show-me-your-work/SKILL.md) | レビューできる決定の記録がほしいとき。決定を、コミットできる TSV に記録する。 |
| [`/create-verification-skill`](./skills/create-verification-skill/SKILL.md) | プロジェクトに、アプリの振る舞いをスクリプトで証明する手段がないとき。機能マップ付きの、プロジェクト専用の検証スキルを生成する。言語やプラットフォームは問わない。 |
| [`/maintain-verification-skill`](./skills/maintain-verification-skill/SKILL.md) | 検証スキルの機能マップが、アプリの実態からずれてきたとき。ソースを読む一斉調査（source wave）と、実際にアプリを動かす確認（live pass）を 1 回行い、証明できた修正を多くとも 1 つの PR にまとめる。 |
| [`/create-formal-verification-skill`](./skills/create-formal-verification-skill/SKILL.md) | 競合、リトライ、ステートマシン、パーサー、unsafe なコードを壊すケースに、テストが届かないとき。合ったツールで性質を 1 つ検証し、変更のたびにそれを検査し直すプロジェクト専用のスキルを生成する。 |
| [`/maintain-formal-verification-skill`](./skills/maintain-formal-verification-skill/SKILL.md) | 形式検証の対象のコードが、前回の検査のあとで変わったとき。モデル、ハーネス、性質をコードに合わせて更新して再実行し、失敗のそれぞれが検証側のずれなのか、コードのバグなのかを報告する。 |
| [`/statechart`](./skills/statechart/SKILL.md) | 振る舞いに責任を持つ人が、仕様を読むのではなく、振る舞いを目で見る必要があるとき。TLA+ のモデルを検査し、ユーザーに見える振る舞いを Mermaid のステートチャートとして描き、ルールを破る状態それぞれに印を付け、各状態に至る最短の経路を示す。 |
| [`/unslop`](./skills/unslop/SKILL.md) | 文章を整えているとき。AI が書いたとわかる特徴を取り除く。 |
| [`/bro`](./skills/bro/SKILL.md) | 直前のメッセージを、専門用語を使わない平易な言葉で言い直してほしいとき。 |
| [`/technical-writing`](./skills/technical-writing/SKILL.md) | ドキュメント、RFC、README、PR の説明、コミットメッセージのための、層を重ねた文書規範（Diátaxis + Google developer style + STE + Global English）。 |

</details>



### 例（examples）

私はたいてい、タスクの最初に [`/poteto-mode`](./skills/poteto-mode/SKILL.md) と入力し、プレイブックへの振り分けを任せる。ほかのスキルは、手順が必要とするときに実行される。いくつかは直接使う。


<details>
<summary>すべての例</summary>

```
bug fix:           /poteto-mode this pr has a subtle bug where the scroll drifts every 750ms even
                   when idle. repro first, then fix and verify.
perf:              /poteto-mode a big list takes a second or two to load even though we virtualize.
                   run a cpu trace and tell me why.
feature:           /poteto-mode build a small feature behind a feature flag. verify it really works.
prototype:         /poteto-mode build two prototypes of the markdown renderer so we can compare.
                   spawn an agent for each.
multi-phase:       /poteto-mode open source these skills as a plugin. nothing internal leaks, work
                   in a temp dir, show me the dependency graph first.
overnight run:     /poteto-mode i'm going to bed. land the stack even if ci flakes. i want
                   everything merged by morning.
babysit:           /poteto-mode check on pr 123. anything outstanding?
visual parity:     /poteto-mode the row spacing is too tall when this flag is on. the second image
                   is correct. repro and fix until it matches.
figure it out:     /poteto-mode i'm stepping away. migrate every caller from the synchronous store
                   to the new async one, keeping behavior identical. i want to trust it was done
                   right when i'm back.
how:               /how do we cancel runs? do we have an n+1 when we look up every run to cancel?
why:               /why is this feature flag not on yet?
architect:         design this instrumentation to be high signal with no false positives. /architect
                   this first.
arena:             /arena take my prompt to the arena verbatim. i want to compare their proposals
                   with yours.
swarm:             /swarm check every package under packages/ against its check.sh. one worker per
                   package. one report.
interrogate:       /interrogate review this pr.
tdd:               /tdd implement
unslop:            can we unslop and tighten the new changes?
reflect:           /reflect that took too long. capture what we learned so the next run doesn't
                   repeat it.
correct:           /correct
show-me-your-work: /show-me-your-work keep a decision trail i can review when i'm back.
automate-me:       /automate-me
help:              /poteto-help which skill should i use to review this branch?
```

</details>

## `poteto-agent` と Comment Sicko のサブエージェント（the `poteto-agent` and Comment Sicko subagents）

[`/poteto-mode`](./skills/poteto-mode/SKILL.md) は、それを受け取ったチャットの中で動く。プレイブック、todo list、すべてのサブエージェントの起動は、そのチャットが受け持つ。

pstack には、実行中に起動する委任先のラッパーとして、[`subagent_type: "poteto-agent"`](./agents/poteto-agent.md) も同梱している。このサブエージェントは、作業を始める前に、`poteto-mode` をインラインの原則の索引も含めて全部読む。`generalPurpose` で代用すると、この読み込みが飛ばされ、作業の仕方がずれていく。プレイブックそのものを動かすために `poteto-agent` を起動しない。

[`/poteto-mode`](./skills/poteto-mode/SKILL.md) と [`subagent_type: "poteto-agent"`](./agents/poteto-agent.md) は、同じラッパーを通る。

pstack には [Comment Sicko](./agents/comment-sicko.md) も同梱している。これは読み取り専用のコメントレビュアーで、`subagent_type: "comment-sicko"` として使える。普段は直接呼ばず、[`/no-comments`](./skills/no-comments/SKILL.md) を通して呼ぶ。

## 原則（principles）

1 つの原則につき 1 つの短いスキルで、全部で 24 ある。`poteto-mode` はそれらの索引をインラインに持ち、タスクの開始時にその索引を読む。個別のファイルがあるのは、ほかのスキルが原則を名前で参照できるようにするためと、索引から各原則の完全なルールを指せるようにするためである。

<details>
<summary>24 の原則すべて</summary>

| 原則 | グループ | ルール |
|---|---|---|
| [怠惰のプロトコル（laziness-protocol）](./skills/principle-laziness-protocol/SKILL.md) | コア | 削除と、問題を解決する最小の変更を優先する。 |
| [基礎から考える（foundational-thinking）](./skills/principle-foundational-thinking/SKILL.md) | コア | ロジックを書く前に適用する：中心となる型とデータ構造を選ぶとき、足場づくりと機能づくりの順序を決めるとき、並行して動く主体が何を共有するかを問うとき。データ構造を正しく決めて、その先のコードが自明になるようにする。 |
| [第一原理から設計し直す（redesign-from-first-principles）](./skills/principle-redesign-from-first-principles/SKILL.md) | コア | 要件を後付けするのではなく、その要件が最初から基礎の前提だったかのように設計し直す。 |
| [前提を疑う（attack-the-premise）](./skills/principle-attack-the-premise/SKILL.md) | コア | 同じ前提に立つ 2 つ以上の修正が、同じ検査（gate）で失敗したときに適用する。その前提に立つ修正をさらに書く前に、前提を書き出して直接確かめる。 |
| [足す前に引く（subtract-before-you-add）](./skills/principle-subtract-before-you-add/SKILL.md) | コア | まず無駄なもの、重複した検証、スタブへの参照を取り除き、簡素になった土台の上に積み上げる。 |
| [読み手の負荷を最小にする（minimize-reader-load）](./skills/principle-minimize-reader-load/SKILL.md) | コア | 問いから答えまでのあいだにある層の数と、読み手が頭の中に抱える隠れた状態を数える。呼び出し元が 1 つしかないラッパーはたたみ、変更できる範囲を狭める。 |
| [成果志向で実行する（outcome-oriented-execution）](./skills/principle-outcome-oriented-execution/SKILL.md) | コア | フェーズの境界がはっきりした、計画的な書き直しや移行のときに適用する。目標のアーキテクチャに向けて収束させる。使い捨ての互換コードで、なめらかな中間状態を保とうとしない。 |
| [体験から始める（experience-first）](./skills/principle-experience-first/SKILL.md) | コア | 実装の都合より、ユーザーの喜びを選ぶ。粗い機能を多く出すより、磨いた機能を少なく出す。 |
| [設計空間を探り尽くす（exhaust-the-design-space）](./skills/principle-exhaust-the-design-space/SKILL.md) | コア | 方針を決める前に、競合するプロトタイプを 2〜3 個作り、並べて比べる。 |
| [てこを作る（build-the-lever）](./skills/principle-build-the-lever/SKILL.md) | コア | 大量の作業に限らず、自明でない作業すべてに適用する：編集、移行、分析、検査。手作業でやる代わりに、その作業を行う道具か、それを証明する道具（codemod、スクリプト、ジェネレーター、サブエージェントが従うスキル）を作る。その道具が、レビュアーが再実行できる成果物になる。 |
| [ドメインをモデル化する（model-the-domain）](./skills/principle-model-the-domain/SKILL.md) | アーキテクチャ | 散らばった条件分岐ではなく、構造としてドメインを表現する。 |
| [境界の規律（boundary-discipline）](./skills/principle-boundary-discipline/SKILL.md) | アーキテクチャ | ガードはシステムの境界（CLI、設定、ネットワーク、外部 API）に集める。内部の型は信頼し、ビジネスロジックは純粋関数に置く。 |
| [型システムの規律（type-system-discipline）](./skills/principle-type-system-discipline/SKILL.md) | アーキテクチャ | 不正な状態を表現できないようにし、意味を持つプリミティブにブランドを付け、外部のデータは境界でパースし、バリアントを網羅し、正となるスキーマから導出する。 |
| [操作を冪等にする（make-operations-idempotent）](./skills/principle-make-operations-idempotent/SKILL.md) | アーキテクチャ | 以前の実行が途中で終わっていても、同じ最終状態に収束させる。 |
| [呼び出し元を移してからレガシー API を消す（migrate-callers-then-delete-legacy-apis）](./skills/principle-migrate-callers-then-delete-legacy-apis/SKILL.md) | アーキテクチャ | 互換レイヤーを残すのではなく、呼び出し元の移行と古い API の削除を、同じ一連の変更で行う。 |
| [共有状態は直列化する前に分離する（separate-before-serializing-shared-state）](./skills/principle-separate-before-serializing-shared-state/SKILL.md) | アーキテクチャ | まず共有をなくす。構造的に直列化するのは、共有された書き手が 1 つであることが本当の不変条件であるときだけにする。 |
| [動くことを証明する（prove-it-works）](./skills/principle-prove-it-works/SKILL.md) | 検証 | タスクを終えたあと、完了を宣言する前に適用する。代わりの指標、自己申告、「コンパイルは通る」ではなく、実物（機能を動かす、実際の値を読む、差分を調べる）で検証する。 |
| [根本原因を直す（fix-root-causes）](./skills/principle-fix-root-causes/SKILL.md) | 検証 | 症状のそれぞれを、システムが最初におかしくなった箇所までたどり、そこで直す。ガードが修正と言えるのは、それが扱う状態が正当なものであるときだけである。 |
| [検証できる単位で順に進める（sequence-verifiable-units）](./skills/principle-sequence-verifiable-units/SKILL.md) | 検証 | 複数の手順からなる作業（一斉の書き換え、移行、似た編集の連続）と、コミットや PR の積み方に適用する。作業を、それぞれ検証できる状態で終わる小さな単位に分け、次に進む前に各単位を確かめ、並びそのものがレビュアーに正しさを示すように届ける順序を決める。 |
| [実装ではなく振る舞いをテストする（test-behavior-not-implementation）](./skills/principle-test-behavior-not-implementation/SKILL.md) | 検証 | テストを書く、変える、残すときに適用する。コードの利用者が観察できることを検証し、その振る舞いが壊れたらテストが失敗し、内部だけが変わったときは通り続けるようにする。 |
| [数値を説明する（explain-the-number）](./skills/principle-explain-the-number/SKILL.md) | 検証 | 計測した数値を信じる、報告する、それをもとに動く前に適用する：高速化、性能の劣化、スループット、レイテンシ、eval の結果。何がその数値を制限しているのかを突き止め、想定している作業とは別のものを計測していた可能性を排除する。 |
| [コンテキストウィンドウを守る（guard-the-context-window）](./skills/principle-guard-the-context-window/SKILL.md) | 委任 | 大量の処理はサブエージェントに回す。メインのスレッドには、生のデータではなく要約を置く。 |
| [人間を待って止まらない（never-block-on-the-human）](./skills/principle-never-block-on-the-human/SKILL.md) | 委任 | 先に進めて結果を示し、人間には後から軌道修正してもらう。確認を求めるのは、取り返しのつかない操作のときだけにする。 |
| [教訓を構造に埋め込む（encode-lessons-in-structure）](./skills/principle-encode-lessons-in-structure/SKILL.md) | メタ | ルールは、文章を増やすのではなく、lint、メタデータのフラグ、実行時のチェック、スクリプトとして組み込む。 |

</details>

## ここに同梱していないもの（not shipped here）

`poteto-mode` が参照しているが、同梱していないものがいくつかある。

- `/deslop` と `deslop` スキルは、`cursor-team-kit` プラグインに含まれている。
- `control-cli`（CLI と TUI 向け）と `control-ui`（ブラウザ、Electron、Web 向け）も、`cursor-team-kit` に含まれている。
- `/create-skill` は Cursor の組み込みのコマンドである。Cursor には、PR を見守る組み込みの `/autopilot` もある。`poteto-mode` の中では、PR の状況についての依頼は、代わりに[見守り（babysit）のプレイブック](./skills/poteto-mode/playbooks/babysit.md)に振り分けられる。

全部そろえたい場合は、pstack と一緒に `cursor-team-kit` もインストールする。

## 計画のスキルがないのはなぜか（why are there no planning skills?）

Cursor にはすでに優れた plan mode があり、pstack とも相性がよい。ただ、私個人は計画というものを信じていない。最良の仕様はコードである。どうしても計画を立てたいなら [`/poteto-mode`](./skills/poteto-mode/SKILL.md) でもできるが、既定の動きではない。

## 自分のものにする（make it yours）

`poteto-mode` は私のスタイルである。読者が求めるものとぴったり同じとは限らない。

[`/automate-me`](./skills/automate-me/SKILL.md) と入力する。このスキルは最近のトランスクリプトを調べ、実際の働き方をもとに `<your-name>-mode` スキルを下書きする。そのスキルは、内部で pstack を経由して動く。pstack を土台に残したまま、`poteto-mode` と並べて自分専用の振り分けのスキルを持てる。

モデルも設定できる。[`/setup-pstack`](./skills/setup-pstack/SKILL.md) と入力する。このスキルは、使えるモデルを検出し、各ロール（コード、判断、レビューのパネル）にモデルを割り当てる小さな常に適用されるルール（always-applied rule）を書き出す。どのスキルもこのルールを読み、ルールがなければ妥当な既定値を使うので、変えたいところだけを上書きすればよい。

既定値が変わっても、変更前に書いたルールは、`/setup-pstack` をもう一度実行するまで古い既定値に固定したままになる。再実行すると、認識できる古い既定値は置き換えられ、モデルが既定値と異なるほかのロールはそのまま残る。認識されない固定を外すには、先にそのロールの行かファイルを削除する。

## オートメーション（automations）

pstack には、休眠状態の [benny のオートメーションパック](https://github.com/cursor/plugins/tree/main/pstack/automations/benny)も同梱している。benny は Slack に届いた問題の報告をトリアージし、確認できたバグを再現して、実際の UI の証拠を付けて直す。そのファイルは、スラッシュコマンドのスキルとしては登録されていない。

セットアップするには、Cursor に [`FOR_AGENTS.md`](https://github.com/cursor/plugins/blob/main/pstack/automations/benny/FOR_AGENTS.md) を読ませる。セットアップでは、パックを対象のリポジトリの `.cursor/automations/benny/` にコピーし、共有のスキルのためにそのリポジトリで pstack を有効にし、ユーザーの設定はコピーしたパックの外に置く。benny は Cursor Automations の上で動くので、Cursor でしか使えない。

## ライセンス（license）

MIT
