# 129. Sum Root to Leaf Numbers

## The Problem
Given the `root` of a binary tree containing digits from `0` to `9` only, each root-to-leaf path represents a number. Return the total sum of all root-to-leaf numbers.

## The Architecture (Base-10 DFS)
Instead of passing values *up* from the leaves, we pass an accumulating mathematical state *down* from the root. As we traverse deeper, we shift the previous sum left by one base-10 digit and add the current node's value: `(current_sum * 10) + node->val`.

## The Code (C++)
```cpp
class Solution {
public:
    int sumNumbers(TreeNode* root) {
        int final_sum = 0;
        dfs(root, 0, final_sum); 
        return final_sum;
    }
    
private:
    void dfs(TreeNode* node, int current_sum, int& final_sum) {
        if (!node) return;
        
        current_sum = (current_sum * 10) + node->val;
        
        if (!node->left && !node->right) {
            final_sum += current_sum;
            return; 
        }
        
        dfs(node->left, current_sum, final_sum);
        dfs(node->right, current_sum, final_sum);
    }
};
```
## Time Complexity: O(N) — We visit every node exactly once.
## Space Complexity: O(H) — The recursion stack uses space proportional to the height of the tree.

## System Design & Multi-Threading (MT) Context
When performing DFS to aggregate data across a tree, there are two primary architectural patterns to consider for scale:

The Accumulator Pattern (Used Above): Passing a shared tracking variable (final_sum) by reference. It is incredibly intuitive but not thread-safe.
If we parallelized this across multiple CPU threads, both branches would try to write to final_sum simultaneously, requiring a Mutex lock and creating a 
massive bottleneck.

The Pure Function Pattern: Returning the integer back up the tree (return dfs(left) + dfs(right)). Because there is no shared external memory, the left 
branch and right branch can be processed on completely separate threads or distributed servers without any locking mechanisms.

```cpp
void dfs(TreeNode* node, int current_sum, int& final_sum) {
    if (!node) return;
    current_sum = (current_sum * 10) + node->val;
    
    if (!node->left && !node->right) {
        final_sum += current_sum; // Mutating shared state
        return; 
    }
    dfs(node->left, current_sum, final_sum);
    dfs(node->right, current_sum, final_sum);
}
