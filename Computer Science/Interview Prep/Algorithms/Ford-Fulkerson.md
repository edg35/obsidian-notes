---
tags: [algorithm, graph, network-flow]
time-complexity: O(E · max_flow)
space-complexity: O(V+E)
---
# Ford-Fulkerson

## Overview
Finds the **maximum flow** from a source node to a sink node in a flow network. Repeatedly finds augmenting paths (paths with remaining capacity) and pushes flow along them until no more augmenting paths exist. The max-flow equals the min-cut (Max-Flow Min-Cut Theorem).

## How It Works
```pseudocode
function fordFulkerson(graph, source, sink):
    // graph[u][v] = capacity of edge u->v (residual graph)
    maxFlow = 0

    while augmenting path exists from source to sink:
        // BFS (Edmonds-Karp) or DFS to find path
        path = bfs(graph, source, sink)
        if not path: break

        // Find bottleneck (min capacity along path)
        pathFlow = min(residual capacity of each edge in path)

        // Update residual capacities
        for each edge (u, v) in path:
            graph[u][v] -= pathFlow
            graph[v][u] += pathFlow   // reverse edge

        maxFlow += pathFlow

    return maxFlow

// Edmonds-Karp variant uses BFS → guarantees O(VE²) regardless of flow values
```

## Complexity Analysis
| Case | Time | Space |
|------|------|-------|
| Best | O(E) | O(V+E) |
| Average | O(E · max_flow) | O(V+E) |
| Worst | O(E · max_flow) | O(V+E) |

(Edmonds-Karp with BFS: O(VE²) — independent of max flow value)

## When To Use
- Maximum flow / minimum cut problems
- Bipartite matching (model as flow network)
- Task assignment, network throughput, circulation problems
- "Maximum number of edge-disjoint paths" between two nodes

## Related Data Structures
- [[Graph]] — residual graph is the key data structure
- [[Queue]] — BFS (Edmonds-Karp variant) uses a queue

## Related Algorithms
- [[Breadth-First Search]] — Edmonds-Karp uses BFS for path finding
- [[Depth-First Search]] — original Ford-Fulkerson uses DFS
