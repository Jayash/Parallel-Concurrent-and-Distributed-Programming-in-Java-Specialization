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

>Note that join() is a lower-level primitive than finish because {\tt join()}join() waits for a specific task, whereas finish implicitly waits for all tasks created in its scope. To implement the finish construct using join() operations, you have to be sure to call join() on every task created in the finish scope.

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

FJ tasks are executed in a ForkJoinPool, which is a pool of Java threads. This pool supports the invokeAll() method that combines both the fork and join operations by executing a set of tasks in parallel, and waiting for their completion. For example, {\tt ForkJoinTask.invokeAll(left, right)}ForkJoinTask.invokeAll(left,right) implicitly performs fork() operations on left and right, followed by join() operations on both objects.
