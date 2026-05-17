---
Time Complexity: O(N)
Type: Iterative
Chapter:
---
> [!NOTE] Sliding Window
> The sliding window algorithm is a technique used to solve problems involving arrays or lists, where you need to keep track of a subset of elements that is continuously changing. This technique is particularly useful for problems that involve finding subarrays or substrings that meet certain criteria, such as maximum sum, minimum length, or specific conditions.

**Examplexqxs

1. Here is the sliding window algorithm example in Java for finding the maximum sum of a subarray of size `K`
   
```java
   public class SlidingWindow {

    public static int maxSumSubarray(int[] arr, int k) {
        int n = arr.length;
        if (n < k) {
            // Throw error arr to small for window size
        }

        // Compute the sum of the first window
        int maxSum = 0;
        for (int i = 0; i < k; i++) {
            maxSum += arr[i];
        }

        int currentSum = maxSum;

        // Slide the window over the array
        for (int i = k; i < n; i++) {
            currentSum += arr[i] - arr[i - k];
            maxSum = Math.max(maxSum, currentSum);
        }

        return maxSum;
    }

    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 4, 5, 6, 7, 8, 9};
        int k = 3;
        int result = maxSumSubarray(arr, k);
    }
}
```


2. You are given an integer array `prices` where `prices[i]` is the price of NeetCoin on the `ith` day. You may choose a **single day** to buy one NeetCoin and choose a **different day in the future** to sell it. Return the maximum profit you can achieve. You may choose to **not make any transactions**, in which case the profit would be `0`.

```java
class Solution {

	public int maxProfit(int[] prices) {
	
		int profit = 0;
		int buy = 0;
		int sell = 1;
	
		while (sell < prices.length) {
			if (prices[sell] > prices[buy]) {
				profit = Math.max(profit, prices[sell] - prices[buy]);
			} else {
				buy = sell;
			}
			
			sell++;
		}
	
		return profit;
	}
}
```

3. You are given an integer array `nums` consisting of `n` elements, and an integer `k`. Find a contiguous subarray whose **length is equal to** `k` that has the maximum average value and return _this value_. Any answer with a calculation error less than `10-5` will be accepted.
   
```java
class Solution {
	public double findMaxAverage(int[] nums, int k) {
		double sum = 0;
		for (int i = 0; i < k; i++) {
			sum += nums[i];
		}
		double ans = -Double.MAX_VALUE;
		
		for (int l = 0, r = l + k - 1; l <= nums.length - k; l++) {
			double avg = sum / k;
			ans = Math.max(ans, avg);
			sum -= nums[l];
			r++;
			if (r < nums.length) sum += nums[r];
		}
		
		return ans;
	}
}
```


4. Given a string `s`, find the length of the **longest** **substring** without repeating characters.
   
```java
class Solution {
	public int lengthOfLongestSubstring(String s) {
		int n = s.length();
		int maxLength = 0;
		Set<Character> charSet = new HashSet<>();
		int left = 0;
		
		for (int right = 0; right < n; right++) {
			if (!charSet.contains(s.charAt(right))) {
				charSet.add(s.charAt(right));
				maxLength = Math.max(maxLength, right - left + 1);
			} else {
				while (charSet.contains(s.charAt(right))) {
					charSet.remove(s.charAt(left));
					left++;
				}
		
				charSet.add(s.charAt(right));
			}
		}
		
		return maxLength;
	}
}
```

