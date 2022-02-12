# Data flow Synchronization and Pipelining
Data flow Synchronization and Pipelining

###### Objectives
- Create split-phase barriers using Java's Phaser construct
- Create point-to-point synchronization patterns using Java's Phaser construct
- Evaluate parallel loops with point-to-point synchronization in an iterative-averaging example
- Analyze pipeline parallelism using the principles of point-to-point synchronization
- Interpret data flow parallelism using the data-driven-task construct

## Split-phase Barriers with Java  Phasers

```
forall (i : [0:n-1]) { 
  print HELLO, i;
  myId = lookup(i); // convert int to a string 
  print BYE, myId;
}
```

The operation **ph.arriveAndAwaitAdvance()**, can be used to implement a barrier through phaser object ph. There are two  possible positions for inserting a barrier between the two print statements above — before or after the call to lookup(i). However, upon closer examination, we can see that the call to  lookup(i) is local to iteration i and that there is no specific need to either complete it before the barrier or to complete it after the barrier.  In fact, the call to lookup(i) can be performed in parallel with the barrier. To facilitate this split-phase barrier (also known as a fuzzy barrier) we use two separate APIs from Java Phaser class —  ***ph.arrive()** and **ph.awaitAdvance()**. Together these two APIs form a barrier, but we now have the freedom to insert a computation such as lookup(i) between the two calls as follows:

```
// initialize phaser ph	for use by n tasks ("parties") 
Phaser ph = new Phaser(n);
// Create forall loop with n iterations that operate on ph 
forall (i : [0:n-1]) {
  print HELLO, i;
  int phase = ph.arrive();
  
  myId = lookup(i); // convert int to a string

  ph.awaitAdvance(phase);
  print BYE, myId;
}
```

## Point-to-Point  Synchronization  with Phasers

| Task 0  | Task 1 | Task 2 |
| ------------- | ------------- | ------------- |
| 1a:X=A();//cost=1 | 1b:Y=B();//cost=2 | 1c:Z=C();//cost=3 |
| 2a:ph0.arrive(); | 2b:ph1.arrive(); | 2c:ph2.arrive(); |
| 3a:ph1.awaitAdvance(0); | 3b:ph0.awaitAdvance(0); | 3c:ph1.awaitAdvance(0); |
| 4a:D(X,Y);//cost=3 | 4b:ph2.awaitAdvance(0); | 4c:F(Y,Z);//cost=1 |
| | 5b:E(X,Y,Z);//cost=2 | |


