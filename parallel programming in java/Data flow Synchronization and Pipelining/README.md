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

Span (critical path length) would be 6 units of time if we used a barrier, but is reduced to 5 units of time if we use individual phasers as shown in the following table

| Task 0  | Task 1 | Task 2 |
| ------------- | ------------- | ------------- |
| 1a:X=A();//cost=1 | 1b:Y=B();//cost=2 | 1c:Z=C();//cost=3 |
| 2a:ph0.arrive(); | 2b:ph1.arrive(); | 2c:ph2.arrive(); |
| 3a:ph1.awaitAdvance(0); | 3b:ph0.awaitAdvance(0); | 3c:ph1.awaitAdvance(0); |
| 4a:D(X,Y);//cost=3 | 4b:ph2.awaitAdvance(0); | 4c:F(Y,Z);//cost=1 |
| | 5b:E(X,Y,Z);//cost=2 | |

Each column in the table represents execution of a separate task, and the calls to arrive() and awaitAdvance(0) represent synchronization across different tasks via phaser objects, ph0, ph1, and ph2, each of which is initialized with a party count of 1 (only one signalling task). (The parameter 0 in awaitAdvance(0) represents a transition from phase 0 to phase 1.)

## One-Dimensional Iterative Averaging with Phasers

In Iterative Averaging we observed that a full barrier is not necessary since forall iteration i only needs to wait for iterations i − 1 and i + 1 to complete their current phase before iteration i can move to its next phase. This idea can be captured by phasers, if we allocate an array of phasers as follows:

```
// Allocate array of phasers
Phaser[] ph = new Phaser[n+2]; //array of phasers
for (int i = 0; i < ph.length; i++) ph[i] = new Phaser(1);

// Main computation 
forall ( i: [1:n-1]) {
  for (iter: [0:nsteps-1]) {
    newX[i] = (oldX[i-1] + oldX[i+1]) / 2;
    ph[i].arrive();
    
    if (index > 1) ph[i-1].awaitAdvance(iter);
    if (index < n-1) ph[i + 1].awaitAdvance(iter); 
    swap pointers newX and oldX;
  }
}
```

grouping/chunking of parallel iterations in a forall can be an important consideration for performance (due to reduced overhead). The idea of grouping of parallel iterations can be extended to forall loops with phasers as follows:

```
// Allocate array of phasers proportional to number of chunked tasks 
Phaser[] ph = new Phaser[tasks+2]; //array of phasers
for (int i = 0; i < ph.length; i++) ph[i] = new Phaser(1);

// Main computation 
forall ( i: [0:tasks-1]) {
  for (iter: [0:nsteps-1]) {
    // Compute leftmost boundary element for group
    int left = i * (n / tasks) + 1;
    myNew[left] = (myVal[left - 1] + myVal[left + 1]) / 2.0;
    
    // Compute rightmost boundary element for group 
    int right = (i + 1) * (n / tasks);
    myNew[right] = (myVal[right - 1] + myVal[right + 1]) / 2.0;
    
    // Signal arrival on phaser ph AND LEFT AND RIGHT ELEMENTS ARE AV 
    int	index = i + 1;
    ph[index].arrive();
    
    // Compute interior elements in parallel with barrier 
    for (int j = left + 1; j <= right - 1; j++)
      myNew[j] = (myVal[j - 1] + myVal[j + 1]) / 2.0;
    // Wait for previous phase to complete before advancing 
    if (index > 1) ph[index - 1].awaitAdvance(iter);
    if (index < tasks) ph[index + 1].awaitAdvance(iter);
    swap pointers newX and oldX;
  }
}
```
