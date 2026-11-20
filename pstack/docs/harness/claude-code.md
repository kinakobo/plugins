# Claude Code での pstack（pstack in Claude Code）

pstack のスキルは Cursor の名前を使う。Claude Code では、このファイルをセッションごとに 1 回読み、スキルに出てくる Cursor の名前を下の表で読み替える。`<pstack>` は `skills/` と `docs/` を含むフォルダを指す。Cursor のフィールドに対応する行がなければ、そのフィールドは渡さず、渡さなかったことをユーザーに伝える。ツールのスキーマが受け付けないフィールドや値は決して渡さない。

## ツール（Tools）

| Cursor | Claude Code |
| --- | --- |
| `Task` | `Agent` |
| `AskQuestion` | `AskUserQuestion`。`-p` モードでは存在しないので、普通の文章で質問する。 |
| todolist を開く、または手順を todolist ツールに写す | セッションに `TodoWrite` があれば使う。なければ一覧を返答の中に書き、フェーズごとに更新する。 |
| 別の pstack スキルを実行する（Run another pstack skill。`/name` または「**name** スキル」） | `<pstack>/skills/<name>/SKILL.md` を `Read` で読み、それに従う。`Skill` ツールはほとんどの pstack スキルを拒否する。ユーザーに入力してもらうときは `/pstack:<name>` と書く。 |
| `Grep`、`Glob` | セッションに `Grep` や `Glob` がなければ、`Bash` で `rg` を使う。 |

## サブエージェントのフィールド（Subagent fields）

| Cursor | Claude Code |
| --- | --- |
| `subagent_type: generalPurpose` | `subagent_type: "general-purpose"` |
| `poteto-agent`、`comment-sicko` | `pstack:poteto-agent`、`pstack:comment-sicko` |
| `poteto-agent` のファイルにある `model:` の行 | Cursor の slug が書いてある。`pstack:poteto-agent` を起動する `Agent` の呼び出しには、毎回、モデル（Models）の表で読み替えた `model` と `effort` を渡し、その slug が API に届かないようにする。 |
| `readonly: true` | `subagent_type: "Explore"`。`Bash` は使えるままなので、依頼文にも「ファイルを編集しない」と書き、終わったあとで `git status` を確かめる。 |
| `readonly: false` | `general-purpose`。セッションの MCP サーバーを使えるままにする。 |
| `environment: "cloud"` | `isolation: "worktree"`。ワーカーがチェックアウトするブランチかコミットを指定する。 |
| `environment: "local"` | 渡さない。 |
| `cloud_base_branch: <b>` | まず `<b>` をチェックアウトするようワーカーに指示する。 |
| `run_in_background: true` | 渡さない。サブエージェントは既定でバックグラウンドで動き、終わると通知が来る。 |
| `resume` | 新しいエージェントを起動し、前の結果を依頼文に貼る。 |
| `file_attachments` | パスを依頼文に列挙する。 |

## モデル（Models）

| Cursor | Claude Code |
| --- | --- |
| `claude-<family>-<version>-<effort>[-fast]`（例: `claude-opus-5-5-xhigh`） | `model: "opus"` と `effort: "xhigh"`。`model` に渡せるのは `opus`、`sonnet`、`haiku`、`fable` だけである。`-fast` は落とす。 |
| `gemini-*` などの Claude 以外の slug | `model` を渡さずにエージェントをセッションのモデルで動かし、そうしたことを伝える。 |
| `auto` または `inherit-parent` | `model` を渡さない。 |
| `pstack-models.mdc` のルール、`~/.cursor/rules/pstack-models.mdc` | `~/.claude/rules/pstack-models.md`。Claude Code は新しいセッションのたびにこのファイルを読み込むので、ロールの行はすでにコンテキストにある。なければファイルを読む。ファイルかそのロールの行がなければ、スキルの既定の slug を使う。 |
| ロールの値 `<model> <effort>`（例: `opus high`） | `model: "opus"` と `effort: "high"`。 |

## setup-pstack

Claude Code は `~/.claude/rules/` にあるすべての `.md` ファイルを新しいセッションのたびに読み込むので、`setup-pstack` はルールをそこに書く。手順には次の変更を加えて従う。

- 検出（Detect）。検出するモデルは、`Agent` ツールのスキーマが挙げる `model` の値であり、それぞれの effort は `effort` の値から選ぶ。実在のモデルは `<model> <effort>` の形（例: `opus xhigh`）で書く。effort が最後のトークンになるので、予算（budget）の手順は書かれたとおりに動く。
- 既定値（Defaults）。既定の slug は上の行で読み替える。`claude-opus-5-5-xhigh-fast` は `opus xhigh`、`claude-opus-5-5-high-fast` は `opus high`、`claude-fable-5-2-high` は `fable high`、`gemini-4-neon-xhigh` は `inherit-parent` になる。
- 書き込み（Write）。`~/.claude/rules/pstack-models.md` を frontmatter なしで書き、コメント行、`# budget` の行、手順 5 のロールごとの行を続ける。ファイル全体を上書きする。ロールの行を `CLAUDE.md` や `AGENTS.md` に書くことは決してしない。

## パス（Paths）

| Cursor | Claude Code |
| --- | --- |
| `.cursor/skills/`、`~/.cursor/skills/` | `.claude/skills/`、`~/.claude/skills/` |
| `~/.cursor/plugins/`、`.cursor-plugin/plugin.json` | `~/.claude/plugins/`、`.claude-plugin/plugin.json` |
| `.cursor/rules/*.mdc`、常に適用されるルール（always-applied rules） | `CLAUDE.md`。`CLAUDE.md` がなければ `AGENTS.md`。Claude Code は `.mdc` ファイルを決して読まない。モデルのルールは例外である。モデル（Models）を参照。 |
| `https://github.com/cursor/plugins/blob/main/pstack/<path>` | そのファイルがあれば `<pstack>/<path>`。プラグインはスキル、`docs/guide/`、`README.md` を同梱している。ローカルのコピーを読み、決して取得しにいかない。ユーザーがリンクを必要とするときは URL を渡す。 |
| `~/.cursor/projects/<slug>/agent-transcripts/` にあるトランスクリプト | `~/.claude/projects/<slug>/<session>.jsonl`。slug はワークスペースのパスの `/` と `.` をそれぞれ `-` に変えたもので、`/Users/you/proj` なら `-Users-you-proj` になる。そのフォルダの直下にある最新の `.jsonl` が現在のセッションである。その `<session>/subagents/` フォルダにサブエージェントのトランスクリプトがある。読むのはこのワークスペースのフォルダだけにする。 |

## Cursor 専用の機能（Cursor-only features）

Claude Code には、Cursor Automations、Bugbot、Cursor の cloud agent、組み込みの PR ツール（built-in PR tool）がない。手順がこれらを必要とするときは、返答に `Skipped in Claude Code: <step>. <reason>.` という 1 行を書いて、次の手順に進む。止まったり、質問したり、やり直したりしない。飛ばした手順が作るはずだったもの（URL やキーなど）をあとの手順が必要とするときは、プレースホルダーを置き、その行でプレースホルダーの名前を挙げる。

| Cursor | Claude Code |
| --- | --- |
| Cursor Automations: ルーチン、`update_state` によるルーチンの呼び出し、`api2.cursor.sh/automations` の webhook、ルーチンの起動、`SendToUser` による秘密情報の要求 | 上の 1 行を書いて飛ばす。 |
| Bugbot | GitHub の PR に Bugbot が付けたコメントは普通のレビューのスレッドなので、手順が示すフォージのコマンドでトリアージする。Bugbot を起動する・待つ・設定する手順は、PR に Bugbot のチェックもコメントもなければ飛ばす。 |
| Cursor の cloud agent | `environment: "cloud"` は、サブエージェントのフィールド（Subagent fields）のとおり worktree に読み替える。cloud agent そのもの（ダッシュボードの状態や専用のマシンなど）が必要な手順は飛ばす。 |
| 組み込みの PR ツール（built-in PR tool） | 手順が示すフォージのコマンド（`gh pr create` など）を使う。これは飛ばしではないので、1 行は書かない。 |
