---
name: principle-migrate-callers-then-delete-legacy-apis
description: "古い呼び出し元が残っている状態で、新しい内部 API を導入するときに適用する。互換レイヤーを残すのではなく、同じ一連の作業の中で呼び出し元を移行し、古い API を削除する。"
disable-model-invocation: true
---

# 呼び出し元を移してからレガシー API を消す（Migrate Callers Then Delete Legacy APIs）

- テストを、新しい契約を検証するように更新し、リファクタリング前の実装の詳細を守るだけのテストは削除する

**適用する条件:**
- 後方互換性に依存する外部のユーザーがいない
- プロジェクトが、足並みを揃えた破壊的変更を受け止められる
- 新しい API が、単純化やリファクタリングの取り組みの一部である
