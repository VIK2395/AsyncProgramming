# Atomic Operations

**Atomic operations** refer to operations that are guaranteed to be indivisible, meaning they either complete entirely or not at all. This ensures that no other thread can interfere with the operation while it is executing, which is critical for maintaining consistency in concurrent programming.

In multithreading, atomic operations prevent **race conditions**, where multiple threads try to access and modify shared data at the same time, leading to unpredictable or erroneous behavior.

---

### What Makes an Operation Atomic?

1. **Indivisibility**: Once an atomic operation starts, it runs to completion without interruption.
2. **No Intermediate States**: The operation cannot be paused or observed in a halfway state.
3. **Consistency**: The operation ensures that data remains in a consistent state after completion.

Atomic operations are often supported directly by the underlying hardware (e.g., CPU instructions) to ensure thread safety without the overhead of more complex synchronization mechanisms like locks.

# Critical Section

A **critical section** is a part of a program where shared resources (such as variables, data structures, or hardware devices) are accessed or modified by multiple threads or processes. The critical section is "critical" because if more than one thread enters it simultaneously, it can lead to **data corruption** or **inconsistent results**. To prevent this, access to the critical section must be **synchronized**, meaning only one thread can execute it at a time.

### Problems Caused by Concurrent Access to Critical Sections

When multiple threads or processes try to modify shared resources concurrently without proper synchronization, it can result in **race conditions**, which cause unpredictable behavior. For example:
- **Data inconsistency**: Two threads may read and write data in conflicting ways, leading to unexpected results.
- **Corrupted shared resources**: Shared data structures (e.g., lists, arrays) can be left in inconsistent states if not properly protected.

To prevent these issues, we need to ensure that only one thread at a time can execute a critical section.

### Synchronizing Access to a Critical Section

To prevent multiple threads from entering a critical section simultaneously, we use synchronization mechanisms such as **locks** or **mutexes**.

#### Key Mechanisms for Managing Critical Sections:

- **`lock` keyword in C#. It is a syntactic sugar for Monitor**
   - Ensures that only one thread can enter the critical section at a time.
- **`Monitor` class inC#**
  - It allows threads to acquire and release locks manually.
  - Ensures that only one thread can enter the critical section at a time.
- **`Mutex` (short for "mutual exclusion")**
  - A **`Mutex`** is similar to a lock but can be used across processes, not just within a single application. This is useful when multiple applications or services need to synchronize access to a resource.
  - **The thread that acquires the lock must be the one to release it.**
  - It is OS level realization.
  - Can contribute to deadlock if multiple threads acquire different mutexes in different orders.
- **`Semaphore`**
  - A **`Semaphore`** allows you to control access to a resource by limiting the number of threads that can access it at the same time. It is commonly used when you want to allow multiple threads to access a resource, but only up to a certain limit.
  - Any thread that has successfully entered the semaphore (by calling WaitOne) can release the semaphore lock, including the thread that originally acquired the semaphore or another thread that is finished with the resource.
    - **Counting Semaphore vs. Binary Semaphore**
      - A **counting semaphore** is used when you need to allow multiple threads to access a resource, but with a limit on the number of threads that can enter the critical section.
        - **It doesn't prevent race conditions**
        -  deadlocks can occur. The risk of deadlock is lower compared to mutexes.
      - A **binary semaphore** is a special case of a counting semaphore where the semaphore's value can only be either 0 or 1. Essentially, it behaves like a mutex in that it ensures that only one thread can enter the critical section at a time.

### Summary of Synchronization Mechanisms in C#:

| Mechanism     | Scope              | Use Case                                          | Example                                                                 |
|---------------|--------------------|---------------------------------------------------|-------------------------------------------------------------------------|
| `lock`        | Single application | Simple, thread-level synchronization              | `lock (object) { // Critical section }`                                 |
| `Monitor`     | Single application | Fine-grained control over locking and signaling   | `Monitor.Enter(lockObj); Monitor.Exit(lockObj);`                        |
| `Mutex`       | Cross-process      | Synchronizing across multiple applications        | `mutex.WaitOne(); mutex.ReleaseMutex();`                                 |
| `Semaphore`   | Single or multiple threads | Limiting access to a resource by multiple threads | `semaphore.WaitOne(); semaphore.Release();`                              |

# Race condition

A race condition is a type of concurrency bug that occurs when multiple threads or processes attempt to modify shared data simultaneously, and the outcome depends on the order or timing of execution. This can lead to unpredictable or incorrect behavior, especially when one thread's operations interfere with another's in an unintended way.

# Dead lock

A deadlock is a situation in concurrent programming where two or more threads (or processes) are blocked forever, waiting for each other to release resources. This results in a cyclic dependency where each thread is holding one resource and waiting for another resource that is held by another thread. As a result, none of the threads can proceed, and the program becomes stuck in a state of permanent waiting.
