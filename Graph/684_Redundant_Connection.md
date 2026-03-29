# 684. Redundant Connection
https://leetcode.com/problems/redundant-connection/

##  The Problem
Given an undirected graph that started as a tree with `N` nodes, exactly one additional edge was added, creating a cycle. Return an edge that can be removed so that the resulting graph is a tree of `N` nodes. If there are multiple answers, return the edge that occurs last in the input.

---

## The Architecture: Disjoint Set Union (Cycle Detection)
While a Depth-First Search (DFS) can detect cycles, running a full $O(V+E)$ traversal every time a new edge is added to a dynamic network is a massive CPU bottleneck. 

The optimal architecture is **Disjoint Set Union (DSU) / Union-Find**.
By maintaining a system of "Root Masters" for connected components, DSU allows us to answer the question, *"Are these two nodes already connected?"* in near $O(1)$ time. If we try to draw a cable between Server A and Server B, and DSU tells us they already share the same Root Master, we have instantly detected a cycle.

### The Optimizations:
1. **Path Compression:** When a node queries its root, it rewires its pointer directly to the absolute root, flattening the tree and making future queries instantaneous.
2. **Union by Rank:** We always attach the smaller network cluster under the larger network cluster to keep the routing tree mathematically shallow.

---

##  The Production Code (C++)
```cpp
class DSU {
private:
    vector<int> parent, rank;
public:
    DSU(int n) {
        parent.resize(n + 1);
        rank.resize(n + 1, 0);
        for (int i = 0; i <= n; i++) parent[i] = i;
    }

    int find(int x) {
        if (parent[x] == x) return x;
        return parent[x] = find(parent[x]); // Path Compression
    }

    bool unite(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);
        
        if (rootX == rootY) return false; // Cycle Detected!

        // Union by Rank
        if (rank[rootX] < rank[rootY]) parent[rootX] = rootY;
        else if (rank[rootX] > rank[rootY]) parent[rootY] = rootX;
        else {
            parent[rootY] = rootX;
            rank[rootX]++;
        }
        return true;
    }
};

class Solution {
public:
    vector<int> findRedundantConnection(vector<vector<int>>& edges) {
        DSU dsu(edges.size());
        
        for (const auto& edge : edges) {
            // If the nodes are already in the same set, this edge creates a loop.
            if (!dsu.unite(edge[0], edge[1])) {
                return edge;
            }
        }
        return {};
    }
};
```

## Complexity Analysis
- Time Complexity: $O(E \times \alpha(V))$ — The Inverse Ackermann function $\alpha(V)$ is practically $O(1)$, making this incredibly fast for real-time edge streaming.
- Space Complexity: $O(V)$ — For the parent and rank tracking arrays.

## System Design Context: STP & Database Deadlocks

1. Networking: Spanning Tree Protocol (STP)

  In enterprise Local Area Networks (LANs), physical redundancy is required (if one switch dies, the network must survive). However, redundant ethernet cables create Broadcast Storms—a packet loops infinitely, consuming all bandwidth and crashing the network.
Network switches use the Spanning Tree Protocol (which relies on this exact spanning-tree mathematics) to detect physical loops and logically disable the redundant ports, preventing the network from melting down.

2. Databases: Deadlock Detection (Wait-For Graphs)

  In distributed databases (like PostgreSQL or MySQL), if Transaction A locks Row 1 and waits for Row 2, while Transaction B locks Row 2 and waits for Row 1, the system freezes. The database engine maintains a "Wait-For Graph" and periodically runs cycle detection. If a cycle (redundant dependency) is found, it kills one of the transactions to heal the system.
