## Problem Statement
Given a square grid of characters where 'S' is the start (bottom-right), 'E' is the end (top-left), 'X' is an obstacle, and digits '1'-'9' represent coins. You can move Up, Left, or Up-Left.Find the maximum sum of coins you can collect on your way to 'E', and the total number of paths that yield this maximum sum. Return the result as an array [maximum_sum, number_of_paths]. If no path exists, return [0, 0]. Both answers must be modulo $10^9 + 7$.

## 1. Brute Force / Queue-Based Approach (SPFA-like)Approach:
We traverse the grid using a queue, starting from 'S'. To avoid pure exponential tree-search behavior, we keep a max_score matrix. We only continue processing a popped state if its running sum is greater than or equal to the recorded max_score for that cell. If a higher score is found for a cell, we overwrite the count and push it back to the queue. If an equal score is found, we aggregate the path counts.

## Drawbacks:
Because a standard queue does not process states in strict topological order, a cell might be popped and process its neighbors, only to be reached later by a better path, forcing it back into the queue for re-evaluation.

```cpp
class Solution {
public:
    vector<int> pathsWithMaxScore(vector<string>& board) {
        int n = board.size();
        long long MOD = 1e9 + 7;

        vector<vector<int>> max_score(n, vector<int>(n, -1));
        vector<vector<int>> path_count(n, vector<int>(n, 0));

        vector<pair<int, int>> dir = {{0, -1}, {-1, 0}, {-1, -1}};

        queue<vector<int>> q;
        
        q.push({n - 1, n - 1, 0});
        max_score[n - 1][n - 1] = 0;
        path_count[n - 1][n - 1] = 1;

        while (!q.empty()) {
            auto v = q.front();
            q.pop();
            
            int i = v[0];
            int j = v[1];
            int summ = v[2];

            // Pruning: If a better path has already reached this cell, discard
            if (summ < max_score[i][j]) {
                continue;
            }

            if (board[i][j] == 'E') {
                continue;
            }

            for (auto [x, y] : dir) {
                int new_i = i + x;
                int new_j = j + y;

                if (new_i < 0 || new_j < 0 || new_i >= n || new_j >= n || board[new_i][new_j] == 'X') {
                    continue;
                }

                int cell_value = (board[new_i][new_j] == 'E') ? 0 : (board[new_i][new_j] - '0');
                int next_sum = summ + cell_value;

                if (next_sum > max_score[new_i][new_j]) {
                    max_score[new_i][new_j] = next_sum;
                    path_count[new_i][new_j] = path_count[i][j]; 
                    q.push({new_i, new_j, next_sum});
                } 
                else if (next_sum == max_score[new_i][new_j]) {
                    path_count[new_i][new_j] = (path_count[new_i][new_j] + path_count[i][j]) % MOD;
                }
            }
        }

        if (max_score[0][0] == -1) {
            return {0, 0};
        }
        return {max_score[0][0], path_count[0][0]};
    }
};
```
### ComplexityTime Complexity:
 - Worst-case $O(V \times E)$ which translates to roughly $O(N^4)$ dynamically, though practically closer to $O(N^2)$ on average due to pruning. Revisits create hidden constants and overhead.
 - Space Complexity: $O(N^2)$ for matrices and up to $O(N^2)$ for the queue.

## 2. Optimal Approach (Bottom-Up DP)

Because movement is strictly bounded to Up, Left, and Up-Left, the grid is a Directed Acyclic Graph (DAG). We can guarantee perfect topological order purely by iterating backward from the start (n-1, n-1) to the end (0, 0).

For every cell, we look at its three possible predecessors (Right, Down, Down-Right). We find the maximum score among reachable predecessors and carry forward the aggregated path counts. We separate the logic of polling dependencies from updating the current cell's state to prevent logic errors.

```cpp
class Solution {
public:
    vector<int> pathsWithMaxScore(vector<string>& board) {
        int n = board.size();
        int MOD = 1e9 + 7;
        
        vector<vector<int>> dp_sum(n, vector<int>(n, -1));
        vector<vector<int>> dp_count(n, vector<int>(n, 0));
        
        dp_sum[n-1][n-1] = 0;
        dp_count[n-1][n-1] = 1;
        
        vector<pair<int, int>> dirs = {{1, 0}, {0, 1}, {1, 1}}; // Down, Right, Down-Right
        
        for (int i = n - 1; i >= 0; i--) {
            for (int j = n - 1; j >= 0; j--) {
                if (board[i][j] == 'X' || (i == n - 1 && j == n - 1)) continue;
                
                int max_prev_sum = -1;
                int current_path_count = 0;
                
                // Phase 1: Poll predecessors
                for (auto [dx, dy] : dirs) {
                    int nx = i + dx;
                    int ny = j + dy;
                    
                    if (nx >= n || ny >= n || dp_sum[nx][ny] == -1) continue;
                    
                    if (dp_sum[nx][ny] > max_prev_sum) {
                        max_prev_sum = dp_sum[nx][ny];
                        current_path_count = dp_count[nx][ny]; // Reset count for new max
                    } else if (dp_sum[nx][ny] == max_prev_sum) {
                        current_path_count = (current_path_count + dp_count[nx][ny]) % MOD;
                    }
                }
                
                // Phase 2: Apply state
                if (current_path_count > 0) {
                    dp_count[i][j] = current_path_count;
                    int cell_val = (board[i][j] == 'E') ? 0 : (board[i][j] - '0');
                    dp_sum[i][j] = (max_prev_sum + cell_val) % MOD;
                }
            }
        }
        
        if (dp_sum[0][0] == -1) return {0, 0};
        
        return {dp_sum[0][0], dp_count[0][0]};
    }
};
```

### Complexity
- Time Complexity: Strict $O(N^2)$. We process each of the $N \times N$ cells exactly once, with exactly 3 constant-time neighbor checks.
- Space Complexity: $O(N^2)$ for the dp_sum and dp_count matrices. (Note: Can be further optimized to $O(N)$ because calculating the current row only depends on the current and previous row).

## 3. Real World Use Cases

- Network Packet Routing (QoS): In distributed network topologies, finding the path with the maximum available bandwidth (max sum) is critical for Quality of Service. Tracking the number of such paths allows routers to utilize Equal-Cost Multi-Path (ECMP) routing, load-balancing traffic across all optimal routes to prevent link congestion.

- Supply Chain & Logistics: When transporting high-value goods, logistics engines optimize for the maximum profit margin route (least tolls, best fuel efficiency). Knowing the total number of mathematically equivalent optimal routes allows the system to easily pivot in case of sudden road closures or vendor disruptions without dropping the profit margin.