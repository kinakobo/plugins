# benny

benny は、Slack に投稿される issue の報告を扱う 2 つの Cursor Automation を提供する。1 つは、報告を 1 件ずつトリアージする。もう 1 つは、確認されたバグを再現し、小さな修正をドラフトで用意することもある。

このディレクトリのファイルは、セットアップと Automation の元になるファイルで、置いただけでは動かない。スラッシュスキルとしても表示されない。

## セットアップ（set it up）

1. Cursor に [`FOR_AGENTS.md`](./FOR_AGENTS.md) を渡し、対象のリポジトリを伝える。
2. セットアップの手順に、このディレクトリ全体を対象リポジトリの `.cursor/automations/benny/` へマージさせる。セットアップは、コピー先にだけあるファイルを残し、ローカルの編集を上書きせずに衝突を確認しなければならない。
3. 共有の依存先を使うため、セットアップの手順に、対象リポジトリの `.cursor/settings.json` で pstack を有効にさせる。

```json
{
	"plugins": {
		"pstack": { "enabled": true }
	}
}
```

4. ユーザーが管理する設定は、コピーしたパックの外（たとえば `.cursor/benny/`）に置く。[`configuration.example.yaml`](./templates/configuration.example.yaml) と [`feature-map.example.md`](./skills/reproduce-and-fix-issues/references/feature-map.example.md) を自分の環境に合わせて書き換える。
5. どちらかの Automation を有効にする前に、`.cursor/settings.json`、`.cursor/automations/benny/`、秘密情報を含まない設定をコミットする。
6. 新しい Automation の下書きをそれぞれ確認するか、既存の Automation をそれぞれのエディターで更新する。そのあと、害のないテスト用の報告を送り、報告元のチャンネルへの投稿がすべて元のスレッドの中にとどまることを確かめる。
