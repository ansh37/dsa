# 1192. Critical Connections in a Network
https://leetcode.com/problems/word-search-ii/description/

## The Problem

Given a network of `n` servers connected by undirected `connections`, find all *critical connections* (Bridges). A critical connection is an edge that, if removed, disconnects the network.
---

## 🧠 The Architecture: Tarjan's Bridge-Finding Algorithm
While Tarjan's Algorithm for Strongly Connected Components (SCC) uses a Stack to track state in *directed* graphs, finding Bridges in an *undirected* graph only requires tracking two temporal states during a Depth-First Search:
1. **Discovery Time (`disc`):** The exact step when the DFS first reached the node.
2. **Lowest Reachable Time (`low`):** The lowest `disc` time a node can reach by taking at most *one* back-edge.

** The Parent Trap:** Because the graph is undirected, traveling $A \rightarrow B$ means $B$ has an immediate edge back to $A$. We must pass the `parent` node into the DFS and explicitly ignore it to prevent false cycles.

### The Bridge Condition
If a child node's lowest reachable node was discovered *after* the current node (`low[neigh] > disc[node]`), it means the child has no back-edges pointing to safety. The child is trapped, making the edge a Bridge.

---

## The Code (C++)
```cpp
class Solution {
private:
    int timer = 0; 
    vector<vector<int>> result;

    void dfs(int node, int parent, vector<vector<int>>& adjList, vector<int>& disc, vector<int>& low) {
        disc[node] = low[node] = timer++;

        for (int neigh : adjList[node]) {
            if (neigh == parent) continue; // Ignore immediate backward path

            if (disc[neigh] == -1) {
                dfs(neigh, node, adjList, disc, low);
                low[node] = min(low[node], low[neigh]);

                // THE BRIDGE CONDITION
                if (low[neigh] > disc[node]) {
                    result.push_back({node, neigh});
                }
            } else {
                // Back-edge found
                low[node] = min(low[node], disc[neigh]);
            }
        }
    }

public:
    vector<vector<int>> criticalConnections(int n, vector<vector<int>>& connections) {
        vector<vector<int>> adjList(n);
        for (const auto& conn : connections) {
            adjList[conn[0]].push_back(conn[1]);
            adjList[conn[1]].push_back(conn[0]);
        }

        vector<int> disc(n, -1);
        vector<int> low(n, -1);
        dfs(0, -1, adjList, disc, low);

        return result;
    }
};
```

## Complexity Analysis
1. Time Complexity: $O(V + E)$ — We visit every vertex and evaluate every edge exactly once.
2. Space Complexity: $O(V + E)$ — To store the Adjacency List graph, plus $O(V)$ for the recursion stack and the temporal arrays.
  
## System Design Context: High-Availability (HA) Infrastructure

This maps directly to Site Reliability Engineering (SRE). When designing AWS VPCs, multi-region clusters, or power grids, any edge returned by this algorithm represents a Single Point of Failure (SPOF). By identifying these critical connections, infrastructure teams know exactly where to lay redundant cables to upgrade the network into a highly available (HA), fault-tolerant system.
