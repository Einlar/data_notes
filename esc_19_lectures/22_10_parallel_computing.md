Moore's law was born as an empirical observation that hardware's performance doubled about every 18 months. This enabled to avoid complex refactoring of code, as simply waiting could resolve bad performances.

Of course, this trend cannot hold forever. This is because the power dissipated by a processor scale as:

$$P = QCV^2 f + V I_{\mathrm{leakage}}$$

where $Q$ is the number of transistors, $C$ is the capacity, $V$ the voltage across the gate, $f$ the clock frequency and $I$ the current. If the gate channel becomes too thin, current starts to flow through the gate, destroying the transistor function.

# Processing Unit
A processing unit does the following:
- Creates a *Program Counter* holding the address of the next instruction
- Fetches the content of memory stored in this address, and loads it into a *Instruction Register*
- The program counter is updated to point to the next instruction's address
- **Decode**: the instruction is decoded, and then **executed**

## Serial computation
In traditional computing, there is a single Processing Unit (PU), and instructions are executed sequentially, i.e. one after the other.

## Parallel computation
If two instructions have no dependency, meaning one does not need the result of the other, they can be (in principle) executed at the same time, by two different PUs.

However this has some limitations: all PUs need to access data which is usually stored in the same place. There are several way to reduce this problem:
- Reuse data and instructions (store them in a temporary place, a **cache**)
- Increase transfer speed
- Increase amount of data to transfer, so that the overhead in each transfer becomes (relatively) less significant
- Use a pattern for memory access, so that different memory locations can be accessed by a single memory transfer (e.g. they are contiguous).

# Terminology
- **Granularity**: size of tasks
- **Scheduling**: order of assignment of tasks (to be executed)
- **Mapping**: assignment of tasks to a PU
- **Load balancing**: the *art* of making all computations end at the same time, so all PUs are available for the *next* instruction
- **Barrier**: a checkpoint, so that all PUs need to have finished calculations to start the next one
- **Speedup**: how much the parallel application is faster than the serial analogue
- **Efficiency**: Speedup per PU
- **Race condition**: (bad) situation when the result of execution depends on sequence/timing of events.
- **Critical section**: only one worker per time can enter

## Flynn's Taxonomy
- **SISD** Single Instruction stream - Single Data stream (basic serial execution)
- **SIMD** Single instruction stream - Multiple data stream (apply the same operation to independent data)
- **MISD** Multiple instruction stream - Single data stream
(different instructions performed on common data)
- **MIMD**: Different processors that act differently on different data
- **SIMT**: Single Instruction - Multiple Thread (SIMD + multithreading, as in GPUs)

# Parallel operations
## Reduce
A big input is *combined* in a single/low-dimensional output (e.g. sum of elements in a vector).
The task can be divided on different PUs, and then all results are combined at the end.

Example: compute the number of 5s in an array. Serial execution:
```cpp
array[M]
numberOf5 = 0
for i in [0,N[:
    if array[i] == 5
        numberOf5++
return numberOf5
```
Parallel execution:
```cpp
numberOf5 = 0
nWorkers = 4 //smaller or equal than number of parallel PUs
count5(array, workerId):
    beg = workderId*N/nWorkers //each worker starts at a different point (first at the start, second at 25%, third at 50%...)
    end = beg + N/nWorkers //and goes on for a fraction of the total array
    for i in [beg,end[:
        if array[i] == 5:
            numberOf5++
```

**Problem**: the same variable `numberOf5` is shared between all workers. This could lead to a simultaneous access, and thus data corruption. For example a worker may read a `3`, and attempts to write a `4`, but before it is allowed to do that, another thread writes a `4`. And so the final number will be `4`, skipping one count.

One possible solution is to have a **critical section**, meaning that can be accessed only by one worker at a time:
```cpp
count5(array, workerId):
    beg = ...
    end = ...
    for i in [beg,end[:
        if array[i] == 5
        lock() //prevents more than one worker to enter there
        numberOf5++
        unlock()
```
This introduces overhead, and at the end results in a bad performance, even worse than that of the serial algorithm!

Several solutions:
- **Privation**: each worker uses a personal variable to count 5s, and at the end the results are combined
- **Transformation of the access pattern**: compute locally most of the operations, and use `lock()` very sparingly

Now performance is a bit better (there is still overhead). However, doubling the number of PUs does not half the time.

# Amdahl's Law
The maximum theoretical throughput is limited by Amdahl's Law. Note that every program has a serial part, that is executed by a **single** PU (by definition).
The speedup as a function of the number $p$ of PUs is then:
$$ S(p) = \frac{T_s}{T_p} $$
where $T_s$ is the time needed for the serial application, and $T_p$ for the parallel one.

However, the speedup is only on the parallel part:
$$ T_p = f T_s + (1-f) T_p = fT_s + \frac{(1-f)t}{p} $$
where $f$ is the fraction of time spent on the serial part. <br/>
Then, by rearranging:
$$ S(p,f) = \frac{T_s}{fT_s + \frac{(1-f)t}{p}} \to \frac{1}{f} = S_{\max}(f) $$
which means that the maximum performance is capped!

Fortunately, note that increasing the size of the problem does not change the time spent executing the sequential part, and only affects the parallel portion. So, let $f(n)$ be the sequential code fraction of the program, with $n$ the number of lines executed sequentially. If we increase the dataset's size, i.e. $n$, then $f(n) \to 0$, so that:
$$ S(n) = f(n) + p[1-f(n)] \Rightarrow S_{\max} \equiv \lim_{n\to \infty} S(n) = p$$
So, in the limit of huge datasets, parallelization is efficient! (Gustafson's Law)

Read Concurrent Containers presentation


# Threads
A **thread** is an execution context, i.e. a set of register values that defines:
- which instructions have to be executed
- the order of execution
Running a thread means:
- fetching execution context
- start running the instructions
To execute another thread:
- save all data of previous thread
- switch to the new context (expensive operation)

**Concurrent** means that operations are passed between two threads (does not imply that they are executed simultaneously, i.e. in parallel).



```cpp
for (int i = 0; i < n; ++i) {
    //emplace_back => call constructor of f on i and add the resulting object to the vector
    v.emplace_back(f, i); //executes asynchronously (takes no time, even if f(i) takes 10s)
}
```