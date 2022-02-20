# Critical Sections and Isolation
Critical Sections and Isolation

Critical/isolated sections are higher-level concurrent programming constructs (relative to locks) that simplify the implementation of mutual exclusion by guaranteeing the absence of deadlocks and livelocks. Object-based isolation relaxes the constraints imposed by critical sections by allowing mutual exclusion to be specified on a per-object basis. Java's atomic variables represent an important, but restricted, case of object-based isolation that is implemented efficiently on all hardware platforms. how object-based isolation can be further relaxed with read/write access modes.

###### Objectives
- Create concurrent programs with critical sections to coordinate accesses to shared resources
- Create concurrent programs with object-based isolation to coordinate accesses to shared resources with more overlap than critical sections
- Evaluate different approaches to implementing the Concurrent Spanning Tree algorithm
- Create concurrent programs using Java's atomic variables
- Evaluate the impact of read vs. write operations on concurrent accesses to shared resources

## Critical Sections

Critical sections and the isolated construct can help concurrent threads manage their accesses to shared resources, at a higher level than just using locks. When programming with threads, it is well known that the following situation is defined to be a data race error — when two accesses on the same shared location can potentially execute in parallel, with least one access being a write. However, there are many cases in practice when two tasks may legitimately need to perform concurrent accesses to shared locations, as in the bank money transfer. 

With critical sections, two blocks of code that are marked as isolated, say A and B, are guaranteed to be executed in mutual exclusion with A executing before B or vice versa. With the use of isolated constructs, it is impossible for the bank money transfer to end up in an inconsistent state because all the reads and writes for one isolated section must complete before the start of another isolated construct. Thus, the parallel program will see the effect of one isolated section completely before another isolated section can start.

## Object-Based Isolation

Object-based isolation, which generalizes the isolated construct and relates to the classical concept of monitors. The fundamental idea behind object-based isolation is that an isolated construct can be extended with a set of objects that indicate the scope of isolation, by using the following rules: if two isolated constructs have an empty intersection in their object sets they can execute in parallel, otherwise they must execute in mutual exclusion. Implementing this capability can be very challenging with locks because a correct implementation must enforce the correct levels of mutual exclusion without entering into deadlock or livelock states. The doubly linked-list object set for a delete() method can be defined as consisting of three objects — the current, previous, and next objects in the list, and that this object set is sufficient to safely enable parallelism across multiple calls to delete(). The Java code sketch to achieve this object-based isolation is as follows:

```
isolated(cur, cur.prev, cur.next, () -> {
    . . . // Body of object-based isolated construct
});
```
The relationship between object-based isolation and monitors is that all methods in a monitor object, M1, are executed as object-based isolated constructs with a singleton object set, {M1}. Similarly, all methods in a monitor object, M2, are executed as object-based isolated constructs with a singleton object set, {M2} which has an empty intersection with {M1}.

## Spanning Tree

Object-based isolation create a parallel algorithm to compute spanning trees for an undirected graph. spanning tree specifies a subset of edges in the graph that form a tree (no cycles), and connect all vertices in the graph. A standard recursive method for creating a spanning tree is to perform a depth-first traversal of the graph, making the current vertex a parent of all its neighbors that don’t already have a parent assigned in the tree.

This approach to parallelize the spanning tree computation executes recursive Compute(c) method calls in parallel for all neighbors, c, of the current vertex, v. Object-based isolation helps avoid a data race in the MakeParent(v,c) method, when two parallel threads might attempt to call MakeParent(v1, c) and MakeParent(v2, c) on the same vertex c at the same time. The role of object-based isolation is to ensure that all calls to MakeParent(v,c) with the same c value must execute the object-based isolated statement in mutual exclusion, whereas calls with different values of c can proceed in parallel.

## Atomic Variables

Atomic Variables, an important special case of object-based isolation which can be very efficiently implemented on modern computer systems. when multiple threads processing an array, each using object-based isolation to safely increment a shared object, cur, to compute an index j which can then be used by the thread to access a thread-specific element of the array. 

However, instead of using object-based isolation, we can declare the index cur to be an Atomic Integer variable and use an atomic operation called getAndAdd() to atomically read the current value of cur and increment its value by 1. Thus, j=cur.getAndAdd(1) has the same semantics as (cur) { \mathtt{ j = cur; cur = cur+1;}j=cur;cur=cur+1; } but is implemented much more efficiently using hardware support on today’s machines. 

Atomic Reference variables, which are reference variables that can be atomically read and modified using methods such as compareAndSet(). If we have an atomic reference ref, then the call to ref.compareAndSet(expected, new) will compare the value of ref to expected, and if they are the same, set the value of ref to new and return true. This all occurs in one atomic operation that cannot be interrupted by any other methods invoked on the ref object. If ref and expected have different values, compareAndSet() will not modify anything and will simply return false.

## Read-Write Isolation

Read-Write Isolation, which is a refinement of object-based isolation, and is a higher-level abstraction of the read-write locks studied earlier as part of Unstructured Locks. The main idea behind read-write isolation is to separate read accesses to shared objects from write accesses. This approach enables two threads that only read shared objects to freely execute in parallel since they are not modifying any shared objects. The need for mutual exclusion only arises when one or more threads attempt to enter an isolated section with write access to a shared object. 

This approach exposes more concurrency than object-based isolation since it allows read accesses to be executed in parallel. In the doubly-linked list example from our lecture, when deleting an object cur from the list by calling delete(cur), we can replace object-based isolation on cur with read-only isolation, since deleting an object does not modify the object being deleted; only the previous and next objects in the list need to be modified.
