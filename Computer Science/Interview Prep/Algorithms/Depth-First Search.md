---
Time Complexity: O(V+E)
Type: Both
Chapter:
---
**Template**:

```java
public boolean[] dfs(int start) { 
	boolean[] isVisited = new boolean[adjVertices.size()]; 
	return dfsRecursive(start, isVisited); 
} 

private boolean[] dfsRecursive(int current, boolean[] isVisited) { 

	isVisited[current] = true;
	visit(current); 
	
	for (int dest : adjVertices.get(current)) { 
		if (!isVisited[dest]) dfsRecursive(dest, isVisited);
	} 
	
	return isVisited;
}

```


> [!NOTE] DFS
> The algorithm starts at the root node (in the case of a graph, you can use any random node as the root node) and examines each branch as far as possible before backtracking.
> 
> This algorithm makes use of a [[Stack]] data structure to keep track of node.

- While his algorithm is not that useful, it is typically augmented with other algorithms.
- Used for determining connectivity, of find bridges/ articulation points.
- Min Spanning tree
- Find Cycles in a graph
- Check if a graph is bipartite
- Find strongly connected components
- Topological sort of node
- Generate mazes

![File:Depth-First-Search.gif - Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/7/7f/Depth-First-Search.gif)

**Examples**:

1. Connected components: Generally, you will start at a node and then mark them as part of them same component using id

```java
int n // number of nodes in graph
Graph g // adjecency list representing graph
int count = 0;
int[] components // size n
boolean[] visited //size n

function findComponent():
	for (i = 0; i < n; i++):
		if (!visited[i]):
			count ++;
			dfs(i)

	return (count, components);

function dfs(at):
	visited[at] = true
	components[at] = count
	for (next: g[at]):
		if(!visited[next]):
		dfs(next)
```

2. Given a 2D grid `grid` where `'1'` represents land and `'0'` represents water, count and return the number of islands. An **island** is formed by connecting adjacent lands horizontally or vertically and is surrounded by water. You may assume water is surrounding the grid (i.e., all the edges are water).

```java
class Solution {

	public int numIslands(char[][] grid) {
		int count = 0;
		
		for (int i = 0; i < grid.length; i++){
			for (int j = 0; j < grid[0].length; j++){
				if (grid[i][j] == '1'){
					count ++;
					dfs(grid, i, j);
				}
			}
		}
		
		return count;
	}

  

	public void dfs(char[][] grid, int i, int j) {
		if(
			i < 0 ||
			j < 0 ||
			i >= grid.length ||
			j >= grid[0].length ||
			grid[i][j] == '0'
		) {
			return;
		}
	
		grid[i][j] = '0';
		
		dfs(grid, i + 1, j);
		dfs(grid, i, j + 1);
		dfs(grid, i - 1, j);
		dfs(grid, i, j - 1);
	}
}
```

3. You are given a matrix `grid` where `grid[i]` is either a `0` (representing water) or `1` (representing land). An island is defined as a group of `1`'s connected horizontally or vertically. You may assume all four edges of the grid are surrounded by water. The **area** of an island is defined as the number of cells within the island. Return the maximum **area** of an island in `grid`. If no island exists, return `0`.

```java
class Solution {

	public int maxAreaOfIsland(int[][] grid) {

		int area = 0;

		for (int i = 0; i < grid.length; i++) {
			for (int j = 0; j < grid[0].length; j++) {
				if (grid[i][j] == 1) {
					area = Math.max(getArea(grid, i, j), area);
				}
			}
		}
		
		return area;
	}

  

	public int getArea(int[][] grid, int i, int j) {
		if (
		
			i < 0 ||
			i == grid.length ||
			j < 0 ||
			j == grid[0].length ||
			grid[i][j] == 0
		){
			return 0;
		}
		
		grid[i][j] = 0;
		
		return 1 + 
			getArea(grid, i + 1, j) + 
			getArea(grid, i, j + 1) + 
			getArea(grid, i - 1, j) + 
			getArea(grid, i, j - 1);
	}
}
```
## Related Data Structures
- [[Stack]] — DFS uses a stack (call stack in recursion, explicit stack iteratively)
- [[Graph]] — primary structure traversed by DFS

## Related Algorithms
- [[Topological Sort]] — DFS post-order produces a topological ordering
- [[Backtracking]] — backtracking is DFS with pruning
