# Threads

In the form of **java.lang.Thread**

When an instance of Thread is created (via a *new* operation), it does not start executing right away; instead, it can only start executing when its *start()* method is invoked.

The Thread class also includes a wait operation in the form of a join() method. If thread t0 performs a t1.join() call, thread t0 will be forced to wait until thread t1 completes, after which point it can safely access any values computed by thread t1.

# Structured Locks (implicit Monitors Lock)
A major benefit of structured locks is that their acquire and release operations are implicit, since these operations are automatically performed by the Java runtime environment when entering and exiting the scope of a **synchronized** statement or method, even if an exception is thrown in the middle.  

synchronized lock is also a kind of **re-entrant** Lock.
```java
x = buffer();
insert() {
    synchronized(x) {
        wait() if x is full;
        buffer[i] = tmp;
        i++;
        notify();
    }
}
remove() {
    synchronized(x) {
        wait() if x is empty;
        tmp = buffer[i];
        remover(tmp);
        i++;
        notify();
    }
}
```
**wait()** and **notify()** operations that can be used to block and resume threads that need to wait for specific conditions.


## [Unstructured Locks](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/locks/Lock.html)
- hand-over-hand Locking:  
implements a non-nested pairing of lock/unlock operations which cannot be achieved with synchronized statements/methods.  
**Re-entrant Lock:(For single thread)**  
A thread could get the same lock many times.(avoid DeadLock)  
***A thread is executing a method with a lock, within which invoke another method that requires the same lock, the thread can directly execute the called method without reacquiring the lock.***

-    tryLock(L1)

- Read-Write Lock
    ```java
    ReentrantReadWriteLock()
    // multiple threads are permitted to acquire a lock \mathtt {L}L in “read mode”
    L.readLock().lock()
    // only one thread is permitted to acquire the lock in “write mode”
    L.writeLock().lock()
    ```

## Liveness
- DeadLock:  
  all threads are blocked indefinitely, thereby preventing any forward progress.
- LiveLock  
  all threads repeatedly perform an interaction that prevents forward progress.  
  eg. an infinite “loop” of repeating lock acquire/release patterns.
- Starvation  
  at least one thread is prevented from making any forward progress.

## Dining Philosophers
Five threads:
- think
- pick up chopsticks
- eat
- put down chopsticks

1. Structured Locks:  
   may get deadlock
2. Unstructured Locks:
   ```java
   tryLock()
   unLock()
   ```
   may get LiveLock

Solution: One philosopher pick up the right chopsticks first, all the other philosophers pick up the left chopsticks first. There still exist Starvation, instead of deadlock or livelock.

---
## Exercise:
```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

// Use ReentrantLock Object
private final ReentrantLock lock = new ReentrantLock();
lock.lock();
try{
    // specific operation
} finally {
    lock.unlock();
}

// Use ReentrantReadWriteLock Object
private final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
private final Lock rl = rwl.readLock();
private final Lock wl = rwl.writeLock();
wl.lock();
try{
    // read operation
} finally {
    wl.unlock();
}

rl.lock();
try{
    // write operation
} finally {
    rl.unlock();
}
```

--- 
# Lock Classification:
## 1. Positive Lock & Negative Lock
### - Positive Lock:  
Think of it in a positive way that no other threads gonna modify the data, thus no lock gonna be applied. Only before current thread is going to update the data, it will check if there is any modification since getting:  
- if no modification, current thread updates it.
- else, return or report error.  
  
#### Implementation:  
**CAS**(Compare and Swap)：  
CAS is a kind of algorithm implements thread synchronization without lock, which involves three operands:
- Value from memory: V
- Value need to compare: A
- Value gonna be written in: B
```java
// pseudo code
// The compare and swap are integrated as a single atomic operation.
Boolean compareAndSwap(V, A, B) {
    if (V == A) {
        V = B;
        return True;
    }
    else
        return False;
}
```
##### Problems:
- ABA problem: CAS has to check if there is any modification to the memory value, if it used to be A, then to B, at last it changes back to A. The CAS will find there is no modification, while actually it does changed. The Solution is to add version number ahead of it, like: A-B-A to 1A-2B-3A.
- Loop Cost: it will keep spinning if not succeed, which will cost a lot for CPU.
- AtomicReference is appended to put several variables to an object to CAS instead of only one variable.

### - Negative Lock:
Negativly believe that there must be some threads modifying the value, thus lock when read or write. Other threads have to be blocked until getting the lock.  
Typically in Java: **Synchronized**

## 2. Fair Lock & Unfair Lock
It is different on the preemption rules, fail locks means it depends on the arriving sequence, while the unfail locks may use priority or something else.
```java
// fail lock
// more overhead
ReentrantLock pairLock = new ReentrantLock（true）
// unfair lock
// default
ReentrantLock pairLock = new ReentrantLock（false）
```
## 3. Exclusive Lock & Shared Lock
Depending on whether the lock can only be held by a single thread or by multiple threads.
For example:  
**ReentrantLock** is an exclusive lock.  
**ReadWriteLock's readLock** is a shared lock.
## 4. Reentrant Lock
Implementation:
keep track of the thread that possesses the lock and associate a counter, when a thread is asking for the lock:
- if the thread is the current owner of the lock, counter plus 1.
- if the thread is not the current owner of the lock, blocked.
```java
public synchronized void helloA(){
...
}

public synchronized void helloB(){
    ...
    helloA();
}
Synchronized Lock is a kind of reentrant lock, when invoking helloA by helloB, it will not be blocked, instead it will be processed immediately.
```
## 5. Spin Lock
### Background
Blocking or waking up a thread requires the operating system to switch the CPU state, which requires processor time.
### Definition:
If the thread that previously locked the synchronization resource has released the lock before the spin is completed, the current thread can directly acquire the synchronization resource without blocking, thereby avoiding the overhead of switching threads.