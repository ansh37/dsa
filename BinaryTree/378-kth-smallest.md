
## 378. Kth Smallest Element in a Sorted Matrix
[https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/)

## The Problem
Given an `n x n` matrix where each of the rows and columns is sorted in ascending order, return the `k`th smallest element in the matrix. Note that it is the `k`th smallest element in the sorted order, not the `k`th distinct element. You must find a solution with a memory complexity better than $O(n^2)$.

### Constraints
* `n == matrix.length == matrix[i].length`
* `1 <= n <= 300`
* `-10^9 <= matrix[i][j] <= 10^9`

## Brute Force Idea
Extract all elements into a 1D array and sort it. 
* Time Complexity: $O(n^2 \log(n^2))$
* Space Complexity: $O(n^2)$

## The Architecture: Optimal
Algorithm: Binary Search on Answer Space (using the `low <= high` template).
Data Structure: Pure variables (In-place).

Instead of searching indices, we search the *values* between the smallest possible number `matrix[0][0]` and the largest `matrix[n-1][n-1]`. We count how many elements are smaller than our `mid` guess.

### 1. Optimal Approach (Binary Search on Values)
```cpp
#include <vector>

using namespace std;

class Solution {
public:
    int countLessEqual(vector<vector<int>>& matrix, int mid) {
        int n = matrix.size();
        int count = 0;
        int row = n - 1;
        int col = 0;

        while (row >= 0 && col < n) {
            if (matrix[row][col] <= mid) {
                count += (row + 1); 
                col++;
            } else {
                row--;
            }
        }
        return count;
    }

    int kthSmallest(vector<vector<int>>& matrix, int k) {
        int n = matrix.size();
        
        int low = matrix[0][0];
        int high = matrix[n-1][n-1];
        int ans = low; 

        while (low <= high) {
            int mid = low + (high - low) / 2;

            if (countLessEqual(matrix, mid) < k) {
                low = mid + 1;
            } else {
                ans = mid;      
                high = mid - 1; 
            }
        }

        return ans; 
    }
};
```
* Optimal Time Complexity: $O(n \log(\text{Max} - \text{Min}))$
* Optimal Space Complexity: $O(1)$

### System Design / Real-Life Impact
**Database Sharding & Indexing:** When data is sharded across multiple sorted databases (similar to rows in this matrix), querying the median or $K^{th}$ percentile element without moving all data to a central server is critical. Binary search on the answer space drastically reduces network I/O.
