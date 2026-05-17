---
tags: [algorithm, graph, shortest-path]
time-complexity: O((V+E) log V)
space-complexity: O(V)
---
# Dijkstra's Algorithm

## Overview
Finds the shortest path from a single source node to all other nodes in a weighted graph with **non-negative edge weights**. The classic interview algorithm for "minimum cost to reach X."

## How It Works
```pseudocode
function dijkstra(graph, source):
    dist[source] = 0
    dist[all others] = ∞
    minHeap = [(0, source)]

    while minHeap is not empty:
        (cost, node) = heapPop(minHeap)

        if cost > dist[node]: continue   // stale entry

        for (neighbor, weight) in graph[node]:
            newDist = dist[node] + weight
            if newDist < dist[neighbor]:
                dist[neighbor] = newDist
                heapPush(minHeap, (newDist, neighbor))

    return dist
```

## Complexity Analysis
| Case | Time | Space |
|------|------|-------|
| Best | O((V+E) log V) | O(V) |
| Average | O((V+E) log V) | O(V) |
| Worst | O((V+E) log V) | O(V) |

## When To Use
- "Cheapest/shortest path" in a weighted graph with no negative edges
- Network routing, maps, word-ladder cost problems
- If all edge weights are equal → use [[Breadth-First Search]] instead
- If negative weights exist → use [[Bellman-Ford]] instead

## Related Data Structures
- [[Graph]] — the input structure
- [[Heap]] — min-heap drives the greedy selection
- [[Queue]] — conceptual analogy; BFS is the unweighted version

## Related Algorithms
- [[Breadth-First Search]] — unweighted equivalent
- [[Bellman-Ford]] — handles negative weights
- [[Prim's Algorithm]] — nearly identical structure, builds MST instead
