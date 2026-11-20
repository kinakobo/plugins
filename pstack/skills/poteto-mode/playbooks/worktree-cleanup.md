### worktree とシミュレーターの片付け（Worktree and simulator cleanup）

1. 現状を記録して監査する。`df -h /` を記録し、`scripts/worktree-audit.sh [-b <pr-base-ref>] [repo-path]` を実行する（principle-build-the-lever）。`-b` を付けないと `MERGED` を `origin/HEAD` に対して判定するので、PR が別のブランチを対象にしているときは `-b origin/<branch>` を渡す。このスクリプトはパスを手で打たず、`git worktree list` から読む。手で打った `myrepo-worktrees/x` では、`.cursor/worktrees/myrepo/x` にある worktree を見落とすからである（principle-encode-lessons-in-structure）。各 worktree を、サイズ、経過時間、マージの状態、コミットしていない作業、PR の状態、最後にそれを触ったチャットで分類し、分類先（bucket）を提案する。トランスクリプトの走査は遅いので、バックグラウンドで実行する。
2. 分類先は助言であって、許可ではない。ピン留めされたチャットと進行中のチャットが、実物の成果物である（principle-prove-it-works）。その一覧をユーザーかサイドバーから得て、候補をすべて突き合わせる。このてこ（スクリプト）は、ユーザーがピン留めしていた worktree を `safe` と判定したことがあるので、ピン留めの一覧を優先する。
3. 消す前に使用状況を確かめる。`verify-recent-chat` の行と、疑わしいものすべてについて、サブエージェントをファンアウトしてトランスクリプトを読ませ、そのチャットがピン留めされているか進行中か、どの worktree を触っているかを報告させる（principle-guard-the-context-window。トランスクリプトは大量である）。ピン留めされたチャットは、バックグラウンドのサブエージェントを通じて、arena や再現のためのツリーを兄弟の worktree に作る。それらは、名前がサイドバーに一度も出ていなくても使用中である。サブエージェントの推論のレベルは、サブエージェント（Subagents）の節に従って仕事ごとに選ぶ。
4. 取り返しのつかない損失の前では止まる。`wip:N` は、追跡されていてコミットされていない N 個の編集である。先に差分を見せて判断を得る。クリーンな worktree は消してもブランチから戻せるが、コミットしていない作業は失われるからである。`scratch:N` は追跡されていない使い捨てのファイルで、消してよいが、ファイル名を挙げる。自律（Autonomy）の節に従い、クリーンでマージ済みで使用中でないものは進める。`wip` と使用中のものでは止まる。
5. 確認した一覧を削除する。パスごとに `git worktree remove --force <path>` を実行する。無視されたビルド成果物のせいでディレクトリが残ったら、それを `rm -rf` し、そのあと `git worktree prune` を実行する。ブランチの ref は残るので、コミットは失われない。`df -h /` で確かめ、一覧を出し直す。
6. シミュレーターと、そのほかの容量を取り戻せるもの。たいていは、シミュレーターが次に大きく容量を空けられる。`xcrun simctl --set testing delete all`（XCTestDevices の複製）、`xcrun simctl delete unavailable`、古いランタイムには `xcrun simctl runtime list` のあとに `runtime delete <id>`。必要ならさらに、Xcode の `DerivedData` と `iOS DeviceSupport`、`~/Library/Application Support/Cursor`（`state.vscdb.backup` と、ワークスペースとして開いたフォルダの名前が付いた `<root>` が膨らむ `snapshots/roots/<root>`）、パッケージのキャッシュ（pnpm、uv、brew、yarn）。消すのは、ユーザーが残すよう言っていないキャッシュだけにする。

これは、ミスを見つけるコードレビューなしにユーザーの状態を消す唯一のプレイブックなので、上のゲートがレビューの役目を果たす。

**返答。** 前後の `df -h /` と空いた容量、削除した worktree、残したそれぞれの理由を 1 行で（どのチャットが使用中か、またはコミットしていない作業）。
