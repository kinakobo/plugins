# ノートを作る（Create a note）

ノートの作成では、ユーザーはブラウザか CLI から題名付きのノートを保存し、書きかけの下書きを取り消し、保存したノートをユーザー向けの別の画面から確かめられる。

## Sub-features

- `create-open` は、ブラウザの各入口から空のエディターを開く。
- `create-save` は、題名と本文を永続化する。
- `create-cancel` は、ブラウザの書きかけの下書きを破棄する。
- `create-cli` は、同じ形のノートをターミナルから作る。

## How to get to it (user POV)

- ブラウザのツールバーで `New note` ボタンを選ぶ。
- ブラウザで、フォーカスが編集可能な欄の外にあるときに `n` を押す。
- ターミナルで `notes create --title <title> --body <body>` を実行する。

## Driving it with control-notes

Preconditions:

- Notes が `http://127.0.0.1:4173` で正常に動いている。
- `Release checklist` という題名のノートがない。
- `control-notes doctor` が、期待どおりの URL と使い捨てのデータディレクトリを報告する。

- **エディターを開く。** `New note` を選ぶ。`control-notes browser click --role button --name "New note"` を実行する。`Note editor` という名前のフォームが現れ、`Title` テキストボックスにフォーカスがある。
- **内容を入力する。** 題名と本文を入力する。`control-notes browser fill --role textbox --name "Title" --value "Release checklist"` と `control-notes browser fill --role textbox --name "Body" --value "Tag and publish"` を実行する。`Save note` ボタンが有効になる。
- **ノートを保存する。** `Save note` を選ぶ。`control-notes browser click --role button --name "Save note"` を実行する。`Note saved` という名前のステータスが現れ、見出しが `Release checklist` になる。
- **永続化を確かめる。** ノートの一覧に戻り、ノートを開き直す。`control-notes browser click --role link --name "All notes"` と `control-notes browser click --role link --name "Release checklist"` を実行する。エディターに、保存した 2 つの値が表示される。
- **下書きを取り消す。** 新しいノートを開き、`Discard me` と入力して `Cancel` を選ぶ。`control-notes browser click --role button --name "New note"`、`control-notes browser fill --role textbox --name "Title" --value "Discard me"`、`control-notes browser click --role button --name "Cancel"` を実行する。ノートの一覧に戻り、`Discard me` のリンクはない。
- **CLI の入口。** 2 つ目のノートを作る。`control-notes cli -- notes create --title "CLI note" --body "Created from terminal" --format json` を実行する。終了コードが `0` で、stdout に新しいノートの ID と題名が含まれる。
- **証明。** `All notes` から、保存した 2 つのノートを開き直す。`control-notes browser snapshot --aria --path artifacts/create-note/list.aria.txt` と `control-notes browser screenshot --path artifacts/create-note/list.png` を実行する。成果物に `Release checklist` と `CLI note` が含まれる。

## Gotchas

- テキストボックスにフォーカスがあるときに `n` を押すと、新しいエディターは開かず、その文字が入力される。
- 題名は、保存時に前後の空白が取り除かれる。下書きの入力値ではなく、表示された題名を検証する。
- 保存のステータスだけでは、証明として不十分である。一覧からノートを開き直す。
- フィクスチャの後片付けで `Release checklist` と `CLI note` を削除するが、それらの証明の成果物は残す。
