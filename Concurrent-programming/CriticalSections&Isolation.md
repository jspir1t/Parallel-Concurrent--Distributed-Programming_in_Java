# Critical Section
Only one thread could access to the Critical Section at any time.  
**Critical sections** and the **isolated construct** can help concurrent threads manage their accesses to shared resources, at a higher level than just using **locks**.

# Object-Based Isolation
An isolated construct can be extended with a set of objects that indicate the scope of isolation, by using the following rules:  
if two isolated constructs have an empty intersection in their object sets they can execute in parallel, otherwise they must execute in mutual exclusion.
```java
isolated(cur) {
    j = cur;
    cur += 1;
}
```

# Aotomic Variables
### Example1:
```java
import java.util.concurrent.atomic.AtomicInteger

private AtomInteger cur = new AtomInteger;
j = cur.incrementAndGet();

// equals to:
isolated (cur) {
    j=cur;
    cur=cur+1;
} 
```
Although the two methods are of same semantics, whil using **Atomic Variables** is more efficient because of the hardware support.

### Example2:  
Atomic Reference variables
```java
// atomic reference ref
ref.compareAndSet(expected, new) {
    /**
    if value of ref and expected are the same, set the value of ref to new and return true.
    If ref and expected have different values, compareAndSet() will not modify anything and will simply return false.
    */
}
```

[Comparation between Synchronized, Locks, Isolated and  Atomic Variables ](https://www.ibm.com/developerworks/library/j-jtp11234/)  
[Java Locks Classification](https://tech.meituan.com/2018/11/15/java-lock.html)

# Read and Write Isolation
