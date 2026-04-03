## 265. Paint House II
[https://leetcode.com/problems/paint-house-ii/](https://leetcode.com/problems/paint-house-ii/)

## The Problem
There are a row of `n` houses, each house can be painted with one of the `k` colors. The cost of painting each house with a certain color is different. You have to paint all the houses such that no two adjacent houses have the same color. The cost of painting each house with a certain color is represented by an `n x k` cost matrix. Return the minimum cost to paint all houses.

### Constraints
* `costs.length == n`
* `costs[i].length == k`
* `1 <= n <= 100`
* `1 <= k <= 20`
* `1 <= costs[i][j] <= 20`

## Brute Force Idea
A standard 2D Dynamic Programming approach where for every house `i` and every color `j`, we loop through all `k` colors of the previous house to find the minimum cost that doesn't share the same color. 
* Time Complexity: $O(n \times k^2)$
* Space Complexity: $O(n \times k)$ to store the DP table.

## The Architecture: Optimal
Algorithm: 1D Dynamic Programming with Two-Minimum State Tracking.
Data Structure: Two 1D Vectors.

Instead of scanning all previous colors every time, we only need to track the absolute minimum cost (`min1`) and the second minimum cost (`min2`) from the previous row. If the current color clashes with `min1`'s color, we use `min2`. Otherwise, we use `min1`.

### 1. Optimal Approach (Two Minimums)
```cpp
#include <vector>
#include <algorithm>
#include <climits>

using namespace std;

class Solution {
public:
    int minCostII(vector<vector<int>>& costs) {
        int n = costs.size();
        if (n == 0) return 0; 
        
        int k = costs[0].size();
        if (k == 0) return 0;
        if (k == 1 && n > 1) return 0; 

        vector<int> prevColorCosts = costs[0]; 

        for (int idx = 1; idx < n; ++idx) {
            int min1 = INT_MAX, min2 = INT_MAX;
            int min1Idx = -1;
            
            for (int i = 0; i < k; ++i) {
                if (prevColorCosts[i] < min1) {
                    min2 = min1;
                    min1 = prevColorCosts[i];
                    min1Idx = i;
                } else if (prevColorCosts[i] < min2) {
                    min2 = prevColorCosts[i]; 
                }
            }

            vector<int> currColorCosts(k, 0);
            for (int j = 0; j < k; ++j) {
                if (j != min1Idx) {
                    currColorCosts[j] = costs[idx][j] + min1;
                } else {
                    currColorCosts[j] = costs[idx][j] + min2;
                }
            }
            
            prevColorCosts = currColorCosts;
        }
        
        return *min_element(prevColorCosts.begin(), prevColorCosts.end());
    }
};
```
* Optimal Time Complexity: $O(n \times k)$
* Optimal Space Complexity: $O(k)$

### System Design / Real-Life Impact
**State Management & Memory Footprint:** This algorithm is the algorithmic equivalent of compressing state. In highly scalable systems (like tracking states in a massive Kafka stream), avoiding nested iterations and reducing memory from an $N \times M$ matrix to a rolling 1D buffer prevents memory blooming and cache misses in the CPU.
