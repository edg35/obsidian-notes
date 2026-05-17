
## Heap Data Structure

**A heap is a special tree-based data structure that satisfies the heap property:**

- **Max Heap:** For any given node, the value of the node is greater than or equal to the values of its children.
- **Min Heap:** For any given node, the value of the node is less than or equal to the values of its children.

**Key characteristics:**

- **Complete binary tree:** All levels are fully filled except possibly the last level, which is filled from left to right.
- **Efficiently represented as an array:** Due to its complete binary tree structure, heaps can be easily stored in an array.

**Common operations:**

- **Insert:** Adds an element to the heap while maintaining the heap property.
- **Delete (or extract):** Removes the root (maximum or minimum element) and restores the heap property.
- **Increase/decrease key:** Changes the value of an element and restores the heap property.
  
**Applications:**

- **Priority queues:** Heaps are often used to implement priority queues where elements have associated priorities.
- **Heap sort:** A sorting algorithm that builds a heap and repeatedly extracts the maximum or minimum element.
- **Graph algorithms:** Used in algorithms like Dijkstra's shortest path and Prim's minimum spanning tree.
### Examples

1. Given an integer array `nums` and an integer `k`, return _the_ `kth` _largest element in the array_ Note that it is the `kth` largest element in the sorted order, not the `kth` distinct element. Can you solve it without sorting?
   
```java
class Solution {
	public int findKthLargest(int[] nums, int k) {
		PriorityQueue<Integer> minHeap = new PriorityQueue<>();

		for (int i = 0; i < k; i++) {
			minHeap.add(nums[i]);
		}
		
		for (int i = k; i < nums.length; i++) {
			if (nums[i] > minHeap.peek()) {
				minHeap.remove();
				minHeap.add(nums[i]);
			}
		}

		return minHeap.peek();
	}
}
```

2. You are given an array of integers `stones` where `stones[i]` is the weight of the `ith` stone. We are playing a game with the stones. On each turn, we choose the **heaviest two stones** and smash them together. Suppose the heaviest two stones have weights `x` and `y` with `x <= y`. The result of this smash is:
	1. If `x == y`, both stones are destroyed, and
	2. If `x != y`, the stone of weight `x` is destroyed, and the stone of weight `y` has new weight `y - x`.
	At the end of the game, there is **at most one** stone left. Return _the weight of the last remaining stone_. If there are no stones left, return `0`.

```java
class Solution {
	public int lastStoneWeight(int[] stones) {
	PriorityQueue<Integer> heap = new PriorityQueue<>
	   (Comparator.reverseOrder());
	
	for (int stone: stones) heap.add(stone);
	
	while (heap.size() > 1) {
		int y = heap.poll();
		int x = heap.poll();
		
		if (x != y) heap.add(y-x);
	}
	
	if (heap.isEmpty()) return 0;
		return heap.poll();
	}
}
```

3. Given an integer array `nums` and an integer `k`, return _the_ `k` _most frequent elements_. You may return the answer in **any order**.
   
```java
class Solution {
	public int[] topKFrequent(int[] nums, int k) {
		// Find the frequency of each number
		Map<Integer, Integer> numFrequencyMap = new HashMap<>();
		for (int n : nums){
			numFrequencyMap.put(n, numFrequencyMap.getOrDefault(n, 0) + 1);
		} 
		
		PriorityQueue<Map.Entry<Integer, Integer>> topKElements = new PriorityQueue<>(
		(e1, e2) -> e1.getValue() - e2.getValue()
		);
		
		for (Map.Entry<Integer, Integer> entry :
			 numFrequencyMap.entrySet()) 
		{
			topKElements.add(entry);
			if (topKElements.size() > k) {
				topKElements.poll();
			}
		}
		
		// Create a list of top k numbers
		int[] topNumbers = new int[k];
		int i = 0;
		
		while (!topKElements.isEmpty()) {
			topNumbers[i] = topKElements.poll().getKey();
			i++;
		}
		
		return topNumbers;
	}
}  
```

4. Given an array of `points` where `points[i] = [xi, yi]` represents a point on the **X-Y** plane and an integer `k`, return the `k` closest points to the origin `(0, 0)`. The distance between two points on the **X-Y** plane is the Euclidean distance (i.e., `√(x1 - x2)2 + (y1 - y2)2`). You may return the answer in **any order**. The answer is **guaranteed** to be **unique** (except for the order that it is in).

```java
import java.util.PriorityQueue;

class Solution {
	public int[][] kClosest(int[][] points, int k) {
		// Use a max-heap to keep the closest k points
		PriorityQueue<int[]> heap = new PriorityQueue<>(
			(e1, e2) -> Double.compare(distance(e2), distance(e1)));
		
		for (int[] p : points) {
			heap.add(p);
			if (heap.size() > k) heap.poll();
		}
				
		int[][] result = new int[k][2];
		int i = 0;
		
		while (!heap.isEmpty()) {
			result[i++] = heap.poll();
		}
		
		return result;
	}
	
	// Calculate the squared Euclidean distance
	private double distance(int[] p) {
		return p[0] * p[0] + p[1] * p[1];
	}
}
```


## Used In
- [[Dijkstra's Algorithm]]
- [[Prim's Algorithm]]
- [[Greedy Algorithm]]
