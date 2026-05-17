
## Contains Duplicate (Easy)

### Solution 1: Hash Set Length

a set only stores unique values, so duplicates are automatically removed.  
Instead of checking each element manually, we simply compare the length of the set to the length of the original array.  
If duplicates exist, the set will contain fewer elements.  
The logic is identical to the earlier approach — this version is just a shorter and more concise implementation of it.

```python

class Solution:
	def hasDuplicate(self, nums: List[int]) -> bool:
		return len(set(nums)) < len(nums)
		
# Time:  O(n)
# Space: O(n)
```


### Solution 2: Hash Set

We can use a hash set to efficiently keep track of the values we have already encountered.  
As we iterate through the array, we check whether the current value is already present in the set.  
If it is, that means we've seen this value before, so a duplicate exists.  
Using a hash set allows constant-time lookups, making this approach much more efficient than comparing every pair.

``` python
class Solution:
    def hasDuplicate(self, nums: List[int]) -> bool:
        seen = set()
        for num in nums:
            if num in seen:
                return True
            seen.add(num)
        return False
        
        
# Time:  O(n)
# Space: O(n)
```
---

## Valid Anagram (Easy)

Given two strings `s` and `t`, return `true` if the two strings are anagrams of each other, otherwise return `false`.

An **anagram** is a string that contains the exact same characters as another string, but the order of the characters can be different.

### Solution: [[Hash Map]]

```python
class Solution:
    def isAnagram(self, s: str, t: str) -> bool:
        if len(s) != len(t):
            return False

        countS, countT = {}, {}

        for i in range(len(s)):
            countS[s[i]] = 1 + countS.get(s[i], 0)
            countT[t[i]] = 1 + countT.get(t[i], 0)
        return countS == countT
        
# Time:  O(n+m)
# Space: O(1)
```
---

## Two Sum (Easy)
Given an array of integers `nums` and an integer `target`, return the indices `i` and `j` such that `nums[i] + nums[j] == target` and `i != j`.

You may assume that _every_ input has exactly one pair of indices `i` and `j` that satisfy the condition.

Return the answer with the smaller index first.

### Solution: [[Hash Map]]

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        prevMap = {}  # val -> index

        for i, n in enumerate(nums):
            diff = target - n
            if diff in prevMap:
                return [prevMap[diff], i]
            prevMap[n] = i
            
# Time:  O(n)
# Space: O(n)
```
---
## Group Anagram (Medium)

Given an array of strings `strs`, group all _anagrams_ together into sublists. You may return the output in **any order**.

An **anagram** is a string that contains the exact same characters as another string, but the order of the characters can be different.

### Solution 1: Sorting

```python
class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        res = defaultdict(list)
        for s in strs:
            sortedS = ''.join(sorted(s))
            res[sortedS].append(s)
        return list(res.values())

# Time:  O(m * nlogn)
# Space: O(m * n)
```

### Solution 2: Hash Table

```python
class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        res = defaultdict(list)
        for s in strs:
            count = [0] * 26 # a ... z
            for c in s: # loop through each character in string
	            # if b = 81, then b - a or 81 - 80 = 1; ord => ascii
                count[ord(c) - ord('a')] += 1
            res[tuple(count)].append(s)
        return list(res.values())

# Time:  O(m * n)
# Space: O(m)
```

## Top k Frequent Elements (Medium)

Given an integer array `nums` and an integer `k`, return the `k` most frequent elements within the array.

The test cases are generated such that the answer is always **unique**.

You may return the output in **any order**.

### My Solution: [[Hash Map]] + [[Heap]]

```python
class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        freq = {}

        # freqency table
        for num in nums:
            freq[num] = 1 + freq.get(num, 0)
        
        freq_heap = [(-value, key) for key, value in freq.items()]
        heapq.heapify(freq_heap)

        res = []

        for i in range(k):
            res.append((heapq.heappop(freq_heap)[1]))

        return res

# Time:  O(nlog(k))
# Space: O(n+k)
```

### Best Solution: Bucket Sort

```python
class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        count = {}
        freq = [[] for i in range(len(nums) + 1)]

        for num in nums:
            count[num] = 1 + count.get(num, 0)
        for num, cnt in count.items():
            freq[cnt].append(num)

        res = []
        for i in range(len(freq) - 1, 0, -1): # from last index to 0, dec by -1
            for num in freq[i]:
                res.append(num)
                if len(res) == k:
                    return res

# Time:  O(n)
# Space: O(n)
```


## Product of Array Except Self (Medium)

Given an integer array `nums`, return an array `output` where `output[i]` is the product of all the elements of `nums` except `nums[i]`.

Each product is **guaranteed** to fit in a **32-bit** integer.

Follow-up: Could you solve it in O(n)O(n) time without using the division operation?

