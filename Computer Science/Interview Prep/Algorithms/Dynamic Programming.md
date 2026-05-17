---
tags: [algorithm, dynamic-programming]
time-complexity: varies by problem
space-complexity: varies by problem
---
# Dynamic Programming

## Overview
An algorithmic technique for solving problems by breaking them into **overlapping subproblems** and storing results to avoid redundant computation. Applies when a problem has **optimal substructure** (optimal solution built from optimal sub-solutions) and **overlapping subproblems**.

## How It Works
```pseudocode
// Top-down (Memoization)
memo = {}
function solve(state):
    if state in memo: return memo[state]
    if base_case(state): return base_value
    result = combine(solve(subproblem1), solve(subproblem2), ...)
    memo[state] = result
    return result

// Bottom-up (Tabulation)
dp = array of size [subproblem space], initialized to base cases
for state in topological order of subproblems:
    dp[state] = combine(dp[subproblem1], dp[subproblem2], ...)
return dp[target]
```

## Complexity Analysis
| Case | Time | Space |
|------|------|-------|
| Best | O(subproblem count) | O(subproblem count) |
| Average | O(subproblem count × transition cost) | O(subproblem count) |
| Worst | O(subproblem count × transition cost) | O(subproblem count) |

## When To Use
- Problem asks for min/max/count/exists over all possibilities → think DP
- Recursive solution has repeated sub-calls (draw the recursion tree — do branches overlap?)
- Keywords: "how many ways", "minimum cost", "longest/shortest subsequence", "can you reach"
- Greedy doesn't work → try DP

**Common DP patterns:**
- 1D DP: Fibonacci, climbing stairs, house robber
- 2D DP: [[Longest Common Subsequence]], edit distance, grid paths
- Interval DP: matrix chain multiplication, burst balloons
- Knapsack: [[Knapsack]], subset sum, coin change
- State machine DP: stock trading problems

## Related Data Structures
- [[Hash Map]] — memoization cache in top-down approach

## Related Algorithms
- [[Knapsack]] — classic DP problem
- [[Longest Common Subsequence]] — classic 2D DP problem
- [[Merge Sort]] — divide & conquer (related but distinct from DP)
- [[Backtracking]] — DP is backtracking + memoization
