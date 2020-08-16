# Actor
A higher level of concurrency control than locks and isolated section, which contains:
- MailBox
- Method
- Local State

The Actor model is reactive, in that actors can only execute methods in response to **messages**; these methods can read/write local state and/or send messages to other actors.

## Example:
If the messages are sent between the same pair of actors, the oreder will be preserved.  

---
## Exercise:  
```java
package edu.coursera.concurrent;
import java.util.List;
import java.util.ArrayList;

/**
 * An example sequential implementation of the Sieve of Eratosthenes,
 * which is used to get the number of prime between 0 and limit.
 */
public final class SieveSequential extends Sieve {
    @Override
    public int countPrimes(final int limit) {
        final List<Integer> localPrimes = new ArrayList<Integer>();
        localPrimes.add(2);
        for (int i = 3; i <= limit; i += 2) {
            checkPrime(i, localPrimes);
        }
        return localPrimes.size();
    }

    private void checkPrime(final int candidate,
            final List<Integer> primesList) {
        boolean isPrime = true;
        final int s = primesList.size();
        for (int i = 0; i < s; ++i) {
            final Integer loopPrime = primesList.get(i);
            if (candidate % loopPrime.intValue() == 0) {
                isPrime = false;
                break;
            }
        }
        if (isPrime) {
            primesList.add(candidate);
        }
    }
}

```
By using their PCDP library, we could transfer the algorithm above to this concurrent one:

```java
package edu.coursera.concurrent;

import edu.rice.pcdp.Actor;
import edu.rice.pcdp.PCDP;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public final class SieveActor extends Sieve {

    /** Using SieveActorActor instances to perform like filter which
     * filter 2, 3, 5... Then, iterate all the filters to calculate the number of primes.
     */
    @Override
    public int countPrimes(final int limit) {
        final SieveActorActor sieveActorActor = new SieveActorActor(2);
        PCDP.finish(() -> {
            // Apparently even numbers are not primes
            for (int i = 3; i <= limit; i+=2) {
                sieveActorActor.send(i);
            }
            sieveActorActor.send(0);
        });
        int numPrimes = 0;
        SieveActorActor loopActor = sieveActorActor;
        while (loopActor != null) {
            numPrimes += loopActor.numLocalPrimes();
            loopActor = loopActor.nextActor();
        }
        return numPrimes;
    }

    /**
     * An actor class that helps implement the Sieve of Eratosthenes in parallel, omit the number that is a multiple of current Actor's number. When there are too much numbers that satisy the current Actor(more than "MAX_LOCAL_PRIME"), you should transfer all rest job to the next Actor.
     */
    public static final class SieveActorActor extends Actor {

        private static final int MAX_LOCAL_PRIME = 500;
        private List<Integer> localPrimes;
        private int numLocalPrimes;
        private SieveActorActor nextActor;

        public SieveActorActor(final int localPrime) {
            this.localPrimes = new ArrayList<>();
            this.localPrimes.add(localPrime);
            this.numLocalPrimes = 1;
            this.nextActor = null;
        }

        public SieveActorActor nextActor() { return nextActor; }
        public int numLocalPrimes() { return numLocalPrimes; }

        @Override
        public void process(final Object msg) {
            final int candidate = (Integer) msg;
            if (candidate <= 0) {
                return;
            } else {
                final boolean locallyPrime = isLocalPrime(candidate);
                if (locallyPrime) {
                    if (numLocalPrimes <= MAX_LOCAL_PRIME) {
                        localPrimes.add(candidate);
                        numLocalPrimes += 1;
                    } else if (nextActor == null) {
                        nextActor = new SieveActorActor(candidate);
                    } else {
                        nextActor.send(msg);
                    }
                }
            }
        }

        /**
         Using Collection stream to filter the number that are not the multiple of current Actor's base number.
         */
        private boolean isLocalPrime(final int candidate) {
            return localPrimes.stream().noneMatch(prime -> candidate % prime == 0);
        }
    }
}

```