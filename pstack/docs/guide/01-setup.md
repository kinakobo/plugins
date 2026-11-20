# pstack をセットアップする（Set up pstack）

このページでは、プラグインをインストールし、pstack が使うモデルを選び、最初のタスクを実行する。セットアップは、コマンド 1 つと短いやり取りで終わる。

## プラグインをインストールする（Install the plugin）

Cursor のチャットで次を実行する。

```text
/add-plugin pstack
```

プラグインがインストールされたことを Cursor が知らせる。

Claude Code では、`/plugin marketplace add cursor/plugins` を実行してから `/plugin install pstack@cursor-plugins` を実行する。Claude Code はプラグインのコマンドに接頭辞を付けるので、このガイドのコマンドは Claude Code ではどれも `pstack:` を前に付けて打つ。たとえば `/pstack:setup-pstack` であり、これはモデルのルールを `~/.claude/rules/pstack-models.md` に書き出す。そのほかの違いは [Claude Code 用のアダプター](../harness/claude-code.md) で説明している。

## モデルを選ぶ（Pick your models）

次を実行する。

```text
/setup-pstack
```

[`/setup-pstack`](../../skills/setup-pstack/SKILL.md) は、使えるモデルを検出し、推論の予算を尋ね、ロール（コードの委任先、判断、レビューのパネル）を 1 つずつ示して、どうしたいかを尋ねる。質問に答えると、`/setup-pstack` は `~/.cursor/rules/pstack-models.mdc` を書き出す。これは、pstack のどのスキルも読む小さなルールである。

既定値は `large` の予算と同じである。コード、文章、判断は Opus 5.5 の `xhigh` の推論で動き、レビューのパネルと swarm workers は `high` で動く。`xhigh` より上で動くロールはない。レビューのパネルを除けば、Fable 5.2 が動くのは、タスクで Fable 5.2 を指定したときか、同じ依頼文で Opus 5.5 を 2 回動かしてどちらもチェックに落ちたあとだけである。`medium` は各ロールを 1 段、`small` は 2 段下げ、どちらも消費するトークンが少ない。

上書きするのは、こだわりのあるものだけでよい。ルールに行のないロールは、スキルの既定値のままになる。既定値に戻すには、そのロールの行を消す。`/setup-pstack` を再実行しても、既定値と違うモデルを指定したロールはそのまま残る。既定値が変わっても、変更より前に書き出したルールは、`/setup-pstack` をもう一度実行するまで古い既定値を固定し続ける。その再実行では、古い既定値だと判別できたものを置き換え、どれを置き換えたかを知らせ、ほかの選択は残す。判別できない固定を外すには、先にそのロールの行かファイルを消す。

Auto を使うとどうなるのか、と思うかもしれない。ロールを `inherit-parent` か `auto` にすると、pstack はサブエージェントの `model` フィールドを省くので、サブエージェントは親のチャットのモデルを引き継ぐ。2 つの値の意味は同じで、どちらもモデルの slug ではない。パネルのロールでは値がリストになり、要素ごとにサブエージェントが 1 つ動くので、リストの長さがパネルの大きさになる。セットアップでは `swarm workers` も設定する。これは `/swarm` のすべてのワーカーが使う既定のモデルで、レースでアームごとにモデルを指定したときだけはそちらが使われる。

## 検証スキルの提案を受けるか断るか（Accept the verification offers, or don't）

セットアップの最後に、`/setup-pstack` は、プロジェクトでアプリの振る舞いを証明する手段として、`verify-*` スキルか既存のハーネスを探す。どちらも見つからなければ、[`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md) で作ることを一度だけ提案する。

承諾すると、`.cursor/skills/verify-<app>/` を書き出す。これはプロジェクトに置くスキルで、ユーザーと同じやり方でアプリを操作する方法をエージェントに教える。引き渡す前に、そのスキルが動くことを一度証明する。断れば、セットアップは次に進む。`/create-verification-skill` はいつでも自分で実行できる。詳しくは [検証してリリースする（Verify and ship）](./06-verify-and-ship.md#プロジェクトの検証スキルを作るcreate-a-project-verification-skill) で説明している。

pstack を使い始めたばかりなら、承諾する。自分の作業を確かめられるエージェントは、確認が通るまで作業を続ける。確かめられないエージェントは、結果をすべてユーザーに返し、手で確かめさせる。このガイドで紹介するもののうち、得るものがいちばん大きいのは検証スキルである。

続いてセットアップは、プロジェクトにまだなければ、[`/create-formal-verification-skill`](../../skills/create-formal-verification-skill/SKILL.md) で形式検証スキルを作ることを一度だけ提案する。テストが確かめるのは、誰かが書こうと思いついたケースである。形式検証スキルは、「ジョブが同時に 2 回実行されることは決してない」のような、コードの重要な性質 1 つを、明示した範囲内のすべてのケースで確かめる。並行処理、リトライ、状態機械、パーサーのように、バグの代償が大きく、壊れるケースにテストがめったに当たらないコードがあるなら、承諾する。そうでなければ断り、状況が変わったら後で実行する。[検証してリリースする（Verify and ship）](./06-verify-and-ship.md#形式検証スキルで性質を証明するprove-a-property-with-a-formal-verification-skill) で説明している。

セットアップが終わったら、新しいチャットを始める。モデルのルールは新しいセッションから適用される。

## コストを抑える（Keep the cost in check）

pstack は、サブエージェントとレビューのパネルで余分にトークンを使う。それが厳密さの代価である。消費を減らすには、次のようにする。

- `/setup-pstack` を再実行し、小さい推論の予算か安いモデルを選ぶ。メインのチャットには強いモデルを使い、コードのロールには安くて速いモデルを使う、という分け方がよい。
- ロールを `auto` か `inherit-parent` にして、チャット自身のモデルで動かす。
- パネルのリストを短くする。要素 1 つにつき、サブエージェントが 1 つ動く。
- `/poteto-mode` は、厳密さが必要な作業のために取っておく。小さくて自明な編集には要らない。

## 最初のタスクを実行する（Run your first task）

実際の作業のうち小さなものを選び、同僚に説明するときのように書く。

```text
/poteto-mode add a --json flag to this command. text output stays byte-identical. verify both.
```

todo list を確認する。最初の項目は、選ばれたプレイブックの手順を写したものである。このプロンプトでは、機能開発（Feature）のプレイブックが選ばれる。

ここからは、普通に続けて指示を書けばよい。チャット全体で `/poteto-mode` を有効にしておくには、`/` のメニューから Enter ではなく Option+Enter（Mac）か Alt+Enter（Windows）で選ぶ。こうすると [Custom Mode](https://cursor.com/docs/skills#using-a-skill-as-a-custom-mode) になり、抜けるまで毎ターンのコンテキストに入り続ける。Custom Mode は Agents Window と CLI で使える。普通の Enter では、スキルは 1 つのメッセージに添付されるだけで、チャットが進むにつれて影響が弱まっていく。

次は [`/poteto-mode` で作業を振り分ける（Route work through `/poteto-mode`）](./02-poteto-mode.md)。
