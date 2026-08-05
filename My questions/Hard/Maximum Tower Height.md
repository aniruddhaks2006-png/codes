# Maximum Tower Height

[LeetCode Hard] [Category: Greedy / Sorting] [Time Complexity: O(N log N)] [Space Complexity: O(N)]

## 📝 Problem Statement

You are given an integer array `weights`, where `weights[i]` represents the weight of the `i`-th block.

You may build a tower consisting of one or more floors.

- Each block can be used **at most once**.
- Each floor consists of **one or more** blocks.
- The **weight of a floor** is the sum of the weights of all blocks assigned to that floor.
- You may **discard** any number of blocks.

A tower is considered **valid** if, for every pair of adjacent floors, the weight of the lower floor is at least 3 times the weight of the upper floor:

`weight(lowerFloor) >= 3 × weight(upperFloor)`

Return the **maximum possible number of floors** that can be built.

## 📥 Examples

## Example 1

- *Input:* weights = [9, 5, 4, 3, 2, 1]
- *Output:* 3
- *Explanation:* One optimal construction is:
  - Bottom: 9
  - Middle: 3
  - Top: 1
  
  Since 9 >= 3 × 3 and 3 >= 3 × 1, the tower is valid.

## Example 2

- *Input:* weights = [1, 2, 3]
- *Output:* 2
- *Explanation:* Bottom: 3, Top: 1. Block `2` is discarded.

## Example 3

- *Input:* weights = [2, 2, 2]
- *Output:* 1
- *Explanation:* It is impossible to build more than one floor while satisfying the required weight ratio.

## ⚙️ Constraints

- 1 <= weights.length <= 2 × 10^5
- 1 <= weights[i] <= 10^9

## 💡 Solution Strategy & Intuition

To maximize the total number of floors, we should build every floor using single blocks whenever possible and minimize the weight of upper floors. 

1. **Greedy Selection with Single Blocks**: Sorting the array allows us to pick the smallest available weight as our top floor ($W_{top}$).
2. **Binary Search / Prefix Sums for Grouping**: To form lower floors, we greedily pick the smallest block or contiguous sum of smaller blocks that satisfies $W_{lower} \ge 3 \times W_{upper}$. 
3. Processing blocks from smallest to largest ensures that upper floors remain as small as possible, maximizing room for subsequent floors below.

## 💻 Implementations

## Python 3

```python
class Solution:
    def maxTowerHeight(self, weights: list[int]) -> int:
        weights.sort()
        floors = 0
        current_weight = 0
        
        # Build floors starting from the top (smallest weights)
        i = 0
        n = len(weights)
        
        while i < n:
            if floors == 0:
                current_weight = weights[i]
                floors += 1
                i += 1
            else:
                target_weight = 3 * current_weight
                accumulated_weight = 0
                
                while i < n and accumulated_weight < target_weight:
                    accumulated_weight += weights[i]
                    i += 1
                
                if accumulated_weight >= target_weight:
                    current_weight = accumulated_weight
                    floors += 1
                else:
                    break
                    
        return floors
