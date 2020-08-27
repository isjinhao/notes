## 线程的中断

### interrupt()

> Interrupts this thread.
>
> Unless the current thread is interrupting itself, which is always permitted, the `checkAccess` method of this thread is invoked, which may cause a `SecurityException` to be thrown.
>
> If this thread is blocked in an invocation of the `wait()`, `wait(long)`, or `wait(long, int)` methods of the Object class, or of the `join()`, `join(long)`, `join(long, int)`, `sleep(long)`, or `sleep(long, int)`, methods of this class, then its interrupt status will be cleared and it will receive an `InterruptedException`.
>
> If this thread is blocked in an I/O operation upon an `InterruptibleChannel`, such as `SocketChannel` ， then the channel will be closed, the thread's interrupt status will be set, and the thread will receive a `java.nio.channels.ClosedByInterruptException`.
>
> If this thread is blocked in a `java.nio.channels.Selector` then the thread's interrupt status will be set and it will return immediately from the selection operation, possibly with a non-zero value, just as if the selector's wakeup method were invoked.
>
> If none of the previous conditions hold then this thread's interrupt status will be set.
>
> Interrupting a thread that is not alive need not have any effect.

`interrupt()` 是一个线程中断的方法，本人只在Java网络编程这门课的实验里用过一次。其可以使得处于阻塞状态的线程抛出一个异常，也就说，它可以用来中断一个正处于阻塞状态的线程。

```java
public class Test {
    public static void main(String[] args) throws IOException  {
        Test test = new Test();
        MyThread thread = test.new MyThread();
        thread.start();
        // 睡2秒，保证thread线程得到执行
        try {
            Thread.currentThread().sleep(2000);
        } catch (InterruptedException e) {
            
        }
        thread.interrupt();
    }
     
    class MyThread extends Thread{
        @Override
        public void run() {
            try {
                System.out.println("进入睡眠状态");
                Thread.currentThread().sleep(10000);
                System.out.println("睡眠完毕");
            } catch (InterruptedException e) {
                System.out.println("得到中断异常");
            }
            System.out.println("run方法执行完毕");
        }
    }
}
```



### isInterrupted()

`isInterrupted()` 判断中断标志是否被置位来中断线程的执行。`interrupt()` 配合 `isInterrupted()` 能够中断正在运行的线程，因为调用 `interrupt()` 方法相当于将中断标志位置为true。

```java
public class Test {
    public static void main(String[] args) throws Exception {
        Test test = new Test();
        MyThread thread = test.new MyThread();
        thread.start();
        Thread.currentThread().sleep(20);
        thread.interrupt();
    }

    class MyThread extends Thread {
        @Override
        public void run() {
            int i = 0;
            boolean b = isInterrupted();
            System.out.println(b + "   " + i);
            while (!b && i < Integer.MAX_VALUE) {
                i++;
                b = isInterrupted();
                System.out.println(b + "   " + i);
            }
            System.out.println(isInterrupted());
            System.out.println("abcd");
        }
    }
}
```

但是以上的暂停程序运行的方法可以被替换为如下代码：

```java
class MyThread extends Thread{
    private volatile boolean isStop = false;
    @Override
    public void run() {
        int i = 0;
        while(!isStop){
            i++;
        }
    }
    
    public void setStop(boolean stop){
        this.isStop = stop;
    }
}
```



### interrupted()

> Tests whether the current thread has been interrupted.  The  *interrupted status* of the thread is cleared by this method.  In other words, if this method were to be called twice in succession, the second call would return false (unless the current thread were interrupted again, after the first call had cleared its interrupted status and before the second call had examined it).  
>
> A thread interruption ignored because a thread was not alive at the time of the interrupt will be reflected by this method returning false.

```java
public class Test {

    public static void main(String[] args) throws Exception {
        Test test = new Test();
        MyThread thread = test.new MyThread();
        thread.start();
        Thread.currentThread().sleep(20);
        thread.interrupt();
    }

    class MyThread extends Thread {
        @Override
        public void run() {
            int i = 0;
            boolean b = interrupted();
            System.out.println(b + "   " + i);
            while (!b && i < Integer.MAX_VALUE) {
                i++;
                b = interrupted();
                System.out.println(b + "   " + i);
            }
            System.out.println(isInterrupted());
            System.out.println("abcd");
        }
    }
}
```

`interrupted()`和`isInterrupted()`的程序差别就是前者在输出abcd之前输出的是false，后者输出的是true。

此三者都是线程中断位相关的方法。每个线程都有一个中断状态位，中断位初始的时候是 false：

- `private native boolean isInterrupted(boolean ClearInterrupted);`：传入true重置状态位，传入false不重置状态位。返回此方法执行完成前线程中断状态位的状态。
- `public void interrupt()`：将中断状态位设置为true。
- `public boolean isInterrupted()`：查看当前状态位但是不影响状态位，内部实现原理`isInterrupted(false)`。
- `public static boolean interrupted()`：重置当前线程状态位（即如果状态位是true，则设置为false），内部实现原理`isInterrupted(true)`。



## LockSupport

这个类是 JDK 提供的方便阻塞和唤醒线程的工具类。

**阻塞**

```java
public static void park(Object blocker);
public static void park();

public static void parkUntil(long deadline);
public static void parkUntil(Object blocker, long deadline);

public static void parkNanos(long nanos);
public static void parkNanos(Object blocker, long nanos);

public static Object getBlocker(Thread t);
```

它阻塞的都是当前线程。这个blocker可以通过 `getBlocker()` 获取。但是没太大用。

**恢复线程**

```java
public static void unpark(Thread thread);
```
**测试**

```java
public class T13_TestLockSupport {
    public static void main(String[] args) {
        Thread t = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                System.out.println(i);
                if (i == 5) {
                    LockSupport.park();
                }
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        t.start();
        
        try {
            TimeUnit.SECONDS.sleep(8);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("after 8 senconds!");
        LockSupport.unpark(t);          // t.interrupt() 也可以使其运行
    }
}
```

注意，park 的时候不释放临界资源。

```java
public class T13_TestLockSupport2 {
    static Object object = new Object();
    public static void main(String[] args) throws Exception {
        Thread t = new Thread(() -> {
            synchronized (object) {
                for (int i = 0; i < 10; i++) {
                    System.out.println(i);
                    if (i == 5) {
                        LockSupport.park();
                    }
                    try {
                        TimeUnit.SECONDS.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
        t.start();
        TimeUnit.SECONDS.sleep(1);
        new Thread(() -> {
            synchronized (object){
                System.out.println("hahahahah");
            }
        }).start();
        TimeUnit.SECONDS.sleep(8);
        System.out.println("after 8 senconds!");
        LockSupport.unpark(t);          // t.interrupt() 也可以使其运行
    }
}
```



## synchronized

按照时间发展呢的顺序，Java中是先出现了 `synchronized`（since 1.0），再出现了 `Lock`（since 5.0）。

在Java中，每一个对象都拥有一个锁标记（monitor），也称为监视器，我们可以使用 `synchronized` 关键字来标记一个方法或者代码块，当某个线程调用该对象的 `synchronized` 方法或者访问 `synchronized` 代码块时，这个线程便获得了该对象的锁，其他线程暂时无法访问这个方法，只有等待这个方法执行完毕或者代码块执行完毕，这个线程才会释放该对象的锁，其他线程才能执行这个方法或者代码块。

- `synchronized` 方法：

```java
public synchronized void insert(){
	
}
```

普通方法获得当前对象的锁，即 `this` 的锁。静态方法获得类的字节码对象的锁。

- `synchronized` 代码块

```java
synchronized(synObject) {
	
}
```

对于 `synchronized` 方法或者 `synchronized` 代码块，当出现异常时，JVM会自动释放当前线程占用的锁，因此不会由于异常导致出现死锁现象。



### synchronized 状态转换

每个锁对象都有一个条件队列，同时有一个阻塞队列，`wait()`之后进入的是条件队列，`notify()`唤醒的也是条件队列。阻塞队列用于等待不能进入临界区的线程。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/857591dc-a9a9-4aa2-845f-24f6fb6e8023" /></div>
### 条件对象

当一个线程进入临界区后却发现某一条件被满足之后它才能执行，比如在银行转账时，A向B账户转账，但是当A账户获得锁后，发现账户中没有钱，需要等待C账户给其转账之后其才能给B账户转账，这时它就需要释放锁，进入等待状态，并且当其的账户余额能保证向B转完账后不为负数这个条件时才能转账，同时当此条件被满足时其他线程需要通知等待的线程让其进入运行状态。方法如下：

- synchronized方法

```java
public synchronized void test(){
	if(条件x不满足)
		wait();
	if(条件x被满足)
		notify() or notifyAll()  // 唤醒等待在条件x上的线程
}
```

- synchronized代码块

```java
synchronized(synObject) {
	if(条件x不满足)
		wait();
	if(条件x被满足)
		notify() or notifyAll()  // 唤醒等待在条件x上的线程
}
```

但是如果还有一个条件y可以迫使线程进入等待状态，在编程时只能将其也等待在条件x上，这就是其不足之一。



## Lock

Lock接口定义的方法如下：

```java
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();
    Condition newCondition();
}
```

- `tryLock()`方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回true，如果获取失败（即锁已被其他线程获取），则返回 false ，也就说这个方法无论如何都会立即返回。在拿不到锁时不会一直在那等待。
- `tryLock(long time, TimeUnit unit)`方法和`tryLock()`方法是类似的，只不过区别在于这个方法在拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回false。如果如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。

Lock接口的典型使用方法如下：

```java
Lock lock = ...;
lock.lock();
try {
    // 处理任务
} catch (Exception ex) {
     
} finally {
    lock.unlock();   // 释放锁
}
```



### ReentrantLock

翻译为是“可重入锁”，意思是如果锁具备可重入性，则称作为可重入锁。像 `synchronized` 和 `ReentrantLock` 都是可重入锁，可重入性实际上表明了锁的分配机制：基于线程的分配，而不是基于方法调用的分配。举个简单的例子，当一个线程执行到某个 `synchronized` 方法时，比如说 `method1`，而在 `method1` 中会调用另外一个`synchronized` 方法 `method2`，此时线程不必重新去申请锁，而是可以直接执行方法 `method2`。

按不同的分类，还有一类锁是中断锁，顾名思义，就是可以响应中断的锁。在Java中，`synchronized` 就不是可中断锁，而 `Lock` 是可中断锁。如果某一线程A正在执行锁中的代码，另一线程B正在等待获取该锁，可能由于等待时间过长，线程B不想等待了，想先处理其他事情，我们可以让它中断自己或者在别的线程中中断它，这种就是可中断锁。

```java
public class Test {
    private Lock lock = new ReentrantLock();   
    public static void main(String[] args)  {
        Test test = new Test();
        MyThread thread0 = new MyThread(test);
        MyThread thread1 = new MyThread(test);
        thread0.start();
        thread1.start();
        
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        thread1.interrupt();
    }  
     
    public void insert(Thread thread) throws InterruptedException{
        lock.lockInterruptibly();   
        // 如果需要正确中断等待锁的线程，必须将获取锁放在外面，然后将InterruptedException抛出
        try {  
            System.out.println(thread.getName() + "得到了锁");
            long startTime = System.currentTimeMillis();
            for(;;) {
                if(System.currentTimeMillis() - startTime >= Integer.MAX_VALUE)
                    break;
                /*
                 *	插入数据
                 */
            }
        } finally {
            System.out.println(Thread.currentThread().getName() + "执行finally");
            lock.unlock();
            System.out.println(thread.getName()+"释放了锁");
        }
    }
}
 
class MyThread extends Thread {
    private Test test = null;
    public MyThread(Test test) {
        this.test = test;
    }
    @Override
    public void run() {
        try {
            test.insert(Thread.currentThread());
        } catch (InterruptedException e) {
            System.out.println(Thread.currentThread().getName() + "被中断");
        }
    }
}
```

在这段代码中，如果线程1首先获得锁，其会一直运行下去，此时线程0得不到锁就会永远等待下去。但是如果线程0首先获得锁，其会一直运行下去，所以此时线程1得不到锁，但是在主线程中线程1启用了 `interrupt()` 方法，而 `lockInterruptibly()` 可以响应中断。



## thread的状态

<div align='center'><img src="http://blogfileqiniu.isjinhao.site/870d4660-c7a1-42b8-ac3c-2630fcde0a21" /></div>
### yield()

```java
// JDK原型
public static native void yield();
```

> A hint to the scheduler that the current thread is willing to yield its current use of a processor. The scheduler is free to ignore this hint.

当 `yield()` 成功的时候会自动放弃时间片，转入就绪状态，然后和其它线程进行CPU的争夺。但是这个方法不一定执行成功，线程调度器可以选择忽略这个方法。



### join()

join方法有三个重载版本：

```
join()
join(long millis)     //参数为毫秒
join(long millis,int nanoseconds)    //第一参数为毫秒，第二个参数为纳秒
```

假如在main线程中，调用 `thread.join` 方法，则 main 线程会等待thread线程执行完毕或者等待一定的时间。

- 如果调用的是无参 `join()` 方法，则等待 thread 执行完毕。
- 如果调用的是指定了时间参数的 `join` 方法，则等待一定的时间。

#### join的实现

```java
public final synchronized void join(long millis)
	 throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) {
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

`join()` 是使用 `wait()` 来实现的，如果线程仍然活着，则等待对应的时间。当调用线程（设为A）执行到其他线程（设为B）的 `join()` 方法时，A阻塞在线程B的this对象（线程B本身），如第12行或第20行所示。从代码上我们看不出来什么时候 `notify` 线程A，但是JDK注释上描述：

> As a thread terminates the this.notifyAll method is invoked.

笔者也是在此知道，当一个线程结束时会通知所有在其上等待的线程。同时 `join()` 是一个不会放弃锁的操作。

```java
public class TT02_T {
    static Object obj = new Object();
    public static void main(String[] args) throws Exception {
        System.out.println("main begin");
        Thread thread = new Thread(() -> {
            System.out.println("thread start");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("thread get lock begin");
            synchronized (obj) {
                System.out.println("thread run concurrent program");
            }
            System.out.println("thread get lock success");

        });

        thread.start();

        Thread.sleep(1000);

//        synchronized (obj){
            System.out.println("thread join main begin");
            thread.join();
            System.out.println("thread join main end");
//        }
    }
}
```

如果将注释的代码放开就会产生死锁，因为 main 线程在24行拿到了锁，但是执行 `join()` 方法时会去申请同一把锁，但是 `join()`  不会释放锁，所以第13行申请不到锁就会陷入等待，同时第26行中 main 线程又在等待 thread 线程执行完，所以就会陷入死锁。



## JVM的内存结构

Java 虚拟机规范中试图定义一种 Java 内存模型（Java Memory Model，JMM）来屏蔽掉各种硬件和操作系统的内存访问差异，以实现让 Java 程序在各种平台下都能达到一致的内存访问效果。

Java 内存模型的主要目标是定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存中取出变量这样的底层细节。此处的变量（Variables）与 Java 编程中所说的变量有所区别，它包括了实例字段、静态字段和构成数组对象的元素，但不包括局部变量与方法参数，因为后者是线程私有的，不会被共享，自然就不会存在竞争问题。为了获得较好的执行效能，Java 内存模型并没有限制执行引擎使用处理器的特定寄存器或缓存和主内存进行交互，也没有限制即时编译器进行调整代码执行顺序这类优化措施。

Java 内存模型规定了所有的变量都存储在主内存（Main Memory）中。每条线程还有自己的工作内存（Working Memory），线程的工作内存中保存了被该线程使用到的变量的主内存副本拷贝，线程对变量的所有操作（读取、赋值等）都必须在工作内存中进行，而不能直接读写主内存中的变量。不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递均需要通过主内存来完成，线程、主内存、工作内存三者的交互关系如图所示。

<div align='center'><img width="60%" src="http://blogfileqiniu.isjinhao.site/e42edafc-24d7-4c3d-81be-cbdeb10049c5" /></div>
### volatile

关键字 `volatile` 可以说是 Java 虚拟机提供的最轻量级的同步机制。当一个变量定义为 `volatile` 之后，它将具备两种特性：

**保证此变量对所有线程的可见性**

这里的 “可见性” 是指当一条线程修改了这个变量的值，新值对于其他线程来说是可以立即得知的。而普通变量不能做到这一点，普通变量的值在线程间传递均需要通过主内存来完成，例如，线程 A 修改一个普通变量的值，然后向主内存进行回写，另外一条线程 B 在线程 A 回写完成了之后再从主内存进行读取操作，新变量值才会对线程 B 可见。关于 `volatile` 变量的可见性，经常会被开发人员误解，认为以下描述成立：“`volatile` 变量对所有线程是立即可见的，对 `volatile` 变量所有的写操作都能立刻反应到其他线程之中，换句话说，`volatile` 变量在各个线程中是一致的，所以基于 `volatile` 变量的运算在并发下是安全的”。这句话的论据部分并没有错，但是其论据并不能得出 “基于 `volatile` 变量的运算在并发下是安全的” 这个结论 。比如以下代码：

```java
public class Test {
    public volatile int inc = 0;
    public void increase() {
        inc++;
    }
    public static void main(String[] args) throws Exception {
        final Test test = new Test();
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++)
                    test.increase();
            }).start();
        }
       Thread.sleep(5000);  // 保证前面的线程都执行完
       System.out.println(test.inc);
    }
}
```

运行它会发现每次运行结果都不一致，都是一个小于10000的数字。这便是由于 `volatile` 不能保证原子性。同时自增操作是不具备原子性的，它包括读取变量的原始值、进行加1操作、写入工作内存。那么就是说自增操作的三个子操作可能会分割开执行，就有可能导致下面这种情况出现：

- 假如某个时刻变量inc的值为10。
- 线程1对变量进行自增操作，线程1先读取了变量inc的原始值，然后线程1被阻塞了；
- 然后线程2对变量进行自增操作，线程2也去读取变量inc的原始值，由于线程1只是对变量inc进行读取操作，而没有对变量进行修改操作，所以不会导致线程2的工作内存中缓存变量inc的缓存行无效，所以线程2会直接去主存读取inc的值，发现inc的值时10，然后进行加1操作，并把11写入工作内存，最后写入主存。
- 然后线程1接着进行加1操作，由于已经读取了inc的值，注意此时在线程1的工作内存中inc的值仍然为10，所以线程1对inc进行加1操作后inc的值为11，然后将11写入工作内存，最后写入主存。
- 那么两个线程分别进行了一次自增操作后，inc只增加了1。

**禁止指令重排序优化**

普通的变量仅仅会保证在该方法的执行过程中所有依赖赋值结果的地方都能获取到正确的结果，而不能保证变量赋值操作的顺序与程序代码中的执行顺序一致。因为在一个线程的方法执行过程中无法感知到这点，这也就是 Java 内存模型中描述的所谓的 “线程内表现为串行的语义”（Within-Thread As-If-Serial Semantics）。

`volatile` 关键字禁止指令重排序有两层意思：

- 当程序执行到 `volatile` 变量的读操作或者写操作时，在其前面的操作的更改肯定全部已经进行，且结果已经对后面的操作可见；在其后面的操作肯定还没有进行；
- 在进行指令优化时，不能将在对volatile变量访问的语句放在其后面执行，也不能把 `volatile` 变量后面的语句放到其前面执行。

可能上面说的比较绕，举个简单的例子：

```
// x、y为非volatile变量
// flag为volatile变量
x = 2;   // 语句1
y = 0;   // 语句2
flag = true;  // 语句3
x = 4;  // 语句4
y = -1;     // 语句5`
```

由于 flag 变量为 `volatile` 变量，那么在进行指令重排序的过程的时候，不会将语句3放到语句1、语句2前面，也不会讲语句3放到语句4、语句5后面。但是要注意语句1和语句2的顺序、语句4和语句5的顺序是不作保证的。

并且 `volatile` 关键字能保证，执行到语句3时，语句1和语句2必定是执行完毕了的，且语句1和语句2的执行结果对语句3、语句4、语句5是可见的。那么我们看一个例子：

```
// 线程1:
context = loadContext();   // 语句1
inited = true;             // 语句2
 
// 线程2:
while(!inited ){
  sleep();
}
doSomethingwithconfig(context);
```

前在这个例子中，有可能语句2会在语句1之前执行，那么久可能导致 context 还没被初始化，而线程2中就使用未初始化的 context 去进行操作，导致程序出错。

这里如果用 `volatile` 关键字对 inited 变量进行修饰，就不会出现这种问题了，因为当执行到语句2时，必定能保证 context 已经初始化完毕。



### volatile关键字的场景

所以总结来说，`volatile` 变量在各个线程的工作内存中不存在一致性问题（在各个线程的工作内存中，`volatile` 变量也可以存在不一致的情况，但由于每次使用之前都要先刷新，执行引擎看不到不一致的情况，因此可以认为不存在不一致性问题），但是 Java 里面的运算并非原子操作，导致 `volatile` 变量的运算在并发下一样是不安全的。由于 `volatile` 变量只能保证可见性，在不符合以下两条规则的运算场景中，我们仍然需要通过加锁（使用 `synchronized` 或 `java.util.concurrent` 中的原子类）来保证原子性。通常来说，使用 `volatile` 必须具备以下2个条件：

- 对变量的写操作不依赖于当前值
- 该变量没有包含在具有其他变量的不变式中

实际上，这些条件表明，可以被写入 `volatile` 变量的这些有效值独立于任何程序的状态，包括变量的当前状态。事实上，我的理解就是上面的2个条件需要保证操作是原子性操作，才能保证使用 `volatile` 关键字的程序在并发时能够正确执行。下面列举几个Java中使用 `volatile` 的几个场景。

**状态标记量**

```java
volatile boolean flag = false;

while(!flag){
    doSomething();
}
 
public void setFlag() {
    flag = true;
}
```

```java
volatile boolean inited = false;
// 线程1:
context = loadContext();  
inited = true;

// 线程2:
while(!inited){
	sleep()
}
doSomethingwithconfig(context);
```

**DCL**

```java
class Singleton {
    // 如果不加volatile可能拿到半个实例
    private volatile static Singleton instance = null;
    private Singleton() { }
    public static Singleton getInstance() {
        // 此层判断的目的是在instance初始化完成之后，直接返回
        if(instance == null) {
            // 可能会有多个线程到达此步，对字节码加锁的目的是使保证只能被构造一次
            synchronized (Singleton.class) {
                // 进入第10行的线程在正常情况下一定会进入到此步，再判断一次，如果被构造了则不再构造
                if(instance == null)
                    instance = new Singleton();
            }
        }
        return instance;
    }
}
```



### Happens-Before原则

很多时候我们写代码会觉得有些东西是理所当然的，比如写在前面的代码先于写在后面的代码执行，但是通过我们到现在为止的解释就知道，经过指令重排序之后，写在前面的代码不一定会比写在后面的代码先执行。这对于程序员来说便是一个灾难，因为你不再知道自己的代码会在什么时候执行的。所以Java提出了一个Happens-Before原则，它的意思是说，在这些规则规定的情况下，如果操作A先行发生于操作B，那么操作A产生的影响能被操作B观察到。如果两个操作之间的关系不在这些规则中间，并且无法从下列规则推导出来，则它们就没有顺序性保障，虚拟机可以对它们随意地进行重排序。

1. 程序次序规则：**一个线程内**，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作
2. 锁定规则：一个 unLock 操作先行发生于后面对同一个锁的 lock 操作
3. volatile 变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作
4. 传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C
5. 线程启动规则：Thread对象的 `start()` 方法先行发生于此线程的每个一个动作
6. 线程中断规则：对线程 `interrupt()` 方法的调用先行发生于被中断线程的代码检测到中断事件的发生
7. 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过 `Thread.join()` 方法结束、`Thread.isAlive() `的返回值手段检测到线程已经终止执行
8. 对象终结规则：一个对象的初始化完成先行发生于他的 `finalize()` 方法的开始

这8条规则中，前4条规则是比较重要的，后4条规则都是显而易见的。下面我们来解释一下前4条规则：

1. 对于程序次序规则：一段程序代码的执行在单个线程中看起来是有序的。注意，虽然这条规则中提到“书写在前面的操作先行发生于书写在后面的操作”，这个应该是程序看起来执行的顺序是按照代码顺序执行的，因为虚拟机可能会对程序代码进行指令重排序。虽然进行重排序，但是最终执行的结果是与程序顺序执行的结果一致的，它只会对不存在数据依赖性的指令进行重排序。因此，在单个线程中，程序执行看起来是有序执行的，这一点要注意理解。事实上，这个规则是用来保证程序在单线程中执行结果的正确性，而DCL问题说的是在多线程中无法保证执行结果的正确性。
2. 锁定规则：也就是说无论在单线程中还是多线程中，同一个锁如果出于被锁定的状态，那么必须先对锁进行了释放操作，后面才能继续进行 lock 操作。
3. volatile变量规则：如果一个线程先去写一个变量，然后一个线程去进行读取，那么写的结果肯定能被读到。
4. 传递规则：实际上就是体现 happens-before 原则具备传递性。这也保证了此原则能被扩大使用。



## 协程

Java 线程的实现方式并没有被 Java 虚拟机规范所规定，但是 HotSpot 虚拟机才用的方式是通过将每一个 Java 线程直接映射到一个操作系统原生线程来实现。同时 HotSpot 自己是不会去干涉线程调度的（可以设置线程优先级给操作系统提供调度建议），全权交给底下的操作系统去处理，所以何时冻结或唤醒线程、该给线程分配多少处理器执行时间、该把线程安排给哪个处理器核心去执行等，都是由操作系统完成的，也都是由操作系统全权决定的。

这种一对一的模型在传统的单体服务上由于一次请求中计算和网络等所消耗的时间远大于线程切换的消耗，所以并没有太大的问题。但是在微服务时代，每个机器都要接受巨大的请求，同时一次请求会形成好多次服务的调用，而每个机器只负责和自己相关的部分。所以每个机器上计算所耗费的时间甚至小于线程切换所带来的消耗，这就会造成严重的浪费。

而协程是在用户态进行线程切换的一种技术，也就是说线程的切换交由虚拟机完成而不是操作系统完成。虽然协程是在用户态完成线程切换，但是它依然会造成上下文切换，所以我们不能说协程可以消除线程切换带来的消耗。

协程和线程的最大区别是，协程是协作式多任务的，而线程通常是抢先式多任务的。抢占式多任务造成的后果是操作系统也不知道一个时间片执行完了下一个实现片由哪个线程获得，所以对上下文的破坏是巨大的。协作时多任务的时候可以通过合理的安排降低线程切换的次数。

协程的主要优势是轻量，无论是有栈协程还是无栈协程，都要比传统内核线程要轻量得多。如果进行量化的话，那么如果不显式设置 -Xss 或 -XX：ThreadStackSize ，则在64位Linux上 HotSpot 的线程栈容量默认是1MB，此外内核数据结构（Kernel Data Structures）还会额外消耗16KB内存。与之相对的，一个协程的栈通常在几百个字节到几KB之间，所以 Java 虚拟机里线程池容量达到两百就已经不算小了，而很多支持协程的应用中，同时并存的协程数量可数以十万计。

由于Java目前并没引入协程在真正的开发中，所以我们仅仅了解一下协程就可以。



## 锁优化

### 锁消除

锁消除是指虚拟机即时编译器在运行时，对一些同步代码，但是被检测到不可能存在共享数据竞争的锁进行消除。锁消除的主要判定依据来源于逃逸分析的数据支持，如果判断到一段代码中，在堆上的所有数据都不会逃逸出去被其他线程访问到，那就可以把它们当作栈上数据对待，认为它们是线程私有的，同步加锁自然就无须再进行。也许读者会有疑问，变量是否逃逸，对于虚拟机来说是需要使用复杂的过程间分析才能确定的，但是程序员自己应该是很清楚的，怎么会在明知道不存在数据争用的情况下还要求同步呢？这个问题的答案是：有许多同步措施并不是程序员自己加入的，同步的代码在Java程序中出现的频繁程度也许超过了大部分读者的想象。

```java
public String concatString(String s1, String s2, String s3) {
	return s1 + s2 + s3;
}
```

这段代码如果在JDK5的编译下，会产生如下代码：

````java
public String concatString(String s1, String s2, String s3) {
	StringBuilder sb = new StringBuffer();
	sb.append(s1);
	sb.append(s2);
	sb.append(s3);
	return sb.toString();
}
````

每个 `StringBuffer.append()` 方法中都有一个同步块，锁就是sb对象。虚拟机观察变量sb，经过逃逸分析后会发现它的动态作用域被限制在 `concatString()` 方法内部。也就是sb的所有引用都永远不会逃逸到 `concatString()` 方法之外，其他线程无法访问到它，所以这里虽然有锁，但是可以被安全地消除掉。



### 锁粗化

原则上，我们在编写代码的时候，总是推荐将同步块的作用范围限制得尽量小——只在共享数据的实际作用域中才进行同步，这样是为了使得需要同步的操作数量尽可能变少，即使存在锁竞争，等待锁的线程也能尽可能快地拿到锁。大多数情况下，上面的原则都是正确的，但是如果一系列的连续操作都对同一个对象反复加锁和解锁，甚至加锁操作是出现在循环体之中的，那即使没有线程竞争，频繁地进行互斥同步操作也会导致不必要的性能损耗。比如我们上面的代码如果没有被虚拟机进行锁消除，那么就会进行反复的加锁。

所以如果虚拟机探测到有这样一串零碎的操作都对同一个对象加锁，将会把加锁同步的范围扩展（粗化）到整个操作序列的外部，就是扩展到第一个 `append()` 操作之前直至最后一个 `append()` 操作之后，这样只需要加锁一次就可以了。



### 自旋锁

由于Java线程的实现是一个用户级线程对应一个内核线程，所以挂起线程和恢复线程的操作都需要转入内核态中完成，这样在共享数据的锁定状态只会持续很短的一段时间的情景下，为了这段时间去挂起和恢复线程并不值得。现在绝大多数的个人电脑和服务器都是多核处理器系统，如果物理机器有一个以上的处理器或者处理器核心，能让两个或以上的线程同时并行执行，我们就可以让后面请求锁的那个线程“稍等一会”，但不放弃处理器的执行时间，看看持有锁的线程是否很快就会释放锁。为了让线程等待，我们只须让线程执行一个忙循环（自旋），这项技术就是所谓的自旋锁。

自旋等待不能代替阻塞，且先不说对处理器数量的要求，自旋等待本身虽然避免了线程切换的开销，但它是要占用处理器时间的，所以如果锁被占用的时间很短，自旋等待的效果就会非常好，反之如果锁被占用的时间很长，那么自旋的线程只会白白消耗处理器资源，而不会做任何有价值的工作，这就会带来性能的浪费。因此自旋等待的时间必须有一定的限度，如果自旋超过了限定的次数仍然没有成功获得锁，就应当使用传统的方式去挂起线程。

自旋次数的默认值是十次，用户也可以使用参数-XX：PreBlockSpin来自行更改。不过无论是默认值还是用户指定的自旋次数，对整个Java虚拟机中所有的锁来说都是相同的。在 JDK6 中对自旋锁的优化，引入了自适应的自旋。自适应意味着自旋的时间不再是固定的了，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定的。如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也很有可能再次成功，进而允许自旋等待持续相对更长的时间，比如持续100次忙循环。另一方面，如果对于某个锁，自旋很少成功获得过锁，那在以后要获取这个锁时将有可能直接省略掉自旋过程，以避免浪费处理器资源。有了自适应自旋，随着程序运行时间的增长及性能监控信息的不断完善，虚拟机对程序锁的状况预测就会越来越精准，虚拟机就会变得越来越“聪明”了。



### 锁升级

### 对象的内存布局

要理解轻量级锁，以及后面会讲到的偏向锁的原理和运作过程，必须要对 HotSpot 虚拟机对象的内存布局（尤其是对象头部分）有所了解。HotSpot 虚拟机的对象头（Object Header）分为两部分，第一部分用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄（Generational GC Age）等。这部分数据的长度在32位和64位的Java虚拟机中分别会占用32个或64个比特，官方称它为“Mark Word”。这部分是实现轻量级锁和偏向锁的关键。另外一部分用于存储指向方法区对象类型数据的指针，如果是数组对象，还会有一个额外的部分用于存储数组长度。

由于对象头信息是与对象自身定义的数据无关的额外存储成本，考虑到Java虚拟机的空间使用效率，Mark Word被设计成一个非固定的动态数据结构，以便在极小的空间内存储尽量多的信息。它会根据对象的状态复用自己的存储空间。例如在32位的 HotSpot 虚拟机中，对象未被锁定的状态下，Mark Word 的32个比特空间里的25个比特将用于存储对象哈希码，4个比特用于存储对象分代年龄，2个比特用于存储锁标志位，还有1个比特固定为0（这表示未进入偏向模式）。对象除了未被锁定的正常状态外，还有轻量级锁定、重量级锁定、GC 标记、可偏向等几种不同状态。

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/38d46335-643c-47f6-94ce-d65ba9a35326" /></div>
### 偏向锁

HotSpot 的作者经过以往的研究发现大多数情况下锁不仅不存在多线程竞争，而且总是由同一线程多次获得，为了让线程获得锁的代价更低而引入了偏向锁。当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要花费 CAS 操作来加锁和解锁，而只需简单的测试一下对象头的 Mark Word 里是否存储着指向当前线程的偏向锁，如果测试成功，表示线程已经获得了锁，如果测试失败，则需要再测试下 Mark Word 中偏向锁的标识是否设置成1（表示当前是偏向锁），如果没有设置，则使用 CAS 竞争锁，如果设置了，则尝试使用 CAS 将对象头的偏向锁指向当前线程。

传统的锁的思想是一旦某个临界资源被某线程（设为A）获取，其他的线程只能等待A线程释放。而偏向锁只是在锁上打一个标记，标识着这个锁目前被A线程使用，这样在不存在锁竞争的情况下，可以消除同步机制的性能消耗。但是一旦有其他线程来竞争锁，就必须采用一定的机制来保证临界资源的安全性。也就是锁会升级。

**偏向锁的撤销**：偏向锁使用了一种等到竞争出现才释放锁的机制，所以当其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁。偏向锁的撤销，需要等待全局安全点（在这个时间点上没有字节码正在执行），它会首先暂停拥有偏向锁的线程，然后检查持有偏向锁的线程是否活着，如果线程不处于活动状态，则将对象头设置成无锁状态，如果线程仍然活着，拥有偏向锁的栈会被执行，遍历偏向对象的锁记录，栈中的锁记录和对象头的 Mark Word 要么重新偏向于其他线程，要么恢复到无锁或者标记对象不适合作为偏向锁，最后唤醒暂停的线程。下图中的线程1演示了偏向锁初始化的流程，线程2演示了偏向锁撤销的流程。

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/%E5%81%8F%E5%90%91%E9%94%81%E7%9A%84%E6%92%A4%E9%94%80.png" /></div>
### 轻量级锁

轻量级锁是 JDK6 时加入的新型锁机制，它名字中的“轻量级”是相对于使用操作系统互斥量来实现的传统锁而言的，因此传统的锁机制就被称为“重量级”锁。不过，需要强调一点，轻量级锁并不是用来代替重量级锁的，它设计的初衷是在没有多线程竞争的前提下，减少传统的重量级锁使用操作系统互斥量产生的性能消耗。

在代码即将进入同步块的时候，如果此同步对象没有被锁定（锁标志位为“01”状态），虚拟机首先将在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，用于存储锁对象目前的 Mark Word 的拷贝（官方为这份拷贝加了一个 Displaced 前缀，即 Displaced Mark Word ），这时候线程堆栈与对象头的状态如下图所示。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/68037ee9-78b6-4b52-bb8c-c74ebfef6795" /></div>
然后，虚拟机将使用 CAS 操作尝试把对象的 Mark Word 更新为指向 Lock Record 的指针。如果这个更新动作成功了，即代表该线程拥有了这个对象的锁，并且对象Mark Word的锁标志位（Mark Word的最后两个比特）将转变为“00”，表示此对象处于轻量级锁定状态。这时候线程堆栈与对象头的状态如下图所示。

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/4ddfb0df-b445-42e2-bdfb-2613f25ad9e6" /></div>
如果这个更新操作失败了，那就意味着至少存在一条线程与当前线程竞争获取该对象的锁。虚拟机首先会检查对象的 Mark Word 是否指向当前线程的栈帧，如果是，说明当前线程已经拥有了这个对象的锁，那直接进入同步块继续执行就可以了，否则就说明这个锁对象已经被其他线程抢占了。如果出现两条以上的线程争用同一个锁的情况，那轻量级锁就不再有效，必须要膨胀为重量级锁，锁标志的状态值变为“10”，此时 Mark Word 中存储的就是指向重量级锁（互斥量）的指针，后面等待锁的线程也必须进入阻塞状态。

上面描述的是轻量级锁的加锁过程，它的解锁过程也同样是通过 CAS 操作来进行的，如果对象的 Mark Word 仍然指向线程的锁记录，那就用 CAS 操作把对象当前的 Mark Word 和线程中复制的 Displaced Mark Word 替换回来。假如能够成功替换，那整个同步过程就顺利完成了；如果替换失败，则说明有其他线程尝试过获取该锁，就要在释放锁的同时，唤醒被挂起的线程。

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/%E5%81%8F%E5%90%91%E9%94%81%E7%9A%84%E6%92%A4%E9%94%80.png" /></div>
### 详细的锁升级图

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/synchronized%E5%8D%87%E7%BA%A7.jpg" /></div>