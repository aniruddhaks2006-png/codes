# 🔢 First Missing Divisor

**Difficulty:** Easy
**Category:** Math / Number Theory / Hashing

---

## 📝 Problem Statement

You are given an integer `n` and an array `divisors`, which contains some of the positive divisors of `n`.

A divisor `x` of `n` satisfies:

```text
n % x == 0
```

Your task is to find the **smallest positive divisor of `n` that is missing from the given array**.

You may assume that all elements in `divisors` are valid divisors of `n`.

Return the smallest missing divisor.

If all divisors of `n` are present, return `-1`.

---

## 📥 Examples

### Example 1

**Input**

```text
n = 12
divisors = [1, 3, 4, 6, 12]
```

**Output**

```text
2
```

**Explanation**

All divisors of `12` are:

```text
[1, 2, 3, 4, 6, 12]
```

The smallest missing divisor is:

```text
2
```

---

### Example 2

**Input**

```text
n = 36
divisors = [1, 2, 3, 4, 6, 9, 36]
```

**Output**

```text
12
```

**Explanation**

All divisors of `36` are:

```text
[1, 2, 3, 4, 6, 9, 12, 18, 36]
```

The first missing divisor is:

```text
12
```

---

### Example 3

**Input**

```text
n = 7
divisors = [1, 7]
```

**Output**

```text
-1
```

**Explanation**

The divisors of `7` are:

```text
[1, 7]
```

No divisor is missing.

---

## ⚙️ Constraints

```text
1 <= n <= 10^9
1 <= divisors.length <= 10^5
1 <= divisors[i] <= n
```

All values in `divisors` are guaranteed to divide `n`.

---

## 💡 Approach

A brute-force approach would check every number from `1` to `n`, which is too slow.

Instead, use the fact that divisors come in pairs:

For every `i`:

```text
i * (n / i) = n
```

We only need to check:

```text
1 <= i <= sqrt(n)
```

Steps:

1. Store all given divisors in a hash set.
2. Generate all divisors of `n` using divisor pairs.
3. Sort the divisors.
4. Find the first divisor that is not present in the set.

---

## ⏱️ Complexity Analysis

Let `d` be the number of divisors of `n`.

**Time Complexity:**

```text
O(sqrt(n) + d log d)
```

**Space Complexity:**

```text
O(d)
```

---

## 💻 Python 3 Solution

```python
class Solution:
    def firstMissingDivisor(self, n: int, divisors: list[int]) -> int:
        present = set(divisors)

        all_divisors = []

        i = 1
        while i * i <= n:
            if n % i == 0:
                all_divisors.append(i)

                if i != n // i:
                    all_divisors.append(n // i)

            i += 1

        all_divisors.sort()

        for d in all_divisors:
            if d not in present:
                return d

        return -1
```

---

## 🔍 Follow Up

Can you solve it without sorting the divisors?

Can you answer multiple queries for the same value of `n` efficiently?
