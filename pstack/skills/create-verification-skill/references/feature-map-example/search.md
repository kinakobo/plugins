# ノートを検索する（Search notes）

検索では、ユーザーは題名か本文の文字列でノートを探し、一致したノートの中身を確かめ、一致がないことと検索が使えないこととを区別できる。

## Sub-features

- `search-open` は、ブラウザの対応している各入口から検索を開く。
- `search-match` は、ノートのデータを変えずに、題名と本文の一致を返す。
- `search-open-result` は、結果をノートのエディターで開く。
- `search-empty` は、一致のないクエリに対して、検索が完了したうえでの空の状態を表示する。
- `search-clear` は、クエリを消して、最近のノートの表示に戻す。
- `search-cli` は、同じ一致したノートをターミナルから返す。

## How to get to it (user POV)

- ブラウザのツールバーで `Search` ボタンを選ぶ。
- ブラウザで、フォーカスが編集可能な欄の外にあるときに `/` を押す。
- ターミナルで `notes search <query>` を実行する。

## Driving it with control-notes

Preconditions:

- Notes が `http://127.0.0.1:4173` で正常に動いている。
- 使い捨てのデータディレクトリに、本文が `Draft budget` の `Quarterly plan` がある。
- `control-notes doctor` が、期待どおりの URL とデータディレクトリを報告する。

- **ツールバーの入口。** `Search` ボタンを選ぶ。`control-notes browser click --role button --name "Search"` を実行する。`Search notes` という名前のダイアログが現れ、その検索ボックスにフォーカスがある。
- **キーボードの入口。** ダイアログを閉じ、ページにフォーカスを移して `/` を押す。`control-notes browser press --key "/"` を実行する。同じダイアログが現れ、ページにスラッシュは入力されない。
- **題名の一致。** `quarterly` と入力する。`control-notes browser fill --role searchbox --name "Search notes" --value "quarterly"` を実行する。`Search results` の一覧に `Quarterly plan` があり、`Grocery list` はない。
- **本文の一致。** クエリを `budget` に置き換える。`control-notes browser fill --role searchbox --name "Search notes" --value "budget"` を実行する。結果の `Quarterly plan` が、本文の一致箇所の抜粋とともに表示されたままになる。
- **結果を開く。** `Quarterly plan` を選ぶ。`control-notes browser click --role link --name "Quarterly plan"` を実行する。ダイアログが閉じ、エディターの見出しが `Quarterly plan` になる。
- **空の状態。** 検索を開き直し、`volcano` と入力する。`control-notes browser fill --role searchbox --name "Search notes" --value "volcano"` を実行する。検索が終わったあとに、`No matching notes` という名前のステータスが現れる。
- **クエリをクリアする。** `Clear search` を選ぶ。`control-notes browser click --role button --name "Clear search"` を実行する。検索ボックスが空になり、結果の一覧の代わりに `Recent notes` の領域が表示される。
- **CLI での一致。** ターミナルから検索する。`control-notes cli -- notes search "quarterly" --format json` を実行する。終了コードが `0` で、stdout に、題名が `Quarterly plan` のオブジェクトが 1 つ含まれる。
- **CLI での不一致。** 存在しない値を検索する。`control-notes cli -- notes search "volcano" --format json` を実行する。終了コードが `0` で、stdout は `[]` である。
- **証明。** 結果が表示された状態を記録する。`control-notes browser snapshot --aria --path artifacts/search/results.aria.txt` と `control-notes browser screenshot --path artifacts/search/results.png` を実行する。どちらの成果物からも、Notes、クエリ、`Quarterly plan` がわかる。

## Gotchas

- エディターか検索ボックスにフォーカスがあるときに `/` を押すと、検索は開かず、文字が入力される。
- 結果は、短いデバウンスのあとに更新される。決まった時間の sleep ではなく、結果の一覧か空のステータスが現れるのを待つ。
- ユーザーが `Include archived` を有効にしない限り、アーカイブ済みのノートは除かれる。
- CLI の出力は、既定では人間向けの形式である。安定した検証には `--format json` を使う。
- 結果を開くと、ブラウザの状態が変わる。別のクエリを証明する前に、検索を開き直す。
