---
Time Complexity: O(logN)
Type: Both
Chapter: 8
---
**Template**:

```java
// Iterative
public static int binarySearch(List<Integer> arr, int target) {
    int left = 0;
    int right = arr.size() - 1;
    int firstTrueIndex = -1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (feasible(mid)) {
            firstTrueIndex = mid;
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }
    return firstTrueIndex;
}
```
```java
// Recursive
public static int binarySearch(int[] a, int x, int low, int high) {
	if (low > high) return -1;
	int mid = low + (high - low) / 2;

	if (a[mid] < x) {
		binarySearch(a, x, mid + 1, high);
	} else if (a[mid > x]) {
		binarySearch(a, x, low, mid - 1);
	} else {
		return mid;
	}
}
```


> [!NOTE] Binary Search
> Binary search is an efficient array search algorithm. It works by narrowing down the search range by half each time. If you have looked up a word in a physical dictionary, you've already used binary search in real life.
> 
> This idea can be implemented both iteratively and recursively. However, the major difference is that the iterative version of binary search uses `O(1)` memory while the recursive version uses `O(log(N))` memory.

**Calculating Mid**:

- Note that when calculating `mid`, if the number of elements is even, there are two elements in the middle. We usually follow the convention of picking the first one, equivalent to doing integer division `(left + right) / 2`.

- In most programming languages, we calculate `mid` with `left + floor((right-left) / 2)` to avoid potential integer overflow. However, in Python, we do not need to worry about integer overflow with `left + right` because Python3 integers can be arbitrarily large.


**Example:**

1. Given a sorted array of integers and an integer called target, find the element that equals the target and return its index. If the element is not found, return -1.

```java
public static int binarySearch(List<Integer> arr, int target) {
        // WRITE YOUR BRILLIANT CODE HERE
        int left, right = 0, arr.size() - 1;
        int index = -1;
        
        while (left <= right) {
            int mid = left + (right - left)/2;
            
            if (arr.get(mid) == target) return mid;
            if (arr.get(mid) < target){
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        
        return -1;
    }
```

2. You are given two sorted arrays, A and B, where A has a large enough buffer at the end of it to hold B, write a method to merge B into A.

```java
public static int[] merge(int[] a, int[] b, int countA, int countB) {
	int idxMerged = countB + countA - 1; // last location of merged array
	int idxA = countA -1; // last element in A
	int idxB = countB -1; // last element in B

	// Merge a and b starting form the last element in each
	while (idxB >= 0) {
		if (idxA >=0 && a[idxA] > b[idxB]) {
			a[idxMerged] = a[idxA]; // copy element
			idxA--;
		} else {
			a[idxMerged] = b[idxB]; // copy element
			idxB--;
		}
		idxMerged--; // move idx
	}
}
```

3. Given a sorted array of distinct integers and a target value, return the index if the target value is found, if not, return the index where it would be found if it were inserted in order. O(log n)

```java
class Solution {
	public int searchInsert(int [], int target) {
		int low, high = 0, nums.length - 1;
		
		// binary search the array
		while (low <= high) {
			int mid = low + (high - low) / 2;

			if (nums[mid] == target) return mid;
			
			if (nums[mid] > target) {
				high = mid + 1;
			} else {
				low = mid - 1;
			}
		}
		
		// return where the target would be inserted
		return (high + (low - high) / 2) + 1;
	}
}
```

4. Given a positive integer num, return `true` _if_ `num` _is a perfect square or_ `false` _otherwise_. A **perfect square** is an integer that is the square of an integer. In other words, it is the product of some integer with itself. You must not use any built-in library function, such as `sqrt`.

```java
class Solution {
	public boolean isPerfectSquare(int num) {
		int low = 1;
		int high = num;
		int mid = 0;

		while (low <= high) {
			mid = low + (high - low) / 2;
			long pow = (long)mid * mid;
			if (pow == num) return true;
		
			if (pow > num) {
				high = mid - 1;
			} else {
				low = mid + 1;
			}
		}
		return false;
	}
}
```

5. You are a product manager and currently leading a team to develop a new product. Unfortunately, the latest version of your product fails the quality check. Since each version is developed based on the previous version, all the versions after a bad version are also bad. Suppose you have `n` versions `[1, 2, ..., n]` and you want to find out the first bad one, which causes all the following ones to be bad. You are given an API `bool isBadVersion(version)` which returns whether `version` is bad. Implement a function to find the first bad version. You should minimize the number of calls to the API.

```java
/* The isBadVersion API is defined in the parent class VersionControl.
boolean isBadVersion(int version); */

public class Solution extends VersionControl {
	public int firstBadVersion(int n) {
		int low = 1;
		int high = n;
		int badVersion = 1;
		
		while (low <= high) {
			int mid = low + (high - low) / 2;
			if(isBadVersion(mid)) {
				badVersion = mid;
				high = mid - 1;
			} else low=mid+1;
		}

		return badVersion;
	}
}
```

6. You are given an `m x n` integer matrix `matrix` with the following two properties:
	1. Each row is sorted in non-decreasing order.
	2. The first integer of each row is greater than the last integer of the previous row. 
	Given an integer `target`, return `true` _if_ `target` _is in_ `matrix` _or_ `false` _otherwise_. You must write a solution in `O(log(m * n))` time complexity.

```java
class Solution {
	public boolean searchMatrix(int[][] matrix, int target) {
		// first we can find the row it is located in and then find where it is
		int lowRow = 0;
		int highRow = matrix.length - 1;
		int lowCol = 0;
		int highCol = matrix[0].length - 1;
		int midRow = 0;
		int midCol = 0;
		
		// find the row
		while (lowRow <= highRow) {
			midRow = lowRow + (highRow - lowRow) / 2;
	
			if (matrix[midRow][lowCol] > target) {
				highRow = midRow - 1;
			} else if (matrix[midRow][highCol] < target) {
				lowRow = midRow + 1;
			} else {
				break;
			}
		}
		
		// find the col
		while (lowCol <= highCol) {
			midCol = lowCol + (highCol - lowCol) / 2;
			
			if (matrix[midRow][midCol] == target) return true;
		
			if (matrix[midRow][midCol] < target) {
				lowCol = midCol + 1;
			} else {
				highCol = midCol - 1;
			}
		}
		
		return false;
	}

}
```