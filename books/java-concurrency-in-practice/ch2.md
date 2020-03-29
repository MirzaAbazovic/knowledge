[home](index.md)

# Chapter 2 Thread safety

## What is thread safety

Writing thread-safe code is, at its core, about **managing access to** state, and in particular to **shared, mutable state**.
An **object’s state** encompasses any data that can affect its externally visible behavior.
By **shared**, we mean that a variable could be accessed by multiple threads
by **mutable**, we mean that its value could change during its lifetime.

Making an object thread-safe requires using synchronization to coordinate access to its mutable state.

Two types of problems arise when multiple threads try to read and write shared data concurrently

1. Thread interference errors (Race conditions)
2. Memory consistency errors 

When multiple threads share the same memory, there is a chance that two or more different threads performing different operations on the same data interleave with each other and create inconsistent data in the memory.

In multithreading, there can be possibilities that the changes made by one thread might not be visible to the other threads and they all have inconsistent views of the same shared data. This is known as memory consistency error.

The primary mechanism for synchronization in Java is the **synchronized** keyword, which provides exclusive locking, but the term “synchronization” also includes the use of **volatile variables**, **explicit locks**, and **atomic variables**.

If multiple threads access the same mutable state variable without appropriate synchronization, your program is broken. There are three ways to fix it:

1. Don’t share the state variable across threads.
2. Make the state variable immutable.
3. Use synchronization whenever accessing the state variable.

It is far easier to design a class to be thread-safe than to retrofit it for thread safety later.

The same object-oriented techniques that help you write well-organized, maintainable classes - such as encapsulation and data hiding - can also help you create thread-safe classes.
The less code thathas access to a particular variable, the easier it is to ensure that all of it uses the proper synchronization,

When designing thread-safe classes, good object-oriented techniques: encapsulation, immutability, and clear specification of invariants—are your best friends.

**First to make your code right, and then make it fast.**
Even then, pursue optimization only if your performance measurements and requirements tell you that you must.

Concurenrency bugs are so difficult to reproduce and debug, the benefit of a small performance gain on some infrequently used code path may well be dwarfed by the risk that the program will fail in the field.

**"thread-safe class" and "thread-safe program"** - a program that consists entirely of thread-safe classes may not be thread-safe, and a thread-safe program may contain classes that are not thread-safe

At the heart of definition of thread safety is the concept of **correctness**.
Correctness means that a **class conforms to its specification**. 
A good specification defines **invariants constraining an object’s state and postconditions describing the effects of its operations.**

Single-threaded correctness is something that "we know it when we see it". 

**Class is thread-safe** if it **behaves correctly when accessed from multiple threads**, regardless of the scheduling or interleaving of the execution of those threads by the runtime environment, and with no additional synchronization or other coordination on the part of the calling code.

Thread-safe class as one that is no more broken in a concurrent environment than in a single-threaded environment.

**Thread-safe classes encapsulate** any needed **synchronization** so that clients need not provide their own.

```java
@ThreadSafe
public class StatelessFactorizer implements Servlet {
    public void service(ServletRequest req, ServletResponse resp) {
        BigInteger i = extractFromRequest(req);
        BigInteger[] factors = factor(i);
        encodeIntoResponse(resp, factors);
    }
}
```

StatelessFactorizer is, like most servlets, **stateless**: it has **no fields and references no fields from other classes**. The **transient state** for a particular computation **exists** solely **in local variables** that are **stored on the thread’s stack** and are **accessible only to the executing thread**.

**Stateless objects are always thread-safe.**

## Atomicity

Add one element of state to what was a stateless object - add a "hit counter"

```java
@NotThreadSafe
public class UnsafeCountingFactorizer implements Servlet {
    private long count = 0;
    public long getCount() { return count; }
    public void service(ServletRequest req, ServletResponse resp) {
        BigInteger i = extractFromRequest(req);
        BigInteger[] factors = factor(i);
        ++count;
        encodeIntoResponse(resp, factors);
    }
}
```

UnsafeCountingFactorizer is not thread-safe.

While the **increment operation, ++count,** may look like a single action because of its compact syntax, it **is not atomic** - does not **execute as a single, indivisible operation**. It is executed as **read-modify-write**.

If the counter is initially 9, with some **unlucky timing** each thread could read the value, see that it is 9, add one to it, and each set the counter to 10.

The possibility of **incorrect results** in the presence of **unlucky timing** is so important in concurrent programming that it has a name: **a race condition**.

### Race conditions

A **race condition** occurs when the **correctness of a computation depends** on the **relative timing** or **interleaving of multiple threads by the runtime**; in other words, when **getting the right answer relies on lucky timing**.

The most common type of race condition is **check-then-act**, where a **potentially stale observation is used to make a decision on what to do next**.

The term **race condition** is often confused with the related term **data race**, which arises when **synchronization is not used to coordinate all access to a shared nonfinal field**. You risk a **data race** whenever a **thread writes a variable that might next be read by another thread or reads a variable that might have last been written by another thread** if both threads do not use synchronization; code with data races has no useful defined semantics under the Java Memory Model. **Not all race conditions are data races, and not all data races are race conditions,** but they both can cause concurrent programs to fail in unpredictable ways. **UnsafeCountingFactorizer has both race conditions and data races.** 

Example: Two friends meeting at 12:00 at Starbucks on University Avenue (when there are 2 Starbucks).

This type of race condition is called **check-then-act**: you observe something to be true (file X doesn’t exist) and then take action based on that observation (create X); but in fact the observation could have become invalid between the time you observed it and the time you acted on it (someone else created X in the meantime), causing a problem (unexpected exception, overwritten data, file corruption).

### Example race conditions in lazy initialization

```java
@NotThreadSafe
public class LazyInitRace {
    private ExpensiveObject instance = null;
    public ExpensiveObject getInstance() {
        if (instance == null)
            instance = new ExpensiveObject();
        return instance;
    }
}
```

A common idiom that uses **check-then-act** is **lazy initialization**. The goal of lazy initialization is to **defer initializing an object until it is actually needed** while at the same time ensuring that it is initialized only once.This is also case with **singleton pattern**.

The hit-counting operation in **UnsafeCountingFactorizer has another sort of race condition**. **Read-modify-write** operations, **like incrementing a counter**, **define a transformation of an object’s state in terms of its previous state**. 
To increment a counter, you have to know its previous value and make sure no one else changes or uses that value while you are in mid-update.

Like most concurrency errors, **race conditions don’t always result in failure: some unlucky timing is also required.**

### Compound actions

**Both LazyInitRace and UnsafeCountingFactorizer contained a sequence of operations that needed to be atomic**, or indivisible, relative to other operations on the same state.

Operations A and B are atomic with respect to each other if, from the perspective of a thread executing A, when another thread executes B, either all of B has executed or none of it has.

An atomic operation is one that is atomic with respect to all operations, including itself, that operate on the same state.

**To ensure thread safety, check-then-act operations (like lazy initialization) and read-modify-write operations (like increment) must always be atomic.**

```java
@ThreadSafe
public class CountingFactorizer implements Servlet {
    private final AtomicLong count = new AtomicLong(0);
    public long getCount() {
        return count.get();
    }
    public void service(ServletRequest req, ServletResponse resp) {
        BigInteger i = extractFromRequest(req);
        BigInteger[] factors = factor(i);
        count.incrementAndGet();
        encodeIntoResponse(resp, factors);
    }
}
```

The **java.util.concurrent.atomic** package **contains atomic variable classes** for effecting atomic state transitions on numbers and object references.

By replacing the **long** counter with an **AtomicLong** , we ensure that all actions that access the counter state are atomic.

Because the state of the servlet is the state of the counter and the counter is thread-safe, our servlet is once again thread-safe.

**When a single element of state is added to a stateless class, the resulting class will be thread-safe if the state is entirely managed by a thread-safe object.**

Use existing thread-safe objects, like AtomicLong, to manage your class’s state.

It is simpler to reason about the possible states and state transitions for existing thread-safe objects than it is for arbitrary state variables, and this makes it easier to maintain and verify thread safety.

## Locking

Let us improve the performance of our servlet by caching the most recently computed result.

```java
@NotThreadSafe
public class UnsafeCachingFactorizer implements Servlet {
    private final AtomicReference<BigInteger> lastNumber = new AtomicReference<BigInteger>();
    private final AtomicReference<BigInteger[]> lastFactors = new AtomicReference<BigInteger[]>();

    public void service(ServletRequest req, ServletResponse resp) {
        BigInteger i = extractFromRequest(req);
        if (i.equals(lastNumber.get()))
            encodeIntoResponse(resp, lastFactors.get()); 
        else {
            BigInteger[] factors = factor(i);
            lastNumber.set(i);
            lastFactors.set(factors);
            encodeIntoResponse(resp, factors);
        }
    }
}
```

When multiple variables participate in an invariant, they are not independent: the value of one constrains the allowed value(s) of the others.
Thus when updating one, you must update the others in the same atomic operation.

Using atomic references, we cannot update both lastNumber and lastFactors simultaneously, even though each call to set is atomic; there is still a **window of vulnerability** when one has been modified and the other has not, and during that time other threads could see that the invariant does not hold.

**To preserve state consistency, update related state variables in a single
atomic operation.**s

### Intrisitc locks

Java provides a **built-in locking mechanism for enforcing atomicity**: the **synchronized block**.

A **synchronized block** has **two parts**: a **reference to an object that will serve as the lock**, and a **block of code to be guarded by that lock**. 

A **synchronized method** is a shorthand for a **synchronized block that spans an entire method body**, and whose **lock is the object on which the method is being invoked**.(Static synchronized methods use the Class object for the lock.)

```java
synchronized (lock) {
    // Access or modify shared state guarded by lock
}
```

**Every Java object can implicitly act as a lock** for purposes of synchronization; these built-in locks are called **intrinsic locks or monitor locks**.


The lock is **automatically acquired** by the **executing thread before entering** a **synchronized block** and **automatically released** when control **exits the synchronized block**, whether by the **normal control path** or by **throwing an exception out of the block**.


Intrinsic locks in Java act as **mutexes (or mutual exclusion locks)**, which means that at **most one thread may own the lock.**

```java
@ThreadSafe
public class SynchronizedFactorizer implements Servlet {
    @GuardedBy("this") private BigInteger lastNumber;
    @GuardedBy("this") private BigInteger[] lastFactors;

    public synchronized void service(ServletRequest req, ServletResponse resp) {
        BigInteger i = extractFromRequest(req);
        if (i.equals(lastNumber))
            encodeIntoResponse(resp, lastFactors);
        else {
            BigInteger[] factors = factor(i);
            lastNumber = i;
            lastFactors = factors;
            encodeIntoResponse(resp, factors);
            }
    }
}
```

In SynchronizedFactorizer, **lastNumber and lastFactors are guarded** by the **servlet object’s intrinsic lock**; this is documented by the **@GuardedBy annotation**.

Listing above makes the **service method synchronized** , so **only one thread may enter service at a time**. This approach is **fairly extreme**, since it inhibits multiple clients from using the factoring servlet simultaneously at all - resulting in **unacceptably poor responsiveness**. This problem which is a **performance problem**, not a **thread safety problem**.

### Reentrancy

But because **intrinsic locks are reentrant**, if a **thread tries to acquire a lock that it already holds, the request succeeds**. Reentrancy means that **locks are acquired on a per-thread rather than per-invocation basis**, this differs from the default locking behavior for pthreads (POSIX threads) mutexes, which are granted on a per-invocation basis.

Reentrancy is implemented by **associating with each lock an acquisition count and an owning thread**. When the **count is zero**, the **lock** is considered **unheld**. When a **thread acquires** a previously unheld **lock**, the **JVM records the owner** and sets the **acquisition count to one**. If that **same thread acquires the lock again**, the **count is incremented**, and when the **owning thread exits the synchronized block, the count is decremented**. When the **count reaches zero, the lock is released**.

**Reentrancy facilitates encapsulation of locking behavior**, and thus **simplifies the development of object-oriented concurrent code.** 

Without reentrant locks, the very natural looking code in Listing below, in which a subclass overrides a synchronized method and then calls the superclass method, would deadlock.

```java
public class Widget {
    public synchronized void doSomething() {
        ...
    }
}

public class LoggingWidget extends Widget {

    public synchronized void doSomething() {
        System.out.println(toString() + ": calling doSomething");
        super.doSomething();
    }
}
```

Reentrancy saves us from deadlock in situations like this.


## Guarding state with locks

Because **locks enable serialized access to the code paths they guard**, we can use them to construct protocols for **guaranteeing exclusive access to shared state**.

**Compound actions on shared state**, such as incrementing a hit counter **(read-modify-write)** or lazy initialization **(check-then-act)**, **must be made atomic to avoid race conditions**. 

Holding a lock for the entire duration of a compound action can make that compound action atomic. 

However, just **wrapping the compound action with a synchronized block is not sufficient; if synchronization is used to coordinate access to a variable, it is needed everywhere that variable is accessed**.

Further, when using locks to coordinate access to a variable, the **same lock must be used wherever that variable is accessed.**

For each mutable state variable that may be accessed by more than one thread, **all accesses** to that **variable** must be **performed with the same lock held**. In this case, we say that the **variable is guarded by that lock**.

The fact that **every object has a built-in lock** is just a **convenience so that you needn’t explicitly create lock objects**. 
It is **up to you to construct locking protocols or synchronization policies** that let you **access shared state safely**, and to **use them consistently** throughout your program.

Every shared, mutable variable should be guarded by exactly one lock. Make it clear to maintainers which lock that is.

A **common locking convention is to encapsulate all mutable state within an object** and to **protect it** from concurrent access **by synchronizing** any code path that accesses mutable state **using the object’s intrinsic lock**.

This pattern is used by many thread-safe classes, such as Vector and other synchronized collection classes. In such cases, all the variables in an object’s state are guarded by the object’s intrinsic lock. 

It is also easy to **subvert this locking protocol accidentally** by **adding a new method** or code path and **forgetting to use synchronization**.

Not all data needs to be guarded by locks - only mutable data that will be accessed from multiple threads.

When a **variable is guarded by a lock** - meaning that **every access to that variable is performed with that lock held** - you’ve ensured that only one thread at a time can access that variable. When a **class has invariants that involve more than one state variable**, there is an additional requirement: **each variable participating in the invariant must be guarded by the same lock**.

**For every invariant that involves more than one variable, all the variables involved in that invariant must be guarded by the same lock.**

Indiscriminate application of synchronized might be either too much or too little synchronization. 
Synchronizing every method can lead to liveness or performance problems.

## Liveness and performance

It is easy to improve the concurrency while maintaining thread safety by narrowing the scope of the synchronized block.
You should be careful not to make the scope of the synchronized block too small; 
you would not want to divide an operation that should be atomic into more than one synchronized block.
It is reasonable to try to exclude from synchronized blocks long-running operations that do not affect shared state, so that other threads are not prevented from accessing the shared state while the long-running operation is in progress.
```java
@ThreadSafe
public class CachedFactorizer implements Servlet {
    @GuardedBy("this") private BigInteger lastNumber;
    @GuardedBy("this") private BigInteger[] lastFactors;
    @GuardedBy("this") private long hits;
    @GuardedBy("this") private long cacheHits;
    
    public synchronized long getHits() { 
        return hits; 
    }
    public synchronized double getCacheHitRatio() {
        return (double) cacheHits / (double) hits;
    }
    public void service(ServletRequest req, ServletResponse resp) {
        BigInteger i = extractFromRequest(req);
        BigInteger[] factors = null;
   
        synchronized (this) {
            ++hits;
            if (i.equals(lastNumber)) {
                ++cacheHits;
                factors = lastFactors.clone();
            }
        }
   
        if (factors == null) {

            factors = factor(i);

            synchronized (this) {
                lastNumber = i;
                lastFactors = factors.clone();
            }
        }
        encodeIntoResponse(resp, factors);
    }
}
```

Atomic variables are useful for effecting atomic operations on a single variable, but since we are already using synchronized blocks to
construct atomic operations, using two different synchronization mechanisms would be confusing and would offer no performance or safety benefit.
The restructuring of CachedFactorizer provides a **balance between simplicity (synchronizing the entire method) and concurrency (synchronizing the shortest possible code paths)**. **Acquiring and releasing a lock has some overhead**, so it is undesirable to break down synchronized blocks too far (such as factoring ++hits into its own synchronized block), even if this would not compromise
atomicity.

When implementing a synchronization policy, **resist the temptation to prematurely sacrifice simplicity (potentially compromising safety) for the sake of performance**.

**Avoid holding locks during lengthy computations or operations at risk of not completing quickly** such as network or console I/O.


[home](index.md)
