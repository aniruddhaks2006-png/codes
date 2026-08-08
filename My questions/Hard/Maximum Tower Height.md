# Maximum Tower Height

**Difficulty:** LeetCode Hard
**Category:** Dynamic Programming / Sorting / Fenwick Tree (2D dominance)
**Time Complexity:** O(N log N)
**Space Complexity:** O(N)

---

## Problem Statement

You are given `n` blocks, where the `i`-th block is defined by two positive
integers: `weights[i]` and `stability[i]`.

A tower is a **stack of blocks** (one block per floor). You may choose any
subset of the blocks and stack them in any order. You may discard any blocks.

A tower of height `H` is a sequence of blocks `b1, b2, ..., bH` where `b1` is
the **bottom** floor and `bH` is the **top** floor. The tower is **valid** if,
for every pair of adjacent floors (lower floor `bi`, upper floor `bi+1`):

1. **Weight Constraint:** `weights[bi] >= 3 * weights[bi+1]`
2. **Stability Constraint:** `weights[bi+1] <= stability[bi]`

Return the **maximum possible number of floors** (blocks) in a valid tower.

### Intuition for the rules

- Going from top to bottom, weights must grow by **at least a factor of 3**
  per floor (the `3x` weight rule).
- A block can sit directly below another block `U` only if its stability is at
  least `weights[U]` — i.e. a block supports everything heavier than itself
  that sits on top of it.

---

## Examples

### Example 1

**Input**

```text
weights  = [2, 4, 10, 15]
stability = [10, 5, 20, 1]
```

**Output**

```text
2
```

**Explanation**

One optimal tower (bottom to top):

- **Floor 1 (bottom):** Block 2 (`weight = 10, stability = 20`)
- **Floor 2 (top):** Block 0 (`weight = 2, stability = 10`)

Validity check:

```text
weight(bottom) = 10 >= 3 x weight(top) = 3 x 2 = 6
weight(top) = 2  <= stability(bottom) = 20
```

Height `3` is impossible: the bottom floor would need weight `>= 3 x 10 = 30`,
and the only remaining block with enough stability (`>= 10`) is Block 2
(`weight = 15, stability = 20`), whose weight `15 < 30`. The remaining blocks
either weigh too little (Block 1: `4`) or are too weak (Block 3: `stability = 1`).

### Example 2

**Input**

```text
weights  = [1, 1, 1]
stability = [1, 1, 1]
```

**Output**

```text
1
```

**Explanation**

To stack two blocks, the lower block would need `weight >= 3 * 1 = 3`, but no
block is heavy enough. So the maximum height is `1`.

### Example 3

**Input**

```text
weights  = [5, 10, 20]
stability = [2, 2, 2]
```

**Output**

```text
1
```

**Explanation**

The `3x` weight rule can be satisfied by stacking `20` under `5`
(`20 >= 3 x 5`), but then the top block weighs `5 > stability(bottom) = 2`.
Any other pairing fails the `3x` rule. So the maximum height is `1`.

---

## Constraints

```text
1 <= n <= 10^5
1 <= weights[i] <= 10^9
1 <= stability[i] <= 10^9
```

---

## Approach

### Key insight: the tower is a DAG longest path

A valid tower is a chain `b1 -> b2 -> ... -> bH` where `bi` is directly below
`bi+1`. Create an edge from a block `A` to a block `B` iff `A` can sit
**directly below** `B`:

```text
weights[A] >= 3 * weights[B]   and   weights[B] <= stability[A]
```

Because `weights[A] >= 3 * weights[B]`, every edge strictly shrinks the weight
by a factor of at least 3. Therefore the graph is a **DAG**, and the answer is
the **longest path** over all blocks.

### Top-down DP

Define `dp[B]` = the length of the longest valid tower **ending with `B` on
top**. Then:

```text
dp[B] = 1 + max { dp[A] : weights[A] >= 3 * weights[B] and stability[A] >= weights[B] }
```

and the answer is `max(dp[B])` over all `B`. (A tower of a single block gives
`dp[B] = 1`.)

### Making it fast

The naive transition scans all `A` for each `B`, giving `O(N^2)`. To speed it
up:

1. **Process blocks in decreasing `weights[B]`.** When we compute `dp[B]`, only
   blocks `A` with `weights[A] >= 3 * weights[B]` (already heavier, hence
   already processed) can appear below `B`.
2. **Two-pointer insertion.** Sweep blocks from heaviest to lightest. Before
   processing `B`, insert into the data structure every block `A` whose weight
   is `>= 3 * weights[B]` (use a pointer, since the threshold only grows as
   weights shrink).
3. **Fenwick tree over stability.** For each inserted block `A` we record its
   `dp[A]` at coordinate `stability[A]`. The query for `B` is:

   ```text
   max { dp[A] : stability[A] >= weights[B] }
   ```

   a *suffix maximum* over the stability axis, done in `O(log N)` with a
   Fenwick (BIT) after coordinate compression.

Overall complexity: `O(N log N)` time, `O(N)` space.

---

## Correctness Sketch

1. **Every valid tower corresponds to a path.** The two adjacency rules are
   exactly the edge definition, so a tower of height `H` is a path of `H`
   blocks and vice-versa. Longer towers = longer paths.
2. **DP recurrence is exact.** `dp[B]` can only extend a tower whose top block
   `A` satisfies the two rules; the recurrence takes the best such `A`. By
   induction over decreasing weights (a topological order), `dp[B]` is the
   maximum tower height ending at `B`.
3. **Fenwick query equals the max over eligible `A`.** A block `A` is eligible
   iff it has been inserted (`weights[A] >= 3 * weights[B]`) and
   `stability[A] >= weights[B]` — exactly the suffix-max query.

---

## Python 3 Implementation

### Reference solution (O(N^2)) — for clarity

```python
class Solution:
    def maxTowerHeight(self, weights: list[int], stability: list[int]) -> int:
        n = len(weights)
        order = sorted(range(n), key=lambda i: -weights[i])
        dp = [1] * n
        best = 1

        for b in order:
            for a in order:
                # a directly below b
                if weights[a] >= 3 * weights[b] and stability[a] >= weights[b]:
                    dp[b] = max(dp[b], dp[a] + 1)
            best = max(best, dp[b])

        return best
```

### Optimal solution (O(N log N))

```python
import bisect


class Solution:
    def maxTowerHeight(self, weights: list[int], stability: list[int]) -> int:
        n = len(weights)
        order = sorted(range(n), key=lambda i: -weights[i])

        stab_vals = sorted(set(stability))

        # Fenwick tree for prefix-max (we query the reversed axis for suffix-max)
        bit = [0] * (len(stab_vals) + 1)

        def update(i: int, val: int) -> None:
            i += 1
            while i < len(bit):
                if val > bit[i]:
                    bit[i] = val
                i += i & -i

        def query(i: int) -> int:
            # max over indices [0, i) — after reversal this is the suffix max
            res = 0
            while i > 0:
                if bit[i] > res:
                    res = bit[i]
                i -= i & -i
            return res

        dp = [0] * n
        ptr = 0  # first index in `order` not yet inserted into the BIT

        for b in order:
            wb = weights[b]

            # insert every block a that is heavy enough to sit below b
            while ptr < n and weights[order[ptr]] >= 3 * wb:
                a = order[ptr]
                pos = bisect.bisect_left(stab_vals, stability[a])
                update(len(stab_vals) - 1 - pos, dp[a])
                ptr += 1

            # best dp[a] among inserted a with stability[a] >= wb
            pos = bisect.bisect_left(stab_vals, wb)
            dp[b] = 1 + query(len(stab_vals) - pos)

        return max(dp)
```

---

## Complexity Analysis

Let `d` be the number of distinct stability values.

**Time Complexity:**

```text
O(N log N)
```

The sweep visits each block once; each Fenwick update and query costs
`O(log N)`. Coordinate compression takes `O(N log N)`.

**Space Complexity:**

```text
O(N)
```

For the Fenwick tree, the `dp` array, and the compressed coordinates.

---

## Follow Up

1. Can you adapt the DP if the required weight multiplier changes from `3` to
   an arbitrary `k >= 2`?
2. Can you output one **witness tower** (the actual block order) that achieves
   the maximum height, in the same `O(N log N)` time?
3. The `3x` multiplier bounds the maximum height by
   `1 + log_3(max(weights) / min(weights))`. Can you use this to derive a
   simpler solution when the multiplier is very large?
