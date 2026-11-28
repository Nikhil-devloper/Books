# 11 — Coding Practice (JavaScript + SQL)

First technical round focus: write correct, clear solutions and explain the logic in simple English.

---

## A. JavaScript Problems

### Q1. Reverse a string `[Critical]`
**Answer:** Convert the string to an array of characters, reverse the array, then join it back. You can also loop from the end and build a new string. Ask whether Unicode/emoji support matters; the simple split approach is usually enough for interviews.
**Logic:** Last character becomes first; walk backward or use built-in reverse.
**Example:**
```js
function reverseString(str) {
  return str.split('').reverse().join('');
}

// loop version
function reverseStringLoop(str) {
  let out = '';
  for (let i = str.length - 1; i >= 0; i--) out += str[i];
  return out;
}

reverseString('hello'); // 'olleh'
```
**Follow-up:** How would you reverse words in a sentence (“hello world” → “world hello”)?

---

### Q2. Check if a string is a palindrome `[Critical]`
**Answer:** A palindrome reads the same forward and backward. Normalize the string (lowercase, remove non-alphanumeric if asked), then compare to its reverse — or use two pointers from both ends. Return true only if all mirrored pairs match.
**Logic:** Compare left and right characters until they meet in the middle.
**Example:**
```js
function isPalindrome(str) {
  const s = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  let left = 0;
  let right = s.length - 1;
  while (left < right) {
    if (s[left] !== s[right]) return false;
    left++;
    right--;
  }
  return true;
}

isPalindrome('Race car'); // true
```
**Follow-up:** What is the time and space complexity of the two-pointer approach?

---

### Q3. Find duplicates in an array `[High]`
**Answer:** Track seen values in a Set (or object). If a value is already seen, add it to a duplicates set. Using a set for duplicates avoids listing the same duplicate many times. This runs in O(n) time.
**Logic:** First time you see a value, remember it; second time, mark it as duplicate.
**Example:**
```js
function findDuplicates(arr) {
  const seen = new Set();
  const dupes = new Set();
  for (const item of arr) {
    if (seen.has(item)) dupes.add(item);
    else seen.add(item);
  }
  return [...dupes];
}

findDuplicates([1, 2, 3, 2, 4, 1]); // [2, 1]
```
**Follow-up:** How would you solve this if sorting is allowed and memory is tight?

---

### Q4. Remove duplicates from an array `[Critical]`
**Answer:** The simplest way is `Set`, which keeps unique values. For objects, you may need a key (like `id`) to decide uniqueness. Preserve order when required — `Set` iteration order matches insertion order in modern JS.
**Logic:** Keep only the first occurrence of each value.
**Example:**
```js
function removeDuplicates(arr) {
  return [...new Set(arr)];
}

function uniqueById(items) {
  const map = new Map();
  for (const item of items) {
    if (!map.has(item.id)) map.set(item.id, item);
  }
  return [...map.values()];
}

removeDuplicates([1, 2, 2, 3]); // [1, 2, 3]
```
**Follow-up:** Difference between unique primitives vs unique objects?

---

### Q5. Find the second-largest number `[High]`
**Answer:** Scan once while tracking the largest and second-largest values. Skip duplicates of the largest so “second-largest” means the next smaller distinct value (confirm with interviewer). Handle edge cases: arrays shorter than 2, or all equal numbers.
**Logic:** One pass — update max and secondMax as you go.
**Example:**
```js
function secondLargest(arr) {
  let max = -Infinity;
  let second = -Infinity;
  for (const n of arr) {
    if (n > max) {
      second = max;
      max = n;
    } else if (n < max && n > second) {
      second = n;
    }
  }
  return second === -Infinity ? null : second;
}

secondLargest([10, 5, 20, 20, 8]); // 10
```
**Follow-up:** How do you find the k-th largest efficiently?

---

### Q6. Find the missing number in 1..n `[High]`
**Answer:** If the array should contain numbers from 1 to n with one missing, use the math sum formula: expected sum is `n(n+1)/2`, subtract the actual sum. XOR of all indexes and values also works without overflow concerns in some languages. Confirm whether numbers are sorted and whether only one is missing.
**Logic:** Expected total minus actual total equals the missing number.
**Example:**
```js
function missingNumber(arr) {
  const n = arr.length + 1; // one number missing from 1..n
  const expected = (n * (n + 1)) / 2;
  const actual = arr.reduce((sum, x) => sum + x, 0);
  return expected - actual;
}

missingNumber([1, 2, 4, 5]); // 3
```
**Follow-up:** How would you find two missing numbers?

---

### Q7. Frequency counter pattern `[Critical]`
**Answer:** Count how often each value appears using an object or Map. This pattern solves anagrams, “same frequency,” and mode problems in O(n). Prefer Map when keys are not only strings. Avoid nested loops that re-count on every step.
**Logic:** One pass to build counts, then read counts to answer the question.
**Example:**
```js
function frequency(arr) {
  const freq = {};
  for (const item of arr) {
    freq[item] = (freq[item] || 0) + 1;
  }
  return freq;
}

function areAnagrams(a, b) {
  if (a.length !== b.length) return false;
  const freq = frequency(a);
  for (const ch of b) {
    if (!freq[ch]) return false;
    freq[ch]--;
  }
  return true;
}

frequency(['a', 'b', 'a']); // { a: 2, b: 1 }
```
**Follow-up:** How do you find the most frequent element from the counter?

---

### Q8. Flatten a nested array `[High]`
**Answer:** Use recursion or `Array.prototype.flat(depth)`. For a full flatten, recurse whenever an element is an array and push leaf values into a result. Interviewers often want the recursive or stack version, not only the built-in.
**Logic:** If item is array, flatten it; else keep the item.
**Example:**
```js
function flatten(arr) {
  const result = [];
  for (const item of arr) {
    if (Array.isArray(item)) result.push(...flatten(item));
    else result.push(item);
  }
  return result;
}

flatten([1, [2, [3, 4], 5]]); // [1, 2, 3, 4, 5]
// built-in: arr.flat(Infinity)
```
**Follow-up:** How would you flatten only one level deep?

---

### Q9. Find max and min in an array `[Medium]`
**Answer:** Initialize min and max to the first element, then scan the rest once. You can also use `Math.max(...arr)` for small arrays, but spreading huge arrays can throw a call-stack/argument limit error — a loop is safer.
**Logic:** Keep running min and max while iterating.
**Example:**
```js
function minMax(arr) {
  if (!arr.length) return null;
  let min = arr[0];
  let max = arr[0];
  for (let i = 1; i < arr.length; i++) {
    if (arr[i] < min) min = arr[i];
    if (arr[i] > max) max = arr[i];
  }
  return { min, max };
}

minMax([3, 1, 9, 2]); // { min: 1, max: 9 }
```
**Follow-up:** Why can `Math.max(...hugeArray)` fail?

---

### Q10. Implement debounce `[Critical]`
**Answer:** Debounce waits until calls pause for `wait` ms, then runs the function once. Useful for search-as-you-type so you do not hit the API on every keypress. Clear the previous timer on each new call and start a new one.
**Logic:** “Run only after things stay quiet for N milliseconds.”
**Example:**
```js
function debounce(fn, wait) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), wait);
  };
}

const onSearch = debounce((q) => console.log('search', q), 300);
```
**Follow-up:** What is the difference between leading-edge and trailing-edge debounce?

---

### Q11. Implement throttle `[Critical]`
**Answer:** Throttle ensures a function runs at most once per `wait` interval, even if called continuously. Useful for scroll/resize handlers. Track the last run time (or use a flag + timer) and ignore calls that arrive too soon.
**Logic:** “Run at a steady rate — at most once every N milliseconds.”
**Example:**
```js
function throttle(fn, wait) {
  let last = 0;
  return function (...args) {
    const now = Date.now();
    if (now - last >= wait) {
      last = now;
      fn.apply(this, args);
    }
  };
}

const onScroll = throttle(() => console.log('scroll'), 200);
```
**Follow-up:** When would you choose throttle over debounce?

---

### Q12. Promise delay (sleep) `[Medium]`
**Answer:** Wrap `setTimeout` in a Promise that resolves after `ms`. This is useful in async flows, retries, and demos. Combine with `await` to pause an async function without blocking the whole process the way a busy loop would.
**Logic:** After N ms, resolve the promise.
**Example:**
```js
function delay(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function demo() {
  console.log('start');
  await delay(1000);
  console.log('after 1s');
}
```
**Follow-up:** Does `delay` block the Node.js event loop?

---

### Q13. Sequential vs concurrent async work `[Critical]`
**Answer:** Sequential means `await` each task one after another — slower total time, simpler control. Concurrent means start tasks together (`Promise.all`) — faster when work is independent. Use `Promise.allSettled` when you want all results even if some fail. Do not make things concurrent if later steps need earlier results.
**Logic:** Dependent steps → sequential; independent I/O → concurrent.
**Example:**
```js
// sequential
async function sequential(ids) {
  const out = [];
  for (const id of ids) {
    out.push(await fetchUser(id));
  }
  return out;
}

// concurrent
async function concurrent(ids) {
  return Promise.all(ids.map((id) => fetchUser(id)));
}
```
**Follow-up:** What is the difference between `Promise.all` and `Promise.allSettled`?

---

### Q14. Group array of objects by a key `[High]`
**Answer:** Build an object/Map whose keys are group names and values are arrays of items. One O(n) pass is enough. This is common for reports (group transactions by status/date).
**Logic:** For each item, push it into the bucket for that key.
**Example:**
```js
function groupBy(arr, key) {
  return arr.reduce((acc, item) => {
    const k = item[key];
    (acc[k] ||= []).push(item);
    return acc;
  }, {});
}

groupBy(
  [{ type: 'debit', n: 1 }, { type: 'credit', n: 2 }, { type: 'debit', n: 3 }],
  'type'
);
// { debit: [...], credit: [...] }
```
**Follow-up:** How would you group by a derived key like month from a date?

---

### Q15. Deep clone a plain object `[Medium]`
**Answer:** For plain JSON-like data, `structuredClone` or `JSON.parse(JSON.stringify(obj))` works with limits (JSON drops functions/undefined, dates become strings). A recursive clone copies objects/arrays level by level. Prefer `structuredClone` in modern Node when available.
**Logic:** Copy each nested object/array so changing the clone does not change the original.
**Example:**
```js
function deepClone(value) {
  if (value === null || typeof value !== 'object') return value;
  if (Array.isArray(value)) return value.map(deepClone);
  const out = {};
  for (const key of Object.keys(value)) out[key] = deepClone(value[key]);
  return out;
}

// Node modern:
// const copy = structuredClone(obj);
```
**Follow-up:** Why is shallow copy (`{...obj}`) not enough for nested objects?

---

### Q16. Chunk an array into groups of size n `[Medium]`
**Answer:** Slice the array into pieces of length `n` until nothing remains. Useful for batch DB inserts or sending data in pages. Validate `n > 0`.
**Logic:** Take next n items repeatedly.
**Example:**
```js
function chunk(arr, size) {
  if (size <= 0) throw new Error('size must be > 0');
  const result = [];
  for (let i = 0; i < arr.length; i += size) {
    result.push(arr.slice(i, i + size));
  }
  return result;
}

chunk([1, 2, 3, 4, 5], 2); // [[1, 2], [3, 4], [5]]
```
**Follow-up:** How would you process each chunk concurrently with a concurrency limit?

---

### Q17. Two-sum (indices of pair that add to target) `[High]`
**Answer:** Use a Map from value → index. For each number `x`, check if `target - x` was seen. This is O(n) vs nested loops O(n²). Confirm whether the same element may be used twice.
**Logic:** Complement lookup while scanning once.
**Example:**
```js
function twoSum(nums, target) {
  const seen = new Map();
  for (let i = 0; i < nums.length; i++) {
    const need = target - nums[i];
    if (seen.has(need)) return [seen.get(need), i];
    seen.set(nums[i], i);
  }
  return null;
}

twoSum([2, 7, 11, 15], 9); // [0, 1]
```
**Follow-up:** How do you return all unique pairs, not just the first?

---

### Q18. Merge two sorted arrays `[Medium]`
**Answer:** Use two pointers, always take the smaller current head, then append leftovers. This is O(n + m) and keeps sorted order. Do not concatenate and sort unless the interviewer accepts the slower O(n log n) approach.
**Logic:** Walk both arrays like a zipper, picking the smaller each time.
**Example:**
```js
function mergeSorted(a, b) {
  const out = [];
  let i = 0;
  let j = 0;
  while (i < a.length && j < b.length) {
    if (a[i] <= b[j]) out.push(a[i++]);
    else out.push(b[j++]);
  }
  while (i < a.length) out.push(a[i++]);
  while (j < b.length) out.push(b[j++]);
  return out;
}

mergeSorted([1, 3, 5], [2, 4, 6]); // [1, 2, 3, 4, 5, 6]
```
**Follow-up:** How is this used inside merge sort?

---

## B. SQL Coding Questions

Assume simple banking-style tables when useful:

```sql
-- users(id, name, email, created_at)
-- accounts(id, user_id, balance, status)
-- transactions(id, account_id, amount, type, created_at)
-- employees(id, name, department, salary, manager_id)
```

---

### Q19. Select users created in the last 30 days `[High]`
**Answer:** Filter `created_at` with a date interval. Use parameterized dates in real apps; for interview SQL, `CURRENT_DATE - INTERVAL` is fine on PostgreSQL.
**Example:**
```sql
SELECT id, name, email, created_at
FROM users
WHERE created_at >= CURRENT_DATE - INTERVAL '30 days'
ORDER BY created_at DESC;
```
**Follow-up:** How would you write this in MySQL?

---

### Q20. Find the second-highest salary `[Critical]`
**Answer:** Classic interview query. Use `DISTINCT` with `ORDER BY ... LIMIT 1 OFFSET 1`, or a subquery with `MAX` less than the overall max. Handle ties carefully — clarify if “second highest distinct salary” is required.
**Example:**
```sql
-- distinct second highest
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;

-- subquery version
SELECT MAX(salary) AS second_highest
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```
**Follow-up:** How do you get the nth highest salary?

---

### Q21. Find duplicate emails `[High]`
**Answer:** `GROUP BY email` and keep groups with `COUNT(*) > 1`. This is the frequency-counter idea in SQL.
**Example:**
```sql
SELECT email, COUNT(*) AS cnt
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```
**Follow-up:** How do you delete duplicates and keep the lowest `id`?

---

### Q22. Join users with their accounts `[Critical]`
**Answer:** Use `INNER JOIN` when you only want users who have accounts. Use `LEFT JOIN` when you also want users with zero accounts. Always join on the foreign key (`accounts.user_id = users.id`).
**Example:**
```sql
SELECT u.id, u.name, a.id AS account_id, a.balance
FROM users u
INNER JOIN accounts a ON a.user_id = u.id
WHERE a.status = 'active';
```
**Follow-up:** When do you choose LEFT JOIN over INNER JOIN?

---

### Q23. Total transaction amount per account `[High]`
**Answer:** Aggregate with `SUM` and `GROUP BY account_id`. Filter by date range in `WHERE` before grouping. Join to accounts if you need account numbers or user names.
**Example:**
```sql
SELECT account_id, SUM(amount) AS total_amount, COUNT(*) AS txn_count
FROM transactions
WHERE created_at >= '2025-01-01'
GROUP BY account_id
ORDER BY total_amount DESC;
```
**Follow-up:** Difference between WHERE and HAVING?

---

### Q24. Accounts with balance above average `[Medium]`
**Answer:** Compare each balance to a scalar subquery that computes `AVG(balance)`. Subqueries in `WHERE` are common for this pattern.
**Example:**
```sql
SELECT id, user_id, balance
FROM accounts
WHERE balance > (SELECT AVG(balance) FROM accounts)
ORDER BY balance DESC;
```
**Follow-up:** How would you compute average per status and compare within each status?

---

### Q25. Running / latest transaction per account `[High]`
**Answer:** For “latest row per group,” use `DISTINCT ON` (PostgreSQL), window functions (`ROW_NUMBER`), or a subquery with `MAX(created_at)`. Window functions are widely accepted in interviews.
**Example:**
```sql
SELECT account_id, id, amount, created_at
FROM (
  SELECT
    t.*,
    ROW_NUMBER() OVER (PARTITION BY account_id ORDER BY created_at DESC) AS rn
  FROM transactions t
) x
WHERE rn = 1;
```
**Follow-up:** What does `PARTITION BY` do in a window function?

---

### Q26. Employees who earn more than their manager `[Medium]`
**Answer:** Self-join the employees table: alias one as employee and one as manager, match `employee.manager_id = manager.id`, then compare salaries.
**Example:**
```sql
SELECT e.name AS employee, e.salary, m.name AS manager, m.salary AS manager_salary
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```
**Follow-up:** How do you list employees with no manager (top bosses)?

---

### Q27. Pagination query `[Medium]`
**Answer:** Use `ORDER BY` with `LIMIT` and `OFFSET` for simple pages. Always order by a stable key so pages do not shuffle. For large offsets, cursor-based (`WHERE id > :lastId`) is more efficient.
**Example:**
```sql
SELECT id, amount, created_at
FROM transactions
WHERE account_id = 42
ORDER BY id
LIMIT 20 OFFSET 40;  -- page 3 if page size 20
```
**Follow-up:** Why is deep OFFSET slow?

---

### Q28. Transactions debit vs credit summary `[High]`
**Answer:** Use conditional aggregation with `SUM(CASE WHEN ...)`. This gives one row with multiple metrics without multiple scans in many engines.
**Example:**
```sql
SELECT
  account_id,
  SUM(CASE WHEN type = 'credit' THEN amount ELSE 0 END) AS total_credit,
  SUM(CASE WHEN type = 'debit' THEN amount ELSE 0 END) AS total_debit
FROM transactions
GROUP BY account_id;
```
**Follow-up:** How would you add net amount as credit − debit?

---

### Q29. Find users with no accounts `[Medium]`
**Answer:** `LEFT JOIN` accounts and keep rows where `accounts.id IS NULL`, or use `NOT EXISTS`. `NOT EXISTS` often reads clearly for anti-joins.
**Example:**
```sql
SELECT u.id, u.name
FROM users u
LEFT JOIN accounts a ON a.user_id = u.id
WHERE a.id IS NULL;

-- equivalent
SELECT u.id, u.name
FROM users u
WHERE NOT EXISTS (
  SELECT 1 FROM accounts a WHERE a.user_id = u.id
);
```
**Follow-up:** Why might `NOT IN` with nullable columns be dangerous?

---

### Q30. Safe update with balance check (conceptual) `[Critical]`
**Answer:** Update only if enough balance remains, ideally inside a transaction with a row lock. Returning the row count tells you whether the debit succeeded. This pattern prevents negative balances under concurrency better than read-then-write in app code alone.
**Example:**
```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 10 AND balance >= 100;

-- check affected rows in app; if 0 → insufficient funds

INSERT INTO transactions (account_id, amount, type)
VALUES (10, 100, 'debit');

COMMIT;
```
**Follow-up:** What is optimistic vs pessimistic locking at a high level?

---

## Quick Patterns to Memorize

| Pattern | JS idea | SQL idea |
|---------|---------|----------|
| Count frequency | Map / object counter | `GROUP BY` + `HAVING` |
| Unique values | `Set` | `DISTINCT` / `GROUP BY` |
| Second best | track max + second | `MAX` subquery / `OFFSET 1` |
| Pair search | complement Map | self-join |
| Latest per group | sort/filter in JS | `ROW_NUMBER` window |
| Wait for quiet | debounce | — |
| Cap call rate | throttle | — |
| Parallel I/O | `Promise.all` | — |

## Interview Tip (30-minute round)

1. Restate the problem and confirm edge cases in one sentence.
2. Say the approach before coding (brute force → better).
3. Write working code first; then mention complexity.
4. Test with one normal case and one edge case out loud.
