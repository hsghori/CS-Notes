# High performance computer architecture notes

Computer architecture is the idea that a computer should be built in the best way to fulfill its main purpose.
For example, a PC should be optimized for "regular" use whereas a supercomputer should be more optimized towards
faster processing and parallelism. We need good computer architecture to improve performance (which can be measured
in many different ways) and to improve the abilities of the computer.

Improvising performance
- Speed
- Battery life
- Weight / size
- Energy efficiency

Improving abilities
- 3D graphics
- Debugging support
- Security

We can design better computers by using advances in circuit design and microbfabrication technology and use them
to design better computers. Thus, computer architecture is very tied to current technology trends, and computer
architects need to keep an eye on the latest in computer technology to design architectures that are not obsolete.

## Measuring performance

### Moore's Law
__Moore's law__ states that every $1.5$ to $2$ years the number of transistors on a chip (processor) will double. This
roughly corresponds to
- the processor speed doubles every two years
- energy used per operation is halved every two years
- memory capacity is doubled every two years

However, Moore's law has some consequences. While processor speed and memory capacity double every two years
(roughly), __memory latency__ (the time to read / write to and from memory) increases at a much slower rate -
approximately 1.1x every two years. This problem is called the memory wall because while the processors have
been getting faster, they are limited by the memory wall and therefore the net speed does not increase as fast.
We solve this problem using caching layers between the processor and main memory which mitigate the problem.

### Performance metrics
When we talk about performance we usually talk about processor speed but even that is not completely accurate.
- Latency is how long it takes a task be be completed
- Throughput is the number of tasks that can be completed in a time period (usually in one second).

Throughput and latency naively have an inverse relationship
$$t = 1 / L$$
But that's not necessarily true since a task can be made up of an assembly line of sub tasks which can be
completed in parallel.

Ie if the latency of a task is four seconds but it is divided into 20 equivalent steps which are completed in
an assembly line then the throughput per second is 5.

We compare performance using __speedup__. The speedup $N$ of system $X$ vs system $Y$ is defined as

$$N = \frac{speed(X)}{speed(Y)}$$

This $speed$ function can be defined in terms of latency or throughput.
- For the latency case, $speed \prop \frac{1}{throughput}$ so $N = \frac{latency(Y)}{latency(X)}$
- For the throughput case, $speed \prop throughput$. So $N = \frac{throughput(X)}{throughput(Y)}$

Obviously $N > 1$ means better performance (better execution time or better throughput) whereas $N < 1$ means worse performance (worse execution time or worse throughput).

We can naively measure performance as $\frac{1}{execution \ time}$. However, this raises a lot of questions:
- What workload are you measuring on
- Does this necessarily apply to other users
- How do we get the workload data

### Benchmarks
We use __benchmarks__ to measure performance instead. Benchmarks are programs or input data that users /
companies have agreed upon for performance measurements. Usually we measure performance using a __benchmark suite__
as opposed to testing with a single benchmark program. A benchmark suite is a set of benchmark programs.

Benchmark types include
- Real applications\
  These are the most representative of performance measurements but they are the hardest to set
  up. These are good for products ready for development.
- Kernels\
  The most time consuming part of an application (eg loops). This way we don't have to set up a whole
  application but rather we set up a bunch of simpler programs. These are good to test prototypes.
- Synthetic benchmarks\
  Similar to kernels but they don't use real application code so they're easier to compile. These are good for
  design studies.
- Peak performance\
  The maximum number of instructions / second the machine can execute. These are good for marketing but not really
  useful otherwise.

Benchmark suites are decided by organizations and generally standardized
- TPC is used for databases, data mining, and other transaction processing
- EEMBC is used for embedded systems.
- SPEC is used to evaluate workstations and raw processors.

To evaluate performance on a benchmarking suite we cannot simply take the average speedup
since those are not compatible.

We can either take the mean of the execution times of each benchmark and use those to calculate the average speedup.
Or we use the geometric mean of the speedup which (for a vector of values $X$ where $|X|=N$) is defined as

$$g = \sqrt[N]{\pi_{i=0}^{N}{(X_i)}}$$

### Calculating performance
The __iron law of performance__ defines the processor or CPU time as:

$$t_{cpu} = instr * cpi * cct$$

Where $cpi$ is the number of cycles executed per instruction and $cct$ is the time it takes to execute one clock
cycle. Looking at CPU time this way allows us to break down and evaluate architectures and programs in different ways.
- The number of instructions in the program is defined by the algorithm, compiler, and instruction set of the processor.
- The cycles per instruction is defined by the instruction set and the processor design.
- The cycle clock time is defined by the processor design, circuit design, and transistor physics.

Note that processor speeds are usually defined in Ghz ($10^9$ cycles / second) so we need to find the inverse
to get the clock cycle time.

Cycles per instruction may differ by instruction type so we can generalize this a bit to say

$$t_{cpy} = \sum_{j}{(instr_j * cpi_j)} * clock-cycle-time$$

__Amdahl's law__ is used to define the speedup when we are speeding up a specific part of a program.

$$Speedup(P, N) = \frac{1}{1-P)+\frac{P}{N}}$$

Where $P$ is the proportion or fraction of the execution time affected by the speedup and $N$ is the speedup of that
enhanced portion (relative to it's unenhanced version). Amdahl's law shows that it is better to have a small speedup
on a large portion of the program than to have large speedup on a small portion. We can see that:

$$\lim_{n \to \infty}{Speedup(0.1, n)} = \frac{1}{0.9 + \frac{0.1}{\infty}} = 1.111$$

which is not particularly impressive. If you put significant effort into speeding up a small portion of the program
you will not have a big effect.

__Lahdma's law__ says that while Amdahl's law says to focus on the "common case", we should not sacrifice the
performance of the "special case". If you significantly reduce the performance of the special case, you will not
see a good speedup.

Amdahl's law also shows that we get diminishing returns if we keep speeding up the same part of the program. This
is because the execution time of the sped up part keeps decreasing. Additionally it is much more difficult to keep
speeding up the same part.

## Pipelining

__Pipelining__ is the idea of using an assembly line to process instructions in a processor to increase the
throughput of the system. While the latency of every individual task may be long, the throughput is much higher
than a naive "one task at a time" approach.

In the simplest program we have
1. A program counter (PC) which looks up the next instruction from memory (IMEM).
2. The instruction is decoded and (while that is happening) we read from the necessary registers.
3. The instruction and values in the registers are passed to the Arithmetic Logic Unit (ALU) to do the actual
   computation.
4. The result from the ALU could be simply stored back in the register or, if the instruction was something like
   a load or store, the result from the ALU is the address in data memory (DMEM) whose value is passed to the
   registers.

![](src/hpca/pipeline-arch.png){width=400px}

These steps can be broken down into a pipeline:
1. Fetch the instruction
2. Read from registers and decode the instruction
3. Do the computation
4. Read from data memory
5. Write to registers

It is easy to see how this greatly improves the throughput of the processor. If the latency of one instruction
being completed is, for example, 20ns, and each of the five steps takes roughly the same amount of time we can see
that after the first instruction has left the pipeline, we will complete one instruction every 4ns as opposed to
every 20ns.

### Pipeline stalls

We may think that the CPI for each step in the pipeline would be 1, but that is not necessarily the case.
The initial instruction adds a bit of latency though that is negligible since the processor is handling billions of
instructions. A much more important concern are __pipelining stalls__, issues in one part of the pipeline which
delay everything behind (and including) that point. These stalls increase the CPI for the pipeline.

Processor pipelining stalls may not be due to an issue with the pipeline itself but rather the instruction order.
Consider the following instructions

```MIPS
LW  R1, 5     // (I1) loads the value 5 into R1
ADD R2, R1, 1 // (I2) adds 1 to R1 and stores it to R2
ADD R3, R2, 1 // (I3) adds 1 to the value in R2 stores it in R3
```

Which are placed in the pipeline as follows

F    |  R / D    |  ALU   |  DMEM   |  W
-----|-----------|--------|---------|-------
I3   | I2        | I1     |         |
I3   | I2        |        | I1      |
I3   | I2        |        |         | I1
I4   | I3        | I2     |         |


If I2 is allowed to pass into the ALU before I1 has reached the W stage, the result of I2 will be incorrect which
will in turn mess up the results of the rest of the program. We see that this causes a stall of 2 cycles (assuming
we can read and write in a single cycle).

Pipelines may also need to be __flushed__. If a `JUMP` instruction is fetched, the processor won't know
which instruction to fetch next since we won't compute the actual address to jump to until the `JUMP` has
reached the ALU. So the processor will attempt to predict which address in IMEM the PC should jump to based on a
__branch predictor__ algorithm. So it will pass the instructions to the pipeline from the predicted branch. If
that branch was correct the processor will continue with the pipeline as normal. But if it is wrong, the
instructions before the ALU stage must be flushed from the pipeline which causes a stall.

F    |  R / D    |  ALU   |  DMEM   |  W
-----|-----------|--------|---------|-------
JUMP | I2        | I1     |         |
B1_1 | JUMP      | I2     | I1      |
B1_2 | B1_1      | JUMP   | I2      | I1
B2_1 |           |        | JUMP    | I2

Branch and jump instructions cause __control dependencies__ that is any instructions who's execution is controlled
by the result of a branch or jump have a control dependence on that instruction

```MIPS
main:
    ADD R3, R2, R1
    BEQ R1, R3, label  // jump to label if R1 == R3
    ADD R2, R3, R4
    SUB R5, R6, R8

label:
    MUL R5, R6, R8
    // ...
```

All the instructions after the `BEQ` (and everything in the `label` subroutine) have a control dependence on
the `BEQ` instruction.

The impact on CPI due to control dependencies can be calculated as:

$$CPI = CPI_{orig} + p*s$$

where $p$ is the percentage of all the instructions that are incorrect branch instructions and $s$ is the
number of cycles lost to a branch misprediction. In our example above, $s=2$ since we lose 2 pipeline stages
(2 cycles) to a misprediction.

### Data dependencies
Pipeline stalls can also occur due to data dependencies. Consider the following program

```MIPS
ADD R1, R2, R3  // I1
SUB R7, R1, R8  // I2
MUL R1, R5, R6  // I3
```

There are three dependencies in this simple program:
1. I2 has a data dependency on I1 since we need to know the correct value of R1 to complete I2. This
   type of dependency is called a Read after Write (RAW), flow, or true dependence.
2. There is a dependency between I1 and I3 since I1 must be completed before I3 since we want the result of I3 to be
   the final value in R1. This is called a Write after Write (WAW) dependence.
3. There is a dependency between I2 and I3 since I2 needs to finish reading from R1 before I3 modifies it. This is
   called a Write after Read (WAR) dependence. This is also called an anti-dependence.

RAW dependencies are called true dependencies whereas WAR and WAW are called false dependencies.

Whether or not a dependency causes a stall depends on the pipeline architecture. In our example pipeline, the WAW
dependence doesn't matter since writes always happen sequentially. WAR dependencies are also non-consequential
for the same reason. However, RAW dependencies do cause a problem. A __hazard__ is when a dependence causes a
problem in the processing pipeline. In out pipeline, true dependencies are hazards if there are less than 3
instructions between the instructions.

To handle hazards we need to detect the hazard and either:
1. flush the dependent instructions
2. stall the dependent instructions
3. fix values read by the dependent instructions

We need to flush for control dependencies but data dependencies can be handled by stalls and __forwarding__.
Forwarding is the idea that if a value that causes a data dependency exists in the pipeline we can "forward" it
to the dependent instructions to prevent stalling.

```MIPS
ADD R1, R2, R3  // I1
SUB R4, R5, R1  // I2
```

once I1 has passed the ALU, the value in R1 can be forwarded to I2 when it reaches the ALU stage allowing I2 to
compute the correct value without a stall. This is not always an option if I2 needs the value in R1 before I2
has computed that value - in that case we have no choice but to stall. We can also use a combination of stalling
and forwarding to increase efficiency.

### Pipeline architecture
Increasing the number of stages in a pipeline has interesting effects. This increase can increase the chance and
penalty of hazards which increases the CPI. However, this can also decrease the work per stage which
decreases the clock cycle time (since each stage / cycle is doing less work). Based on the iron law of performance:

$$t_{cpu} = instructions * cycles-per-instruction * clock-cycle-time$$

We can see that increasing CPI and decreasing clock cycle time have contradictory effects on execution time so we
would think that the goal is to find the pipeline architecture that maximizes performance. However, we also must
consider power consumption. As the clock frequency increases, the processor uses more power - so increasing the
number of stages also increases the power consumption of the processor. So the actual goal is to find the pipeline
architecture that maximizes performance and minimizes power.

![](src/hpca/pipeline-opt.png){width=400px}

## Control hazards - Branch prediction and predication

Branches and jumps are particularly expensive operations since they cause control hazards that waste many cycles.
Therefore we need to design mechanisms to mitigate the pipeline stalls caused by these operations.

A branch instruction is of the form

```MIPS
BEQ R1, R2, label
```

When the instruction is fetched, the pipeline does not even know it is a branch. So a naive implementation of a
pipeline would always predict that the branch is not taken and would have a 50% accuracy. So a branch predictor
algorithm must take in only the PC of the next instruction to fetch and decide
1. whether or not the instruction is a taken branch
2. if the branch is taken, what is the target PC.

The simplest branch predictor is to simply predict that the branch is never taken. In this case:
- If the next instruction is a branch you either spend 1 or 3 cycles.
- If the next instruction is not a branch you spend 1 cycle.

A naive "branch is not taken" predictor has an approximately 88% accuracy. However, we can use the equation
$CPI = CPI_{orig} + p*s$ to see that improving the branch prediction accuracy can generate huge speedups. Especially
for deeper pipelines with multiple instructions per cycle. We can also easily see that for deeper pipelines with
multiple instructions per cycle, branch misprediction wastes a lot of resources.

Since it seems like $PC_{next} = f(PC_{curr})$ it is almost impossible to actually make any predictions about
whether or not the instruction is branch let alone if we should take said branch. However, that is not the whole
story. A branch predictor also knows the history of how the branches have behaved in the past and can use that
history to make predictions.

### Branch prediction algorithms

#### Branch target buffer
The __branch target buffer__ (BTB) is the simplest branch predictor algorithm that uses history to make branch
predictions. It keeps a lookup table which maps $PC_{curr} \to PC_{next}$. At fetch, it will use that lookup table
to decide the next PC, carry that prediction through the pipeline, and update the lookup table in case of a
misprediction. BTB can only work if the table is small and the mapping function is easy to compute
since it needs to have a one cycle latency. The mapping function uses the 10 least significant bits of the
$PC_{curr}$ (that are different among different instructions) as the index for its entry in the table. This is very
easy to compute and allows instructions close to not override each other in the lookup table.

#### N-bit predictors
We do __direction prediction__, predicting whether or not to use the BTB, using a very similar method. We use the
least significant relevant bits to index into a __branch history table__ (BHT) which stores whether or not we
predict that the instruction at the PC is a taken branch (1) or not (0). If the instruction is a taken branch we
use the BTB to find the index of the next instruction. Otherwise we can simply iterate the PC by one. This table
acts as a filter for the BTB. If we see that an instruction is a taken branch, we update its entry in the BHT and
the BTB but if the instruction is not a taken branch we just set its value in the BHT to 0. This allows us to limit
the BTB to only correspond to taken branches.

This __one bit prediction__ model works well for situations where branches are (almost) always taken or not taken.
However, it is not good at branches which have an unextreme taken vs not taken ratio (ie if the branch is taken 60%
of the time). Observe that each "anomaly" (ie misprediction) results in 2 mispredictions (the initial misprediction
and the misprediction to "fix" the initial misprediction). The one bit predictor also does not work well on short
loops (particularly when they are nested within other loops).

These issues can be alleviated using a _two bit predictor_. A two bit predictor stores 2 bits in the BHT (so the
possible values are 00, 01, 10, and 11) where the most significant bit signifies whether or not the instruction is
a taken branch.

Value | Meaning
------|------------------
00    | strong not taken
01    | weak not taken
10    | weak taken
11    | weak not taken

When we are in a strong state, if we mispredict we change the less significant bit and move to the weak state. If
we are in the weak state, if we mispredict we move to the weak inverse state. If we predict correctly in the weak
state we move to the strong state. We can see that a single anomaly causes one misprediction (not two).

The values in the 2 bit BHT should start in one of the weak states since, generally taken branches are more common
than not taken and starting in a weak state has minimal mispredictions if we guess the initial state incorrectly.
However, starting in the weak state does have a worse worst case scenario if the branch alternates.

Increasing the number of bits in the counter is not usually worth the size cost. Increasing the number of bits in
the predictor (beyond 2) is most advantageous when anomalous events happen in streaks. But this is reasonably
uncommon so we don't usually want to do it.

#### History based predictors
We can see that N-bit predictors are not good at predicting alternating patters. A __history based predictor__ is a
predictor that attempts to learn these types of patterns. History based predictors look at the "history" of the
branch (whether or not the instruction was a taken branch the last time it saw the same sequence of N resolutions).

The BHT of a history based predictor stores a $(history, prediction_0, prediction_1, ..., )$ where each prediction
is a two bit counter. When the row of the BHT is called, the history is used to index into the tuple to choose the
prediction.

If the history (the last outcome) was "taken" (or 1), the $prediction_1$ will be used as in the two bit predictor.
We then update $predcition_1$ based on whether or not the branch was correctly predicted and update the history.
Note that this example history is 1 bit (taken vs not taken) but we can have higher bit histories, we just need to
increase the size of the BHT by 2 bits for every 1 bit increase in the history. For example, a 2 bit history BHT
will have a 2 bit history index and 4 prediction values.

```
       BHT                                           BTB or iterate by one (depending on onutput of BHT)
PC -> [ history | counter 00 | counter 01 | ... ] -> next PC

```

An N-bit history predictor can predict patterns where $length \leq N + 1$. However, each entry costs $history \
size * counter \ size * 2^{history \ size}$. Since we use 2 bit counters this can be generalized to:

$$cost_{single \ entry} = N + 2^{N + 1}$$

We can see that the total cost is equal to:

$$\Sigma cost = 2^{N} * (N+2^{N+1})$$

where M is the number of bits we are using from the PC to index into the PHC.

N-bit predictors also waste a lot of memory. An N-bit pattern can only have N possible unique histories
meaning it only needs N counters. However, the algorithm actually allocates $2^N$ counters for the pattern.

#### PShare and GShare predictors
We can use __shared counters__ to mitigate the waste of an N-bit history predictor. In the shared counters model,
the PC indexes into a __pattern history table__ (PHT) which stores the history for that PC. the output of the
PHT is combined with the PC in some way to attempt to create a relatively unique index. This index is used to
index into a BHT with only one counter (which should correspond to the PC / history combination). The BHT
value determines the prediction and the history and BHT counter are updated when the branch is resolved.

```
       PHT                             BHT            BTB or iterate by one depending on output from BHT
PC -> [ history ] - PC XOR history -> [ counter ] -> next PC
```

This is not a collision proof system, but if we have a relatively large BHT we can mitigate that issue and waste
much less space than the standard history predictor.

If we index using the M least significant bits of the PC and are using N bits of history we can see that we
need a PHT of $2^M * N$ bits and a BHT of $2^{max(M, N) + 1}$ bits (the output of the XOR will have the size
of the larger of the PC and history). So the total cost is:

$$\sigma cost = N*2^M + 2^{max(M, N) + 1}$$

which is much less than that of the standard history based predictor. This type of predictor is called a
__pshare predictor__ or a private history predictor. A pshare predictor will store a history for every
PC. A __gshare predictor__ is a predictor that uses a single N-bit global history which is XOR-ed with the PC
to index into the BHT. Both pshare and gshare use shared counters.

Pshare predictors are good when a branch's previous behavior is predictive of its future behavior (ie if the
resolution of a branch is only dependent on previous resolutions of the branch). This means pshare predictors
work well with loops, alternating behaviors, etc.

Gshare predictors are good when a branch's future behavior is dependent on that of other branches. For example,
branches with contradictory or linked conditions. Gshare generally needs a larger history than pshare because
it needs to combine more elements into the global history to get accurate predictions.

#### Tournament and hierarchal predictors
Since pshare and gshare predictors work better on different branch types, we want to incorporate both into
our branch predictor.

A __tournament predictor__ is a predictor that attempts to choose between pshare and gshare for each branch. The
way this works is by maintaining separate gshare and a pshare predictors as well as a __meta-predictor__ which is a
table indexed using the PC which maintains a 2 bit counter for each PC. This counter determines whether, for each
PC, we should trust the gshare or pshare predictor.

```
PC -> gshare -------> | select between    | ---> next pc
├---> pshare -------> | gshare and pshare |
└---> meta-predictor -----^
```

The gshare and pshare predictors are "trained" by iterating their counters as usual and the meta predictor is
trained using which prediction was correct a standard 2 bit direction predictor. 00 corresponds to strongly gshare
and 11 corresponds to strongly pshare - so when gshare is right, decrement the counter and if the pshare is
right increment the counter. If they are both correct or incorrect the meta-prediction does not change.

Tournament predictors work quite well but that are resource intensive since we must maintain two large, complex
predictors. __Hierarchal predictors__ attempt to solve that challenge by maintaining one large, complex predictor
(which we call the good predictor) and one simple, low resource predictor (which we call the ok predictor). For
example we may keep one pshare predictor (good) and one 2 bit counter predictor (ok). We update the ok predictor
on every branch decision since that is very cheap but only update the good predictor when the ok predictor does
not predict the specific PC's branch resolution well. Thus we can limit the number of entries required for the
good predictor which allows us to create more complex, better predictors for harder-to-predict branches and
use a lower resource predictor for the majority of branches (which are simple). Both hierarchal predictors
and tournament predictors can have more than 2 predictors.

In a hierarchal predictor the complex predictors will maintain a "tag" array which is indexed using the PC. This
predictor indicates whether or not the PC should be predicted by the complex predictor. So if the ok predictor
consistently mispredicts a specific branch, it will be added to the complex predictor and tagged so that prediction
is used. If the hierarchal predictor has more than 2 predictors, each complex predictor will maintain its own
tag array and there will be a set hierarchy of which predictions are most valuable (more complex predictions should
be used if they exist).

#### Return address stack predictors
So far we've dealt with branches that have a binary target - either the PC is iterated by oen or it is a set
value defined by the instruction. However, function return instructions do not work that way because a function
can be called from any location and therefore the target is not predictable by the BTB. To deal with this issue
we maintain a __return address stack__ (RAS) which is a very small stack data structure that stores the
return address of the function. So when a function is called, the address that the function should return to
is pushed onto the stack and when the return statement is hit, the PC should be set to the address popped from
the stack.

The RAS is much smaller than the program's overall stack memory and it can get full. When the RAS gets full we
want to wrap around the stack (overriding previous entries). Wrapping around is a better approach than just not
pushing to the RAS because programmers generally compose functions of a lot of nested functions so wrapping around
will allow for correct prediction of more overall function (most of the small nested functions) returns even though
the parent function returns will be mispredicted.

We determine whether or not an instruction is a return in one of two ways. The first is to train a binary predictor
to determine whether or not an instruction is a return. This can be acomplished with a single bit predictor since
we only need to see the instruction once to know if it is a return or not. The second is through __predecoding__.
The processor maintains a cache of instructions and, during the fetch process, will decode some of the function
to determine whether the instruction is already in the cache or it needs to be fetched fully. So we can add logic
to also determine whether the instruction is a return, store that in the cache (in one bit) alongside the
instruction, and use that to determine whether or not to use the RAS. Predecoding is generally used to determine
if the instruction is a branch or return and can be used to determine the size of the instruction (and potentially
also fetch the next instruction if there is space in the cache).

### Predication
__Predication__ is another means of avoiding control hazards. Instead of making a prediction as to the next
PC (whether or not to take the branch), when we predicate we do both branches' work and throw away the incorrect
path. So when we predicate we always waste ~50\% of the instructions (assuming both paths have similar sizes). So
the cost of predication is always $0.5 * \Sigma ic_{branch}$ whereas the cost of branch misprediction is
`instructions per stage * stages before branch resolution`.

Obviously predication is not usually better than prediction. Loops, function calls, and large if / else
statements are better served by prediction since the cost of misprediction in these cases is usually less than
that of predication. However, small if / else statements may be better served by predication depending on the
number instructions in the branches and the branch predictor accuracy.

Predication is built off of a compiler concept called __if conversion__. Consider the following if statement:

```C
if (condition) {
    x = arr[i];
    y++;
} else {
    x = arr[j];
    y--;
}
```

The compiler would convert this code into something like

```C
x1 = arr[i];
x2 = arr[j];
y1 = y + 1;
y2 = y - 1;
x = (condition) ? x1 : x2;
y = (condition) ? y1 : y2;
```

The ternary operation (`x = (condition) ? x1 : x2 `) is handled using a __conditional move__ instruction. MIPS has
`MOVZ` and `MOVN` instructions defined as:

```MIPS
MOVZ Rd Rs Rt  // Rd = (Rt == 0) ? Rs : Rd;
MOVN Rd Rs Rt  // Rd = (Rt != 0) ? Rs : Rd;
````

So in the case above we could convert the C code into something like:

```MIPS
MOV R3, cond
MOV R1, x1
MOV R2, x2
MOVZ x, R1, R3
MOVN x, R2, R3

MOV R1, y1
MOV R2, y2
MOVZ y, R1, R3
MOVN y, R2, R3
```

x86 has a much larger set of conditional move oerators (`CMOVZ`, `CMOVNZ`, `CMOVGT`, etc).

If conversion requires a compiler with the correct support. It will remove conditional branches that are hard to
predict but comes at the cost of using more registers and requires that we use additional `movc` instructions
to select the correct result. We can alleviate these issues using __full predication__, the process of converting
every instruction in the ISA into a `movc` instruction. This requires extensive compiler support.

An example of full predication is the intel itanium processor. Each instruction has an extra 6 bits (the least
significant six bits) which contain the __qualifying predicate__. The qualifying predicate tells the processor
whether or not to store the result of the instruction in the destination register. In this processor, the
qualifying predicate is a pointer to a set of conditional registers (64 registers) which store the conditions.

In general we can use the following equation to figure out whether or not we should use branch prediction or
predication for a specific branch. We can solve this problem using the equations we have already used for
branch prediction.

$$cost_{branch\ prediction} = base + P(mispredict) * penalty$$

Note that $base$ and $penalty$ should be the same units, usually $cpb_{avg}$ and $penalty_{cycles}$ or $ipb_{avg}$
and $penalty_{instr}$. We can easily figure out the number of cycles or instructions for a predicated branch by
simply counting the number of instructions in the converted code and converting that to cycles if need be.
So $cost_{branch\ predication}$ is a constant and we can easily solve the linear equation:

$$c_{branch\ predication} < c_{branch\ prediction}$$

to determine which strategy to choose.

## Data Hazards
Branch prediction and predication are great at eliminating control hazards. However, data dependencies can cause
hazards which can stall the pipeline.

### Instruction level parallelism (ILP)
One obvious way of speeding up our execution is by parallelizing our execution pipeline. However, if we have
two instructions `I1` and `I2` where there is a RAW dependency from `I1 -> I2`, these instructions cannot
be executed in the same cycle. So we see than an ideal processor that can parallelize an infinite number of
instructions will have a CPI from 0 to 1 where the CPI is close to 0 when the instructions are independent and
the CPI is close to 1 if the instructions are dependent on each other.

We can see that if a pipeline has forwarding, if instructions `I1` and `I2` have a RAW dependency, `I2` must be
stalled by at least one cycle. Similarly, if `I1` and `I2` have a WAW dependency, `I2` must be stalled by at
least one cycle (before the write stage) so that the value in `I2` is the final value in the register. Recall
that RAW dependencies are also called __true__ dependencies whereas WAR and WAW dependencies are __false__ or
__name__ dependencies. This is because RAW dependencies are fundamental to the program whereas WAR and WAW
dependencies are more coincidental (we are using the same register for two unrelated actions). So we can
increase our ILP potential by removing false dependencies from the program.

We can use __register renaming__ to remove false dependencies from our programs. Processors divide registers into
__architectural registers__ which are used by the compiler and processor and __physical registers__ which is
everywhere a value can be stored. In register renaming, during the decode stage, the processor will rename the
architecture registers to physical registers and uses the __register allocation table__ (RAT) which maps
architectural registers to the physical location where their value is stored.

Let's say we have a program

```MIPS
ADD R1, R2, R3
SUB R4, R1, R5
XOR R6, R7, R8
MUL R5, R8, R9
ADD R4, R8, R9
```

The processor will initially maintain a RAT like:

Arch register | Physical register
--------------|-------------------
0             | P0
1             | P1
2             | P2
3             | P3
...           | ...

When we write to a register (as in `I1`) we rename the destination register in the RAT. So `ADD R1, R2, R3`
becomes `ADD P10, P2, P3` and the RAT becomes:

Arch register | Physical register
--------------|-------------------
0             | P0
1             | P10
2             | P2
3             | P3
...           | ...

This effectively removes WAW and WAR dependencies since different registers are being used to write the values.

__Instruction level parallelism__ (ILP) is the IPC of a program under the assumptions that
1. the processor can execute the entire pipeline in 1 cycle
2. the processor can execute any number of instructions in parallel
3. the processor must obey true dependencies

ILP is a property of an individual program and has nothing to do with the architecture it is being run on.
To determine the ILP of a program, we first rename the registers then calculate the IPC of the program under the
ideal processor assumptions.

When computing ILP we assume that the processor has perfect branch prediction and no structural dependencies. This
means that we can actually treat branch instructions as No-Op instructions and treat the branch body itself as a
regular instruction (and calculate its cycle solely based on its data dependencies).

The IPC of a program on a specific processor is always less than or equal to the ILP of the program. When looking
at IPC we often care about the "issue" of the processor (how many instructions can be issued per cycle), the
order of the processor (whether or not instructions can be executed out of order), and the ALU limitations.
If a processor is narrow issue and in order, the limiting factor is usually the narrow issue but if the processor
is wide issue and in order, the order limits the IPC. So if we have a wide issue processor we should make sure
it is out of order to benefit from the issue. 

