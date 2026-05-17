## Tree Data Structure: A Quick Overview

**A tree is a hierarchical data structure where each element (node) can have zero or more children.** It's often visualized as an upside-down tree, with the root at the top and branches extending downwards.

### Key Components:

- **Node:** Contains data and references to its children.
- **Root:** The topmost node in the tree.
- **Parent:** The node directly above another node.
- **Child:** A node directly below another node.
- **Leaf:** A node with no children.

### Types of Trees:

- **Binary Tree:** Each node has at most two children.
- **Binary Search Tree (BST):** A binary tree where the left child's value is less than the parent's, and the right child's value is greater.
- **AVL Tree:** A self-balancing BST.
- **Red-Black Tree:** Another self-balancing BST

  
> [!NOTE] Binary Tree
> trees where each node has at most two children
> exactly one root
> exactly one path from root to leaf


```java
class Node {
	private int val;
	private Node left;
	private Node right;
	
	public Node (int val, Node left, Node right) {
		this.val = val;
		this.left = left;
		this.right = right;
	}
}
```

**Examples:**

1. [[Depth-First Search]] Values: Take in a tree root and traverse 
	1. Use a [[Stack]] to keep track of nodes to visit
	2. Time: O(N)
	3. Space: O(N)

```java
class Solution {
	public void DFSIterative(Node root) {
		if (root == null) return;
		
		Stack<Node> s = new Stack<Node>(root);

		while (s.length > 0) {
			Node curr = s.pop();
			
			if (curr.right) s.push(curr.right);
			if (curr.left) s.push(curr.left);
		}
	}

	public void DFSrecursive(Node root) {
		if (root == null) return;
		
		DFSrecursive(root.left);
		DFSrecursive(root.right);
	}
}
```

2. [[Breadth-First Search]] Values: Take in a tree root and traverse
	1. Use a [[Queue]] to keep track of nodes to visit
	2. Time: O(N)
	3. Space: O(N)

```java
class Solution {
	public void BFSIteravtive(Node root) {
		if (root == null) return;
	
		Queue<Node> q = new Queue<>(root);
		
		while (q.length > 0) {
			Node curr = q.dequeue();
			
			if (curr.left) q.enqueue(curr.left);
			if (curr.right) q.enqueue(curr.right);
		}
	}
}
```

3. Target in Tree: find if target is in tree
	1. traverse tree nodes, using compare operators to determine which child to look at next. This is a variation of [[Binary Search]]
	2. Time: O(log N)
	3. Space: O(N)

```java
class Solution {
	public boolean findTarget (Node root; int target) {
		if (root == null) return false;
		
		Node curr = root;
		while (curr != null) {
			if (curr.val == target) return true;
			if (curr.val > target) {
				curr = curr.left;
			} else {
				curr = curr.right;
			}
		}
		
		return false;
	}

	public boolean findTargetR (Node root; int target) {
		if (root == null) return false;
		if (root.val == target) return true;
		
		if (root.val > target) return findTargetR(root.left, target);
		return findTargetR(root.right, target);
	}
}
```

4. Invert Binary Trees: Given the `root` of a binary tree, invert the tree, and return _its root_.

```java
class Solution {
	public TreeNode invertTree(TreeNode root) {
		if (root == null) return null;
		TreeNode node = new TreeNode();
		node.right = invertTree(root.left);
		node.val = root.val;
		node.left = invertTree(root.right);
		return node;
	}
}
```

5. Tree Sum

```java
class Solution {
	// Recursive
	public int treeSum(TreeNode root) {
		if (root === null) return 0;
		return root.val + treeSum(root.left) + treeSum(root.right);
	}

	//Iterative Breath or Depth
	public int treeSum(TreeNode root) {
		if (root == null) return 0;
		int sum = 0;
		Queue<TreeNode> queue = new Queue<>();
		queue.add(root);

		while (queue.length() > 0) {
			TreeNode curr = queue.remove();
			sum+=curr.val;
			if (curr.left != null) queue.add(curr.left);
			if (curr.right != null) queue.add(curr.right);
		}
		
		return sum;
 	} 
}
```

6. Min Tree Value

```java
class Solution {
    public int treeMinValue(TreeNode root) {
        if (root == null) return Integer.MAX_VALUE;

        int leftMin = treeMinValue(root.left);
        int rightMin = treeMinValue(root.right);
        
        return Math.min(root.val, Math.min(leftMin, rightMin));
    }
}
```

7. Max root to leaf path sum

```java
class Solution {
	public int rootToLeaf(TreeNode root) {
		if (root == null) return Integer.MIN_VALUE;
		if (root.left == null && root.right == null) return root.val;
		
		int leftPath = rootToLeaf(root.left);
		int rightPath = rootToLeaf(root.right);

	
		return root.val + Math.max(leftPath, rightPath);
	}
}
```

8. Given the `root` of a binary tree and an integer `targetSum`, return `true` if the tree has a **root-to-leaf** path such that adding up all the values along the path equals `targetSum`.
   
```java
class Solution {
	public boolean hasPathSum(TreeNode root, int targetSum) {
		if (root == null) return false;
		
		if (root.left == null && 
			root.right == null && 
			root.val == targetSum) return true;
		
		if (hasPathSum(root.left, targetSum - root.val) || 
			hasPathSum(root.right, targetSum - root.val))
		{
			return true;
		} else {
			return false;
		}
	}
}
```




## Used In
- [[Prim's Algorithm]]
- [[Greedy Algorithm]]
