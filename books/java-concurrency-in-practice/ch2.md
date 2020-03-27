[home](index.md)

# Chapter 2 Thread safety

Writing thread-safe code is, at its core, about **managing access to** state, and in particular to **shared, mutable state**.
An **object’s state** encompasses any data that can affect its externally visible behavior.
By **shared**, we mean that a variable could be accessed by multiple threads
by **mutable**, we mean that its value could change during its lifetime.

Making an object thread-safe requires using synchronization to coordinate access to its mutable state.

The primary mechanism for synchronization in Java is the **synchronized** keyword, which provides exclusive locking, but the term “synchronization” also includes the use of **volatile variables**, **explicit locks**, and **atomic variables**.

If multiple threads access the same mutable state variable without appropriate synchronization, your program is broken. There are three ways to fix it:

1. Don’t share the state variable across threads.
2. Make the state variable immutable.
3. Use synchronization whenever accessing the state variable.

It is far easier to design a class to be thread-safe than to retrofit it for thread safety later.

The same object-oriented techniques that help you write well-organized, maintainable classes - such as encapsulation and data hiding - can also help you create thread-safe classes.
The less code thathas access to a particular variable, the easier it is to ensure that all of it uses the proper synchronization,

When designing thread-safe classes, good object-oriented techniques: encapsulation, immutability, and clear specification of invariants—are your best friends.

First to make your code right, and then make it fast.
Even then, pursue optimization only if your performance measurements and requirements tell you that you must.

Concurenrency bugs are so difficult to reproduce and debug, the benefit of a small performance gain on some infrequently used code path may well be dwarfed by the risk that the program will fail in the field.


"thread-safe class" and "thread-safe program" - a program that consists entirely of thread-safe classes may not be thread-safe, and a thread-safe program may contain classes that are not thread-safe

[home](index.md)
