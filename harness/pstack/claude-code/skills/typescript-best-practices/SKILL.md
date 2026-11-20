---
name: typescript-best-practices
description: TypeScript のベストプラクティス。.ts や .tsx のファイルを読む・編集するときに使う。
paths: ["**/*.ts", "**/*.tsx"]
disable-model-invocation: true
---

# TypeScript のベストプラクティス（TypeScript best practices）

まず原則スキル **type-system-discipline**（型システムの規律）を適用する。

| ルール | 要約 |
|------|---------|
| 判別可能なユニオン型（Discriminated unions） | バリアントは `kind` のリテラルの判別子で表し、ありえない状態を表現できないようにする。省略可能なフィールドを寄せ集めた型にしない。 |
| ブランド型（Branded types） | プリミティブに `& { readonly __brand: "X" }` でブランドを付け、取り違えられないようにする。検証は境界で 1 回だけ行う。ブランドは実際に検証する型ガード（`input is AgentId`）で付け、決して `as` では付けない。 |
| 構成的なモデリング（Constructive modeling） | 不正な値を構築できない形に型を組む。空でない配列は `[T, ...T[]]`、偶数長は `[T, T][]`、範囲は `start` と `duration` で表す。実行時のガードで守るのでも、篩型（refinement types）があればと願うのでもない。 |
| 最も単純な全域の型（Simplest total type） | `T[]` に対する操作がすべて全域（total）である限り、`T[]` のままにする。緩い型のせいで `!`、キャスト、「起きるはずがない」例外の throw が必要になる箇所でだけ、`NonEmpty<T>` に強める。 |
| `any` より `unknown`（`unknown` over `any`） | 外部のデータは `unknown` とする。 |
| 型ガードより先にスキーマ（Schemas before guards） | プロパティを 1 つずつ調べる型ガードを手で書く前に、リポジトリの実行時スキーマライブラリを使い、`z.infer` のようにスキーマから型を推論する。 |
| `as` のキャストをしない（No `as` casts） | どの `as` も、実行時にクラッシュする可能性を残す。キャストは検証のあとにだけ行う。 |
| 絞り込みの優先順位（Narrowing hierarchy） | 判別子での switch > `in` 演算子 > `typeof`/`instanceof` > ユーザー定義の型ガード > `as`。 |
| 型ガード（Type guards） | 主張する型を必ず実際に検証する。嘘をつくガードは `as` より悪い。安全だと読める名前のせいで、バグに気づきにくくなるからである。名前は `isX` か `hasX` にする。 |
| 網羅性（Exhaustiveness） | default の分岐に `const _exhaustive: never = x;` を直接書き、新しいバリアントを足したときにコンパイラがエラーを出すようにする。 |
| `as` より `satisfies`（`satisfies` over `as`） | リテラル型を広げずに値を検証する。 |
| 境界での検証（Boundary validation） | データが入ってくる場所でパースし、名前の付いたドメインの型にする。`Record<string, unknown>`（どう書いたものであれ）は、そのパースより内側に持ち込まない。内側では型を信頼する。原則スキル **boundary-discipline**（境界の規律）を参照。 |
| スキーマから導く型（Schema-derived types） | 新しい interface を宣言する前に、`Pick`/`Omit`/`Parameters`/`ReturnType`/`Awaited`/`typeof` を使えないか考える。 |
| オブジェクト引数（Object args） | 位置引数ではなくオブジェクトを渡し、引数の並びが説明なしで読めるようにする。ホットパス（フレームごとの描画、トークナイザー、パーサー）では使わない。 |
| 本物のテスト（Real tests） | 実行できるものはモックしない。フレームワークの本物のテスト用プリミティブを、リークや破棄（disposable）のチェック付きで使い、UI は動いているビルドで確かめる。モックするのは、ローカルで実行できないものだけにする。 |
| 構造化したテレメトリ（Structured telemetry） | ID からデバッグできるだけの文脈を持つ、構造化したロガーの診断を使う。リリースするコードに `console.log` を残さない。 |

例: `references/patterns.md`。
