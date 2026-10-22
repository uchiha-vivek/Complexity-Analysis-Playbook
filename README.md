# Complexity Analysis Playbook

This is the playbook i am recently using for analyzing the code e2e !

A step-by-step method for reading any function and answering: **what does it do, what is `n`, which data structures does it use, what is the time cost, what is the space cost, and what happens at scale.**

Use it in this order every time. Don't skip to the answer.

---

## Contents

1. [The 10-step method at a glance](#1-the-10-step-method-at-a-glance)
2. [Step 1 to 3: Understand the function, name `n`, list the data structures](#2-steps-1-to-3-understand-name-n-list-structures)
3. [Step 4: Count the time (rules)](#3-step-4-count-the-time)
4. [Recognising each complexity from the code shape](#4-recognising-each-complexity-from-the-code-shape)
5. [Hidden loops in JS built-ins](#5-hidden-loops-in-js-built-ins)
6. [Recursion: stack, recurrence, and how to read it](#6-recursion)
7. [Step 5: Count the space](#7-step-5-count-the-space)
8. [Step 6 to 7: Cases and amortized cost](#8-steps-6-to-7-cases-and-amortized-cost)
9. [Step 8: Trace it back (prove your answer)](#9-step-8-trace-it-back)
10. [Step 9: Measure it](#10-step-9-measure-it)
11. [Step 10: Think like a systems engineer](#11-step-10-think-like-a-systems-engineer)
12. [Worked example: three solutions to one problem](#12-worked-example)
13. [Analysis template (copy and fill)](#13-analysis-template)
14. [Cheat sheets](#14-cheat-sheets)
15. [Common mistakes](#15-common-mistakes)
16. [Production: memory structure, scale and cost](#16-production-memory-structure-scale-and-cost)
17. [Practice routine](#17-practice-routine)

---

## 1. The 10-step method at a glance

| # | Step | Question to answer |
|---|---|---|
| 1 | Purpose | What does it do, as input → output, in one sentence? |
| 2 | Inputs | What can grow? Name each one (`n`, `m`, `k`…). |
| 3 | Data structures | What is each one, and what do its operations cost? |
| 4 | Time | How many basic operations as inputs grow? |
| 5 | Space | What extra memory grows with the input? (heap + call stack) |
| 6 | Cases | Best, average, worst. Report the worst unless told otherwise. |
| 7 | Amortized | Is there an occasional expensive step that averages out? |
| 8 | Trace back | Prove it: count operations on a tiny input by hand. |
| 9 | Measure | Double `n` and watch time and memory. |
| 10 | System view | Where does it run, how big is `n` in real life, what dominates? |

Definitions used throughout:

- **Time complexity**: how the number of steps grows with input size.
- **Space complexity**: how much *extra* memory grows with input size. The input itself is not counted. This is called *auxiliary space*.
- **Big-O**: an upper bound on growth. It ignores constants and lower terms.

---

## 2. Steps 1 to 3: Understand, name `n`, list structures

### Step 1: One-sentence purpose
Write it as `input → output`. If you can't, you can't analyze it yet.

> `findDuplicates(ids)`: takes a list of patient IDs, returns the IDs that appear more than once.

### Step 2: Name every input that can grow
- One array of rows → `n`.
- Two inputs (patients and payers) → `n` and `m`. **Never merge them into one `n`.**
- A string → `n` = its length.
- A number → depends. For a loop up to the value, `n` is the value. For digit-by-digit work, `n` is the number of digits (log of the value).
- A tree → `n` = number of nodes, `h` = height.
- A graph → `V` = vertices, `E` = edges.
- A matrix → `r` rows and `c` columns.

Also ask: **is `n` capped by validation?** A note limited to 1000 characters has a bounded `n`, so its cost is bounded.

### Step 3: List the data structures and their costs
This is the knowledge that makes analysis fast. Memorise this table.

| Structure | Read by index/key | Search | Insert | Delete | Notes |
|---|---|---|---|---|---|
| Array | O(1) | O(n) | O(1) at end (`push`), O(n) at start (`unshift`) or middle | O(1) at end (`pop`), O(n) at start/middle | `shift`/`unshift`/`splice` move every later item |
| Object / `Map` | O(1) avg | O(1) avg by key | O(1) avg | O(1) avg | Hash table. Worst case O(n) on collisions, rare |
| `Set` | n/a | O(1) avg (`has`) | O(1) avg | O(1) avg | Hash table, unique values |
| Sorted array | O(1) | O(log n) (binary search) | O(n) (keep it sorted) | O(n) | |
| String | O(1) by index | O(n) (`includes`, `indexOf`) | O(n), immutable so every change copies | n/a | `a + b` copies |
| Stack (array `push`/`pop`) | O(1) top | O(n) | O(1) | O(1) | LIFO |
| Queue (array `push`/`shift`) | O(1) front | O(n) | O(1) | O(n) with `shift` | `shift` is O(n) on a big array, use an index pointer or linked list |
| Linked list | O(n) | O(n) | O(1) at known node | O(1) at known node | |
| Binary heap / priority queue | O(1) peek min | O(n) | O(log n) | O(log n) | |
| Balanced BST | O(log n) | O(log n) | O(log n) | O(log n) | |
| Trie | n/a | O(L), L = word length | O(L) | O(L) | |
| Graph (adjacency list) | O(1) neighbours | BFS/DFS O(V + E) | O(1) add edge | O(deg) | |

---

## 3. Step 4: Count the time

Go **line by line**. Give each line a cost, then combine.

### The combining rules

| Situation | Rule | Example |
|---|---|---|
| Steps one after another | **Add** | O(n) then O(m) → O(n + m) |
| Loop inside a loop | **Multiply** | O(n) × O(m) → O(n × m) |
| `if / else` branches | Take the **larger** branch | O(n) or O(1) → O(n) |
| Loop that does O(1) work | iterations × 1 | `for` over n items → O(n) |
| Loop that halves or doubles | **log** | `i *= 2` → O(log n) |
| Drop constants | `3n` → O(n) | |
| Drop lower terms | `n² + n + 20` → O(n²) | |
| Different inputs stay different | `O(n + m)`, not `O(n)` | |

### Add or multiply? A quick test
- "Do this, **then** do that" → add.
- "For each of these, do that" → multiply.

### Counting iterations of a dependent inner loop
```js
for (let i = 0; i < n; i++)
  for (let j = i + 1; j < n; j++)  // runs n-1, n-2, ... 1 times
```
Total = (n-1) + (n-2) + … + 1 = n(n-1)/2 ≈ n²/2 → **O(n²)**. The "½" is a constant, so it is dropped.

---

## 4. Recognising each complexity from the code shape

Look at the **shape of the loops and calls**, not at the number of lines.

### O(1): constant
No loop over the input. Indexing, `Map.get`, `Set.has`, `push`, `pop`, arithmetic.
```js
const first = arr[0];
const payer = payerById.get(id);
```
**Clue:** the cost does not change when the input gets bigger.

### O(log n): logarithmic
The problem **shrinks by a constant fraction each step** (usually half).
```js
// binary search on a sorted array
let lo = 0, hi = arr.length - 1;
while (lo <= hi) {
  const mid = (lo + hi) >> 1;
  if (arr[mid] === target) return mid;
  if (arr[mid] < target) lo = mid + 1;
  else hi = mid - 1;               // range halves every pass
}
```
**Clues:**
- the loop variable is multiplied or divided: `i *= 2`, `i = Math.floor(i / 2)`, `n >>= 1`
- a range is cut in half each pass (`lo/hi/mid`)
- recursion that calls itself on **one** half
- walking down a balanced tree from root to leaf

**Quick check:** how many times can you halve `n` before reaching 1? That is log₂ n. For n = 1,000,000 it is about 20.

### O(n): linear
One pass over the data, constant work per item.
```js
for (const row of rows) total += row.amount;
rows.find(...); rows.map(...); rows.filter(...); str.split("\n")
```
**Clue:** one loop (or one chain of independent loops) over the input.

### O(n log n): linearithmic
Either "n items, each doing log n work" or "divide in half and do n work to combine".
```js
arr.sort((a, b) => a - b);           // built-in sort
for (const x of items) {             // n times
  binarySearch(sorted, x);           // log n each
}
```
**Clues:**
- you see `.sort()`
- a loop of n whose body is an O(log n) operation (heap push/pop, binary search, balanced-tree op)
- merge sort or quick sort shape: split in half, recurse on both, merge in O(n)

### O(n²): quadratic
A loop over `n` containing another loop (or hidden loop) over `n`.
```js
for (const a of items)
  for (const b of items) ...         // explicit nested loop

for (const a of items)
  if (other.includes(a)) ...         // hidden loop: includes is O(n)

for (const a of items)
  const p = payers.find(...)         // hidden loop: find is O(m), so O(n × m)
```
**Clues:**
- two nested loops over the same input
- `includes`, `indexOf`, `find`, `filter`, `splice`, `slice`, or spread **inside** a loop over the same-sized data
- building a string with `+=` in a loop (each step copies)
- bubble, selection and insertion sort

### O(n³): cubic
Three nested loops, such as naive matrix multiplication or "check every triple".
```js
for (i) for (j) for (k) ...
```

### O(2ⁿ): exponential
Each step **branches into 2 (or more) calls, and the depth is about n**. Typical for "include it or skip it" choices (subsets), or naive recursion on overlapping subproblems.
```js
function fib(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);    // 2 calls, depth n, so about 2^n calls
}

function subsets(i, current) {       // every item: take it or leave it
  if (i === items.length) { out.push([...current]); return; }
  subsets(i + 1, current);           // leave
  current.push(items[i]);
  subsets(i + 1, current);           // take
  current.pop();
}
```
**Clues:**
- a recursive function that calls itself **more than once** and only reduces its input by a constant (`n - 1`, `n - 2`)
- the work *doubles* when `n` goes up by 1
- subsets, power set, "try every combination of yes/no"

**Fix:** memoization or dynamic programming often turns O(2ⁿ) into O(n). For `fib`, caching results gives O(n).

### O(n!): factorial
Generating **every ordering** (permutation) of n items. Branching shrinks by 1 each level: `n × (n-1) × (n-2) × …`.
```js
function permutations(arr, current = []) {
  if (arr.length === 0) { out.push(current); return; }
  for (let i = 0; i < arr.length; i++) {          // n choices
    const rest = [...arr.slice(0, i), ...arr.slice(i + 1)];
    permutations(rest, [...current, arr[i]]);     // then n-1, n-2, ...
  }
}
```
**Clues:**
- a loop that tries each remaining item, then recurses on the rest
- "all orderings", brute-force travelling salesman, naive scheduling
- n = 10 is 3.6 million, n = 15 is over 1 trillion, so it is only usable for tiny `n`

### Growth comparison (operations for given n)

| n | log n | n | n log n | n² | 2ⁿ | n! |
|---|---|---|---|---|---|---|
| 10 | 3 | 10 | 33 | 100 | 1,024 | 3,628,800 |
| 20 | 4 | 20 | 86 | 400 | ~1 million | ~2.4 × 10¹⁸ |
| 100 | 7 | 100 | 664 | 10,000 | ~10³⁰ | impossible |
| 1,000 | 10 | 1,000 | ~10,000 | 1,000,000 | impossible | impossible |
| 1,000,000 | 20 | 10⁶ | ~2 × 10⁷ | 10¹² | impossible | impossible |

Rule of thumb for a ~1 second budget: O(n²) is fine up to about 10⁴; O(n log n) up to about 10⁶ or 10⁷; O(2ⁿ) only up to about 25.

### Decision flowchart: "what is this loop?"

```
Does the code touch the input at all?
 ├─ No                                  → O(1)
 └─ Yes
     ├─ Is the problem cut in half each step?
     │    ├─ one branch followed        → O(log n)
     │    └─ both halves processed      → O(n log n)  (divide and conquer)
     ├─ One loop over n, O(1) body?     → O(n)
     ├─ Loop of n with an O(log n) body → O(n log n)
     ├─ Loop of n with an O(n) body     → O(n²)   (check for hidden loops!)
     ├─ Three nested loops?             → O(n³)
     ├─ Recursion calling itself 2+ times, depth ~n → O(2ⁿ) (or O(kⁿ))
     └─ Recursion that loops over remaining items → O(n!)
```

---

## 5. Hidden loops in JS built-ins

These look like one line but loop internally. Check them **first** when hunting for accidental O(n²).

| Call | Cost | Allocates new memory? |
|---|---|---|
| `arr.indexOf(x)`, `arr.includes(x)`, `arr.find(fn)`, `arr.findIndex(fn)`, `arr.some(fn)`, `arr.every(fn)` | O(n) | no |
| `arr.map(fn)`, `arr.filter(fn)` | O(n) | **yes**, new array |
| `arr.forEach(fn)`, `arr.reduce(fn)` | O(n) | no (unless the reducer builds something) |
| `arr.slice()`, `arr.concat()`, `[...arr]` | O(n) | **yes**, a copy |
| `arr.sort()` | O(n log n) | in place, but the engine may use O(n) scratch space |
| `arr.reverse()` | O(n) | in place |
| `arr.shift()`, `arr.unshift(x)` | O(n) | no (but moves every item) |
| `arr.splice(i, k)` | O(n) | no |
| `arr.push(x)`, `arr.pop()` | O(1) amortized | no |
| `arr.join(sep)` | O(total length) | **yes**, a string |
| `str.split(sep)` | O(n) | **yes** |
| `str.includes(s)`, `str.indexOf(s)` | O(n × len(s)) worst case | no |
| `str + str2`, template literal | O(length of result) | **yes** |
| `str.slice()`, `str.trim()`, `str.toLowerCase()`, `str.replace()` | O(n) | **yes** |
| `Object.keys(o)`, `Object.entries(o)`, `Object.values(o)` | O(keys) | **yes** |
| `{ ...obj }` | O(keys) | **yes** |
| `JSON.stringify(x)`, `JSON.parse(s)` | O(size) | **yes** |
| `Map.get/set/has/delete`, `Set.add/has/delete` | O(1) avg | no |
| `new Map(arr)`, `new Set(arr)` | O(n) | **yes** |
| `Array.from({ length: n })` | O(n) | **yes** |
| `Promise.all(list)` | O(n) to set up | yes |
| Regex `.test`, `.match` | usually O(n), can be exponential with bad patterns | maybe |

### The classic accidents

```js
// O(n * m): find inside a loop
for (const req of requests) {
  const payer = payers.find((p) => p.id === req.payerId);
}
// Fix: O(n + m)
const byId = new Map(payers.map((p) => [p.id, p]));
for (const req of requests) {
  const payer = byId.get(req.payerId);
}
```

```js
// O(n^2) copying: spread inside reduce
const merged = items.reduce((acc, x) => [...acc, x], []);   // copies acc every step
// Fix: O(n)
const merged = [];
for (const x of items) merged.push(x);
```

```js
// O(n^2): string += in a loop may copy repeatedly in some engines
let s = "";
for (const part of parts) s += part;
// Safer: O(n)
const s = parts.join("");
```

```js
// O(n^2): shift in a loop
while (queue.length) { const x = queue.shift(); ... }
// Fix: advance an index
for (let head = 0; head < queue.length; head++) { const x = queue[head]; ... }
```

### Chains add, they don't multiply
```js
note.split("\n").map(f).join("\n");   // O(n) + O(n) + O(n) = O(n)
```
Each step runs **after** the previous one finishes. Space is also O(n), roughly 3 copies, which Big-O writes as O(n).

---

## 6. Recursion

### 6.1 How to recognise recursion
A function that **calls itself**, directly or through another function. Always look for:

1. **Base case**: when does it stop?
2. **Recursive case**: how is the input made smaller?
3. **Branching factor**: how many times does it call itself per run?
4. **Work per call**: what else happens inside each call?

### 6.2 Time of recursion = (number of calls) × (work per call)
Draw the **call tree**. Count nodes, multiply by the work in each node.

### 6.3 Space of recursion = depth of the call stack (plus any heap you allocate)
Each active call occupies a **stack frame** (its parameters and local variables). Frames pile up until a base case returns. So:

> **Recursive stack space = maximum depth × space per frame**

Only the **deepest single path** counts, because frames on other branches have already returned.

### 6.4 Reading the stack from the code

```js
function sum(arr, i = 0) {
  if (i === arr.length) return 0;      // base case
  return arr[i] + sum(arr, i + 1);     // one call, input shrinks by 1
}
```
- Calls: n + 1 in a straight chain.
- Time: O(n).
- **Stack depth: n**, so space is **O(n)**. A loop would be O(1) space.
- Real risk: a large `n` (roughly 10,000 or more in Node) throws `RangeError: Maximum call stack size exceeded`.

Stack at `sum([1,2,3])` at its deepest:
```
sum(arr, 3)   <- base case, returns 0
sum(arr, 2)
sum(arr, 1)
sum(arr, 0)
(caller)
```

```js
function binarySearch(arr, target, lo = 0, hi = arr.length - 1) {
  if (lo > hi) return -1;
  const mid = (lo + hi) >> 1;
  if (arr[mid] === target) return mid;
  return arr[mid] < target
    ? binarySearch(arr, target, mid + 1, hi)
    : binarySearch(arr, target, lo, mid - 1);       // only ONE of these runs
}
```
- One call per level, the range halves each time.
- Time: O(log n). **Stack depth: log n**, so space is O(log n).

```js
function fib(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
}
```
- Time: about 2ⁿ calls, so **O(2ⁿ)**.
- Stack depth: the deepest path is `fib(n) → fib(n-1) → … → fib(1)`, which is n frames. Space is **O(n)**, *not* O(2ⁿ), because the two calls run one after the other, not at the same time.

```js
function mergeSort(arr) {
  if (arr.length <= 1) return arr;
  const mid = arr.length >> 1;
  const left  = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));
  return merge(left, right);            // O(n) merge
}
```
- Tree depth is log n. Each level does O(n) total work (slicing and merging). Time: **O(n log n)**.
- Stack depth: log n. But slices and merges allocate O(n) on the heap, so total space is **O(n)**.

### 6.5 Recurrence cheat sheet
Write the recursion as `T(n) = (number of calls) × T(smaller input) + (work per call)`.

| Recurrence | Pattern in code | Result |
|---|---|---|
| `T(n) = T(n-1) + O(1)` | one call on `n-1`, constant work (`sum`, linear recursion) | O(n) |
| `T(n) = T(n-1) + O(n)` | one call on `n-1`, linear work each time | O(n²) |
| `T(n) = T(n/2) + O(1)` | one call on half (binary search) | O(log n) |
| `T(n) = T(n/2) + O(n)` | one call on half, linear work | O(n) |
| `T(n) = 2T(n/2) + O(1)` | two calls on halves, constant work (tree traversal) | O(n) |
| `T(n) = 2T(n/2) + O(n)` | two calls on halves, linear merge (merge sort) | O(n log n) |
| `T(n) = 2T(n-1) + O(1)` | two calls on `n-1` (naive fib, subsets) | O(2ⁿ) |
| `T(n) = n × T(n-1)` | loop over choices, recurse on the rest (permutations) | O(n!) |

### 6.6 Tail calls and stack safety
- JavaScript engines (except Safari) do **not** optimise tail calls. Do not rely on them.
- If depth can reach thousands, convert to a loop or an explicit stack (an array you `push` and `pop`), which lives on the heap and is not limited by call-stack size.

```js
// recursive DFS: O(depth) stack, can overflow
function dfs(node) { for (const c of node.children) dfs(c); }

// iterative DFS: same time, heap-based stack, no overflow
function dfsIter(root) {
  const stack = [root];
  while (stack.length) {
    const node = stack.pop();
    for (const c of node.children) stack.push(c);
  }
}
```

### 6.7 Fixing exponential recursion with memoization
```js
function fib(n, memo = new Map()) {
  if (n <= 1) return n;
  if (memo.has(n)) return memo.get(n);        // O(1)
  const v = fib(n - 1, memo) + fib(n - 2, memo);
  memo.set(n, v);
  return v;
}
```
- Each `n` is computed once, so time is **O(n)**.
- Space is O(n) for the memo plus O(n) stack, so **O(n)**.

---

## 7. Step 5: Count the space

Ask: **"What did I allocate that grows with the input?"**

### What counts
| Counts | Does not count |
|---|---|
| New arrays, objects, Maps, Sets, strings that scale with input | The input arguments themselves |
| Copies (`[...a]`, `slice`, `concat`, `map`, `filter`, `split`, `join`) | A fixed number of variables (O(1)) |
| Recursion stack depth | The returned output, *if the question says to exclude output* |
| Memo tables and caches | |

State which convention you use. "O(1) extra, O(d) output" is a good way to be explicit.

### Space cheat sheet
| Code | Extra space |
|---|---|
| A few counters and loop variables | O(1) |
| `const out = []` that gets one item per input | O(n) |
| `new Set(ids)` | O(n) |
| `[...arr]`, `arr.slice()` | O(n) |
| `arr.map(...)`, `arr.filter(...)` | O(n) |
| `sort()` in place | O(1) to O(log n) to O(n) depending on the engine's algorithm, treat as O(n) to be safe |
| Linear recursion depth n | O(n) |
| Halving recursion | O(log n) |
| 2D grid `n × m` | O(n × m) |
| Memo table for n states | O(n) |

### Chains allocate repeatedly
```js
const quoted = note.split("\n").map(l => `> ${l}`).join("\n");
// split: O(n) array, map: another O(n) array, join: O(n) string
// total ~3n memory at peak, written as O(n)
```

### Reducing space
- Process items one at a time instead of building a full array (streaming, generators).
- Loop with an index instead of `slice`.
- Sort in place if mutating the input is acceptable.
- Reuse two variables instead of a full table (rolling DP).
- Convert deep recursion to a loop.

### The time/space trade-off
Memory is usually the thing you spend to buy speed.

| You add | You gain |
|---|---|
| A `Map`/`Set` of size n | Lookups drop from O(n) to O(1) |
| A memo table | Exponential → polynomial |
| A pre-sorted copy | Binary search at O(log n) |
| A cache | Skip repeated I/O |

---

## 8. Steps 6 to 7: Cases and amortized cost

### Best, average, worst
```js
arr.find(x => x === target)
```
- Best: target is first → O(1)
- Worst: target is last or missing → O(n)
- Report **worst case** unless asked otherwise.

Typical differences:
| Algorithm | Best | Average | Worst |
|---|---|---|---|
| Linear search | O(1) | O(n) | O(n) |
| Binary search | O(1) | O(log n) | O(log n) |
| Quick sort | O(n log n) | O(n log n) | O(n²) |
| Merge sort | O(n log n) | O(n log n) | O(n log n) |
| Hash lookup | O(1) | O(1) | O(n) (collisions) |
| Insertion sort | O(n) (already sorted) | O(n²) | O(n²) |

### Amortized cost
Some operations are occasionally expensive but cheap on average.

`arr.push(x)` is O(1) amortized. Now and then the engine resizes the backing storage (an O(n) copy), but that happens rarely enough that n pushes cost O(n) in total, which is O(1) each.

Use "amortized" only when a rare expensive step is paid for by many cheap ones. It is not a way to hide a loop.

### Early exit
`some`, `every`, `find`, and `for … break` can stop early. The worst case is still O(n), but real-world runs are often much faster.

---

## 9. Step 8: Trace it back

**Trace back** means proving your Big-O answer by hand instead of guessing. Use a tiny input and count.

### Method 1: Count operations on small `n`
Pick a *basic operation* (usually the comparison or the innermost line). Count how many times it runs for n = 1, 2, 3, 4, 5.

```js
function findDuplicates(ids) {
  const dups = [];
  for (let i = 0; i < ids.length; i++)
    for (let j = i + 1; j < ids.length; j++)
      if (ids[i] === ids[j]) dups.push(ids[i]);   // <- count this comparison
  return dups;
}
```
| n | comparisons |
|---|---|
| 2 | 1 |
| 3 | 3 |
| 4 | 6 |
| 5 | 10 |
| 10 | 45 |

Pattern: n(n-1)/2. Doubling n from 5 to 10 took it from 10 to 45 (about ×4). That confirms **O(n²)**.

### Method 2: Instrument with a counter
```js
let ops = 0;
function findDuplicates(ids) {
  const dups = [];
  for (let i = 0; i < ids.length; i++)
    for (let j = i + 1; j < ids.length; j++) {
      ops++;
      if (ids[i] === ids[j]) dups.push(ids[i]);
    }
  return dups;
}
for (const n of [10, 20, 40, 80]) {
  ops = 0;
  findDuplicates(Array.from({ length: n }, (_, i) => i));
  console.log(n, ops);   // 45, 190, 780, 3160  -> x4 each time
}
```

### Method 3: Draw the call tree (recursion)
For `fib(4)`:
```
fib(4)
├─ fib(3)
│   ├─ fib(2)
│   │   ├─ fib(1)
│   │   └─ fib(0)
│   └─ fib(1)
└─ fib(2)
    ├─ fib(1)
    └─ fib(0)
```
Count nodes (9 calls for n = 4). Notice `fib(2)` is computed twice. That repetition is why memoization helps.

To find the **stack depth**, find the longest root-to-leaf path (here `fib(4) → fib(3) → fib(2) → fib(1)`, depth 4).

### Method 4: Follow one value through the code
Pick a concrete input and write what each variable holds after each step (a trace table).

`binarySearch([1,3,5,7,9,11], 11)`

| step | lo | hi | mid | arr[mid] | action |
|---|---|---|---|---|---|
| 1 | 0 | 5 | 2 | 5 | 5 < 11, lo = 3 |
| 2 | 3 | 5 | 4 | 9 | 9 < 11, lo = 5 |
| 3 | 5 | 5 | 5 | 11 | found |

6 items took 3 steps. 12 items would take 4. Each doubling adds one step, so **O(log n)**.

### Method 5: Walk it backwards from the output
Ask: "To produce this output, what is the minimum work that *must* happen?" If you must read every input item, you can't beat O(n). If you must produce all pairs, you can't beat O(n²). If you must list all permutations, n! is unavoidable. This sets a lower bound and tells you when you're done optimising.

### Method 6: Trace a function back to its callers (systems tracing)
Complexity compounds across call boundaries.
1. Find the function's callers (search for its name, or use `graphify path "<caller>" "<function>"`).
2. Ask how many times each caller calls it.
3. Multiply.

```
for each of R requests (loop in the caller):      R times
   → getPayer(request)  which does payers.find()  O(P)
total: O(R × P)
```
A function that is O(n) alone becomes O(n²) when called inside an O(n) loop. Always trace upward.

---

## 10. Step 9: Measure it

Reasoning can be wrong. Measure to confirm.

### Doubling test
```js
function time(fn, n) {
  const ids = Array.from({ length: n }, (_, i) => i % (n / 2));
  const t = performance.now();
  fn(ids);
  return performance.now() - t;
}
for (const n of [1000, 2000, 4000, 8000, 16000]) {
  console.log(n, time(findDuplicates, n).toFixed(1) + " ms");
}
```

When `n` doubles, the time:

| Time becomes | Complexity |
|---|---|
| about the same | O(1) or O(log n) |
| ×2 | O(n) |
| a bit more than ×2 | O(n log n) |
| ×4 | O(n²) |
| ×8 | O(n³) |
| squares (or worse) | O(2ⁿ) (adding 1 to n doubles it) |

### Measuring memory
```js
global.gc?.();                                   // run node with --expose-gc
const before = process.memoryUsage().heapUsed;
const result = fn(input);
const after = process.memoryUsage().heapUsed;
console.log(((after - before) / 1024 / 1024).toFixed(2), "MB");
```
Repeat for doubling `n`. If memory doubles, space is O(n).

### Tips for honest measurements
- Warm up first (run once before timing). V8 optimises hot code.
- Run several times and take the median.
- Use inputs that hit the **worst case** (missing target, all duplicates, reverse-sorted).
- Use large enough `n` that the work dwarfs timer noise.
- `console.time("label")` / `console.timeEnd("label")` is fine for quick checks.
- Node profiler: `node --prof app.js` or `node --cpu-prof app.js` to find the real hot spot in a running service.

---

## 11. Step 10: Think like a systems engineer

Big-O answers "how does it scale?". It does not answer "what is slow today?". Ask these too.

| Question | Why it matters |
|---|---|
| **Where does it run?** Request handler, cron job, UI thread, worker? | A request handler has to answer fast. A cron job can take longer. |
| **What is the realistic `n`?** Today, next year, worst day? | O(n²) on n = 30 is harmless. On n = 30,000 it is an outage. |
| **What dominates?** CPU loop, database call, network call, disk? | One database round trip (milliseconds) costs more than 10,000 loop iterations (microseconds). Fix the dominant cost first. |
| **How many round trips?** One query, or one per row (the N+1 problem)? | N+1 is "O(n) database calls" and it is usually the real bottleneck. |
| **Does it block the event loop?** | Node runs JS on one thread. A long synchronous loop freezes every other request. |
| **Does it hold everything in memory?** | Can you stream, page, or batch instead? |
| **Is the result cacheable?** | Skipping repeated work beats optimising it. |
| **Is the input capped or validated?** | A max length turns unbounded cost into bounded cost. |
| **What breaks first as it grows?** Memory, CPU, URL length, rate limit, connection pool? | Name the first failure and design for it. |
| **Concurrency?** Multiple callers at once? Locks? Races? | Complexity per call times concurrent callers. |

> For turning these answers into memory, CPU and money on a real server, see [section 16](#16-production-memory-structure-scale-and-cost).

### N+1 example
```js
// O(n) database round trips, usually the real problem
for (const id of ids) {
  const row = await db.getRequest(id);
}

// 1 round trip (or a few batches)
const rows = await db.getRequests(ids);              // WHERE id IN (...)
```
Two practical limits to keep in mind: huge `IN (...)` lists can exceed URL or query size limits, so process in **chunks**. And large result sets should be **paged** (keyset paging stays O(page size) however large the table is).

### Latency numbers to keep in your head (rough)
| Operation | Order of magnitude |
|---|---|
| CPU operation / simple loop step | nanoseconds |
| Memory read | ~100 ns |
| Database query on same network | ~1 to 10 ms |
| API call to another service | ~10 to 500 ms |
| 1 million loop iterations in JS | ~1 to 10 ms |

So 1 database call is about as costly as hundreds of thousands of loop iterations. This is why step 10 often matters more than shaving a constant off a loop.

---

## 12. Worked example

**Problem:** given `n` patient IDs, return the IDs that appear more than once.

### Version A: nested loops
```js
function findDuplicates(ids) {
  const dups = [];
  for (let i = 0; i < ids.length; i++) {
    for (let j = i + 1; j < ids.length; j++) {
      if (ids[i] === ids[j]) dups.push(ids[i]);
    }
  }
  return dups;
}
```
1. Purpose: list of IDs → list of repeated IDs.
2. Input: `ids`, size `n`.
3. Structure: array.
4. Time: nested loops → n(n-1)/2 comparisons → **O(n²)**.
5. Space: `dups` only → **O(1)** extra (O(d) counting output).
6. Cases: always n²/2 comparisons, no early exit. (Also note: an ID appearing 3 times is pushed 3 times, so this version has a correctness quirk. Always check correctness before optimising.)
7. At scale: 1,000 IDs ≈ 500,000 steps. 100,000 IDs ≈ 5 billion steps and the event loop freezes.

### Version B: Set
```js
function findDuplicates(ids) {
  const seen = new Set();
  const dups = new Set();
  for (const id of ids) {
    if (seen.has(id)) dups.add(id);
    else seen.add(id);
  }
  return [...dups];
}
```
4. Time: one loop × O(1) operations → **O(n)**.
5. Space: `seen` up to n items → **O(n)**.
- Trade-off: spent O(n) memory to move from O(n²) to O(n).

### Version C: sort first
```js
function findDuplicates(ids) {
  const sorted = [...ids].sort();
  const dups = [];
  for (let i = 1; i < sorted.length; i++) {
    if (sorted[i] === sorted[i - 1]) dups.push(sorted[i]);
  }
  return dups;
}
```
4. Time: copy O(n) + sort O(n log n) + scan O(n) → **O(n log n)**.
5. Space: copy → **O(n)**. Sorting `ids` directly would be O(1) extra but mutates the caller's array.

| Version | Time | Space | Best when |
|---|---|---|---|
| A nested | O(n²) | O(1) | tiny inputs, memory-critical |
| B Set | O(n) | O(n) | the usual choice |
| C sort | O(n log n) | O(n) | you need sorted data anyway |

---

## 13. Analysis template

Copy this for every function you analyse.

```markdown
### Function: `name(args)`   (file:line)

**1. Purpose (input → output):**

**2. Inputs and sizes:**
- n = 
- m = 
- Capped by validation? 

**3. Data structures and their op costs:**
- 

**4. Time:**
| Line / block | Cost |
|---|---|
|  |  |
Combine (add sequential, multiply nested): 
**Time = O(   )**

**5. Space:**
- Heap allocations that grow: 
- Recursion depth: 
**Space = O(   )**

**6. Best / average / worst:**

**7. Amortized parts:**

**8. Trace-back check (small n count or call tree):**

**9. Measured (doubling test):**
| n | time | memory |
|---|---|---|

**10. System view:**
- Runs where: 
- Real n today / at scale: 
- Dominant cost: 
- Callers (multiply their loops): 
- Could it block the event loop? 
- First thing to break at scale: 

**Verdict / improvement:**
```

---

## 14. Cheat sheets

### 14.1 Pattern → complexity

| You see | Complexity |
|---|---|
| Indexing, `Map.get`, `Set.has`, `push`, `pop` | O(1) |
| `i *= 2`, `i >>= 1`, `lo/hi/mid`, one-branch halving recursion | O(log n) |
| One `for` / `for…of` / `map` / `filter` / `find` / `split` / `join` | O(n) |
| `sort()`, loop of n with O(log n) body, merge/quick sort | O(n log n) |
| Two nested loops, or `find`/`includes`/`indexOf`/`splice`/spread inside a loop | O(n²) (or O(n × m)) |
| Three nested loops | O(n³) |
| Recursion with 2+ self-calls and input shrinking by 1 | O(2ⁿ) |
| Recursion with a loop over remaining items, then recurse | O(n!) |
| Graph/tree traversal visiting each node and edge once | O(V + E) |
| Loop of n with an `await` database call inside | O(n) round trips (N+1) |

### 14.2 Sorted order of growth (best to worst)
```
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(n³) < O(2ⁿ) < O(n!)
```

### 14.3 Common algorithms

| Algorithm | Time | Space |
|---|---|---|
| Linear search | O(n) | O(1) |
| Binary search (iterative) | O(log n) | O(1) |
| Binary search (recursive) | O(log n) | O(log n) |
| Bubble / selection / insertion sort | O(n²) | O(1) |
| Merge sort | O(n log n) | O(n) |
| Quick sort | O(n log n) avg, O(n²) worst | O(log n) |
| Heap sort | O(n log n) | O(1) |
| `Array.prototype.sort` (V8, TimSort) | O(n log n) | O(n) |
| Hash map build from n items | O(n) | O(n) |
| BFS / DFS | O(V + E) | O(V) |
| Dijkstra with heap | O((V + E) log V) | O(V) |
| Naive `fib` | O(2ⁿ) | O(n) stack |
| Memoized `fib` | O(n) | O(n) |
| All subsets | O(2ⁿ × n) | O(n) stack + output |
| All permutations | O(n! × n) | O(n) stack + output |

### 14.4 Recursion quick read

| Question | How to answer |
|---|---|
| How many self-calls per run? | Count them in the body. 1 = chain, 2+ = tree |
| How does the input shrink? | `n-1` (slow shrink), `n/2` (fast shrink) |
| Time? | (number of calls in the tree) × (work per call) |
| Stack space? | Length of the longest root-to-leaf path × frame size |
| Heap space? | Anything allocated per call that survives, plus memo/output |
| Overflow risk? | Depth over a few thousand in Node |

---

## 15. Common mistakes

1. **Merging inputs into one `n`.** `O(n × m)` is not `O(n²)`. Keep them separate.
2. **Forgetting hidden loops.** `includes`, `find`, `indexOf`, `slice`, spread, `splice`, `shift`.
3. **Adding when you should multiply** (or the reverse). "Then" adds, "for each" multiplies.
4. **Counting the input as space.** Auxiliary space excludes it. State the convention.
5. **Saying O(2ⁿ) space for recursion.** Space is the depth of the stack, not the number of calls.
6. **Ignoring that `await` in a loop is serial.** n sequential round trips = n × latency. Batch or use `Promise.all` (mind concurrency limits).
7. **Optimising CPU while a database call dominates.** Measure first.
8. **Trusting the best case.** Use the worst case unless you have a reason not to.
9. **Treating `Map`/`Set` as free.** They are O(1) time but cost O(n) memory.
10. **Forgetting that string `+=`, spread in `reduce`, and `shift` loops can be quadratic.**
11. **Calling something O(1) because `n` is small today.** Say "O(n), with n ≤ 50 today".
12. **Confusing amortized with average.** Amortized is a guarantee over a sequence of operations. Average assumes random input.
13. **Skipping correctness.** A fast wrong function is worse than a slow right one.

---

## 16. Production: memory structure, scale and cost

Big-O says how work grows. This section turns it into **bytes, CPU-seconds and dollars** on a real server (Azure App Service, AWS EC2, a container).

All numbers below are **rough orders of magnitude** for reasoning. Replace them with your own measurements (Step 9) and your plan's real prices.

### 16.1 The memory structure of a Node.js process

A Node process is one OS process. What the platform measures (and kills you for) is its total memory, called **RSS** (resident set size).

```
Server / VM / App Service instance        (total RAM, e.g. 1.75 GB, 3.5 GB, 8 GB ...)
├── Operating system + platform agent      (fixed overhead, not yours)
├── Other processes / other apps on the same plan   (they share the same RAM)
└── Your Node process  (RSS = everything below)
    ├── V8 heap                 <- your objects, arrays, Maps, Sets, strings, closures
    │   ├── young generation    (short-lived objects, collected often and cheaply)
    │   └── old generation      (long-lived objects, collected rarely and expensively)
    ├── External memory         <- Buffers, file reads, HTTP bodies, TLS (outside the heap)
    ├── Call stack              <- one thread, ~1 MB default; recursion frames live here
    └── Code + runtime          <- Node binary, loaded modules, JIT-compiled code (~50 to 100+ MB baseline)
```

Key points:

| Fact | Consequence |
|---|---|
| **Heap is capped** (`--max-old-space-size`). Default is roughly 2 to 4 GB depending on Node version and machine memory. Check with `v8.getHeapStatistics().heap_size_limit`. | Going over throws `JavaScript heap out of memory` and the process crashes. |
| **The platform limit is separate and usually lower.** App Service and containers enforce the plan or container memory. | The process can be killed (OOM) before V8's own limit is reached. |
| **Memory that is still referenced is never freed.** A big array held by a cache, a global, or a closure stays. | A "leak" is usually an unbounded O(n) structure that nothing clears. |
| **Garbage collection pauses grow with heap size.** | A 2 GB heap pauses longer than a 200 MB one. This hurts latency (p99). |
| **Your O(n) space is paid per request that is running at the same time.** | Space cost multiplies by concurrency (see 16.4). |
| **Node runs your JS on one thread.** | One process uses about one CPU core for your code, however many cores the machine has. |

### 16.2 Where it runs: App Service vs EC2 (what is shared, what is limited)

| | Azure App Service | AWS EC2 (or any VM) |
|---|---|---|
| What you rent | A **plan** (instance size = fixed vCPU and RAM) | An **instance** (fixed vCPU and RAM) |
| Who shares the RAM | **Every app on the same plan** shares the plan's instances | Everything on the VM (you decide what runs) |
| What happens at the limit | The platform recycles or restarts the worker, requests fail meanwhile | The Linux OOM killer ends the biggest process, or the box starts swapping and crawls |
| CPU model | N vCPUs per instance. One Node process uses ~1, so the rest are idle unless you run more processes | Same |
| To use all cores | Scale out to more instances, or run several Node processes (cluster, PM2) | Same |
| Memory metric to watch | "Memory working set" / private bytes | Needs the CloudWatch agent (memory is not reported by default) |
| Scaling | Scale **up** (bigger plan) or **out** (more instances, memory is *per instance*) | Bigger instance type, or more instances behind a load balancer |

Two consequences:

1. **Scaling out multiplies cost but does not fix per-request memory.** If one request needs 3 GB and the instance has 1.75 GB, ten instances won't help. Fix the algorithm or the data flow.
2. **Shared plans add up.** If QA and prod (or several apps) share one plan, their memory and CPU are one budget. A heavy QA job can slow production. Treat a shared plan as one machine.

### 16.3 Estimating bytes: how big is `n` really?

Space complexity is "O(n)". Production needs **bytes per item**.

| Item (V8, 64-bit, rough) | Approx. size |
|---|---|
| Small integer in an array | 4 to 8 bytes |
| One-character-per-byte string (Latin-1) | ~1 byte per char + ~16 to 24 bytes header |
| String with non-Latin characters | ~2 bytes per char + header |
| Small object with a few fields (a parsed row) | ~50 to 150 bytes |
| Object with 10 to 20 fields (typical DB row) | ~300 to 1,000 bytes |
| `Set` / `Map` entry | ~40 to 80 bytes of overhead + the key and value themselves |
| Array slot (pointer to object) | 8 bytes (plus the object) |
| JSON text of a row | often 2 to 5× smaller than the object in memory |

Estimation formula:

```
peak memory per request ≈ n × bytes_per_item × copies_alive_at_once
```

`copies_alive_at_once` is the number of full-size structures that exist at the same moment. Count them from the code:

```js
const rows   = await db.fetchAll();            // copy 1: n rows
const mapped = rows.map(transform);            // copy 2: n rows (rows still alive)
const byId   = new Map(mapped.map(r => [r.id, r])); // copy 3: Map + temp array of pairs
const json   = JSON.stringify(mapped);         // copy 4: a string the size of everything
```
That function holds about 4 copies at its peak even though it is "just O(n)". Peak is what crashes you, not the final answer.

### 16.4 Multiplying it out: the capacity formulas

Four numbers, measured or estimated, give you everything else.

| Symbol | Meaning | How to get it |
|---|---|---|
| `λ` | Requests (or jobs) per second | Metrics, logs, or forecast |
| `C` | CPU time per request (seconds) | Measure (`--cpu-prof`, timing) or `ops × ns_per_op` |
| `M` | Peak memory per request (bytes) | Measure heap before/after, or 16.3 estimate |
| `W` | Time a request spends in the system (seconds) | Response time metric |

Formulas:

```
CPU cores needed        = λ × C ÷ target_utilisation        (use 0.6 to 0.7 for headroom)
Requests in flight      = λ × W                              (Little's law)
Memory needed           = baseline + (requests in flight × M)
Throughput of 1 process ≤ 1 ÷ C                              (one thread, if C is pure CPU)
Instances needed        = max( cores needed ÷ vCPU per instance,
                               memory needed ÷ usable RAM per instance )
Monthly cost            = instances × price per instance-hour × 730
Cost per 1,000 requests = monthly cost ÷ (monthly requests ÷ 1,000)
```

**Whichever of CPU or memory runs out first sets the instance count.** That is the real bottleneck to optimise.

### 16.5 What O(n²) time costs at scale (CPU)

Assume a conservative **10⁸ simple operations per second per core** for real JS (object access, comparisons, function calls). Raw tight loops can reach 10⁹. Always measure your own.

| n | O(n) | O(n log n) | O(n²) |
|---|---|---|---|
| 1,000 | 0.01 ms | 0.1 ms | 10 ms |
| 10,000 | 0.1 ms | 1.3 ms | **1 s** |
| 100,000 | 1 ms | 17 ms | **100 s** |
| 1,000,000 | 10 ms | 200 ms | **~10,000 s (2.8 hours)** |

Read it like this: at a million items, O(n²) is not "a bit slow", it is **hours of one core** for a single call, and O(n) is milliseconds. The ratio between them is `n` itself (a million times).

#### The event-loop effect (why one slow function hurts everyone)

Node runs JS on one thread. While an O(n²) loop runs, **no other request is served by that process**.

```
Request A starts a 2 s synchronous loop
Requests B, C, D arrive during those 2 s       -> they all wait
Every user on that process sees +2 s latency; health checks may time out
```
So the damage is not only CPU cost. It is **latency for everyone sharing the process**. Check it with `perf_hooks.monitorEventLoopDelay()`.

### 16.6 What O(n) space costs at scale (memory)

| n (rows in the structure) | bytes per item | copies | Per request | 10 requests at once |
|---|---|---|---|---|
| 10,000 | 500 B | 3 | 15 MB | 150 MB |
| 100,000 | 500 B | 3 | 150 MB | 1.5 GB |
| 1,000,000 | 500 B | 3 | **1.5 GB** | **15 GB** |

A small instance (1.75 GB or 3.5 GB RAM) can run out of memory at **a single request** with n = 1,000,000. The space grows with `n` and **again with concurrency**, so the two multiply:

```
memory = n × bytes_per_item × copies × concurrent_requests
```

### 16.7 Worked example: the same endpoint, two implementations

**Scenario (illustrative):** an endpoint receives a list of n = 10,000 items and matches each to a payer. Traffic is λ = 20 requests per second. Instance has 2 vCPU and 3.5 GB RAM. Assumed price: **$0.05 per vCPU-hour** (replace with your plan's real price).

**Version A: `payers.find()` inside the loop, plus nested comparison, O(n²)**

```
C  = n²/2 = 5 × 10⁷ ops ÷ 10⁸ ops/s = 0.5 s CPU per request
Cores needed = 20 × 0.5 ÷ 0.7 ≈ 14 cores
Per Node process: 1 ÷ 0.5 = 2 requests/s maximum
Processes needed = 20 ÷ 2 = 10 processes  ->  about 7 instances of 2 vCPU
Cost ≈ 14 vCPU × $0.05 × 730 ≈ $511 / month
Latency: queues build up; the event loop is blocked half of every second
```

**Version B: `Map` lookup, O(n)**

```
C  = 10⁴ ops ÷ 10⁸ ops/s = 0.1 ms CPU per request
Cores needed = 20 × 0.0001 ÷ 0.7 ≈ 0.003 cores  ->  one small instance is plenty
Extra memory: Map of 10⁴ entries × ~100 B ≈ 1 MB per request
Cost ≈ the minimum instance, regardless of this endpoint
Latency: milliseconds
```

| | A: O(n²) | B: O(n) |
|---|---|---|
| CPU per request | 0.5 s | 0.1 ms |
| Cores for 20 req/s | ~14 | <1 |
| Extra memory per request | ~0 | ~1 MB |
| Rough monthly CPU cost | ~$500 | the minimum instance |
| Blocks the event loop? | Yes, badly | No |

B spends ~1 MB per request to cut CPU by **5,000×**. That trade (a little memory for a lot of time) is almost always right. It stops being right when `M × concurrency` approaches the instance RAM.

### 16.8 A step-by-step cost-efficiency strategy

Do this per endpoint, job or function that matters.

1. **Inventory the hot paths.** List endpoints and background jobs, with calls per day (logs, App Insights, metrics).
2. **Find `n` for each.** Rows, items, patients, characters. Note today's value, the biggest value seen, and growth per month.
3. **Get the complexity.** Use steps 1 to 7 of this playbook, and don't forget the callers (Method 6 in section 9).
4. **Measure `C` and `M`.** CPU time and peak memory per call at realistic `n` (section 10). Use the worst realistic input.
5. **Multiply by volume.** `CPU-seconds per day = calls per day × C`. `Peak memory = in-flight × M`.
6. **Compare to capacity.** Divide by what one instance gives you (cores and RAM). Whichever limit hits first is your bottleneck.
7. **Rank by cost.** Sort endpoints by CPU-seconds per day and by peak memory. The top 1 to 3 are where optimisation pays. Ignore the rest.
8. **Pick the fix** (table below) by cheapest and safest first.
9. **Re-measure** after the fix, and record before/after.
10. **Add guards:** limits, alerts and a load test so it stays fixed.

#### Order of fixes (cheapest and biggest first)

| Order | Lever | Typical effect | Example |
|---|---|---|---|
| 1 | **Remove unneeded work** | Eliminates the cost | Don't load rows you don't show; select fewer columns |
| 2 | **Fix I/O**: batch, avoid N+1, add an index, move the join into SQL | Often 10 to 1000× | One `WHERE id IN (...)` instead of n queries |
| 3 | **Fix the algorithm / data structure** | O(n²) → O(n), `find` → `Map` | Section 5 patterns |
| 4 | **Bound the input**: validation caps, paging, chunking | Caps worst case | Note limited to 1,000 chars; page of 200 rows |
| 5 | **Stream instead of load all** | O(n) memory → O(1) or O(page) | Read and process in chunks, `for await` |
| 6 | **Cache** | Skip repeated work | Cache config loaded on every request (watch memory growth and invalidation) |
| 7 | **Move heavy work off the request path** | Protects latency and the event loop | Queue + background worker, or a separate process |
| 8 | **Scale out / up** | Buys capacity, never fixes complexity | More instances, bigger plan |

Scaling (last row) is the *most expensive* fix and the only one that does not remove the problem. Doing it first just hides an O(n²) behind a bigger bill.

### 16.9 Failure modes to recognise in production

| Symptom | Likely cause | Check |
|---|---|---|
| Memory climbs steadily and never drops | Unbounded cache, global array, listeners not removed (a leak) | Heap snapshot, compare over time |
| Memory spikes on a few requests, then falls | Large O(n) allocations (peak memory per request) | Memory graph vs request log timing |
| `JavaScript heap out of memory` | Heap limit reached | `heap_size_limit`, raise `--max-old-space-size` **only after** fixing the cause |
| Platform restarts the app, no Node error | Plan or container memory limit | Memory working set metric |
| CPU 100% on one core, others idle | One busy Node thread (O(n²) loop) | `--cpu-prof` flame graph |
| Health checks fail, then recover | Event loop blocked | `monitorEventLoopDelay` |
| p99 latency far above p50 | Large requests, GC pauses, or queued behind a blocking request | Latency percentiles by payload size |
| `Maximum call stack size exceeded` | Deep recursion (O(n) stack) | Convert to a loop (section 6.6) |
| Slow only for the biggest clients | Complexity grows with their `n` | Latency vs `n` plot |
| Everything slow when a batch job runs | Job shares a process or plan with the API | Move job to its own worker or schedule |

### 16.10 Measuring in production

| What | Tool |
|---|---|
| Heap, RSS, external memory | `process.memoryUsage()`, `v8.getHeapStatistics()` |
| Event loop lag | `perf_hooks.monitorEventLoopDelay()` |
| CPU hot spots | `node --cpu-prof`, Chrome DevTools flame graph |
| What holds the memory | Heap snapshot (`v8.writeHeapSnapshot()`) |
| Per-request timing | Structured logs with duration and `n` (log `n` next to the time, so you can plot cost vs size) |
| App Service | Platform metrics (CPU percentage, memory working set, response time, requests), Log Analytics, alerts |
| EC2 | CloudWatch CPU, network and disk; add the CloudWatch agent for memory |
| Capacity test | `autocannon`, `k6`, `wrk`: ramp up traffic and watch CPU, memory, p95 and p99 |

A cheap, powerful habit: **log `n` and duration for every heavy operation.**

```js
const t = performance.now();
const result = await heavyThing(items);
log.info("heavy_thing", { n: items.length, ms: Math.round(performance.now() - t) });
```
After a week you can plot `ms` against `n` and see the complexity curve from real production data.

#### Capacity test procedure
1. Use production-like data size (not 10 rows).
2. Start at low traffic and ramp up (for example 5 → 10 → 20 → 40 req/s).
3. At each step record CPU %, memory, p50, p95, p99, error rate.
4. The first step where p95 jumps or errors appear is your **capacity**.
5. Divide your real peak traffic by that capacity to get headroom. Keep at least 30%.

### 16.11 Capacity and cost worksheet (copy and fill)

```markdown
### Endpoint / job: `name`

**Volume**
- Calls per day:            
- Peak requests/s (λ):      
- Typical n / worst n:      
- n growth per month:       

**Per call (measured at worst realistic n)**
- Complexity time / space:  O(   ) / O(   )
- CPU time (C):             
- Peak memory (M):          
- Copies alive at peak:     
- Wall time (W):            

**Derived**
- Cores needed  = λ × C ÷ 0.7 =               
- In flight     = λ × W =                     
- Memory needed = baseline + in-flight × M =  
- Blocks event loop? how long?                

**Capacity of current instance**
- vCPU / RAM:                                 
- Limit hit first (CPU or memory):            
- Instances needed:                           

**Cost**
- Price per instance-hour:                    
- Monthly cost attributable:                  
- Cost per 1,000 requests:                    

**Plan**
- Fix (from the ordered table):               
- Expected C and M after:                     
- Re-measured:                                
- Guard added (limit / alert / test):         
```

### 16.12 Rules of thumb

1. **Cost = how often × how much each time.** Calls per day (`λ`) multiplied by CPU and memory per call (`C`, `M`).
2. **Complexity decides how the cost grows. Volume decides how many times you pay it.** A mild inefficiency at huge volume can cost more than a severe one that runs once a day.
3. **O(n²) at n = 10⁴ is already about a second of CPU. At 10⁶ it is hours.** Treat any O(n²) over user-sized data as a defect.
4. **Space is paid at the peak, and again for every request in flight.**
5. **Small `n` hides everything.** Test at the biggest realistic size, not the demo size.
6. **Fix I/O first, then algorithm, then cache, then scale.** Scaling only buys time.
7. **A bigger plan does not fix a single request that needs more RAM than the instance has.**
8. **Raising `--max-old-space-size` is a bandage, not a fix.**
9. **One blocked event loop slows every request on that process.**
10. **Log `n` next to duration.** Production will tell you the real complexity.
11. **Shared plans share failure.** One heavy job or environment can take down another.
12. **Every cap you add (max length, page size, max rows) turns an unbounded cost into a bounded one.**

---

## 17. Practice routine

1. Pick **one function a day** from the repo.
2. Fill in the [template](#13-analysis-template) without running anything.
3. Write your answers, then **trace back** with a tiny input (Step 8).
4. **Measure** with the doubling test (Step 9). Check you were right.
5. If wrong, find which hidden loop or structure cost you missed.
6. Look at its callers and ask how the cost compounds (Method 6).
7. After about 20 functions you will start spotting patterns on sight.

### Questions to ask about any function

1. What are my `n` and `m`?
2. Which data structures am I using, and what do their operations cost?
3. Is there a loop in a loop, or a hidden loop in a built-in?
4. Does it call itself? How many times, how deep?
5. What am I allocating that grows with the input?
6. What is the worst case?
7. What is the realistic size of `n` in production?
8. Is the real bottleneck I/O (database, network) rather than the CPU?
9. Who calls it, and how many times?
10. What breaks first as `n` grows?

