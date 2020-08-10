# Loop Parallelism

### Tools:
- forall
- Stream API

## Parallel Loop
The most general way is to think of each iteration of a parallel loop as an async task, with a finish construct encompassing all iterations. 
```java
finish {
for (p = head; p != null ; p = p.next)
    async compute(p);
}
```
However, further efficiencies can be gained by paying attention to counted-for loops for which the number of iterations is known on entry to the loop (before the loop executes its first iteration). We then learned the forall notation for expressing parallel counted-for loops, such as in the following vector addition statement (in pseudocode):
```java
forall (i : [0:n-1])
    a[i] = b[i] + c[i]
```
Java streams can be an elegant way of specifying parallel loop computations that produce a **single** output array
```java
a = IntStream.rangeClosed(0, N-1)
        .parallel()
        .toArray(i -> b[i] + c[i]);
```
Summary: Streams are a convenient notation for parallel loops with <font color=yellow> at most one output array, but the forall notation is more convenient for loops that create/update multiple output arrays</font>

## Barrrier
```java
// the HELLO’s and BYE’s from different forall iterations may be interleaved in the printed output
forall (i : [0:n-1]) {
    myId = lookup(i); // convert int to a string 
    print HELLO, myId;
    print BYE, myId;
}

// Then, we showed how inserting a barrier between the two print statements could ensure that all HELLO’s would be printed before any BYE’s.
forall (i : [0:n-1]) {
    // phase1
    myId = lookup(i); // convert int to a string 
    print HELLO, myId;
    barrier();
    // phase2
    print BYE, myId;
}
```

## Iteration Grouping
*Reduce the number of tasks created to be closer to the number of processor cores, so as to reduce the overhead of parallel execution.*
```java
// this approach creates n tasks, one per forall iteration, which is wasteful when (as is common in practice) n is much larger than the number of available processor cores
forall (i : [0:n-1]) 
    a[i] = b[i] + c[i]

// reduced the degree of parallelism from n to the number of groups
forall (g:[0:ng-1])
    for (i : mygroup(g, ng, [0:n-1]))
        a[i] = b[i] + c[i]
```

There are two well known approaches for iteration grouping: block and cyclic:  
*The former approach (block) maps consecutive iterations to the same group, whereas the latter approach (cyclic) maps iterations in the same congruence class (mod ng) to the same group.*

PCDP library provides helper methods: **forallChunked()** and **forall2dChunked()**
example:
```java
forall2dChunked(0, N - 1, 0, N - 1, (i, j) -> {
    . . . // Statements for parallel iteration (i,j)
});
```

---
Exercise:
```java
forseq2d(0, N - 1, 0, N - 1, (i, j) -> {
    C[i][j] = 0.0;
    for (int k = 0; k < N; k++) {
        C[i][j] += A[i][k] * B[k][j];
    }
});

forall2dChunked(0, N-1, 0, N-1, (i, j) ->
{
    C[i][j] = 0.0;
    for (int k = 0; k < N; k++) {
        C[i][j] += A[i][k] * B[k][j];
    }
});
```