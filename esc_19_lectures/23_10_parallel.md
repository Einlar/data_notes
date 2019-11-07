# TBB
Manually mapping logical tasks onto threads is difficult, but fortunately there are libraries that do this automatically! For example, Intel's TBB Runtime Library maps **tasks** onto threads, maximizing *load balancing*, and can be used in any environment.

```cpp
tbb::parallel_for(
    tbb::blocked_range<int>(0,N,<G>), //automatically partition the big range (from 0 to N) in a set of smaller ranges
    [&](const tbb::blocked_range<int>& range) 
    { //lambda function to apply to every "blocked_range" (smaller range object => garanteed to be contiguous)
        for(int i = range.begin(); i < range.end(); ++i) {
            x[i] ++;
        }
    }, <partitioner> //function that does the partition
);
```

`parallel_for` can be also nested. It does not automatically lock, so memory needs to be manually managed.

Some observations:
- Loop size must be sufficiently high (>1M) to gain performance
- If the performance is better with a higher number of cores, the application is said to **scale strongly**
- Sometimes, adding more thread can decrease performance (overhead dominates)
- **GrainSize**: defines the *granularity* of the problem, i.e. the number of elements that each thread handles. Performance is worse if granularity is too low (too much overhead) or too high (almost no parallelizing). Usually, it is better to let TBB define the granularity automatically - optimizing load balance (avoid idle threads). However, sometimes granularity is known a-priori, and in these cases specifying it can lead to performance gain.

**Depth-first execute**: if there are multiple tasks with dependencies, execute the ones that "unlock" other tasks first (prioritize "going down through the tree"). 
Start from the first node, and explore nodes on the left, going down on the graph. When an extremum is visited, go back to the previous node, and then eventually go down again. Repeat until all nodes are explored.
