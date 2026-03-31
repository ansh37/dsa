# 210. Course Schedule II

https://leetcode.com/problems/course-schedule-ii/description/

## The Problem
Given `numCourses` and a list of `prerequisites` where `prerequisites[i] = [a, b]` indicates that you must take course `b` first if you want to take course `a`. Return the ordering of courses you should take to finish all courses. If it is impossible (due to a cycle), return an empty array.

---

## The Architecture: Topological Sort (3-State DFS)
This is a classic **Directed Acyclic Graph (DAG)** problem. We must order the nodes such that for every directed edge $U \rightarrow V$, node $U$ comes before node $V$ in the ordering.

To solve this, we use a **3-State Depth-First Search (DFS)** to simultaneously build the topological ordering and detect cycles (deadlocks). 

* **State 0 (Unvisited / White):** The node has not been explored.
* **State 1 (Visiting / Gray):** The node is currently in the active DFS recursion stack. If we encounter a neighbor that is also in State 1, we have detected a cyclical dependency (a back-edge).
* **State 2 (Visited / Black):** The node and all of its downstream dependencies have been successfully processed and pushed to the result array.

---

## Code (C++)
```cpp
class Solution {
private:
    // Returns TRUE if a cycle is detected, FALSE if safe
    bool detectCycleDFS(int node, vector<vector<int>>& graph, vector<int>& state, vector<int>& result) {
        if (state[node] == 1) return true; // Cycle detected (Deadlock)
        if (state[node] == 2) return false; // Already fully processed

        // 1. Mark as "Visiting"
        state[node] = 1;

        // 2. Process all dependencies
        for (const int neigh : graph[node]) {
            if (detectCycleDFS(neigh, graph, state, result)) {
                return true;
            }
        }

        // 3. Mark as "Fully Visited" and add to output
        state[node] = 2;
        result.push_back(node);
        
        return false;
    }

public:
    vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
        vector<vector<int>> graph(numCourses);
        for (const auto& edge : prerequisites) {
            graph[edge[1]].push_back(edge[0]);
        }

        vector<int> state(numCourses, 0); 
        vector<int> result;

        for (int i = 0; i < numCourses; ++i) {
            if (state[i] == 0) {
                if (detectCycleDFS(i, graph, state, result)) {
                    return {}; // Cycle detected, impossible to finish
                }
            }
        }

        // Because we pushed to a vector instead of a stack, it is in reverse topological order.
        reverse(result.begin(), result.end());
        return result;
    }
};

```

## Complexity Analysis
- Time Complexity: $O(V + E)$ — We visit every vertex and evaluate every directed edge exactly once.
- Space Complexity: $O(V + E)$ — To store the adjacency list graph, recursion stack, and state array.

## System Design Context: 
Build Tools & Package Managers: Topological Sorting is the mathematical backbone of almost all dependency resolution engines:

1. Build Systems (Webpack / Make / Bazel): When compiling projects, files depend on other files. Build systems construct a Directed Graph of these imports and run a Topological Sort to determine the exact compilation order. Cycle detection (State 1) throws a "Circular Dependency Error".
2. Package Managers (NPM / Maven): Maps out the dependency tree to figure out the exact sequence to download and install libraries so that no package tries to install before its requirements exist.
