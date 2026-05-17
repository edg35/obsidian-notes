## Stack Data Structure

**A stack is a linear data structure that follows the LIFO (Last In, First Out) principle.** Imagine a stack of plates; the last plate you put on is the first one you take off.

### Key Operations

- **Push:** Adds an element to the top of the stack.
- **Pop:** Removes and returns the element at the top of the stack.
- **Peek:** Returns the top element without removing it. 
- **IsEmpty:** Checks if the stack is empty.
- **IsFull:** Checks if the stack is full (in case of fixed-size stacks).
### Applications

Stacks are used in various applications:

- **Browser history:** Stores visited pages in a stack.
- **Function calls:** Keeps track of function calls using a call stack.
- **Undo/redo functionality:** Implements undo/redo operations using stacks.
- **Expression evaluation:** Converts infix expressions to postfix or prefix and evaluates them.
- **Backtracking algorithms:** Used for solving problems like maze solving, N-Queens, etc.

### Implementation

Stacks can be implemented using either arrays or linked lists.

**Array implementation:**

- Fixed size.
- Efficient for push and pop operations.
- Can lead to overflow or underflow errors.

**Linked list implementation:**

- Dynamic size.
- More flexible.
- Potentially slower push and pop operations due to memory allocation.
### Examples

1. Given a string `s` containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid. An input string is valid if:
	1. Open brackets must be closed by the same type of brackets.
	2. Open brackets must be closed in the correct order.
	3. Every close bracket has a corresponding open bracket of the same type.
   
```java
class Solution {
	public boolean isValid(String s) {
		Stack<Character> stack = new Stack<Character>();
		
		// Loop through every character in the string
		for (char c : s.toCharArray()) {
			if (c == '(' || c == '[' || c == '{') {
				stack.push(c);
			} else { 
				if (stack.isEmpty()) return false;
				char top = stack.peek();
			
				if ((c == ')' && top == '(') || 
					(c == ']' && top == '[') || 
					(c == '}' && top == '{')) 
				{
					stack.pop();
				} else {
					return false;
				}
			}
		}
	
		return stack.isEmpty();
	}
}
```

2. You are given an array of strings `tokens` that represents an arithmetic expression in a [Reverse Polish Notation](http://en.wikipedia.org/wiki/Reverse_Polish_notation). Evaluate the expression. Return _an integer that represents the value of the expression_. **Note** that:
	1. The valid operators are `'+'`, `'-'`, `'*'`, and `'/'`.
	2. Each operand may be an integer or another expression.
	3. The division between two integers always **truncates toward zero**.
	4. There will not be any division by zero.
	5. The input represents a valid arithmetic expression in a reverse polish notation.
	6. The answer and all the intermediate calculations can be represented in a **32-bit** integer.

```java
class Solution {
	public int evalRPN(String[] tokens) {
		Stack<Integer> stack = new Stack<>();
		
		for (String token: tokens) {
			if (isOperator(token)) {
				int n1 = stack.pop();
				int n2 = stack.pop();
				stack.push(performOperation(token, n2, n1));
			} else {
				stack.push(Integer.parseInt(token));
			}
		}
		
		return stack.pop();
	}
	
	public boolean isOperator(String token) {
		return token.equals("+") || 
			   token.equals("-") || 
			   token.equals("*") ||
			   token.equals("/");
	}
	
	public int performOperation(String token, int n1, int n2) {
		int result = 0;
	
		if (token.equals("+")) {
			result = n1 + n2;
		} else if (token.equals("-")) {
			result = n1 - n2;
		} else if (token.equals("*")) {
			result = n1 * n2;
		} else {
			result = n1 / n2;
		}
	
		return result;
	}
}
```

