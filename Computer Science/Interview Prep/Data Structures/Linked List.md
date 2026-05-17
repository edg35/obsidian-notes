## Linked List: A Quick Overview

**A linked list is a linear data structure where elements are not stored in contiguous memory locations.** Instead, each element, called a _node_, contains data and a reference (or pointer) to the next node in the sequence.

### Key Components of a Linked List:

- **[[Node]]:** The basic unit of a linked list, containing data and a pointer to the next node.
- **Head:** A pointer to the first node in the list.
- **Tail:** A pointer to the last node in the list (optional, depending on implementation).

### Types of Linked Lists:

- **Singly Linked List:** Each node has a data field and a pointer to the next node.
- **Doubly Linked List:** Each node has a data field, a pointer to the next node, and a pointer to the previous node.
- **Circular Linked List:** The last node's pointer points back to the first node, creating a circle.

### Advantages of Linked Lists:

- Dynamic size: Can grow and shrink as needed.
- Efficient insertion and deletion: Can be performed in constant time, O(1), given the position of the node.
- No memory wastage: Space is allocated only for the necessary nodes.

### Disadvantages of Linked Lists:

- Random access is inefficient: Accessing a specific element requires traversing the list from the beginning, leading to O(n) time complexity.
- Extra memory for pointers: Each node requires additional space for the pointer(s).
- Potential for more complex implementation: Compared to arrays, linked lists can have more intricate code for operations like traversal and searching.

### Common Operations:

- Insertion: Adding a node at a specific position.
- Deletion: Removing a node from a specific position.
- Search: Finding a node with a particular value.
- Traversal: Visiting each node in the list sequentially.

### Examples

1. Given the `head` of a sorted linked list, _delete all duplicates such that each element appears only once_. Return _the linked list **sorted** as well_.
   
```java
class Solution {
	public ListNode deleteDuplicates(ListNode head) {
		if (head == null) return head;

		ListNode ptr = head.next;	
		ListNode prev = head;

		while (ptr != null) {
			if (ptr.val == prev.val) {
				prev.next = ptr.next;
				ptr.next = null;
				ptr = prev.next;
			} else {
				ptr = ptr.next;
				prev = prev.next;
			}
		}
		
		return head;
	}
}
```

2. Given the `head` of a singly linked list, reverse the list, and return _the reversed list_.
   
```java
class Solution {
	public ListNode reverseList(ListNode head) {
		ListNode curr = head;
		ListNode next = null;
		ListNode prev = null;
		
		while(curr != null){
			next = curr.next;
			curr.next = prev;
			prev = curr;
			curr = next;
		}
		
		return prev;
	}
}
```


![Screenshot 2024-08-09 at 10.19.16 AM.png](https://assets.leetcode.com/users/images/fbde4436-ff05-4e68-9ce8-2c3831a72d25_1723213522.6917145.png)
![Screenshot 2024-08-09 at 10.19.19 AM.png](https://assets.leetcode.com/users/images/0983269c-9744-42bb-b2c8-ceddb9c53f8d_1723213525.2782161.png)
![Screenshot 2024-08-09 at 10.19.21 AM.png](https://assets.leetcode.com/users/images/9ba18d0b-da91-4d06-bcb4-e0dc40bfd34e_1723213535.1843827.png)
![Screenshot 2024-08-09 at 10.19.24 AM.png](https://assets.leetcode.com/users/images/13f8a2fa-7249-45b0-8eb5-91d865ae8115_1723213537.8953936.png)
![Screenshot 2024-08-09 at 10.19.26 AM.png](https://assets.leetcode.com/users/images/c57f67f2-9441-4b99-9ad5-2b921c1e2652_1723213540.113058.png)
![Screenshot 2024-08-09 at 10.19.29 AM.png](https://assets.leetcode.com/users/images/38b7676c-a5dc-4825-a7e2-81316bd56938_1723213551.1550446.png)
![Screenshot 2024-08-09 at 10.19.31 AM.png](https://assets.leetcode.com/users/images/762b129e-0fab-484e-9cf9-ac8453912b1a_1723213553.9502525.png)
![Screenshot 2024-08-09 at 10.19.33 AM.png](https://assets.leetcode.com/users/images/21935ff7-9897-42fd-aa89-2abca28864f1_1723213556.1770194.png)
![Screenshot 2024-08-09 at 10.19.39 AM.png](https://assets.leetcode.com/users/images/5330362a-262f-4ec4-948b-b30ccdc8a070_1723213560.7062523.png)
![Screenshot 2024-08-09 at 10.19.42 AM.png](https://assets.leetcode.com/users/images/c3fb74b9-4164-49ff-b355-a90709f256a9_1723213562.565261.png)

3. You are given the heads of two sorted linked lists `list1` and `list2`. Merge the two lists into one **sorted** list. The list should be made by splicing together the nodes of the first two lists. Return _the head of the merged linked list_.
   
```java
class Solution {
	public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
		if(list1 == null) return list2;
		if(list2 == null) return list1;

		if(list1.val < list2.val){
			list1.next = mergeTwoLists(list1.next, list2);
			return list1;
		} else {
			list2.next = mergeTwoLists(list1, list2.next);
			return list2;
		}
	}
}
```

