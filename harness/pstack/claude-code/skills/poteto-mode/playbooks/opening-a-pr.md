### PR を開く（Opening a PR）

ほかのすべてのプレイブックの最後に実行する。

trunk から分岐した git の worktree で作業し、関係のない作業を入れない。リベースして、小さく、それぞれマージできるコミットにまとめる。コミットの前に、`cursor-team-kit` の `/deslop` を差分に対して実行する。

タイトルは `type(scope): subject` とし、変更の中心になる実在のシンボルがあれば、その名前を入れる。本文は 1 分で読める 40 行未満の説明にし、次の `##` 見出しを順に置き、空の見出しは省く。`## Why`、`## What changed`、`## Scope`、`## Tradeoffs`（書かなければレビュアーが尋ねるであろう、採らなかった代替案だけ）、`## Blast Radius`、`## Verification`。

後続の PR は、ベースブランチを指定して積み重ねる。PR は ready の状態で、実行環境の PR ツールを `draft: false` で使って開く。PR ツールがなければ、`--draft` を付けずに `gh pr create` を使うか、Origin の CLI がリポジトリを解決できるなら `origin pr create --status open` を使う。PR を開いても、見守り（babysit）は始まらない。
