# async await

C# is synchronous, but we can perform asynchronous tasks.

- We can mark methods with async keyword.
- Async methods are intended to be non-bloking operations.
- `async` and `await` keywords don't cause **additional threads to be created**. The method runs on **currernt synchronization context** and the synchronization context decides where to continue excecuting the tail(if SynchronizationContext.Current == null, then the continuation of the async method is scheduled to the thread pool via the default `TaskScheduler`).
  - I/O operations itself do not require a CPU/thread - we only wait for the result to become available. So the thread is **released/freed-up/yields control and returns to the caller** to do other work instead of waiting for the I/O to complete. The OS performs the actual I/O operation. The I/O operation itself is offloaded to the operating system and is handled without consuming any threads. When the I/O completes, the OS notifies the runtime via an event or callback mechanism.
  - We can use Task.Run to move CPU-bound work to background thread.
- We should always use `await` in async methods. When `await` is meet the control returns to the method that called the async method. If no `await` is used no any benefits and the method would work as sync method. Without `await` before called async method, the calling method continue execution not waiting for the called async method to return result.  
- `ConfigureAwait(true)`. Default true. Configures for **the same** synchronization contex/thread to be used **to resume the async method tail after `await`** that created the task. `ConfigureAwait(false)` - says the tail can continue in any synchronization contex/thread from thread pool.
  - `ConfigureAwait(true)` important for the UI thread as only UI thread has access to the UI elements.
- Console application, like web serves don't have `synchronization contex`. It is null. So `ConfigureAwait` has no effect in them and behave as if `ConfigureAwait(false)`.
- Under the hood a state machine is used to handle async wait method execution.

```c#
namespace ConsoleApp
{
    internal class Program
    {
        static async Task Main(string[] args)
        {
            Console.WriteLine($"SynchronizationContext.Current {SynchronizationContext.Current?.ToString() ?? "null"}");
            Console.WriteLine($"ThreadId1 {Thread.CurrentThread.ManagedThreadId}");
            Console.WriteLine("Main. Before DoSomethingAsync");
            await DoSomethingAsync();
            Console.WriteLine($"ThreadId6 {Thread.CurrentThread.ManagedThreadId}");
            Console.WriteLine("Main. After DoSomethingAsync");
        }

        static async Task DoSomethingAsync()
        {
            Console.WriteLine("DoSomethingAsync start");
            Console.WriteLine($"ThreadId2 {Thread.CurrentThread.ManagedThreadId}");
            await DoSomethingNestedAsync();
            Console.WriteLine($"ThreadId5 {Thread.CurrentThread.ManagedThreadId}");
            Console.WriteLine("DoSomethingAsync end");
        }

        static async Task DoSomethingNestedAsync()
        {
            Console.WriteLine("DoSomethingNestedAsync start");
            Console.WriteLine($"ThreadId3 {Thread.CurrentThread.ManagedThreadId}");
            await Task.Delay(2000);
            Console.WriteLine($"ThreadId4 {Thread.CurrentThread.ManagedThreadId}");
            Console.WriteLine("DoSomethingNestedAsync end");
        }
    }
}
```
Output
```
SynchronizationContext.Current null
ThreadId1 1
Main. Before DoSomethingAsync
DoSomethingAsync start
ThreadId2 1
DoSomethingNestedAsync start
ThreadId3 1
ThreadId4 6
DoSomethingNestedAsync end
ThreadId5 6
DoSomethingAsync end
ThreadId6 6
Main. After DoSomethingAsync
```

```c#
namespace ConsoleApp
{
    internal class Program
    {
        static async Task Main(string[] args)
        {
            Console.WriteLine($"SynchronizationContext.Current {SynchronizationContext.Current?.ToString() ?? "null"}");
            Console.WriteLine($"ThreadId1 {Thread.CurrentThread.ManagedThreadId}");
            Console.WriteLine("Main. Before DoSomethingAsync");
            DoSomethingAsync(); // HERE NO AWAIT
            Console.WriteLine($"ThreadId6 {Thread.CurrentThread.ManagedThreadId}");
            Console.WriteLine("Main. After DoSomethingAsync");
        }

        static async Task DoSomethingAsync()
        {
            Console.WriteLine("DoSomethingAsync start");
            Console.WriteLine($"ThreadId2 {Thread.CurrentThread.ManagedThreadId}");
            await DoSomethingNestedAsync();
            Console.WriteLine($"ThreadId5 {Thread.CurrentThread.ManagedThreadId}");
            Console.WriteLine("DoSomethingAsync end");
        }

        static async Task DoSomethingNestedAsync()
        {
            Console.WriteLine("DoSomethingNestedAsync start");
            Console.WriteLine($"ThreadId3 {Thread.CurrentThread.ManagedThreadId}");
            await Task.Delay(2000);
            Console.WriteLine($"ThreadId4 {Thread.CurrentThread.ManagedThreadId}");
            Console.WriteLine("DoSomethingNestedAsync end");
        }
    }
}
```
Output
```
SynchronizationContext.Current null
ThreadId1 1
Main. Before DoSomethingAsync
DoSomethingAsync start
ThreadId2 1
DoSomethingNestedAsync start
ThreadId3 1
ThreadId6 1
Main. After DoSomethingAsync
```

# Add synchronization context to console application

Install the Nito.AsyncEx NuGet package
```bash
dotnet add package Nito.AsyncEx
```

```c#
using Nito.AsyncEx;

class Program
{
    static void Main(string[] args)
    {
        // Log the thread ID in the Main method
        Console.WriteLine($"[Main] Thread ID: {Thread.CurrentThread.ManagedThreadId}");

        // Run the async context
        AsyncContext.Run(async () =>
        {
            Console.WriteLine($"[AsyncContext.Run] Thread ID: {Thread.CurrentThread.ManagedThreadId}");
            Console.WriteLine("Starting application...");

            // Call an asynchronous method
            await DoAsyncWork();

            Console.WriteLine($"[AsyncContext.Run] After await - Thread ID: {Thread.CurrentThread.ManagedThreadId}");
            Console.WriteLine("Finished application.");
        });
    }

    static async Task DoAsyncWork()
    {
        Console.WriteLine($"[DoAsyncWork] Before delay - Thread ID: {Thread.CurrentThread.ManagedThreadId}");

        // Simulate asynchronous work
        await Task.Delay(1000);

        Console.WriteLine($"[DoAsyncWork] After delay - Thread ID: {Thread.CurrentThread.ManagedThreadId}");
    }
}
```
Output
```
[Main] Thread ID: 1
[AsyncContext.Run] Thread ID: 1
Starting application...
[DoAsyncWork] Before delay - Thread ID: 1
[DoAsyncWork] After delay - Thread ID: 1
[AsyncContext.Run] After await - Thread ID: 1
Finished application.
```

This demonstrates that `AsyncContext` maintains a single-threaded execution context, **similar to how a UI thread works**.
