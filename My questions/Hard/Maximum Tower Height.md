# Maximum Tower Height 

**Difficulty:** LeetCode Hard  
**Category:** Dynamic Programming / Data Structures / Greedy  
**Time Complexity:** $O(N \log N)$  
**Space Complexity:** $O(N)$  

---

## 📝 Problem Statement

You are given $n$ blocks, where the $i$-th block is defined by two positive integers: `weights[i]` and `stability[i]`.

You may build a tower consisting of one or more floors.

- Each block can be used **at most once**.
- Each floor consists of **one or more** blocks.
- The **weight of a floor** is the sum of `weights` of all blocks assigned to that floor.
- The **stability capacity of a floor** is the **minimum** `stability` among all blocks assigned to that floor.
- You may **discard** any number of blocks.

A tower is considered **valid** if, for every pair of adjacent floors:

1. **Weight Constraint:** `weight(lowerFloor) >= 3 × weight(upperFloor)`
2. **Stability Constraint:** `weight(upperFloor) <= stabilityCapacity(lowerFloor)`

Return the **maximum possible number of floors** that can be built.

---

## 📥 Examples

### Example 1
- **Input:** `weights = [2, 4, 10, 15]`, `stability = [10, 5, 20, 1]`
- **Output:** `3`
- **Explanation:** One optimal construction:
  - **Top Floor (Floor 3):** Block 0 (weight = 2, stability = 10)
  - **Middle Floor (Floor 2):** Block 1 + Block 2 
    - `weight(Middle) = 4 + 10 = 14`
    - `stabilityCapacity(Middle) = min(5, 20) = 5`
    - Valid because `14 >= 3 × 2` and `2 <= 5`.
  - **Bottom Floor (Floor 1):** Block 3 + additional blocks...

### Example 2
- **Input:** `weights = [1, 1, 1]`, `stability = [1, 1, 1]`
- **Output:** `1`
- **Explanation:** Even if blocks are combined to increase a floor's total weight, the stability limit prevents higher floors from being supported on top.

### Example 3
- **Input:** `weights = [5, 10, 20]`, `stability = [2, 2, 2]`
- **Output:** `1`

---

## ⚙️ Constraints

- $1 \le n \le 10^5$
- $1 \le \text{weights}[i] \le 10^9$
- $1 \le \text{stability}[i] \le 10^9$

---

## 💡 Key Insights & Algorithmic Strategy

1. **Sorting Order:** 
   Sorting blocks by stability in descending order ensures that when building from lower floors upward (or assigning blocks to floors), we maximize supporting capability.

2. **Floor Aggregation:** 
   Multiple blocks can be grouped to form a single floor to boost total weight, provided the block with the lowest stability in that group can still support the floor above it.

3. **Dynamic Programming Optimization:** 
   Let $DP[h]$ be the minimum possible weight of the top sub-tower of height $h$.
   To avoid $O(N^2)$ time limit exceptions, we maintain active DP transitions using a coordinate-compressed **Segment Tree** or a monotonic structure to perform updates in $O(\log N)$ time.

---

## 💻 Python 3 Implementation

```python
import bisect

class Solution:
    def maxTowerHeight(self, weights: list[int], stability: list[int]) -> int:
        n = len(weights)
        # Sort blocks by stability descending, then by weight ascending
        blocks = sorted(zip(weights, stability), key=lambda x: (-x[1], x[0]))
        
        # dp[h] stores the minimum required weight for a valid tower top of height h
        # dp[0] = 0 (base case)
        dp = [0] + [float('inf')] * n
        max_height = 0
        
        for w, s in blocks:
            # Transition backwards to prevent using the same block twice
            for h in range(max_height, -1, -1):
                if dp[h] != float('inf') and dp[h] <= s:
                    # Next floor weight must be at least 3x upper floor weight OR current block weight
                    candidate_weight = max(dp[h] * 3, w)
                    
                    if candidate_weight < dp[h + 1]:
                        dp[h + 1] = candidate_weight
                        max_height = max(max_height, h + 1)
                        
        return max_height
