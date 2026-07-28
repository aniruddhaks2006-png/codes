# Valid Joker Groups

![LeetCode Medium](https://shields.io)
![Category: Dynamic Programming](https://shields.io)
![Time Complexity: O(N)](https://shields.io)
![Space Complexity: O(1)](https://shields.io)

## 📝 Problem Statement

You are given a string `s` consisting of digits `'1'` to `'9'` and the character `'J'`, where `'J'` represents a joker wildcard. 

The length of the string is always a multiple of 3. You may rearrange the characters in any order before partitioning them into groups of exactly 3 characters.

A group is considered **valid** if it satisfies one of the following conditions:
* **Set**: All three cards represent the same digit (e.g., `"111"`, `"555"`, `"999"`).
* **Run**: The three cards represent three consecutive digits in increasing order (e.g., `"123"`, `"456"`, `"789"`).

A joker (`'J'`) may represent any digit from `1` to `9`. Each joker can be assigned exactly one value and may be used only once.

Return `true` if it is possible to rearrange the string into valid groups. Otherwise, return `false`.

---

## 📥 Examples

### Example 1
* **Input:** `s = "321111"`
* **Output:** `true`
* **Explanation:** Rearrange the string into `"123"` and `"111"`.

### Example 2
* **Input:** `s = "12J111"`
* **Output:** `true`
* **Explanation:** Assign the joker the value `3` and rearrange into `"123"` and `"111"`.

### Example 3
* **Input:** `s = "112233"`
* **Output:** `true`
* **Explanation:** Rearrange into `"123"` and `"123"`.

### Example 4
* **Input:** `s = "114"`
* **Output:** `false`
* **Explanation:** It is impossible to form either a valid set or a valid run.

### Example 5
* **Input:** `s = "JJJ"`
* **Output:** `true`
* **Explanation:** The jokers may represent `"111"` or `"123"`.

---

## ⚙️ Constraints
* `3 <= s.length <= 3 × 10^5`
* `s.length % 3 == 0`
* `s[i]` is either `'J'` or a digit from `'1'` to `'9'`.

---

## 💡 Solution Strategy & Intuition

Because the string can be rearranged freely, the absolute layout of the characters does not matter. The problem naturally reduces to a frequency-matching task using the counts of digits `1` through `9` and the wildcard pool `J`.

### Key Optimizations
1. **Run Reduction Rule**: Three identical runs (e.g., three `"123"` runs) consume the exact same card frequency footprint as three individual sets (`"111"`, `"222"`, `"333"`). Therefore, we never need to track starting more than **2 runs** at any specific digit sequence.
2. **State Space Compression**: We track structural continuity via a micro-DP lookup array bounded by a size of 9 states. 
   * `j` = Active runs ending at the current digit `i`.
   * `k` = Active runs continuing past the current digit `i`.
   * `l` = Proposed new runs starting at digit `i`.

---

## 💻 Implementations

### Python 3

```python
class Solution:
    def isValidJokerGroups(self, s: str) -> bool:
        counts = [0] * 12
        jokers = 0
        
        # Step 1: Build frequency map
        for char in s:
            if char == 'J':
                jokers += 1
            else:
                counts[int(char)] += 1

        # dp[state_index] stores the minimum number of jokers needed.
        # State index = (runs_at_i_minus_2 * 3) + runs_at_i_minus_1
        dp = [float('inf')] * 9
        dp[0] = 0 

        # Step 2: Traverse digit range (padded to 11 to close trailing sequences)
        for i in range(1, 12):
            next_dp = [float('inf')] * 9
            c = counts[i] if i <= 9 else 0

            for j in range(3):
                for k in range(3):
                    current_jokers = dp[j * 3 + k]
                    if current_jokers == float('inf'):
                        continue

                    for l in range(3):
                        needed = j + k + l
                        
                        if c >= needed:
                            rem = (c - needed) % 3
                            jokers_needed = (3 - rem) % 3
                        else:
                            jokers_needed = needed - c

                        total_jokers = current_jokers + jokers_needed
                        next_state = k * 3 + l

                        if total_jokers < next_dp[next_state]:
                            next_dp[next_state] = total_jokers
            dp = next_dp

        return dp[0] <= jokers
```

### C++

```cpp
#include <string>
#include <vector>
#include <algorithm>

class Solution {
public:
    bool isValidJokerGroups(std::string s) {
        std::vector<int> counts(12, 0); 
        int jokers = 0;

        // Step 1: Build frequency map
        for (char c : s) {
            if (c == 'J') jokers++;
            else counts[c - '0']++;
        }

        std::vector<int> dp(9, -1);
        dp[0] = 0; 

        // Step 2: Traverse digit range
        for (int i = 1; i <= 11; ++i) {
            std::vector<int> next_dp(9, -1);
            int c = (i <= 9) ? counts[i] : 0;

            for (int j = 0; j < 3; ++j) {
                for (int k = 0; k < 3; ++k) {
                    int current_jokers = dp[j * 3 + k];
                    if (current_jokers == -1) continue;

                    for (int l = 0; l < 3; ++l) {
                        int needed = j + k + l;
                        int jokers_needed = 0;

                        if (c >= needed) {
                            int rem = (c - needed) % 3;
                            jokers_needed = (3 - rem) % 3;
                        } else {
                            jokers_needed = needed - c;
                        }

                        int total_jokers = current_jokers + jokers_needed;
                        int next_state = k * 3 + l;

                        if (next_dp[next_state] == -1 || total_jokers < next_dp[next_state]) {
                            next_dp[next_state] = total_jokers;
                        }
                    }
                }
            }
            dp = std::move(next_dp);
        }
        return (dp[0] != -1 && dp[0] <= jokers);
    }
};
```

---

## 📊 Complexity Analysis

* **Time Complexity:** $\mathcal{O}(N)$  
  Populating the frequency map takes a single linear pass over the input string of length $N$. The DP transitions execute over a fixed loop size ($11 \times 3 \times 3 \times 3 = 297$ constant operations), leading to an execution runtime scale of $\mathcal{O}(N)$.
  
* **Space Complexity:** $\mathcal{O}(1)$  
  The memory requirements are completely independent of $N$. The execution state is tracked strictly inside a 12-element frequency vector and two tiny 9-element DP state buffers.

---

## 📈 Follow-up Goals Met
* [x] Solved in **$\mathcal{O}(N)$** time complexity.
* [x] Strictly bounded to **$\mathcal{O}(1)$** auxiliary space allocation.
* [ ] 
