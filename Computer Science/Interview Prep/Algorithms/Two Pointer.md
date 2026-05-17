
### Examples

1. Given an integer array `nums` sorted in **non-decreasing** order, return _an array of **the squares of each number** sorted in non-decreasing order_.

```java
class Solution {
	public int[] sortedSquares(int[] nums) {
		int[] res = new int[nums.length];
		int left = 0;
		int right = nums.length - 1;
		
		for (int i = nums.length - 1; i >= 0; i--) {
			if (Math.abs(nums[left]) > Math.abs(nums[right])) {
				res[i] = nums[left] * nums[left];
				left++;
			} else {
				res[i] = nums[right] * nums[right];
				right--;
			}
		}

		return res;
	}
}
```

2. Write a function that reverses a string. The input string is given as an array of characters `s`.
   
```java
class Solution {
	public void reverseString(char[] s) {
		char temp;	
		int left = 0;
		int right = s.length - 1;

		while (left < right) {
			temp = s[left];
			s[left] = s[right];
			s[right] = temp;
			left++;
			right--;
		}
	}
}
```

3. A phrase is a **palindrome** if, after converting all uppercase letters into lowercase letters and removing all non-alphanumeric characters, it reads the same forward and backward. Alphanumeric characters include letters and numbers. Given a string `s`, return `true` _if it is a **palindrome**, or_ `false` _otherwise_.
   
```java
class Solution {
	public boolean isPalindrome(String s) {
		String newString = s.replaceAll("[^a-zA-Z0-9]+", "").toLowerCase();
		int fp = 0;
		int bp = newString.length()-1;
		
		while(fp < bp){
			if(newString.charAt(fp) != newString.charAt(bp)) return false;
			fp++;
			bp--;
		}
	
		return true;
	}
}
```

