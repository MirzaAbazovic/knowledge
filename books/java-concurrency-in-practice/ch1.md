[home](index.md)

# Chapter 1 Introduction

Let me paraphrase Albert Allen Bartlett "The greatest shortcoming of the developers is our inability to understand threding".

Threading-related problems are hard to detect, bugs they produce do not manifest themselfs predictably, so they (often) appear in production.

There are low-level mechanisims and design-level policies.

"The language provides low-level mechanisms such as synchronization and condition waits, but these
mechanisms must be used consistently to implement application-level protocols or policies."

"To address the abstraction mismatch between Java’s low-level mechanisms and the necessary design-level policies, we present a simplified set of rules for writing concurrent programs."

Threads are the easiest way to tap the computing power of multiprocessor systems.

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

Sequential programming is intuitive and natural. 
Finding the right balance of sequentiality and asyncrony is often characteristic of efficient programs.
 
Since the basic unit of scheduling is thread on 2 CPU single-threaded program is giving up 50% of resources.
Even on single CPU multi-threaded program achieve better throughput by not waiting for blocking I/O operations.

Group jobs in similar/same batches and execute them 

Single threaded non-blocking I/O vs Multi-threading (MT)

MT One thread per request(user) (Java Servlet)
One thread for all using non blocking I/O (server side javascript - node)

Maybe truth is in the middle -> 
some hybrid model: multi-threaded with single thread for set of requests (users) in combination with non blocking I/O (VErtex)
   
 
[home](index.md)
