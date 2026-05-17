---
tags: [algorithm, graph]
time-complexity: O(V+E)
space-complexity: O(V)
---
# Topological Sort

## Overview
Produces a linear ordering of nodes in a **Directed Acyclic Graph (DAG)** such that for every directed edge u → v, node u appears before v. Used for dependency resolution, build systems, course scheduling.

## How It Works
```pseudocode
// Kahn's Algorithm (BFS / in-degree approach)
function topoSort(graph):
    compute inDegree[node] for all nodes
    queue = all nodes where inDegree == 0
    result = []

    while queue is not empty:
        node = dequeue(queue)
        result.append(node)
        for neighbor in graph[node]:
            inDegree[neighbor] -= 1
            if inDegree[neighbor] == 0:
                enqueue(queue, neighbor)

    if len(result) != V: return "cycle detected"
    return result

// DFS approach: append node to stack after visiting all neighbors,
// then reverse the stack.
```

## Complexity Analysis
| Case | Time | Space |
|------|------|-------|
| Best | O(V+E) | O(V) |
| Average | O(V+E) | O(V) |
| Worst | O(V+E) | O(V) |

## When To Use
- Task/course scheduling with prerequisites
- Build system dependency ordering
- Detecting cycles in a directed graph (if topo sort can't complete, a cycle exists)
- DP on DAGs (process nodes in topo order)

## Related Data Structures
- [[Graph]] — must be a DAG
- [[Queue]] — used in Kahn's BFS approach
- [[Stack]] — used in the DFS approach

## Related Algorithms
- [[Depth-First Search]] — the DFS post-order variant
- [[Breadth-First Search]] — Kahn's algorithm is BFS-based
- [[Bellman-Ford]] — on DAGs, shortest paths can be solved in O(V+E) via topo order
