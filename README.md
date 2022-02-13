# Parallel-Concurrent-and-Distributed-Programming-in-Java-Specialization
Parallel, Concurrent, and Distributed Programming in Java Specialization

## Parallel programming in java

Parallelism is when multiple tasks OR several parts of a unique task literally run at the same time, e.g. on a multi-core processor. Parallelism does not require two tasks to exist. It literally physically run parts of tasks OR multiple tasks, at the same time using the multi-core infrastructure of CPU, by assigning one core to each task or sub-task.

> In a single-core CPU, we may get concurrency but NOT parallelism.

![image](https://user-images.githubusercontent.com/7610065/153738608-aa5c6f7a-78a9-45d0-b385-77937267011f.png)

- [Task Parallelism](https://github.com/Jayash/Parallel-Concurrent-and-Distributed-Programming-in-Java-Specialization/tree/main/parallel%20programming%20in%20java/Task%20Parallelism)
- [Functional Parallelism](https://github.com/Jayash/Parallel-Concurrent-and-Distributed-Programming-in-Java-Specialization/tree/main/parallel%20programming%20in%20java/Functional%20Parallelism)
- [Loop Parallelism](https://github.com/Jayash/Parallel-Concurrent-and-Distributed-Programming-in-Java-Specialization/tree/main/parallel%20programming%20in%20java/Loop%20Parallelism)
- [Data flow Synchronization and Pipelining](https://github.com/Jayash/Parallel-Concurrent-and-Distributed-Programming-in-Java-Specialization/tree/main/parallel%20programming%20in%20java/Data%20flow%20Synchronization%20and%20Pipelining)

## Concurrent programming in java

Concurrency means multiple tasks which start, run, and complete in overlapping time periods, in no specific order. Concurrency is essentially applicable when we talk about a minimum of two tasks or more. When an application is capable of executing two tasks virtually at the same time, we call it a concurrent application.

![image](https://user-images.githubusercontent.com/7610065/153738603-58918ed6-ba32-4cfe-99f3-cdc2bc1d0e53.png)


## Concurrent and parallel executions and parallelism

![image](https://user-images.githubusercontent.com/7610065/153738620-ce5d5ad2-1373-4b63-a511-efc921cc5631.png)

![image](https://user-images.githubusercontent.com/7610065/153738637-262a58c1-348b-4587-983a-7ddb246f9444.png)

- An application can be concurrent but not parallel, which means that it processes more than one task at the same time, but no two tasks are executing at the same time instant.
- An application can be parallel but not concurrent, which means that it processes multiple sub-tasks of a task in a multi-core CPU at the same time.
- An application can be neither parallel nor concurrent, which means that it processes all tasks one at a time, sequentially.
- An application can be both parallel and concurrent, which means that it processes multiple tasks concurrently in a multi-core CPU at the same time.
