# Task Parallelism
Task Parallelism

###### Objectives
- Task parallelism using Asynkc/Finish constructs
- Create task-parallel programs using Java's Fork/Join Framework
- Interpret Computation Graph abstraction for task-parallel programs
- Evaluate the Multiprocessor Scheduling problem using Computation Graphs
- Assess sequetional bottlenecks using Amdahl's Law

## Task Creation and Termination (Async, Finish)

We can use Async notation for task creation: “async ⟨stmt1⟩”, causes the parent task (i.e., the task executing the async statement) to create a new child task to execute the body of the async, ⟨stmt1⟩, asynchronously (i.e., before, after, or in parallel) with the remainder of the parent task. 
We can use Finish notation for task termination: “finish ⟨stmt2⟩” causes the parent task to execute ⟨stmt2⟩, and then wait until ⟨stmt2⟩ and all async tasks created within ⟨stmt2⟩ have completed. Async and finish constructs may be arbitrarily nested.

The array-sum example:
```
finish {
  async S1; // asynchronously compute sum of the lower half of the array
  S2;       // compute sum of the upper half of the array in parallel with S1
}
S3; // combine the two partial sums after both S1 and S2 have finished
```

## Creating Tasks in Java's Fork/Join Framework

we can also use async and finish functionality using Java’s standard Fork/Join (FJ) framework. In this framework, a task can be specified in the compute() method of a user-defined class that extends the standard **RecursiveAction** class e.g. (L) in the FJ framework.
The method L.fork(), creates a new task that executes L’s compute() method.
The call to L.join() then waits until the computation created by L.fork() has completed

>Note that join() is a lower-level primitive than finish because join() waits for a specific task, whereas finish implicitly waits for all tasks created in its scope. To implement the finish construct using join() operations, you have to be sure to call join() on every task created in the finish scope.

```
private static class ASum extends RecursiveAction {
  int[] A; // input array
  int LO, HI; // subrange
  int SUM; // return value
  . . .
  @Override
  protected void compute() {
    SUM = 0;
    for (int i = LO; i <= HI; i++) SUM += A[i];
  } // compute()
}
```

FJ tasks are executed in a ForkJoinPool, which is a pool of Java threads. This pool supports the invokeAll() method that combines both the fork and join operations by executing a set of tasks in parallel, and waiting for their completion. For example, ForkJoinTask.invokeAll(left,right) implicitly performs fork() operations on left and right, followed by join() operations on both objects.


## Computation Graphs, Work, Span, Ideal Parallelism

![image](https://user-images.githubusercontent.com/7610065/153711214-8ba8909f-e930-43af-b410-effb34fbe461.png)
- Computation Graphs (CGs)
  - A set of vertices or nodes, in which each node represents a step consisting of an arbitrary sequential computation. 
  - A set of directed edges that represent ordering constraints among steps.

For fork–join programs, it is useful to partition the edges into three cases:
- Continue edges that capture sequencing of steps within a task.
- Fork edges that connect a fork operation to the first step of child tasks.
- Join edges that connect the last step of a task to all join operations on that task. 

CGs can be used to define data races, an important class of bugs in parallel programs.  We  say that a data  race occurs on location L in a computation graph, G,  if there exist steps S1  and S2  in G such that there is no path of directed edges from S1  to S2  or from S2  to S1  in G, and both S1  and S2  read or write L (with at least one of the accesses being a write, since two parallel reads do not pose a problem).

CGs can also be used to reason about the ideal    parallelism of a parallel program as follows:
- Define WORK(G) to be the sum of the execution times of all nodes in CG G, 
- Define SPAN(G) to be the length of a longest path in G, when adding up the execution times of all nodes in the path. The longest paths are known as critical paths, so SPANalso represents the **critical path length** (CPL) of G.

> we define the ideal parallelism of Computation Graph G as the ratio, **WORK(G)/SPAN(G)**. The ideal parallelism is an upper limit on the speedup factor that can be obtained from parallel execution of nodes in computation graph G

## Amdahl's Law

if q ≤ 1 is the fraction of WORK in a parallel program that must be executed sequentially, then the best speedup that can be obtained for that program for any number of processors, P , is Speedup(P)≤ 1/q.

Amdahl’s Law reminds us to watch out for sequential bottlenecks both when designing parallel algorithms and when implementing programs on real machines.  As an example, if q = 10%, then Amdahl's Law reminds us that the best possible speedup must be ≤ 10 (which equals 1/q ), regardless of the number of processors available.
