# Artificial Intelligence

- [Artificial Intelligence](#artificial-intelligence)
- [Search Problem](#search-problem)
	- [Agents, States, and the Tree Search Algorithm](#agents-states-and-the-tree-search-algorithm)
	- [Uninformed Search Algorithms](#uninformed-search-algorithms)
		- [Depth First Search](#depth-first-search)
		- [Breadth First Search](#breadth-first-search)
		- [Uniform Cost Search](#uniform-cost-search)
	- [Informed Search and Heuristic Functions](#informed-search-and-heuristic-functions)
		- [A* search](#a-search)
		- [Properties of Heuristic Functions](#properties-of-heuristic-functions)
	- [Bidirectional Search](#bidirectional-search)
	- [Tridirectional Search](#tridirectional-search)
	- [Constraint Satisfaction Problem (CSP)](#constraint-satisfaction-problem-csp)
		- [Backtracking Search](#backtracking-search)
		- [Iterative Improvement Algorithms](#iterative-improvement-algorithms)
		- [Local Search](#local-search)
	- [Adversarial Search](#adversarial-search)
		- [Minimax](#minimax)
		- [Alpha Beta Pruning](#alpha-beta-pruning)
		- [Expectimax](#expectimax)
		- [N-player games](#n-player-games)
		- [Rational Agents](#rational-agents)
	- [Non Deterministic Search - Markov Decision Properties](#non-deterministic-search---markov-decision-properties)
		- [Value Iteration](#value-iteration)
		- [Reinforcement Learning](#reinforcement-learning)
- [Probabilistic Reasoning](#probabilistic-reasoning)
		- [Probability Basics](#probability-basics)
		- [Markov Models](#markov-models)
		- [Baysean Networks](#baysean-networks)
- [Machine Learning](#machine-learning)
	- [Naive Bayes](#naive-bayes)
	- [Perceptrons](#perceptrons)
	- [Kernels and Clustering](#kernels-and-clustering)

# Search Problem

## Agents, States, and the Tree Search Algorithm
An agent in artificial
we are usually defining how some agent interacts with the world around it. Agents can
classified broadly into two categories.

A __reflex agent__ is an agent whose actions are chosen by a specific set of inputs.
A reflex agent is a direct mapping from a set of inputs / memory to a specific output
with no consideration for possible future outcomes. The rationality and robustness of
a reflex agent's actions depends on the complexity of the mapping - a complex enough
function can lead to a very rational agent.

A __planning agent__ is an agent whose actions are chosen by creating a model of the world
and inferring how it would respond to specific inputs. These agents have some model of the
world and a goal. A __complete planning agent__ will always find a way to achieve its
goal and a __optimal planning agent__ will find the best solution given a cost function.

Planning agents can be further subclassified into generic planning agents and replanning
agents. Where a planning agent comes up with a single complete plan and completes it, a
__replanning agent__ splits its goals into smaller subgoals and comes up with
plans as it completes the subgoals.

Both reflex and planning agents "know" the state of the world. In general the __state space__
is an encoding of "the world" at a moment in time. Agents use this representation to make
decisions and therefore encoding a complex state space is important to properly solving
problems in AI. A __successor function__ models how agents can transition between states.
Typically a successor function will define how the state changes given an agent's action
or some other event in the state. A __goal state__ is a state where the agent has
achieved it's goal.

The __search problem__ is the process of getting to a goal state from a given start state.
This problem can be reformulated as a graph search problem. We define the __state space
graph__ as a graph where the vertices are individual states and the edges represent
transitions between states. Edge $(x, y)$ means that there exists some action $a$ such that given
a successor function $f : States,\ Actions) \to \ States$, $f(x, a) = y$.  A __search tree__ is a
slightly different representation of the state space graph where
the root is the current state and the children are the successor states. Both state space graphs
and search trees are too big to feasibly store, but they are useful representations in finding
a goal state.

To solve the search problem we use a generic __tree search algorithm__ which can be solved in a
few different ways. The main idea is to initialize the search tree with the initial state, and
check successor states using some strategy which minimizes the number of states we have to check
before we reach the goal.

```
def tree_search(problem, strategy):
	initiate the search tree using the initial state of problem
	do:
		get next node based on strategy
		if there are no candidates for expansion:
			return False
		if the node contains a goal state:
			return the corresponding solution (path from start to node)
		else: expand the node and add the resulting nodes to the search tree
```

In these search algorithms we are literally searching the search
tree itself. Thus we want to choose an optimal strategy to avoid the overhead of
building the large search tree. We say that we __expand__ a node when we "get" it based
on the strategy. We also keep a __fringe__ of nodes which we have collected from previously
expanded nodes, but we haven't expanded yet. The strategy involves intelligently picking
the next node to expand from the fringe.

Graph Search is a slight modification of tree search where every time we expand a state,
we keep track of the set of states already expanded.
We expand the search tree node-by-node following tree search, but before expanding a
node we check to make sure its state is new. If not new, skip it. This set of expanded
nodes is called a __closed set__.

## Uninformed Search Algorithms

### Depth First Search
Depth First Search (DFS) is a classic graph search algorithm which can be applied towards
the search problem. The strategy is to expand the "deepest" node first. Practically, we
treat the fringe as a stack and push successors onto the fringe in a
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

------------------------------------------------------------------

Property          |
----------        | ----------
                  |
Fringe            | Stack
                  |
Complete          | Yes if there are no cycles
                  |
Optimal           | No. Will always find the left-most path
                  |
Time Complexity   | $O(b^m)$ where $b$ is the branching factor and $m$ is the depth
                  |
Space Complexity  | $O(b * m)$ where $b$ is the branching factor and $m$ is the depth of the tree.

-----------------------------------------------------------------


We can see that DFS is best if the shallowest goal state is on the "leftmost"
side of the search tree. However, it is not optimal if the goal states on the
left side of the search tree are deep in the tree or if the goal states are
on the right side of the search tree. In these cases we can use __iterative
deepening__ to cut off the DFS search after a specific depth limit and iteratively
keep increasing the depth limit until a solution has been found. This takes advantage
of the relatively low space complexity of DFS.

### Breadth First Search
Breadth First Search (BFS) is another classic graph search algorithm. The strategy is
to expand the shallowest node (path) first. We treat the stack as a queue and
explore the fringe a layer at a time.

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

------------------------------------------------------------------

Property          |
----------        | ----------
                  |
Fringe            | Queue
                  |
Complete          | Yes
                  |
Optimal           | Yes. Only if the transition cost between all states is 1 (unweighted search tree).
                  |
Time Complexity   | $O(b^s)$ where $b$ is the branching factor and $s$ is the level of the shallowest solution.
                  |
Space Complexity  | $O(b^s)$ where $b$ is the branching factor and $s$ is the depth of the shallowest level.

-----------------------------------------------------------------

DFS and BFS are very similar algorithms but have drastically different use cases.
We want to use DFS if the depth is small compared to the branching factor (ie if each
state doesn't have too many branches) and BFS if the solutions are relatively shallow.

### Uniform Cost Search
Uniform Cost Search (UCS) is similar to Djikstra's algorithm except it terminates after
the search algorithm has found a goal state. We push nodes onto the fringe in order
of cost (with the lowest cost prioritized first). We treat the fringe as a priority queue
with lower costs assigned higher priority.

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

------------------------------------------------------------------

Property          |
----------        | ----------
                  |
Fringe            | Priority Queue (lower cost $\to$ higher priority)
                  |
Complete          | Yes
                  |
Optimal           | Yes
                  |
Effective Depth   | $c^{* } \times \epsilon$ where the solution costs $c^{* }$ and arcs cost at least $\epsilon$
                  |
Time Complexity   | $O(b^{c^* \times \epsilon})$ where $b$ is the branching factor and $c^* $ is the solution cost.
                  |
Space Complexity  | $O(b^{c^* \times \epsilon})$ where $b$ is the branching factor and $c^* $ is the solution cost.

-----------------------------------------------------------------

## Informed Search and Heuristic Functions
The search algorithms we've talked about above have been uninformed search.
That means that they haven't taken into account the direction (in the search
tree) where the solution may be. __Informed search__ algorithms use __heuristic
functions__ to infer the best successors to avoid expanding too much of the
search tree.

A __heuristic function__ is a function that estimates how close a state is to a goal. Each
heuristic is designed for a specific search problem.

__Greedy Search__ is the simplest informed search algorithm. The idea behind
greedy search is to expand the node that you think is closest to a goal state.

### A* search
A* search is a combination of UCS and Greedy Search. We push states onto the
fringe with $priority(state) \ = \ path\_cost(state) \ + \ heuristic(state)$.
This allows us to explore states with the lowest heuristic and path cost - ie
the state closest to the goal and cheapest to reach.

```Python
def aStarSearch(problem, heuristic=nullHeuristic):
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
                    p_queue.push(suc, suc[2] + heuristic(suc[0], problem))
        curr = p_queue.pop()
    return curr[1]
```

------------------------------------------------------------------

Property          |
----------        | ----------
                  |
Fringe            | Priority Queue
                  |
Complete          | Yes
                  |
Optimal           | Yes, with an __admissable__ heuristic.
                  |
Time Complexity   | Exponential
                  |
Space Complexity  | Exponential

-----------------------------------------------------------------

A* search is used in everything from video games and robot movement to
natural language.

### Properties of Heuristic Functions
A* search is built on using the right heuristic function. Picking the wrong
heuristic function can break optimally by trapping good plans on the fringe.
We define a heuristic function $h$ as __admissible__ if $0 \leq h(n) \leq h^{ * } (n)$
where $h^{ * } (n)$ is the true cost to the nearest goal.

Given two heuristics $h_a$ and $h_c$ and $\forall n \ h_a(n) \geq h_c(n)$,
$h_a$ dominates $h_c$. The maximum of all admissible heuristic
is the max heuristic and the minimum of all admissible heuristics returns
0.

We define a heuristic function $h$ is __consistent__ if for every state $n$ and each
successor $p$ of $n$ $h(n) \leq c(n, p) + h(p)$ and $h(G) = 0$. A consistent heuristic
is one such that the heuristic of a state $n$ is less than or equal to the cost to get
to some successor $p$ from $n$ and the estimated cost to reach the goal from $p$.
Consistency is said to be stronger than admissibility - that is, a consistent heuristic
function is also admissible.

## Bidirectional Search

Bidirectional search is a variant of UCS and A* search that allows us to cut down on the number of
nodes expanded during search. In bidirectional search, we run two searches simultaneously - one
from the start node and the other from the goal node. When those searches intersect, we know that
our queue must contain a shortest path from the start to the goal so we can stop expanding nodes
and just evaluate nodes on the intersection of the two searches.

## Tridirectional Search

Tridirectional search is a type of search we can use

## Constraint Satisfaction Problem (CSP)
A Constraint Satisfaction Problem (CSP) is a subset of search problems where the
goal itself is important, but the path is irrelevant. In a CSP, a state is defined
by variable $x_i$ which can take on a value from domain $D$. The goal test is a
set of constraints specifying allowable combinations of values from subsets of variables.

CSPs are a simple example of a formal representation language. This allows for useful
general purpose algorithms with more power and efficiency than standard search algorithms.
We can solve a CSP by applying a specialized algorithm to a __constraint graph__. A
constraint graph is a graph where the vertices are variables and the
edges show constraints. Constraint graphs' vertices can have finite, infinite, continuous,
or discrete domains. Constraint graphs' edges can be unary, binary, or higher order.

We can solve a CSP inefficiently using tree search. The initial state is where no vertices
have been assigned and current states are states where values are assigned. Non-goal states are
states with a partial assignment of values. The successor function assigns k value to an
unassigned variable. The goal test tests if the current assignment is complete and
satisfies all constraints.

Using this approach we could apply BFS, DFS, UCS, or A* search to a CSP.
However, these are very inefficient on their own. Consider the map coloring
case - the successor function of a standard search problem doesn't consider
the restrictions of a CSP. So using one of these algorithms would (generally)
require traversing the entire search tree.

### Backtracking Search
Backtracking search is a specialized CSP algorithm where we assign one variable
at a time and check the constraints as we go.

```Python
def backtracking_search(csp):
	return recursive_backtrack({}, csp)

def recursive_backtrack(assignment, csp):
	if assignment is complete:
		return assignment
	var = select_unassigned(csp.variables, assignment, csp)
	for each value in domain_values(var, assignment, csp):
		if value.isConsistent(csp.constraints(assignment)):
			assignment.add(var == value)
			result = recursive_backtrack(assignment, csp)
		if result != false:
			return result
		assignment.remove(var == value)
	return False
```

We can improve the general idea of backtracking search by ordering
intelligently (ordering) and ruling out assignment quickly (filtering).

__Filtering__

Filtering is a process of ruling out bad assignments quickly.

- Forward Checking \
Using forward checking, we keep track of domains for unassigned variables and cross off bad
options. We then remove values that violate a constraint in another variable when
adding to the existing assignment. When an unassigned variable's domain is empty we backtrack.
- Edge consistency \
An edge X -> Y is consistent if for every X in the tail there is some
Y in the head that can be assigned without violating a constraint. We make an edge
consistent by removing potential assignment values from the tail. Forward checking is
essentially enforcing consistency of an incoming arc. To enforce edge consistency
we visit all edges until they are all consistent and use forward checking to make the
edge consistent. If there is an arc that cannot be made to be consistent (ie
attempting to make the arc consistent results in an empty domain)
backtrack.\
Edge consistency is more powerful than forward checking, but it's much
more expensive.
- K-Consistency
  - 1-Consistency: Each single node's domain has n values which meets the
    node's unary conditions.
  - 2-Consistency (Edge consistency): For each pair of nodes, any consistent
    assignment to one can be extended to the other.
  - K-Consistency: For each k nodes, any consistent assignment to k-1 can be
    extended to the kth node.
    - Much more expensive for larger k.
    - Strong K-consistency - essentially k-1, k-2, ..., 1 consistency.

__Ordering__

Ordering is the processes of choosing the next variable in a backtracking
search algorithm. The general idea of ordering is to choose the variable with
the fewest legal values left in its domain (ie the most constrained variable).
This is also called __fail fast ordering__.

__Structure__

We can also use the constraint graph's structure to efficiently solve a CSP.

- Connected components
	- Check the constraint graph for discrete components. Then you can
	  solve the problems independently.
- Tree Structured CSP
	- If the constraint graph has no loops, the CSP can be solved in $O(n
	  d^2)$ time.
	- Algorithm for tree structured CSPs
		1. Choose a root variable and order the variables so that parents
		   precede children.
		2. Remove backwards: For $i = n...2$ apply
		   makeConsistent(Parent(X_i), X_i)
		3. Assign forward: for $i = 1...n$, assign $X_i$ consistently with
		   $parent(X_i)$. Note that there will always be a consistent
		   solution in this phase. The reasoning for this is intuitive
		   based on how we enforce consistency in the backwards pass.
- Nearly Tree Structured CSPs
	- Cutset Conditioning - Instantiate a set of variables such that the
	  remaining constraint graph is a tree.
	- if the cutset is of size c, we may need to instantiate all possible
	  versions of the cutset. So the algorithms exponential in c and
	  linear in the rest of the tree.
	- Finding the cutset is NP hard.
- Tree Decomposition
	- Create a tree structured graph of "mega variables" (small groups of
	  connected variables).
	- Each mega variable encodes part of the original CSP.
	- Subproblems overlap to ensure consistent solutions.

### Iterative Improvement Algorithms
A local search method is an algorithm which takes a completely assigned state
and iteratively improves it. To apply iterative improvement to a CSP we take
an assignment with unsatisfied constraints (not a goal state) and iteratively
reassign values of variables.

- Algorithm:
	- Repeat until solved (while not solved):
		1. Variable selection - randomly select any conflicted variable.
		2. Value selection - choose a value that violates the fewest
		   constraints. (min conflicts heuristic)

Given random initial state we can solve the n-queens problem in almost
constant time with high probability. The same appears to be true for any
randomly generated CSP except in the narrow range of the ratio: $R = \frac{number\ of\
constraints}{number\ of\ variables}$.
For most R we can solve CSP almost instantaneously. However there is a
small range of R where CSP become hard to solve iteratively - this is
where a lot of natural problems are.

### Local Search
Local search improves a single option until you can't make it better. There
is no concept of a fringe / keeping track of the search tree.
The successor function of local search takes a complete assignment and
modifies it as opposed to extending a plan.
Generally, local search is much faster and more memory efficient than tree
search.

## Adversarial Search
Adversarial search is a class of search algorithms that calculates a strategy for
a scenario with adversarial agents. We think of these types of problems as deterministic
games. A deterministic game is defined as follows:

- States - S (initial state is s_0)
- Players P={1...N} usually take terns
- Actions A
- Transition Function - SxA -> S how the state evolves in response to
  actions
- Terminal Test - S -> (t,f), a goal state / end state
- Terminal Utilities - SxP -> R, for an end state, how much this is worth
  to each player

In a __zero sum game__ the agents have opposite utilities (values on outcomes).
This lets us think of a single value that some players maximize and
others minimize. These games are much simpler than a general game.

The __value (utility) of a state__ is the best achievable outcome from that state.
In the terminal state, this value is known. For non-terminal states,
$V(s) = max_{s' \in children(s))} V(s')$. That is, the value of a state is the
maximum value of the values of its children.

In general, utilities are functions from outcomes to real numbers that describe
an agent's preference. Utilities can be defined in many ways so long as they
summarize the agent's goal. Any "rational" preferences can be summarized as
a utility function. We generally hard code utilities and let behaviors emerge.

### Minimax
The __minimax value__ is the best value we can achieve assuming an optimal adversary.
In minimax we consider the search tree as layers - one layer where an
agent maximizes the end value and another layer where the adversary tries
to minimize the same value. For state under the agent's control, the value of
the state is the maximum of the values of its children. For a state under the
adversary's control, the value of the state is the minimum of the values of its children.

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

def value(state):
	if state is terminal:
		return terminal(state) # value of terminal state
	if nextAgent is max:
		return max_values(state)
	elif nextEvent is min:
		return min_values(state)
```

This minimax algorithm is essentially a DFS. Therefore it is:
- Time: O(b^m)
- Space: O(bm)

We can alleviate the resource problems by using depth limited search
instead of DFS and replace the terminal utilities with some evaluation
function for non-terminal positions. This estimates a value of a non-
terminal state. This doesn't guarantee optimal play, but going deeper in the tree
(searching more plies) makes a big difference.

__Iterative deepening__ is the process of iteratively increasing the depth of the search
tree and using the deepest result you can calculate within a time limit. For a
branching factor of $k$ we find that the number of nodes in a tree of depth $d$ is

$$n = \frac{k^{d+1} - 1}{2}$$

An __evaluation function__ is a function that inputs a non-terminal state and returns
an approximation of the minimax value of the state. Typically we use a linear
combination of important features, but we can have more complicated (non-linear)
evaluation functions. When developing an evaluation function, we often want to choose
something simple enough that it can be efficiently calculated throughout the
tree.

### Alpha Beta Pruning
The structure of a minimax search tree allows us to prune the search tree a bit.
If we consider the a MAX root node we need to find the maximum of all of the
successor nodes. So for each successor node $n$, we need to find the minimum value
by iterating over n's children. As we loop over n's children, n's estimate of the children's min can
only drop (or remain the same). Then let $\alpha$ be the best value that
MAX can get at any choice point along the current path from the root. If n becomes
less than $\alpha$, MAX will avoid it so we can stop considering n's other
children (we already know n won't be considered).

```
                 root                (MAX)
          /       |         \
        x1        x2         x3      (MIN)
      /   \      /  \       /  \
     x11 x12  x21  x22   x31   x32   (MAX)
   ...        ...       ...
```

The minimax value of root will be a min of $x_1, x_2, x_3$. If we
calculate $MIN(x1) = \alpha$, then $MAX(root) >= \alpha$. Then when we're
calculating $x_2$ and $x_3$, if any child has a value less than $\alpha$, we don't
need to consider that entire branch since we know the lower bound
on the root.

The inverse (MIN root) is symmetric. $\beta$ represents the lowest value a MIN
node can be on the path to the root.

```Python
# alpha : MAX's best option on the path to the root
# beta : MIN's best option on the path to the root

def max_value(state, alpha, beta):
	v = -inf
	for each successor of state:
		v = max(v, value(successor, alpha, beta))
		alpha = max(alpha, v)
		if alpha >= beta:
			return beta
	return v

def min_value(state, alpha, beta):
	v = inf
	for each successor of state:
		v = min(v, value(successor, alpha, beta))
		beta = min(beta, v)
		if alpha >= beta:
			return alpha
	return v

def value(state, alpha=MIN_INT, beta=MAX_INT):
	if state is terminal:
		return terminal(state) # value of terminal state
	if nextAgent is max:
		return max_values(state, alpha, beta)
	elif nextAvent is min:
		return min_values(state, alpha, beta)
```

You can't apply alpha beta pruning for action selection - the value at
the children has to be correct for optimal action selection.

### Expectimax
Minimax assumes a perfect player - however, our agents are not always working
against perfect adversaries. Sometimes our "adversaries" aren't adversarial
at all but are operating on chance. __Expectimax Search__ computes the average
score under optimal play.

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

This assumes a tree of the form

```
                  root                (MAX)
          /        |         \
         x1        x2         x3 (EXPECTED)
        /  \      /  \       /  \
      x11 x12  x21  x22   x31   x32   (MAX)
   ...        ...       ...
```

The general expectimax tree cannot be pruned since we can't generalize
any bounds. Expectimax, like minimax, is a very expensive process so we use
evaluation functions to estimate the expected value of a non-terminal
node. This is much more difficult to do in expectimax than in minimax.
To make expectimax work we need to estimate some probability distribution.

Some games have mixtures of minimax and expectimax. That is that the search
tree has mixtures of min, max, and expected value layers. An example of this
is Backgammon. In backgammon, a player makes a move, then a di is rolled, and
based on that roll, an adversary makes a move.

```
                root                 (MAX)
         /        |         \
        x1        x2         x3 (EXPECTED)
       /  \      /  \       /  \
     x11 x12  x21  x22   x31   x32   (MIN)
       ...        ...       ...
```


If a node has multiple players and / or is not a zero sum game, we can use a
search tree with different terminal utilities for each player.

### N-player games
For n-player games, we need to treat minimaz a bit differently.

```
                root                 (MAX P1)
         /        |         \
        x1        x2         x3 	 (MAX P2)
       /  \      /  \       /  \
     x11 x12  x21  x22   x31   x32   (MAX P3)
	 |
	(0, 1, 1)
       ...        ...       ...
```

In an n-player game, the terminal states are n-tuples where each element is the
evaluation function from the perspective of that player. Then each layer attempts
to maximize that player's specific outcome.


### Rational Agents
An agent must have preferences among:

- Prizes: A, B, etc
- Lotteries: situations with uncertain prizes - L = {(p, A), ((1-p),
  B)}
- Preference A > B
- Indifference A ~ B
- An expected value node is essentially a lottery, and an expectimax search
is a large recursive lottery. The prizes are essentially representative of the utilities.

We want to put some constraints on preferences before we call them rational.

- Transitivity: (A > B) and (B > C), then A > C
- Orderability: A > B or B > A or A ~ B
- Continuity: A > B > C then there exists a lottery [p, A: 1-p, C] ~ B
- Substitutability: A ~ B then there exists a lottery [p, A; 1-p; C] ~ [p,
  B; 1-p, C]
- Monotonicity: A ~ B then p >= q then [p, A; 1-p, B] >= [q, A; 1-q, B]

If preferences satisfy these axioms we call them rational. Given any preferences satisfying these constraints there exists a real valued function U such that values assigned by U preserve preferences of both prizes and lotteries.

## Non Deterministic Search - Markov Decision Properties
__Non-deterministic search__ is a search problem when the outcome of our actions
are uncertain. An example of this search problem is Grid World. Grid World is a maze
like world. The agent lives in a grid and walls block the gent's path. Actions do not
always go as planned. The agent receives a reward each time step. There's a small "living"
reward each step and a big reward at the end of the game. Note that we use
"reward" to mean some feedback. Reward can be negative (bad).

We solve these kinds of problems using a __Markov Decision Process (MDP)__.
A Markov Decision Process is defined by:

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

The __Markov principle__ is that the future only depends on the present, not the past.
Unlike in a regular search problem where our functions return a plan or path
to reach the goal, in an MDP we return an optimal "policy", represented by the
function $\pi* \ : S \to A$. Generally, a policy $\pi$ gives an action for each state.
and an optimal policy is a policy that maximizes expected utility if followed.
An explicit policy defines a reflex agent. Expectimax follows a similar idea but it
only computed an action for the current state. It doesn't compute an explicit policy.

When constructing MDPs we have to consider how to optimize our reward. Ie
should the agent prefer to obtain a reward sooner or later? Typically we
want the agent to prefer rewards sooner - to do this we add a factor $\gamma$
into our functions which actions to reduce the reward value for later rewards.
This is called discounting.

These games could also last forever so we need to find a way to terminate the
game. We could set finite horizons (depth limited search), use discounting, or
create an __absorbing state__ - a guarantee that every state will eventually result
in a terminal state.

A __queue state__ $(s,a)$ is an action we've committed to from some state s, but
that hasn't been carried out yet. The __utility of a state__ $V* (s)$ is the
expected reward starting in s and acting optimally. This is the same as
expectimax utility. The __utility of a queue state__ $Q* (s,a)$ is the
expected reward starting out having taken action a from state s and
then acting optimally. The __optimal policy__ $\pi* (s)$ is the optimal
action from state s.

We define the optimal utility of a state as:
$$V* (s) = max_{a} {Q* (s,a)}$$
$$Q* (s,a) = \sum_{s'} { T(s, a, s') * [ R(s, a, s') + \gamma * V* (s') ] }$$

This is called the Bellman equation.

Observe that $\sum_{s'}{T(s, a, s') * [ R(s, a, s') + \gamma * V* (s')
]}$ is effectively finding the expected value of the action $a$.

$V_k$ is the optimal value of s if the game ends in k more time steps.
We call this time limited but it's really depth limited.

### Value Iteration
We can determine the value of a state using __value iteration__. Value
iteration is an expectation maximization algorithm used to calculate state
values.

1. Set $V_{0} = 0$ for all states. Ie $V_{0}[s] = 0 \ \forall s \in S$
2. Then $V_{k+1} = \max_{a} { \sum_{s'}{T(s,a,s') [R(s,a,s') + \gamma V_{k}
(s')]}}$.
3. Repeat until convergence.

__Example:__ Consider the following racecar. If the car drives slowly he cannot
overheat. However he gets more reward (has a higher chance of winning the
race) if he drives fast. From any state he can either accelerate or decelerate.

States  | Cool          |      Hot      |  Overheated
--------|---------------|---------------|--------------
Actions |   P(s')  | R  |   P(s')  | R  |   P(s')  | R
--------|----------|----|----------|----|----------|---
Go Slow | 1.0 Cool | +1 | 0.5 Cool | +1 | 1.0 OH   | 0
x       | 0.0 Hot  | +0 | 0.5 Hot  | +1 |          | 0
Go Fast | 0.5 Cool | +2 | 1.0 OH   | -10| 1.0 OH   | 0
x       | 0.5 Hot  | +2 | x        | x  | x         | 0

 We can use this table and the Bellman equation to calculate the optimal policy
 for this situation.

x     | Cool               | Hot                | Overheated
------|--------------------|--------------------|-------------------
x     | Value | Opt Action | Value | Opt Action | Value | Opt Action
------|-------|------------|-------|------------|-------|-----------
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

__Policy Evaluation__ is the process of calculating the values of states
given a fixed policy is pretty easy. $V^{\pi} (s)$ is the expected total
discounted reward starting in $s$ and following policy $\pi$.
$$V^{\pi} (s) = \sum_{s'}{T(s, \pi (s), s') [R(s, \pi (s), s') + \gamma V^
{\pi} (s')]}$$

This is effectively the original Bellman equation, but without maxing over
the different actions (since $\pi (s)$ is a fixed action). We compute the values in the same way.

1. Set $V_0 (s) = 0 \forall s \in S$.
2. $V_{k+1}^{\pi}(s) = \sum_{s'} T(s, \pi (s),s') [R(s, \pi (s), s') + V_
{k}^{\pi} (s')]$
- This computation is $A$ times more efficient than computing the optimal
policy (since we don't need to max over all actions).

Without the max, the system of equation just turns into a system of linear
equations - so we can also solve the system using any linear equation solving
method.

Given the optimal values we can figure out the policy by using a one step expecimax.

$$\pi* (s) = argmax_{a}(\sum_{s'}{(T(s, a, s') [R(s, a, s') + \gamma V* (s')])})$$

"Action selection from values is kinda a pain in the butt" - Dan Klein. However, knowing the optimal Q values make it trivial to find the optimal action.
$$\pi* (s) = argmax_{a} Q(s, a)$$

__Policy iteration__ is an iterative, EM algorithm for finding an optimal policy.

1. Policy Evaluation: calculate utilities for some fixed policy (not
optimal utilities) until convergence.
	- Let $\pi (s)$ be the policy calculated in step 2 (or the initial).
	- Do until values converge
	$$V_{k+1}^{\pi}(s) = \sum_{s'} T(s, \pi (s),s') [R(s, \pi (s), s')+
		V_{k}^{\pi} (s')]$$
2. Policy Improvement: update policy using one-step look ahead with r
resulting converged (but not optimal) utilities at future values.
	- Let V(s) be the values calculated in step 1.
	$$\pi (s) = argmax_{a}\sum_{s'}{T(s, a, s') [R(s, a, s') + \gamma V*
	(s')]}$$
3. Repeat steps 1 and 2 until policy converges.

The improvement step is not faster than value iteration. The evaluation
step is significantly faster and happens many times before an improvement
step. That's where we see speedup relative to value iteration.
Value iteration and policy iteration do the same thing. Both provide optimal
values and policies (if you do it right). In value iteration, every iteration
updates both the values and (implicitly) the policy. In policy iteration we do
several passes that updates the utilities and then update the policy

### Reinforcement Learning
__Reinforcement learning__ is an extremely powerful technique in machine learning
and artificial intelligence. Reinforcement learning uses the result of an agent's actions
in the environment to learn an optimal policy. When an agent performs an action, the
environment returns a reward adn a "percept" (aka a state). A reinforcement learning
algorithm wants to maximize this reward as the number of actions performed approaches
infinity - that is, we can use early actions to learn, but eventually we should be
able to perform the actions that lead to the highest reward. In reinforcement learning
we assume that the world is a Markov Decision
Process (MDP). We're still trying to learn a policy, state values, etc. But we
don't know the probability distribution. So we need to try a bunch of actions,
learn from our successes and failures, and eventually learn the distribution of
the MDP and thereby find the optimal policy.

Reinforcement learning can be __online__ or __offline__. Online learning is "learning by
doing" in the actual environment whereas offline learning occurs in a simulated
environment. Reinforcement learning can also be __model based__ or __model free__.
A model based learning approach learns an approximate model of the MDP based on
the actions taken. This model is used to generate a policy. A model free approach
calculates the expected value of each state directly using past experiences rather than
explicitly defining a model.

__Passive reinforcement learning__ is a reinforcement learning model where an
agent is acting in the real world and we have no control over its actions.
We are essentially watching a fixed policy and are trying to learn that policy.
Passive reinforcement learning has two main subcategories:
- __Direct Evaluation__ is a passive reinforcement learning process where we note
the result of the actions at each state and use the long term average to estimate
the policy. This works out in the end. However it can result in odd intermediate
results because we don't take into account state connection. Each
state is learned separately and therefore takes a long time to learn.
- Sample based policy evaluation - an approximation of the policy
evaluation algorithm using sample averages as opposed to known transition
functions / rewards.
	- Every time you're at $s$, take sample of outcomes $s'$ by doing the
	action and average.
	$$sample_1 = R(s, \pi(s), s_{1}') + V_k^{\pi}(s_1')$$
	$$sample_2 = R(s, \pi(s), s_{2}') + V_k^{\pi}(s_2')$$
	...
	$$sample_n = R(s, \pi(s), s_{n}') + V_k^{\pi}(s_n')$$
	take the average of n samples
	- The big problem is that we can't garuntee we'll get back to $s$.
- Temporal difference learning
	- Learn from every experience. Update V(s) each time we experience a
	transition. Likely outcomes $s'$ will contribute to updates more
	often.
	$$sample = R(s, \pi(s), s') + \gamma V^{\pi}(s')$$
	$$V^{\pi}(s) = (1 -\alpha) V^{\pi}(s) + \alpha * sample$$
	or
	$$V^{\pi}(s) =  V^{\pi}(s) + \alpha(sample - V^{\pi}(s))$$
	- $\alpha$ is called the learning rate.
	- The learning rate creates an "exponential moving average" which
	weights more recent samples more heavily.
	- This works for passive learning, but not active.
	- Active reinforcement learning - we need to learn the Q values as
	well as the state values.

- Full reinforcement learning - we want to learn optimal policies
	- We don't know transitions or rewards. However we can choose the actions.
	- This is an online process.
- Q Learning is a modification of the Bellman equations which allows us to
learn Q values. For a chance node we can learn the Q values by taking the
average of the optimal values for each state - where each value is defined as
the max of it's Q values.
$$Q_{k+1}(s,a) = \sum_{s'}{T(s,a,s') [R(s,a,s') + \gamma max_{a'}{Q(s',a)}]}$$
- Q Learning:
	- receive a sample s, a, s', r)
	- consider the old estimate Q(s,a)
	- calculate $sample = R(s, a, s') + max_{a'}(Q(s', a))$
	- update q value: $Q(s, a) = Q(s, a) - \alpha (Q(s, a) - sample)$
	- This process is essentially the same as temporal difference learning.
- Q learning will converge to the optimal policy even if you're acting
sub-optimally.
	- this is called off policy learning.
	- You have to explore enough.
	- The learning rate has to be small.
	- In the limit it doesn't matter how we select actions.
- Exploration vs Exploitation
	- Exploration - trying new things
	- Exploitation - going with what you know to be good
	- There is a balance between exploration and exploitation that needs to be
	struck.
- A simple scheme for forcing exploration is to force a random action (based on
some small probability).
	- This is called epsilon greedy.
	- This could backfire once you've learned the optimal policy.
		- This can be mitigated by lowering epsilon over time.
- Exploration function
	- Take a value estimate and visist counda dn return an
	optimistic utility $F(u, n) = u + k / n$.
		- k is some bonus
	- Then alter the Q update: $Q(s,a) <- _ {\alpha} R(s, a, s') + max_{a'}{f(Q
	(s',a'),N(s',a'))}$.
		- N is the number of times that action has been taken.
	- This not only boosts actions that haven't been tried but also boosts
	paths that lead to actions that haven't been tried.
- Regret is a measure of your total mistakes.
	- A measure of total mistake cost - the difference between our expected
	awards including youthful suboptimality and optimal (expected) rewards.
	- The exploration reduces regret compared to random exploration.
- Approximate Q Learning
	- A method of reinforcement learning with a large number of states.
	- We want to generalize a state as a vector (array) of features.
	- Our Q values are linear combinations of weighted feature pairs. The
	challenge is coming up with intelligent features.
	- Algorithm:
		- receive a sample $(s, a, s', r)$
		- calculate $difference = [r + \max_{a'}Q(s', a')] - Q(s, a)$
		- update $w_i = w_i + \alpha difference \ f_i(s,a)$.
		- $f_i (s,a)$ is the ith feature of $(s,a)$.
- Q learning is a great tool, but the feature based approaches that work well
aren't usually the ones that approximate V and Q well. Our solution is to learn
policies that maximize rewards as opposed to the values that predict them.
- Policy search starts from an ok solution (eg Q learning) then fine tunes by
conducting gradient descent on the feature weights.

# Probabilistic Reasoning

### Probability Basics
- A random variable is some aspect of the world about which we have
uncertainty. A random variable has a domain of possible values and is denoted
by a capital letter.
- Probability Distribution: the probability associated with each possible value
of a random variable. A probability distribution over random variable $R$
defines $P(R = x) \ \forall x \in D_R$. $P(R=x)$ can also be written $P(x)$ for
shorthand. A probability distribution must follow two properties:
	1. $P(x) >= 0 \forall x \ \in D_R$
	2. $\sum_{x \in D_R} {P(x)} = 1$
- Joint distribution: A joint distribution over a set of random variables $X_1,
X_2, ..., X_n$ specifies the probability of each possible assignment $P(X_1 =
x_1, X_2 = x_2, ... , X_n = x_n)$.
	- Given a distribution of $n$ variables with domain sizes $d$, the
	distribution is of size $d^n$.
- A probabilistic models is a joint distribution over a set of random
variables.
	- Given a joint distribution we can calculate the probability of any one
	event by summing over all arrangements where the event occurs.
	- An event is a subset of the assignments.
- A marginal distribution is a sub-tables which eliminates variables from a
joint distribution.
	- Essentially building a joint distribution over a subset of the random
	variables.
	- An easy example - given some joint distribution $X_1$, $X_2$ we can
	calculate $P(X_1 = x) = \sum_{X_2=x_2}P(X_1 = x, X_2=x_2)$
- Conditional probability: $P(a | b) = \frac{P(a, b)}{P(b)}$
- Conditional distributions are probability distributions over some variables
given fixed values of others.
- Product rule: $P(x, y) = P(x | y) P(y)$ - this is a method for encoding a
joint distribution.
- Chain Rule: We can write any joint distribution $P(x_1, x_2, ..., x_n)$ as
$$P(x_1, x_2, ..., x_n) = P(x_1)P(x_2 | x_1)P(X_3 | x_2, x_1) ... $$
or
$$P(x_1, x_2, ..., x_n) = \Pi_{i}^{n}{P(x_i | x_1 ... x_{i-1})}$$
- Bayes Rule: $P(x | y) = \frac{P(y | x) P(x)}{P(y)}$.
	- Prior Probability: $P(x)$
	- Posterior Probability: $P(x | y)$
- Two variables are independent if $P(X,Y) = P(X)P(Y)$ for all possible values
of $X$ and $Y$.
- Conditional Independence: Variables A and B are conditionally independent given
C if:
	- $P(A, B | C) = P(A | C)P(B | C)$
	- $P(A | B, C) = P(A | C)$

### Markov Models

- A markov model is a probabilistic inference model that is used to make
inferences based on sequences of actions over time and space. A markov model is
essentially a sequence of random variables such that each variable is only
dependent on the previous variable (or some set of previous variables)

```
 (x1) -> (x2) -> (x3) -> ... -> (xN)
```

- $P(x1), P(x2|x2), P(x3|x2) ... $ are called transition probabilities.
- Then we can use the chain rule to see determine the probability of a specific
sequence.
$$P(X_1, X_2, ..., X_T) = P(X_1) \Pi_{i=2}^{T} {P(X_i | X_{i-1})}$$
- Note here we assume that $X_i \bot X_{i-2} | X_{i-1}$. or that $X_i$ and $X_
{i-2}$ are conditionally independent given $X_{i-1}$.
- Markov models, when run for long enough, usually converge to a "stationary
distribution". We can solve for the stationary distribution either by running
the simulation for a while and by solving a set of linear equations. For a
binary state system
$$P(X_{\infty} = a) = P(X_{\infty} = a, X_{\infty - 1} = a) + P(X_{\infty} =
a, X_{\infty - 1} = b)$$
$$P(X_{\infty} = b) = P(X_{\infty} = b, X_{\infty - 1} = a) + P(X_{\infty} =
b, X_{\infty - 1} = b)$$
$$P(X_{\infty} = a) + P(X_{\infty} = b) = 1$$

__Hidden Markov Models__

- Markov models are very useful for reasoning over some kind of sequence (over
time or space).
- A Hidden Markov Model is a Markov Model with "hidden" and "observed" states

```
 (z1) -> (z2) -> (z3) -> ... -> (zN)
  |       |       |      ...     |
  V       V       V              V
 (x1) -> (x2) -> (x3) -> ... -> (xN)
```

- We define $E_{z_{i}}(x)$ as $P(x_i = x | z_i = z)$ and $T(z', z)$ as $P(z_
{i+1} = z' | z_{i} = z).

- Based on our knowledge of MMs above, determining the joint distribution of a
HMM is reasonably straight forward.
	- Let $B(Z_t)=P(Z_t | x_1...x_t)$
	- $B$ is a belief vector that essentially gives the probability of the state
	of the hidden state at time $t$ given the current "evidence" (observed
	states).
	- Then for one time step,
	$$P(Z_{t+1} | x_1...x_t) = \sum_{z}P(Z_{t+1} | Z_t = z) P(z_t |
	x_1...x_t)$$
	- And
	$$P(Z_{t+1} | x1...x_{t+1}) = \frac{P(x1...x_{t+1} | Z_{t+1})P(Z_{t+1} |
	x_1...x_t)}{P(x1...x_{t+1})}$$
- Forward Algorithm is a dynamic algorithm to get the probability of the
current state given all observed evidence. Eg find $P(z_t | x_1...x_t)$
$$P(z_t | x_1...x_t) = \frac{P(z_t,  x_1...x_t)}{P(x_1...x_t)}$$
$$P(z_t, x_1...x_t) = \sum_{z_{t-1}}{P(z_{t-1}, z_t, x_1...x_t)}$$
$$ = \sum_{z_{t-1}}{P(z_{t-1}, x_1...x_{t-1})P(x_1|x_{t-1}) P(x_t | z_t)}$$
$$ = P(x_t | z_t) \sum_{z_{t-1}}{P(z_{t-1}, x_1...x_{t-1})P(x_1|x_{t-1})}$$
$$P(z_t | x_1...x_t) =\frac{ P(x_t | z_t) \sum_{z_{t-1}}{P(z_{t-1}, x_1...x_
{t-1})P(x_1|x_{t-1})} } {P(x_1...x_t}$$
- Then the probability distribution of the HMM is given by
$$P(x_1...x_N, z_1...z_N) = P(z_1)E_{z_1}(x_i) \Pi_{k=2}^{n}{T(Z_{k}, Z_{k-1}) E_
{z_{k}}(x_k)}$$

__Particle Filtering__

- Particle Filtering is an approximate inference technique that reduces the
state space of an HMM (useful
if Z is too large to use exact inference). Instead of tracking values of Z,
track samples. Each sample is called a particle and time per step is linked to
the number of samples taken.
- Every particle can take some value $X_i$ - there are $ N << |X| particles in
the system. $P(x)$ represents the number of particles that take on the value
$x$ - so increasing the number of particles (obviously) increases the accuracy
of the approximate inference.
- Particle Filtering Algorithm:
	1. Distribute particles in some standard way (random, uniform, etc)
	2. For each particle x, update x based on change in time:
	$$x' = sample(P(X | x))$$
	where x' is the updated value of the particle, X' is the set of possible values
	of the particle, and x is the old value of the particle. Here we are essentially
	sampling from the transition probability distribution for a time step.
	3. For each particle x, and some noisy observation (evidence e), update x based
	on the observation:
	$$w(x) = P(e | x)$$
	$$B'(X) \propto P(e | X) B(X)$$
	where B'(X) is the new belief function based on the particles, P(e | X) is the
	probability of the evidence based on the current particle distribution (essentially
	the product of all the weights) and B(X) is the original particle belief distribution.
	Note that $B'(X)$ is not a probability distribution. We can renormalizes by resampling
	particles from $B'(X)$.

### Baysean Networks
- A baysean network is a graphical approach to representing a joint probability
distribution over some set of random variables. Bayes nets are used to build a
model of the world.
- A bayes net directed, acyclic graph (DAG)
where each vertex represents a random variable $X$ and an edge from $x$ to $y$
defines some "causation" or dependence of $y$ on $x$.
	- A bayes net consists of the dependencies and local "conditional
	probability tables" (CPTs) ie $P(X | parents_{X})$
	- We can use this local information along with the dependency information
	encored in the graph to "infer" the joint probability distrbution encoded
	by the network using the following equation:
	$$P(x_1, x_2, ..., x_n) = \Pi_{i=1}^{n} {P(x_i | parents(x_i))}$$
- A general joint distribution is of size $O(2^N)$ for $N$ variables. A Bayes
net is $O(N * 2^k)$ if each node has up to $k$ parents.
- Independence in bayes nets.
	- `(A) -> (B) <- (C)` : A and C are independent (blocking); A and C are
	conditionally dependent given B
	- `(A) -> (B) -> (C)` : A and C are conditionally independent given B
	(blocking); A and C are dependent
	- `(A) <- (B) -> (C)` : A and C are conditionally independent given B
	(blocking); A and C are dependent
- Sets $A$ and $B$ of random variables are independent given set $C$ if there
exists no unblocked path from the vertices in $A$ to the vertices in $B$ if the
vertices in $C$ are observed. $A$ and $B$ are said to be D-seperated by $C$.
	- So to determine if any vertices are independent given a set of observed
	variables we simply check to see if all paths between the variables are
	blocked.

__Inference__

- Inference is essentially the process of finding some useful information about
from a bayes net. Usually we want to know the "posterior probability" of some
query variables or the "most likely explanation".
	1. Posterior Probability: $P(Q | E_1 = e_i, ..., E_k = e_k)$
	2. Most Likely Explanation: $argmax_q P(Q = q | E_1 = e_i, ..., E_k = e_k)$
- Inference by enumeration is the most naive form of inference
	- Evidence variables $E_1...E_k = e_1...e_k$
	- Query variables: $Q$
	- Hidden variables: $H_1...H_r$
	- Calculate
	$$P(Q | e_1...e_k) = \frac{P(Q, e_1'...e_k)}{P(e_1,...,e_k)}$$
	$$P(Q, e_1,...,e_k) = \sum_{h_1...h_r}{P(Q, h_1,...,h_r, e_1,...,e_k)}$$
	- Note that final equation involves summing over all possible hidden
	variables. Also recall that $P(x_1, x_2, ..., x_k) = \Pi_{i=1}^{n} {P(x_i |
	parents(x_i))}$.
	- This is obviously a very inefficient solution. In fact, inference in a
	Bayes Net is an NP-hard problem.
- Factors
	- A "factor" is a conglomeration of nodes where some may be specified and
	other unspecified.
	- A single node is (itself) a factor.
- Joining Variables
	- We can We can join factors together by combining CPTs.

```
R = rain, T = traffic, L = late

Bayes Net
----------
(R) -> (T) -> (L)

P(R)
+r = 0.1
-r = 0.9

P(T | R)
+r +t = 0.8
+r -t = 0.2
-r +t = 0.1
-r -t = 0.9

P(L | T)
+t +l = 0.3
+t -l = 0.7
-t +l = 0.1
-t -l = 0.9

Factors
-------
Join on R
P(R, T) = P(T | R) P(R)
+r +t = .08
+r -t = .02
-r +t = .09
-r -t = .81

Join on T
P(R, T, L) = P(L | R, T) P(R, T)
+r +t -l = 0.024
+r +t +l = 0.056
+r -t -l = 0.002
+r +t +l = 0.018
-r +t +l = 0.027
-r +t -l = 0.063
-r -t +l = 0.081
-r -t -l = 0.0729
```

- Eliminating Variables
	- We can also use factors to eliminate hidden variables (essentially by
	summing variables together).

```
P(R, T)
+r +t = .08
+r -t = .02
-r +t = .09
-r -t = .81

Eliminate R = sum_R P(R, T)
+t = .17
-t = .83
```

- Join and elimination are the only two operations we need to do inference
enumeration and variable elimination.
	- In inference by enumeration we essentially join over all hidden variables
	and then eliminate the hidden variables.
- In variable elimination we essentially join on a single hidden variable and
then eliminate it.
	- Given some query $P(Q | E_1=e_1,...E_k=e_k)$
	- Start with local CPTs
	- While there are still hidden variables
		- Pick a hidden variable H
		- Join all factors mentioning H
		- Eliminate (sum over) H - this generates another factor.
	- Join all remaining factors and normalize by $P(e_1...e_k)$.
- Example

```
(R) -> (T) -> (L)
```

- Inference by enumeration
$$P(L) = \sum_{t}{\sum_{r}{P(L|t) P(t | r) P(r)}}$$
	1. $P(t | r) P(r)$ - join on R
	2. $P(L | t) P(r, t$ - join on T
	3. $\sum_{r}$ - eliminating R
	4. $\sum_{t}$ - eliminating T
- Variable Elimination
$$P(L) = \sum_{t}{P(L | t) \sum_{r}{P(r) P(t | r)}}$$
	1. $P(r) P(t | r)$ - join on R
	2. $\sum_{r}$ - eliminating R
	3. $P(L | t)$ - join on T
	4. $\sum_{t}$ - eliminating T

- The innovation in variable elimination is picking $H$ in an intelligent way.
Some choices of hidden variables would result in large factors whereas other
choices result in smaller factors - obviously smaller factors lead to more
efficient inference.
	- Generally speaking we should pick factors with less connections first
	since more connections implies a bigger factor table.
	- If we are combining k factors with domain sizes d, the resulting factor
	is of size	$O(d^k)$.
	- In variable elimination we care about the size of the factor tables.
	- The computational and space complexity of variable elimination is
	determined by the largest factor. And the elimination ordering can greatly
	affect the size of the largest factor.
	- There is no general "optimal ordering" which always results in small
	factors.
- A polytree is a directed graph with no undirected cycles. We can always find
an efficient ordering with polytrees.

__Approximate Inference__

- Approximate inference is a more efficient means of inference from a Bayes
Net. Obviously (as the name implies), approximate inference is not as exact as
the types of inference discussed above but it is often a more efficient method
of inference.
- Prior Sampling
	- Ignore all evidence and sample from the joint distribution.
	- Do inference by counting results.

	```Python
	def prior_sampling(X, evidence):
		'''
		 X is an ordering of the vertices in the bayes net such that
		  parents come before children
		 evidence is a dcit mapping vertices to the observed value
		'''
		x = np.array(n) # sample
		for i in range(1, n+1):
			x[i] = sample(P(X[i] | parents(X[i])))
		return x
	```

	- This process generates samples with probability
	$$S_{PS}(x_1,...,x_n) = \Pi_{i=1}^{n}P(x_i | parents(X_i)) = P
	(x_1,...,x_n)$$
	- As we increase the number of samples, our sampling becomes more
	accurate.
- Rejection Sampling
	- Very similar to prior sampling but with an "early abort" principle.
	- Ie if our sample isn't consistent with the query, abort that particular
	sample construction.

	```Python
	def rejection_sampling(X, evidence):
		'''
		 X is an ordering of the vertices in the bayes net such that
		  parents come before children
		 evidence is a dcit mapping vertices to the observed value
		'''
		x = np.array(n) # sample
		for i in range(1, n+1):
			x[i] = sample(from P(X[i] | parents(X[i])))
			if not x[i].consistent(evidence):
				return
		return x
	```

- Likelihood Weighting
	- Rejection sampling works well, but is non-ideal if the evidence is
	unlikely and / or far upstream. In these cases, there will be a lot of
	wasted work done on samples that are eventually going to be rejected.
	- We can fix this problem by "fixing" the evidence values and weighting the
	sample by the probability of the evidence given their parents.

	```Python
	def likelihood_weighting(X, evidence):
		'''
		 X is an ordering of the vertices in the bayes net such that
		  parents come before children
		 evidence is a dcit mapping vertices to the observed value
		'''
		x = np.array(n) # sample
		w = 1.0
		for i in range(1, n):
			if X[i] in evidence:
				x[i] = evidence[X[i]]
				w *= P(x[i] | parents(X[i]))
			else:
				x[i] = sample(P(X[i] | parents(X[i])))
		return x, w
	```

	- Thus we sample from a distribution
	$$S_{WS}(z, e) = \Pi_{i=1}^{l}P(z_i | parents(Z_i))$$
	$$w(x, e) = \Pi_{i=1}^{m}P(e_i | parents{E_i})$$
	- $l$ is the number of variables that are not evidence, $m$ is the number
	of variables that are evidence. (therefore $Z$ is the set of
	non-evidence variables and $E$ is the set of evidence variables). Therefore:
	$$S_{WS}(z, e) * w(z, e) = P(x_1, x_2, ..., x_n)$$
		- That is the product of the sample and weight recovers the original
		distribution in the limit.
- Gibbs Sampling
	- Likelihood Weighting is a good technique, but it doesn't take account of
	the evidence throughout the entire distribution. Ie downstream evidence is
	not efficiently considered at upstream variables.
	- The main idea behind Gibbs Sampling is that we start with some random /
	arbitrary assignment that is consistent with the evidence, sample one
	variable at a time based on the current conditions, and repeat this process.
	- This algorithm requires us to (in a sense) dome some inference, but these
	inferences are reasonably easy to calculate.

	```
	C = cloudy, S = sprinklers, R = raining, W = grass is wet
	    (C)
	    /  \
	  (S)  (R)
	    \   /
	     (W)

	 Evidence is +r (ie it did rain)
	 1) Randomly set all other variables
	 	+c, -s, +w, +r
	 2) Pick one variable
	   val_s = sample(P(s | +c, +r, +w)) // assume that becomes +s
	 3) repeat
	   val_w = sample(P(w | +c, +r, +s))
	 ...

	```

		- If we need to calculate $P(S | +c, +r, +w)$ we only need to join
		the CPTs containing $S$ which becomes relatively easy.

__Decision Networks__

- A decision network is essentially a network which incorporates standard Bayes
Nets and agents' actions. The bayes nets allow us to calculate expected utility
for each action.
- Node types
	- `()` - Bayes net node
	- `[]` - Actions
	- `<>` - Utility
- Action Selection Procedure
	1. Instantiate all evidence
	2. Set action node(s) each possible way
	3. Calculate the posterior for all parents of the utility nodes given the
	evidence.
	4. Calculate expected utility for each action
	5. Choose maximizing action.
- A utility table defines the utility of a utility node based on all values of
the parents of the utility node.
- In a decision network we can classify actions as "active" or "sensing". We
want to compute the value of acquiring information to determine when to take a
active action as opposed to gathering more information.
	- To accomplish this we can compute the maximum expected utility when we
	can't sense vs the maximum expected utility when we can sense.
- The value of observing a variable $E'$ given an observation of some variable
$e$ is given as:
$$VPI(E' | e) = \sum_{e'}P(e' | e)MEU(e, e') - MEU(e)$$
and
$$MEU(e) = max_{a} \sum_{s}{P(s | e)U(s, a)}$$
where $S$ is the set of all possible states and $s$ is an individual state.
	- Essentially, the value of perfect information of variable $E'$ is
	difference between the expected utility when we observe $E'$ and that when
	we don't.
- This develops from the general idea that:
$$MEU(e) = max_{a} \sum_{s}{P(s | e)U(s, a)}$$
If we see that $E'=e'$ then
$$MEU(e, e') = max_{a} \sum_{s} (P(s | e, e') U(s, a)) $$
But $E'$ is a random variable whose value is unknown so we can't know what its
value is so we find the expected value.
$$MEU(e, E') = \sum_{e'}(P(e' | e)MEU(e, e'))$$
Then:\
$$VPI(E' | e) = MEU(e, E') - MEU(e)$$
- VPI is non-negative and non-additive.
- If $parents(U) \perp Z | currentEvidence$ Then $VPI(Z) | currentEvidence > 0$.

- Example

```
[ Umbrella ]
          \
           -- < Utility >
          /
( Weather )
    |
    V
( Forecast )

F     P(F)
Good  0.59
Bad   0.41

W     P(W | F = Bad)
Sun   0.34
Rain  0.66

W     P(W | F = Good)

Rain

A        W      U(A, W)
leave    sun    100
leave    rain   0
take     sun    20
take     rain   70

MEU(F = bad) = max EU(a, F = bad)
MEU(F = good) = max EU(a, F = good)

```

# Machine Learning

- Machine learning is a very hyped subset of artificial intelligence that is used to make decisions or classifications based on trends in data. ML is an extremely powerful and diverse field (see machine learning notes). Here we go over some very basic ML algorithms.
- In general machine learning we have some model of our data, and we want to tune parameters to fit the example data (training data) we've seen. We define the likelihood function ($L(X, \theta)$) to be the likelihood of the data X and the model parameters $\theta$. We can assume that:

$$L(x, \theta) = \Pi_i P_{\theta}(x_i) = \Pi_i P(x_i | \theta)$$

- We want to maximize the likelihood function - this is the general idea behind the training part of an ML algorithm.

$$\theta_{ML} = arg \ max_{\theta'} P(X, \theta)$$

- There may also be a set of hyperparameters (learning rate, smoothening factor, dropout, etc) that we want to tune independently of the training data. Specifically we would run the training algorithm ont the training data for different values of the hyperparameters and pick the value of the hyperparameters and regular parameters that gives the best result.

## Naive Bayes

- Model based classification is the process of building a model where both the labels and features are random variables. We instantiate any observed features and query for the distribution of the labels conditioned on the features.
- The naive bayes model assumes that all features are conditionaly independent of each other given the label. THis makes this an extremely easy model to build.
- For example, the naive bayes model for digit recognition may look like:

```
val[0][0] -------->
val[0][1] -------->
...
val[1][0] -------->    Y
val[1][1] -------->
...
val[n-1][n-1] ---->
```

- Here our goal is to find $P(Y | X)$ where $Y = \{0, 1, 2, ..., 9\}$ and $X$ is the combination of feature values. Since we make a naive bayes assumption, we know that

$$P(Y | X) = \frac{P(Y, X)}{P(X)}$$

so
$$P(Y = y, X) = P(y) \times \Pi_{i,j=0,0}^{n-1,n-1}P(x_{i,j} = val[i,j] | Y = y)$$

Then

$$P(X) = \sum_{y=0}^{9}{P(Y=y, X)}$$

and

$$P(Y = y | X) = \frac{P(y) \times \Pi_{i,j=0,0}^{n-1,n-1}P(x_{i,j} = val[i,j] | Y = y) } {\sum_{y=0}^{9}{P(Y=y, X)}}$$


- For Naive bayes we need to know the parameters $\theta$ of the model - specifically these are the CPTs P(x_{i,j} | Y) and $P(Y)$.
- The easiest way to get these CPTs is to parse the data and count the number of specific occupance.
- Naive bayes can also be applied to text. The bag of words Naive bayes model is constructed such that feature $W_i$ is the word at position $i$. We assume that words are conditionally independent of each other given the label. In this case we want to classify emails as SPAM or HAM (not spam) based on the words they contain.

- In a Bayes net (and in ML models in general) we need to be able to handle unseen features since our training data may not have every possible feature value. Specifically we don't want to assign a 0 probability to unseen features because that could lead to division by zero errors. So we use Laplace smoothing. The basic idea behind Laplace smoothing is to pretend you saw every outcome once more than you actually did.

$$P_{lap}(x) = \frac{count(x) + 1}{\sum_{x}{count(x) + 1}}$$
$$P_{lap}(x) = \frac{count(x) + 1}{N + |X|}$$

- We can actually say $P_{lap-k}(x) = \frac{count(x) + k}{N + k|X|}$. Larger values of k ignore the training data whereas smaller values of k "trust" the training data more.
- We can do this with conditional probabilities

$$P_{lap-k}(x | y) = \frac{count(x,y) + k}{c(y) + k|X|}$$

- A big part of building a naive bayes model is extracting useful features from the data. For example, when classifying digits, the value of individual pixels is the only information encoded in the data, but we can extract a lot more information from the data. For example we can check the relative height and width of the digit, whether or not it has closed loops, etc.

## Perceptrons

- A perceptron is an error driven classifier. That means that it is trained  by computing the classificatino of specific training examples and uses the error between the classification and the correct answer to affect the model parameters.
- Naive bayes models can incorporate a variety of features, but tend to do best in homogeneous case (eg all vectors are the same type).
- A perceptron is a linear classifier that draws inspiration from the neuron.

$$activation_{w}(x) = sign(\sum_{i}w_{i}f_{i}(x)) = sign(w \dot f(x))$$

- The training algorithm of a perceptron tunes the value of $w$ which defines the decision boundary between the classes.
- The general training algorithm for a binary perceptron

```Pytohon
def train(X, Y, k, alpha):
	for iters in range(k):
		for x in X:
			y_hat = calssify(x)
			if y_hat != y:
				w -= alpha * x

def classify(x):
	return sign(w.dot(x))
```

- If the data is linearly separable, a perceptron is guaranteed to be able to converge to a solution.
- For multiple classes, we keep a weight vector $w_y$ for each class. Then to classify a sample $x$ we use

$$y = arg max_{y} w_y \dot f(x)$$

- After we classify a training example we must lower the score of the wrong answer and rais the score ofthe correct answer

```Python
def train(X, Y, k, alpha):
	for iters in range(k):
		for x in X:
			y_hat = classify(x)
			if y_hat != y:
				w[y_hat] += alpha * x
				w[y] -= alpha * x

def classify(x):
	return argmax(w.dot(x))
```

- On a linearly separable dataset we want o iterate until we make no changes to w, but usually we run for a limited number of iterations because most datasets aren't linearly separable.
- Perceptrons are mistake bound. This means that the maximum number of mistakes a binary perceptron can make is $\frac{k}{\delta^2}$ where $k$ is the number of features and $\delta$ is the margin (space between posative and negative class).
- There are a few problems with the perceptron
	- If the data isn't separable, the weights will thrash around during training. We usually average weight vectors over time to allevviate this problem.
	- The decision boundary isn't the one with the best margin.
	- The perceptron can easily overtrain.
- MIRA (Margin Infused Relaxed Algorithm) is a linear classifier that tries to pick a decision boundary with a better boundary. MIRA updates $w_y$ and $w_{y'}$ using the rule:

```Python
if y != y_hat:
	w[y] -= tau * x
	w[y_hat] += tau * x
```

- MIRA calculates $\tau$ by minimizing the sum of squares error: $min_{w} \frac{1}{2}\sum_{y}||w_y - w'_{y}||$ for each update where $w_{y}$ is the new value of the weight vector for y.

$$min_{w} \frac{1}{2}\sum_{y}||w_y - w'_{y}||$$
$$w_{y'} * x \geq w_y * x + 1$$
which is the same as minimizing
$$min_{\tau} || \tau * x ||^2$$
subject to the constraint
$$w_{y'} * x \geq w_y * x + 1$$
which means
$$(w'_ {y'} + \tau x) * x = (w'_{y} - \tau x) * x + 1$$
$$\tau = \frac{(w'_{y} - w'_{y'}) * x + 1}{2 x * x}$$

- This is very similar to the support vector machine which tries to maximize the margin of the decision boundary. However, MIRA maximizes the margin of individual examples whereas SVM maximizes the margin of the entire dataset.

## Kernels and Clustering

- Clustering is a means of classifying data based on how similar it is to examples you've already seen.
- 1 Nearest Neighbor is the easiest example. For a new example, assign it the label of the nearest classified data point.
- K Nearest Neighbors is similar to the 1 nearest neighbor example above, but we utake amajority vote of the k nearest neighbor classification. This KNN algorithm is an extremely simple, but powerful machine learning algorithm.
- This is an example of a Non-parametric model
	- Parametric : fixed set of parameters, more data means better settings
	- Non-parametric : complexity of the classifier increases with data, better in the limit, often worse in the non-limit.
- KNN is based on a similarity function which defines who close together two samples are.
	- An easy similarity function is to simply take the dot product of two features (after normalization).
	- Invariant metrics are metrics whose similarities shouldn't change under certain transformations (for example rotation, scaling, translation, stroke thickness, etc).
		- A way of encoding invariant metrics is to create a second dataset that contains a bunch of transformaed samples. Then we run the similarity function and return the pair with the highest similarity.
		- Another way of encoding invariant metrics is to just increase the size of our training data.
	- Template deformations is the process of creating an ideal version of each category and finding the best fit to the image with minimum variance.

- The dual representation of the perceptron is an alternate way of mathematically representing a perceptron. As defined a perceptron is defined as:

$$score(y, x) = w_{y} \dot x$$

but
$$w_{y}  = \sum_{i}a_{i,y}x_i$$
where $a_i$ is the update factor associated with sample $x_1$.

Then:

$$score(y, x) = \sum_{i}a_{i,y}(x_i \dot x)$$
$$score(y, x) = \sum_{i}a_{i,y} K(x_i, x)$$

- This allows you to compute the score without the weight. The function $K$ is a kernel function which is essentially a similarity function.
- We can use non-linear kernel functions to create non-linear decision boundaries.
	- A non-linear kernel can add features to the data space creating a higher dimensional space that can be linearly separated. The separator is linear in the higher dimensional space but non-linear in the original dimensionality.
- The dual perceptron is also called a kernel perceptron.
- Clustering is the process of inferring similarity between data. The point isn't to assign labels to data, but to group data together by similarity.
- K Means clustering
	- Pick K random points ass cluster centers (means)
	- Alternate
		- Assign data instances to closest mean
		- Assign a new mean as the average of the assigned points
	- Stop when no assignments change.
- Agglomerative clustering
	- All data points are there own clusters, then "agglomerate" two closest points. Repeat for a defined ammount of imes.

