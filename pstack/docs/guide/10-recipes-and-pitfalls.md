# レシピと落とし穴（Recipes and pitfalls）

そのまま写して使えるプロンプトと、誰もが一度はする間違いをまとめる。パスや完了条件は自分のものに置き換える。レシピはわざとくだけた書き方にしてある。実際にはそのように打ち込まれるものであり、スキルはそれでも意図を問題なく読み取る。

![ロボットたちがレシピの箱を見ながら料理するそばで、彼女ができあがった料理を味見している。カウンターの上には /how、/tdd、/loop と書かれたカードが留めてある。](./images/recipes.jpg)

## 見慣れないサブシステムを理解する（Understand an unfamiliar subsystem）

```text
use /how first to understand how this initialization works. then use /why to figure out why it broke recently.
```

まず仕組み、次に経緯である。どちらのスキルの報告も、どの情報源を調べたかを示すので、答えが何に基づいているかが分かる。

## コードに触れる前に、まとまりのない報告を言い直させる（Restate a noisy report before touching code）

```text
/poteto-mode read this thread. restate the underlying issue in your own words, in plain english. don't change any code yet.
```

読み違いは言い直しの中に現れ、その時点ならメッセージ 1 通で正せる。エージェントが自分の見立てを述べるまでは、自分の見立ては伏せておく。`/in-your-own-words` は、プレイブックを使わずに言い直しだけを求める。

## 送る前にプロンプトを磨く（Sharpen a prompt before you send it）

```text
/sharpen write the onboarding doc for the new billing service.
```

エージェントは、頼まれてはいないが専門家ならやること、専門家が出発点にする素材、その方法とツール、結果が正しいことを外から確かめる手段を挙げる。そして、それらを求めるように書き直したプロンプトを返し、タスク自体は行わない。書き直しを読んで問題がなければ送る。

## 選ぶ前にプロトタイプを作る（Prototype before you pick）

```text
/poteto-mode prototype a few options for the settings layout. put them behind a switcher and send me screenshots of each.
```

説明からではなく、動くものから選べる。レイアウトやタイミングについての疑問には、エージェントが途中で自分で答えを出す。

## 固まった設計を計画にする（Turn a settled design into a plan）

```text
/poteto-mode turn this design into a plan. small verifiable PRs, each with its own proof.
```

頼むのは設計が固まってからにする。成果物は計画そのもので、計画には、それを実行するプレイブックの名前が書かれる。

## 設計についてセカンドオピニオンをもらう（Get a second opinion on a design）

```text
ask /arena for a second opinion on this thread and our approach
```

いまの設計は、いくつかある候補のうちの 1 つになる。統合した結果から、パネルがより良いものを見つけたのか、手元の設計を裏付けたのかが分かる。やり直すのに高くつく設計と比べれば、かかるコストは小さい。

## 独立した部分を並列にチェックする（Check independent slices in parallel）

```text
/swarm check every package under packages/ against its check.sh. one worker per package. one report.
```

各ワーカーが 1 つのパッケージを受け持つ。親はすべての部分の結果を待ち、ワーカーの出力をそのまま並べるのではなく、`PASS`、`ISSUES`、`BLOCKED` のいずれかの報告を 1 つ返す。

## ブランチを疑ってかかってレビューする（Review a branch skeptically）

```text
/interrogate the whole branch, but skeptically. don't change anything yet. no nitpicks unless it's an actual bug or regression in behavior.
```

添えた条件には、それぞれ意味がある。「don't change anything yet」で読むだけの作業にとどまり、細かい指摘についての条件でノイズが前もって除かれるので、検証済みの指摘は読む時間に見合うものになる。

## 失敗するテストを通してバグを直す（Fix a bug through a failing test）

```text
/poteto-mode repro the duplicate write first. if there's a cheap test path, /tdd it. then fix and rerun.
```

「if there's a cheap test path」が大事である。壊れやすいモックを使って無理にテストを書いても、実際のコマンドを実行するより証明できることは少ない。プレイブックは、そう判断して伝えてよいことになっている。

## 報告を再現し、証明付きで直す（Repro and fix a report with proof）

```text
/poteto-mode repro this with /verify-<app>. if it repros on main, fix it and show me a video as proof.
```

「if it repros on main」があれば、バグがすでに直っているときに実行を早めに止められる。動画があれば、差分を読む前に修正を確かめられる。

## テストではめったに届かない性質を証明する（Prove a property tests rarely reach）

```text
/create-formal-verification-skill the retry queue. a message must never be delivered twice.
```

得られるのは、明示した範囲内のすべての場合で検査された 1 つの性質、コードがすでにその性質を破っていればコードの用語で書かれた反例、そして次のエージェントがそのコードを変えるたびに実行するプロジェクトのスキルである。

## 数値を投稿する前に吟味する（Vet a number before you post it）

```text
/benchmark-checklist vet this 40% speedup before it goes in the pr description
```

速くなった、遅くなった、測れるほどの差はない、判定不能、のいずれかが、実行回数、範囲、数値を制限しているものとともに返ってくる。

## 同じ間違いを正し続けるのをやめる（Stop correcting the same mistake）

```text
/correct agents keep adding new config flags without registering them in the schema
```

修正は、アーキテクチャ、型、lint、テストのいずれかとしてリポジトリに入るので、次のエージェントはその間違いをできなくなる。

## 作業を始めずにやり方を尋ねる（Ask how without starting the work）

```text
/poteto-help how do i get poteto-mode to stay on every turn?
```

答えと、送るためのプロンプトと、根拠になるソースへのリンクが返ってくる。そのプロンプトを送るまで、何も実行されない。

## 離れているあいだの実行をごまかしのないものに保つ（Keep a run honest while you're away）

```text
im going to bed, keep going autonomously until every fixture passes. do not stop. keep a decision log i can audit in the morning.
```

取り決めの全体は [夜間の無人実行のページ](./07-overnight.md)にある。タスクと完了条件がすでに会話の中に出ているなら、この短い形で足りる。

## それていく実行を引き戻す（Redirect a drifting run）

舵取りのプロンプトは 1 行で済む。

```text
i said the goal is to repro. i did not ask for a fix yet.
```

```text
apply prove it works. show me the real output, not the build log.
```

```text
/unslop that, no emdashes
```

言葉を増やす必要はほとんどない。必要なのは適切な名前であり、その語彙は [原則のページ](./08-principles.md)にある。

## 返答を平易な言葉で受け取る（Get the reply in plain words）

```text
/bro
```

プロンプトはこれだけである。[`/bro`](../../skills/bro/SKILL.md) は、直前のメッセージを、専門用語を使わず、より短く、人が人に話すように言い直す。返答が技術的には行き届いているのに、何を言っているのか分からないときに使う。

## 動かしたままにしたチャットの状況を把握する（Catch up on a chat you left running）

```text
/ketchup
```

[`/ketchup`](../../skills/ketchup/SKILL.md) は、最後のメッセージ以降に何が変わったかを平易な言葉で伝え、そのあと、読者の判断を待っている事項を 1 つずつ、推す案、証拠、リスクとともに挙げる。報告のあいだ、何も変更しない。

状況の説明を目で見たいときは、代わりに [`/ketchup-video`](../../skills/ketchup-video/SKILL.md) を実行する。何が起きたかを、字幕付きのスライドでできた短い動画で見せ、読者の判断を待っている事項をその下にテキストで並べる。

## 落とし穴（The pitfalls）

- **プロンプトでスキルを列挙する。** 「use /architect then /arena then /interrogate」は、プレイブックがすでに決めている手順の順序を入れ替えてしまう。目標と制約を述べる。スキルの名前を出すのは、既定の動きを上書きしたいときだけにする。
- **あいまいな完了条件。** 「make it better」では、`/loop` が確かめられるものが何もない。成否を判定できるコマンドや成果物を与える。
- **原因についての自分の仮説から話す。** エージェントは、指し示された場所を探す。まず問題を言い直させ、そのあとで自分の見当を伝える。
- **最初の設計で決める。** 1 回の試みでは、モデルが最初に思いついた形に固まってしまう。プロトタイプや `/architect` を求め、証拠に基づいて選ぶ。
- **抽象的な計画を磨き上げる。** コードの裏付けがない計画に敵対的なレビューをかけると、決して起きないリスクがでっち上げられる。未解決の問いをプロトタイプで決着させ、そのうえで作ったものをレビューする。
- **1 つの worktree で並列のエージェントを動かす。** 互いの変更を上書きし、どのエージェントが何を変えたか誰にも分からなくなる。cloud agent として動かすか、「own worktree per attempt」と伝える。
- **ループを信頼する前にループさせる。** 自分の作業を検証できないループは、未検証の作業を速く生み出すだけである。まず検証スキルを動くようにする。
- **吟味していない数値を信じる。** 温まったキャッシュや、飛ばされたコードパスで、偽の高速化が出ることがある。数値をどこかに出す前に `/benchmark-checklist` を実行する。
- **同じ間違いを手で正す。** チャットでの訂正は 1 回の実行にしか役立たない。`/correct` はリポジトリを直すので、後の実行で同じ間違いが繰り返されなくなる。
- **網羅のために `/arena` を使う。** `/arena` は、1 つの設計やコードの依頼文を繰り返し実行し、ベースを 1 つ選んで、ほかの良い部分を接ぎ木する。`/swarm` は、部分や、宣言した競争のアームに分けて実行し、1 つの報告に集約する。
- **レビューコメントをすべて受け入れる。** ボットも人間も、本物の指摘とノイズを 1 つの一覧に並べてくる。`/interrogate` は、再実行で証明された指摘だけを残し、何をなぜ落としたかを挙げる。どちらの判断も読者が覆せる。
- **`auto` をモデルの slug として扱う。** `auto` と `inherit-parent` は、「model フィールドを省き、サブエージェントが親のチャットのモデルを引き継ぐ」という意味である。ロールについては [セットアップ](./01-setup.md)で扱っている。
- **グリーンのビルドを根拠に成功を報告する。** ビルドで分かるのは、コンパイルが通ることだけである。実際のコマンド、操作の流れ、保存された値、プロファイルを求め、返答に証拠があることを期待する。
- **`SKILL.md` を自己流で書く。** [スキルの作成・変更（Authoring or modifying a skill）プレイブック](../../skills/poteto-mode/playbooks/authoring-a-skill.md)を通して、検証とレビューが行われるようにする。

ガイドは以上である。途中を飛ばしてここに来たなら、[セットアップ](./01-setup.md)に戻って、実際のタスクを 1 つ実行する。習慣は、読むことではなく使うことで身につく。

[ガイドの目次](./README.md)に戻る。
