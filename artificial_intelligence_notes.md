---
title: Artificial Intelligence Notees
author: Haroon Ghori
geometry: margin=1in
---

### Search Problem
- An agent in articial intelligence is an artificial actor in an AI problem. 
- Reflex agents 
	- An agent whose actions are chosen by a specific set of inputs. 
	- A reflex agent is a direct mapping from a set of inputs / memory to a specific output with no consideration for possible future outcomes. 
	- A reflex agent can be rational if your function is complex enough. 
- Planning agents 
	- A planning agent is an agent that asks "what if". 
	- A planning agent creates a model of the world and how it __would__ respond to actions.
		- This requires that the agent has a model of the world and some kind of goal. 
	- Complete planning agent: Finds a way to achieve its goals
	- Optimal planning agent: Finds the best solution based on a given cost function. 
	- Planning agent: Comes up with a single complete plan and executes it.
	- Replanning agent: Comes up with many plans one after the other. (ie accomplishes smaller goals)
- Search problem 
	- Getting to a specific goal.
	- State space: encodes how the world is at a moment in the plan. 
	- Succesor function: models how you think the world works. 
		- Where can i get to immedietly, how can I achieve that, and what changes
	- Start state: where we begin 
	- Goal tet: a state that constitutes an end / success
	- Solution: A sequence of actions which transforms the start stae into the goal test. 
- Search problems are always models of the world. If we add too many variables the problem becomes unsolvable or not realistically or easily solvable. 
	- World State: actual representation of all variables in the world 
		- Too large 
	- Problem State (Search space): subset of world state variables that are relevant to the problem 
		- Still very large
		- Would like to not enumerate the entire search space.
		- Can make any programatic choice for encodin search space. 
- State space graph - a mathematical representation of a search problem
	- Nodes are some abstarction of word configuration
	- Arcs represent successors (results of actions)
	- Goal test is a set of goal nodes.
		- Think terminal state in DFA 
	- In a search graph each state occurs only once (regardless of the number of ways to get to it). 
	- Generally a state space graph is too big to actually build. 
- Search Tree 
	- root : current state 
	- Children are the successor states. 
	- Nodes show states, but they are actually __plans__ to reach that state (a node in the tree is a path in the graph). 
	- This tree is also too big to feasibly store, so for most problems we don't actually build the entire tree. 
- Searching with a search tree
	- When searching with a search tree, the idea is to expand out potential plans and maintain a "fringe" of partial plans under construction. 
	- We are trying to expand out as few plans as possible.
	```
	def tree_search(problem, strategy):
		initiate the search tree using the initial state of problem
		do:
			get next node based on strategy
			if there are no candidates for expansion:
				return False
			if the node contains a goal state:
				return the corresponding solution (path from start to node)
			else:
				expand the node and add the resulting nodes to the search tree
	```
	- The biggest question is how you explore the fringe nodes.
		- ie what is `strategy`?
- __Note that in these search algorithms we are literally searching the search tree itself.__ 
- Depth First Search
	- Strategy is to expand the deepest node (path) first. 
	- We treat the fringe as a stack and push successors onto the fringe in a defined way (ie always in the same direction order). This allows us to explore the deepest path first.  
	```Python 
def depthFirstSearch(problem):
    visited, stack = set(), util.Stack()
    curr = (problem.getStartState(), [], 0)
    while not problem.isGoalState(curr[0]):
        (state, steps, cost) = curr
        if state not in visited:
            visited.add(state)
            for suc in problem.getSuccessors(state):
                if suc[0] not in visited:
                    tmp = steps + [suc[1]]
                    suc = (suc[0], tmp, suc[2])
                    stack.push(suc)
        curr = stack.pop()
    return curr[1]
	```
	- Complete: If there are no cycles - will always find a path. 
	- Optimal: No - will not always find the best path. Will always find the x-most path if x is the direction we push onto the fringe first. 
	- Time Complexity: Exponential $O(b^m)$ where $m$ is the depth of the tree. 
	- Space Complexity: $O(b * m)$ where $b$ is the branching factor and $m$ is the depth of the tree. 
	- This is a general view of the search tree for DFS.
	```
               |        /\
               |       /--\
             m |      /  b \
               |     /      \
               |     --------
	```
- Breadth First Search 
	- Strategy is to expnd the shallowest node (path) first.
	- Push nodes onto the fringe in order of shallowness. 
	- We treat the fringe as a queue. This allows us to explore the fringe a layer at a time. 
	```Python
def breadthFirstSearch(problem):
    visited, queue = set(), util.Queue()
    curr = (problem.getStartState(), [], 0)
    while not problem.isGoalState(curr[0]):
        (state, steps, cost) = curr
        if state not in visited:
            visited.add(state)
            for suc in problem.getSuccessors(state):
                if suc[0] not in visited:
                    tmp = steps + [suc[1]]
                    suc = (suc[0], tmp, suc[2])
                    queue.push(suc)
        curr = queue.pop()
	return curr[1]
	```
	- Complete: Yes - if a solution exists, BFS must find it. 
	- Optimal: Yes - if the costs are all 1. 
	- Time complexity: Exponential $O(b^s)$ where $s$ is the level of the shallowest solution. 
	- Space complexity: $O(b^s)$ where $b$ is the branching factor and $s$ is the depth of the shallowest level. 
	- This is a general view of the search tree for DFS.
	```
               |        /\
             s |       /--\
               |      /  b \
                     /      \
                     --------
	```
- DFS vs BFS 
	- BFS if the solutions are relatively shallow.
	- DFS if the depth is small compared to the branching factor. 
- Iterative Deepening
	- Since DFS has a better space complexity than BFS we may want to take advantage of that. 
	- Run DFS with a depth liimt of 1.
	- Run DFS with depth limit 2.
	- Run DFS with depth limit 3.
	- Keep going until DFS returns solution. 
- Uniform Cost Search 
	- Very similar to BFS but we prioritize by cost instead of depth.
	- Push nodes (paths) onto the fringe in order of cost (lowest cost first). 
	- We treat the fringe as a prioirity queue (lowest cost --> highest priority). This allows us to explore the fringe by exploring the cheapest nodes first. 
	```Python
def uniformCostSearch(problem):
    visited, p_queue = set(), util.PriorityQueue()
    curr = (problem.getStartState(), [], 0)
    while not problem.isGoalState(curr[0]):
        (state, steps, cost) = curr
        if state not in visited:
            visited.add(state)
            for suc in problem.getSuccessors(state):
                if suc[0] not in visited:
                    tmp = steps + [suc[1]]
                    suc = (suc[0], tmp, suc[2] + cost)
                    p_queue.push(suc, suc[2])
        curr = p_queue.pop()
    return curr[1]
	```
	- If the solution costs $C^* $ and arcs cost at least $\epsilon$ then the effective depth is roughly $c^* / \epsilon$. 
	- Complete: Yes
	- Optimal: Yes
	- Time complexity: $O(b^{$C^* / \epsilon})$. 
	- Space complexity: $O(b^{$C^* / \epsilon})$. 
- The search algorithms we've talked about above have been "uninformed search". That means that they haven't taken into account the direction (in the search treee) where the solution may be. Informed search algorithms use __heuristic function__ to infer the best successors to avoid expanding too much of the search tree. 
- Heuristic - a function that estimates how close a state is to a goal. A heuristic is designed for a particular search problem. 
- Greedy Search - expand the node that you think is closest to a goal state. 
- A* search 
	- Essentially a combination of UCS and Greedy Search. 
	- Push states onto the fringe with $priority = path_cost(state) + heuristic(state)$. So explore nodes with the lowest estimated heuristic (closest to goal) AND path cost (cheap to reach). 
	```Python
	def aStarSearch(problem, heuristic):
    visited, p_queue = set(), util.PriorityQueue()
    curr = (problem.getStartState(), [], 0)
    while not problem.isGoalState(curr[0]):
        (state, steps, cost) = curr
        if state not in visited:
            visited.add(state)
            for suc in problem.getSuccessors(state):
                if suc[0] not in visited:
                    tmp = steps + [suc[1]]
                    suc = (suc[0], tmp, suc[2] + cost)
                    p_queue.push(suc, suc[2] + heuristic(suc[0], problem))
        curr = p_queue.pop()
    return curr[1]
	```
	- Complete: Yes 
	- Optimal: Yes
	- Time Complexity: Exponential
	- Space complexity: Exponential
- Admissibility 
	- Inadmisible heuristics break optimality by trapping good plans on the fringe 
	- Admissible heuristics slow down bad plans but never outweigh true costs. 
	- A heuristic $h$ is admissible if: $0 \leq h(n) \leq h^{ * } (n)$ where $h^{ * } (n)$ is the true cost to the nearest goal. 
	- Proof - Optimality of A* 
		- In a search tree we have two goal states, G (good) and B (bad). 
		- $f(x)$ is the priority of state x. 
		- If B is on the fringe, some ancestor n of G is on the fringe too. 
		- We can say tht n will be expanded before B because 
			- $f(n) \leq f(G)$ because h is admissible
			- $f(G) \leq f(B)$ because B is suboptimal
				- $f(G) = c_G + h(G)$, $f(B) = c_B + h(B)$ and $h(G) = h(B) = 0$ since h is admissible. 
				- Since B is bad, $c_G \leq c_B$. 
	- Given two heuristics $h_a$ and $h_c$ and $\forall n \ h_a(n) \geq h_c(n)$, $h_a$ dominates $h_c$. 
	- Heuristics form a semi lattice. The maximum of all admissible heuristic is the max heuristic and the minimum of all admissible heuristics returns 0. 
- A* search is used in everything from video games and robot movement to natural language. 
- Graph Search 
	- Every time we expand a state, we keep track of the set of states already expanded. 
	- Expand the search tree node-by-node but before expanding a node, check to make sure its state is new. If not new, skip it. 
	- This set of expanded nodes is called a __closed set__. 
	- Consistency - for any node $N$ and each successor $P$ of $N$, the estimated cost of reaching the goal from $N$ is no greater than the step cost of getting to $P$ plus the estimated cost of reaching the goal from $P$. 
	$$h(N) \leq c(N \to P) + h(P)$$
	- Consistency is stronger than admissibility - that is, a consistent heuristic is admissible. 
	- The graph search implimentation A* search is complete with a consistent heuristic. 
	








