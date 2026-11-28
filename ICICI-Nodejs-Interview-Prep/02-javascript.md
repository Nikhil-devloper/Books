# JavaScript Interview Prep — ICICI Node.js Backend (First Round, 30 min)

**Priority note:** Expect closures, `this`/call-apply-bind, Promises/async-await, and 1–2 output questions. var/let/const and array methods are quick filters; Promise combinators and shallow/deep copy show senior depth. Marked `[Critical]` items are highest likelihood in 30 minutes.

---

### Q1. Difference between `var`, `let`, and `const`? `[Critical]`

| | **var** | **let** | **const** |
|---|--------|---------|-----------|
| **Scope** | Function | Block | Block |
| **Hoisting** | Yes (init `undefined`) | Yes (TDZ) | Yes (TDZ) |
| **Reassign** | Yes | Yes | No |
| **Redeclare** | Yes (same scope) | No | No |
| **Global** | Creates `window` prop (browser) | No | No |

**Answer:** Prefer `const` by default, `let` when reassignment is needed, avoid `var`. `let`/`const` are block-scoped and exist in the **Temporal Dead Zone (TDZ)** from block start until initialization — accessing them early throws `ReferenceError`.
**Example:**
```js
console.log(a); // undefined (var hoisted)
var a = 1;

console.log(b); // ReferenceError (TDZ)
let b = 2;

const c = { x: 1 };
c.x = 2;        // OK — binding is const, object is mutable
// c = {};      // TypeError
```
**Follow-up:** Are `const` objects immutable? → No. Only the binding is fixed; freeze with `Object.freeze` (shallow).

---

### Q2. What is hoisting and the Temporal Dead Zone? `[Critical]`
**Answer:** Hoisting means declarations are processed before execution. `var`/`function` declarations are initialized early (`var` → `undefined`; function declarations fully available). `let`/`const` are hoisted but uninitialized until the declaration line — that gap is the **TDZ**.
**Example:**
```js
foo(); // works — function declaration hoisted
function foo() { return 1; }

bar(); // TypeError — undefined is not a function
var bar = function () { return 2; };
```
**Follow-up:** Class declarations? → Also TDZ like `let`; cannot use before initialization.

---

### Q3. What is a closure? Give a practical use. `[Critical]`
**Answer:** A **closure** is a function that remembers variables from its **lexical scope** even after the outer function has returned. Used for data privacy, partial application, module patterns, and callbacks that retain context (e.g., request handlers capturing config).
**Example:**
```js
function makeCounter() {
  let count = 0;
  return () => ++count;
}
const c = makeCounter();
c(); // 1
c(); // 2 — `count` is private via closure
```
**Follow-up:** Memory concern? → Closures keep referenced outer variables alive; don’t close over large unused objects.

---

### Q4. Explain scope in JavaScript. `[High]`
**Answer:** JS has **global**, **function**, **block** (`let`/`const`/`class`), and **module** scope. Lookup follows the **scope chain** outward. There is no block scope for `var`. Shadowing occurs when an inner binding reuses an outer name.
**Example:**
```js
const x = 'global';
function f() {
  const x = 'function';
  if (true) {
    const x = 'block';
    console.log(x); // block
  }
  console.log(x);   // function
}
```
**Follow-up:** Lexical vs dynamic scope? → JS is lexical: where the function is **written** decides free-variable lookup, not where it is called (`this` is the separate binding rule).

---

### Q5. How does `this` work? `[Critical]`
**Answer:** `this` depends on **how a function is called**, not where it is defined (except arrows). Rules: default (undefined in strict / global in sloppy), implicit (obj.method), explicit (`call`/`apply`/`bind`), `new` (new instance). Arrow functions inherit `this` from the enclosing lexical scope.
**Example:**
```js
const obj = {
  n: 1,
  regular() { return this.n; },
  arrow: () => this.n, // lexical this (not obj)
};
obj.regular(); // 1
const f = obj.regular;
f(); // undefined (strict) — lost receiver
```
**Follow-up:** Why does `this` break in callbacks? → Passing `obj.method` detaches it; fix with `bind`, arrow wrapper, or call as method.

---

### Q6. `call` vs `apply` vs `bind`? `[Critical]`

| | **call** | **apply** | **bind** |
|---|----------|-----------|----------|
| **Sets `this`** | Yes | Yes | Yes |
| **Args** | Comma list | Array | Pre-set (partial) |
| **Runs** | Immediately | Immediately | Returns new function |

**Answer:** `call`/`apply` invoke now with a given `this`. `bind` returns a permanent bound function (useful for event handlers). `apply` is handy when args are already an array (spread + `call` is modern equivalent).
**Example:**
```js
function greet(g, p) { return `${g} ${this.name}${p}`; }
const user = { name: 'Ada' };

greet.call(user, 'Hi', '!');          // Hi Ada!
greet.apply(user, ['Hi', '!']);       // Hi Ada!
const bound = greet.bind(user, 'Hi'); // bound('!') => Hi Ada!
```
**Follow-up:** Can you re-bind a bound function? → Further `bind` does not change the already-bound `this`.

---

### Q7. Arrow functions vs regular functions? `[High]`
**Answer:** Arrows have **lexical `this`**, no own `arguments`, cannot be used with `new`, and have no `prototype`. Regular functions get dynamic `this` and are constructors when called with `new`. Use arrows for callbacks; use regular methods when you need dynamic `this`.
**Example:**
```js
class Timer {
  delay = 100;
  start() {
    setTimeout(() => console.log(this.delay), 0); // arrow keeps class this
  }
}
```
**Follow-up:** Arrow as object method? → Usually wrong — `this` won’t be the object when called as `obj.method()`.

---

### Q8. Prototypes and inheritance in JS? `[High]`
**Answer:** Every object has an internal `[[Prototype]]` link (`__proto__` / `Object.getPrototypeOf`). Property lookup walks the **prototype chain**. `class` syntax is sugar over constructor functions + `prototype`. Prefer composition; use `Object.create` or `class extends` when inheritance is needed.
**Example:**
```js
function Animal(name) { this.name = name; }
Animal.prototype.speak = function () { return this.name; };

class Dog extends Animal {
  speak() { return `${super.speak()} woof`; }
}
new Dog('Rex').speak(); // Rex woof
```
**Follow-up:** Own vs inherited property? → `hasOwnProperty` / `Object.hasOwn` vs `in` operator (includes prototype chain).

---

### Q9. Explain Promises and states. `[Critical]`
**Answer:** A Promise is a placeholder for a future value with states: **pending**, **fulfilled**, **rejected**. Settling is one-way. `.then` returns a new Promise (chainable). Unhandled rejections are serious in Node (can crash depending on version/flags). Prefer `async/await` for readability; still know `.then` for composition.
**Example:**
```js
const p = new Promise((resolve, reject) => {
  fs.readFile('f.txt', (e, d) => (e ? reject(e) : resolve(d)));
});
p.then(buf => buf.toString()).catch(err => console.error(err));
```
**Follow-up:** Promise vs callback hell? → Promises flatten chains and centralize errors via `.catch` / try-catch with await.

---

### Q10. How does `async`/`await` work? `[Critical]`
**Answer:** `async` functions always return a Promise. `await` pauses the **async function** (not the whole thread) until the Promise settles, then continues as a microtask. Errors become rejected Promises; use `try/catch`. Parallelism needs starting multiple Promises before awaiting them.
**Example:**
```js
async function load() {
  // sequential — slower
  const a = await fetchA();
  const b = await fetchB();

  // parallel — better
  const [c, d] = await Promise.all([fetchA(), fetchB()]);
}
```
**Follow-up:** Does await block the event loop? → No. Only the async function’s continuation waits; other events still run.

---

### Q11. `Promise.all` vs `allSettled` vs `race` vs `any`? `[Critical]`

| Method | Resolves when | Rejects when | Result |
|--------|---------------|--------------|--------|
| **`Promise.all`** | All fulfill | First rejection | Array of values (order preserved) |
| **`Promise.allSettled`** | All settle | Never (for input rejections) | `{status, value\|reason}[]` |
| **`Promise.race`** | First settles | First rejection (if it rejects first) | That value/reason |
| **`Promise.any`** | First fulfillment | **All** reject → `AggregateError` | First fulfilled value |

**Answer:** Use `all` when every result is required; `allSettled` when you need every outcome; `race` for timeouts/first done; `any` for first success among mirrors.
**Example:**
```js
await Promise.all([p1, p2]); // fail-fast
await Promise.allSettled([p1, p2]); // always get statuses

const withTimeout = Promise.race([
  fetch(url),
  new Promise((_, rej) => setTimeout(() => rej(new Error('timeout')), 5000)),
]);
```
**Follow-up:** Does `all` cancel the others on reject? → No — other Promises still run; you only stop waiting. Need AbortController for true cancel.

---

### Q12. Event loop output (JS-focused) — what prints? `[Critical]`
```js
console.log('start');

setTimeout(() => console.log('timeout'), 0);

Promise.resolve()
  .then(() => {
    console.log('then1');
    return Promise.resolve();
  })
  .then(() => console.log('then2'));

console.log('end');
```
**Answer:** Output: **`start` → `end` → `then1` → `then2` → `timeout`**. Sync first; Promise microtasks before macrotask (`setTimeout`).
**Example:** Returning a Promise inside `then` chains another microtask before later handlers.
**Follow-up:** Where would `queueMicrotask` fit? → Same queue as Promise jobs (roughly), after sync, before timers.

---

### Q13. What is the output? `[Critical]`
```js
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}
async function async2() {
  console.log('async2');
}
console.log('script start');
async1();
console.log('script end');
```
**Answer:** Output: **`script start` → `async1 start` → `async2` → `script end` → `async1 end`**. `await` yields after `async2`’s sync body; the rest of `async1` runs as a microtask after current sync code.
**Example:** Treat “code after await” like a `.then` continuation.
**Follow-up:** If `async2` returned a thenable that resolves later? → `async1 end` waits until that Promise fulfills.

---

### Q14. What is the output? `[High]`
```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 0);
}
```
**Answer:** First loop prints **`3 3 3`** (`var` is function-scoped — one shared `i`). Second prints **`0 1 2`** (`let` creates a new binding per iteration).
**Example:** Fix with `var` using an IIFE: `(function (i) { setTimeout(() => console.log(i), 0); })(i)`.
**Follow-up:** Same with `Promise.then` instead of setTimeout? → Same scoping result; only the queue differs (micro vs macro).

---

### Q15. `map` vs `filter` vs `reduce` vs `forEach`? `[High]`

| Method | Returns | Purpose | Mutates array? |
|--------|---------|---------|----------------|
| **`map`** | New array (same length) | Transform each element | No |
| **`filter`** | New array (≤ length) | Keep elements matching predicate | No |
| **`reduce`** | Single accumulated value | Fold to any result | No (unless you mutate in reducer) |
| **`forEach`** | `undefined` | Side effects only | No (callback may mutate) |

**Answer:** Prefer `map`/`filter`/`reduce` for declarative transforms; `forEach` when you only need side effects (and don’t need break — use `for...of` to exit early). Avoid async `forEach` — it won’t await.
**Example:**
```js
const nums = [1, 2, 3, 4];
nums.map(n => n * 2);             // [2,4,6,8]
nums.filter(n => n % 2 === 0);    // [2,4]
nums.reduce((a, n) => a + n, 0);  // 10
nums.forEach(n => console.log(n));
```
**Follow-up:** Why is async forEach a bug? → It fires Promises without waiting; use `for...of` + `await` or `Promise.all(arr.map(async ...))`.

---

### Q16. Shallow copy vs deep copy? `[High]`

| | **Shallow** | **Deep** |
|---|-------------|----------|
| **Copies** | Top-level only | Nested objects recursively |
| **Nested refs** | Shared with original | Independent (ideally) |
| **Common APIs** | spread, `Object.assign`, `Array.from` | `structuredClone`, careful recursion, libs |

**Answer:** Shallow copy duplicates only the first level — nested objects still share references. Deep copy clones nested structures. `JSON.parse(JSON.stringify(x))` fails on functions, `undefined`, Dates, Maps, circular refs. Prefer `structuredClone` in modern Node.
**Example:**
```js
const a = { user: { name: 'A' } };
const shallow = { ...a };
shallow.user.name = 'B'; // also changes a.user.name

const deep = structuredClone(a);
deep.user.name = 'C';     // a unchanged
```
**Follow-up:** Does spread deep-copy arrays of objects? → No — only the array shell; elements are the same references.

---

### Q17. Spread vs rest operators? `[Medium]`
**Answer:** **Spread** expands iterables/objects into elements/properties. **Rest** collects remaining elements/params into an array/object. Same `...` syntax; role depends on position (call/literal vs binding).
**Example:**
```js
const arr = [1, 2, 3];
Math.max(...arr);              // spread

function f(first, ...rest) {   // rest params
  return rest;
}
f(1, 2, 3); // [2, 3]

const { a, ...others } = { a: 1, b: 2, c: 3 }; // others = {b:2,c:3}
```
**Follow-up:** Spread on objects — own enumerable only? → Yes; does not copy prototype or non-enumerable props.

---

### Q18. Debounce vs throttle? `[High]`
**Answer:** **Debounce** delays running until quiet for N ms (search box). **Throttle** runs at most once per N ms (scroll/resize handlers). Both reduce call frequency; choose based on whether you need the trailing quiet call or steady sampling.
**Example:**
```js
function debounce(fn, ms) {
  let t;
  return (...args) => {
    clearTimeout(t);
    t = setTimeout(() => fn(...args), ms);
  };
}

function throttle(fn, ms) {
  let last = 0;
  return (...args) => {
    const now = Date.now();
    if (now - last >= ms) {
      last = now;
      fn(...args);
    }
  };
}
```
**Follow-up:** Leading vs trailing debounce? → Leading fires immediately then ignores; trailing waits for silence (common for search).

---

### Q19. Optional chaining `?.` and nullish coalescing `??`? `[Medium]`
**Answer:** `?.` short-circuits and returns `undefined` if the base is `null`/`undefined` — safe deep access. `??` returns the right side only when the left is `null` or `undefined` (not for `0`/`''`/`false`). Don’t confuse `??` with `||`.
**Example:**
```js
const city = user?.address?.city; // undefined if any link missing

const port = config.port ?? 3000; // 0 is kept
const bad  = config.port || 3000; // 0 becomes 3000 — often wrong
```
**Follow-up:** `?.()` and `?.[]`? → Yes — optional call and optional element access.

---

### Q20. What is the output? (coding concept) `[High]`
```js
const obj = {
  a: 1,
  b: function () { return this.a; },
  c: () => this.a,
};
console.log(obj.b());
console.log(obj.c());
```
**Answer:** `obj.b()` → **`1`** (method call sets `this` to `obj`). `obj.c()` → **`undefined`** in modules/strict Node (arrow’s lexical `this` is not `obj`, typically module/`undefined`).
**Example:** Never use arrows for methods that need the receiver object as `this`.
**Follow-up:** What if you `const { b } = obj; b()`? → `undefined` — destructuring loses the receiver.

---

### Q21. Equality: `==` vs `===`, and `Object.is`? `[Medium]`
**Answer:** `===` is strict (no coercion). `==` coerces types (avoid in most code). `Object.is` is like `===` but distinguishes `+0`/`-0` and treats `NaN` as equal to `NaN`.
**Example:**
```js
0 == false;     // true (coercion)
0 === false;    // false
NaN === NaN;    // false
Object.is(NaN, NaN); // true
```
**Follow-up:** Why avoid `==`? → Hidden coercions (`'' == 0`, `null == undefined`) cause subtle bugs.

---

### Q22. Mutable vs immutable patterns — why do interviews care? `[Medium]`
**Answer:** Accidental shared mutation via shallow copies causes hard bugs in concurrent async flows. Prefer copying before change, or immutable updates (`map` instead of mutating arrays). In Node services, shared in-memory objects across requests are a classic footgun.
**Example:**
```js
// Bad: mutate shared defaults
function addItem(list = []) {
  list.push('x');
  return list;
}

// Better
function addItem(list = []) {
  return [...list, 'x'];
}
```
**Follow-up:** Default params evaluated once? → Default value expressions run **per call when missing** — but the classic bug is mutating a shared object/array default still returned/held elsewhere.

---

### Q23. What is the output? (closure + loop classic) `[Critical]`
```js
function createFuncs() {
  const fns = [];
  for (var i = 0; i < 3; i++) {
    fns.push(() => i);
  }
  return fns;
}
console.log(createFuncs().map(f => f()));
```
**Answer:** Output: **`[3, 3, 3]`**. All closures share one `var i`, which is `3` after the loop. Fix with `let i` or bind the value per iteration.
**Example:**
```js
for (let i = 0; i < 3; i++) fns.push(() => i); // [0,1,2]
```
**Follow-up:** Interviewers may ask you to fix it without `let` — use IIFE or `fns.push(((i) => () => i)(i))`.

---

### Q24. Generators / iterators — quick senior check? `[Medium]`
**Answer:** Generators (`function*`) pause with `yield` and return an iterator. Useful for lazy sequences, custom iterables, and some async flows (`async function*`). Most backend code uses async/await more than generators, but knowing `Symbol.iterator` shows depth.
**Example:**
```js
function* range(n) {
  for (let i = 0; i < n; i++) yield i;
}
[...range(3)]; // [0,1,2]
```
**Follow-up:** Can you `await` inside a generator? → Use `async function*` and `for await...of`.

---

### Q25. Common coding pitfalls in Node/JS interviews? `[High]`
**Answer:** (1) Floating promises without await/catch. (2) Mutating shared objects. (3) `typeof null === 'object'`. (4) Sorting numbers without comparator (`[10,2].sort()` → lexicographic). (5) Confusing `slice`/`splice`. (6) JSON date round-trip becomes string.
**Example:**
```js
[10, 2, 1].sort();           // [1, 10, 2] — wrong for numbers
[10, 2, 1].sort((a, b) => a - b); // [1, 2, 10]

typeof null; // 'object' — language quirk
Array.isArray([]); // true — correct array check
```
**Follow-up:** How do you deep-freeze? → Recursive `Object.freeze` or use a library; built-in freeze is shallow.

---

## Quick revision checklist (30-min round)

1. var/let/const + TDZ one-liner + loop closure output  
2. Explain closure with a counter or private state  
3. `this` rules + call/apply/bind table  
4. Promise combinators table — pick the right one for a scenario  
5. One async/await parallel vs sequential example  
6. Shallow vs deep copy + `structuredClone`  
7. Solve 2 JS event-loop / async output questions aloud  

---

*Priorities: `[Critical]` = very likely · `[High]` = common · `[Medium]` = if time remains*
`)
