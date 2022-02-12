# Loop Parallelism
Loop Parallelism

###### Objectives
- Create programs with loop-level parallelism using the Java Stream constructs
- Evaluate loop-level parallelism in a matrix-multiplication example
- Examine the barrier construct for parallel loops
- Evaluate parallel loops with barriers in an iterative-averaging example
- Apply the concept of iteration grouping/chunking to improve the performance of parallel loops

## Parallel Loops

The most general way is to think of each iteration of a parallel loop as an async task, with a finish construct encompassing all iterations. This approach can support general cases such as parallelization of the following pointer-chasing while loop:
```
finish {
for (p = head; p != null ; p = p.next) {
    async compute(p);
  }
}
```

## Parallel Matrix Multiplication

```
for(i : [0:n-1]) {
  for(j : [0:n-1]) { c[i][j] = 0;
    for(k : [0:n-1]) {
      c[i][j] = c[i][j] + a[i][k]*b[k][j]
    }
  }
}
```
for-i, for-j and for-k loops can be converted to forall loops, i.e., can be executed in parallel, we can see that it is safe to convert for-i and for-j into forall loops, but for-k must remain a sequential loop to avoid  data races. There are some trickier ways to also exploit parallelism in the for-k loop, but they rely on the observation that summation is algebraically associative even though it is computationally non-associative.

## Barriers in Parallel Loops

```
forall (i : [0:n-1]) {
        myId = lookup(i); // convert int to a string 
        print HELLO, myId;
        print BYE, myId;
}
```
The fact that the HELLO’s and BYE’s from different forall iterations may be interleaved in the printed output, e.g., some HELLO’s may follow some BYE’s. Then, we showed how inserting a barrier between the two print statements could ensure that all HELLO’s would be printed before any BYE’s.

Thus, barriers extend a parallel loop by dividing its execution into a sequence of phases. While it may  be possible to write a separate forall loop for each phase, it is both more convenient and more efficient to instead insert barriers in a single forall loop, e.g., we would need to create an intermediate data structure to communicate the myId values from one forall to another forall if we split the above forall into two loops. Barriers are a fundamental construct for parallel loops that are used in a majority of real-world parallel applications.

## One-Dimensional Iterative Averaging

recurrence relation x(i) = x(i-1) + x(i+1) / 2 with a boundary conditions of x(0) = 0 and x(n) = 1

The Jacobi method for solving such equations typically utilizes two arrays, oldX[] and newX[]. A naive approach to parallelizing this method would result in the following pseudocode:
```
for (iter: [0:nsteps-1]) {
  forall (i: [1:n-1]) {
    newX[i] = (oldX[i-1] + oldX[i+1]) / 2;
  }
  swap pointers newX and oldX;
}
```
Though easy to understand, this approach creates nsteps × (n − 1) tasks, which is too many. Barriers can help reduce the number of tasks created as follows:

```
forall ( i: [1:n-1]) {
  localNewX = newX; localOldX = oldX;
  for (iter: [0:nsteps-1]) {
    localNewX[i] = (localOldX[i-1] + localOldX[i+1]) / 2;
    NEXT; // Barrier
    swap pointers localNewX and localOldX;
  }
}
```

In this case, only (n − 1) tasks are created, and there are nsteps barrier (next) operations, each of which involves all (n − 1) tasks. This is a significant improvement since creating tasks is usually more expensive than performing barrier operations.

##  Iteration Grouping:  Chunking of Parallel Loops

```
forall (i : [0:n-1]) a[i] = b[i] + c[i]
```
We observed that this approach creates n tasks, one per forall iteration, which is wasteful when (as is common in practice) n is much larger than the number of available processor cores.

To  address this problem,  we  learned a common tactic used in practice that is referred to as loop chunking    or iteration grouping, and focuses on reducing the number of tasks created to be closer to the number of processor cores, so as to reduce the overhead of parallel    execution:

With iteration grouping/chunking, the parallel vector addition example above can be rewritten as follows:

```
forall (g:[0:ng-1])
  for (i : mygroup(g, ng, [0:n-1])) a[i] = b[i] + c[i]
```

> Note that we have reduced the degree of parallelism from n to the number of groups, ng, which now equals   the  number  of  iterations/tasks  in  the forall construct.

There are two well known approaches for iteration grouping: block and cyclic. The former approach (block) maps consecutive iterations to the same group, whereas the latter approach (cyclic) maps iterations in the same congruence class (mod ng) to the same group. With these concepts, you should now have a better understanding of how to execute forall loops in practice with lower overhead.
