# Functional Parallelism
Functional Parallelism

###### Objectives
- Functional parallelism using the Future construct
- Create functional-parallel programs using Java's Fork/Join Framework
- Apply the princple of memoization to optimize functional parallelism
- Create functional-parallel programs using Java Streams
- Explain the concepts of data races and functional/structural determinism

## Future Tasks

Future tasks are tasks with return values, and a future object is a “handle” for accessing a task’s return value. There are two key operations that can be performed on a future object, A:
- Assignment — A can be assigned a reference to a future object returned by a task of the form, future { ⟨ task-with-return-value ⟩ }. The content of the future object is constrained to be single assignment, and cannot be modified after the future task has returned.

- Blocking read — the operation, A.get(), waits until the task associated with future object A has completed, and then propagates the task’s return value as the value returned by A.get().  Any statement, S, executed after A.get() can be assured that the task associated with future object A must have completed before S starts execution.

## Creating Future  Tasks  in Java’s  Fork/Join Framework

how to express future tasks in Java’s Fork/Join (FJ) framework. Some key differences between future tasks and regular tasks in the FJ framework are as follows:
- A future task extends the **RecursiveTask** class in the FJ framework, instead of **RecursiveAction** as in regular tasks.
- The compute() method of a future task must have a non-void return type, whereas it has a void return type  for  regular tasks.
- A method call like left.join() waits for the task referred to by object left in both cases, but also provides the task’s return value in the case of future tasks.

## Memoization

“memoization”, is used to remember results of function calls f (x) as follows:
- Create a data structure that stores the set for each call f (x) that returns y
- Perform look ups in that data structure when processing calls of the form f (x') when x' equals one of the xi inputs for which f(xi) has already been computed.

## Java Streams

Streams provide a functional approach to operating on collections of data e.g.

```
students.stream()
    .filter(s -> s.getStatus() == Student.ACTIVE)
    .mapToInt(a -> a.getAge())
    .average();
```

An important benefit of using Java  streams when possible is that the  pipeline can be made to execute in parallel by designating the source to be a parallel stream, i.e., by simply replacing students.stream() in the above code by students.parallelStream()  or Stream.of(students).parallel(). This form of functional parallelism is a major convenience for the programmer, since they do not need to worry about explicitly allocating intermediate collections (e.g., a collection of all active students), or about ensuring that parallel accesses to data collections are  properly synchronized.

## Determinism and Data Races

 The relationship between determinism and data races in parallel programs
 
 - A parallel program is said to be **functionally deterministic** if it always computes the same answer when given the same input, and **structurally deterministic** if it always computes the same computation graph, when given the same input.

The presence of data races often leads to functional and/or structural nondeterminism because a parallel program with data races may exhibit different behaviors for the same input, depending on the relative scheduling and timing of memory accesses involved in a data race. In general, the absence of data races is not sufficient to guarantee determinism.

> Furthermore, there may be cases of “benign” nondeterminism for programs with data races in which different executions with the same input may generate different outputs, but all the outputs may be acceptable in the context of the application, e.g., different locations for a search pattern in a target string.
