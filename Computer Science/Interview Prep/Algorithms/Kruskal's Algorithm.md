---
tags: [algorithm, graph, mst]
time-complexity: O(E log E)
space-complexity: O(V)
---
# Kruskal's Algorithm

## Overview
Builds a **Minimum Spanning Tree (MST)** by sorting all edges by weight and greedily adding the cheapest edge that doesn't form a cycle. Uses [[Union-Find]] to detect cycles efficiently.

## How It Works
```pseudocode
function kruskal(graph):
    sort all edges by weight ascending
    uf = UnionFind(V)
    mst = []
    totalCost = 0

    for (u, v, weight) in edges:
        if uf.find(u) != uf.find(v):    // no cycle
            uf.union(u, v)
            mst.append((u, v, weight))
            totalCost += weight
            if len(mst) == V - 1: break  // MST complete

    return totalCost, mst
```

## Complexity Analysis
| Case | Time | Space |
|------|------|-------|
| Best | O(E log E) | O(V) |
| Average | O(E log E) | O(V) |
| Worst | O(E log E) | O(V) |

(Dominated by sorting; Union-Find operations are near O(1) with path compression.)

## When To Use
- Finding minimum cost to connect all nodes (same problem class as Prim's)
- Sparse graphs (fewer edges → sorting is faster)
- When edges are already sorted or nearly sorted
- When Union-Find is already in play (e.g., network connectivity problems)

## Related Data Structures
- [[Graph]] — edge-list input; works on undirected weighted graphs

## Related Algorithms
- [[Union-Find]] — cycle detection backbone of this algorithm
- [[Prim's Algorithm]] — alternative MST algorithm (better for dense graphs)
- [[Greedy Algorithm]] — Kruskal's is a greedy algorithm
