## Queue Data Structure

**A queue is a linear data structure that follows the FIFO (First In, First Out) principle.** This means the first element added to the queue is the first one to be removed. It's like a line of people waiting for their turn; the person who arrives first is the first to be served.

### Key Operations

- **Enqueue:** Adds an element to the rear of the queue.
- **Dequeue:** Removes and returns the element at the front of the queue.
- **Peek:** Returns the element at the front of the queue without removing it. 
- **IsEmpty:** Checks if the queue is empty.
- **IsFull:** Checks if the queue is full (in case of fixed-size queues).
### Applications

Queues are used in various applications:

- **Task scheduling:** Manages processes waiting for CPU time.
- **Breadth-First Search (BFS):** Explores graph nodes level by level.
- **Buffering data:** Stores data temporarily before processing.
- **Print queues:** Holds print jobs in a waiting line.
- **Real-time systems:** Handles events and messages.

### Implementation

Queues can be implemented using either arrays or linked lists.

**Array implementation:**

- Can be implemented as a circular queue to avoid shifting elements.
- Efficient for enqueue and dequeue operations.
- Can lead to overflow or underflow errors.

**Linked list implementation:**

- Dynamic size.
- More flexible.
- Potentially slower enqueue and dequeue operations due to memory allocation.
  
### Examples


## Used In
- [[Breadth-First Search]]
- [[Ford-Fulkerson]]
