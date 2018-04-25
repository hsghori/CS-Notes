---
title: Parallel Programming Notees
author: Haroon Ghori
geometry: margin=1in
---

### Introdcution to Parallel Programming 
- Moore's Law 
	- The number of transistors that can be inexpensively placed on an integrated
	circuit doubles approximately every two years. 
	- This "law" has held for the last fifty yeats. 
- Moor's law is technically true but deoesn't mean our chips are faster. 
- Dennard scaling
	- As transistors get smaller, their power density stays constant - power is
	proportional to area. 
	- Therefore performance per watt increases exponentially.
	- This phenomenon flatlined in 2006. The most cited reason is that at small
	sizes, current leakage becomes a big issue and causes the chip to heat up. 
	This leads to "thermal runaway" and increases energy cost. 
	- so while transistors / chip have been
	increasing exponentially, clock speed, power, and performance / clock have
	remained fairly constant. 
- Moore's law implicitly resulted in parallelism - increasing the number of
transistors increased the number of independent processing unites on the chip.
Software engineering trends are just now catching up to this parallel
architecture. 
- Amdahl's Law: Improving a portion $P$ of a computation by factor $s$ results
in an overall speedup of 
$$S_{latency}(s) = \frac{1}{(1-P) + \frac{P}{s}}$$
This law essentially means that the speedup is limitted to the fraction that 
can be improved by speedup. 
	- $P$ is the fraction of the code that runs in parallel
	- $s$ is the number of resources
	- $S_{latency}(s)$ is the observed speedup
- The non-parallel part of the code is said to be "sequential". 
- We define speedup as $\frac{T(1)}{T(s)}$ where $T(1)$ is the time to execute 
a task on a single resource and $T(s)$ is the time to execute a task on $s$
resources. 
- Gustafson's Law: Improving a portion $P$ of a computation by factor $s$
results in an overall speedup of 
$$S_{latency}(s) = (1-P) + sP$$
	- Weak scaling is really a measure of the overhead needed to 
- Strong scaling: how the solution time varies with the number of processors 
for a fixed total problem size. 
	- Designing a strong scaling experiment: Run a program on a fixed problem 
	size and vary the number of threads. 
	- Strong scaling attempts to answer: how much faster can we solve this 
	problem with parallelism? 
	- Addressed by Amdahl's law
- Weak scaling: how the solution time varies with the number of processors for 
a fixed problem size per processor. 
	- Designing a weak scaling experiment: Run a program with a fixed number of
	iterations per thread and vary the number of threads. 
	- Weak scaling attempts to answer: can we solve bigger problems with more
	parallelism? 
	- Addressed by Gustafosan's law.
	- Weak scaling is becoming less feasible as processor growth is 
	outstripping memory growth. Since weak scaling involves solving bigger 
	problems, more memory is key. 
	- Weak scaling is difficult to express in algorithms.  

### Factors Against Parallelism
- there are three major factors against parallelism. 
	1. Startup costs associated with initiating processes. 
		- May overshadow actual processing time. 
		- Involves thread/process creation, data movement, etc. 
	2. Interference - slowdown resulting from multiple processes accessing 
	shared memory. 
		- Resources: memory, I/O, system bus, sub-processors 
		- Software synchronization: locks, latches, mutex, barriers 
		- Hardware synchronization: cache faults, HW mutexes and locks. 
	3. Skew - when breaking a single task into many smaller tasks, not all 
	tasks may be of the same size
		- Therefore not all tasks finish at the same time. 
- There are a few other factors that can degrade parallelism
	- I/O, network communication, failed processes, etc. Generally we focus on
	communication among processes as a major chokepoint. 
- Parallel computation proceeds in phases - first we compute some local data,
then exchange the data with other processes. 
	- Communication is governed by the latency (fixed cost to send a message) 
	and the link capacity. 
	- Latency dominates for small messages and bandwidth dominates large messages. 
	- We often want to overlap communications to reduce wait time between 
	computing phases. 
- Bulk Synchronous Parallel is a natural abstraction for parallel computation. 
	- Processing is divided into "supersteps" - each superstep has three 
	phases, compute, communicate, and barrier. This allows for no overlap.  
	- Communication and barrier are sequential (unoptimized). 

### Serial to Parallel / OpenMP
- The easiest way to write parallel code is to convert some serial code to
parallel execution. The idea
behind this process is to write some serial code, profile it to find the
slowest parts, and use a framework like OpenMP to parallelize (and therefore
increase the speed of) the code.
- This is not the best way to create parallel programs, just the easiest. 
- OpenMP is a parallel programming environment for master/slave and/or fork/
join execution model, loop parallelism, and shared memory architectures. It
allows us to write serial programs in C++ or fortran, add parallel
directives, and get a parallel program that computes the exact same result. 
	- OpenMP is not fundamentally a serial to parallel environment, but it is 
	most often used in that way. 
- OpenMP works with a shared memory architecture.
	- A shared memory architecture is hardware that provides the abstraction
	of coherent read/write to common memory from multiple resources
		- Coherent - repeatable reads 
		- Abstraction that there is a single memory for all processes 
		- Data can be shared between process by reading and writing to memory. 
	- This abstraction holds even if there are different physical memories. 
- To run a a block in parallel using OpenMP we need to include omp.h and call 
the directive `#pragma omp parallel <options>`{.c++}. 
	- We can easily parallelize for loops using the directive `#pragma omp 
	parallel for <scheduling opeions>`.{c++}. 
	- The scheduling options define how to schedule the threads (how many
	iterations to run before switching, etc). 
	- `<scheduling options> = kind [,chunk size]`.
		- static - divide loop into equal sized chunks 
		- dynamic - build internal work queue and sipatch blocksize at a time
		- guided - dynamic schedulign with decreasing block size for load balancing. 
		- auto - compiler chooses from above 
		- runtime - runtime configuration chooses from above. 
		- chunk size = number of iterations per thread. 
- Technically speaking, OpenMP is master/worker. There is some entry thread 
that runs the serial content and creates other threads. 
- OpenMP can be thought of as "loop parallelism" - ie running a loop in
parallel. This is a pretty easy concept to think about and work with when 
coming from serial programming. However we need to be careful about how our
loops access memory - especially nested loops. 


### System Architectures
- Flynn's Taxonomy - characterizes machines by the number of instruction 
streams and data streams. 
	- SISD - single instruction, single data 
	- SIMD - single instruction, multiple data 
	- MISD - multiple instruction, single data 
		- This doesn't exist 
	- MIMD - multiple instruction, multiple data
- SISD is called the Von Neumann architecture. This implements a universal
turing machine and conforms to serial algorithmic analysis. This is how most
people think of a computer
- SIMD has a single instruction stream that is carried out on multiple data
streams. Eg one program happening on multiple data sets. 
- MIMD defines most of the machines we're interested in - multi cores, SMP,
clusters, etc. 
- Flynn's taxonomy is a bit too simplistic to discuss the wide range of parallel
architectures in existence. 
- Shared Memory Systems. 
	- Large class defined by the shared memory architecture model. 
	- Threads exchange data through reads and writes to memory. 
	- Has some synchronization constructs to control sharing. 
- Symmetric Multi-Processor 
	- Shared memory (MIMD) system. All processors can access all memory. 
	- All memory locations can be accessed at equal speeds. This is called 
	symetric memory access. 
	- SMPs have scaling limits. Increasing the number of processors increases
	competition for memory (fixed memory). 
- Non Uniform Memory Access (NUMA) 
	- Shared Memory (MIMD) system. All processors can access all memory. 
	- Memory access is not symmetric - eg each processor can access different
	memory locations with different speeds based on physical "closeness" to the
	memory. Each processor has a cache to mitigate potential memory access 
	bottlenecks. 
	- Same programming semantics as SMP but more difficult to do efficiently. 
	Must consider memory access and caching. 
- Message passing
	- Each processor / node has its own private memory. 
	- Nodes synchronize actions and exchange data by sending messages to each
	other. 
	- MPI - libraries that allow for collective operations, synchronization, etc.
		- Explicit handling of data distribution and inter process communication. 
	- Map/reduce - divides computation into data parallel and data dependent
	portions. This is a better abstraction of hardware and more restrictive. 
- Hybrid Architectures 
	- A message passing machine has SMP at each of its nodes. 
- Just as there are different models for processor architecture there are
different architectures for memory and cache architectures. 
- Memory itself is an abstraction - it is really a steep hierarchy of caches. 
	- different levels have different performance, capacity, sharing, 
	semantics, etc. 
- Cache Hierarchy
	- L1 - on chip 
		- One per processor
	- L2 - on chip. 
		- 10 x slower, bigger, cheaper than L1. Shared among coes
	- L3 - most modern processors 
	- Main Memory 
		- 10x slower, 2000x bigger, 20x cheaper than L2. Shared among multiple
		processors. 
	- SSDs
		- 10x slower, 100x bigger, 20x cheaper than memory 
	- disk dirves 
		- 10 x slower, 5x bigger, 5x cheaper than RAM 
- Caching concepts 
	- Cache line - fundamental unit of cachine (line size is a power of 2)
	- Inclusive / exclusive: 
		- Strictly inclusive - all data in L1 must be in L2
		- Exclusive - data in L1 cannot be in L2 
	- Associativity - number of locations a cache block can go into 
	- Unified - shared among multiple cores 
- Based on this hierarchy, different memory access patterns can result in
drastically different performance for the same computation. In parallel
programming we want to use all the data in a cache line and if the program is
referencing the data in the cache more than once. 
	- Accessing a single cache requires loading the entire line. 
- Memory terms
	- Aligned - access range starts / ends on cache line boundaries 
	- Sequential - a continuous range of bytes 
	- Coalesced - uses all associated regions of memory. 

### Loops 
- Loops are relatively simple constructs to parallelize, but we need to take a
few factors into account. 
- Loop carried dependencies: when one iteration of a loop depends on the
computations from other iterations. 
	- Addressed by loop rewriting, removable dependencies, and seperable
	dependencies. 
- Loop optimizations: changing loops so they can run faster. 
	- Fusion: Merge loops to create larger tasks - amortize startup. 
	- Coalesce - coalesce loops to get more UEs and thus more parallelism. 
	- Unrolling - loops that do little work have high startup costs, unroll 
	loops
	by hand to reduce startup cost. 
	- Memory Access Pattern - consider how loops iterate over memory and 
	optimize to access sequential memory. 2D array is stored in rows - better 
	to access the entire row in one loop. 
	- Loop tiling - localizes memory twice (in cache lines and in cache 
	registers).
- OpenMP reductions - in OpenMP we sometimes need to share a value between
threads while avoiding race conditions. We can do this by marking a block as
critical, but that can be inefficient. We use reduction functions to more
efficiently handle these cases without interference. 

```C++
#pragma omp parallel for shared(max_val)
for (int i = 0; i < 10; i++) {
	#pragma omp critical 
	{
		if (att[i] > max_val) {
			max_val = arr[i];
		}
	}
}
```

we can reduce this code to

```C++
#pragma omp parallel for reduction(max : max_val)
for (int i = 0; i < 10; i++) {
	if (arr[i] > max_val) {
		max_val = arr[i];
	}
}
```

This essentially tells the compiler to use the max function to update max_val.

### OS Abstractions 
- A task is a sequence of instructions that operate as a group. 
- Unit of Execution (UE) is the execution context for a task
- Preprocessing element (PE): the hardware element that runs the UE
- A process is the operating system's abstraction for a running program. 
	- A process provides the illusion that the program is the only one running 
	on the system. 
	- Each process appears to have exclusive use of the hardware and execute
	instructions one after another without interruption. 
	- Processes do not share memory. 
	- The process abstraction allows for multiple serial programs to run
	concurrently and in parallel. 
- Context switch: how the operating system switches among running processes. 
- Context: the per-process state maintained by the OS. 
	- The context is required to suspend and resume processes and includes the
	program counter, register file, and contents of memory. 
- Virtual Memory - the memory abstraction processes have. This allows them to
believe that they have exclusive use of memory. The memory itself contains the
state of a single program (code, data, stack, heap, mapped memory).
- We can use processes to create parallel programs
	- `fork()` creates a copy of the running process 
	- `exec()` loads a new program into the process. 
	- A parallel implementation using process would esentially fork multiple 
	times and execute the same process on each fork. The OS assigns the 
	processes to different cores. 
- Properties of multi process programs 
	- Processes have no shared state
		- Applications that need to share state must do so explicitly using
		inter-process communication. These interfaces are generally clunky. 
	- No shared state means no dependencies - theis makes parallelizing easier
	using a message passing architecture. 
	- Multiprocess programs are generally the simplest parallel programs. 
- A thread is a concurrent execution unit within a process. 
	- Unlike a process, threads can share memory. 
	- Threads have a more lightweight context (thread ID, stack, and registers). 
	- They share the heap and libraries and can share data using shared memory. 
- Every process has at least one thread. 
- Simultaneous multithreading - run multiple threads on each core each cycle. 
- Hyperthreading - processor and OS advertise two threads per core. 
- Threading can greatly increase performance - as much as linear speedup with
each core. Other benefits are more incremental
	- Simultaneous multi threading can improve speedup by 15-30%
	- Threading can overlap cache stalls with the execution of other threads 
	- Can decrease cycles per instruction for some workloads. 
- Properties of multi thread programs 
	- Shared variables
	- Easy to communicate between threads
	- Hidden dependencies may be hard to debug
	- Can be abstracted using object oriented programming. 
- Threads in Java
	- Java has a standard interface to handle threads. The Runnable interface 
	is used to handle the execution and the Thread class handles 
	multithreading. 
	- There are multiple ways to implement multithreading but the easiest (and
	preferable) way is to implement Runnable and wrap it in a thread. 
	- When we run a thread we should call join() on it to wait for it to 
	finish. 
	- In Java threading we may need to read / write to shared variables.
	Uncontrolled sharing can lead to unpredictable outcomes and race 
	conditions. 
		- Atomic - in transactions either all actions happen or none happens. 
		- Synchronize - a block that's only accessed by one thread at a time. 
		- volatile - a variable that is guaranteed to have memory synchronized 
		on each access. 

### Synchronization 
- Synchronization is the process of preventing multiple UEs from accessing the 
same resources at once - ie preventing race conditions. 
- Synchronization properties:
	- Mutual exclusion - the property that only one UE can access a shared 
	resource at a time. 
	- Deadlock freedom - the property that at least one UE will eventually be
	able to access the resource. Ie the UEs won't get stuck waiting on the 
	resource forever. 
	- Starvation freedom - the property that a UE won't be forever locked out
	from using the resource. Ie if a UE wants to use a resource it will 
	eventually be able to use the resource. 
- Transient communication - a form of communication thhat requires both parties
to communicate at the same time. Ex: talking on the phone, shouting from a 
distance, etc. 
- Persistent communication - a form of communication that allows both parties 
to communicate at different times. Ex: writing notes, raising flags, etc.
- We can characterize concurrency with two properties
	- Safety (correctness) - what are the symantic garuntees expressed by a 
	locking protocol under concurrent execution. 
	- Liveness - how can the execution of one thread by delayed by other threads 
- Locks - A lock is a "lock" or hold on a resource. There are a few ways a 
parallel program can implement locks. 
	- Spinlock - a lock which causes the thread trying to acquire it to wait in a
	loop (spin) while checking to see if the lock is available. 
		- Spinlocks are generally used in OS, but not so great for parallel processes. 
		- When the lock is freed, all the other UEs will try to acquire it - 
		performance may vary. 
	- Ticket lock - a lock that uses a FIFO ticket system (like the DMV) to 
	control which thread can access resources. 
		- There is some queue of threads waiting to access a critical section. 
		- There is a variable queue_len which keeps track of the number of 
		threads in the queue. 
		- When a thread enters the queue, it sets `thread.ticket = queue_len` 
		and increments queue_len (`queue_len++`). 
		- When a thread leaves the critical section it returns `dequeue =
		thread.ticket++`. Every other thread checks to see if `thread.ticket =
		dequeue`. If so, that thread can now enter the critical section. 
		- Ticket lock is FIFO, mutually exclusive, and deadlock free. 
- Semaphore - a variable used to control access to a shared resource. 
Semaphores are usually implemented in operating systems. These are especially
useful when more than one process can access the resource -

```
# s = semaphore_value
await s > 0;
down(s)
critical section 
up(s)
```

- Synchronization algorithms:
	- Peterson's algorithm 

	```
	// process 0
	b[0] = true 
	victim = 0
	await (b[1] = false or victim = 1);
	critical section 
	b[0] = false

	// process 1
	b[1] = true
	victim = 1
	await (b[0] = false or victim = 0)
	critical section 
	b[1] = false
	```

	- b[x] indicates process x wants to enter the critical section. 
	- Write to victim indicates who got there first
	- This algorithm is mutually exclusive, starvation resistant, contention 
	free, and has arbitrary wait. It lasso requires atomic registers. 
	- This algorithm requires busy waiting - eg the await construct. This is a 
	good construct when there are many processes, there's no othe ruseful work 
	to do, or wait times are very short. The alternative is to sleep / restart. This is usually implemented by hardware interrupts and has more overhead. 
	- Lamport's algorithm scales mutual exclusion to n processes

	```
	start: 
	b[i] = true;
	x = i
	if y != 0
		b[i] = false 
		await y == 0
		goto start
	y = i
	if x != i
		b[i] = false
		for j = 1 ... n:
			await b[j] = false
		if y != i 
			await y = 0
			goto start
	critical section 
	y = 0
	b[i] = false
	```

	- This algorithm has fast MutEx properties. However, starvatio nis 
	possible since the wait time is unbounded. 
	- Peterson and Lamport's algorithms are too simple to be used in practice but
	demonstrate fundamental trade-offs in synchronization. Space vs speed vs
	fairness. 
	- Both of these algorithms require atomic registers which are not available in
	distributed memory machines. 
	- Lampart's bakery algorithm is an algorithm meant to improve saftey in the
	usage of shared resources among mutual threads using mutual exclusion. 
		- Lamport envisioned a bakery with a numbering machine at its entrance 
		so each customer is given a unique number. Numbers increase by one as 
		customers enter the store. A global counter displays the number of the 
		customer that is currently being served. All other customers must wait 
		in a queue until the baker finishes serving the current customer and 
		the next number is displayed. When the customer is done shopping and 
		has disposed of his or her number, the clerk increments the number, 
		allowing the next customer to be served. That customer must draw 
		another number from the numbering machine in order to shop again.
		- According to the analogy, the "customers" are threads, identified by 
		the letter i, obtained from a global variable.
		- Due to the limitations of computer architecture, some parts of 
		Lamport's analogy need slight modification. It is possible that more 
		than one thread will get the same number n when they request it; this 
		cannot be avoided. Therefore, it is assumed that the thread identifier 
		i is also a priority. A lower value of i means a higher priority and 
		threads with higher priority will enter the critical section first.
	- The Bakery algorithm is FIFO and satisfies mutual exclusion. 
	- The bakery algorithm is not fast. 
- Common atomic operations
	- Read, write, test-and-set, swap, fetch and add, read-modify-write,
	compare-and-swap
- Barrier - a software implementation to synchronize processes on 
asynchronous hardware. 

### Message Passing
- Clusters and multi-process programs with no shared state sometimes need to 
communicate with each other. 
	- SPMD model (single program, multiple data), master / slave, loop 
	parallelism
- MPI is the "assembly language of parallel programming" - it is a message 
passing interface with support for C++ and Fortran. It is used to send and 
receive messages across processes. 
- When writing message passing protocols it's important to avoid potential 
deadlocks. 
	- Synchronous - the caller waits for the message to be delivered prior 
	to returning. This is the fallback mode for MPI messaging. 
	- deadlocks are cycles in a dependency graph. 
	- the easiest way to avoid deadlocks is to either synchronize message 
	passing, avoid cyclic message passing, or both. We generally synchronize
	message passing between pairs of processes. 
	- MPI has the option for non-blocking send/recv, but they are generally
	more complex than we need. Runtime is trying to do this anyways. 
	- we can also create a barrier explicitly in MPI to synchronize actions but
	that creates global stalls. 

### IO and Checkpointing
- Checkpointing is a technique to add fault tolerance into computing systems. 
It basically consists of saving a snapshot of the application's state, so that 
it can restart from that point in case of failure. 
- Checkpointing requires that we do some I/O - but I/O can take a long time. 
- We find that if startup costs are amortized, writing more small files is 
better than writing fewer large files. 

### Map / Reduce
- The MapReduce framework is a parallel programming model based off of Lisp
(a functional programming language). 
- In Lisp A map takes an input function and some values and applies the
function to each value. A reduce combines all of the elements of a sequence
using a binary operator. 
- In Parallel Programming a map takes an input pair and produces a set of 
intermediate key / value pairs. The reduce function takes an input key (from
the map function) and a set of values for that key and merges them into some
smaller solution space. 
- For example we can count the number of occurences of a word in a string 
using MapReduce

```
map(String key, String value):
	for word in value:
		intermediate_map.add(word, "1")

reduce(String key, Iterator values):
	result = 0
	for count in values:
		result += int(count)
	return result 
```

- We can parallelize this model by splitting the input data into M "shards" 
each of which can be processed on different, parallel machines. This 
essentially parallelizes the map phase. 
- We can partition the reduce function by splitting the intermediate map into R
shards and processing those on different machiens. 
- This process is overseen by a master thread which manages the partitioning 
of tasks. 
- Google File System (GFS)
	- Files are divided into fixed-size chunks of 64 megabytes, similar to 
	clusters or sectors in regular file systems, which are only extremely 
	rarely overwritten, or shrunk; files are usually appended to or read. 
	- A GFS cluster consists of multiple nodes. These nodes are divided into 
	two types: one Master node and a large number of Chunkservers. 
	- Each file is divided into fixed-size chunks which are stored in the chunk
	servers. Each chunk is assigned a unique 64-bit label by the master node 
	at the time of creation, and logical mappings of files to constituent 
	chunks are maintained. Each chunk is replicated several times throughout 
	the network. 
	- The Master server does not usually store the actual chunks, but rather 
	all the metadata associated with the chunks. All this metadata is kept 
	current  by the Master server periodically receiving updates from each 
	chunk server.
	- Permissions for modifications are handled by a system of time-limited,
	expiring "leases", where the Master server grants permission to a process 
	for a finite period of time during which no other process will be granted 
	permission by the Master server to modify the chunk. The modifying 
	chunkserver, which is always the primary chunk holder, then propagates the 
	changes to the chunkservers with the backup copies. The changes are not 
	saved until all chunkservers acknowledge, thus guaranteeing the completion 
	and atomicity of the operation.
	- Programs access the chunks by first querying the Master server for the
	locations of the desired chunks. If the chunks are not being operated on, 
	the Master replies with the locations, and the program then contacts and 
	receives the data from the chunkserver directly.
	- Unlike most other file systems, GFS is not implemented in the kernel of 
	an operating system, but is instead provided as a userspace library.
