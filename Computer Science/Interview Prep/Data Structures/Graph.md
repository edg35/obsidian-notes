### Introduction to Graphs as a Data Structure

A graph is a data structure used to represent relationships between objects. It consists of:

- **Vertices (or [[Node]]):** These are the objects that are connected.
- **Edges:** These are the connections between the vertices.

Graphs can be **directed** (where edges have a direction) or **undirected** (where edges don't have a direction). They can also be **weighted** (where edges have associated weights) or **unweighted**.

### Types of Graphs

1. **Directed Graph (Digraph):** The edges have a direction, indicating a one-way relationship.
2. **Undirected Graph:** The edges have no direction, indicating a two-way relationship.
3. **Weighted Graph:** The edges have weights, representing costs or distances.
4. **Unweighted Graph:** The edges have no weights.

### Representing a Graph in Java

There are several ways to represent a graph in Java. Two common methods are:

1. **Adjacency Matrix:**
    
    - A 2D array where `matrix[i][j]` is `1` (or the edge weight) if there's an edge from vertex `i` to vertex `j`, otherwise it's `0`.
2. **Adjacency List:**
    
    - An array or list of lists, where each index represents a vertex and the list at that index contains all vertices adjacent to it.

### Example: Graph Representation in Java

Here's a simple example of representing an undirected, unweighted graph using an adjacency list:

```java
import java.util.ArrayList;
import java.util.List;

class Graph {
    private int vertices; // Number of vertices
    private List<List<Integer>> adjList; // Adjacency list

    // Constructor
    public Graph(int vertices) {
        this.vertices = vertices;
        adjList = new ArrayList<>(vertices);

        for (int i = 0; i < vertices; i++) {
            adjList.add(new ArrayList<>());
        }
    }

    // Add edge to the graph
    public void addEdge(int src, int dest) {
        adjList.get(src).add(dest); // Add edge from src to dest
        // Add edge from dest to src (since the graph is undirected)
        adjList.get(dest).add(src);
    }

    // Print the graph
    public void printGraph() {
        for (int i = 0; i < vertices; i++) {
            System.out.print("Vertex " + i + ":");
            for (Integer edge : adjList.get(i)) {
                System.out.print(" -> " + edge);
            }
            System.out.println();
        }
    }

    public static void main(String[] args) {
        Graph graph = new Graph(5);

        graph.addEdge(0, 1);
        graph.addEdge(0, 4);
        graph.addEdge(1, 2);
        graph.addEdge(1, 3);
        graph.addEdge(1, 4);
        graph.addEdge(2, 3);
        graph.addEdge(3, 4);

        graph.printGraph();
    }
}
```

### Explanation:

- **Vertices:** The graph is initialized with a specified number of vertices.
- **Adjacency List:** For each vertex, we maintain a list of adjacent vertices.
- **addEdge Method:** Adds an edge between two vertices. Since this graph is undirected, we add the edge in both directions.
- **printGraph Method:** Outputs the adjacency list representation of the graph.

### Algorithms

- [[Depth-First Search]]
- [[Breadth-First Search]]


### Examples

1.  DepthFirstPrint function

```java
public void depthFirstPrint(int startVertex) { 
	boolean[] visited = new boolean[vertices];
	depthFirstPrintHelper(startVertex, visited); 
	// Start the DFS from the startVertex 
} 

// Helper function for DFS
private void depthFirstPrintHelper(int vertex, boolean[] visited) {
	visited[vertex] = true; 
	// Mark the current vertex as visited 
	System.out.print(vertex + " "); 
	// Print the current vertex 
	// Recur for all adjacent vertices 
	for (int adjacentVertex : adjList.get(vertex)) { 
		if (!visited[adjacentVertex]) { 
			depthFirstPrintHelper(adjacentVertex, visited); 
		} 
	} 
}
```

2. Write a method, _hasPath_, that takes in an object representing the adjacency list of a directed acyclic graph and two nodes (_src_, _dst_). The method should return a boolean indicating whether or not there exists a directed path between the _source_ and _destination_ nodes.

```java
class Solution {
	public static boolean hasPath(
	Map<String, List<String>> graph, String src, String dst) 
{
    if (src == dst) return true;

    for(String node: graph.get(src)) {
      if (hasPath(graph, node, dst)) return true;
    }
  
    return false;
  }
}
```

3. Write a method, _undirectedPath_, that takes in a list of edges for an undirected graph and two nodes (_nodeA_, _nodeB_). The method should return a boolean indicating whether or not there exists a path between _nodeA_ and _nodeB_.

```java
import java.util.List;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;

class Source {
  public static boolean undirectedPath(List<List<String>> edges, String nodeA, String nodeB) {
    Map<String, List<String>> graph = buildGraph(edges);
    return dfs(graph, nodeA, nodeB, new HashSet<>());
  }
  
  public static boolean dfs(Map<String, List<String>> graph, String src, String dst, HashSet<String> visited) {
    if (src == dst) {
      return true;
    }
    
    if (visited.contains(src)) {
      return false;
    }
    visited.add(src);
    
    for (String neighbor : graph.get(src)) {
      if (dfs(graph, neighbor, dst, visited)) {
        return true;
      }
    }
    
    return false;
  }
  
  public static Map<String, List<String>> buildGraph(List<List<String>> edges) {
    Map<String, List<String>> map = new HashMap<>();
    for (List<String> edge : edges) {
      String a = edge.get(0);
      String b = edge.get(1);
      if (!map.containsKey(a)) {
          map.put(a, new ArrayList<>());
      }
      if (!map.containsKey(b)) {
          map.put(b, new ArrayList<>());
      }
      map.get(a).add(b);
      map.get(b).add(a);
    }
    return map;
  }

  public static void run() {
    // this function behaves as `main()` for the 'run' command
    // you may sandbox in this function , but should not remove it
  }
}
```

4. There is a **bi-directional** graph with `n` vertices, where each vertex is labeled from `0` to `n - 1` (**inclusive**). The edges in the graph are represented as a 2D integer array `edges`, where each `edges[i] = [ui, vi]` denotes a bi-directional edge between vertex `ui` and vertex `vi`. Every vertex pair is connected by **at most one** edge, and no vertex has an edge to itself. You want to determine if there is a **valid path** that exists from vertex `source` to vertex `destination`.
   
```java
class Solution {

	public boolean validPath(
	int n, int[][] edges, int source, int destination) {
	
		List<List<Integer>> graph = makeGraph(edges, n);
		Queue<Integer> queue = new LinkedList<>();		
		boolean[] visited = new boolean[n];		
		queue.add(source);		
		visited[source] = true;
				
		while (!queue.isEmpty()) {
			int curr = queue.poll();
			if (curr == destination) return true;
		
			for (int neighbor: graph.get(curr)){
			
				if (!visited[neighbor]){
					visited[neighbor] = true;
					queue.add(neighbor);
				}
			}	
		}

		return false;
	}

	public List<List<Integer>> makeGraph(int[][] edges, int n) {
		List<List<Integer>> graph = new ArrayList<>();
		
		for (int i = 0; i < n; i++) {
			graph.add(new ArrayList<>());
		}

		for (int[] e: edges) {
			graph.get(e[0]).add(e[1]);
			graph.get(e[1]).add(e[0]);
		}
		
		return graph;	
	}
}
```

5. Write a method, _connectedComponentsCount_, that takes in the adjacency list of an undirected graph. The method should return the number of connected components within the graph.

```java
import java.util.Map;
import java.util.List;
import java.util.HashSet;
import java.util.Set;

class Source {
  public static int connectedComponentsCount(
  Map<Integer, List<Integer>> graph) 
  {
    int count = 0;
    Set<Integer> visited = new HashSet<>();
    
    for (int key: graph.keySet()) {
      if (dfs(graph, key, visited)) count ++;
    }

    return count;
  }

  public static boolean dfs(
  Map<Integer, List<Integer>> graph, int node, Set<Integer> visited) 
  {
    if(visited.contains(node)) return false;
    visited.add(node);
    
    for (int neighbor: graph.get(node)) {
      dfs(graph, neighbor, visited);
    }

    return true;
  }
}
```

6. Write a method, _largestComponent_, that takes in the adjacency list of an undirected graph. The method should return the size of the largest connected component in the graph.
   
```java
import java.util.Map;
import java.util.List;
import java.util.Set;
import java.util.HashSet;

class Source {
  public static int largestComponent(
	  Map<Integer, List<Integer>> graph) 
  {
    int size = 0;
    Set<Integer> visited = new HashSet<>();

    for (int node: graph.keySet()) {
      int component = dfs(graph, node, visited);
      size = Math.max(component, size);
    }
    
    return size;
  }

  public static int dfs(
	  Map<Integer, List<Integer>> graph, int node, Set<Integer> visited) 
  {
    if (visited.contains(node)) return 0;
    visited.add(node);
    int total = 1;

    for (int neighbor: graph.get(node)) {
      total += dfs(graph, neighbor, visited);
    }

    return total;
  }
}  
```

## Used In
- [[Breadth-First Search]]
- [[Depth-First Search]]
- [[Dijkstra's Algorithm]]
- [[Bellman-Ford]]
- [[Prim's Algorithm]]
- [[Kruskal's Algorithm]]
- [[Topological Sort]]
- [[Ford-Fulkerson]]
