---
tags: [algorithm, graph, shortest-path]
time-complexity: O(VE)
space-complexity: O(V)
---
# Bellman-Ford

## Overview
Finds shortest paths from a single source in a weighted graph that **may contain negative-weight edges**. Also detects negative-weight cycles. Slower than Dijkstra but more general.

## How It Works
```pseudocode
function bellmanFord(graph, source):
    dist[source] = 0
    dist[all others] = ∞

    // Relax all edges V-1 times
    repeat V-1 times:
        for each edge (u, v, weight) in graph:
            if dist[u] + weight < dist[v]:
                dist[v] = dist[u] + weight

    // Check for negative-weight cycles
    for each edge (u, v, weight) in graph:
        if dist[u] + weight < dist[v]:
            return "negative cycle detected"

    return dist
```

## Complexity Analysis
| Case | Time | Space |
|------|------|-------|
| Best | O(E) | O(V) |
| Average | O(VE) | O(V) |
| Worst | O(VE) | O(V) |

## When To Use
- Graph has negative edge weights (e.g., currency arbitrage, debt problems)
- Need to detect negative-weight cycles
- Graph is sparse and Dijkstra's heap overhead isn't worth it
- If no negative weights → prefer [[Dijkstra's Algorithm]]

## Related Data Structures
- [[Graph]] — edge-list representation is most natural here

## Related Algorithms
- [[Dijkstra's Algorithm]] — faster alternative when no negative weights
- [[Topological Sort]] — for DAGs, DP on topological order is even faster (O(V+E))
