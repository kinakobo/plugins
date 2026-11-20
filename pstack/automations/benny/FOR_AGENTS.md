# benny の Automation の意図（benny automation intent）

## 自動化したいこと（what i want to automate）

1 つの Slack の issue チャンネルで連携して動く、2 つの Cursor Automation がほしい。

### Automation 1: issue の報告のトリアージ（automation 1: triage issue reports）

- トリガー: 設定した報告元の Slack チャンネルに、誰かがトップレベルの新しい報告を投稿したら、この Automation にはその報告から始まり、元のスレッドの位置（coordinates）を保持してほしい。
- 振る舞い: スレッドと添付ファイルを読み、報告を、バグまたはパフォーマンスの問題、機能要望、質問・フィードバック、振り分け直しのいずれかに分類し、振り分ける前に、問題を担当していそうなレイヤーを辿ってほしい。
- トラッカー: 設定したトラッカーで重複を探し、確実な重複なら既存のチケットを更新し、新規であることが明らかなバグに限ってチケットを作ってほしい。
- ツール: Slack のスレッドの読み取りと返信の権限、設定したトラッカーの連携、任意の振り分けマップを使えるようにしたい。
- 結果: 報告元のスレッドに、短い判定と `[benny:bug]`、`[benny:performance]`、`[benny:other]` のいずれかを付けた返信を、ちょうど 1 件だけ投稿してほしい。bug と performance のマーカーには、トラッカーの URL を含めてよい。
- 境界: この Automation には、報告元のチャンネルに決してルートメッセージを投稿してほしくない。

### Automation 2: 確認されたバグの再現と修正（automation 2: reproduce and fix confirmed bugs）

- トリガー: この Automation には、同じトップレベルの新しい報告か、セットアップで選んだサポートされている別のトリガーから始まり、元のスレッドで信頼できるトリアージのマーカーを待ってほしい。
- ゲート: 誰かが明らかに修正を引き受けているときは止まってほしい。既存のプルリクエストやマージ済みのコミットで報告の問題が直る可能性があるなら、競合する変更を作るのではなく、それを検証してほしい。
- 振る舞い: 設定した操作アダプター（control adapter）と機能マップを使い、報告どおりの症状を実際の UI で 2 回再現し、スクリーンショット、動画、読み取りだけで行う状態の照合を記録してほしい。
- 修正: 既存のプルリクエストは、その上に自分の変更を書かずに検証してほしい。再現が確認できたら、範囲を限った根本原因の修正を 1 回試み、テストが安く書けるなら TDD を使い、影響範囲をスモークテストし、修正前と修正後の証明が通ったときに限ってドラフトのプルリクエストを開いてよい。
- ツール: Slack のスレッドの読み取りと返信の権限、リポジトリとその履歴へのアクセス、ドラフトのプルリクエストの作成、設定したトラッカー、操作アダプターを使えるようにしたい。
- 結果: 報告元のスレッドか、任意の運用チャンネルのスレッドに、証拠と検証済みの結果を投稿し、任意でドラフトのプルリクエストも出してほしい。途中経過の投稿は簡潔にしてほしい。
- 境界: この Automation には、報告元のチャンネルに決してルートメッセージを投稿してほしくない。

### 共通の規則（shared rules）

- 報告元のチャンネルとルートスレッドの位置は、実行の最初から最後まで変えないでほしい。
- ユーティリティやデバッグ用のボットは証拠として扱い、委任や修正の引き受けとは見なさない。
- サブエージェントに手伝わせてよいが、サブエージェントは Slack に投稿できず、Slack の認証情報も受け取れない。
- このパック全体を、対象リポジトリの `.cursor/automations/benny/` にコミットしてほしい。パックの `SKILL.md` ファイルは Automation が直接読む指示であり、プラグインに登録されたスキルではない。
- `why`、`tdd`、`unslop` や必要な原則のスキルといった共有の依存先のためだけに、対象リポジトリにコミットした `.cursor/settings.json` で pstack を有効にしてほしい。
- 実際に動く各 Automation のプロンプトには、コミット済みの手順ファイルを直接読ませたい。プラグインのキャッシュのパス、コピーした抜粋、スラッシュスキルの探索は使いたくない。
- ユーザーが管理する設定、機能マップ、振り分けマップ、秘密情報は、パックの更新で上書きされないように `.cursor/automations/benny/` の外に置く。
- チャンネルの位置、トラッカーへのアクセス、操作アダプター、機能マップのいずれかが欠けているか不確かなときは、どちらの Automation も安全側に倒して止まってほしい（fail closed）。
- プルリクエストはドラフトだけにしてほしい。マージもデプロイもしない。

### 自分の設定（my configuration）

- 報告元の Slack チャンネル: `<channel>`
- 任意の運用チャンネル: `<channel or none>`
- リポジトリと既定ブランチ: `<repo>`、`<branch>`
- トラッカー: `<type, team, project, labels, intake status>`
- 振り分けマップ: `<path or none>`
- トリアージのアカウント: `<slack identity>`
- 操作スキル: `<configured skill or adapter>`
- 機能マップ: `<committed same-repo path outside the copied pack, or behavior to paraphrase>`
- モデル: `<triage, reproduce, code, media review>`
- 状態を示す絵文字の文字列: `<seen, reproducing, reproduced, blocked, fixing, failed, pull request opened>`
- 予算: `<polling, verdict wait, follow-up, repro, rejection, fix>`
- 任意のボットトークンでできること: `<none, file download, or editable operations status>`

[`configuration.example.yaml`](./templates/configuration.example.yaml) と [`feature-map.example.md`](./skills/reproduce-and-fix-issues/references/feature-map.example.md) から始める。これらは、このパックの外（たとえば `.cursor/benny/` の下）にコピーしてから埋める。秘密の値は、シークレットマネージャーか環境変数に置く。

## エージェント向け（for the agent）

人間は、Cursor にこのファイルを渡してセットアップを始める。benny のスラッシュスキルを探したり、見つかったものを呼び出したりしない。

1. Automation を動かすリポジトリはどれかを尋ねる。
2. この `FOR_AGENTS.md` を含むディレクトリを、コピー元のパックとして扱う。
3. コピー元のパック全体を `<target-repository>/.cursor/automations/benny/` にマージする。
4. コピー先にだけあるファイルはすべて残す。関係のないファイルを決して削除せず、ユーザーが管理する設定、機能マップ、振り分けマップを決して上書きしない。
5. パックが管理するパスにあるコピー先の既存ファイルが異なるときは、差分を確認し、ローカルの編集を捨てずにマージする。誰が管理するファイルかがはっきりしないときは、置き換える前に止まって尋ねる。
6. コピーした `FOR_AGENTS.md` と `skills/setup-benny/SKILL.md` が対象リポジトリにあることを確かめる。
7. 対象リポジトリの `.cursor/automations/benny/skills/setup-benny/SKILL.md` を直接読み、それに従う。

対象リポジトリの `.cursor/settings.json` に、次のエントリーをマージしてほしい。

```json
{
	"plugins": {
		"pstack": { "enabled": true }
	}
}
```

関係のない設定とプラグインはすべて残す。既存のファイルが JSONC なら、コメントと JSONC として正しい構文を保つ。

検証は、対象リポジトリをルートにした新しいエージェントで行ってほしい。pstack の `why`、`tdd`、`unslop` と、benny が使う原則のスキルが、プロジェクトスコープで解決されることを確かめる。現在のセッションや、ユーザースコープでインストールしたものから読み込まれたスキルは数えない。

プロジェクトスコープのプラグインが使えないか、共有の依存先のどれかが解決されないときは、止まって、何が失敗したかを説明する。`.cursor/automations/benny/skills/` をプラグインのマニフェストに追加したり、そのファイルがスラッシュスキルの一覧に現れると期待したりしない。

どちらかの Automation を有効にする前に、`.cursor/settings.json`、`.cursor/automations/benny/`、参照している秘密情報を含まない設定をコミットしなければならないことを、私に伝える。私がはっきり頼むまで、Automation を作ったり更新したりしない。

初めて作るときは、組み込みの `/automate` を、トリアージ用に 1 回、再現と修正用に 1 回使う。1 つ目の Automation について、下書きのレビュー、承認、準備状況の確認、Automations エディターへの引き継ぎを終えてから、2 つ目に取りかかる。

この意図と完成した設定を、言い換えてそれぞれの下書きに書き込む。トリアージのプロンプトでは、必ず `.cursor/automations/benny/skills/triage-issue-reports/SKILL.md` を読んで従わせる。再現のプロンプトでは、必ず `.cursor/automations/benny/skills/reproduce-and-fix-issues/SKILL.md` を読んで従わせる。これらのリポジトリ相対パスを使うのは、Automation を動かすリポジトリにそのファイルがコミットされていることを `/automate` が確かめてからにする。

既存の Automation については、`/automate` で中身を確かめたり更新したりしない。設定を検証してから、コピーしたセットアップのファイルにある、項目の簡潔なチェックリストを使い、私が各 Automation をそれぞれのエディターで直接編集できるようにする。重複を作らない。
