# 212. Word Search II

https://leetcode.com/problems/word-search-ii/description/

## The Problem
Given an `m x n` board of characters and a list of strings `words`, return all words on the board. Words are constructed from sequentially adjacent cells (horizontally or vertically).

---

## The Architecture: Memory-Safe Trie + Backtracking
This is the ultimate test of state management and C++ Memory Ownership. 

### The Optimizations:
1. **$O(1)$ String Memory at Leaves:** Instead of passing and concatenating a string parameter through $O(3^L)$ recursive DFS frames, we store the completed string directly inside the leaf `TrieNode`. When we find a valid path, we pluck the string instantly.
2. **Native Deduplication:** Once a word is found, we set the node's string to `""`. This mathematically guarantees we never add the same word twice, eliminating the need for an expensive `unordered_set`.
3. **RAII & Smart Pointers:** We use `std::unique_ptr` to manage the Trie's memory. The Trie automatically destructs and cleans up the Heap when it goes out of scope, guaranteeing zero memory leaks.

---

## The Production Code (C++)
```cpp
#include <vector>
#include <string>
#include <memory>

using namespace std;

class TrieNode {
public:
    string word; 
    vector<unique_ptr<TrieNode>> child; // Exclusive Ownership

    TrieNode() {
        word = "";
        child.resize(26); 
    }
};

class Solution {
private:
    void dfs(int i, int j, vector<vector<char>>& board, TrieNode* root, vector<string>& result) {
        // 1. Physical Bounds, 2. Visited State, 3. Trie Validity
        if (i < 0 || j < 0 || i >= board.size() || j >= board[0].size() || 
            board[i][j] == '#' || !root->child[board[i][j] - 'a']) {
            return;
        }

        char c = board[i][j];
        // .get() extracts the raw observer pointer from the unique_ptr
        root = root->child[c - 'a'].get(); 
        
        // Did we hit a completed word?
        if (root->word != "") { 
            result.push_back(root->word);
            root->word = ""; // Native Deduplication
        }

        board[i][j] = '#'; // Mark Visited

        dfs(i + 1, j, board, root, result);
        dfs(i - 1, j, board, root, result);
        dfs(i, j + 1, board, root, result);
        dfs(i, j - 1, board, root, result);

        board[i][j] = c; // Backtrack
    }

public:
    vector<string> findWords(vector<vector<char>>& board, vector<string>& words) {
        unique_ptr<TrieNode> root = make_unique<TrieNode>();

        for (const string& word : words) {
            TrieNode* curr = root.get(); // Observer pointer
            for (char ch : word) {
                if (!curr->child[ch - 'a']) {
                    curr->child[ch - 'a'] = make_unique<TrieNode>();
                }
                curr = curr->child[ch - 'a'].get();
            }
            curr->word = word; // Store the full word at the leaf
        }

        vector<string> result;
        for (int i = 0; i < board.size(); ++i) {
            for (int j = 0; j < board[0].size(); ++j) {
                dfs(i, j, board, root.get(), result); 
            }
        }

        // 'root' goes out of scope here, recursively destroying the entire Trie.
        return result;
    }
};
```
## Complexity Analysis
1. Time Complexity: $O(M \times N \times 3^L)$ — Where $M \times N$ is the board size and $L$ is the maximum length of a word. At each step, we explore 3 directions (excluding where we came from).
2. Space Complexity: $O(W \times L)$ — For the Trie memory, where $W$ is the number of words.
  
## System Design Context: Memory Leaks & The Observer Pattern

In C++, raw new and delete operations are prone to memory leaks, especially if an exception is thrown mid-execution. In a production database engine, a memory leak during a massive string search would eventually consume all server RAM and cause a crash.By upgrading the data structure to use std::unique_ptr for node ownership while traversing with raw Observer Pointers (TrieNode*), we apply the RAII (Resource Acquisition Is Initialization) paradigm. We guarantee perfect memory safety without sacrificing traverse speed.
