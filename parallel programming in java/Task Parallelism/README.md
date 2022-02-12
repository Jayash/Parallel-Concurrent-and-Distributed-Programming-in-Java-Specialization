# Task Parallelism
Task Parallelism

###### Objectives
- Task parallelism using Asynkc/Finish constructs
- Create task-parallel programs using Java's Fork/Join Framework
- Interpret Computation Graph abstraction for task-parallel programs
- Evaluate the Multiprocessor Scheduling problem using Computation Graphs
- Assess sequetional bottlenecks using Amdahl's Law

###### Task Creation and Termination (Async, Finish)
Async notation for task creation: “async ⟨stmt1⟩”, causes the parent task (i.e., the task executing the async statement) to create a new child task to execute the body of the async, ⟨stmt1⟩, asynchronously (i.e., before, after, or in parallel) with the remainder of the parent task. 
Finish notation for task termination: “finish ⟨stmt2⟩” causes the parent task to execute ⟨stmt2⟩, and then wait until ⟨stmt2⟩ and all async tasks created within ⟨stmt2⟩ have completed. Async and finish constructs may be arbitrarily nested.

The array-sum example:
```
finish {
  async S1; // asynchronously compute sum of the lower half of the array
  S2;       // compute sum of the upper half of the array in parallel with S1
}
S3; // combine the two partial sums after both S1 and S2 have finished
```

###### Creating Tasks in Java's Fork/Join Framework
