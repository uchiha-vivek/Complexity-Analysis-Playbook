# Math and Logic for Engineers

The mathematics and logic behind complexity analysis, written for JavaScript / TypeScript work. It supports [complexity-analysis-playbook.md](complexity-analysis-playbook.md).

Each topic has: the idea, the formulas, a tiny example, and where you'll meet it.

## Contents

- [Tier 1: needed to follow the playbook](#tier-1-needed-to-follow-the-playbook)
  1. Exponents and logarithms
  2. Summations
  3. Big-O, Ω, Θ
  4. Growth-rate ordering
  5. Factorials and combinatorics
  6. Boolean logic
  7. Negation and De Morgan's laws
  8. Sets
- [Tier 2: going deeper](#tier-2-going-deeper)
  9. Recurrences and the Master Theorem
  10. Amortized analysis
  11. Relations and functions
  12. Induction and invariants
  13. Probability basics
  14. Graph basics
  15. Modular arithmetic
  16. Binary and bit operations
- [Tier 3: production and cost](#tier-3-production-and-cost)
- [Tier 4: reasoning habits](#tier-4-reasoning-habits)
- [Worked example: the math of one function](#worked-example-the-math-of-one-function)
- [One-page cheat sheet](#one-page-cheat-sheet)
- [Self-test](#self-test)

---

# Tier 1: needed to follow the playbook

## 1. Exponents and logarithms

**Idea.** An exponent says "multiply by itself": `2ⁿ` doubles n times. A logarithm is the reverse: `log₂ n` answers **"how many times can I halve n before I reach 1?"**

| Formula | Meaning |
|---|---|
| `2³ = 8` ⇔ `log₂ 8 = 3` | log undoes the exponent |
| `aˣ · aʸ = aˣ⁺ʸ` | exponents add when you multiply |
| `log(a·b) = log a + log b` | logs turn products into sums |
| `log(aᵏ) = k · log a` | exponents come out front |
| `log_b n = log n / log b` | any base converts to any other by a constant |
| `2¹⁰ = 1,024 ≈ 10³` | useful for quick estimates |

Because the base only changes a constant factor, **Big-O ignores it**: O(log₂ n) = O(log₁₀ n) = O(ln n).

```js
Math.log2(1_000_000);  // ≈ 19.9   a million items need ~20 halvings
Math.log2(1_000_000_000); // ≈ 29.9   a billion needs only ~30
```

**You'll meet it:** binary search, balanced tree depth, heaps, `i *= 2` loops, "why 1,000,000 → 20".

## 2. Summations

**Idea.** `Σ` means "add up a pattern". Nested loops produce sums, so you need a few standard ones.

| Sum | Result | Shows up as |
|---|---|---|
| `1 + 2 + … + n` | `n(n+1)/2` ≈ n²/2 | dependent inner loop (`j = i + 1`) |
| `n + n + … + n` (n times) | `n²` | plain nested loops |
| `1 + 2 + 4 + … + 2ᵏ` | `2ᵏ⁺¹ − 1` | doubling arrays, binary-tree levels |
| `1 + 1/2 + 1/3 + … + 1/n` | ≈ `ln n` | some hashing and sorting averages |
| `c + c + … + c` (n times) | `c·n` | a constant-work loop |

```js
// How many times does the body run?
for (let i = 0; i < n; i++)
  for (let j = i + 1; j < n; j++) { /* body */ }
// (n-1) + (n-2) + ... + 1 = n(n-1)/2   ->  O(n²)
```

**Quick derivation of 1 + 2 + … + n** (Gauss): write the sum forwards and backwards; each column adds to `n + 1`; there are `n` columns; the total counted twice is `n(n+1)`, so the sum is `n(n+1)/2`.

## 3. Big-O, Ω, Θ

**Formal definitions** (for `f(n)` = your function's real step count, `g(n)` = the simple shape):

| Notation | Meaning | Plain words |
|---|---|---|
| `f = O(g)` | there exist `c > 0` and `n₀` so that `f(n) ≤ c·g(n)` for all `n ≥ n₀` | **at most** this fast-growing |
| `f = Ω(g)` | there exist `c > 0` and `n₀` so that `f(n) ≥ c·g(n)` for all `n ≥ n₀` | **at least** this fast-growing |
| `f = Θ(g)` | both `O(g)` and `Ω(g)` | **exactly** this growth (tight) |

Example: `f(n) = 3n² + 5n + 20`.
- Take `c = 4`, `n₀ = 8`: `3n² + 5n + 20 ≤ 4n²` once `n² ≥ 5n + 20`, true from n = 8 (64 ≥ 60). So `f = O(n²)`.
- Also `f ≥ 3n²`, so `f = Ω(n²)`, and therefore `f = Θ(n²)`.

**Rules that follow from the definition:**
- Drop constants: `O(5n) = O(n)`.
- Drop lower terms: `O(n² + n) = O(n²)`.
- Sequential adds: `O(f) + O(g) = O(max(f, g))` (with different inputs, keep both: `O(n + m)`).
- Nested multiplies: `O(f) × O(g)`.
- Big-O is an **upper** bound, so saying "binary search is O(n)" is technically true but useless. Prefer the tightest bound (Θ).

**Careful:** "worst case" and "Big-O" are different ideas. Worst/best/average choose **which input** you analyse. O/Ω/Θ choose **which kind of bound** you put on it.

## 4. Growth-rate ordering

```
O(1) < O(log n) < O(√n) < O(n) < O(n log n) < O(n²) < O(n³) < O(2ⁿ) < O(n!) < O(nⁿ)
```

- **Polynomial** (`nᵏ`) grows slower than **exponential** (`2ⁿ`) for large n. This is the dividing line between "scales" and "does not scale".
- Any power of `log n` is smaller than any positive power of `n`: `(log n)⁵ < n^0.1` eventually.
- Order matters only for large n. At n = 10, `2ⁿ = 1,024` is bigger than `n² = 100`, but `n²` at n = 1,000 is 1,000,000 while `2ⁿ` is astronomically larger.

## 5. Factorials and combinatorics

| Idea | Formula | Example |
|---|---|---|
| Factorial | `n! = n·(n−1)·…·1` | 5! = 120 |
| Orderings (permutations) of n items | `n!` | 10 items → 3,628,800 |
| Subsets of n items | `2ⁿ` | each item in or out |
| Choose k of n (order ignored) | `C(n,k) = n! / (k!·(n−k)!)` | C(5,2) = 10 |
| **Pairs** from n items | `C(n,2) = n(n−1)/2` | the nested-loop count |
| Ordered k of n | `n!/(n−k)!` | picking a first, second, third |

The link to code: **a loop over all pairs does exactly `C(n,2)` iterations**, which is why it is O(n²). A recursion that tries every subset is O(2ⁿ). One that tries every ordering is O(n!).

Counting rules:
- **Multiplication principle:** if step 1 has `a` options and step 2 has `b`, together there are `a × b`.
- **Addition principle:** if the choices are mutually exclusive, add them.

## 6. Boolean logic

Two values (`true`, `false`) and a few operators.

| A | B | A && B (AND) | A \|\| B (OR) | A ^ B (XOR) | !A (NOT) |
|---|---|---|---|---|---|
| T | T | T | T | F | F |
| T | F | F | T | T | F |
| F | T | F | T | T | T |
| F | F | F | F | F | T |

**Short-circuit evaluation in JS** (it matters for cost and for side effects):
- `A && B`: if `A` is falsy, returns `A` and **never evaluates B**.
- `A || B`: if `A` is truthy, returns `A` and **never evaluates B**.
- `A ?? B`: uses `B` only when `A` is `null` or `undefined` (not for `0`, `""`, `false`).

```js
if (user && user.isAdmin && expensiveCheck(user)) { ... }
// Put the cheap, most-likely-to-fail test first: expensiveCheck usually never runs.
```

**Truthy / falsy gotcha.** Falsy values are `false, 0, -0, 0n, "", null, undefined, NaN`. Everything else is truthy, including `"0"`, `[]`, `{}`.

**Useful identities:**
| Identity | Name |
|---|---|
| `A && true = A`, `A || false = A` | identity |
| `A && false = false`, `A || true = true` | domination |
| `A && A = A`, `A || A = A` | idempotent |
| `A && B = B && A` | commutative (but not for side effects!) |
| `A && (B || C) = (A && B) || (A && C)` | distributive |
| `!!A = A` (as a boolean) | double negation |

**XOR** is "exactly one": `A ^ B` is true when they differ. Handy for "toggle" and "changed?" checks.

## 7. Negation and De Morgan's laws

```
!(A && B)  ==  !A || !B
!(A || B)  ==  !A && !B
```

In words: **to negate an AND, flip each part and change AND to OR** (and the reverse).

```js
// "Skip unless the request is open AND has a payer"
if (!(req.open && req.payerId)) return;
// is the same as
if (!req.open || !req.payerId) return;
```

**Negating comparisons:**
| Condition | Its negation |
|---|---|
| `a < b` | `a >= b` |
| `a === b` | `a !== b` |
| `x == null` | `x != null` |
| `a < b && b < c` | `a >= b \|\| b >= c` |

**Negating quantifiers** (this is exactly how `some` and `every` relate):
```
!(every x: P(x))   ==  some x: !P(x)
!(some x: P(x))    ==  every x: !P(x)
```
```js
!items.every(isValid)  ===  items.some(i => !isValid(i));
```

**Vacuous truth gotcha:**
```js
[].every(x => false);  // true   "all zero items satisfy it"
[].some(x => true);    // false  "there is no item that satisfies it"
```
An empty list passes `every` automatically. This can silently let bad data through if you forget to check for emptiness.

**Tips for readable conditions:**
- Prefer positive wording and early returns (guard clauses).
- When a condition has two or more `!`, apply De Morgan and see whether it reads better.
- Test the four corners of a two-variable condition (TT, TF, FT, FF).

## 8. Sets

**Idea.** A set is an unordered collection of **distinct** items. `|S|` is its size (cardinality).

| Operation | Symbol | Meaning | Complexity with JS `Set` |
|---|---|---|---|
| Membership | `x ∈ S` | is x in S | O(1) avg (`has`) |
| Union | `A ∪ B` | in A or B (or both) | O(\|A\| + \|B\|) |
| Intersection | `A ∩ B` | in both | O(min(\|A\|, \|B\|)) |
| Difference | `A \ B` | in A but not in B | O(\|A\|) |
| Symmetric difference | `A Δ B` | in exactly one | O(\|A\| + \|B\|) |
| Subset | `A ⊆ B` | every item of A is in B | O(\|A\|) |
| Empty set | `∅` | no items | |

```js
const union        = new Set([...A, ...B]);
const intersection = new Set([...A].filter(x => B.has(x)));
const difference   = new Set([...A].filter(x => !B.has(x)));
const isSubset     = [...A].every(x => B.has(x));
```

**Set laws (they mirror Boolean logic):**
| Set law | Boolean twin |
|---|---|
| `A ∩ B` | `A && B` |
| `A ∪ B` | `A \|\| B` |
| complement `Aᶜ` | `!A` |
| `(A ∩ B)ᶜ = Aᶜ ∪ Bᶜ` | De Morgan |
| `A ∩ (B ∪ C) = (A ∩ B) ∪ (A ∩ C)` | distributive |
| `\|A ∪ B\| = \|A\| + \|B\| − \|A ∩ B\|` | inclusion–exclusion |

**Why the cost of intersection is `min(...)`:** loop over the smaller set and test membership in the larger one (O(1) each).

**You'll meet it:** deduplicating IDs, "which patients are in the queue but not yet checked", comparing two lists of payers, the `Set` in the `findDuplicates` example.

---

# Tier 2: going deeper

## 9. Recurrences and the Master Theorem

A recursive function's time is described by a **recurrence**: `T(n) = a·T(n/b) + f(n)`.
- `a` = number of recursive calls
- `n/b` = size of each sub-problem
- `f(n)` = work done outside the calls (splitting, merging)

**Master Theorem** (for `f(n) = Θ(nᵈ)`):

| Compare `a` with `bᵈ` | Result | Intuition |
|---|---|---|
| `a < bᵈ` | `Θ(nᵈ)` | the top-level work dominates |
| `a = bᵈ` | `Θ(nᵈ · log n)` | every level costs the same, there are log n levels |
| `a > bᵈ` | `Θ(n^(log_b a))` | the leaves dominate |

| Algorithm | `a, b, d` | Recurrence | Result |
|---|---|---|---|
| Binary search | 1, 2, 0 | `T(n/2) + 1` | `a = bᵈ` → Θ(log n) |
| Tree traversal | 2, 2, 0 | `2T(n/2) + 1` | `a > bᵈ` → Θ(n) |
| Merge sort | 2, 2, 1 | `2T(n/2) + n` | `a = bᵈ` → Θ(n log n) |
| Halve, then linear work | 1, 2, 1 | `T(n/2) + n` | `a < bᵈ` → Θ(n) |

The theorem does not cover `T(n) = T(n−1) + …` (shrinks by subtraction). Use these instead:
- `T(n) = T(n−1) + 1` → `n` (linear)
- `T(n) = T(n−1) + n` → `n(n+1)/2` → O(n²)
- `T(n) = 2T(n−1) + 1` → `2ⁿ − 1` → O(2ⁿ)

**Recursion-tree method** (works for anything): draw the tree, find the work per level, find the number of levels, multiply.

## 10. Amortized analysis

**Idea.** Some operations are occasionally expensive but cheap on average **over a sequence**. Amortized cost = total cost of a sequence ÷ number of operations. It is a **guarantee**, not a probability.

**Dynamic array (`push`) with doubling:** when full, copy to an array twice as big.
- Copy costs happen at sizes 1, 2, 4, 8, …, n.
- Total copy work = `1 + 2 + 4 + … + n < 2n`.
- n pushes cost `n + 2n = 3n` → **O(1) amortized per push**.

Methods:
- **Aggregate:** total cost ÷ n (as above).
- **Accounting:** charge each cheap op a little extra "credit" to pay for the rare expensive one.
- **Potential:** a function of the data structure's state that tracks stored-up work.

Don't confuse it with **average case** (which assumes random input). Amortized holds for the **worst possible** sequence.

## 11. Relations and functions

**Relation.** A set of pairs `(a, b)` linking items of one set to items of another. A table row linking `request → payer` is a relation.

**Function.** A relation where **each input has exactly one output**. A `Map` is a function from keys to values.

| Property | Meaning | Real example |
|---|---|---|
| One-to-one (injective) | different inputs give different outputs | user id → user |
| Onto (surjective) | every possible output is hit | |
| Many-to-one | several inputs share an output | request → payer (many requests, one payer) |
| One-to-many | one input has several outputs | payer → requests |
| Many-to-many | needs a join table | agents ↔ payers (routing rules) |

**Properties of a relation `R` on one set:**
| Property | Rule | Example |
|---|---|---|
| Reflexive | `a R a` | `a ≤ a` |
| Symmetric | `a R b ⇒ b R a` | "same practice as" |
| Antisymmetric | `a R b` and `b R a ⇒ a = b` | `≤` |
| Transitive | `a R b` and `b R c ⇒ a R c` | `<`, "is ancestor of" |

- **Equivalence relation** = reflexive + symmetric + transitive. It splits a set into groups (e.g. "same payer" groups requests). This is what **grouping and `Map`-keying** rely on.
- **Total order** = antisymmetric + transitive + any two items comparable. This is what **sorting** needs.

**Sort comparators must be consistent.** `arr.sort((a, b) => a - b)` is valid because numeric `<` is a strict ordering. A comparator that returns random or contradictory results (`a < b` and `b < a`) gives **undefined order** and can break binary search later.

```js
// Good: consistent, transitive
items.sort((a, b) => a.date - b.date || a.id.localeCompare(b.id));
// Bad: not a valid ordering
items.sort(() => Math.random() - 0.5);
```

**Also:** a **pure function** (same input → same output, no side effects) is easy to test and to memoize. Memoization only works for pure functions.

## 12. Induction and invariants

**Why it matters.** Complexity says "fast". Induction and invariants say "correct". Correctness comes first.

**Proof by induction** (works for loops and recursion):
1. **Base case:** true for the smallest input.
2. **Step:** assume true for `k` (the hypothesis), show it is true for `k + 1`.

Example: prove `1 + 2 + … + n = n(n+1)/2`.
- Base: n = 1 → `1 = 1·2/2` ✓.
- Step: assume true for `k`. Then `1 + … + k + (k+1) = k(k+1)/2 + (k+1) = (k+1)(k+2)/2` ✓.

**Loop invariant.** A statement that is true **before every iteration**. Show: (1) true at the start, (2) each iteration keeps it true, (3) when the loop ends, the invariant gives you the answer.

```js
function max(arr) {            // arr is non-empty
  let best = arr[0];
  for (let i = 1; i < arr.length; i++) {
    // invariant: best is the max of arr[0..i-1]
    if (arr[i] > best) best = arr[i];
  }
  return best;                 // at exit, best is the max of arr[0..n-1]
}
```

**Recursion correctness:** show the base case is right and that, if the recursive call is right on the smaller input, the combined result is right. Also show the input **gets smaller** each call (this proves it terminates).

## 13. Probability basics

| Idea | Formula | Use |
|---|---|---|
| Probability | `P(event) = favourable / total` (equally likely) | |
| Independent events | `P(A and B) = P(A)·P(B)` | combined failure rates |
| Complement | `P(not A) = 1 − P(A)` | "at least one" = 1 − "none" |
| Expected value | `E[X] = Σ x · P(x)` | average cost of a random operation |
| Uniform distribution | each outcome equally likely | idealised hashing |

**Examples:**
- A 1% failure per call, 100 independent calls: P(at least one failure) = `1 − 0.99¹⁰⁰ ≈ 63%`. Small error rates compound.
- **Hash tables:** with a good hash and `m` buckets holding `n` items, the expected bucket length is `n/m` (the load factor). Keeping it low is why resizing keeps lookup O(1) on average.
- **Birthday effect:** a collision becomes likely after roughly `√m` random items in `m` slots, much sooner than people expect. It matters for random IDs and short hash codes.
- **Quicksort:** a random pivot gives expected O(n log n), but bad pivots give O(n²). "Average case" assumes random input, which real data may not be.

## 14. Graph basics

A **graph** is vertices (nodes) `V` joined by edges `E`.

| Term | Meaning |
|---|---|
| Directed / undirected | edges have a direction, or not |
| Degree | number of edges at a vertex |
| Path / cycle | a walk along edges / a path that returns to its start |
| Tree | connected, no cycles, `n − 1` edges |
| DAG | directed, no cycles (dependency graphs, job pipelines) |
| Complete graph | every pair is joined: `n(n−1)/2` edges (undirected) |

- **Storage:** adjacency list = O(V + E) memory (good for sparse graphs). Adjacency matrix = O(V²) memory.
- **BFS / DFS** visit every vertex and edge once: **O(V + E)**.
- **Topological order** exists only for a DAG (use it to order dependent steps).
- A tree with `n` nodes and height `h`: lookups O(h). Balanced: `h = log n`. Degenerate (a list): `h = n`.

**Where it appears:** routing rules, dependency order of migrations, call graphs (the project's graphify graph is one), retry/state machines.

## 15. Modular arithmetic

`a % m` is the remainder after dividing `a` by `m`. Results lie in `0 … m−1` for non-negative `a`.

| Use | Code |
|---|---|
| Hash to a bucket | `hash % bucketCount` |
| Wrap around / round-robin | `agents[i % agents.length]` |
| Even / odd | `n % 2 === 0` |
| Pagination offset | `page * size`, `Math.floor(i / size)` for the page number |
| Cyclic schedules | `(day + k) % 7` |

**JS gotcha:** `%` keeps the sign of the dividend.
```js
-1 % 5;                 // -1   (not 4)
((-1 % 5) + 5) % 5;     // 4    true mathematical modulo
```

Properties: `(a + b) % m = ((a % m) + (b % m)) % m`, and the same for multiplication. Useful for preventing overflow and for checksums.

## 16. Binary and bit operations

- Base 2: each position is a power of 2. `1011₂ = 8 + 2 + 1 = 11`.
- `n` bits can represent `2ⁿ` values. 8 bits = 256 values, 32 bits ≈ 4.3 billion.
- **Storage units:** 1 KB = 1,024 B, 1 MB = 1,024 KB, 1 GB = 1,024 MB (powers of 2). Storage vendors sometimes use powers of 10, so check which.

| Operator | Meaning | Example |
|---|---|---|
| `a & b` | bitwise AND | `5 & 3 = 1` |
| `a \| b` | bitwise OR | `5 \| 3 = 7` |
| `a ^ b` | bitwise XOR | `5 ^ 3 = 6` |
| `~a` | bitwise NOT | |
| `a << k` | multiply by 2ᵏ | `3 << 2 = 12` |
| `a >> k` | divide by 2ᵏ (floor), keeps sign | `13 >> 1 = 6` |

**Gotchas in JS:**
- Bitwise operators convert to **32-bit signed integers**. `(lo + hi) >> 1` is safe up to about 2 billion, but `2**40 >> 1` gives a wrong answer. For big values use `Math.floor(x / 2)`.
- Flags: store several booleans in one integer (`FLAG_A = 1, FLAG_B = 2, FLAG_C = 4`), test with `&`.

---

# Tier 3: production and cost

These reuse the numbers from [section 16 of the playbook](complexity-analysis-playbook.md#16-production-memory-structure-scale-and-cost). Here is the math behind them.

| Topic | Key idea | Formula |
|---|---|---|
| **Units** | ns = 10⁻⁹ s, µs = 10⁻⁶, ms = 10⁻³ | 10⁸ ops/s ⇒ 10 ns per op |
| **Orders of magnitude** | Estimate to the nearest power of 10 first | 10⁶ items × 500 B ≈ 500 MB |
| **Little's law** | average items in a system = arrival rate × time spent | `L = λ · W` |
| **Utilisation** | `ρ = λ · C` (fraction of one core busy) | keep ρ below ~0.7 |
| **Queueing blow-up** | wait time grows like `ρ / (1 − ρ)`, so latency climbs steeply near 100% | ρ = 0.5 → 1×, 0.9 → 9×, 0.99 → 99× |
| **Mean vs median** | the mean is dragged by outliers | one 30 s request among 99 at 100 ms: mean ≈ 400 ms, median 100 ms |
| **Percentiles** | p95 = value that 95% of requests are below | report p50, p95, p99, not only the mean |
| **Throughput** | requests per second | `1 / C` per core if CPU-bound |
| **Cost per unit** | monthly cost ÷ monthly volume | cost per 1,000 requests |
| **Break-even** | fixed cost ÷ saving per unit | when a cache or rewrite pays for itself |
| **Proportional reasoning** | double `n`, see time × 2 / × 4 / × 8 | the doubling test |

**Utilisation example.** If each request uses `C = 0.05 s` of CPU and you get `λ = 14` per second, `ρ = 0.7`. At `λ = 19`, `ρ = 0.95` and queues build quickly, so latency spikes before the CPU is "full".

**Compounding percentages:** each stage that is 99.9% reliable, chained across 10 stages, gives `0.999¹⁰ ≈ 99%`. Reliability multiplies, it does not average.

---

# Tier 4: reasoning habits

| Habit | What it means | Example |
|---|---|---|
| **Worst / average / best** | state which case you analyse | `find` worst = missing item |
| **Counterexample** | one failing input disproves "always" | an empty array, or all duplicates |
| **Contrapositive** | "if A then B" ≡ "if not B then not A" | `if (!ok) return;` guards "if reached, ok" |
| **Converse is not equivalent** | "if A then B" does not give "if B then A" | "every saved patient was checked" ≠ "every checked patient was saved" |
| **Quantifiers** | "for all" (`every`) vs "there exists" (`some`) | order matters: "every agent has some patient" ≠ "some patient is shared by every agent" |
| **Edge cases** | empty, one item, duplicates, max size, negative, `null`, unicode | test these first |
| **Invariants** | what stays true | "this lock belongs to one agent" |
| **Pre/post-conditions** | what must hold before, what is promised after | function contracts and validation |
| **Reduce the problem** | solve a smaller version first | recursion, divide and conquer |
| **Check units and bounds** | does the answer's size make sense? | 10⁴ ops in 5 seconds is wrong |

---

# Worked example: the math of one function

```js
function pairsWithSameKey(rows) {
  let count = 0;
  for (let i = 0; i < rows.length; i++)
    for (let j = i + 1; j < rows.length; j++)
      if (rows[i].key === rows[j].key) count++;
  return count;
}
```

**1. Count iterations (summation).**
The body runs `(n−1) + (n−2) + … + 1`. By Gauss, that is `n(n−1)/2`.

**2. Same answer by combinatorics.**
The loops pick every unordered pair of rows, so the count is `C(n,2) = n(n−1)/2`. ✓

**3. Big-O from the definition.**
`n(n−1)/2 = n²/2 − n/2 ≤ n²/2`, so with `c = 1/2`, `f = O(n²)`. And for `n ≥ 2`, `n(n−1)/2 ≥ n²/4`, so `f = Ω(n²)`. Hence `Θ(n²)`.

**4. Numbers.**
n = 10,000 → 49,995,000 iterations. At 10⁸ ops/s, about 0.5 s of one core.

**5. A faster version, with the set/function view.**
Group rows by key (a function `key → rows`), then count pairs inside each group (`C(g,2)` for a group of size `g`).

```js
function pairsWithSameKey(rows) {
  const sizeByKey = new Map();                       // function: key -> group size
  for (const r of rows) sizeByKey.set(r.key, (sizeByKey.get(r.key) ?? 0) + 1);
  let count = 0;
  for (const g of sizeByKey.values()) count += (g * (g - 1)) / 2;   // C(g,2)
  return count;
}
```

**6. Cost of the new version.**
One pass to build the Map (O(n) time, up to O(n) space), one pass over groups (≤ n). Total **O(n) time, O(n) space**.

**7. Prove they agree (induction/invariant idea).**
Two rows form a pair if and only if they share a key (an equivalence relation: reflexive, symmetric, transitive). The relation splits the rows into groups, and every pair lies inside exactly one group. So total pairs = sum of `C(g,2)` over groups. ✓

**8. Check by computer** (counter test):
```js
const rows = [{key:"a"},{key:"a"},{key:"a"},{key:"b"}];
// a: C(3,2)=3, b: C(1,2)=0  -> 3, both versions must return 3
```

This one function used: summation, combinatorics, Big-O definition, equivalence relations, functions (`Map`), and a proof of correctness.

---

# One-page cheat sheet

**Logs and exponents**
```
log₂(2ᵏ) = k     log(ab) = log a + log b     log(aᵏ) = k log a     base change = constant
log₂ 10⁶ ≈ 20    log₂ 10⁹ ≈ 30               2¹⁰ ≈ 10³
```

**Sums**
```
1+2+…+n = n(n+1)/2     1+2+4+…+2ᵏ = 2ᵏ⁺¹−1     c·n for n copies of c     Σ1/i ≈ ln n
```

**Counting**
```
n! orderings     2ⁿ subsets     C(n,k)=n!/(k!(n−k)!)     pairs = n(n−1)/2
```

**Big-O**
```
O: ≤ c·g     Ω: ≥ c·g     Θ: both     drop constants and lower terms
sequential: add     nested: multiply     branches: take max
```

**Master Theorem** `T(n) = aT(n/b) + Θ(nᵈ)`
```
a < bᵈ → n^d     a = bᵈ → n^d log n     a > bᵈ → n^(log_b a)
```

**Logic**
```
!(A && B) = !A || !B          !(A || B) = !A && !B
!every(P) = some(!P)          !some(P) = every(!P)
[].every(...) = true          [].some(...) = false
contrapositive: (A → B) ≡ (!B → !A)
```

**Sets**
```
|A ∪ B| = |A| + |B| − |A ∩ B|     (A ∩ B)ᶜ = Aᶜ ∪ Bᶜ     A \ B = A ∩ Bᶜ
```

**Production**
```
Little: L = λW     utilisation ρ = λC     wait ∝ ρ/(1−ρ)     cores = λC / 0.7
memory = n × bytes × copies × concurrent requests
```

**JS gotchas**
```
-1 % 5 = -1     bitwise ops are 32-bit     truthy "0" and []     sort comparator must be consistent
```

---

# Self-test

Try these before looking at the answers.

1. How many times can you halve 1,000,000,000 before reaching 1?
2. How many iterations does `for i in 0..n, for j in i+1..n` run for n = 100?
3. What is `Θ` of `5n² + 100n + 7`?
4. Rewrite `!(a > 5 && b === 0)` using De Morgan.
5. What does `[].every(x => x > 100)` return, and why can that cause a bug?
6. A recursion has `T(n) = 3T(n/2) + n`. Using the Master Theorem, what is its complexity?
7. A request uses 0.2 s of CPU and 4 requests arrive per second. What is the utilisation of one core?
8. Using `Set`s, how would you find IDs present in `queue` but not in `done`, and what is its cost?
9. 100 independent calls each fail 2% of the time. What is the chance at least one fails?
10. What does `-7 % 3` give in JS, and how do you get the mathematical modulo?

**Answers**

1. About 30 (`log₂ 10⁹ ≈ 29.9`).
2. `100·99/2 = 4,950`.
3. `Θ(n²)`.
4. `a <= 5 || b !== 0`.
5. `true`. An empty list passes "every" vacuously, so empty or missing data can look valid. Check for emptiness first.
6. `a = 3, b = 2, d = 1`: `bᵈ = 2 < 3 = a`, so `Θ(n^(log₂ 3)) ≈ Θ(n^1.585)`.
7. `ρ = 4 × 0.2 = 0.8` (80% busy; latency will rise quickly. Above ~0.7 is risky).
8. `const doneSet = new Set(done); const todo = queue.filter(id => !doneSet.has(id));` Cost: O(|done|) to build plus O(|queue|) to filter → **O(|queue| + |done|)** time, O(|done|) space.
9. `1 − 0.98¹⁰⁰ ≈ 1 − 0.133 = 86.7%`.
10. `-1`. True modulo: `((-7 % 3) + 3) % 3 = 2`.

