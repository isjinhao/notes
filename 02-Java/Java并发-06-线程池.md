---
title: 08-并发
tags:
  - 应届求职复习
  - 面试
categories:
  - 应届求职复习
mathjax: true
abbrlink: 31922
date: 2019-07-28 21:47:55
---



## Callable和Future和FutureTask

### Callable

`Callable` 位于 `java.util.concurrent` 包下，它也是一个接口，在它里面也只声明了一个方法，只不过这个方法叫做 `call()`：

```java
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

可以看到，这是一个泛型接口，`call()` 函数返回的类型就是传递进来的V类型。

那么怎么使用 `Callable` 呢？一般情况下是配合 `ExecutorService` 来使用的，在 `ExecutorService` 接口中声明了若干个 `submit()` 方法的重载版本：

```java
<T> Future<T> submit(Callable<T> task);
<T> Future<T> submit(Runnable task, T result);
Future<?> submit(Runnable task);
```

第一个 `submit` 方法里面的参数类型就是 `Callable`。

暂时只需要知道 `Callable` 一般是和 `ExecutorService` 配合来使用的，具体的使用方法讲在后面讲述。



### Future

`Future` 就是对于具体的 `Runnable` 或者 `Callable` 任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过 `get()` 获取执行结果，该方法会阻塞直到任务返回结果。

`Future` 类位于 `java.util.concurrent` 包下，它是一个接口：

```java
public interface Future<V> {

    /**
     * Attempts to cancel execution of this task.  This attempt will
     * fail if the task has already completed, has already been cancelled,
     * or could not be cancelled for some other reason. If successful,
     * and this task has not started when {@code cancel} is called,
     * this task should never run.  If the task has already started,
     * then the {@code mayInterruptIfRunning} parameter determines
     * whether the thread executing this task should be interrupted in
     * an attempt to stop the task.
     *
     * <p>After this method returns, subsequent calls to {@link #isDone} will
     * always return {@code true}.  Subsequent calls to {@link #isCancelled}
     * will always return {@code true} if this method returned {@code true}.
     *
     * @param mayInterruptIfRunning {@code true} if the thread executing this
     * task should be interrupted; otherwise, in-progress tasks are allowed
     * to complete
     * @return {@code false} if the task could not be cancelled,
     * typically because it has already completed normally;
     * {@code true} otherwise
     */
    boolean cancel(boolean mayInterruptIfRunning);

    /**
     * Returns {@code true} if this task was cancelled before it completed
     * normally.
     *
     * @return {@code true} if this task was cancelled before it completed
     */
    boolean isCancelled();

    /**
     * Returns {@code true} if this task completed.
     *
     * Completion may be due to normal termination, an exception, or
     * cancellation -- in all of these cases, this method will return
     * {@code true}.
     *
     * @return {@code true} if this task completed
     */
    boolean isDone();

    /**
     * Waits if necessary for the computation to complete, and then
     * retrieves its result.
     *
     * @return the computed result
     * @throws CancellationException if the computation was cancelled
     * @throws ExecutionException if the computation threw an
     * exception
     * @throws InterruptedException if the current thread was interrupted
     * while waiting
     */
    V get() throws InterruptedException, ExecutionException;

    /**
     * Waits if necessary for at most the given time for the computation
     * to complete, and then retrieves its result, if available.
     *
     * @param timeout the maximum time to wait
     * @param unit the time unit of the timeout argument
     * @return the computed result
     * @throws CancellationException if the computation was cancelled
     * @throws ExecutionException if the computation threw an
     * exception
     * @throws InterruptedException if the current thread was interrupted
     * while waiting
     * @throws TimeoutException if the wait timed out
     */
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

在 `Future` 接口中声明了5个方法，下面依次解释每个方法的作用：

**cancel()**

用来取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。参数 `mayInterruptIfRunning` 表示是否允许取消正在执行却没有执行完毕的任务，如果设置true，则表示可以取消正在执行过程中的任务。如果任务已经完成，则无论 `mayInterruptIfRunning` 为true还是false，此方法肯定返回false，即如果取消已经完成的任务会返回false；如果任务正在执行，若 `mayInterruptIfRunning` 设置为true，则返回true，若`mayInterruptIfRunning` 设置为false，则返回false；如果任务还没有执行，则无论 `mayInterruptIfRunning` 为true还是false，肯定返回true。

**isCancelled()**

表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。

**isDone()**

表示任务是否已经完成，若任务完成，则返回true；

**get()**

用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；

**get(long timeout, TimeUnit unit)**

用来获取执行结果，如果在指定时间内，还没获取到结果，就直接抛出TimeoutException。



### FutureTask

Future提供了三种功能：

- 判断任务是否完成；
- 能够中断任务；
- 能够获取任务执行结果。

`Future` 只是一个接口，是无法直接用来创建对象使用的，因此就有了下面的 `FutureTask`。`ExecutorService` 返回的实际类型就是 `FutureTask`。我们先来看一下 `FutureTask` 的实现：

```java
public class FutureTask<V> implements RunnableFuture<V>
```

`FutureTask` 类实现了 `RunnableFuture` 接口，我们看一下 `RunnableFuture` 接口的实现：

```java
public interface RunnableFuture<V> extends Runnable, Future<V>
```

可以看出 `RunnableFuture` 继承了 `Runnable` 接口和 `Future` 接口，而 `FutureTask` 实现了 `RunnableFuture` 接口。所以它既可以作为 `Runnable` 被线程执行，又可以作为`Future` 得到 `Callable` 的返回值。

`FutureTask` 提供了2个构造器：

```java
public FutureTask(Callable<V> callable);
public FutureTask(Runnable runnable, V result);
```



### 使用示例

#### 使用Callable+Future获取执行结果

```java
public class Test {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newCachedThreadPool();
        Task task = new Task();
        Future<Integer> result = executor.submit(task);
        executor.shutdown();
         
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
         
        System.out.println("主线程在执行任务");
         
        try {
            System.out.println("task运行结果"+result.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
         
        System.out.println("所有任务执行完毕");
    }
}
class Task implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
        System.out.println("子线程在进行计算");
        Thread.sleep(3000);
        int sum = 0;
        for(int i=0;i<100;i++)
            sum += i;
        return sum;
    }
}
```

#### 使用Callable+FutureTask获取执行结果

```java
public class Test {
    public static void main(String[] args) {
        // 第一种方式
        ExecutorService executor = Executors.newCachedThreadPool();
        Task task = new Task();
        FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
        executor.submit(futureTask);
        executor.shutdown();
        
        // 第二种方式，注意这种方式和第一种方式效果是类似的，
        // 只不过一个使用的是ExecutorService，一个使用的是Thread
        /*
        	Task task = new Task();
        	FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
        	Thread thread = new Thread(futureTask);
        	thread.start();
        */
        
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
        
        System.out.println("主线程在执行任务");
        
        try {
            System.out.println("task运行结果"+futureTask.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        System.out.println("所有任务执行完毕");
    }
}
class Task implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
        System.out.println("子线程在进行计算");
        Thread.sleep(3000);
        int sum = 0;
        for(int i=0;i<100;i++)
            sum += i;
        return sum;
    }
}
```



## 线程池

### Executors

> Factory and utility methods for `Executor`, `ExecutorService`, `ScheduledExecutorService`, `ThreadFactory`, and `Callable` classes defined in this package. This class supports the following kinds of methods:
>
> - Methods that create and return an `ExecutorService` set up with commonly useful configuration settings.
> - Methods that create and return a `ScheduledExecutorService` set up with commonly useful configuration settings.
> - Methods that create and return a "wrapped" `ExecutorService`, that disables reconfiguration by making implementation-specific methods inaccessible.
> - Methods that create and return a `ThreadFactory` that sets newly created threads to a known state.
> - Methods that create and return a `Callable` out of other closure-like（类似闭包） forms, so they can be used in execution methods requiring `Callable`.

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/5e1e37f2-d40e-44d4-b960-7a02a3ea5866" /></div>



### Executor

```java
public interface Executor {

    /**
     * Executes the given command at some time in the future. The command
     * may execute in a new thread, in a pooled thread, or in the calling
     * thread, at the discretion of the {@code Executor} implementation.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be
     * accepted for execution
     * @throws NullPointerException if command is null
     */
    void execute(Runnable command);
}
```

> An object that executes submitted Runnable tasks. This interface provides a way of decoupling（解耦） task submission from the mechanics of how each task will be run, including details of thread use, scheduling, etc. An Executor is normally used instead of explicitly creating threads. For example, rather than invoking `new Thread(new(RunnableTask())).start()` for each of a set of tasks, you might use:
>
> ```java
> Executor executor = anExecutor;
> executor.execute(new RunnableTask1());
> executor.execute(new RunnableTask2());
> ...
> ```
>
> However, the Executor interface does not strictly require that execution be asynchronous. In the simplest case, an executor can run the submitted task immediately in the caller's thread:
>
> ```java
> class DirectExecutor implements Executor {
>        public void execute(Runnable r) {
>            r.run();
>        }
> }
> ```
>
> More typically, tasks are executed in some thread other than the caller's thread. The executor below spawns（产生） a new thread for each task.
>
> ```java
> class ThreadPerTaskExecutor implements Executor {
>        public void execute(Runnable r) {
>            new Thread(r).start();
>        }
> }
> ```
>
> Many Executor implementations impose（强加） some sort of limitation on how and when tasks are scheduled. The executor below serializes the submission of tasks to a second executor, illustrating（解释） a composite（复合的） executor.
>
> ```java
> class SerialExecutor implements Executor {
>        final Queue<Runnable> tasks = new ArrayDeque<Runnable>();
>        final Executor executor;
>        Runnable active;
> 
>        SerialExecutor(Executor executor) {
>            this.executor = executor;
>        }
> 
>        public synchronized void execute(final Runnable r) {
>            tasks.offer(new Runnable() {
>                public void run() {
>                    try {
>                        r.run();
>                    } finally {
>                        scheduleNext();
>                    }
>                }
>            });
>            if (active == null) {
>                scheduleNext();
>            }
>        }
> 
>        protected synchronized void scheduleNext() {
>            if ((active = tasks.poll()) != null) {
>                executor.execute(active);
>            }
>        }
> }
> ```
>
> The Executor implementations provided in this package implement `ExecutorService`, which is a more extensive interface. The `ThreadPoolExecutor` class provides an extensible thread pool implementation. The Executors class provides convenient factory methods for these Executors.
>
> Memory consistency effects: Actions in a thread prior to submitting a Runnable object to an Executor happen-before its execution begins, perhaps in another thread.



### ExecutorService

> An Executor that provides methods to manage termination and methods that can produce a `Future` for tracking progress of one or more asynchronous tasks.
>
> An `ExecutorService` can be shut down, which will cause it to reject new tasks. Two different methods are provided for shutting down an `ExecutorService`. The shutdown method will allow previously submitted tasks to execute before terminating, while the `shutdownNow` method prevents waiting tasks from starting and attempts to stop currently executing tasks. Upon termination, an executor has no tasks actively executing, no tasks awaiting execution, and no new tasks can be submitted. An unused `ExecutorService` should be shut down to allow reclamation（开垦，填海） of its resources.
>
> ```java
> void shutdown();
> 
> // return a list of the tasks that were awaiting execution.
> List<Runnable> shutdownNow();	
> ```
>
> Method submit extends base method `Executor.execute(Runnable)` by creating and returning a Future that can be used to cancel execution and/or wait for completion. Methods `invokeAny` and `invokeAll` perform the most commonly useful forms of bulk execution, executing a collection of tasks and then waiting for at least one, or all, to complete. (Class `ExecutorCompletionService` can be used to write customized variants of these methods.)
>
> The Executors class provides factory methods for the executor services provided in this package.
>
> **Usage Examples**
>
> Here is a sketch（草图） of a network service in which threads in a thread pool service incoming requests. It uses the preconfigured `Executors.newFixedThreadPool` factory method:
>
> ```java
> class NetworkService implements Runnable {
>        private final ServerSocket serverSocket;
>        private final ExecutorService pool;
> 
>        public NetworkService(int port, int poolSize)
>            throws IOException {
>            serverSocket = new ServerSocket(port);
>            pool = Executors.newFixedThreadPool(poolSize);
>        }
> 
>        public void run() { // run the service
>            try {
>                for (;;) {
>                    pool.execute(new Handler(serverSocket.accept()));
>                }
>            } catch (IOException ex) {
>                pool.shutdown();
>            }
>        }
> }
> 
> class Handler implements Runnable {
>        private final Socket socket;
>        Handler(Socket socket) { this.socket = socket; }
>        public void run() {
>            // read and service request on socket
>        }
> }
> ```
>
> The following method shuts down an `ExecutorService` in two phases, first by calling shutdown to reject incoming tasks, and then calling `shutdownNow`, if necessary, to cancel any lingering tasks:
>
> ```java
> void shutdownAndAwaitTermination(ExecutorService pool) {
>        pool.shutdown(); // Disable new tasks from being submitted
>        try {
>            // Wait a while for existing tasks to terminate
>            if (!pool.awaitTermination(60, TimeUnit.SECONDS)) {
>                pool.shutdownNow(); // Cancel currently executing tasks
>                // Wait a while for tasks to respond to being cancelled
>                if (!pool.awaitTermination(60, TimeUnit.SECONDS))
>                    System.err.println("Pool did not terminate");
>            }
>        } catch (InterruptedException ie) {
>            // (Re-)Cancel if current thread also interrupted
>            pool.shutdownNow();
>            // Preserve interrupt status
>            Thread.currentThread().interrupt();
>        }
> }
> ```
>
>
> Memory consistency effects: Actions in a thread prior to the submission of a Runnable or Callable task to an `ExecutorService` happen-before any actions taken by that task, which in turn happen-before the result is retrieved via `Future.get()`.



### AbstractExecutorService

> Provides default implementations of `ExecutorService` execution methods. This class implements the submit, `invokeAny` and `invokeAll` methods using a `RunnableFuture` returned by `newTaskFor`, which defaults to the `FutureTask` class provided in this package. For example, the implementation of `submit(Runnable)` creates an associated `RunnableFuture` that is executed and returned. Subclasses may override the `newTaskFor` methods to return `RunnableFuture` implementations other than `FutureTask`.
>
> Extension example. Here is a sketch of a class that customizes `ThreadPoolExecutor` to use a `CustomTask` class instead of the default `FutureTask`:
>
> ```java
> public class CustomThreadPoolExecutor extends ThreadPoolExecutor {
>        static class CustomTask<V> implements RunnableFuture<V> {...}
>        protected <V> RunnableFuture<V> newTaskFor(Callable<V> c) {
>            return new CustomTask<V>(c);
>        }
>        protected <V> RunnableFuture<V> newTaskFor(Runnable r, V v) {
>            return new CustomTask<V>(r, v);
>        }
>        // ... add constructors, etc.
> }
> ```



### ThreadExecutorService

> An `ExecutorService` that executes each submitted task using one of possibly several pooled threads, normally configured using Executors factory methods.
>
> Thread pools address two different problems: they usually provide improved performance when executing large numbers of asynchronous tasks, due to reduced per-task invocation overhead（开销）, and they provide a means of（一种手段） bounding（边界） and managing the resources, including threads, consumed when executing a collection of tasks. Each `ThreadPoolExecutor` also maintains some basic statistics, such as the number of completed tasks.
>
> To be useful across a wide range of contexts, this class provides many adjustable parameters and extensibility hooks. However, programmers are urged to use the more convenient Executors factory methods `Executors.newCachedThreadPool` (unbounded thread pool, with automatic thread reclamation), `Executors.newFixedThreadPool` (fixed size thread pool) and `Executors.newSingleThreadExecutor` (single background thread), that preconfigure settings for the most common usage scenarios. Otherwise, use the following guide when manually configuring and tuning this class:
>
> **Core and maximum pool sizes**
>
> A `ThreadPoolExecutor` will automatically adjust the pool size (see `getPoolSize`) according to the bounds set by corePoolSize (see `getCorePoolSize`) and maximumPoolSize (see `getMaximumPoolSize`). 
>
> When a new task is submitted in method `execute(Runnable)`, and fewer than corePoolSize threads are running, a new thread is created to handle the request, even if other worker threads are idle. If there are more than corePoolSize but less than maximumPoolSize threads running, a new thread will be created only if the queue is full. By setting corePoolSize and maximumPoolSize the same, you create a fixed-size thread pool. By setting maximumPoolSize to an essentially unbounded value such as `Integer.MAX_VALUE`, you allow the pool to accommodate an arbitrary（任意的、专制的、随心所欲的） number of concurrent tasks. Most typically, core and maximum pool sizes are set only upon construction, but they may also be changed dynamically using `setCorePoolSize` and `setMaximumPoolSize`.
>
> **On-demand construction**
>
> By default, even core threads are initially created and started only when new tasks arrive, but this can be overridden dynamically using method `prestartCoreThread` or `prestartAllCoreThreads`. You probably want to prestart threads if you construct the pool with a non-empty queue.
>
> **Creating new threads**
>
> New threads are created using a `ThreadFactory`. If not otherwise specified, a `Executors.defaultThreadFactory` is used, that creates threads to all be in the same `ThreadGroup` and with the same NORM_PRIORITY priority and non-daemon status. By supplying a different `ThreadFactory`, you can alter the thread's name, thread group, priority, daemon status, etc. If a `ThreadFactory` fails to create a thread when asked by returning null from newThread, the executor will continue, but might not be able to execute any tasks. Threads should possess（拥有） the "modifyThread" RuntimePermission. If worker threads or other threads using the pool do not possess this permission, service may be degraded（降级）: configuration changes may not take effect in a timely（及时） manner（方式）, and a shutdown pool may remain in a state in which termination is possible but not completed.
>
> **Keep-alive times**
>
> If the pool currently has more than corePoolSize threads, excess threads will be terminated if they have been idle for more than the keepAliveTime (see `getKeepAliveTime(TimeUnit)`). This provides a means of reducing resource consumption（消费） when the pool is not being actively used. If the pool becomes more active later, new threads will be constructed. This parameter can also be changed dynamically using method `setKeepAliveTime(long, TimeUnit)`. Using a value of `Long.MAX_VALUE` `TimeUnit.NANOSECONDS` effectively disables idle threads from ever terminating prior to shut down. By default, the keep-alive policy applies only when there are more than corePoolSize threads. But method `allowCoreThreadTimeOut(boolean)` can be used to apply this time-out policy to core threads as well, so long as（只要） the keepAliveTime value is non-zero.
>
> **Queuing**
>
> Any `BlockingQueue` may be used to transfer and hold submitted tasks. The use of this queue interacts with pool sizing:
>
> - If fewer than corePoolSize threads are running, the Executor always prefers adding a new thread rather than queuing.
> - If corePoolSize or more threads are running, the Executor always prefers queuing a request rather than adding a new thread.
> - If a request cannot be queued, a new thread is created unless this would exceed（超过） maximumPoolSize, in which case, the task will be rejected.
>
> There are three general strategies for queuing:
>
> 1. Direct handoffs. A good default choice for a work queue is a `SynchronousQueue` that hands off tasks to threads without otherwise holding them. Here, an attempt to queue a task will fail if no threads are immediately available to run it, so a new thread will be constructed. This policy avoids lockups（锁定） when handling sets of requests that might have internal dependencies. Direct handoffs generally require unbounded maximumPoolSizes to avoid rejection of new submitted tasks. This in turn admits the possibility of unbounded thread growth when commands continue to arrive on average faster than they can be processed.
> 2. Unbounded queues. Using an unbounded queue (for example a `LinkedBlockingQueue` without a predefined capacity) will cause new tasks to wait in the queue when all corePoolSize threads are busy. Thus, no more than corePoolSize threads will ever be created. (And the value of the maximumPoolSize therefore doesn't have any effect.) This may be appropriate when each task is completely independent of others, so tasks cannot affect each others execution; for example, in a web page server. While this style of queuing can be useful in smoothing out transient（短暂的） bursts（爆发） of requests, it admits the possibility of unbounded work queue growth when commands continue to arrive on average faster than they can be processed.
> 3. Bounded queues. A bounded queue (for example, an ArrayBlockingQueue) helps prevent resource exhaustion when used with finite maximumPoolSizes, but can be more difficult to tune（调） and control. Queue sizes and maximum pool sizes may be traded off（权衡） for each other: Using large queues and small pools minimizes CPU usage, OS resources, and context-switching overhead, but can lead to artificially（人为的） low throughput. If tasks frequently block (for example if they are I/O bound), a system may be able to schedule time for more threads than you otherwise（相反的 adj.） allow. Use of small queues generally requires larger pool sizes, which keeps CPUs busier but may encounter unacceptable scheduling overhead, which also decreases throughput.
>
> **Rejected tasks**
>
> New tasks submitted in method `execute(Runnable)` will be rejected when the Executor has been shut down, and also when the Executor uses finite bounds for both maximum threads and work queue capacity, and is saturated（饱和的）. In either case, the execute method invokes the `RejectedExecutionHandler.rejectedExecution(Runnable, ThreadPoolExecutor)` method of its `RejectedExecutionHandler`. Four predefined handler policies are provided:
>
> - In the default ThreadPoolExecutor.AbortPolicy, the handler throws a runtime RejectedExecutionException upon rejection.
> - In ThreadPoolExecutor.CallerRunsPolicy, the thread that invokes execute itself runs the task. This provides a simple feedback control mechanism that will slow down the rate that new tasks are submitted.
> - In ThreadPoolExecutor.DiscardPolicy, a task that cannot be executed is simply dropped.
> - In ThreadPoolExecutor.DiscardOldestPolicy, if the executor is not shut down, the task at the head of the work queue is dropped, and then execution is retried (which can fail again, causing this to be repeated.)
>
> It is possible to define and use other kinds of RejectedExecutionHandler classes. Doing so requires some care especially when policies are designed to work only under particular capacity or queuing policies.
>
> **Hook methods**
>
> This class provides protected overridable `beforeExecute(Thread, Runnable)` and `afterExecute(Runnable, Throwable)` methods that are called before and after execution of each task. 
>
> These can be used to manipulate（操纵） the execution environment; for example, reinitializing ThreadLocals, gathering statistics, or adding log entries. Additionally, method terminated can be overridden to perform any special processing that needs to be done once the Executor has fully terminated.
>
> If hook or callback methods throw exceptions, internal worker threads may in turn fail and abruptly（意外的） terminate.
>
> **Queue maintenance**
>
> Method `getQueue()` allows access to the work queue for purposes of monitoring and debugging. Use of this method for any other purpose is strongly discouraged. Two supplied methods, `remove(Runnable)` and `purge` are available to assist in storage reclamation（开垦） when large numbers of queued tasks become cancelled.
>
> **Finalization**
>
> A pool that is no longer referenced in a program AND has no remaining threads will be shutdown automatically. If you would like to ensure that unreferenced pools are reclaimed（回收） even if users forget to call shutdown, then you must arrange that unused threads eventually die, by setting appropriate keep-alive times, using a lower bound of zero core threads and/or setting `allowCoreThreadTimeOut(boolean)`.
>
> Extension example. Most extensions of this class override one or more of the protected hook methods. For example, here is a subclass that adds a simple pause/resume feature:
>
> ```java
> class PausableThreadPoolExecutor extends ThreadPoolExecutor {
>     private boolean isPaused;
>     private ReentrantLock pauseLock = new ReentrantLock();
>     private Condition unpaused = pauseLock.newCondition();
> 
>     public PausableThreadPoolExecutor(...) { super(...); }
> 
>     protected void beforeExecute(Thread t, Runnable r) {
>         super.beforeExecute(t, r);
>         pauseLock.lock();
>         try {
>             while (isPaused) unpaused.await();
>         } catch (InterruptedException ie) {
>             t.interrupt();
>         } finally {
>             pauseLock.unlock();
>         }
>     }
> 
>     public void pause() {
>         pauseLock.lock();
>         try {
>             isPaused = true;
>         } finally {
>             pauseLock.unlock();
>         }
>     }
> 
>     public void resume() {
>         pauseLock.lock();
>         try {
>             isPaused = false;
>             unpaused.signalAll();
>         } finally {
>             pauseLock.unlock();
>         }
>     }
> }
> ```

#### ThreadGroup

> A thread group represents a set of threads. In addition, a thread group can also include other thread groups. The thread groups form a tree in which every thread group except the initial thread group has a parent.
>
> A thread is allowed to access information about its own thread group, but not to access information about its thread group's parent thread group or any other thread groups.

线程组的出现是为了更方便地管理线程。

线程组是父子结构的，一个线程组可以集成其他线程组，同时也可以拥有其他子线程组。从结构上看，线程组是一个树形结构，每个线程都隶属于一个线程组，线程组又有父线程组，这样追溯下去，可以追溯到一个根线程组——System线程组。

```java
public static void main(String[] args) {
    ThreadGroup mainThreadGroup = Thread.currentThread().getThreadGroup();
    ThreadGroup systenThreadGroup = mainThreadGroup.getParent();
    ThreadGroup parent = systenThreadGroup.getParent();

    System.out.println("systenThreadGroup name = " + systenThreadGroup.getName());
    System.out.println("mainThreadGroup name = " + mainThreadGroup.getName());
    System.out.println(parent.getName());
}
// systenThreadGroup name = system
// mainThreadGroup name = main
// Exception in thread "main" java.lang.NullPointerException
// 		at Test.main(Test.java:9)
```

`java.lang.ThreadGroup` 提供了两个构造函数：

| Constructor                                  | Description                                              |
| -------------------------------------------- | -------------------------------------------------------- |
| ThreadGroup(String name)                     | 根据线程组名称创建线程组，其父线程组为main线程组         |
| ThreadGroup(ThreadGroup parent, String name) | 根据线程组名称创建线程组，其父线程组为指定的parent线程组 |

**自定义异常处理**

```java
public class ThreadGroupDemo {

    public static void main(String[] args) {
        ThreadGroup threadGroup1 = new ThreadGroup("group1") {
            // 继承ThreadGroup并重新定义以下方法
            // 在线程成员抛出unchecked exception
            // 会执行此方法
            public void uncaughtException(Thread t, Throwable e) {
                System.out.println(t.getName() + ": " + e.getMessage());
            }
        };

        // 这个线程是threadGroup1的一员
        Thread thread1 = new Thread(threadGroup1, new Runnable() {
            public void run() {
                // 抛出unchecked异常
                System.out.println(1 / 0);
            }
        });

        thread1.start();
    }
}
```

#### 自定义线程池使用

ThreadPoolExecutor 使用其内部池中的线程执行给定任务(Callable 或者 Runnable)。ThreadPoolExecutor 包含的线程池能够包含不同数量的线程。池中线程的数量由以下变量决定：

- corePoolSize
- maximumPoolSize

当一个任务委托给线程池时，如果池中线程数量低于 corePoolSize，一个新的线程将被创建，即使池中可能尚有空闲线程。如果内部任务队列已满，而且有 corePoolSize 个线程正在运行，但是运行线程的数量低于maximumPoolSize，一个新的线程将被创建去执行该任务。

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler);
```

- corePoolSize：线程池核心线程数（平时保留的线程数）
- maximumPoolSize：线程池最大线程数（当workQueue都放不下时，启动新线程，最大线程数）
- keepAliveTime：超出corePoolSize数量的线程的保留时间。
- unit：keepAliveTime单位
- workQueue：阻塞队列，存放来不及执行的线程
  - ArrayBlockingQueue：构造函数一定要传大小
  - LinkedBlockingQueue：构造函数不传大小会默认为（Integer.MAX_VALUE ），**当大量请求任务时，容易造成 内存耗尽。**
  - SynchronousQueue：同步队列，一个没有存储空间的阻塞队列 ，将任务同步交付给工作线程。
  - PriorityBlockingQueue：优先队列
- threadFactory：线程工厂
- handler：饱和策略
  - AbortPolicy（默认）：直接抛弃
  - CallerRunsPolicy：用调用者的线程执行任务
  - DiscardOldestPolicy：抛弃队列中最久的任务
  - DiscardPolicy：抛弃当前任务

**阿里巴巴开发手册**

线程池不使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。

说明： Executors 返回的线程池对象的弊端如下：

1. FixedThreadPool 和 SingleThreadPool : 允许的请求队列长度为 Integer.MAX_VALUE ，可能会堆积大量的请求，从而导致 OOM 。
2. CachedThreadPool 和 ScheduledThreadPool : 允许的创建线程数量为 Integer.MAX_VALUE ，可能会创建大量的线程，从而导致 OOM。

#### 自定义ThreadFactory

> An object that creates new threads on demand. Using thread factories removes hardwiring of calls to new Thread, enabling applications to use special thread subclasses, priorities, etc.
>
> The simplest implementation of this interface is just:
>
> ```java
> class SimpleThreadFactory implements ThreadFactory {
>        public Thread newThread(Runnable r) {
>            return new Thread(r);
>        }
> }
> ```
>
> The `Executors.defaultThreadFactory` method provides a more useful simple implementation, that sets the created thread context to known values before returning it.

**DefaultThreadFactory**

```java
static class DefaultThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;

    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() :
        		Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" + poolNumber.getAndIncrement() + "-thread-";
    }

    public Thread newThread(Runnable r) {
        Thread t = new Thread(group, r,
                              namePrefix + threadNumber.getAndIncrement(),
                              0);
        if (t.isDaemon())
            t.setDaemon(false);
        if (t.getPriority() != Thread.NORM_PRIORITY)
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}
```

`SecurityManager` 获取的 `ThreadGroup` 如下：

```java
private static ThreadGroup getRootGroup() {
    ThreadGroup root =  Thread.currentThread().getThreadGroup();
    while (root.getParent() != null) {
        root = root.getParent();
    }
    return root;
}
```

`DefaultThreadFactory` 的 `ThreadGroup`：

```java
public static void main(String[] args) {
    ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
        2,
        2,
        60,
        TimeUnit.SECONDS,
        new LinkedBlockingQueue<>(),
        Executors.defaultThreadFactory(),
        new ThreadPoolExecutor.AbortPolicy()
    );

    threadPoolExecutor.execute(new Thread() {
        @Override
        public void run() {
            System.out.println(this.getThreadGroup().getName()); 	// main
        }
    });
}
```

#### 自定义RejectedExecutionHandler

```java
/**
 * A handler for tasks that cannot be executed by a {@link ThreadPoolExecutor}.
 */
public interface RejectedExecutionHandler {

    /**
     * Method that may be invoked by a {@link ThreadPoolExecutor} when
     * {@link ThreadPoolExecutor#execute execute} cannot accept a
     * task.  This may occur when no more threads or queue slots are
     * available because their bounds would be exceeded, or upon
     * shutdown of the Executor.
     *
     * <p>In the absence of other alternatives, the method may throw
     * an unchecked {@link RejectedExecutionException}, which will be
     * propagated to the caller of {@code execute}.
     *
     * @param r the runnable task requested to be executed
     * @param executor the executor attempting to execute this task
     * @throws RejectedExecutionException if there is no remedy
     */
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}
```

测试

```java
public class T14_MyRejectedHandler {

    public static void main(String[] args) {
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(1, 1,
                0, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>(6),
                new MyThreadFactory(),
                new MyHandler());

        for (int i = 0; i < 100; i++) {
            threadPoolExecutor.submit(() -> {
                try {
                    TimeUnit.SECONDS.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
    }

    static class MyHandler implements RejectedExecutionHandler {
        private AtomicInteger count = new AtomicInteger(1);
        @Override
        public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
            System.out.println(r.getClass() + "    " + count.get());
            count.incrementAndGet();
        }
    }

    static class MyThreadFactory implements ThreadFactory {
        public Thread newThread(Runnable r) {
            System.out.println("new Thread ... ");
            return new Thread(r);
        }
    }
}
```

此时会发现，现在拒绝策略走的是我们的 `MyHandler`，输出的是类是：`java.util.concurrent.FutureTask`。



### 执行流程

**submit()**

```java
public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    execute(ftask);
    return ftask;
}
```

```java
protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
    return new FutureTask<T>(runnable, value);
}
```

在 `submit()` 方法里，我们提交的任务会被包装成一个 `RunnableFuture`，而 `newTaskFor()` 返回的真实类型就是我们上面测试输出的 `FutureTask`。

**ctl**

> The main pool control state, ctl, is an atomic integer packing two conceptual fields workerCount, indicating the effective number of threads runState, indicating whether running, shutting down etc.
>
> In order to pack them into one int, we limit workerCount to (2^29)-1 (about 500 million) threads rather than (2^31)-1 (2 billion) otherwise representable. If this is ever an issue in the future, the variable can be changed to be an AtomicLong, and the shift/mask constants below adjusted. But until the need arises, this code is a bit faster and simpler using an int. 
>
> The workerCount is the number of workers that have been permitted to start and not permitted to stop. The value may be transiently different from the actual number of live threads, for example when a `ThreadFactory` fails to create a thread when asked, and when exiting threads are still performing bookkeeping（簿记） before terminating. The user-visible pool size is reported as the current size of the workers set. 
>
> The runState provides the main lifecycle control, taking on values: 
>
> - RUNNING: Accept new tasks and process queued tasks （初始状态）
> - SHUTDOWN: Don't accept new tasks, but process queued tasks 
> - STOP: Don't accept new tasks, don't process queued tasks, and interrupt in-progress tasks 
> - TIDYING（整理）: All tasks have terminated, workerCount is zero, the thread transitioning to state TIDYING will run the terminated() hook method 
> - TERMINATED: terminated() has completed 
>
> The numerical order among these values matters（重要）, to allow ordered comparisons. The runState monotonically increases over time, but need not hit（击中、到达） each state. The transitions are: 
>
> - RUNNING -> SHUTDOWN: On invocation of shutdown(), perhaps implicitly（隐含的） in finalize() 
> - (RUNNING or SHUTDOWN) -> STOP: On invocation of shutdownNow() 
> - SHUTDOWN -> TIDYING: When both queue and pool are empty 
> - STOP -> TIDYING: When pool is empty 
> - TIDYING -> TERMINATED: When the terminated() hook method has completed 
>
> Threads waiting in `awaitTermination()` will return when the state reaches TERMINATED. Detecting the transition from SHUTDOWN to TIDYING is less straightforward than you'd like because the queue may become empty after non-empty and vice versa during SHUTDOWN state, but we can only terminate if, after seeing that it is empty, we see that workerCount is 0 (which sometimes entails a recheck -- see below).

<div align="center"><img width="90%" src="http://blogfileqiniu.isjinhao.site/8aa49fea-3648-4996-afab-f406d366e09b" /></div>
**execute()**

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     */
    int c = ctl.get();
    // worker线程小于核心线程数，增加一个线程
    if (workerCountOf(c) < corePoolSize) {
        // 增加线程成功了，就可以返回
        if (addWorker(command, true))
            return;
        // 增加线程不成功，此时ctl会被更改
        c = ctl.get();
    }
    // 如果工作线程数大于等于核心线程数，线程池的状态是RUNNING，并且添加进队列成功
    if (isRunning(c) && workQueue.offer(command)) {
        // 由于在进入队列前后，线程可能会死掉或者线程池会shutdown，所以需要recheck
        // 比如在ctl获取之后，command进入队列前，队列的内容执行完毕，线程死掉。
        int recheck = ctl.get();
        // 如果线程池没有在运行（执行过shutdown），并且从任务队列中移除成功，reject此任务
        if (!isRunning(recheck) && remove(command))
            reject(command);
        // 如果线程池没在运行，但是移除失败，说明此任务已经被完成
        // 所以此处判断是够为0是由于，在线程池构造方法中，核心线程数允许为0，此时需要增加一个线程
        // 也就是corePoolSize为0且我们提交的是第一个任务的情况
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 如果线程池不是运行状态，或者任务进入队列失败，则尝试创建worker执行任务。
    // 这儿有3点需要注意：
    // 1. 线程池不是运行状态时，addWorker内部会判断线程池状态
    // 2. addWorker第2个参数表示是否创建核心线程
    // 3. addWorker返回false，则说明任务执行失败，需要执行reject操作
    else if (!addWorker(command, false))
        reject(command);
}
```

**addWorker()**

```java
/**
 * Checks if a new worker can be added with respect to current
 * pool state and the given bound (either core or maximum). If so,
 * the worker count is adjusted accordingly, and, if possible, a
 * new worker is created and started, running firstTask as its
 * first task. This method returns false if the pool is stopped or
 * eligible（合格、可以选） to shut down. It also returns false if the thread
 * factory fails to create a thread when asked.  If the thread
 * creation fails, either due to the thread factory returning
 * null, or due to an exception (typically OutOfMemoryError in
 * Thread.start()), we roll back cleanly.
 *
 * @param firstTask the task the new thread should run first (or
 * null if none). Workers are created with an initial first task
 * (in method execute()) to bypass queuing when there are fewer
 * than corePoolSize threads (in which case we always start one),
 * or when the queue is full (in which case we must bypass queue).
 * Initially idle threads are usually created via
 * prestartCoreThread or to replace other dying workers.
 *
 * @param core if true use corePoolSize as bound, else
 * maximumPoolSize. (A boolean indicator is used here rather than a
 * value to ensure reads of fresh values after checking other pool
 * state).
 * @return true if successful
 */
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);
        // 1. 线程池状态大于SHUTDOWN时，直接返回false
        // 2. 线程池状态等于SHUTDOWN，且firstTask不为null，直接返回false
        // 3. 线程池状态等于SHUTDOWN，且队列为空，直接返回false
        if (rs >= SHUTDOWN && 
            	(rs != SHUTDOWN || firstTask != null || workQueue.isEmpty()))
            return false;
        
        // 能到达这一步的条件是线程池处于RUNNING状态或
        // 状态为SHUTDOWN，且 firstTask 为空 且 workQueue 不空
        for (;;) {
            int wc = workerCountOf(c);
            // Worker数量超过容量，直接返回false
            if (wc >= CAPACITY || wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            // 使用CAS的方式增加worker数量。
            // 若增加成功，则直接跳出外层循环进入到第二部分
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            // 线程池状态发生变化，对外层循环进行自旋
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            // 对 workers 的操作是串行化的
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                // 这儿需要重新检查线程池状态
                int rs = runStateOf(ctl.get());

                // 虽然在第35行判断ctl的状态了，但是在那之后，ctl可能会发生改变
                // 比如被SHUTDOWN，那么此时就不能添加worker
                if (rs < SHUTDOWN || (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            // 启动worker线程
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        // Worker线程启动失败，说明线程池状态发生了变化（关闭操作被执行），需要进行terminated()
        if (!workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

**addWorkerFailed()**

```java
private void addWorkerFailed(Worker w) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        if (w != null)
            workers.remove(w);
        decrementWorkerCount();
        tryTerminate();
    } finally {
        mainLock.unlock();
    }
}
```

**Worker**

先不分析JDK是如何实现线程池里的线程的，假设一下，如果这个需求让我们实现，会做哪些考虑。首先，线程是死循环的，不断的从任务队列中获取任务进行处理。当线程没有任务时需要停止，但是经过TimeOut时间之后需要给线程 `interrupt()`。而在JDK的实现中完成这个功能的是类 `Worker`。

```java
/**
 * Class Worker mainly maintains interrupt control state for
 * threads running tasks, along with other minor bookkeeping.
 * This class opportunistically extends AbstractQueuedSynchronizer
 * to simplify acquiring and releasing a lock surrounding each
 * task execution.  This protects against interrupts that are
 * intended to wake up a worker thread waiting for a task from
 * instead interrupting a task being run.  We implement a simple
 * non-reentrant mutual exclusion lock rather than use
 * ReentrantLock because we do not want worker tasks to be able to
 * reacquire the lock when they invoke pool control methods like
 * setCorePoolSize.  Additionally, to suppress interrupts until
 * the thread actually starts running tasks, we initialize lock
 * state to a negative value, and clear it upon start (in
 * runWorker).
 */
private final class Worker
    extends AbstractQueuedSynchronizer
    implements Runnable	{

    /** Thread this worker is running in.  Null if factory fails. */
    final Thread thread;
    /** Initial task to run.  Possibly null. */
    Runnable firstTask;
    /** Per-thread task counter */
    volatile long completedTasks;

    /**
      * Creates with given first task and thread from ThreadFactory.
      * @param firstTask the first task (null if none)
      */
    Worker(Runnable firstTask) {
        setState(-1); // inhibit interrupts until runWorker
        this.firstTask = firstTask;
        
        // 实际上在这，是将自己给封装起来了，以thread运行后，运行的是Workder的run方法
        this.thread = getThreadFactory().newThread(this);
    }

    /** Delegates main run loop to outer runWorker  */
    public void run() {
        runWorker(this);
    }

    // Lock methods
    //
    // The value 0 represents the unlocked state.
    // The value 1 represents the locked state.

    protected boolean isHeldExclusively() {
        return getState() != 0;
    }

    protected boolean tryAcquire(int unused) {
        if (compareAndSetState(0, 1)) {
            setExclusiveOwnerThread(Thread.currentThread());
            return true;
        }
        return false;
    }

    protected boolean tryRelease(int unused) {
        setExclusiveOwnerThread(null);
        setState(0);
        return true;
    }

    public void lock()        { acquire(1); }
    public boolean tryLock()  { return tryAcquire(1); }
    public void unlock()      { release(1); }
    public boolean isLocked() { return isHeldExclusively(); }

    void interruptIfStarted() {
        Thread t;
        if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
            try {
                t.interrupt();
            } catch (SecurityException ignore) {
            }
        }
    }
}
```

**runWorker()**

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        // 循环重队列中获取任务
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            
            // 如果线程池处于停止状态 且 线程没有被中断，中断当前线程
            // 如果线程被中断了（getTask期间），看看线程池是不是停止，如果没有停止，清除中断
            if ((runStateAtLeast(ctl.get(), STOP) || 
                 		(Thread.interrupted() && runStateAtLeast(ctl.get(), STOP)))
                	&& !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    // 这一步是串行的
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```

**getTask()**

```java
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?

    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        // 1.状态是 STOP
        // 2.状态是 SHUTDOWN，但 workQueue 是空
        // 
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            // 返回空，在runWorker那会阻塞，所以需要在这里将活动Worker降低一个
            decrementWorkerCount();
            return null;
        }

        int wc = workerCountOf(c);

        // Are workers subject to culling?
        // 当前的Worker是否允许时间
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        // 条件拆解
        // 1.wc > maximumPoolSize && wc > 1
        // 2.wc > maximumPoolSize && workQueue.isEmpty()
        // 3.(timed && timedOut) && wc > 1
        // 4.(timed && timedOut) && workQueue.isEmpty()
        // 解释
        // 1.如果比最大线程数大 且 还有其他正在运行的线程
        // 2.如果比最大线程数大 且 工作队列为空
        // 3.如果当前线程获取任务超时 且 还有其他正在运行的线程
        // 4.如果当前线程获取任务超时 且 工作队列为空
        // 结果
        // 使当前线程不再工作
        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }
		// 能够运行到这一步的条件是
        // 1.wc <= maxmaximumPoolSize && !(timed && timeOut)
        // 2.wc <= 1 && !workQueue.isEmpty()
        // 解释
        // 1.正在运行的线程个数没有超过最大线程数 且 获取任务没有超时
        // 2.没有其他的线程在运行 且 任务队列不空
        try {
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
            workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            // 如果被中断了，重新计时
            timedOut = false;
        }
    }
}
```

**小结**

一个Worker是一个真正运行的线程。循环的从任务队列里面取出Runable来执行。实现了AQS是为了在执行真正任务的时候屏蔽中断，只有在获取任务（getTask()）的时候才能响应中断。



### ScheduledExecutorService

java.util.concurrent.ScheduledExecutorService 是一个 ExecutorService， 它能够将任务延后执行，或者间隔固定时间多次执行。 任务由一个工作者线程异步执行，而不是由提交任务给 ScheduledExecutorService 的那个线程执行。

```java
public class TestExecutor {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ScheduledExecutorService scheduledExecutorService =
            						Executors.newScheduledThreadPool(5);
        ScheduledFuture scheduledFuture =
            scheduledExecutorService.schedule(
            	new Callable() {
                	public Object call() throws Exception {
                        System.out.println("Executed!");
                        return "Called!";
                    }
            	},
            	5,
            	TimeUnit.SECONDS);	// 5 秒后执行
        scheduledExecutorService.shutdown();
    }
}
```

**ScheduledExecutorService 的实现**

ScheduledExecutorService 是一个接口，你要用它的话就得使用 java.util.concurrent 包里对它的某个实现类。
ScheduledExecutorService 具有以下实现类：ScheduledThreadPoolExecutor

**创建一个 ScheduledExecutorService**

如何创建一个 ScheduledExecutorService 取决于你采用的它的实现类。但是你也可以使用 Executors 工厂类
来创建一个 ScheduledExecutorService 实例。比如：

```java
ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(5);
```

**ScheduledExecutorService 的使用**

一旦你创建了一个 ScheduledExecutorService，你可以通过调用它的以下方法：

- schedule (Callable task, long delay, TimeUnit timeunit)

```java
public class TestExecutor {
    public static void main(String[] args) throws Exception {

        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(5);
        ScheduledFuture scheduledFuture = scheduledExecutorService.schedule(
            new Callable() {
                public Object call() throws Exception {
                    System.out.println("Executed!");
                    return "Called!";
                }
            },
            5,
            TimeUnit.SECONDS);
        System.out.println("result = " + scheduledFuture.get());
        scheduledExecutorService.shutdown();

    }
}
```

- schedule (Runnable task, long delay, TimeUnit timeunit)
- scheduleAtFixedRate (Runnable, task long initialDelay, long period, TimeUnit timeunit)

这一方法规划一个任务将被定期执行。该任务将会在首个 initialDelay 之后得到执行，然后每个 period 时间之
后重复执行。如果给定任务的执行抛出了异常，该任务将不再执行。如果没有任何异常的话，这个任务将会持续循环执行到ScheduledExecutorService 被关闭。如果一个任务占用了比计划的时间间隔更长的时候，下一次执行将在当前执行结束执行才开始。计划任务在同一时间不会有多个线程同时执行。

- scheduleWithFixedDelay (Runnable, long initialDelay, long period, TimeUnit timeunit)

除了 period 有不同的解释之外这个方法和 scheduleAtFixedRate() 非常像。scheduleAtFixedRate() 方法中，period 被解释为前一个执行的开始和下一个执行的开始之间的间隔时间。而在本方法中，period 则被解释为前一个执行的结束和下一个执行的结束之间的间隔。因此这个延迟是执行结束之间的间隔，而不是执行开始之间的间隔。