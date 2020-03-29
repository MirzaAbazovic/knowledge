[home](index.md)

# Chapter 1 Introduction

Let me paraphrase Albert Allen Bartlett "The greatest shortcoming of the developers is our inability to understand threding".

Threading-related problems are hard to detect, bugs they produce do not manifest themselfs predictably, so they (often) appear in production.

There are low-level mechanisims and design-level policies.

"The language provides low-level mechanisms such as synchronization and condition waits, but these
mechanisms must be used consistently to implement application-level protocols or policies."

When we say - "multiple tasks are executing at the same time", what we actually mean is that "multiple tasks are making progress during the same period of time."

The tasks are executed in an interleaved manner. The operating system switches between the tasks so frequently that it appears to the users that they are being executed at the same physical instant.

Therefore, Concurrency does not mean Parallelism. In fact, Parallelism is impossible on a single processor system.

"To address the abstraction mismatch between Java’s low-level mechanisms and the necessary design-level policies, we present a simplified set of rules for writing concurrent programs."

Threads are the easiest way to tap the computing power of multiprocessor systems.

Every process has at least one thread - called the main thread. The main thread can create additional threads within the process.

Threads within a process share the process’s resources including memory and open files. However, every thread has its own call stack. Since threads share the same address space of the process, creating new threads and communicating between them is more efficient.

Motivations for simultaneous processes and threads (aka lightweight process) also are:

- Resource utilisation (Give process that can do work resource.)
- Fairness (Users and programs share resources by time slicing.)
- Convenience (More programs coordinated instead one.)

Difference between processes and threads are that threads share process-wide resources: memory, file handles,
but each thread has his own program counter, stack and local variables. 
Processes communicate using sockets, signal handlers, shared memory, semaphores and files.  
Threads exploit hardware parallelism on multi proc. sys. -> multiple threads can be scheduled simultaneously on multiple CPUs.

Threads are basic unit of scheduling and are executing simultaneously or asynchronously.
Threads have same variables and allocate objects from the same heap.

Unit of concurrency:

Concurrency is a very broad term, and it can be used at various levels. For example:

- Multiprocessing - Multiple Processors/CPUs executing concurrently. The unit of concurrency here is a CPU.
- Multitasking - Multiple tasks/processes running concurrently on a single CPU. The operating system executes these tasks by switching between them very frequently. The unit of concurrency, in this case, is a Process.
- Multithreading - Multiple parts of the same program running concurrently. In this case, we go a step further and divide the same program into multiple parts/threads and run those threads concurrently.


Sequential programming is intuitive and natural.
Finding the right balance of sequentiality and asyncrony is often characteristic of efficient programs.
 
Since the basic unit of scheduling is thread on 2 CPU single-threaded program is giving up 50% of resources.
Even on single CPU multi-threaded program achieve better throughput by not waiting for blocking I/O operations.

Group jobs in similar/same batches and execute them.

Single threaded non-blocking I/O vs Multi-threading (MT)

MT One thread per request(user) (Java Servlet)
One thread for all using non blocking I/O (server side javascript - node)

Maybe truth is in the middle -> 
some hybrid model: multi-threaded with single thread for set of requests (users) in combination with non blocking I/O (VErtex)

Modern GUI frameworks, such as the AWT and Swing toolkits, replace the main event loop with an event dispatch thread (EDT).

Most GUI frameworks are single-threaded subsystems, so the main event loop is effectively still present, but it runs in its own thread under the control of the GUI toolkit rather than the application.

If, the long-running task is executed ina separate thread, the event thread remains free to process UI events, making the UI more responsive.

### Safety hazards

When threads were more esoteric, concurrency was an “advanced” topic; now, mainstream developers must be aware of thread-safety issues.

```java
@NotThreadSafe
public class UnsafeSequence {
    private int value;
    /** Returns a unique value. */
    public int getNext() {
    return value++;
    }
}
```

The increment notation, someVariable++ , may appear to be a single operation, but is in fact three separate operations: read the value, add one to it, and write out the new value.

Operations in multiple threads may be arbitrarily interleaved by the runtime, it is possible for two threads to read the value at the same time, both see the same value, and then both add one to it.
The result is that the same sequence number is returned from multiple calls in different threads.

If a class is annotated with @ThreadSafe , users can use it with confidence in a multithreaded environment, maintainers are put on notice that it makes thread safety guarantees that must be preserved, and software analysis tools can identify possible coding errors.

Common concurrency hazard is **race condition**.

Because threads share the same memory address space and run concurrently, they can access or modify variables that other threads might be using.

This is a **tremendous convenience**, because it makes **data sharing much easier** than would other inter-thread communications mechanisms. But it is also a **significant risk**: threads can be confused by having **data change unexpectedly/unpredictably**.

Allowing **multiple threads** to access and modify the same variables **introduces an element of nonsequentiality into** an otherwise **sequential programming model**, which can be confusing and **difficult to reason about**.

Java provides synchronization mechanisms to coordinate such access.
Making getNext a synchronized method.

```java
@ThreadSafe
public class Sequence {
    @GuardedBy("this")
    private int value;

    public synchronized int getNext() {
        return value++;
        }
}
```

Compiler, hardware, and runtime are allowed to take substantial liberties with the timing and ordering of actions, such as caching variables in registers or processor-local caches where they are temporarily (or even permanently) invisible to other threads.

### Liveness hazards

When developing concurrent code: **safety cannot be compromised**.
Use of threads introduces additional safety hazards not present in single-threaded programs and introduces additional forms of **liveness failure** that do not occur in single-threaded programs

While safety means "nothing bad ever happens", liveness concerns the complementary goal that "something good eventually happens".

A liveness failure occurs when an activity gets into a state such that it is permanently unable to make forward progress. In single-threaded programs inifinite loop is example of liveness failure in multi-threaded programs some forms of liveness failures are:
- deadlock
- starvation
- livelock

Most concurrency bugs that cause liveness failures, do not always manifest themselves in development or testing.

### Performace hazards

While liveness means that something good eventually happens, eventually may not be good enough - we often want good things to happen quickly
Performance issues subsume a broad range of problems including poor service time, responsiveness, throughput, resource consumption, or scalability.

Context switches are more frequent in applications with many threads, and have significant costs: saving and restoring execution context, loss of locality, and CPU time spent scheduling threads instead of running them

When threads share data, they must use synchronization mechanisms that can  compiler optimizations, flush or invalidate memory caches, and create synchronization traffic on the shared memory bus.


## Threads are everywhere

Every Java application uses threads. When the JVM starts, it creates threads for JVM housekeeping tasks (garbage collection, finalization) and a main thread for running the main method.

The AWT (Abstract Window Toolkit) and Swing user interface frameworks create threads for managing user interface events.

Timer creates threads for executing deferred tasks. Component frameworks, such as servlets and RMI create pools of threads and invoke component methods in these threads.

[home](index.md)
