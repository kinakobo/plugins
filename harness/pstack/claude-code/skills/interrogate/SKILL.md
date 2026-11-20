---
name: interrogate
description: "「interrogate」「adversarial review」「multi-model review」「challenge this」「stress test this code」「find blind spots」「tear this apart」と言われたときに使う。攻撃役が変更や設計を壊そうとし、実行で証明された発見だけが残る。"
disable-model-invocation: true
---

# 問い詰める（Interrogate）

In Claude Code, read [the Claude Code adapter](../../docs/harness/claude-code.md) once per session before you follow this skill. It maps the Cursor names in pstack skills to Claude Code.

攻撃役（adversary）が変更や設計を壊そうとする。発見が残るのは、実行によって証明されたときだけである。発見を報告する。修正は適用しない。

## 枠組みを決める（Frame）

範囲は、ユーザーが指定したものである。指定がなければ、ベースブランチとのマージベースから作業ツリーまでの、ブランチの diff である。ユーザーのメッセージ、コミット、PR の説明から、**in-your-own-words** スキルで意図を述べる。意図がはっきりしないときはユーザーに尋ねる。

## ファンアウト（Fan out）

**arena** スキルのフェーズ A（Phase A）の手順 3 と 4、およびフェーズ B（Phase B）を、4 つの変更を加えて実行する。`arena runners` の代わりに `interrogate reviewers` の行を読む。範囲内の未コミットの変更を各 worktree にコピーする。各攻撃役を `generalPurpose` として起動する。arena のタスクと根拠の代わりに、範囲、意図、次の依頼文を各攻撃役に渡す。

> この変更を壊す。各発見は、実際のコードに対する実行で証明する。たとえば、失敗するテスト、再現スクリプト、誤った振る舞いを示すコマンドの出力である。設計や計画が対象なら、プロトタイプを動かすか、いまのツリーを調べる。再現用のファイルは自分の worktree に置く。各発見は、タイトル、実行したコマンドそのもの、それが出力した内容、意図に照らして本来出るべき出力を添えて報告する。実行していないものは含めない。スタイル、命名、構造、将来の変更があって初めて問題になるリスクもこれに当たる。発見がゼロでも正当な結果である。

## 検証（Verify）

arena のフェーズ F（Phase F）に従い、各発見のコマンドを再実行して検証する。出力に失敗が現れ、その失敗が意図を損なうなら、その発見を残す。残りは捨てる。根本原因が同じ発見はまとめる。

arena のフェーズ C、D、E は飛ばす。攻撃役はそれぞれ別の失敗を見つけるので、判定には、1 人の攻撃役の一覧ではなく、検証済みの発見をすべて残す。

## 出力（Output）

- **意図（Intent）**: 言い直した内容。
- **発見（Findings）**: 検証済みの発見ごとに、コマンド、その出力、それが意図をどう損なうか、それを見つけた攻撃役。残った発見がなければ「検証済みの発見なし」と書く。
- **除外（Dropped）**: 攻撃役ごとに、捨てた発見の数とその理由。
