---
title: Artificial Intelligence Notees author: Haroon Ghori geometry: margin=1in
---

## Search Problem

### Planning Problems / Tree Search 
- An agent in articial intelligence is an artificial actor in an AI problem.
- Reflex agents
	- An agent whose actions are chosen by a specific set of inputs.
	- A reflex agent is a direct mapping from a set of inputs / memory to a
	  specific output with no consideration for possible future outcomes.
	- A reflex agent can be rational if your function is complex enough.
- Planning agents
	- A planning agent is an agent that asks "what if".
	- A planning agent creates a model of the world and how it __would__
	  respond to actions.
		- This requires that the agent has a model of the world and some kind
		  of goal.
	- Complete planning agent: Finds a way to achieve its goals
	- Optimal planning agent: Finds the best solution based on a given cost
	  function.
	- Planning agent: Comes up with a single complete plan and executes it.
	- Replanning agent: Comes up with many plans one after the other. (ie
	  accomplishes smaller goals)
- Search problem
	- Getting to a specific goal.
	- State space: encodes how the world is at a moment in the plan.
	- Succesor function: models how you think the world works.
		- Where can i get to immedietly, how can I achieve that, and what
		  changes
	- Start state: where we begin
	- Goal tet: a state that constitutes an end / success
	- Solution: A sequence of actions which transforms the start stae into the
	  goal test.
- Search problems are always models of the world. If we add too many variables
  the problem becomes unsolvable or not realistically or easily solvable.
	- World State: actual representation of all variables in the world
		- Too large
	- Problem State (Search space): subset of world state variables that are
	  relevant to the problem
		- Still very large
		- Would like to not enumerate the entire search space.
		- Can make any programatic choice for encodin search space.
- State space graph - a mathematical representation of a search problem
	- Nodes are some abstarction of word configuration
	- Arcs represent successors (results of actions)
	- Goal test is a set of goal nodes.
		- Think terminal state in DFA
	- In a search graph each state occurs only once (regardless of the number
	  of ways to get to it).
	- Generally a state space graph is too big to actually build.
- Search Tree
	- root : current state
	- Children are the successor states.
	- Nodes show states, but they are actually __plans__ to reach that state (a
	  node in the tree is a path in the graph).
	- This tree is also too big to feasibly store, so for most problems we
	  don't actually build the entire tree.
- Searching with a search tree
	- When searching with a search tree, the idea is to expand out potential
	  plans and maintain a "fringe" of partial plans under construction.
	- We are trying to expand out as few plans as possible.\
```
def tree_search(problem, strategy): 
	initiate the search tree using the initial
		state of problem 
	do: 
		get next node based on strategy 
		if there are no candidates for expansion: 
			return False 
		if the node contains a goal state:
			return the corresponding solution (path from start to node) 
		else: expand the node and add the resulting nodes to the search tree
```
	- The biggest question is how you explore the fringe nodes.
		- ie what is `strategy`?
- __Note that in these search algorithms we are literally searching the search
  tree itself.__
- Depth First Search
	- Strategy is to expand the deepest node (path) first.
	- We treat the fringe as a stack and push successors onto the fringe in a
	  defined way (ie always in the same direction order). This allows us to
	  explore the deepest path first.

	```Python 
	def depthFirstSearch(problem):
		visited, stack = set(), Stack()
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
	- Optimal: No - will not always find the best path. Will always find the
	  x-most path if x is the direction we push onto the fringe first.
	- Time Complexity: Exponential $O(b^m)$ where $m$ is the depth of the tree.
	- Space Complexity: $O(b * m)$ where $b$ is the branching factor and $m$ is
	  the depth of the tree.
- Breadth First Search
	- Strategy is to expnd the shallowest node (path) first.
	- Push nodes onto the fringe in order of shallowness.
	- We treat the fringe as a queue. This allows us to explore the fringe a
	  layer at a time. 
	
	```Python 
	def breadthFirstSearch(problem):
	    visited, queue = set(), Queue()
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
	- Time complexity: Exponential $O(b^s)$ where $s$ is the level of the
	  shallowest solution.
	- Space complexity: $O(b^s)$ where $b$ is the branching factor and $s$ is
	  the depth of the shallowest level.
- DFS vs BFS
	- BFS if the solutions are relatively shallow.
	- DFS if the depth is small compared to the branching factor.
- Iterative Deepening
	- Since DFS has a better space complexity than BFS we may want to take
	  advantage of that.
	- Run DFS with a depth liimt of 1.
	- Run DFS with depth limit 2.
	- Run DFS with depth limit 3.
	- Keep going until DFS returns solution.
- Uniform Cost Search
	- Very similar to BFS but we prioritize by cost instead of depth.
	- Push nodes (paths) onto the fringe in order of cost (lowest cost first).
	- We treat the fringe as a prioirity queue (lowest cost --> highest
	  priority). This allows us to explore the fringe by exploring the cheapest
	  nodes first.

	```Python 
	def uniformCostSearch(problem):
	    visited, p_queue = set(), PriorityQueue()
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

	- If the solution costs $C^* $ and arcs cost at least $\epsilon$ then the
	  effective depth is roughly $c^* / \epsilon$.
	- Complete: Yes
	- Optimal: Yes
	- Time complexity: $O(b^{$C^* / \epsilon})$.
	- Space complexity: $O(b^{$C^* / \epsilon})$.
- The search algorithms we've talked about above have been "uninformed search".
  That means that they haven't taken into account the direction (in the search
  treee) where the solution may be. Informed search algorithms use __heuristic
  function__ to infer the best successors to avoid expanding too much of the
  search tree.
- Heuristic - a function that estimates how close a state is to a goal. A
  heuristic is designed for a particular search problem.
- Greedy Search - expand the node that you think is closest to a goal state.
- A* search
	- Essentially a combination of UCS and Greedy Search.
	- Push states onto the fringe with $priority = path_cost(state) +
	  heuristic(state)$. So explore nodes with the lowest estimated heuristic
	  (closest to goal) AND path cost (cheap to reach).

	```Python 
	def aStarSearch(problem, heuristic=nullHeuristic):
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
	- Inadmisible heuristics break optimality by trapping good plans on the
	  fringe
	- Admissible heuristics slow down bad plans but never outweigh true costs.
	- A heuristic $h$ is admissible if: $0 \leq h(n) \leq h^{ * } (n)$ where
	  $h^{ * } (n)$ is the true cost to the nearest goal.
	- Proof - Optimality of A*
		- In a search tree we have two goal states, G (good) and B (bad).
		- $f(x)$ is the priority of state x.
		- If B is on the fringe, some ancestor n of G is on the fringe too.
		- We can say tht n will be expanded before B because
			- $f(n) \leq f(G)$ because h is admissible
			- $f(G) \leq f(B)$ because B is suboptimal
				- $f(G) = c_G + h(G)$, $f(B) = c_B + h(B)$ and $h(G) = h(B) =
				  0$ since h is admissible.
				- Since B is bad, $c_G \leq c_B$.
	- Given two heuristics $h_a$ and $h_c$ and $\forall n \ h_a(n) \geq
	  h_c(n)$, $h_a$ dominates $h_c$.
	- Heuristics form a semi lattice. The maximum of all admissible heuristic
	  is the max heuristic and the minimum of all admissible heuristics returns
	  0.
- A* search is used in everything from video games and robot movement to
  natural language.
- Graph Search
	- Every time we expand a state, we keep track of the set of states already
	  expanded.
	- Expand the search tree node-by-node but before expanding a node, check to
	  make sure its state is new. If not new, skip it.
	- This set of expanded nodes is called a __closed set__.
	- Consistency - for any node $N$ and each successor $P$ of $N$, the
	  estimated cost of reaching the goal from $N$ is no greater than the step
	  cost of getting to $P$ plus the estimated cost of reaching the goal from
	  $P$. $$h(N) \leq c(N \to P) + h(P)$$
	- Consistency is stronger than admissibility - that is, a consistent
	  heuristic is admissible.
	- The graph search implimentation A* search is complete with a consistent
	  heuristic.

### Constraint Satisfaction Problem (CSP)
- Identification problem: the goal itself is important as opposed tot he path.
- Constraint Satisfaction Problem: A subset of search problems. Come up with
  some variable assignment that satisfies a set of constraints.
	- A state is defined by varilables $X_i$ with values froma a domain $D$.
	- The goal test is a set of constraints specifying allowable combinatiosn
	  of values for subsets of variables.
- Constraint Satisfaction Problem is a simple example of aformal representation
  language. Allows useful general purpose algorithms with more power than
  standard search algorithms.
- Example: Color a map of australia such that each state is a different color.
	- Variables: WA, NT, Q, NSW, V, SA, T (states)
	- Domains: (red, green, blue)
	- Constraints:
		- WA != NT (implicit constraint)
		- $(WA, NT) \in \{red,\ green), (red, blue),...)$
	- Solutions are assignments satisfying all constraints

	```
	 NT --- WA -- Q
	  \   /  \   /
	    SA -- NSW
	     \     /
	      \   /
	        V

	        T
	```

- Constraint graph: nodes are variable, edges show constraints.
- CSPs can have finite or infinite domains and coninuous or discrite domains.
- Constraints can be unary (restriction on one variable), binary (restriction
  between two variables), or higher order.
- Applying standard search to CSP
	- Iinitial State: Nothing has been assigned.
	- Current State: defined by values assigned (is a partial assignment of
	  values).
	- Successor function: assign k value to an unassigned variabl.
	- Goal Test: tests if the current assignment is complete and satisfies all
	  constraints.
- Using this approach we could apply BFS, DFS, UCS, or A* search to a CSP.
  However, these are very iniefficient on their own. Consider the map coloring
  case - the successor function of a standard search problem doesn't consider
  the restrictins of a CSP. So using one of these algorithms would (generally)
  require traversing the entire search tree.
- Backtracking Search
	- Assign one variabel at a time
	- Check constraints as you go.

	```
	def backtracking_search(csp): 
		return recursive_backtrack({}, csp)

	def recursive_backtrack(assignment, csp): 
		if assignment is complete: 
			return assignment 
		var = select_unassigned(csp.variables, assignment, csp) 
		for each value in domain_values(var, assignement, csp): 
			if value is consistent with assignemtn given csp.constraints: 
				add var == value to assignment 
				result = recursive_backtrack(assignment, csp) 
			if result != false return result
			remove var == value from assignment 
		return False
	```

- We can improve the general idea of backtracking search by ordiering
  intelligently (ordering) and ruling out assignment quickly (filtering).
- Filtering
	- Forward Checking
		- Keep track of domains fro unassigned variiables and cross off bad
		  options
		- Remove values that violate a constraint in another variable when
		  adding to the existing assignment.
		- When an unassigned variable's domain is empty we backtrack.
	- Edge consistency
		- An edge X -> Y is conssitent if for every x in the tail there is some
		  Y in the head that can ve assigned without violating a constraint.
			- We make an edge consistent by removing potential assignment
			  values from the tail.
		- Forward checking is enforcing consistency of an incoming arc.
		- Visit all edges until they are all consistent
			- If there is an arc that cannot be made to be consistent (ie
			  attempting to make the arc consistent results in an empty domain)
			  backtrack.
		- Edge consistency does not detect all failures.
	- Edge consistency is more powerful than forward checking, but it's much
	  more expensive.
- Ordering
	- Choosing the next variable: Choose the variable with the fewest legal
	  values left in its domain (most contained variable).
	- Called fail fast ordering.
	- Choosing the variable's value: Choose the value that rules out the fewest
	  other values (least constraining value).
- K-Consistency
	- 1-Consistency: Each single node's domain has n values which meets the
	  node's unary conditions.
	- 2-Consistency (Edge consistency): For each pair of nodes, any consistent
	  assignment to one can be extended to the other.
	- K-Consistency: For each k nodes, any consistent assignment to k-1 can be
	  extended to the kth node.
		- Much more expensive for larger k.
		- Strong K-consistency - essentially k-1, k-2, ..., 1 consistency.
- Structure
	- Taking advantage of the constraint graph structure to efficiently solve a
	  problem.
	- Connected components
		- Check the constraint graph for discrete components. Then you can
		  solve the problems independently.
	- Tree Structured CSP
		- If the constrant graph has no loops, the CSP can be solved in  O(n
		  d62) time.
		- Algorithm for tree structured CSPs
			1. Choose a root variable and order the variables so that parents
			   preceed children.
			2. Remove backwards: For i = n...2 apply
			   makeConsistent(Parent(X_i), X_i)
			3. Assign forward: for i = 1...n, assign X_i consistently with
			   Parent(X_i). Note that there will always be a consistent
			   solution in this phase. The reasoning for this is intuitive
			   based on how we enforce consistency in the backwards pass.
	- Nearly Tree Structured CSPs
		- Cutset Conditioning - Instantiate a set of variables such that the
		  remaining constraint graph is a tree.
		- if the cutset is of size c, we may need to instantiate all possible
		  versions of the cutset. So the algorithmis exponential in c and
		  linear in the rest of the tree.
		- Finding the cutset is NP hard.
	- Tree Decompopsition
		- Create a tree structured graph of "mega variables" (small groups of
		  connected variables).
		- Each mega variable encodes part of the original CSP.
		- Subproblems overlap to ensure consistent solutions.
- Iterative Improvement Algorithms
	- Local search method - typically works with complete states (all variables
	  assigned)
	- To apply to CSP
		- Take an assignment with unsatisfied constraints (not a goal state).
		- Operators reassign values of variables.
	- Algorithm:
		- Repeat until solved (while not solved):
			1. Variable selection - randomly select any conclicted variable.
			2. Value selection - choose a value that violates the fewest
			   constraints. (min conflicts heuristic)
	- Given random initial state we can solve the n-queens problem in almost
	  constant time with high probability.
	- The same appears to  be true for any randomly generated CSP except in
	  narrow range of the ratio.
	- R =  number of constraints / number of variables
	- For most R we can solve CSP almost instantaneously. However there is a
	  small range of R where CSP become hard to solve iteratively - this is
	  where a lot of natural problems are :(.

### Local Search 
- Local search improves a single option until you can't make it better. There
  is no concept of a fringe / keeping track of the search tree.
- The successor function of local search takes a complete assignment and
  modifies it as opposed to extending a plan.
- Generally, local search is much faster and more memory efficient than tree
  search.

### Adversarial Search 
- Algorithm that calculates a strategy which reccomends a move from each state.
- Deterministic Games
	- States - S (initial state is s_0)
	- Players P={1...N} usually take terns
	- Actions A
	- Transition Function - SxA -> S how the state evolves in response to
	  actions
	- Terminal Test - S -> (t,f), a goal state / end state
	- Terminal Utilities - SxP -> R, for an end state, how much this is worth
	  to each player
- Zero Sum Games - Agents have opposite utilities (values on outcomes).
	- This lets us think of a single value that some players maximize and
	  others minimize.
	- Much simpler than a general game.
- Value of a state: the besst achievable outcome from that state.
	- In the terminal state, this value is known.
	- For non-terminal states, V(s) = max_{s' \in children(s))} V(s'). That is,
	  the value of a state is the maximum value of the values of its children.
- Minimax Values: The best value we can achieve assuming an optimal adversary.
	- In minimax we consider the search tree as layers - one layer where an
	  agent maximizes the end value and another layer where the adversary tries
	  to minimize the same value.
	- For state under the agent's control, the value of the state is the
	  maximum of the values of its children.
	- For a state under the adversary's control, the value of the state is the
	  minimum of the values of its children. 
	 
	  ```Python 
	  def max_values(state):
		if state is terminal: 
			return terminal(state) # value of terminal state 
		v = -inf 
		for each successor of state: 
			v = max(v, min_values(successor))
			return v

		def min_value(state) 
			if state is terminal: 
				return terminal(state) # value of terminal state 
				v = inf 
				for each successor of state: 
				v = min(v, max_values(successor))
			return v

		def value(state): if state is terminal: 
			return terminal(state) # value of terminal state 
				if nextAgent is max: 
					return max_values(state) 
				elif nextAvent is min: 
					return min_values(state)
		```

	- This minimaz algorithm is essentially a DFS. Therefore it is:
		- Time: O(b^m)
		- Space: O(bm)
	- We can alleviate the resource problems by using depth limited search
	  instead of DFS and replace the terminal utilities with some evaluation
	  function for non-terminal positions. This estimates a value of a non-
	  tterminal state.
		- This doesn't garuntee optimal play, but going deeper in the tree
		  (searching more plies) makes a big difference.
		- Iterative deepening can be used to give the best result possible
		  given a time limit.
- Evaluation function: a function that inputs a non-terminal state and returns
  an approximation of the minimax value of the state.
	- Typically we use a linear combination of important features, but we can
	  have emore complicated (nonlinear) evaluation functions.
- Alpha Beta Pruning
	- Minimax allows us to prune the search tree a bit.
	- Consider the a MAX root node
		- We're computing the min value of some node n by looping over n's
		  children.
		- As we loop over n's children, n's estimate of the children's min can
		  only drop (or remain the same).
		- Let a be the best value that MAX can get at any choice point along
		  the current path from the root.
		- If n becomes less than a, MAX will avoid it so we can stop
		  considering n's other children (we already know n won't be
		  considered).

		```
		                 root                (MAX) 
		          /       |         \ 
		        x1        x2         x3      (MIN) 
		      /   \      /  \       /  \ 
		     x11 x12  x21  x22   x31   x32   (MAX)
		   ...        ...       ...
		```

			- the minimax value of root will be a min of x1, x2, and x3. If we
			  calculate MIN(x1) == a, then MAX(root) >= a. Then when we're
			  calculating x2 and x3, if any child has a value < a, we don't
			  need to consider that entire branch since we know the lower bound
			  on the root.
		- The inverse (MIN root) is symetric. 
		
		```Python
			# alpha : MAX's best option on the path to the root
			# beta : MIN's best option on the path to the root

			def max_value(state, alpha, beta): 
				v = -inf 
				for each successor of state: 
				v = max(v, value(successor, alpha, beta)) 
				if v >= Beta: 
					return v 
				alpha = max(alpha, v) 
			return v

			def min_value(state, alpha, beta): 
			v = inf 
			for each successor of state: 
				v = min(v, value(successor, alpha, beta)) 
				if v <= alpha: 
					return v 
				beta = min(beta, v) 
			return v

			def value(state): 
				if state is terminal: 
					return terminal(state) # value of terminal state 
				if nextAgent is max: 
					return max_values(state, alpha, beta)
				elif nextAvent is min: 
					return min_values(state, alpha, beta)
		```

	- You can't apply alpha beta pruning for action selection - the value at
	  the children has to be correct for optimal action selection.
- Minimax assumes a perfect player - however, our agents are not always working
  against perfect adversaries. Sometimes our "adversaries" aren't adversarila
  at all but are operating on chance.
- Expectimax Search
	- Computing the average score under optimal play. 
	
	```Python 
	  def max_value(state): 
	  	v = -inf 
	  	for each successor of state: 
	  		v = max(value(successor)) 
	  	return v

		def exp_value(state): 
			v = 0 
			for each successor of state: 
				p = probability(successor) 
				v += p * value(successor) return v
			return v
		
		def value(state): 
			if state is terminal: 
				return terminal(state) 
			if nextAgent is max: 
				return max_value(state) 
			elif nextAgent is exp: 
				return exp_value(state)
		```

	- This assumes a tree of the form

	```
	                  root                (MAX) 
	          /        |         \ 
	         x1        x2         x3 (EXPECTED)
	        /  \      /  \       /  \ 
	      x11 x12  x21  x22   x31   x32   (MAX)
	   ...        ...       ...
	```

	- The general expectimax tree cannot be pruned since we can't generalize
	  any bounds.
	- Expectimax, like minimax, is a very expensive process so we use
	  evaluation functions to estimate the expected value of a non-terminal
	  node. This is much more difficult to do in expectimax than in minimax.
	- To make expectimax work we need to estimate some probability
	  distribution.
- Some games have mixtures of minimax and expectimax. That is that the search
  tree has mixtures of min, max, and expected value layers.

```
                 root                (MAX) 
          /        |         \ 
        x1        x2         x3 (EXPECTED)
       /  \      /  \       /  \ 
     x11 x12  x21  x22   x31   x32   (MIN)
       ...        ...       ...
```

- An example of this is Backgammon. In backgammo, a player makes a move,
  then a di is rolled, and based on that roll, an adversary makes a move.
- If a node has multiple players and / or is not a zero sum game, we can use a
  search tree with different terminal utilities for each player.
- Maximum expected utility - a rational agent should choose the action that
  maximizes its expected utility given its knowledge.
- Utilities
	- Utilities are functions from outcomes (states fo the world) to real
	  numbers that describe an agent's preference.
	- Utilities can be defined in many ways so long as they summarize th
	  eagent's goal.
	- Any "rational" preferences can be summarized as a utility function.
	- We generally hard code utilities and let behaviors emerge.
- Preferences
	- An agent must have preferences among:
		- Prizes: A, B, etc
		- Lotteries: situations with uncertain prizes - L = {(p, A), ((1-p),
		  B)}
	- Preference A > B
	- Indifference A ~ B
	- An expected value node is essentially a lottery, and an expectimax search
	  is a large recursive lottery.
	- The prizes are essentially representative of the utilities.
- Rationality
	- We want to put some constraints on preferences before we call them
	  rational.
	- Transitivity: (A > B) and (B > C), then A > C
	- Orderability: A > B or B > A or A ~ B
	- Continuity: A > B > C then there exists a lottery [p, A: 1-p, C] ~ B
	- Substantiability: A ~ B then there exists a lottery [p, A; 1-p; C] ~ [p,
	  B; 1-p, C]
	- Monotonicity: A ~ B then p >= q then [p, A; 1-p, B] >= [q, A; 1-q, B]
	- If preferences satisfy these axioms we call them rational.
	- Given any preferences satisfying these constraints there exists a real
	  valued function U such that values assigned by U preserve preferences of
	  both prizes and lotteries.

### Non Deterministic Search - Markov Dicision Properties
- Non-deterministic search is a search problem when the outcome of our actions
are uncertain. 
- Grid World - maze like world. The agent lives in a grid and walls blck the
agent's path. Actions do not always go as planned. 
	- 80% N -> N 
	- 10% N -> W
	- 10% N -> E 
	- If there is a well inth edirection the agent would have taken the agent 
	stays put. 
	- The agent receives a reward each time step. There's a small "living" 
	reward each step and a big reward at the end of the game. Note that we use 
	"reward" to mean some feedback. Reward can be negative (bad). 
- We solve these kinds of problems usinng a Markov Decision Process (MDP). 
- A Markov Decision Process is defined by:
	- A set of states: $s \in S$
	- A set of actions: $a \in A$
	- A transition function: $T(s, a, s')$
		- This defines the probability that action $a$ from state $s$ leads to 
		state $s'$. 
		- Alo called the model or dynamics. 
	- A reward function: $R(s)$ or $R(s')$
	- A start state: $s_0$
	- An (optional) terminal state. 
		- MDPs can be infinite
- Markov principle - the future only depends on the present, not the past. 
- Unlike in a regular search problem where our functions return a plan or path
to reach the goal, in an MDP we return an optimial "policy": $\pi* \ : S \to 
A$.
	- A policy $\pi$ gives an action for each state. 
	- An optimal policy is a policy that maximizes expected utiltiy if followed. 
	- An explicit policy defines a reflex agent. 
	- Expectimax is similar but it only computed an action for the current 
	state. It doesn't compute an explicit policy. 
- When constructing MDPs we have to consider how to optimize our reward. Ie
should the agent prefer to obtain a reward sooner or later? 
	- Typically we want the agent to prefer rewards sooner - to do this we add 
	a factor $\gamma$ into our functions which actus to reduce the reward 
	value for later rewards. This is called discounting. 
- These games could also last forever
	- We could set finite horizons (depth limited search)
	- Discounting - the exponenetial decay reduces rewards as the limit 
	increases. 
	- Absorbing state - garuntee that every state will eventually result in a
	terminal sttate. 
- A queue state (s,a) is an action we've comitted to from some state s, but 
that hasn't been caried out yet. 
- Optimal Quantities
	- Utility of a state: $V* (s)$ is the expected utility starting in s and 
	acting optimally. Expectimax utility. 
	- Utility of a queue state: $Q* (s,a)$ is the expected utility starting out
	having taken action a from state s and then acting optimally. 
	- Value of a stae is the expected future utility from a state
	- Optimal policy: $\pi * (s)$ the optimal action from state s
- We define the optimal utility of a state as:
$$ V* (s) = max_{a} {Q* (s,a)}$$
$$ Q* (s,a) = \sum_{s'} { T(s, a, s') * [ R(s, a, s') + \gamma * V* (s') ] }$$
- This is called the Bellman equation. 
	- Observe that $\sum_{s'}{T(s, a, s') * [ R(s, a, s') + \gamma * V* (s') 
	]}$ is effectively finding the expected value of the action $a$. 
- Time limitted values 
	- $V_k$ is the optimal value of s if the game ends in k more time steps. 
	- We call this time limited but it's really depth limited. 
- Value iteration: Assuming we have a small number of possible states:
	- Set $V_{0} = 0$ for all states. Ie $V_{0}[s] = 0 \ \forall s \in S$
	- Then $V_{k+1} = \max_{a} { \sum_{s'}{T(s,a,s') [R(s,a,s') + \gamma V_{k} 
	(s')]}}$. 
	- Repeat until convergence. 
		- This process will converge to unique optimal values. 
- Example: Consider the following racecar. If the car drives slowly he cannot
overheat. However he get's more reward (has a higher chance of winning the
race) if he drives fast. From any state he can either accelerate or decelerate.

        | Cool          |      Hot      |  Overheated  
-------------------------------------------------------
        |   P(s')  | R  |   P(s')  | R  |   P(s')  | R 
--------------------------------------------------------        
Go Slow | 1.0 Slow | +1 | 0.5 Slow | +1 | 1.0 OH   | 0 
        |          |    | 0.5 Fast | +1 |          | 0 
Go Fast | 0.5 Fast | +2 | 1.0 OH   | -10| 1.0 OH   | 0 
          0.5 Slow | +2 |          |    |          | 0 

 We can use this table and the Bellman equation to calculate the optimal policy
 for this situation. 

      | Cool               | Hot                | Overheated
--------------------------------------------------------------------
      | Value | Opt Action | Value | Opt Action | Value | Opt Action 
 -------------------------------------------------------------------
 V_2  |  3.5  |  Go Fast   |  2    | Go slow    | 0     | NA
 V_1  |   2   |  Go Fast   |  1    | Go slow    | 0     | NA
 V_0  |   0   |  NA        |  0    | NA         | 0     | NA

We can calculate 
$$V_1 (cold) = max_{a} {\sum_{s'}{T(s,a,s')[R(s,a,s') + V_0{s'}]}}$$
$$V_1 (cold) = max(T_(cool, slow, cool) [R(cool, slow, cool) + V_0 (cool)] + T_
(cool, slow, hot) [R(cool, slow, hot) + V_0 (hot)], T_(cool, fast, cool) [R
(cool, slow, cool) + V_0 (cool)], T_(cool, fast, hot) [R(cool, fast, hot) + V_0
(hot)])$$

This example doesn't include the $\gamma$ (discount) parameter. 



