---
title: Data Structures Notes
author: Haroon Ghori
geometry: margin=1in
---

#### Wrapper Classes
- java primative types
	- int
	- double
	- char
	- float
	- long
	- byte
	- boolean
- Wrapper Classes
	- int => Integer
	- double => Double
	- char => Character
	- ...
- Wrapper classes are instantiable classes that correspond to primitive data types.
- Autobox - primitive types can be automatically be converted to wrapper classes when needed.

#### Generics
- Creates a "generic" methods and classes that work with unspecified data types.
	- Generic method header
		```Java
		public static <T> <returnType> methodName(T thing)
		```
	- Generic class header
		```Java
		public class myDoubleGen<T1, T2>
		```
	- Implementing Generics
		- You cannot implement an array of generics directly
			- `T[] ra = new T[n]` (not allowed)
			- `T[] ra = (T[]) new Object[n]` (allowed)
		- Any type `<T>` can fit into a generic type so long as it's an instantiate class (eg Wrapper Class)
			- we must use `<T> = Integer` instead of `<T> = int`

#### Java Collections Framework
- Collection - an object that groups multiple elements in a single unit
	- typically data items that form a "natural" group
- Collections Framework - a unified architecture for representing and manipulating collections.
    - `List<T>` extends `Collection<T>`
		- for manipulating a sequence of values
		- adds positional operations for indexing into a Collection
		- al elements occupy a numbered position
		- positions of N elements go from 0 to N-1
		methods
			- `get(where)`
			- `indexOf(item)`
			- `set(where, item)`
			- `add(where, item)`
			- `remove(where)`
	- `Set<T>` extends `Collection<T>`
		- Unordered, unique "set" of values
		- `SortedSet<T> extends Set<T>` introduces ordering
	- `Queue<T>` extends `Collection<T>`
		- usually "first in first out" (FIFO) but could be priority queue
			- adds to end, removes and "peeks" at beginning
		methods
			- `push()`
			- `peek()`
			- `pop()`
		- A deque is a double ended queue
	- `Collection<T>`
		- `int size()`
		- `void clear()`
		- `boolean isEmpty()`
		- `boolean contains(T o)`
		- `boolean add(T)`
	- `Map<K,V>` (maps are not collections)
		- stores key/value pairs
			- each pair is an entry (nested class)
			- K is the key, V is the value
				- key must be unique
				- key could be paired with a collection which includes multiple values
- Implementations
	- `List<T>`
		- `ArrayList<T>`
		- `LinkedList<T>`
	- `Set<T>`
		- `SortedSet<T>`
			- `TreeSet<T>`
	- `Map<K, V>`
		- `HashMap<K,V>`
		- `TreeMap<K,V>`
		- `LinkedMap<K,V>`

#### Abstract Data Types
- Data type - a collection of values and the operations that can be applied to them
	- Interface
		- Gives you a list of operations (method headers) that can be performed on a data type, but doesn't define what they do / how they work. Tells the programmer how the user can interact with the object, but leaves the definitions to the programmer.
	- Implementation
		- Defines the methods (interactions) in the interface.
- Equivalence Relation
	- A relation ~ over a set S is a set of pairs of items from S
		- Note '~' represents a relation
		- write x~y to indicate that {x,y} exists in ~
	- A relation ~ over a set S is an equivalence relation if it is
		- reflexive
			- for al x E S, x~x
		- symmetric
			for all x,y E S, x~y implies y~x
		- transitive
			- for all x,y,z E S, x~y and y~z implies x~z
	- An equivalence relation ~ over the set S partitions S into disjoint (non overlapping) subsets
		- two elements in the same subsets are considered equivalent with respect to ~
	- Dynamic Equivalence Problem
		- Sometimes we need to modify the equivalence relation as we go, changing the partition
			- the formerly disjointed sets get combined
		- Given n equivalence relation over S and two items from S, how can we determine if the elements are equivalent.

#### Bag
- Basically a Set (from the java collections framework) but duplicates are allowed.
	- Order doesn't matter
	- Size doesn't matter
	- Operations
		- create
		- add
		- clear
		- size
		- isEmpty
		- remove
		- list
		- contains
- Unlike sets Bags do not have union, intersection, and setDifference methods

#### Partition
- a collection of disjoint subsets
- begins with every element in its own subset
	- the subsets are then repeatedly joined via a union operator
- Simplify names
	- let N be the number of items in a structure
	- then we assume that the names of the items in our structure are {0, N-1}
- A Structure with a Quick Union Method
	- to start each item is in it's own set named with its own name
	- use an array to store names of sets
		- position i holds the name of the set that contains item i
	- performing find(x)
		- return array[x]
		- takes O(1) time
			- constant time
			- very fast
	- performing union(a, b)
		- find(a) to learn i, the name of 'a's set
		- find(b) to learn j, the name of 'b's set
		- Go through the array and replace all 'j's with 'i's
		- O(N) time
- A Structure with a Quick Union Method
	- Use a tree to represent each set with the root being the name of the set
		- one "top" element is called the root with related elements branching "below"
		- elements with no branches are called leaves
	- each node keeps track of it's parent
	- tree is represented by an array where array[i] gives the parent of item i
		-array[0] = -1 since root has no parent
	- performing find(x)
		- check arrray[x]
			- if array[x] == -1
				- return x
			- else
				- check array[array[x]]
			- repeat until we find the root (name) of the set
		- performing union(a, b)
			- find(a) to find the root of a
			- find(b) to find the root of b
			- set a[b] to b
				- the b is the parent of a
		- use union-by-size
			- set the root value for b to be -size(b).
				- then search for any negative number instead of -1 when performing find(x)
			- then always attatch the smaller tree to the larger toree to keep trees small.
			- then no tree can have depth greater than log(N)
		- use path compression
			- when performing find(x) set all nodes between x and the root to point to the root. Compresses all paths such that find(x) becomes much faster

#### Iterators
- An object that allows us to access each element in a collection in turn.
Interfaces
- Iterator
- Iterable
	- Defines a collection that is iterable
- Iteratiors do not neccesarily operate in order. They just garuntee that every element in the iteration will be selected.
- An iterator's remove() method is optional. Calling it might result in an UnspportedOperationException
- Modifying a collection while an iterator is in use can cause problems.
	- When detected will throw a ConccurrentModificationException. Happens often when remove() is called.

#### Linked List
- Collection with discontinuous memory allocation
- capacity is not fixed
- each element "points" to the next element in the list (Singly linked list)
	- thus each element (called a node) must have a data part and a next part
	- the final node's next part points to null
	- the pointer to the beginning of the list is called the head pointer
	- the pointer to the end of the list is called the tail pointer
- each element points to the next and previous element in the list (Doubly linked list)
	- stores twice the ammount of data, but very useful

#### Stack
- Last In First Out (LIFO) structure. User has no access to inner elements
- Modifications to stack only occur at the end (called the top)
- Operations
	- `void push(T items)`
		- adds an item to the top of the stack
	- `T pop()`
		- removes the top item
	- `T peek()`
		- returns the top item
	- `int size()`
		- returns the stack's size
	- `boolean isEmpty()`
		- returns whether or not the stack is empty

#### Queue
- First In First Out (FIFO) structure. User only has access to first element and can only add to the end.
- Operations
	- `void enquee()`
		- adds to the end of the queue
	- `T dequeue()`
		- removes from the front of the queue
	- `T front()`
		- returns the first element
	- `int size()`
		- returns the size of the queue
	- `boolean isEmpty()`
		- returns whether or not the queue is empty
- Deque
	- Similar to queue, but the user can add or remove from the beginning or end
	- Operations
		- `void addFront(item)`
			- adds to the front of the deque
		- `void addBack(item)`
			- adds to the back of the deque
		- `T removeFront()`
			- removes from the front of the deque
		- `T removeBack()`
			- remmoves from the back of the deque

#### Hash Table
- Collection in which items are stored in a table
- Location of a particular item is determined by a "hash function" instead of position relative to rest of the data
	- if a hash function maps two items to the same location we call this a hash colision.
	- ideally a hash function will provide O(1) access time
- Java has built in Hash Functions for general classes (String, Integer, etc), but sometimes it is useful to write your own functions.
	- typically we create a table of size 'm' with indices [0, m-1].
	- then if x = Hash(item) we reduce x % m such that it will fit in the table
- Collision Resolution
	- Seperate chaining
		- use seperate linked list for each bucket
	- Open adressing strategies
		 - items don't always get added to their ideal location
	- Linear Probing
		- find index = Hash(x) % M
		- check if index is empty
		- if index is empty, add item to index
		- if index is not empty
			- for K = 0, 1, ... M-1
				if (index + K) % M is empty
					insert item in (index + k) and end loop
		- For this strategy to be eficient the table needs to have a lot of empty space. (M > what we really "need"). Then theoretically there will be empty spaces where collision-items can be added close to their "actual" space (as per the Hash function) which maximizes search efficiency.
		- Search(item)
			- Find index = Hash(item)
			- check index
			- if Table(index) = item return
			- else
				- for k = 0, 1, M-1
					if Table(index + k) == item, return
					else if Table(index + k != null), keep searching
					else if Table(index + k == null), return

#### Trees
- node, root, internal, external (no children = leaf)
- parent, child, sibling, ancestors, descendents
- edge, path
	- Edge is a link between two nodes
	- Path is the set of edges between two nodes. The path length is the number of edges between two noeds
- depth of node
	- root=0, 1+depth(parent),
- height of node
	- leaf=0, 1+max child height
- ordered tree:
	- children of a node are ordered, ie, book TOC
- Tree Algorithms
	- compute depth of node
	- compute height of node (two methods)
	- traversals
	    - breadth-first
	    - depth-first
		- preorder
		- postorder
	- int size()
		- O(1)
	- boolean isEmpty()
		- O(1)
	- `Iterable<E> breadthFirst()`
	- `Iterable<E> preOrder()`
	- `Iterable<E> postOrder()`
- Binary Trees
	- every node has at most 2 children (called left & right usually)
	- depth-first traversals:
	  - Pre-Order
	      - Check root, then left subtree, then right subtree
	  - Post-Order
	  	  - Check left subtree, then right subtree then root
	  - In-Order
	  	  - Check left subtree, then root, then right subtree
- Binary Tree Properties
	- Every node has at most 2 children
	- assume n nodes, height h:
	    - h < n
	    - log n <= h because n <= 2^(h+1) - 1 = sum (i=0,h) 2^i
	    - nodes at depth i <= 2^i
	- Proper Binary Tree
		- All nodes have 0 or 2 children
	- Complete Binary Tree
		- Full from top to bottom, left to right
	    - h <= log(N)
	- Balanced Binary Tree
		- difference in height of left and right subtrees is at most 1
- Binary Tree Implementations
		- Recursive Linked Structure:
			- Bnode<T> has T data, Bnode<T> parent, Bnode<T> left, Bnode<T> right
		- Ranked Sequence Representation:
			- Use array, waste slot at index 0
			- Root goes into index 1
			- Left child of node at i is at index 2i
			- Right child of node at i is at index 2i+1
			- Parent of node at i is at index i/2 (integer division)
- Binary Search Tree
	- A binary tree where items are placed into the tree based on their value.
	- Algorithms
		- Find K
		```Python
			def find(k, root):
				if (root == null):
					return false
				if (k == root.data):
					return true
				else if (k < root.data):
					return find(k, root.left)
				else:
					return find(k, root.right)
		```
		- Insert K
		```Python
			def insert(k, root):
				if (root == null):
					return new Node(k)
				if (k < root.data):
					root.left = insert(k, root.left)
				else:
					root.right = insert(k, root.right)
		```
		- Remove K
		```Python
			def remove(k, root):
				if (k < root.data):
					root.left = remove(k, root.left)
				else if (k > root.data):
					root.right = remove(k, root.right)
				else:
					if (root.isLeaf): #leaf
						return null
					else if (root.left != null && root.right == null): #only right child
						return root.right
					else if (curr.left == null && curr.right != null): #only left child
						return root.left
					else:
						curr = findMin(root.right)
		                #delete minimum on the right
		                root.right = this.delete(root.right, curr.data)
		                root.data = curr.data
		                return root
		```
	- Example: arithmetic expression evaluation
		- Consider this arithmetic expression: (1+2) * -6 - 2 * (14/5)
		- This binary tree represents it and can be used to compute the result:
			```
		                      -
		                /          \
		               *            *
		             /   \        /   \
		            +     -      2     /
		          /  \     \          /  \
		         1    2     6        14   5
		     ```
		- in-order traversal for fully parenthesized in-fix form: $(((1+2)*(-6))-(2*(14/5)))$
		- pre-order traversal generates pre-fix notation: `- * + 1 2 - 6 * 2 / 14 5`
		- post-order traversal generates post-fix notation: `1 2 + 6 - * 2 14 5 / * -`

- AVL Tree
	- An AVL tree is a binary search tree which must maintain the Height Balance Property
		- The absolute value of the difference between the heights of any node's children must be less than or equal to one.
	- insert, search operations are O(log N)
	- Algorithms
		- insert
		    - insert item as in binary search tree
		    - starting with new node, go up to update heights
		    - if find a node that has become unbalanced
		    - do restructure to rebalance
		    - rebalance reduces height of subtree by 1, so no need to propagate up
		    - O(log n)
		- remove
		    - start as with binary search tree
		    - ancestor of deleted node (not replaced node) may become unbalanced
		    - go up from deleted node to find first unbalanced
		    - do rotation - may reduce the height of the subtree
		    - propagate/check upward to root!
		    - O(log n)
		- balance
			- Check root
				- If left-left imbalance, do right rotation
				- If right-right imbalance, do left rotation
				- If left-right imbalance, do left then right rotation
				- If right-left imbalance, do right then left rotation

![](src/data-struct/avl-rotation.png)

#### Priority Queue
- A data structure, similar to a queue which gets objects with maximum priority.
- Entry object: (key, value) pair, keys must be comparable
- main methods:
	- `insert(entry)`
	- `removeMin()`
	- `min()` (get)
- Hash table: fast inserts depending on collisions
	- Good if there are lots of different priorities
- Balanced binary search tree
	- Operations are log(N)
- (circular) ordered array
	- O(N) to insert
	- O(1) for findMin and deleteMin
- Unordered array
	- O(1) to insert
	- O(N) for findMin, deleteMin
- Note that ordered/unordered arrays could be linked lists too
- Heap
	- complete binary tree:
		- N nodes, height log N
		- full except for rightmost last level
		- last node corresponds to level numbering
	- order property:
		- every node key (priority) <= it's childrens' keys (priorities)
	- Implimentations
		- Ranked Sequence array rep of complete binary tree
			- level numbering:
		        - f(v) = 1 if root
		        - left child = 2f(parent(v))
		        - right child = 2f(parent(v)) + 1
			- root, parent, leftChild, rightChild, isInternal, isExternal, isRoot
			- iterator methods for elements, positions, children
			- worst case array size = N
	- Algorithms
		- Insert
			Algorithm
				- add next leaf node according to next open spot
				- bubble value up based on priority.
			- Time Complexity
				- Add O(1)
				- Bubble Up O(log(N))
		- FindMin
			- Algorithm
				- get root value
			- Time Complexity
				- O(1)
		- DeleteMin
			- Algorithm
				- returns root
				- moves last leaf value to root
				- bubble root value down based on priority
			- Time Complexity
				- remove root O(1)
				- find last
					- Array O(1)
					- Tree O(log(N))
				- Bubble Down O(log(N))
		- Bubble Up (minimum heap)
			- Algorithm
			```Python
				def bubbleUp(index): #based on array implimentation
					if (index <= 1): #at root
			            return;
			        val = heap.get(index)
			        parent = heap.get(index / 2)
			        #if value has lower priority than parent, swap
			        if (val < parent):
			            heap.set(index / 2, val)
			            heap.set(index, parent)
			            bubbleUp(index / 2)
			        return;
			 ```
			- Time Complexity
				- O(log(N))
		- Bubble Down (minimum heap)
			- Algorithm
			```Python
				def bubbleDown(index) #based on array implimentation
					if (2 * index > this.size): #at a leaf
			            return
			        val = this.heap.get(index)
			        childIndex = this.maxChild(index)
			        if (childIndex < 1)
			            return
			        child = this.heap.get(childIndex);
			        #swap if value has higher priority than child, swap
			        if (val > child):
			            heap.set(index, child)
			            heap.set(childIndex, val)
			            bubbleDown(childIndex)
			        return
			```
			- Time Complexity
				- O(log(N))
		- Bottom Up Build
			- Algorithm
				- Put all values in array
				- Starting at height h - 1 check each value and bubble down if neccesary
				- Go to next level and repeat until the root has been reached
			- Time Complexity
				- O(N)
		- Top Down Build (Standard Build)
			- Algorithm
				- Create empty heap
				- Add values one by one, bubble up as needed
			- Time Complexity
				- O(Nlog(N))

#### Graph
- Terminology
	- N vertices (nodes) connected by M edges (lines)
	- Degree of vertex is number of incident edges
	- Adjacent vertices u and v (edge (u, v) exists) are called neighbors
		- directed
			- edges that go in one direction (from u to v, but not v to u)
		- undirected
			- edges taht go in both directions (from u to v and v to u)
	- Path from u to v is a sequence of edges starting at u that take you to v, with no repeated edges (path length = number of edges on it)
	- Cycle in a graph: a path of length at least 1 whose first and last vertices are the same
	- Connected graph G: there is a path in G from every vertex to every other vertex
	- Acyclic graph
		- A graph with no cycles
	- Tree
		- An acyclic connected graph
	- Forest
		- A disjoint set of trees
	- Subgraph of G
		- Subset of graph G’s edges (and associated vertices)
	- Spanning tree of a connected graph G
		- A subgraph that contains all of G’s vertices and is a single tree
- Implimentations of Graph
	- Edge List
		- List of ordered pair edges (u, v)
	- Adjascency List
		- A linked structure containing the vectors, linked to its neighbors
		Example
		```
			[A] -> E -> F
			[B] -> A -> N
			[C] -> M -> P
			...
			[Z] -> B -> C
		```
	- Adjascency Matrix
		- Two dimensional boolean or integer array of indicating when there's an edge between vectors u and v
		Example
		```
	             A B C .. Z
			A
			B
			C
			...
			Z
		```
- Algorithms
	- Traversals and Searhes
		- Breadth-first
			- Approximates level-order tree traversal
			- Move one level further away from starting vertex during each round
		- Depth-first
			- Generalization of pre-order tree traversal
			- Find longest path from start that you can without repeating vertices, then backtrack as needed to try di↵erent long paths until you reach all vertices
	- Topological Sort
		- Goal: order the vertices of a directed, acyclic graph in some sequence so that for each edge (u, v) in the graph, vertex u appears earlier in list than vertex v
			- Note that a vertex's indegree is the number of edges leading to the vertex
			- a vertex's outdegre is the number of edges leading away from the vertex
		Algorithm
			def topologicalSort():
				Collection collection = new List()
				while (!graph.isEmpty):
					v = findVertexInDegreeZero()
					if (v == null):
						return #graph must have cycle, so abort
					collection.add(v)
					graph.remove(v)
	- Shortest Path Problems
		- Single Source, Unweighted Graph (Breadth First Search)
			```Python
				def unweightedBestPath(start):
					for (v in graph):
						dist[v] = infinity
						prev[v] = null
					dist[start] = 0
					prev[start] = -1
					//Perform a breadth-first traversal to process nodes
					queue.enqueue(start)
					while queue is not empty
						x = queue.dequeue()
						For (each v in neighbors(x) && prev[v] == null):
							dist[v] = dist[x]+1
							prev[v] = x
							queue.enqueue(v)
			```
			- Time Complexity
				- O(V + E)
		- Single Source, Weighted Graph (Dijkstra's Algorithm)
=			```Python
				def weightedBestPath(start):
					for each v in graph:
						dist[v] 1
						prev[v] null
						found[v] false
					dist[start] = 0
					prev[start] = -1
					for i in range (0, V):
						x = getUnfoundVertexWithMinDistance()
						found[x] = true
						for each v in neighbors(x):
							if (dist[x] + weight[(x, v)] < dist[v]):
								dist[v] = dist[x] + weight[(x, v)]
								prev[v] = x
			```
			- Time Complexity
				- O(N^2)
		- All Sources Graph
			- Run single source path algorithm V times
			- Time Complexity
				- O(N^3)
	- Minimum Spanning Tree
		- Find the tree that connects all vertices in the graph with the least total cost.
			- Prim's Algorithm
				- Similar to Dijkstra's algorithm. Starts at a source vector and finds the minimum cost path to each other vector
				```Python
					def prim(start):
						Graph newGraph = new Graph()
						for each v in graph:
							dist[v] = infinity
							prev[v] = null
							found[v] = false
						dist[start] = 0
						prev[start] = -1
						for i in range (0, V):
							x = getUnfoundVertexWithMinDistance()
							found[x] = true
							newGraph.add(x)
							for each v in neighbors(x):
								if (weight[(x, v)] < dist[v]):
									dist[v] = weight[(x, v)]
									prev[v] = x
				```
				- Time Complexity
					- O(N^2) or O(Mlog(N))
					- M is the number of edges, N is the number of vertices
			- Kruskal's Algorithm
				- Finds the smallest edge and adds to the tree if adding that edge doesn't create a cycle.
				- Algorithm
				```Python
					def kruskal():
						Graph newGraph = new Graph()
						Heap heap = new Heap() #minHeap
						for each edge in graph:
							heap.add(edge)
						while (!heap.isEmpty):
							e = heap.pull()
							if (!newGraph.willCreateCycle(e)):
								newGraph.add(e)
				```
				- Time Complexity
					- O(Mlog(N))
					- M is the number of edges, N is the number of vertices

#### Skip List
- series of ordered levels, each a doubly linked list with sentinels
	- L0 has all the elements
	- Li has on average 2^(log n - i) elements = n/2^i
	- Lh (top level) has sentinels
	- each entry has collection of links to go across and down
- good choice in practice for sets, maps, dictionaries
- search & update methods are O(log n) expected time (probabilistic analysis)
- uses randomization
- Operations
	- find K
		- start at top leftmost position (-inf)
		- while pos != null
			- skip forward to largest key <= K
			- move down 1 position
	- insert K
		- do (skip)search to find position in L0
		- insert K in L0
		- flip coin to decide if insert K in next level
		- keep flipping and inserting up until no
	- remove K
		- do search (find) to find position in L0
		- remove from each level as you search downward
```
Example

Lh:  s                                            e
L3:  s       17                  42               e
L2:  s       17      31          42          55   e
L1:  s   12  17      31  38      42  44      55   e
L0:  s   12  17  20  31  38  39  42  44  50  55   e
```

#### Treap
- Each key is given a random priority and inserted as in a binary search tree (not balanced. The nodes are rotated so that the priorities form a (max)heap
	- The idea is to use random priorities to roughly rebalance tree
- Operations
	- find K
	    - do standard binary tree search ignoring priorities
	- insert K
	    - generate random priority for key K
	    - binary search where the new leaf for K belongs (BST on keys)
		- while new node has higher priority than parent, rotate nodes
	- delete K
	    - if K is at leaf node, just remove
		- if K has one child, replace with child
		- if K has two children, do like BST remove and then fix heap property:
	        - swap K node with key order successor, rotate to restore heap
		- alternative: find K, change priority to -1, rotate down to
	        leaf then remove

#### Splay Tree
- These are an alternative Binary Search Tree implementation of a map,
dictionary or set where elements are moved up to the root each time they are
accessed so that their next access is faster.  This is still a binary
search tree, but not necessarily balanced.
- Splaying
	- rotations after every operation (including find) to move accessed node to root:
		- zig (if accessed node is child of root)
		- zig-zag (like double AVL rotation (LR or RL))
		- zig-zig (2 single rotations: parent-grandparent first, then child-parent)
	    - continue splaying upwards until the splayed node is the root
	- which node to splay:
		- find: found node or leaf where search ends (if not found)
		- insert: inserted node
		- remove: parent of actual leaf node that gets removed
	- complexity
		- O(d) to splay node at depth d
		- worst case O(h) for any operation (splay leaf)
		- M operations cost O(M log N) time
		- amortized cost of each operation is O(log n), some better, some worse

#### B Tree
- m-ary search tree (m=2 gives us a binary search tree)
- tree is always balanced!
- each node holds up to m-1 values
- for each internal node:  ceil(m/2) <= # children <= m
- the values in the nodes are ordered and direct search to a child
- height is O(log N) with base m (constant of m within log base 2)
- as values are inserted, nodes are split as necessary in a process that recurses up to the root if necessary
	- this is how balance is maintained
- useful for disk accesses instead of putting all data in memory:
	- for large m, each node is stored on a block of memory
	- do binary search to find a value in a node, or direct to child
	- each block access (as you go to the next level) is a disk access
	- there are at most O(log N) disc accesses

#### Algorithmic Time Complexity
- Counting the number of basic operations
	- Not neccesarily exact
	- Big-O notation isused to describe the upper bound of time complexity
	- Gives an idea of how quickly/efficiently a method will run.
	- Measuring run times
		- single assignments, operations, input/output constant
		- statemtnts in sequence
			- sum of statement times
		- decision statemeents
			- max of its parts + conditions (test boolean) execution time
		- loop
			- maximum number of iterations multiplied by the sum of body statements + condition execution time
			- nested lops can be very complicated
		- method call
			- sum of all statements in method body
- Big O, Big Omega, and Big Theta
	- Let T(N) represent the worst case running time of an algorithm with input size N.
		- by default we measure the worst case scenario
	- Big O: T(N) ~ O(f(N)) if there exists a 'c' and an 'n' such that T(N) <= cf(N) for all N >= n.
		- Upper bound function of worst case runtime.
		Note: We use '~' to represent "is an element of".
		- There are a large number of functions that can bound any given method. We then use the smallest possible function which satisfies the Big O conditions
	- Big Omega: T(N) ~ Omega(g(N) if there exists c, n such that T(N) >= cg(N) for all N >= n.
		- Lower bound function of worst case runtime
		- All functions have a g(N) of c (constant time)
	- Big Theta: T(N) ~ Theta(h(N)) if and only iff T(N) ~ O(h(N)) and T(N) ~ Omega(N)).
		Sandwhich theorem: cf(N) and cg(N) are of the same type, but c and n can differ. Upper bounded by cf(N) and lower bounded by cg(N). THerefore we can use Theta(N) to describe a range of worst case runtimes.

	- Example
	Linear Search algorithm
	```Java
		for (int i = 0; i<N; i++){
			if (a[i] == value) {
				return i;
			}
		}
		return -1
	```
	- Analysis
	```
		initializing i = 1 operation
		Worst case N iterations
			inside loop body = 1 operation
			array access = 1 operation
			iterate i = 2 operations (because i = i+1)
			check condition i<N = 1 operation
		Last condition (i<N) check = 1 operation
		return = 1 operation
	```
	Then $T(N) = 5N + 3$ and for all $N >= n$
	$T(N) <= cN$ for $c = 4$ and $n = 3$ Therefore $T(N) ~ O(N)$
	$T(N) >= cN$ for $c = 3$ and $n = 1$ Therefore $T(N) ~ Omega(N)$
	Therefore $T(N) ~ Theta(N)$
- Standard T(N) Functions (in order of efficiency)
	- c
		- constant time
	- log log log ... log(N)
		- multi log time
	- log(N)
		- logarithmic time
	- (log(N))^2
		- log squared time
	- (log(N))^k
		- log k time
	- N
		- linear time
	- Nlog(N)
		- linearithmic time (or just Nlog(N) time)
	- N^2
		- qadratic time
	- N^3
		- cubic time
	- c^N
		- exponential time

#### Sorting
- Insertion Sort
	- for each value, move left as far as it needs to go
	- repeat for each position i from 2 to N
	- Time Complexity
		- O(N^2)
	```
		6 5 3 1 8 7 2 4
		5 6 3 1 8 7 2 4
		3 5 6 1 8 7 2 4
		1 3 5 6 8 7 2 4
		1 3 5 6 7 8 2 4
		1 2 3 5 6 8 7 4
		1 2 3 4 5 6 7 8
	```
- Bubble Sort
	- from left to right, compare adjacent values, swap if out of order
	- repeat N-1 times or until no changes
	- largest values bubble to the end
	- Time Complexity
		- O(N^2)
	```
		6 5 3 1 8 7 2 4
		5 3 1 6 7 2 4 8
		3 1 5 6 2 4 7 8
		1 3 5 2 4 6 7 8
		1 3 2 4 5 6 7 8
		1 2 3 4 5 6 7 8
	```
- Selection Sort
	- mimimum based: find smallest value and swap into position
	- repeat for each position i from 1 to N-1
	- smallest values are placed into the correct positions
	- Time Complexity
		- O(N^2)
	```
		6 5 3 1 8 7 2 4
		1 6 5 3 8 7 2 4
		1 2 6 5 3 8 7 4
		1 2 3 6 5 8 7 4
		1 2 3 4 6 5 8 7
		1 2 3 4 5 6 8 7
		1 2 3 4 5 6 7 8
	```
- Merge Sort
	- Split array recursively by half until each array has length of 1
	- Merge and sort the arrays
	- Time Complexity
		- O(Nlog(N))
	```
		6 5 3 1 8 7 2 4
		6 5 3 1  8 7 2 4
		6 5  3 1  8 7  2 4
		6  5  3  1  8  7  2  4
		5 6  1 3  7 8  2 4
		1 3 5 6  2 4 7 8
		1 2 3 4 5 6 7 8
	```
- Quick Sort
	- Pick pivot element
		- Median of three: median of first, middle, last
	- Swap pivot to last position
    - Work from both ends to make swaps
    	- Iterate from both sides
    	- If left[i] > pivot and right[j] < pivot, swap them
    	- Every element left is <, every element right is >=
    - When ends join, put pivot element at union spot, then recurse
	- Time Complexity:
		- worst case O(N^2) if subsection only shrinks by 1 each time
		- best case O(N log N) (divide problem in half each time)
		- randomized version - get expected O(N log N) time
		- in practice: faster than mergesort, better space
	```
		20 13 7 71 31 10 5 50 17
		median of: 20, 31, 17 -> 20  pivot
		17 13 7 71 31 10 5 50 |20
		17 13 7 5 31 10 71 50 |20
		17 13 7 5 10 31 71 50 |20
		17 13 7 5 10 |20| 71 50 31
		first pass done, recurse on each partition
		17 13 7 5 10 20 71 50 31
		left median of 3: 17, 7, 10 => 10
		right median of 3: 71, 31, 50 => 50
		5  13  7  17 |10 |20| 71  31  |50
		5  7  13  17 |10 |20| 31  71  |50
		5  7  |10|  17 13 |20| 31  |50|  71
		Left, left median of 3: 5, 7
		left, right median of 3: 17, 13
		5 7 10 13 17 20 31 50 71
	```
- Bucket Sort
	- Add all values to an HashTable with M buckets
	- Then remove in order
	- Time Complexity
		- O(N + M)
	```
		6 5 3 1 8 7 2 4
		[1] -> 1
		[2] -> 2
		[3] -> 3
		[4] -> 4
		[5] -> 5
		[6] -> 6
		[7] -> 7
		[8] -> 8
		1 2 3 4 5 6 7 8
	```
- Heap Sort
	- Build a heap (bottom up or top down)
	- Remove items from heap one by one
	- Time Complexity
		- O(N log N)
- Radix Sort
	- Used to sort items using multiple paramaters
		- For example string containing letters and numbers (ie 3A33B5) can be sorted in order by comparing each value.
	- Start with rightmost dimension and create HashTable based on possible values
		- for integer 0 - 9
		- for character unicode range
	- Add values to HashTable (hashOne) based on current dimension values
	- Create new HashTable (hashTwo) based on new dimensions
	- Remove values from hashOne and add to hashTwo
	- Repeat until all dimensions have been checked
	- Time Complexity
		- O(d(M + N))

#### Text Processing
- Strings are usually encoded as arrays of characters, each represented by a certain number of bits
- Then we must decide the bit combinations to store each character
- Fixed Length
	- Fixed length encoding scheme give each character a fixed number of bits
		- For an alphabet of 26 characters and 1 space we would need at least 27 possible combinations. This can be achieved using characters with a fixed bit length of 5 because 2^4 < 27 < 2^5
			Example
			A = 00000
			B = 00001
			C = 00010
			...
		- Unicode - 8 bits/character
		- This type of scheme may result in a creating more bits than neccesary
- Variable-length encoding scheme:
	- Use character frequencies to make specialized encoding,
	- more frequent characters have shorter codes.
	- issue is knowing when one character stops and the next starts in a
	  bit stream
	- "prefix code": no encoding is a prefix of any other encoding
	- Huffman Encoding Scheme
		- use variable length encodings for characters in text based on frequency
			- shorter codes for more frequent characters
			- no code can be a prefix of another (for decoding purpose)
			- called "prefix code"
		- binary tree to represent the code
			- left is 0, right is 1
			- characters stored at leaves => prefix code guarantee
			- frequences stored in nodes
		    - path from root to leaf gives encoding for that character
		    - create by repeatedly merging min frequency trees into larger one
			- O(N) to read file (compute frequencies) and then encode
			- construct optimal code in O(C log C) time w/heap PQ
			- need to put encoding table in result file for decoding
		- Example
			```
				                    24
				               /           \
				             10             14
				           /   \          /    \
				         4      6        6         8
				       /  \    / \      / \       /   \
				      2   2   3   3    3   3    4       4
				      a   r   e   t   / \  sp  / \     / \
				                     1  2     2  2     2   2
				                     !  u    /\  /\    /\  /\
				                            W l o  v  D S c  s

				a 000
				r 001
				e 010
				t 011
				! 1000
				u 1001
				sp 101
				W 11000
				l 11001
				o 11010
				v 11011
				D 11100
				S 11101
				c 11110
				s 11111
			```
- Tries - more generalized approach
	- trees for storing strings (the actual text to be searched)
	- used for pattern matching & prefix matching
	- especially good for actual word matching
	```
		Example - consider this passage: She sells seashells by the
		seashore and he makes mounds of mud in the mountains.

		 a  b  h  i      m      o      s     t
		 |  |  |  |   /  |  \   |     / \    |
		 n  y  e  n  a   o   u  f    e   h   h
		 |           |   |   |      / \  |   |
		 d           k   u   d     a   l e   e
		             |   |         |   |
		             e   n         s   l
		             |  / \       /    |
		             s  d t      h     s
		                | |     / \
		                s a    e   o
		                 ins  lls  re   (compressed for brevity)
	```
	- General approach:
		- let S be set of strings from alphabet A s.t. none is prefix of another
			- trie nodes labelled with characters from A (except root)
			- path from root to external node forms a string in text
			- order children according to alphabet
		- properties for S strings in C size alphabet with M chars total in text
			- internal node has <= C children
			- has S external nodes
			- height = length of longest string
			- #nodes is O(M)
			- construction takes O(CM)
			- search for pattern size N takes O(CN)
		- compressed trie (Patricia trie)
			- combine nodes with single children that form chain
			- store substring indices of where each chain occurs in the original passage instead of all the characters at nodes (so only need to store 2 integers instead of multiple characters)
			- nodes is O(S)

#### JUnit Testing
- Tags
	- `@Before`
		- Tags a method that will run before each test
	- `@BeforeClass`
		- Tags a method that runs once at the beginning of the test class
	- `@Test`
		- Tags a test
	- `@After`
		- Tags a method that will run after each test
	- `@AfterClass`
		- Tags a method that runs once at the end of the test class
- Import Statements
	```Java
		import static org.junit.Assert.assertEquals;
		import static org.junit.Assert.assertNotNull;
		import static org.junit.Assert.assertNotSame;
		import static org.junit.Assert.assertNull;
		import static org.junit.Assert.assertSame;
		import static org.junit.Assert.assertTrue;
		import static org.junit.Assert.assertFalse;
		import static org.junit.Assert.fail;
		import org.junit.Before;
		import org.junit.BeforeClass;
		import org.junit.Test;
	```
- Compile and Run
	```Bash
		javac -cp <junit file name>.jar;. <class>.java
		java -cp <junit file name>.jar; <hamcrest jar file>.jar org.junit.runner.JUnitCore <class>
	```
	- Replace ; with : for OSX and Linux
