# 218. The Skyline Problem
https://leetcode.com/problems/the-skyline-problem/

## The Problem
A city's skyline is the outer contour of the silhouette formed by all the buildings in that city when viewed from a distance. Given the locations and heights of all the buildings, return the skyline formed by these buildings collectively.
The input is an array of buildings where `buildings[i] = [left, right, height]`. The output must be a list of "key points" `[x, y]` that uniquely define the contour.

---

##  The Architecture: Sweepline + Lazy Deletion Max-Heap
This problem requires marrying two distinct algorithmic concepts: processing chronological events (**Sweepline**) and maintaining a dynamic maximum state (**Max-Heap**).

### 1. The Sweepline Physics & The Negative Hack
We shred each building into two independent events: a `START` event and an `END` event. We then sort these events by their X-coordinate.
**The Edge Cases:** If multiple events happen at the exact same X-coordinate, we must process them in a very specific order to prevent the skyline from "falsely dipping" or creating jagged steps:
1. Starts must be processed before Ends.
2. Taller Starts must be processed before Shorter Starts.
3. Shorter Ends must be processed before Taller Ends.

**The SDE-3 Hack:** We can satisfy all three of these complex sorting rules natively by storing `START` heights as **negative numbers**, and `END` heights as **positive numbers**. A standard `std::sort` will perfectly order the events.

### 2. The Hardware Optimization: Priority Queue vs. Multiset
While a `std::multiset` works for tracking the active building heights, it is backed by a Red-Black Tree. Every node is dynamically allocated on the Heap, causing massive **CPU Cache Misses** during traversal.

Instead, we use a `std::priority_queue` backed by a contiguous `std::vector` to maximize L1/L2 CPU Cache hits. Because a Priority Queue lacks a `.remove()` function, we implement **Lazy Deletion** using an `unordered_map` to track "dead" buildings, only popping them when they bubble up to the absolute top of the queue.

---

### Approach 1, Using Priority Queue
```cpp
#include <vector>
#include <queue>
#include <unordered_map>
#include <algorithm>

using namespace std;

class Solution {
public:
    vector<vector<int>> getSkyline(vector<vector<int>>& buildings) {
        vector<pair<int, int>> events;

        // 1. Shred into events (START = negative height, END = positive height)
        for (const auto& b: buildings) {
            events.push_back({b[0], -b[2]});
            events.push_back({b[1], b[2]});
        }

        // Standard ascending sort perfectly handles all 3 edge-case tie-breakers
        sort(events.begin(), events.end());

        vector<vector<int>> result;
        priority_queue<int> pq; // Max-Heap backed by std::vector
        pq.push(0); // Ground level anchor
        
        unordered_map<int, int> deletedFreq; // Lazy Deletion Tracker
        int prevMaxHeight = 0;

        // 2. Sweep the line from left to right
        for (const auto& event : events) {
            int currentX = event.first;
            int height = event.second;

            if (height < 0) {
                // START EVENT: Push real positive height
                pq.push(-height);
            } else {
                // END EVENT: Mark this building as dead
                deletedFreq[height]++;
            }

            // 3. The Lazy Deletion Engine
            // Clear out any dead buildings that have bubbled to the top
            while (!pq.empty() && deletedFreq[pq.top()] > 0) {
                deletedFreq[pq.top()]--;
                pq.pop();
            }

            // 4. Evaluate the Skyline
            int currentMaxHeight = pq.top();
            
            // If the absolute maximum height changed, record a key point
            if (currentMaxHeight != prevMaxHeight) {
                result.push_back({currentX, currentMaxHeight});
                prevMaxHeight = currentMaxHeight;
            }
        }

        return result;
    }
};
```

### Approach 2: Using Multiset

```cpp
#include <vector>
#include <set>
#include <algorithm>

using namespace std;

class Solution {
public:
    vector<vector<int>> getSkyline(vector<vector<int>>& buildings) {
        // 1. Shred the buildings into independent Sweepline Events
        // Pair structure: {x_coordinate, height}
        vector<pair<int, int>> events;
        for (const auto& b : buildings) {
            events.push_back({b[0], -b[2]}); // START = Negative Height
            events.push_back({b[1], b[2]});  // END = Positive Height
        }

        // 2. Sort events. The negative hack handles all tie-breakers perfectly.
        sort(events.begin(), events.end());

        vector<vector<int>> result;
        
        // 3. The "Active Roster". Multiset keeps heights sorted automatically.
        multiset<int> activeHeights;
        
        // Push 0 so the multiset is never empty (representing ground level)
        activeHeights.insert(0); 
        int prevMaxHeight = 0;

        // 4. Sweep the line from left to right
        for (const auto& event : events) {
            int currentX = event.first;
            int height = event.second;

            if (height < 0) {
                // It's a START event. Add the real (positive) height to the roster.
                activeHeights.insert(-height);
            } else {
                // It's an END event. 
                // CRITICAL: Use .find() to erase only ONE instance of this height.
                // If you just do activeHeights.erase(height), it will delete ALL buildings of that height!
                activeHeights.erase(activeHeights.find(height));
            }

            // 5. What is the current maximum height? (Last element in a multiset)
            int currentMaxHeight = *activeHeights.rbegin();

            // 6. If the skyline changed, we found a key point!
            if (currentMaxHeight != prevMaxHeight) {
                result.push_back({currentX, currentMaxHeight});
                prevMaxHeight = currentMaxHeight;
            }
        }

        return result;
    }
};
```

## Complexity Analysis
1. Time Complexity: Amortized $O(N \log N)$. Shredding and sorting the $2N$ events takes $O(N \log N)$. Inside the loop, pq.push() takes $O(\log N)$. Even though there is a while loop for lazy deletion, every building is pushed and popped exactly once over the lifetime of the algorithm, meaning the total pop operations bound to $O(N \log N)$.
2. Space Complexity: $O(N)$ to store the events array, the contiguous std::vector backing the priority queue, and the hash map.

## System Design Context: Deferred Cleanup (Tombstoning) 
The "Lazy Deletion" pattern used here maps directly to how massive distributed databases (like Cassandra or MongoDB) handle DELETE operations. Physically deleting a record from a dense B-Tree or contiguous disk block in real-time is an expensive $O(N)$ operation. Instead, systems write a fast $O(1)$ Tombstone (just like our deletedFreq map). The database pretends the data is gone for read queries, and physically cleans up the memory later asynchronously during a background compaction process.
