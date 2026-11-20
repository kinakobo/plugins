# Bugbot のトリアージ（Bugbot triage）

見守り（Babysit）のプレイブック（`../playbooks/babysit.md`）が Bugbot やレビューの自動化のコメントを扱うときに、この参考資料を使う。目的は、Bugbot を既定で無視することではない。すべてのコメントを、必ずコードを変えなければならないものとして扱うのをやめることである。

## 判断の評価基準（Decision rubric）

行動する前に、Bugbot のスレッドを 1 つずつ分類する。

- `fix`: コメントが、実際にありそうな問題（正しさ、セキュリティ、プライバシー、データの消失、認証、課金、移行、冪等性、競合状態、リリースされる振る舞いのいずれか）を指摘している。該当するコードを持つ PR のうち、スタックで最も下の PR で直し、コミットの SHA を添えて返答し、スレッドを解決する。
- `dismiss`: コメントが、文書化された低リスクのノイズのパターンに一致し、現在のコードや文脈から、その懸念にはコードの変更が要らないことを証明できる。短い理由を添えて返答し、スレッドを解決する。
- `ask`: コメントが、新しい種類のもの、重大度の高いもの、セキュリティ・プライバシー・データに関わるもの、または曖昧なものである。推測せず、ユーザーに尋ねる。

迷ったら尋ねる。ノイズの多いコード品質のコメントを飛ばしても損失は小さいが、本物のデータやセキュリティのバグを飛ばせばそうはいかない。

## 学んだパターンの書式（Learned pattern format）

今後のパターンは、この形で追加する。

```markdown
### <short pattern name>

- Confidence: candidate | recurring | strong
- Skip when: <conditions that must be true>
- Do not skip when: <risk boundaries>
- Example signal: <phrases or code context that identify the pattern>
- Source: <PR/comment URL or short historical note>
```

例が 1〜2 件なら `candidate` を使う。実際に却下した例が複数あれば `recurring` を使う。`strong` は、パターンの範囲が狭く、繰り返し検証され、リスクが低いときだけ使う。

## 繰り返し現れる、飛ばしてよい候補（Recurring skip candidates）

### 意図した UI やデザインシステムの見た目の変更（Intentional UI or design-system visual changes）

- Confidence: candidate
- Skip when: PR の説明、スクリーンショット、デザインレビュー、近くのコードのいずれかが見た目の変更を明示しており、Bugbot のコメントは、共有の見た目の既定値が変わったことを言い直しているだけである。
- Do not skip when: コメントが、アクセシビリティ、フォーカスの見え方、キーボード操作、色のコントラスト、または PR が意図して変えたのではないコンポーネントの API の契約を指摘している。
- Example signal: フォーカスのアウトライン、ボタンの大きさ、余白、共有コンポーネントの見た目の既定値についてのコメントで、オーナーが「intentional」や「intended」（意図どおり）と返答しているもの。

### Bugbot から見えない、スタックの上の PR やスタック内での使用（Upstack or stack-local usage Bugbot cannot see）

- Confidence: candidate
- Skip when: Bugbot が export、コンポーネント、ヘルパー、ファイルを未使用だと指摘しているが、使っているフォージの PR の一覧と差分、スタックの上の PR の差分、または PR の文脈から、スタックのあとの PR で使われていることがわかる。
- Do not skip when: 現在の PR がスタックの一部ではない、そのシンボルが公開 API である、またはスタックの上で使われているという話を確かめられない。
- Example signal: 「Exported component is never used」に、「used upstack」のような人間の返答が付いているもの。

### 並行実装中の一時的な重複（Temporary duplication during parallel implementation）

- Confidence: candidate
- Skip when: 削除、置き換え、または検証の途中にある古い経路と並べて新しい経路を保つために、PR が意図して少量のコードを重複させている。
- Do not skip when: 重複したコードがセキュリティ、課金、データアクセス、API の振る舞いを変える、または長く使う共有の抽象を作れば明らかにリスクが下がる。
- Example signal: 「Significant duplication」や「duplicated validation logic」というコメントに対し、オーナーが、古い経路は削除される、または重複したロジックは意図してローカルにしていると説明しているもの。

### 既存のフレームワークやコンポーネントの不変条件が警告の内容を保証している（Existing framework or component invariant covers the warning）

- Confidence: candidate
- Skip when: その懸念が、現在の差分か近くのコードで確認できる共有コンポーネント、フレームワークの契約、型の不変条件、または信頼できる唯一の情報源（single source of truth）によって、すでに保証されている。
- Do not skip when: 不変条件が仮定されているだけで強制されていない、タイミングに依存している、または値が食い違いうる非同期や状態の境界をまたいでいる。
- Example signal: 共有のポップオーバーがビューポートの範囲を強制しているのに、内側のポップオーバーに max-height がないと指摘するコメント。または、ローカルで確認した値と渡した値が同じ情報源から来ているのに、nullable な値を指摘するコメント。

### オーナーが宣言した後続の作業や先送りした整理（Owner-declared follow-up or deferred cleanup）

- Confidence: candidate
- Skip when: PR のオーナーがその問題を既知の後続の作業だと明言しており、現在の PR で振る舞いが悪化せず、コメントがリスクの高い領域についてのものではない。
- Do not skip when: エージェントがオーナーの意見なしに動いている、問題がプロダクトの振る舞いに関わる中程度以上の重大度のものである、または先送りすると新しいリグレッションをマージすることになる。
- Example signal: 「I'll worry about that later」や「we'll delete this eventually」（あとで考える、いずれ消す）。

### 自ら取り下げた、または誤検知と明示したルールのコメント（Self-withdrawn or explicit false-positive rule comments）

- Confidence: recurring
- Skip when: コメントの本文か、あとの Bugbot の返答が、その指摘は取り下げた、規則に適合している、または誤検知だと明示しており、エージェントが関係する規則を手元で確かめられる。
- Do not skip when: 証拠が、リスクの高い問題に人間が説明なしに「false positive」（誤検知）と言っていることだけである。
- Example signal: ファイル名の規則についてのコメントで、本文がそのファイルはすでに適合していると述べているもの。

## 既定で尋ねるもの（Ask by default）

以前の PR で似たものを却下していても、次の分類は自動で飛ばさない。

- セキュリティ、プライバシー、認証、課金、データの保持、学習データ、権限の境界に関する指摘。
- 重大度の高い指摘。
- 移行、スキーマ、冪等性、並行性、システムをまたぐ振る舞いに関する指摘。
- 提案された修正が小さく、プロダクトの意図を変えずに明らかにリスクを下げるコメント。

過去のデータでは、人間がセキュリティやデータの流れに関するコメントを却下することがあった。それらはオーナーの個別の判断として扱い、チーム全体で飛ばしてよい規則とはみなさない。

## 最近の見守りから得た学びの候補（Candidate learnings from recent babysits）

見守りの途中かあとで、チームに役立ちそうだがまだ固まっていない学びの候補が出てきたら、ここに追記する。いくつかの PR でパターンが確かめられたら、繰り返し現れる候補は上の節に格上げするのが望ましい。

### ブラウザのネイティブな振る舞いを手作業で再実装したもの（Manual reimplementations of native browser behavior）

- Confidence: candidate
- Skip when: 実際上は一度もない。差分がブラウザのネイティブな振る舞いを手作業の同等物に置き換えている場合（ネイティブの sticky → JS で配置した複製、ネイティブのスクロール対象の決定 → 転送した wheel/touch イベント、描画順による遮蔽 → mask/clip-path）、そのコードに対する Bugbot のロジックのバグの指摘は、一貫して正当だった。
- Do not skip when: 指摘が、そうしたコードでのイベント転送の漏れ（wheel の deltaMode、タッチでのパン、端でのスクロールチェイン、タップの許容範囲）、mask/clip とヒットテストの食い違い、observer と React の状態のタイミングの競合に関するものである。既定では直す。
- Example signal: 「masks do not affect hit-testing」「overlay blocks wheel scroll」「ignores deltaMode」「runs in the IntersectionObserver callback before React applies state」。
- Source: sticky による遮蔽を扱った 1 つの PR。Bugbot が 6 回実行され、約 18 件の指摘があり、すべて却下せずに直した。

### 契約テストのずれの主張は安く確かめられるので、先にテストを実行する（Contract-test drift claims are cheaply verifiable — run the test first）

- Confidence: candidate
- Skip when: 確認そのものは決して飛ばさない。コマンド 1 つで済む。PR が、プロトコルや
  ドキュメントの文章を固定する契約テスト（SKILL.md に対する正規表現、ドキュメントの文言の
  スナップショット）を含んでおり、Bugbot が「the test no longer matches the doc」（テストが
  ドキュメントと合わなくなった）と主張する（またはその逆）なら、分類する前に PR の先端で
  そのテストを実行する。失敗すれば主張が実際に確かめられ、成功すれば却下の返答に添える
  具体的な反証になる。
- Do not skip when: 該当なし。これは確認の近道であって、却下のパターンではない。何度も
  実行されたら却下に傾けるという経験則は、ここでは誤作動する。文章を固定するテストは、
  前の修正のラウンドが文章を編集するからこそずれるのである。
- Example signal: 前の修正のコミットで、固定された一節の文言を変えた PR に対する
  「Contract test omits the pre-fix wait」。先端でテストを実行すると、指摘された
  アサーションでちょうど失敗した。
- Source: 文章を固定するテストを含む 1 つの PR で、Bugbot が 8 回実行された。それまでの
  回の指摘はすべて修正して解決していたが、7 回目の主張は本物だった。

### 同じ PR のあとのコミットですでに直っている、古いセキュリティレビューの指摘（Stale security-review finding already fixed later in the same PR）

- Confidence: candidate
- Skip when: エージェントによるセキュリティレビュー（または類似のもの）が、認可やバリデーションの呼び出しがないと主張しているが、現在の PR の先端には、まさにそのゲートが（テスト付きで）明らかに含まれている。たいていは、レビューの実行後の強化のコミットで追加されたものである。
- Do not skip when: 挙げられたヘルパーが問題のプリンシパルに対しては何もしない、チェックが守るべき副作用のあとに実行される、または指摘されたプリンシパルに対するテストがない。
- Example signal: 先端ではまさにそのガードが副作用の前にすでに呼ばれているのに、HIGH の「missing authorization check」の指摘があるもの。
- Source: 強化のコミットがレビューの実行よりあとだった、webhook のエンドポイントの PR 1 つ。

### 意図して狭くしたエラー条件を広げると、本当のエラーが隠れる（Widening a deliberately narrow error condition would mask the real error）

- Confidence: candidate
- Skip when: 指摘が、狭いエラー条件（特定の `errno`、エラーコード、ステータスの分類）を
  何でも受ける条件に広げるよう求めており、その狭さが実際の区別を表している。典型は、
  `ENOENT` のときだけ働く依存のフォールバックである。「バイナリがインストールされていない」と
  「コマンドが実行されて失敗した」は別の状況である。0 以外の終了コードならどれでも再試行すると、
  正当な失敗（見つからない、認証の期限切れ、ネットワーク）をフォールバックで実行し直し、
  フォールバックのエラーを報告して、本当のエラーを隠してしまう。
- Do not skip when: 狭い条件が、同じ分類に属するケース（`EACCES` のような、別の「バイナリが
  使えない」errno や、別の通信レベルの失敗）を取りこぼしている、扱われない経路でデータが
  失われるか中途半端な状態が残る、または再試行が冪等であり、しかも元のエラーも
  報告される。
- Example signal: 「only retries when X fails with ENOENT … never tries the fallback even
  when a working Y exists」という指摘が、失敗した操作ではなく、存在しない依存のために
  フォールバックを持つコードを指しているもの。
- Source: CLI の名前を変えた 1 つの PR。そのフォールバックは、失敗したコマンドではなく、
  存在しないバイナリのためのものだった。
