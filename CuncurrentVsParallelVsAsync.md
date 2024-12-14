# Concurrent vs Parallel vs Asynchronous programming

### Parallel programming (REAL PARALLELISM)

- Parallel programming involves executing **multiple tasks simultaneously/in parallel** on multiple threads or cores.
- Node.js(V8) is single-threaded. But parallelism can be achieved using Worker Threads.

### Concurrent programming (TASK-SWITCHING)

- Concurrent programming involves multiple tasks being executed on the same thread, swithching time between them.
- Node.js' V8 thread excecutes tasks "received" from event loop. So the event loop implicitly "prforms swithching between tasks" running on the thread.
- "Fake parallelism"

### Asynchronous programming (NON-BLOCKING)

- Asynchronous programming involves non-blocking operations that **don't block the program's execution** while waiting for tasks(e.g., I/O) to complete. Tasks are handled in the background, and their results are returned later.
- Even though Node.js (V8) is single-threaded, I/O are delegated to the Underlying System(e.g., the OS or libuv library). The Underlying System performs I/O operations in parallel using multiple threads or asynchronous system calls.
- Once the underlying system completes the task, it notifies Node.js via callbacks or Promises so that the results can be planned/processed in the main thread. The underlying system pushes results to event loop.
