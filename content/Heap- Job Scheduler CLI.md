# 0.0 Intro to Heap Data Structure

Heap is specialized tree based Data Structure, that satisfy the heapify property every time an element is added or removed and are fundamentally complete binary trees.

- In `Max Heap`, every parent node is greater than or equal to its parent node.
- Similarly, in `Min Heap`, every parent node is less than or equal to its parent node.
- Sibling nodes don't need to ordered relative to each other.

Heaps are often implemented in the form of dynamic arrays  and mathematically we can say for an array of size `n` which are the number of nodes in our heap:

- **Height of the Heap**: For n nodes, the height of the heap tree is: 

					Height = $\lfloor log_2 (n) \rfloor$

- A node at index `i` has children at `2i + 1` and `2i + 2` (0- based indexing) and parent at $\lfloor \frac{(i-1)}{2} \rfloor$ .
- **Completeness**: The structure guarantees optimal space utilization with no "holes" in the tree.


```
Array: [50, 30, 40, 10, 20, 35]
Max-Heap Tree:       50
          /    \
        30      40
       /  \    /
     10   20  35
```

- Typical Heap Operation include the following operations:
	1. `insert()` to insert an element in the heap 
	2. `pop()` the `min/max` element
	3. `heapify()` at every stage the tree structure changes on adding/ removing an element
	4.  `top()` to get the `min/max element` 

