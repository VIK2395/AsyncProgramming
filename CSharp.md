# async await

C# is synchronous, but we can perform asynchronous task.

- We can mark methods with async keyword.
- Async methods are intended to be non-bloking operations.
- `async` and `await` keywords don't cause additional threads to be created. The method runs on currernt synchronization context and uses the calling thread (good for I/O operations as they do not require CPU and we only wait for the result to become available).
  - We can use Task.Run to move CPU-bound work to background thread.
- We should always use `await` in async methods. When `await` is meet the control returns the method that called the async method. If no await is used no any benefits and the method would work as usual sync method.
- `ConfigureAwait(true)`. Default true. Configures for **the same** synchronization contex/thread to be used **to resume the async method tail after `await`** that created the task. `ConfigureAwait(false)` - says the tail can continue in any synchronization contex/thread from thread pool.
  - `ConfigureAwait(true)` important for the UI thread as only UI thread has access to the UI elements.
- Console application, like web serves don't have `synchronization contex`. It is null. So `ConfigureAwait` has no effect in them and behave as if `ConfigureAwait(false)`.
- Under the hood a state machine is used to handle async wait method execution.
