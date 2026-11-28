# TypeScript — ICICI Node.js Backend Interview Prep

TypeScript is expected knowledge for a senior Node.js backend role. Focus on type safety, when to use each feature, and how `tsconfig` shapes correctness in production banking/fintech APIs.

---

### Q1. What is TypeScript vs JavaScript? `[Critical]`
**Answer:** TypeScript is a statically typed superset of JavaScript that compiles (transpiles) to plain JS. It adds interfaces, generics, enums, and compile-time checks so many bugs are caught before runtime. At runtime you still run JavaScript; TypeScript exists only at build/dev time unless you use a runtime loader like `tsx`.
**Example:**
```ts
// TypeScript — type error at compile time
function transfer(amount: number) {
  return amount.toFixed(2);
}
transfer("100"); // Error: Argument of type 'string' is not assignable to parameter of type 'number'

// JavaScript — fails (or behaves oddly) only at runtime
function transfer(amount) {
  return amount.toFixed(2);
}
transfer("100"); // TypeError: amount.toFixed is not a function
```
**Follow-up:** Why use TS in Node backends? → Safer refactors, better IDE autocomplete, clearer API contracts between services.

| | TypeScript | JavaScript |
|---|---|---|
| Typing | Static (compile-time) | Dynamic (runtime) |
| Tooling | Strong IDE support, refactor safety | Limited without JSDoc |
| Runtime | Erased; runs as JS | Native runtime language |
| Learning curve | Higher | Lower |

---

### Q2. Interface vs type — when do you use each? `[Critical]`
**Answer:** Both describe object shapes. `interface` supports declaration merging and is preferred for object contracts that may be extended. `type` aliases can represent unions, intersections, primitives, tuples, and mapped types — things interfaces cannot express alone. Prefer `interface` for public object APIs; use `type` for unions and computed types.
**Example:**
```ts
interface User {
  id: string;
  name: string;
}
interface User {
  email: string; // declaration merging — both fields exist
}

type Status = "pending" | "approved" | "rejected"; // union — type only
type Admin = User & { role: "admin" };             // intersection
```
**Follow-up:** Can interfaces express unions? → No. Use `type` for unions like `"A" | "B"`.

| Feature | `interface` | `type` |
|---|---|---|
| Object shapes | Yes | Yes |
| Unions / primitives | No | Yes |
| Declaration merging | Yes | No |
| Extends / implements | Natural fit | Works via intersection |
| Mapped / conditional types | Limited | Full support |

---

### Q3. What is the difference between `any` and `unknown`? `[Critical]`
**Answer:** `any` disables type checking — you can call anything on it and TypeScript will not complain. `unknown` is the type-safe top type: you must narrow (typeof, instanceof, type guards) before using it. Prefer `unknown` for untrusted input (HTTP body, JSON.parse, external APIs).
**Example:**
```ts
function handleAny(value: any) {
  value.trim(); // compiles — may crash at runtime
}

function handleUnknown(value: unknown) {
  // value.trim(); // Error
  if (typeof value === "string") {
    value.trim(); // safe after narrowing
  }
}

const raw: unknown = JSON.parse(payload);
```
**Follow-up:** When is `any` acceptable? → Rare escape hatches, gradual migration, or third-party typings that are broken — never as the default for request bodies.

| | `any` | `unknown` |
|---|---|---|
| Assignable to other types | Yes (unsound) | No (must narrow) |
| Operations allowed | All | Almost none until narrowed |
| Safety | None | Forces validation |
| Use case | Escape hatch | Untrusted / external data |

---

### Q4. What are union and intersection types? `[High]`
**Answer:** A union (`A | B`) means the value can be either A or B — you must narrow before accessing exclusive members. An intersection (`A & B`) means the value must satisfy both A and B at once. Unions model alternatives (status, event kinds); intersections compose capabilities (mixins, enriched entities).
**Example:**
```ts
type Success = { ok: true; data: string };
type Failure = { ok: false; error: string };
type Result = Success | Failure; // union

function handle(r: Result) {
  if (r.ok) console.log(r.data);
  else console.error(r.error);
}

type Timestamped = { createdAt: Date };
type Audited = User & Timestamped; // intersection — has User fields + createdAt
```
**Follow-up:** Discriminated unions? → Share a literal tag field (`kind` / `ok`) so TypeScript narrows automatically in each branch.

---

### Q5. Explain generics in TypeScript. `[Critical]`
**Answer:** Generics parameterize types so functions, classes, and interfaces stay reusable without losing type information. Instead of returning `any`, a generic preserves the caller’s type through the call. Common patterns: `Repository<T>`, `ApiResponse<T>`, `Promise<T>`, and constrained generics (`T extends ...`).
**Example:**
```ts
function first<T>(items: T[]): T | undefined {
  return items[0];
}
const n = first([1, 2, 3]);     // number | undefined
const s = first(["a", "b"]);    // string | undefined

interface ApiResponse<T> {
  data: T;
  status: number;
}

function wrap<T extends { id: string }>(entity: T): T {
  return entity; // constrained — T must have id
}
```
**Follow-up:** Default type parameters? → `type Box<T = string> = { value: T }` so callers can omit `T` when string is fine.

---

### Q6. What are enums? When should you avoid them? `[Medium]`
**Answer:** Enums define a named set of constants. Numeric enums are reverse-mapped and can surprise you; string enums are clearer for APIs and logs. Many teams prefer union literal types (`"ACTIVE" | "INACTIVE"`) instead of enums because they erase cleanly, need no runtime object, and work better with tree-shaking.
**Example:**
```ts
enum HttpMethod {
  GET = "GET",
  POST = "POST",
}

// Prefer in many codebases:
type HttpMethod = "GET" | "POST" | "PUT" | "PATCH" | "DELETE";

const constEnumLike = {
  ACTIVE: "ACTIVE",
  CLOSED: "CLOSED",
} as const;
type AccountStatus = typeof constEnumLike[keyof typeof constEnumLike];
```
**Follow-up:** `const enum`? → Inlined at compile time; can break with `isolatedModules` / certain bundlers — know your build setup.

| Approach | Runtime cost | Prefer when |
|---|---|---|
| String enum | Yes (object) | Need namespace + values |
| Numeric enum | Yes + reverse map | Rarely in APIs |
| Union literals | No | Most API contracts |
| `as const` object | Yes (object) | Shared constants + type |

---

### Q7. Name important utility types and when you use them. `[High]`
**Answer:** Utility types transform existing types without rewriting them. Core ones for backend work: `Partial`, `Required`, `Readonly`, `Pick`, `Omit`, `Record`, `Exclude`, `Extract`, `NonNullable`, `ReturnType`, `Parameters`, `Awaited`. Use them to derive DTOs, update payloads, and map keys safely.
**Example:**
```ts
interface Account {
  id: string;
  balance: number;
  currency: string;
  closedAt?: Date;
}

type CreateAccount = Omit<Account, "id" | "closedAt">;
type UpdateAccount = Partial<Pick<Account, "balance" | "currency">>;
type AccountMap = Record<string, Account>;
type SyncResult = Awaited<ReturnType<typeof fetchBalance>>;
```
**Follow-up:** Difference between `Omit` and `Exclude`? → `Omit` removes keys from an object type; `Exclude` removes members from a union.

| Utility | Effect |
|---|---|
| `Partial<T>` | All props optional |
| `Required<T>` | All props required |
| `Readonly<T>` | All props readonly |
| `Pick<T, K>` | Keep keys K |
| `Omit<T, K>` | Remove keys K |
| `Record<K, V>` | Map keys K → value V |
| `NonNullable<T>` | Remove `null` \| `undefined` |
| `Awaited<T>` | Unwrap Promise layers |

---

### Q8. What is type narrowing? `[High]`
**Answer:** Type narrowing refines a wide type to a more specific one using control-flow analysis. Common techniques: `typeof`, `instanceof`, equality checks, `in` operator, truthiness, discriminated unions, and custom type predicates (`value is T`). This is how `unknown` and unions become usable safely.
**Example:**
```ts
function printId(id: string | number) {
  if (typeof id === "string") {
    console.log(id.toUpperCase());
  } else {
    console.log(id.toFixed(0));
  }
}

function isAccount(x: unknown): x is Account {
  return typeof x === "object" && x !== null && "balance" in x;
}
```
**Follow-up:** Exhaustiveness check? → Use `never` in a `default`/`else` branch so adding a new union member fails compilation until handled.

---

### Q9. What is type assertion (`as` / angle-bracket)? When is it dangerous? `[High]`
**Answer:** A type assertion tells the compiler to treat a value as a given type without runtime validation. It does not convert data. Overusing assertions hides bugs — especially casting API payloads straight to domain types. Prefer validation libraries (Zod, Joi, class-validator) that infer or produce typed outputs.
**Example:**
```ts
const body = req.body as CreateAccountDto; // unsafe if body is wrong shape

// Safer pattern:
const parsed = CreateAccountSchema.parse(req.body); // throws if invalid; typed output
```
**Follow-up:** `as const`? → Not a cast to silence errors; it makes literals readonly and narrows to literal types.

---

### Q10. Explain `readonly` and optional properties. `[Medium]`
**Answer:** `readonly` marks a property (or array via `ReadonlyArray<T>` / `readonly T[]`) as immutable after assignment at the type level — runtime mutation is still possible unless you enforce it elsewhere. Optional properties (`prop?: T`) mean the key may be missing; the value type is effectively `T | undefined` when read (with subtle differences under `exactOptionalPropertyTypes`).
**Example:**
```ts
interface Config {
  readonly appName: string;
  port?: number;
}

const cfg: Config = { appName: "payments" };
// cfg.appName = "x"; // Error

function connect(opts: { host: string; timeout?: number }) {
  const timeout = opts.timeout ?? 5000;
}
```
**Follow-up:** `Readonly<T>` vs `readonly` on fields? → `Readonly<T>` makes all properties readonly; field-level `readonly` is selective.

| Concept | Meaning |
|---|---|
| `prop?: T` | Key may be absent |
| `prop: T \| undefined` | Key usually present; value may be undefined |
| `readonly prop` | Cannot reassign via typed code |
| `readonly T[]` | Cannot push/pop via typed array APIs |

---

### Q11. What does TypeScript strict mode do? `[Critical]`
**Answer:** `strict: true` enables a bundle of checks: `strictNullChecks`, `noImplicitAny`, `strictFunctionTypes`, `strictBindCallApply`, `strictPropertyInitialization`, `noImplicitThis`, `useUnknownInCatchVariables` (in recent versions), and related flags. It catches null/undefined mistakes and forces explicit types where inference cannot help. Senior teams keep strict on for production services.
**Example:**
```ts
// With strictNullChecks
function getBalance(account?: Account) {
  // return account.balance; // Error: possibly undefined
  return account?.balance ?? 0;
}
```
**Follow-up:** Can you enable flags one by one? → Yes during migration, but the interview-expected answer is: turn on full `strict` for new services.

---

### Q12. What belongs in `tsconfig.json` for a Node backend? `[High]`
**Answer:** `tsconfig.json` controls compiler options, include/exclude paths, and project references. Important options: `target`, `module` / `moduleResolution`, `strict`, `outDir`, `rootDir`, `esModuleInterop`, `skipLibCheck`, `sourceMap`, and often `noUncheckedIndexedAccess`. For Node 18+/20+, modern setups use `"module": "NodeNext"` and `"moduleResolution": "NodeNext"`.
**Example:**
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "outDir": "dist",
    "rootDir": "src",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "sourceMap": true,
    "noUncheckedIndexedAccess": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```
**Follow-up:** `skipLibCheck`? → Skips type-checking `.d.ts` in node_modules for faster builds; does not weaken checks on your own source.

---

### Q13. How do you type Express Request / Response safely? `[Medium]`
**Answer:** Extend Express types via declaration merging for `req.user` after auth middleware, or wrap handlers with generic helpers. Avoid casting `req` to `any`. For bodies/query/params, validate first, then attach typed data.
**Example:**
```ts
declare global {
  namespace Express {
    interface Request {
      user?: { id: string; roles: string[] };
    }
  }
}

app.get("/me", (req, res) => {
  if (!req.user) return res.status(401).json({ error: "Unauthorized" });
  res.json({ id: req.user.id });
});
```
**Follow-up:** Why not put secrets on `req`? → Keep only identity/claims needed downstream; never attach raw tokens longer than needed.

---

### Q14. What is structural typing (duck typing) in TypeScript? `[Medium]`
**Answer:** TypeScript uses structural typing: compatibility is based on shape, not on explicit `implements`/`extends` names. If an object has the required properties with compatible types, it is assignable. This differs from nominal typing in languages like Java (class identity matters).
**Example:**
```ts
interface Point { x: number; y: number }
const p = { x: 1, y: 2, label: "A" };
const q: Point = p; // OK — extra props allowed when assigning from a variable
```
**Follow-up:** Excess property checks? → Object literals assigned directly are checked more strictly for unknown keys.

---

### Q15. How do `never` and `void` differ? `[Medium]`
**Answer:** `void` means a function does not return a useful value (often `undefined`). `never` means a value that never occurs — functions that always throw or infinite loops, or the remaining type after exhaustive narrowing. Use `never` for exhaustiveness; do not confuse it with `void`.
**Example:**
```ts
function log(msg: string): void {
  console.log(msg);
}

function assertNever(x: never): never {
  throw new Error(`Unexpected: ${JSON.stringify(x)}`);
}
```
**Follow-up:** Can `never` be assigned to every type? → Yes — `never` is the bottom type; every type is assignable from `never`, but almost nothing is assignable to `never`.
