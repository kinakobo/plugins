# 変更を作り、diff をきれいにする（Build the change and clean the diff）

実装のプレイブックには、共通の規律が 1 つある。観察したことを伝え、証拠を求めるのはプレイブックに任せる。このページでは、よくある実装のタスクごとにプロンプトに何を書くかを示し、そのあとで、diff をレビューしやすく保つための片付けの習慣を説明する。

## 実装のプレイブックには、知っていることをプロンプトで伝える（Prompt each build playbook with what you know）

バグのプロンプトでは、症状を述べ、まず再現を求める。

```text
/poteto-mode this command emits two records after a retry. repro first, then fix and verify.
```

機能のプロンプトでは、振る舞いと、変えてはならないものを述べる。

```text
/poteto-mode add a --json flag. text output stays byte-identical. verify both forms.
```

リファクタリングのプロンプトでは、構造を動かす前に振る舞いを固定する。

```text
/poteto-mode move parsing into one module, zero behavior change. record the current output first and prove it's unchanged after.
```

性能のプロンプトでは、印象ではなく測定値を述べる。

```text
/poteto-mode startup takes 1.8s on this fixture. trace it, fix the measured cause, show me before and after.
```

どれも対応するプレイブック、つまり [バグ修正（Bug fix）](../../skills/poteto-mode/playbooks/bug-fix.md)、[機能開発（Feature）](../../skills/poteto-mode/playbooks/feature.md)、[リファクタリング（Refactoring）](../../skills/poteto-mode/playbooks/refactoring.md)、[性能問題（Perf issue）](../../skills/poteto-mode/playbooks/perf-issue.md) に振り分けられ、入力しなかった手順はプレイブックが補う。修正の前に再現する、実装の前にデータの形を決める、構造を変える前に振る舞いを固定する、最適化の前にプロファイルを取る、といった手順である。

1 つの数値を継続的に改善したいときは、[山登り（Hillclimb）のプレイブック](../../skills/poteto-mode/playbooks/hillclimb.md) がある。指標、目標値、最低限の試行回数を渡すと、測定のハーネスを固定したまま、仮説を 1 つずつ試すループを回す。改善したものは残し、それ以外はすべて元に戻す。

性能のプレイブックはどちらも、得た数値に [`/benchmark-checklist`](../../skills/benchmark-checklist/SKILL.md) をかける。性能問題（Perf issue）はベースラインとそれ以降のすべての数値を吟味し、山登り（Hillclimb）はハーネスを固定する前にそのハーネスを吟味する。自分で `/benchmark-checklist` を打つべき場面は、[検証してリリースする（Verify and ship）](./06-verify-and-ship.md#測った数値を-benchmark-checklist-で吟味するvet-a-measured-number-with-benchmark-checklist) で説明している。

修正より先に原因を知りたいこともある。リーク、アイドル時の CPU の空回り、表示の乱れのように、いま起きている症状には、[実行時の解析（Runtime forensics）のプレイブック](../../skills/poteto-mode/playbooks/runtime-forensics.md) が、動いているプロセスに計測を仕込む。すでに取得したプロファイルには、[トレースの解析（Trace forensics）のプレイブック](../../skills/poteto-mode/playbooks/trace-forensics.md) が、その成果物を読み、時間を多く使っているフレームをソースに対応づける。どちらも修正ではなく診断を返す。

```text
/poteto-mode here's a cpuprofile from the slow startup. tell me where the time goes and which source lines own it. no fix yet.
```

## `/tdd` で失敗するテストを先に書く（Write the failing test first with `/tdd`）

手元で手軽にテストできるバグなら、プロンプト全体が 2 語で済む。

```text
/tdd implement
```

文脈がそろっていれば、これで足りる。[`/tdd`](../../skills/tdd/SKILL.md) は、意図した理由で失敗する最小のテストを書き、次に修正を書き、テストを再実行する。テストに大がかりなハーネスの準備や壊れやすいモックが必要になるなら、スキルはそう伝え、代わりに最も近い実行可能な確認を使う。実際のコマンドのほうが強い証拠になる場面で、無理にテストを書かない。

## TypeScript の規則は名前で読み込む（Load the TypeScript rules by name）

[`typescript-best-practices`](../../skills/typescript-best-practices/SKILL.md) は、型システムの原則を具体的な規則にしたものである。判別可能なユニオン、境界での `unknown`、バリアントの網羅、スキーマから導いた型などがある。自動では読み込まれないので、`.ts` や `.tsx` のファイルに触れるタスクでは `/typescript-best-practices` と入力する。

## コミットの前に片付ける（Clean before you commit）

[PR を開く（Opening a PR）のプレイブック](../../skills/poteto-mode/playbooks/opening-a-pr.md) は、コミットのたびに、その前に diff へ `/deslop` をかける。poteto-mode は、PR の説明とコミットの本文に [`/unslop`](../../skills/unslop/SKILL.md) を適用する。`/deslop` は pstack ではなく `cursor-team-kit` プラグインに同梱されている。入っていなければ、同じ結果を普通の言葉で頼む。処理をなぞるだけのコメント、根拠のないガード、使われていない互換用の経路、無関係な編集を取り除くように、と頼めばよい。

文章には、`/unslop` に対象と、追加したい規則があればそれも渡す。

```text
/unslop the readme changes, no emdashes
```

使ううちに、自分なりの略記ができてくる。このスキルは、`unslop that, tighten it` のような短いプロンプトからでも意図を十分に読み取る。

## `/no-comments` でコメントを取り除く（Strip the comments with `/no-comments`）

コメントには専用の見直しが必要であり、それはコメントを書いたエージェント以外が行わなければならない。書き手は、人が自分のコメントをかばうのと同じように、自分のコメントをかばう。だから、レビューの前に、コメントを書いていない者に見せる。

```text
/no-comments the diff
```

[`/no-comments`](../../skills/no-comments/SKILL.md) は [Comment Sicko](../../agents/comment-sicko.md) を起動する。Comment Sicko は読み取り専用のレビュアーで、残してよいものの短い一覧を持っている。ライセンスのヘッダー、公開 API のドキュメントコメント、コードでは表せないことを説明するリンク、作り変えられない外部依存のせいでそうせざるを得ない振る舞いである。それ以外はすべて消す。自分のコードの中にある意外な箇所には、こうした免除はない。そのコメントはリファクタリングを求めるフラグとして返され、`/no-comments` は受け入れたフラグを根本原因から直す。「do not remove」（消さないこと）のように制約を主張するコメントなら、その主張を型、テスト、lint として表現することをスキルが提案する。いずれにしても、コメントは取り除かれる。

役割分担は、はっきり区別しておくとよい。`/deslop` はコードから slop（AI が書きがちな冗長で空疎な記述）を取り除き、`/unslop` は文章から slop を取り除き、`/no-comments` はコメントを、それを書いていないレビュアーに渡す。

**落とし穴：** 片付けは、やってもやらなくてもよい仕上げではない。処理をなぞるだけのコメントや、防御のためだけの不要なコードが残った diff は、レビュアーには未完成に見える。そして、次のバグはその余分なコードに潜む。diff が水増しされていると感じたら、レビューで指摘されてからではなく、コミットの前に `deslop it` と言う。

次は [検証してリリースする（Verify and ship）](./06-verify-and-ship.md)。
