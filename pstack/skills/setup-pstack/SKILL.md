---
name: setup-pstack
description: "pstack がロールごとにどのモデルを、どの推論の予算で使うかを設定する。使えるモデルを検出し、スキルの既定値を上書きする常に適用されるルール（always-applied rule）を書く。/setup-pstack、「pstack のモデルを設定して」、「pstack の予算」、または pstack のモデルの選び方を変えたいときに使う。"
---

# pstack のセットアップ（Setup pstack）

pstack のロールごとのモデルを設定する、常に適用されるルール（always-applied rule）である `~/.cursor/rules/pstack-models.mdc` を書く。

## 手順（Steps）

### 1. 使えるモデルを検出する（Detect available models）

このセッションで `Task` サブエージェントに渡せるモデルの slug を列挙する。これが信頼できる情報源である。ユーザーが利用できるモデルを列挙するモデルの API か CLI を Cursor も提供しているなら、漏れをなくすためにそちらを優先する。1 つも検出できなければ、使える slug を貼り付けてもらうようユーザーに頼む。使えると確認していない実在の slug は、決して書かない。エイリアスの `inherit-parent` と `auto` は、検出された slug ではないが、常に有効である。

### 2. 現在の状態を読み込む（Load current state）

ロールからモデルへの既定の対応は、下の手順 5 に示すルールの形である。`~/.cursor/rules/pstack-models.mdc` がすでにあれば、それを読み、その `# budget` の行とロールの値を現在の選択として扱う。なければ、その既定値から始める。`hardest tasks`、`architect runners`、`reflect tooling`、`why investigators` のように、ロールが手順 5 にない行は、廃止されたロールのものである。その行は削除する。

以前の既定値は、パネルの席に別の Gemini のモデルを、その後 `gemini-4-neon-high` を置いていた。そのため、どのルールでも、パネルのロール（`arena runners`、`arena cross-judge pool`、`interrogate reviewers`）にある `gemini-4-neon-high` の項目、または `gemini-4-neon-*` 以外の `gemini-4-*` の項目は、古い既定値である。それぞれ `gemini-4-neon-xhigh` に置き換える。それ以外のレベルの Gemini 4 Neon の項目はユーザーの選択なので、そのまま残す。どの項目を置き換えたかをユーザーに伝える。

`# defaults: 3` の行がないルールは、以前のセットアップで書かれたものである。そのセットアップはすべてのロールについて行を書いたので、値の一部はユーザーの選択ではなく古い既定値である。手順 3 の前に、次の値を置き換える。

- `arena cross-judge pool` のリストが、順序を問わず、`claude-fable-*`、`claude-opus-*`、`gemini-*` の項目をちょうど 1 つずつ持つなら、手順 5 にあるそのロールのリストに置き換える。Fable と Opus は同じベンダーなので、Fable の項目がジャッジになれるのは、Opus の項目もジャッジになれる場合だけだった。
- どのロールも `xhigh` より上では動かないので、`# budget: unlimited` の行は `large` にする。
- ルールに `# defaults: 2` の行もなければ、`grok-*` に一致する slug を古い slug とする。`feature, refactoring`、`bug-fix`、`perf-issue`、`hillclimb`、`swarm workers` にある古い slug は、手順 5 にあるそのロールの既定値に置き換える。パネルのロール（`arena runners`、`arena cross-judge pool`、`interrogate reviewers`）で、リストが順序を問わず `claude-opus-*` の項目 1 つと古い slug 1 つだけからなるなら、手順 5 にあるそのロールのリストに置き換える。それ以外のリストでは、古い slug をそれぞれ `claude-opus-5-5-high-fast` にする。

それ以外の値はすべてユーザーの選択なので、そのまま残す。ほかのロールにある古い slug や、`inherit-parent` と `auto` の値もすべて残す。どの値を置き換えたかをユーザーに伝える。

### 3. 予算を決め、対応づけ、確認する（Budget, map, and confirm）

**(a) 予算を尋ねる。** 自由記述より AskQuestion を優先する。次の 3 つの選択肢を、このとおりのラベルで示し、ルールに予算が記録されていれば現在の予算を伝える。ルールがなければ、`large` がスキルの既定値に一致すると伝える。

- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) 予算を適用する。** スキルの既定値から作業用の表を作る。再実行のときは、ファミリー、リスト、エイリアス（`inherit-parent`、`auto`）を変えたロールをそのまま残す。ファミリーは **arena** スキルのフェーズ A（Phase A）が定めるものであり、Fable はそれだけで 1 つのファミリーである。各ロールには目標の effort があり、それは既定値の effort を予算の分だけ動かしたものである。`xhigh` > `high` > `medium` > `low` の段階で、`large` はそのまま、`medium` は 1 段、`small` は 2 段下げる。どのロールも `xhigh` より上では動かず、段階のいちばん下にある目標はそこにとどまる。既定値のほとんどは `xhigh` であり、予算の各ラベルは、それらが行き着く先を示している。ロールにあるすべての実在の slug（パネルの項目も含む）の effort のトークンを、ロールの目標に設定する。effort のトークンは、最後のトークン、または末尾に `fast` があるときはその 1 つ前のトークンである。結果が検出された slug でなければ、同じファミリーの検出された slug のうち、effort が目標以下で最も高いものを使う。それもなければ、そのロールに選択が必要だと印を付ける。`inherit-parent` と `auto` は変わらない。つまり、`medium` は `claude-opus-5-5-xhigh-fast` を `claude-opus-5-5-high-fast` に、`claude-opus-5-5-high-fast` を `claude-opus-5-5-medium-fast` に変え、`small` はそれらを `claude-opus-5-5-medium-fast` と `claude-opus-5-5-low-fast` に変える。Opus 5.5 は常に `-fast` の slug で動かすので、Opus 5.5 の値はすべて `-fast` で終わる。Fable 5.2 と Gemini には fast の slug がないので、決して書かない。Gemini 4 Neon の slug は `low` から `xhigh` まであり、`max` や `fast` の slug はないので、その席は最上位の slug である `gemini-4-neon-xhigh` で動かす。パネルのロールにある Gemini 4 Neon の項目は、独自の目標（`xhigh` を予算の分だけ動かしたもの）を取る。つまり、`large` では `gemini-4-neon-xhigh` のまま、`medium` では `gemini-4-neon-high`、`small` では `gemini-4-neon-medium` を書く。

**(c) ロールを見せて確認する。** すべてのロールをそのモデルとともに見せ、検出された集合にない実在の slug には、選択が必要だと印を付ける。手順 2 で削除した行もすべて挙げる。そのまま受け入れるか、特定のロールを変えるかを尋ね、選択肢として、`xhigh` 以下の検出されたモデルと、`inherit-parent` と `auto` を示す（どちらも、このロールは親のチャットのモデルで動くという意味であり、Auto を使うユーザーはこれで Auto のままになる）。自由記述より AskQuestion を優先する。パネルのロール（arena runners、interrogate reviewers）では値はリストであり、項目ごとに（エイリアスの項目も含めて）サブエージェントが 1 つ動くので、リストの長さが数を決める。`arena cross-judge pool` もリストだが、Arena が動かすジャッジは 1 つである。Arena は親とは別のベンダーの値だけを、リストの順に試すので、好みの順に並べる。Fable とほかの Claude のモデルは同じベンダーである。`inherit-parent` や `auto` の値は、決してジャッジにならない。`swarm workers` は、レースや比較がアームごとに別のモデルを割り当てない限り、すべてのワーカーの既定のモデルである。

### 4. 検証する（Validate）

書く実在の slug はすべて、検出された集合にあり、`xhigh` 以下でなければならない。`inherit-parent` と `auto` は常に通る。選ばれた実在の slug が使えなければ、止まってもう一度尋ねる。

### 5. ルールを書く（Write the rule）

`~/.cursor/rules/pstack-models.mdc` を、`alwaysApply: true`、`# defaults: 3` の行、選んだラベルとそのラベルが示す effort を書いた `# budget` の行、ロールごとに 1 行で書く。ロールのラベルは poteto-mode と同じものを使う。再実行しても結果が変わらない（冪等である）よう、ファイル全体を上書きする。形は次のとおり。

```
---
description: pstack per-role model choices (overrides skill defaults)
alwaysApply: true
---
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit-parent` or `auto` as a value: the role runs on the parent chat model (omit Task `model` and spawn `generalPurpose`). Alias entries in a panel list still count toward its fan-out.
# defaults: 3
# budget: large (xhigh)
feature, refactoring: claude-opus-5-5-xhigh-fast
bug-fix: claude-opus-5-5-xhigh-fast
perf-issue: claude-opus-5-5-xhigh-fast
hillclimb: claude-opus-5-5-xhigh-fast
judgment and prose: claude-opus-5-5-xhigh-fast
arena runners: claude-fable-5-2-high, claude-opus-5-5-high-fast, gemini-4-neon-xhigh
arena cross-judge pool: claude-opus-5-5-high-fast, gemini-4-neon-xhigh
swarm workers: claude-opus-5-5-high-fast
interrogate reviewers: claude-fable-5-2-high, claude-opus-5-5-high-fast, gemini-4-neon-xhigh
```

### 6. 完了を伝える（Confirm）

ルールを書いたこと、それが新しいセッションから適用されることをユーザーに伝える。このスキルを再実行すると、ルールが更新される。

### 7. 任意で検証スキルを提案する（Offer a verification skill (optional)）

プロジェクトに、証明のために実際のアプリを操作する手段（`verify-*` スキル、または既存のハーネス）があるかを確かめる。なければ、一度だけ提案する。「エージェントがユーザーと同じやり方でアプリを操作し、変更が動くことを証明できるように、プロジェクトローカルの検証スキルを作りませんか。/create-verification-skill で生成できます。」承諾されたら、`/create-verification-skill` を呼び出す（pstack がワークスペース、ユーザー、プラグインのどこにインストールされていても解決される）。断られたら、押し付けずに先へ進む。

### 8. 任意で形式検証スキルを提案する（Offer a formal verification skill (optional)）

プロジェクトにすでに形式検証スキル（コードに対してモデル検査器、証明、またはほかの検証器を動かすプロジェクトのスキル）があるかを確かめる。なければ、一度だけ提案する。「テストが届くケースだけでなく、範囲内のすべてのケースでコードの重要な性質が成り立つことをエージェントが証明できるように、形式検証スキルを作りませんか。/create-formal-verification-skill で生成できます。」承諾されたら、`/create-formal-verification-skill` を呼び出す（pstack がワークスペース、ユーザー、プラグインのどこにインストールされていても解決される）。断られたら、押し付けずに先へ進む。
