---
tags: [algorithm, dynamic-programming]
time-complexity: O(n·W)
space-complexity: O(n·W) — optimizable to O(W)
---
# Knapsack

## Overview
The **0/1 Knapsack** problem: given `n` items each with a weight and value, and a bag of capacity `W`, find the maximum value you can carry. Each item can be taken at most once. A foundational DP problem that patterns many interview questions.

## How It Works
```pseudocode
function knapsack(weights, values, W):
    n = len(weights)
    // dp[i][w] = max value using first i items with capacity w
    dp = 2D array (n+1) x (W+1), initialized to 0

    for i in 1..n:
        for w in 0..W:
            // don't take item i
            dp[i][w] = dp[i-1][w]
            // take item i (if it fits)
            if weights[i-1] <= w:
                dp[i][w] = max(dp[i][w],
                               dp[i-1][w - weights[i-1]] + values[i-1])

    return dp[n][W]

// Space-optimized: use 1D dp array, iterate w backwards
for i in 1..n:
    for w in W..weights[i-1]:   // reverse to avoid using item twice
        dp[w] = max(dp[w], dp[w - weights[i-1]] + values[i-1])
```

## Complexity Analysis
| Case | Time | Space |
|------|------|-------|
| Best | O(n·W) | O(W) (1D optimized) |
| Average | O(n·W) | O(n·W) |
| Worst | O(n·W) | O(n·W) |

## When To Use
- "Pick a subset to maximize value under a constraint" → 0/1 Knapsack
- Variant: unbounded knapsack (items can repeat) → iterate `w` forward
- Variant: subset sum (can we hit exactly W?) → same DP, boolean values
- Variant: coin change, partition equal subset sum

## Related Data Structures
- [[Hash Map]] — for memoization in top-down variant

## Related Algorithms
- [[Dynamic Programming]] — parent technique
- [[Backtracking]] — brute-force baseline before optimizing with DP
