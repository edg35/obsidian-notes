---
tags: [algorithm, dynamic-programming]
time-complexity: O(m·n)
space-complexity: O(m·n) — optimizable to O(min(m,n))
---
# Longest Common Subsequence

## Overview
Given two strings (or sequences), find the length of their longest subsequence that appears in both — **without reordering characters**. A subsequence doesn't have to be contiguous (unlike a substring). Classic 2D DP problem.

## How It Works
```pseudocode
function lcs(s1, s2):
    m = len(s1), n = len(s2)
    // dp[i][j] = LCS length of s1[0..i-1] and s2[0..j-1]
    dp = 2D array (m+1) x (n+1), initialized to 0

    for i in 1..m:
        for j in 1..n:
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1    // characters match
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])  // skip one

    return dp[m][n]

// To reconstruct the actual subsequence, backtrack from dp[m][n]
```

## Complexity Analysis
| Case | Time | Space |
|------|------|-------|
| Best | O(m·n) | O(min(m,n)) (2-row optimized) |
| Average | O(m·n) | O(m·n) |
| Worst | O(m·n) | O(m·n) |

## When To Use
- Diff tools, version control (finding common base)
- Edit distance variant (LCS is the basis for min insertions/deletions)
- "Longest common subsequence/substring" directly
- DNA sequence alignment

## Related Data Structures
- [[Hash Map]] — for memoization in top-down variant

## Related Algorithms
- [[Dynamic Programming]] — parent technique
- [[Merge Sort]] — both use divide & conquer decomposition on sequences
