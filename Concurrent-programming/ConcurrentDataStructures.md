Concurrent Data Structures forms an essential software layer in all multithreaded programming systems.  

# Optimistic Concurrency Control([Optimistic Lock](./Threads&Locks.md))
## Definition:
OCC assumes that multiple transactions can frequently complete without interfering with each other. While **running**, transactions use data resources **without acquiring locks** on those resources. **Before committing**, each transaction verifies that no other transaction has modified the data it has read. If the check reveals conflicting modifications, the committing transaction **rolls back** and can be restarted.

eg. [Compare and Set](./Threads&Locks.md) application on getAndAdd() method:
```java
// pseudo implementation

// synchronized implementation
GetAndAdd(delta) {
    cur = this.GET();
    next = cur + delta;
    this.set(next);
    return cur;
}

// No deadlock and livelock
{
    compareAndSet();
    GetAndAdd(delta) {
        // loop until the operation succeed, avoid deadlock
        while (true) {
            cur = this.get();
            next = cur + delta;
            // if two threads, T1 and T2 call compareAndSet() with the same curVal that matches A’s current value, only one of them will succeed in updating A with their newVal.
            if (this.compareAndSet(cur, next)) {
                // eventually will succeed, avoid livelock
                return true;
            }
        }
    }
}
```
## Analysis:  
OCC is generally used in environments with **low data contention**. When conflicts are rare, transactions can complete without the expense of managing locks and without having transactions wait for other transactions' locks to clear, leading to higher throughput than other concurrency control methods. However, if contention for data resources is frequent, the cost of repeatedly restarting transactions hurts performance significantly; it is commonly thought that other concurrency control methods have better performance under these conditions. However, locking-based ("pessimistic") methods also can deliver poor performance because locking can drastically limit effective concurrency even when deadlocks are avoided.  
## Steps:
- **Begin**: Record a timestamp marking the transaction's beginning.
- **Modify**: Read database values, and tentatively write changes.
- **Validate**: Check whether other transactions have modified data that this transaction has used (read or written).
- **Commit/Rollback**: If there is no conflict, make all changes take effect. If there is a conflict, resolve it, typically by aborting the transaction, although other resolution schemes are possible. 

# Concurrent Queue
java.util.concurent.ConcurrentLinkedQueue
```java
ENQ(x) {
    while(true) {
        // replace an object reference like tail by an AtomicReference
        if (!tail.next.compareAndSet(null, x)) {
            continue;
        }
    }
}
```

# Linearizability
### A goal for implementation of a concurrent data structure:  
ensure that all its executions are linearizable by using whatever combination of constructs (e.g., locks, isolated, actors, optimistic concurrency) is deemed appropriate to ensure correctness while giving the maximum performance.

---
# Exercise
```java
/**
 * ParComponent represents a single component in the graph. A component may
 * be a singleton representing a single node in the graph, or may be the
 * result of collapsing edges to form a component from multiple nodes.
 */
public void computeBoruvka(final Queue<ParComponent> nodesLoaded, final SolutionToBoruvka<ParComponent> solution) {
    ParComponent loopNode = null;
    while (!nodesLoaded.isEmpty()) {
        loopNode = nodesLoaded.poll();

        if (loopNode == null || loopNode.isDead || !loopNode.lock.tryLock()) {
            continue;
        }

        // Get minimum edge adhered to node loopNode and contracted respective nodes
        final Edge<ParComponent> e = loopNode.getMinEdge();
        if (e == null) {
            loopNode.lock.unlock();
            solution.setSolution(loopNode);
            break;
        }

        final ParComponent other = e.getOther(loopNode);
        if (other.isDead || !other.lock.tryLock()) {
            loopNode.lock.unlock();
            // if the other node cannot get lock, rolling back by add loopNode to the tail of queue 
            nodesLoaded.add(loopNode);
            continue;
        }

        other.isDead = true;
        loopNode.merge(other, e.weight());
        loopNode.lock.unlock();
        other.lock.unlock();
        // release locks and add the contracted node to queue for further operation
        nodesLoaded.add(loopNode);
    }
}
```