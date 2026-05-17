---
tags: [algorithm, graph, mst]
time-complexity: O(E log V)
space-complexity: O(V)
---
# Prim's Algorithm

## Overview
Greedily builds a **Minimum Spanning Tree (MST)** by growing a connected tree one edge at a time, always picking the cheapest edge that connects a new vertex. Works on undirected weighted graphs.

## How It Works
```pseudocode
function prim(graph, start):
    inMST = {start}
    minHeap = [(0, start, -1)]   // (cost, node, parent)
    totalCost = 0
    mstEdges = []

    while minHeap is not empty and |inMST| < V:
        (cost, node, parent) = heapPop(minHeap)

        if node in inMST: continue

        inMST.add(node)
        totalCost += cost
        if parent != -1: mstEdges.append((parent, node, cost))

        for (neighbor, weight) in graph[node]:
            if neighbor not in inMST:
                heapPush(minHeap, (weight, neighbor, node))

    return totalCost, mstEdges
```

## Complexity Analysis
| Case | Time | Space |
|------|------|-------|
| Best | O(E log V) | O(V) |
| Average | O(E log V) | O(V) |
| Worst | O(E log V) | O(V) |

## When To Use
- Finding minimum cost to connect all nodes (network wiring, road building)
- Dense graphs (Prim's performs better than Kruskal's on dense graphs)
- When you need to grow the tree from a specific start node

## Related Data Structures
- [[Graph]] — undirected weighted input
- [[Heap]] — min-heap selects the cheapest next edge
- [[Trees]] — the output is a spanning tree

## Related Algorithms
- [[Kruskal's Algorithm]] — alternative MST algorithm (better for sparse graphs)
- [[Dijkstra's Algorithm]] — nearly identical implementation; Dijkstra tracks node cost, Prim tracks edge cost
- [[Greedy Algorithm]] — Prim's is a greedy algorithm
