# 140. Word Break II
https://leetcode.com/problems/word-break-ii/description/

## The Problem
Given a string `s` and a dictionary of strings `wordDict`, add spaces in `s` to construct a sentence where each word is a valid dictionary word. Return all such possible sentences in any order.

---

## The Architecture: Trie + Memoized DFS
Running pure backtracking explodes to $O(2^N)$ time complexity. We optimize this using two Senior-level architectures:

1. **The Trie (Prefix Tree):** Instead of generating expensive substrings and hashing them at every step, we store the dictionary in a Trie. We traverse the string character-by-character using $O(1)$ pointer jumps.
2. **Memoization:** We cache the valid suffixes for each index `i`. If we reach index `i` again from a different path, we instantly return the cached sentences.

** Pointer Semantics Warning:** Trees must be built dynamically on the Heap using pointers (`TrieNode*`). Creating node objects on the Stack and assigning them by value (`TrieNode node = root`) causes Object Slicing and destroys the tree structure.

---

## Code (C++)
```cpp
class TrieNode {
public:
    unordered_map<char, TrieNode*> children;
    bool is_Word;
    TrieNode() { is_Word = false; }
};

class Solution {
private:
    TrieNode* root;
    unordered_map<int, vector<string>> memo;

    vector<string> dfs(int idx, const string& s) {
        if (idx == s.size()) return {""}; 
        if (memo.count(idx)) return memo[idx];

        vector<string> validSentences;
        TrieNode* curr = root;

        for (int i = idx; i < s.size(); ++i) {
            char ch = s[i];
            if (!curr->children.count(ch)) break;
            curr = curr->children[ch];

            if (curr->is_Word) {
                string word = s.substr(idx, i - idx + 1);
                vector<string> suffixes = dfs(i + 1, s);

                for (const string& suffix : suffixes) {
                    if (suffix.empty()) validSentences.push_back(word);
                    else validSentences.push_back(word + " " + suffix);
                }
            }
        }
        return memo[idx] = validSentences;
    }

public:
    vector<string> wordBreak(string s, vector<string>& wordDict) {
        root = new TrieNode();
        for (const string& word : wordDict) {
            TrieNode* curr = root;
            for (char ch : word) {
                if (!curr->children.count(ch)) curr->children[ch] = new TrieNode();
                curr = curr->children[ch];
            }
            curr->is_Word = true;
        }
        return dfs(0, s);
    }
};
```

## Complexity Analysis

1. Time Complexity: $O(N^2 + 2^N)$ — In the worst case (e.g., "aaaaaa", dict=["a", "aa"]), the number of valid sentences is exponential. The Trie traversal itself is highly optimized.
2. Space Complexity: $O(N \times L + 2^N)$ — For the Trie storage ($N$ words of length $L$) and the memoization cache.

## System Design Context: MongoDB Text Indexes
Using a Hash Set requires generating a brand new substr string for every single length combination, thrashing the memory allocator. A Trie completely eliminates substring generation during the search phase. This maps directly to how MongoDB Text Indexes and Elasticsearch optimize prefix searches and wildcard text matching under the hood.
