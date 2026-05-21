# Making A Large Island (LeetCode 827)

## Problem Statement
You are given an $n \times n$ binary matrix `grid`. You are allowed to change **at most one** `0` to be `1`.

Return the size of the largest island in `grid` after applying this operation. An island is a 4-directionally connected group of `1`s.

---

## Brute Force Approach
The most intuitive approach is to simulate the process for every single possibility.

1. Iterate through every cell in the grid.
2. When you find a `0`, temporarily flip it to a `1`.
3. Run a full Depth-First Search (DFS) or Breadth-First Search (BFS) to find the size of the new island.
4. Track the maximum size found, then flip the `1` back to `0` (backtrack).
5. Repeat for all `0`s.

**Complexity:**
* **Time Complexity:** $\mathcal{O}(N^4)$ where $N$ is the dimension of the grid. There are $\mathcal{O}(N^2)$ zeros in the worst case, and for each zero, we perform a DFS that takes $\mathcal{O}(N^2)$ time. This will result in a Time Limit Exceeded (TLE) error for large grids.
* **Space Complexity:** $\mathcal{O}(N^2)$ for the visited matrix or recursion stack during DFS.

---

## Optimal Approach (Graph Coloring / Component IDs)
Instead of recalculating the island sizes repeatedly, we can precompute and cache the sizes of all existing islands, treating them as distinct components. 

**Step 1: Paint the Islands**
Iterate through the grid. Whenever you find an unvisited `1`, run a DFS to find all connected `1`s. Assign every cell in this island a **unique ID** (starting from `2`, since `0` and `1` are already used). Store the mapping of `Island ID -> Total Area` in a Hash Map.

**Step 2: Evaluate the `0`s**
Iterate through the grid again, this time looking for `0`s. For every `0`, check its 4 immediate neighbors (Up, Down, Left, Right). 
* Use a Hash Set to collect the unique IDs of the neighboring islands (using a set prevents counting the same island twice if it wraps around a `0`).
* Sum the areas of these unique neighboring islands using our Hash Map.
* Add `1` to this sum (representing the flipped `0` itself).

**Step 3: Edge Cases**
If the grid contains no `0`s (it is entirely `1`s), the above step will return `0`. We must keep track of the maximum island size found during Step 1 to handle this gracefully.

**Complexity:**
* **Time Complexity:** $\mathcal{O}(N^2)$. We traverse the grid entirely during the DFS phase, and then traverse it once more to evaluate the `0`s. Fetching and summing from the Hash Map takes $\mathcal{O}(1)$ time.
* **Space Complexity:** $\mathcal{O}(N^2)$ for the Hash Map and the maximum recursion depth of the DFS call stack.

---

## C++ Solution

```cpp
class Solution {
public:
    int largestIsland(vector<vector<int>>& grid) {
        int n = grid.size();
        unordered_map<int, int> area;
        int island_id = 2; // Start from 2 to avoid confusing with 0 (water) and 1 (unvisited land)
        int max_area = 0;
        
        vector<pair<int, int>> dirs = {{0, 1}, {0, -1}, {-1, 0}, {1, 0}};

        // Step 1: DFS to map out all islands and their sizes
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                if (grid[i][j] == 1) {
                    int current_area = dfs(grid, i, j, island_id, dirs);
                    area[island_id] = current_area;
                    max_area = max(max_area, current_area); // Handles the "grid is all 1s" edge case
                    island_id++;
                }
            }
        }

        // Step 2 & 3: Iterate through all 0s, check 4-way neighbors, and calculate potential max
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                if (grid[i][j] == 0) {
                    unordered_set<int> seen_islands;
                    int potential_area = 1; // +1 for flipping this '0' to '1'

                    for (const auto& dir : dirs) {
                        int ni = i + dir.first;
                        int nj = j + dir.second;
                        
                        // If it's a valid neighbor and it is land (id >= 2)
                        if (ni >= 0 && ni < n && nj >= 0 && nj < n && grid[ni][nj] > 1) {
                            seen_islands.insert(grid[ni][nj]);
                        }
                    }

                    // Sum the areas of all UNIQUE adjacent islands
                    for (int id : seen_islands) {
                        potential_area += area[id];
                    }
                    
                    max_area = max(max_area, potential_area);
                }
            }
        }

        return max_area;
    }

private:
    // Helper returns the size of the island while mutating the grid with the ID
    int dfs(vector<vector<int>>& grid, int i, int j, int id, const vector<pair<int, int>>& dirs) {
        if (i < 0 || i >= grid.size() || j < 0 || j >= grid[0].size() || grid[i][j] != 1) {
            return 0;
        }
        
        grid[i][j] = id; // Mark the cell with the unique island ID
        int size = 1;
        
        for (const auto& dir : dirs) {
            size += dfs(grid, i + dir.first, j + dir.second, id, dirs);
        }
        
        return size;
    }
};
```

## Real-Life Scenarios

This algorithm pattern (Connected Component Labeling) is widely used in systems involving spatial data or grid processing:
- Image Segmentation & Computer Vision: Identifying distinct objects in a 2D image array (e.g., finding the sizes of tumors in medical MRI scans, or identifying contiguous pixels in a greenscreen mask).
- Geographic Information Systems (GIS): Evaluating the impact of building infrastructure. For example, calculating the maximum contiguous landmass created if a single bridge is built between two disconnected islands.
- Game Development: Procedural generation algorithms checking terrain walkability or evaluating the structural integrity of connected resource nodes (like a vein of ore in Minecraft).

## Follow-Up: Streaming Updates (Dynamic Grid) 
- What if the grid is no longer static? Imagine an empty grid where pixels of land (1s) are dropping in one by one continuously.
- After every single drop, we must return the size of the largest island.
- Our current approach degrades rapidly here because we would have to recalculate the map areas frequently, leading to heavy performance hits.
- Optimal Architecture Pivot: Disjoint Set Union (DSU) / Union-FindWhen moving from a static state (batch processing) to a dynamic state (streaming updates), the standard graph traversal becomes obsolete.
- We pivot to a DSU architecture:1D Flattening: Treat the 2D grid as a 1D array where the index is calculated as id = r * N + c.
- This allows for a cache-efficient 1D parent array.
- Union by Size: When a new 1 drops at index X, set parent[X] = X and size[X] = 1. Check its 4 neighbors. If a neighbor Y is already land, perform a Union(X, Y). During the union, attach the smaller component to the larger one and update the root's size.
- Path Compression: Ensure every accessed node points directly to the root of its island.Result: Adding a pixel and finding the new maximum island size drops to an amortized $\mathcal{O}(1)$ time complexity per stream event (technically $\mathcal{O}(\alpha(N))$, which is practically constant).
