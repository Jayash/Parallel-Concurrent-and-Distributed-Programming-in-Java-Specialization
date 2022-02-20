# Threads and Locks
Threads and Locks

###### Objectives
- Understand the role of Java threads in building concurrent programs
- Create concurrent programs using Java threads and the synchronized statement (structured locks)
- Create concurrent programs using Java threads and lock primitives in the java.util.concurrent library (unstructured locks)
- Analyze programs with threads and locks to identify liveness and related concurrency bugs
- Evaluate different approaches to solving the classical Dining Philosophers Problem


Threads and locks have served as primitive building blocks for concurrent programming for over five decades. All computing platforms today include some form of support for threads and locks, and make them available for use by developers in a wide range of programming languages. created, joined, and synchronized using structured (e.g., synchronized statements/methods) and unstructured (e.g., java.util.concurrent libraries) locks in Java. New classes of bugs that can arise when concurrent programs need to access shared resources. These bugs are referred to as violations of liveness/progress guarantees, and include deadlock, livelock, and starvation.

## Java Threads

A unique aspect of Java compared to prior mainstream programming languages is that Java included the notions of threads (as instances of the java.lang.Thread class) in its language definition right from the start.

When an instance of Thread is created (via a new operation), it does not start executing right away; instead, it can only start executing when its start() method is invoked. The statement or computation to be executed by the thread is specified as a parameter to the constructor.

The Thread class also includes a wait operation in the form of a join() method. If thread \mathtt t0 performs a t1.join() call, thread t0 will be forced to wait until thread t1 completes, after which point it can safely access any values computed by thread t1. Since there is no restriction on which thread can perform a join on which other thread, it is possible for a programmer to erroneously create a deadlock cycle with join operations. (A deadlock occurs when two threads wait for each other indefinitely, so that neither can make any progress.)

## Structured Locks

Structured locks can be implemented using synchronized statements and methods in Java. Structured locks can be used to enforce mutual exclusion and avoid data races. A major benefit of structured locks is that their acquire and release operations are implicit, since these operations are automatically performed by the Java runtime environment when entering and exiting the scope of a synchronized statement or method, even if an exception is thrown in the middle.

wait() and notify() operations can be used to block and resume threads that need to wait for specific conditions. For example, a producer thread performing an insert() operation on a bounded buffer can call wait() when the buffer is full, so that it is only unblocked when a consumer thread performing a remove() operation calls notify(). Likewise, a consumer thread performing a remove() operation on a bounded buffer can callwait() when the buffer is empty, so that it is only unblocked when a producer thread performing an insert() operation calls notify(). Structured locks are also referred to as intrinsic locks or monitors.

## Unstructured Locks 

Unstructured locks (which can be obtained in Java by creating instances of ReentrantLock()). Explicit lock() and unlock() operations on unstructured locks can be used to support a hand-over-hand locking pattern that implements a non-nested pairing of lock/unlock operations which cannot be achieved with synchronized statements/methods. The tryLock() operations in unstructured locks can enable a thread to check the availability of a lock, and thereby acquire it if it is available or do something else if it is not. The read-write locks (which can be obtained in Java by creating instances of ReentrantReadWriteLock()), whereby multiple threads are permitted to acquire a lock L in “read mode”, L.readLock().lock(), but only one thread is permitted to acquire the lock in “write mode”, L.writeLock().lock().

However, it is also important to remember that the generality and power of unstructured locks is accompanied by an extra responsibility on the part of the programmer, e.g., ensuring that calls to unlock() are not forgotten, even in the presence of exceptions.

## Liveness and Progress Guarantees

Three ways in which a parallel program may enter a state in which it stops making forward progress. For sequential programs, an “infinite loop” is a common way for a program to stop making forward progress, but there are other ways to obtain an absence of progress in a parallel program. The first is deadlock, in which all threads are blocked indefinitely, thereby preventing any forward progress. The second is livelock, in which all threads repeatedly perform an interaction that prevents forward progress, e.g., an infinite “loop” of repeating lock acquire/release patterns. The third is starvation, in which at least one thread is prevented from making any forward progress. 

The term “liveness” refers to a progress guarantee. The three progress guarantees that correspond to the absence of the conditions listed above are deadlock freedom, livelock freedom, and starvation freedom. 

Starvation and livelock are much less common a problem than deadlock, but are still problems that every designer of concurrent software is likely to encounter.

**Starvation**
Starvation describes a situation where a thread is unable to gain regular access to shared resources and is unable to make progress. This happens when shared resources are made unavailable for long periods by "greedy" threads. For example, suppose an object provides a synchronized method that often takes a long time to return. If one thread invokes this method frequently, other threads that also need frequent synchronized access to the same object will often be blocked.

**Livelock**
A thread often acts in response to the action of another thread. If the other thread's action is also a response to the action of another thread, then livelock may result. As with deadlock, livelocked threads are unable to make further progress. However, the threads are not blocked — they are simply too busy responding to each other to resume work. This is comparable to two people attempting to pass each other in a corridor: Alphonse moves to his left to let Gaston pass, while Gaston moves to his right to let Alphonse pass. Seeing that they are still blocking each other, Alphone moves to his right, while Gaston moves to his left. They're still blocking each other

## Dining Philosophers Problem 

In Dining Philosophers Problem there are five threads, each of which models a “philosopher” that repeatedly performs a sequence of actions which include think, pick up chopsticks, eat, and put down chopsticks. 

First, using structured locks, this solution could lead to a deadlock scenario (but not livelock). Second, using unstructured locks with tryLock() and unlock() operations that never block, this solution could lead to a livelock scenario (but not deadlock). Finally, observed how a simple modification to the first solution with structured locks, in which one philosopher picks up their right chopstick and their left, while the others pick up their left chopstick first and then their right, can guarantee an absence of deadlock. 
