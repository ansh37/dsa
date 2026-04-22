## 2452. Words Within Two Edits of Dictionary
LeetCode Link: https://leetcode.com/problems/words-within-two-edits-of-dictionary/

## The Problem
You are given two string arrays, `queries` and `dictionary`. All words in each array comprise lowercase English letters and have the same length.
For each `query` in `queries`, you can form a word from the `dictionary` if you can change at most **two characters** in the `query` to match a word in the `dictionary`.
Return an array containing all words from `queries` that can be matched.

**Constraints:**
- `1 <= queries.length, dictionary.length <= 100`
- `1 <= queries[i].length, dictionary[j].length <= 100`

## Architecture: Pragmatism vs. Scalability

This problem presents a classic L5 system design choice: Do you build for the immediate constraints, or do you overengineer for hypothetical scale?

**1. The Pragmatic Approach (Brute Force):**
Given the incredibly small constraints ($100 \times 100 \times 100$), the maximum number of operations is $1,000,000$. A modern CPU can execute this in ~1 millisecond. A brute-force iteration using nested loops requires $O(1)$ auxiliary space, avoids all memory allocation overhead, and is functionally the most optimal solution for these specific bounds.

**2. The Scalable Approach (Trie + DFS):**
If the dictionary contained 100,000 words, brute force would fail. The scalable architecture requires building a **Trie** from the dictionary. We then run a Depth First Search (DFS) for each query.
The paradigm shift here is **Cost-Based Pruning (Short-Circuiting):**
- As we traverse the Trie, we track the number of `edits` (mismatches).
- If `edits > 2`, we immediately prune that branch of the DFS.
- If we successfully reach the end of a word with `edits <= 2`, we return `true` immediately, pushing the query to our answer array and skipping all remaining Trie branches to prevent duplicate entries.



## Approaches

| Approach | Time Complexity | Space Complexity | Why it fails/succeeds |
| :--- | :--- | :--- | :--- |
| **Pragmatic Iteration** | **$O(Q \cdot D \cdot L)$** | **$O(1)$** | **Best for these constraints.** Avoids pointer chasing, recursion overhead, and heap allocations. Executes instantly. |
| **Trie + DFS Pruning** | **$O(D \cdot L + Q \cdot 26^2 \cdot L)$** | **$O(D \cdot L)$** | **Best for massive scale.** While overkill for 100 words, it is the mathematically correct system architecture if the dictionary is huge, as it groups common prefixes. |

## C++ Code: Option 1 - Pragmatic Brute Force (Constraint-Optimized)

```cpp
#include <vector>
#include <string>

using namespace std;

class Solution {
public:
    vector<string> twoEditWords(vector<string>& queries, vector<string>& dictionary) {
        vector<string> ans;
        
        for (const string& query : queries) {
            for (const string& word : dictionary) {
                int edits = 0;
                for (int i = 0; i < query.size(); ++i) {
                    if (query[i] != word[i]) {
                        edits++;
                    }
                    if (edits > 2) break; // Prune early
                }
                
                if (edits <= 2) {
                    ans.push_back(query);
                    break; // Move to the next query immediately
                }
            }
        }
        return ans;
    }
};
```

## C++ Code: Option 2 - Trie + DFS (Scale-Optimized)

```cpp
#include <vector>
#include <string>

using namespace std;

class TrieNode {
public:
    vector<TrieNode*> child = vector<TrieNode*>(26, nullptr);
    bool end = false;
};

class Solution {
private:
    void insert(const string& str, TrieNode* root) {
        TrieNode* curr = root;
        for(char ch : str) {
            if (!curr->child[ch - 'a']) {
                curr->child[ch - 'a'] = new TrieNode();
            }
            curr = curr->child[ch - 'a'];
        }
        curr->end = true;
    }

    bool dfs(int idx, const string& query, TrieNode* node, int edits) {
        if (edits > 2) return false;

        if (idx == query.size()) {
            return node->end;
        }

        for (int i = 0; i < 26; ++i) {
            if (node->child[i]) {
                int cost = (i != (query[idx] - 'a')) ? 1 : 0;
                // Short-circuit the moment we find ONE valid path
                if (dfs(idx + 1, query, node->child[i], edits + cost)) {
                    return true; 
                }
            }
        }
        return false;
    }

public:
    vector<string> twoEditWords(vector<string>& queries, vector<string>& dictionary) {
        TrieNode* root = new TrieNode();
        for(const string& word : dictionary) {
            insert(word, root);
        }

        vector<string> ans;
        for (const string& query : queries) {
            if (dfs(0, query, root, 0)) {
                ans.push_back(query);
            }
        }

        return ans;
    }
};
```

## Real-World Use Case
### Search Engine Autocorrect (Fuzzy Matching)
When a user types a query into Google or Amazon search, they often make typos. The backend search engine does not search for the exact misspelled string. Instead, it checks the typed string against a massive dictionary of known valid search terms (often stored in a highly distributed Trie or Finite State Transducer). By allowing a "Levenshtein distance" (edit distance) of 1 or 2, the engine can instantly suggest "Did you mean: *correct word*?" without doing a full table scan of the database.
