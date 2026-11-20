# TypeScript のパターン（TypeScript patterns）

`SKILL.md` の各ルールのコード例。根にある原則は言語に依存しない。原則スキル **type-system-discipline**（型システムの規律）と **boundary-discipline**（境界の規律）を参照。

## ブランド型（Branded types）

プリミティブにブランドを付け、取り違えられないようにする。検証は境界で 1 回だけ行う。下流のコードは型を信頼する。

```ts
type AgentId = string & { readonly __brand: "AgentId" };

function isAgentId(input: string): input is AgentId {
  return isUUID(input);
}

function parseAgentId(input: string): AgentId {
  if (!isAgentId(input)) throw new Error(`Invalid agent id: ${input}`);
  return input;
}

function focusAgent(id: AgentId): void {
  /* input is trusted */
}
```

`readonly __brand: 'X'` の形に合わせる。新しい書き方を発明しない。

## 判別可能なユニオン型（Discriminated unions）

バリアントはリテラルの判別子で表す。どのバリアントも同じフィールド名を持ち、その値はバリアントごとに異なるので、ありえない組み合わせを表現できない。

```ts
// Don't. Boolean + optionals lets contradictory states exist.
type DiffState = { loading: boolean; diff?: GitDiff; error?: string };

// Do. Only valid states exist.
type DiffState =
  | { kind: "loading" }
  | { kind: "ready"; diff: GitDiff }
  | { kind: "error"; error: string };
```

判別子の名前（`kind`、`type`、`tag`）は 1 つに決めて、それで通す。

## 構成的なモデリング（Constructive modeling）

緩い型を実行時のチェックで制限するのではなく、どれも正当な部品から型を組み立てる。

空でない配列は、可変長タプルで表す。

```ts
type NonEmpty<T> = [T, ...T[]];

// Don't: T[] plus a length check every caller must repeat
function pickWinner(entries: string[]): string {
  if (entries.length === 0) throw new Error("no entries");
  return entries[Math.floor(Math.random() * entries.length)];
}

// Do: an empty value of the type can't exist
function pickWinner(entries: NonEmpty<string>): string {
  return entries[Math.floor(Math.random() * entries.length)];
}
```

ただの `T[]` を受け取る箇所では、ガードで 1 回だけ絞り込む。それ以降、空でないという事実は型として引き継がれる。

```ts
const isNonEmpty = <T>(arr: T[]): arr is NonEmpty<T> => arr.length > 0;
```

偶数長は、ペアの配列で表す。

```ts
type Pairs<T> = [T, T][];
```

時間の範囲は、開始と長さで表す。

```ts
// Don't: a comment holds the invariant
type TimeRange = { start: Date; end: Date }; // start <= end

// Do: a negative range can't be written; derive end when needed
type TimeRange = { start: Date; durationMs: number };
```

`durationMs` はただの number のままにする。ブランドを付ける（ブランド型（Branded types）のとおりに）のは、長さを期待する箇所に生の数値を渡せてしまうときだけで、反射的には付けない。不正な状態を構築できなくなる表現を選び、そのうえで必要な読み方を用意する（`pairs.flat()`、`rangeEnd()` のヘルパー）。

## 最も単純な全域の型（Simplest total type）

何でも強めない。`T[]` に対する操作がすべて全域（total）なら、`T[]` のままにする。

```ts
const sum = (xs: number[]) => xs.reduce((a, b) => a + b, 0); // [] is 0, fine
```

緩い型のせいで使う箇所に嘘を書くことになるなら、型を強める。その兆候は `!`、`arr[0] as T`、「起きるはずがない」例外の throw である。

```ts
// Don't: partiality smuggled past the compiler
function newestSession(sessions: Session[]): Session {
  return sessions.at(0)!;
}

// Do: strengthen the input; the assertion disappears
function newestSession(sessions: NonEmpty<Session>): Session {
  return sessions[0];
}
```

結果を `Session | undefined` に弱めるのが、もう 1 つの全域なシグネチャである。

## `any` より `unknown`（`unknown` over `any`）

外部のデータは常に `unknown` とする。使う前に絞り込む。

```ts
// Don't
function handle(input: any) {
  return input.foo.bar;
}

// Do
function handle(input: unknown) {
  if (typeof input === "object" && input !== null && "foo" in input) {
    // narrowed; compiler verifies access
  }
}
```

外部のデータの出どころには、RPC のペイロード、`JSON.parse`、`postMessage`、IPC、ファイルの中身、環境変数、データベースの結果がある。

## 手書きの型ガードより先にスキーマ（Schemas before hand-rolled guards）

外部のデータに対して、プロパティを 1 つずつ調べる型ガードを書く前に、リポジトリの実行時スキーマライブラリと既存のスキーマを探す。検証は 1 つのスキーマに任せ、TypeScript の型はそのスキーマから導く。スキーマ、重複した interface、ガードの 3 つを別々に保守して、食い違わせることのないようにする。

```ts
import { z } from "zod";

const UserSchema = z.object({
  id: z.string().uuid(),
  role: z.enum(["admin", "member"]),
});

type User = z.infer<typeof UserSchema>;

function parseUser(input: unknown): User {
  return UserSchema.parse(input);
}
```

失敗が想定内の分岐なら `safeParse` を使う。リポジトリが別のスキーマライブラリを使っているなら、それに相当する推論のヘルパーを使う。ガード 1 つのために、スキーマの依存を新しく足さない。このルールは、コードベースがすでに信頼しているスキーマの仕組みを優先する。

## `as` のキャストをしない（No `as` casts）

どの `as` も、実行時にクラッシュする可能性がある。キャストは、型システムが主張を検証したあとにだけ行う。

```ts
import { z } from "zod";

// Don't
const user = data as User;

// Don't
function isUser(data: unknown): data is User {
  return typeof data === "object" && data !== null && "id" in data;
}

// Do
const userSchema = z.object({ id: z.string(), name: z.string() });
type User = z.infer<typeof userSchema>;

function parseUser(data: unknown): User {
  return userSchema.parse(data);
}
```

型が先にあるときは、バリデーターに、それが証明する型の注釈を付ける。そうすると、型が求めるより少ないことしか証明しないバリデーターを、コンパイラが拒否する。下のオブジェクトから `name` を取り除くと、代入がコンパイルエラーになる。

```ts
type User = { id: string; name: string };

const userSchema: z.ZodType<User> = z.object({ id: z.string(), name: z.string() });
```

既存のコードから `as` を取り除くときは、TypeScript が型を推論できない理由を突き止める。

- 判別子がない: 判別子を足し、判別可能なユニオン型に切り替える。
- 元の型が広すぎる（例: `Record<string, unknown>`）: 絞り込む。
- 型の付いていない境界: その形を受け持つスキーマでパースする。スキーマを足すのは、まだ存在しない場合だけにする。
- 本当に型で表現できない: ブランド型か `satisfies` を使う。

## 絞り込みの優先順位（Narrowing hierarchy）

最善のものから最後の手段の順に並べる。

1. **判別可能なユニオン型での switch / if。** コンパイラが自動で絞り込む。
2. **`in` 演算子。** `"key" in obj` は、そのキーを含むバリアントに絞り込む。
3. **`typeof` / `instanceof`。** プリミティブとクラスのインスタンスに使う。
4. **ユーザー定義の型ガード。** ここまでの方法で足りないときに使う。
5. **`as` のキャスト。** 検証のあとにだけ使う。

```ts
function area(s: Shape): number {
  if ("radius" in s) return Math.PI * s.radius ** 2; // narrowed to circle
  return s.width * s.height; // narrowed to rect
}
```

## 型ガード（Type guards）

ガードは主張する型を実際に検証しなければならない。嘘をつくガードは `as` より悪い。

```ts
function isCircle(s: Shape): s is Shape & { kind: "circle" } {
  return s.kind === "circle";
}
```

できるなら、判別子による絞り込みを選ぶ。

## 網羅性（Exhaustiveness）

default の分岐で、判別対象の値を `never` 型のローカル変数に代入する。

```ts
// Value-returning switch
function area(s: Shape): number {
  switch (s.kind) {
    case "circle":
      return Math.PI * s.radius ** 2;
    case "rect":
      return s.width * s.height;
    default: {
      const _exhaustive: never = s;
      return _exhaustive;
    }
  }
}

// Void switch
function handle(s: Shape): void {
  switch (s.kind) {
    case "circle":
      drawCircle(s);
      break;
    case "rect":
      drawRect(s);
      break;
    default: {
      const _exhaustive: never = s;
      void _exhaustive;
    }
  }
}
```

値を返す switch では return の形、文として使う switch では void の形にする。

## `as` より `satisfies`（`satisfies` over `as`）

`satisfies` は、リテラル型を広げずに検証する。

```ts
// Don't. Widens, loses literal types.
const config = { theme: "dark", cols: 3 } as Config;

// Do. Validates AND preserves literal types.
const config = { theme: "dark", cols: 3 } satisfies Config;
// config.theme is "dark" (literal), not string
```

## 境界での検証（Boundary validation）

データが入ってくる場所で 1 回だけ検証する。内側では型を信頼する。原則スキル **boundary-discipline**（境界の規律）を参照。

- **ワイヤーフォーマット**（proto、JSON-RPC）: `ignoreUnknownFields` を付けてパースし、前方互換の変更で古いクライアントが壊れないようにする。
- **永続化した JSON:** バージョン付きの blob にし、パースを try/catch で囲む。
- 呼び出しの連鎖の奥深くで**検証し直さない**。

## スキーマから導く型（Schema-derived types）

`.proto`、OpenAPI の仕様、GraphQL のスキーマ、データベースのマイグレーションがすでに形を定義しているなら、それを複製せず、生成された型から導く。

```ts
// Don't. Duplicate shape, drifts when the schema changes.
type CheckSummary = {
  totalCount: number;
  checks: { name: string; status: string }[];
};
function renderChecks(s: CheckSummary) {
  /* ... */
}

// Do. Derive from the generated schema type.
import type { ChecksMessage } from "<generated module>";
function renderChecks(s: Pick<ChecksMessage, "totalCount" | "checks">) {
  /* ... */
}
```

新しい interface を書く前に、`Pick`、`Omit`、`Parameters`、`ReturnType`、`Awaited`、`typeof` を使えないか考える。

## オブジェクト引数（Object args）

```ts
// Don't. Swap two args, still compiles.
openFile(uri, {
  startLineNumber: 10,
  startColumn: 1,
  endLineNumber: 10,
  endColumn: 1,
});

// Do. Order-independent, self-documenting.
openFile({
  uri,
  selection: {
    startLineNumber: 10,
    startColumn: 1,
    endLineNumber: 10,
    endColumn: 1,
  },
});
```

ホットパスでは使わない。フレームごとの描画、トークナイザー、パーサーなど、割り当てのコストが問題になる密なループの中がそれにあたる。
