---
title: Artificial Intelligence Notees
author: Haroon Ghori
geometry: margin=1in
---

### Agents
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
 		```Python
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
 	- Depth First Search
 		- Strategy is to expand the deepest node (path) first. 
 		- We treat the fringe as a stack and push successors onto the fringe in a defined way (ie always in the same direction order). This allows us to explore the deepest path first.  
 		```Python 
 			def depthFirstSearch(problem):
			    visited, stack = set(), util.Stack()
			    desc = dict()
			    stack.push((problem.getStartState(), None, 1))
			    lst = []
			    while True:
			        (vertex, step, cost) = stack.pop()
			        if problem.isGoalState(vertex):
			            curr = (vertex, step, cost)
			            break
			        if vertex not in visited:
			            visited.add(vertex)
			            parent = (vertex, step, cost)
			            for successor in problem.getSuccessors(vertex):
			                if successor[0] not in visited:
			                    stack.push(successor)
			                    desc[successor] = parent
			    while curr[0] != problem.getStartState():
			        lst.append(curr[1])
			        curr = desc[curr]
			    lst.reverse()
			    return lst
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
			    desc = dict()
			    queue.push((problem.getStartState(), None, 1))
			    lst = []
			    while True:
			        (vertex, step, cost) = queue.pop()
			        if problem.isGoalState(vertex):
			            curr = (vertex, step, cost)
			            break
			        if vertex not in visited:
			            visited.add(vertex)
			            parent = (vertex, step, cost)
			            for successor in problem.getSuccessors(vertex):
			                if successor[0] not in visited:
			                    queue.push(successor)
			                    desc[successor] = parent
			            print problem.getSuccessors(vertex)
			    while curr[0] != problem.getStartState():
			        lst.append(curr[1])
			        curr = desc[curr]
			    lst.reverse()
			    return lst
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
 		- We treat the fringe as a queue. This allows us to explore the fringe by exploring the cheapest nodes first. 
 		```Python
 			def uniformCostSearch(problem):
			    visited, p_queue = set(), util.PriorityQueue()
			    desc = dict()
			    p_queue.push((problem.getStartState(), None, 0), 0)
			    lst = []
			    while True:
			        (vertex, step, cost) = p_queue.pop()
			        if problem.isGoalState(vertex):
			            curr = (vertex, step, cost)
			            break
			        if vertex not in visited:
			            visited.add(vertex)
			            parent = (vertex, step, cost)
			            for successor in problem.getSuccessors(vertex):
			                if successor[0] not in visited:
			                    successor = (successor[0], successor[1], successor[2] + cost)
			                    p_queue.push(successor, successor[2])
			                    desc[successor] = parent
			    while curr[0] != problem.getStartState():
			        lst.append(curr[1])
			        curr = desc[curr]
			    lst.reverse()
			    return lst
 		```
 		- If the solution costs $C^* $ and arcs cost at least $\epsilon$ then the effective depth is roughly $c^* / \epsilon$. 
 		- Complete: Yes
 		- Optimal: Yes
 		- Time complexity: $O(b^{$C^* / \epsilon})$. 
 		- Space complexity: $O(b^{$C^* / \epsilon})$. 










