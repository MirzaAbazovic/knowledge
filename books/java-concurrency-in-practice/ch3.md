[home](index.md)

# Chapter 3 Sharing objects

This chapter examines techniques for sharing and publishing objects so they can be safely accessed by multiple threads.

Sharing and publishing object add managing access to shared, mutable state using the java.util.concurrent library classes the foundation for building thread-safe classes and safely structuring concurrent applications.

We want not only to prevent one thread from modifying the state of an object when another is using it, but also to ensure that **when a thread modifies the state of an object, other threads can actually see the changes that were made**.

## Visibility

Sharing variables without synchronization. Don’t do this.

```java
public class NoVisibility {
    private static boolean ready;
    private static int number;
    
    private static class ReaderThread extends Thread {
        public void run() {
            while (!ready)
                Thread.yield();
            System.out.println(number);
        }
    }
    
    public static void main(String[] args) {
        new ReaderThread().start();
        number = 42;
        ready = true;
    }
}
```

**NoVisibility** could **loop forever** because the value of ready might never become visible to the reader thread. 
Even more strangely, **NoVisibility could print zero** because the write to ready might be made visible to the reader thread before the write to number, a phenomenon known as **reordering**.

There is no guarantee that operations in one thread will be performed in the order given by the program, as long as the reordering is not detectable from within that thread - even if the reordering is apparent to other threads.
In the **absence of synchronization**, the **Java Memory Model permits the compiler to reorder operations and cache values in registers**, and **permits CPUs to reorder operations and cache values in processor-specific caches**.

In the absence of synchronization, the compiler, processor, and runtime can do some downright weird things to the order in which operations ap pear to execute.

Attempts to reason about the order in which memory actions “must” happen in insufficiently synchronized multithreaded programs will almost certainly be incorrect.

Easy way to **avoid these complex issues**: **always use the proper synchronization whenever data is shared across threads**.




## Publication and escape

## Thread Confinement

## Immutability

## Safe publication

[home](index.md)
