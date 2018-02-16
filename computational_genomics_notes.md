---
title: Computational Genomics Notes
author: Haroon Ghori
geometry: margin=1in
---

# Introduction to Computational Genomics
- DNA Sequencing very inexpensive and done very frequently. Very common use in medicine and research.
	- Rare diseases
	- Ancient humans
	- Tumors (cancer)
	- Study basic genetics
- Computational Genomics - computational algorithms, techniques, etc used in DNA sequencing and analysis.
	- Example: Denovo shotgun assembly problem
	- Understanding underlying areas of comp genomics helps us know what works in specific situations, when techniques will fail, and helps improve current techniques.
- Chain termination method (Sanger Sequencing)
	- First method of DNA sequencing
	- Dominant method of DNA sequencing until the Human Genome Project (HGP)
	- Wom a Nobel Prize
- Massivly Parallel Sequencing (Second / Next Generation Sequencing)
	- Lowered cost
	- Improved speed
	- Improved accuracy
- Basic Biology
	- DNA is made up of 4 bases: A, C, G, T
	- Double helix
	- Complimentary base pairs
		- A::T
		- C::G
	- DNA can be written as a string of base pairs
- Sequencing
	- Read short stretches of DNA in parallel
		- Known as sequencing reads
		- Very short compared to entire chromosome

### Sequencing by Synthesis
- Computer watches natural DNA replication process to create a string sequence of the DNA. 
- Lab work: extract DNA, convert to multiple short, single stranded templates, put on sllides. 
	- DNA helicase, Ligase
- Add DNA Polymerase and bases
	- Bases have an attached terminator which forces the replication process to stop after each complimentary base is added. 
	- Terminators flouresce with a certain color corresponding to their base.
	- Computer uses the coloring to determine which base was added
	- Cleave off the terminators
	- Next base is added
	- repeat until replication process is complete. 
	- Based on this data we can figure out the sequence of the "template strand" and the "compliment strand"
	- Note: the florescence gets slightly masked by the florescences of the bases around it.
		- Sometimes a base without a terminator is added to a sequence - this makes the sequence out of sync and creates sequencing errors. 
	- Base caller software uses the colors it detects to determine the probability that there was a sequencing error (ie the base call was incorrect). 
		$$Q = -10 * log_10(p)$$
	Where Q is base quality and p is probability that base call is incorrect. As Q goes up, confidence in correctness goes up. Q = 10, 1 in 10 chance of wrong, Q = 20, 1 in 100 chance of wrong
- Once all reads are completed we must put the sequences together to get usable data. This is done (generally) by pattern matching based on qualities of the read and reference genomes from the Human Genome Projects and others.
	- Read allignment problem
		- First large problem in genomics
- If there is no reference genome putting the reads together gets much more difficult. 
- FASTQ Files
	- Real world DNA sequencing files
	- Each "read" contains
		- Line 1: ID of experiment, other related information
		- Line 2: Actual base pairs
		- Line 3: Other information
		- Line 4: Base quality characters
	- Each base quality is an ASCII encoding of the base quality (Q)
		- Given letter q in the base quality character corresponding to the base b.
		- then the Q value associated with q (and by extension b) is given by `Q = ord(q) - 33`
		- This is known as phred33 encoding
- Tries
	- A trie "try" is a tree represting a collection of string (kesys) over an alphabet Sigma. It is the smallest tree such that
		- each edge is labeled with a character c E Sigma
		- for a given node, at most one child edge has label c, for any c E Signma
		- Each key is "spelled out" along some path starting at root
		- The leaf nodes contain data which can be used as part of a map
	- To traverse the trie, start at the root and process each character.
	- If we add strings a, b, ... to the trie and len(a) + len(b) + ... = N then the trie has at most N edges
	- Checking for key P in a trie where len(P) = n touches (in the worst case) O(n)
	- The worst combination of keys in a trie is a set of keys which start with different letters.
	- Possible node representations
		- dense lookup table
			- 2 dimensional array of pointers
			- every possible combination in the alphabet
			- pointer is null if there is no child
		- hash table (dictionary)
			- key = character, value = pointer
		- parallel sorted arrays
			- sparese array containing characters corresponding to pointers
- Exact Matching Algorithms
	- The problem: Given a string T, find the location(s) where a pattern P occurs in T. 
		- Assuming len(P) < len(T)
	- Naive Algorithm
		- The most simple and intuitive matching algorithm
		- Essentially try all different alignments of P in T and return match locations.
			```Python
				def naive_matching(T, P):
					locations = []
					for i in range(0, len(T) - len(P)):
						for j in range(0, len(P)):
							if P[j] != T[i+j]:
								break
							if j == len(P) - 1:
								locations.append(i)
					return locations
			```
		- Algorithm is generaly O(N) -> varies by relative sizes of strings
	- Boyer More Algorithm
		- Very popular exact matching algorithm for strings.
		- Much more efficient than naive algorithm.
		- Components
			- Try allignments in left to right, but character comparasins in left to right
			Example
			```
				T: there would have been a time
				P: -------word---------------->
				            <-
			```
			- Bad Character Rule
				- Upon a mismatch skip allginemtns until mismatch becomes a match or P moves past mismmatched character
			- Good Suffix Rule
				- let t = substring that is matched in the current alignment of P in T.
				Example
				```
					        x_t_
					T: CGTGCCTACTTACTTAC
					P: CTTACTTAC
					     '''-->
				```
				- Upon a mismatch skip allignments until there are no mismatches between P and t or P moves past t.
		- When using Boyer Moore we use the "rule" that gives the maximum shift - minimizes the number of alignments to check. 
		- To use Boyer Moore we need to build lookup tables to figure out skips 
		Example 
			```
				P = TCGC
				  T C G C
				A 0 1 2 3
				C 0 - 2 -
				G 2 1 - 0
				T - 0 1 2
			```
			- By creating a lookup table, Boyer Moore is said to preprocess the pattern P.
	- Preprocessing
		- Preprocessing is performing some computation before doing the actual computation to speed up computation over all.
			- For example, creating a lookup table in Boyer Moore is preprocessing
		- Costly to do once, but amortized over many iterations
		- Matching algorithm that uses a preprocessed version of text T is said to be offline
		- Matching algorithm that doesn't use a preprocessed version of text T is online
		- Main use of offline in genomics
			- Looking for originis of multiple DNA snippets in large (well defined) genome (ie Human Genome). Eg when pattern P varies, but text T is relatively static.
		- In genomics, offline algorithms tend to be very useful.
	- Indexing
		- Offline algorithm which preprocces text into an "indices".
		- Text can be indexed by order or group.
			- Real world example: index in a book, decimal system (library), asile in a grocery store
			- Substrings
				- Intuitive option
				- Given text T, we want to index into substrings of length k (k-mers)
				- iterate through T, take every substring of length k and add the substring + its index in T to a data structure
			- Sequences
				- Given text T, we want to index by sequence
					- Sequence is a "pattern" in the string
					- Every substring is a sequence, not every sequence is a substring
					- Tends to increase the specificity of the filter (inex hit is more likely to lead to match)
			- Substring / Sequence Implimentation
				This is essentially a multimap (python dictionary): `{KEY: (VALUE_1, VALUE_2,..., VALUE_N)}`
				- Sorted list
					- Use binary search to find specific key
						- O(log(N))
						- look up python bisect module (bisect_left)
						- relatively slow, but doesn't take up much space
				- Hash table
					- Use hash code to find specific key
						- O(1) search
						- python dict structure
						- relatively fast, but takes up more space
						```Python
							for i in range(0, len(T) - k):
						      if T[i:i+k] in key:
						          index[T[i:i+k].append(i)
						      else:
						          index[T[i:i+k]] = [i]
					    ```
					Example
					```
						T: CGTGCGTGCTT
						split in to 5-mers
						CGTGC 0, 4
						GTGCG 1
						TGCCG 2
						GCGTG 3
						GTGCT 5
						TGCTT 6
						^^^ Preprocessing step (Indexing)
					```
					- Now if we search for a string of length > 5
					```
						P: GCGTGC
						search for [GCGTG] -> exists at index 3 (Index hit)
						now verify rest of string
						First letter of index 4 = C
						Last letter of P = C
						Therefore P exists at index 3 (match)
					```
				- When taking substring indices you don't neccesarily have to take every k-mer in T. You can index every other k-mer, every third, etc. 
					- This reduces space, makes querrying faster, etc. 
					- If you use this approach you must use offsets (i) in P such that if you use every n-th k-mer in P, you take n offsets such that `i % 0..n = [0...n-1]`
			- Suffix
				- We can use the suffixes of the text T as indices. 
					- This intuitively seems to require a large ammount of space. However if we choose the right data structure, it doesn't.
				- Implimentation
					- Suffix Trie
						- We can use the suffixes of the text T as indices. 
							- This intuitively seems to require a large amount of space. However if we choose the right data structure, it doesn't.
						- Essentially a Trie built out of all suffixes of the string.
						- A substring of T must be a prefix to one of the suffixes. 
						- Longest repeated substring
							- Traverse the tree and look for he deepest node with a branch
						- Number of times a substring appears
							- Given substring S, traverse the Trie from the root using the characters of S. Once we've reached node S determine the number o leaf nodes under S.
						- A suffix trie's growth is upper bounded by O(N^2) and lower bounded by O(N)
							- We can see this by examining the function that constructs a suffix trie
								- Essentially the function consists of two nested for loops which, in best case is O(N) and in worst case is O(N^2)
						- Suffix tries from DNA grow close to the worst case - ie O(N^2)
							- Thus the suffix trie is not practical for DNA
					- Modified suffix trie
						- Given a suffix trie, for every path with no branches, coalesce into one edge with a string value at the edge.
						- Every node has either 0 or >=2 edges
						- There are at most m leaf nodes
						- There are at most m-1 non leaf nodes
						- Worst case, this type of trie grows linearly O(N)		

### Approximate Matching
- Given DNA T and pattern P, what if we wanted to find places in T where P is "almost a match" - in computational genomics this could be due to variation or sequencing errors. 
- We really want to solve the "approximate matching problem" instead of exact matching. 
	- Mismatches
	- Insertions
	- Deletions
	- Hamming distance: Given 2 strings X and Y of equal length, the hamming distance between X and Y is the minimum number of substitutions needed to turn one into the other. 
		- Simply, the number of differences between X and Y
		- Hamming distance is not defined for `len(X) != len(Y)`
	- Edit (Levenstein) distance: Given 2 strings X and Y, the edit distance is the minimum number of edits (substitutions, insertions, deletions) needed to turn one into the other. 
- The Pigeonhole Principle
	- If P occurs in T with up to k edits, then at least one of p1, p2, ...pk+1 must appear with 0 edits.
	- Given a pattern P of length N with up to K errors (N > K). Split P into partitions P1, P2, ..., Pk, Pk+1 and check each partition for exact matches. If one is an exact match, check rest of pattern approximate match with < K errors.
		- partitions must be nonempty, nonoverlapping
		- Advantages
			- can use any exact matching algorithm with modifications
			- can use hamming distance or edit distance
		- Disadvantages
			- large k leads to small partitions -> increases verification
			- requires K+1 searches (less efficient)
			- lots of verification steps
		- Running Boyer More with Pigeonhole Principle
			- Basic Algorithm
				- Define K as the max number of edits
				- split P into k+1 partitions
				- run Boyer More for each partition
				- find matches for each partition
				- verify for all matches
			- Cost
				- If N is the number of characters checked when k = 0
				- $N_k = N * (k+1)^2$
					- Running through k+1 Boyer Moores (one for each partition) and increasing the number of matches / verification steps
				- Clock time increases by a proportional ammount
				- Number of matches increases
	- More general version of the Pigeonhole Principle
		- Given a pattern P of length N with up to K edits (N > K). Then at least n of $p_{0}, p_{1}, p_{2},...p_{k}, p_{k+1},...p_{k+n}$ must appear with 0 edits.
			- We can use this principle to make the filter more specific.
		- Let $p_{0},...p_{j}$ be a partitioning of P. If P occurs with up to k edits then at least on of $p__{0}, p_{1},...,p_{j}$ must occur with $\leq floor(k/j)$ edits.
- Q-Gram Lemma
	- Split P into overlapping partitions of length q
		- called q-grams
	- If there are k edits in a P of length n with q-grams the worst case number of disrupted q grams is given by the expression $n-1+1-kq$
	- If P occurs in T with up to k edits, allignment must containn t exact matches of length q, where $t >= n-q+1-kq$
		- Once these matches are found we can use verification to find approximate matches.
	- Given n and k, how long a q-gram is garunteed to match
		$n-1+1-kq >= 1$
		$q <= n / (k+1)$
		- If we want to pick the largest possible q we pick q = floor(n / (k+1))
	- If P occurs with <= k edits, allignment contains an exact matcho of length q where `q = floor(n / (k+1))`
