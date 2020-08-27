## AQS

Java并发包（JUC）中提供了很多并发工具，这其中，很多我们耳熟能详的并发工具，譬如ReentrangLock、Semaphore，它们的实现都用到了一个共同的基类--**AbstractQueuedSynchronizer**，简称AQS。AQS是一个用来构建锁和同步器的框架，使用AQS能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的Semaphore，ReentrantLock，ReentrantReadWriteLock 等等皆是基于AQS的。当然，我们自己也能利用AQS非常轻松容易地构造出符合我们自己需求的同步器。

**AQS 原理概览**

AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。

> CLH（Craig、Landin and Hagersten）：CLH队列是一个单向链表，AQS中的队列是CLH变体的虚拟双向队列（FIFO），AQS是通过将每条请求共享资源的线程封装成一个节点来实现锁的分配。

看个AQS（AbstractQueuedSynchronizer）原理图：

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/d48cb1f9-7a07-41dd-8a7e-7a089619db97" /></div>
AQS使用一个int成员变量来表示同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。AQS使用CAS对该同步状态进行原子操作实现对其值的修改。

```java
private volatile int state; // 共享变量，使用volatile修饰保证线程可见性
```

状态信息通过 protected 类型的`getState()`，`setState()`，`compareAndSetState()`进行操作

```java
// 返回同步状态的当前值
protected final int getState() {  
    return state;
}
 // 设置同步状态的值
protected final void setState(int newState) { 
    state = newState;
}
// 原子地（CAS操作）将同步状态值设置为给定值update如果当前同步状态的值等于expect（期望值）
protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

**AQS 对资源的共享方式**

- Exclusive（独占）：只有一个线程能执行，如 `ReentrantLock`。又可分为公平锁和非公平锁：
  - 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
  - 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
- Share（共享）：多个线程可同时执行，如Semaphore/CountDownLatch。

ReentrantReadWriteLock 可以看成是组合式，因为 ReentrantReadWriteLock 也就是读写锁允许多个线程同时对某一资源进行读。

不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可（`tryReleaseShared()`、`tryRelease()`、`tryAcquire()`和`tryAcquireShared()`），至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在上层已经帮我们实现好了。

**AQS底层使用了模板方法模式**

同步器的设计是基于模板方法模式的，如果需要自定义同步器一般的方式是这样：

1. 使用者继承 `AbstractQueuedSynchronizer` 并重写指定的方法。（这些重写方法很简单，无非是对于共享资源 state 的获取和释放）
2. 将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。

**AQS使用了模板方法模式，自定义同步器时需要重写下面几个AQS提供的模板方法：**

```java
isHeldExclusively()		// 该线程是否正在独占资源。只有用到Condition才需要去实现它。
tryAcquire(int)			// 独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)			// 独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)	// 共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数
    					// 表示成功，且有剩余资源。
tryReleaseShared(int)	// 共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

默认情况下，每个方法都抛出 `UnsupportedOperationException`。 这些方法的实现必须是内部线程安全的，并且通常应该简短而不是阻塞。AQS类中的其他方法都是 final ，所以无法被其他类使用，只有这几个方法可以被其他类使用。 

以 ReentrantLock 为例，state 初始化为0，表示未锁定状态。A线程 `lock()` 时，会调用 `tryAcquire()` 独占该锁并将 state+1。此后，其他线程再 `tryAcquire()` 时就会失败，直到A线程 `unlock()` 到 state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。

再以 `CountDownLatch` 以例，state 初始化为N（注意N要与线程个数一致）。每个线程执行完后 `countDown()` 一次，state 会 CAS 减1。等到所有子线程都执行完后（即state=0），会 `unpark()` 主调用线程，然后主调用线程就会从 `await()` 函数返回，继续后续动作。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现`tryAcquire-tryRelease`、`tryAcquireShared-tryReleaseShared`中的一种即可。但AQS也支持自定义同步器同时实现独占和共享两种方式，如`ReentrantReadWriteLock`。



### Node

Node结点是对每一个访问同步代码的线程的封装，其包含了需要同步的线程本身以及线程的状态，如是否被阻塞，是否等待唤醒，是否已经被取消等。变量 waitStatus 则表示当前被封装成Node结点的等待状态，共有4种取值CANCELLED、SIGNAL、CONDITION、PROPAGATE（传播）。

- CANCELLED：值为1。在同步队列中等待的线程等待超时或被中断，处于此状态的结点不会再转为其他状态，尤其此节点里的线程不会再被阻塞。
- SIGNAL：值为-1。标志着此结点的后继结点是阻塞状态，此结点运行时后续结点需要进行登台，其释放资源或进入CANCELLED状态后需要唤醒后续结点。
- CONDITION：值为-2。标识的结点处于等待队列中，当其他线程调用了 Condition 的 `signal()` 方法后，CONDITION状态的结点将从等待队列转移到同步队列中，等待获取同步锁。
- PROPAGATE：值为-3。仅存在共享模式中，在共享模式中，表示Node的unpark可以向后传播。（类似读者写者问题中一个写者会阻塞N个读者，前N-1个读者就是PROPAGATE状态）。
- 0状态：值为0。代表初始化状态。

AQS在判断状态时，可以通过用 `waitStatus>0` 表示取消状态，而 `waitStatus<0` 表示有效状态。

#### 同步队列

同步队列中的线程不都是有意义的线程。有一个无意义的哨兵节点。同时按照JDK注释的话来说，这里也可以使用AtomicInteger，但使用原生CAS这样做是为了以后更好的增强性能。

> Setup to support compareAndSet. We need to natively implement this here: For the sake of permitting future enhancements, we cannot explicitly subclass AtomicInteger, which would be efficient and useful otherwise. So, as the lesser of evils, we natively implement using hotspot intrinsics API. And while we are at it, we do the same for other CASable fields (which could otherwise be done with atomic field updaters).

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/f2084acb-15ce-4f4f-a0ed-1d32734fe2b8" /></div>
同时我们知道CAS操作在使用的时候是需要一个内存地址的，代码中的体现就是下面的 `xxxOffset`。

```java
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long stateOffset;
private static final long headOffset;
private static final long tailOffset;
private static final long waitStatusOffset;
private static final long nextOffset;

static {
    try {
        stateOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("state"));
        headOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("head"));
        tailOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("tail"));
        waitStatusOffset = unsafe.objectFieldOffset
            (Node.class.getDeclaredField("waitStatus"));
        nextOffset = unsafe.objectFieldOffset
            (Node.class.getDeclaredField("next"));

    } catch (Exception ex) { throw new Error(ex); }
}
```



## ReetrantLock

`ReetrantLock` 是独占模式的AQS的实现，我们下面通过这个类来看看如何实现一个独占的AQS同步器。主要分析两部分：

- 通用的 `acquire-release` 模板
- 公平和非公平的 `tryAcquire()` 和 `tryRelease()` 实现。



### acquire(int)

此方法是独占模式下线程获取共享资源的顶层入口。如果获取到资源，线程直接返回，否则进入等待队列，直到获取到资源为止，且整个过程忽略中断的影响。这也正是 `lock()` 的语义，当然不仅仅只限于 `lock()`。获取到资源后，线程就可以去执行其临界区代码了。下面是 `acquire()` 的源码：

```java
public final void acquire(int arg) {
	if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
		selfInterrupt();
}
```

函数流程如下：

1. `tryAcquire()`：尝试直接去获取资源，如果成功则直接返回；
2. `addWaiter()`：如果 `tryAcquire()` 失败，将该线程加入等待队列的尾部，并标记为独占模式；
3. `acquireQueued()` 使线程在等待队列中获取资源，一直获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。
4. 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断 `selfInterrupt()`，将中断补上。这个补上中断仅仅是将中断标志位补上。但是此时这个中断标志是没有被清除的，也就是说此时 `isInterrupted()`方法返回是true。



**tryAcquire(int)**

> Attempts to acquire in exclusive mode. This method should query if the state of the object permits it to be acquired in the exclusive mode, and if so to acquire it.
>
> This method is always invoked by the thread performing acquire. If this method reports failure, the acquire method may queue the thread, if it is not already queued, until it is signalled by a release from some other thread.

此方法尝试去获取独占资源。如果获取成功，则直接返回true，否则直接返回false。这也正是 `tryLock()` 的语义，还是那句话，当然不仅仅只限于 `tryLock()`。如下是 `tryAcquire()`的源码：

```java
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

这个方法是我们再自定义同步器的时候需要实现的方法，之所以没有定义成abstract，是因为独占模式下只用实现tryAcquire-tryRelease，而共享模式下只用实现tryAcquireShared-tryReleaseShared。如果都定义成abstract，那么每个模式也要去实现另一模式下的接口。

**addWaiter(Node)**

此方法用于将当前线程加入到等待队列的队尾，并返回当前线程所在的结点：

```java
private Node addWaiter(Node mode) {
    // 以给定模式构造结点。mode有两种：EXCLUSIVE（独占）和SHARED（共享）
    Node node = new Node(Thread.currentThread(), mode);
    // 尝试快速方式直接放到队尾。
    Node pred = tail;
    if (pred != null) {	// 从enq()方法指导，tail==null时需要先创建一个Head
        node.prev = pred;
        // 使用CAS操作向末尾插入Node，compareAndSetTail(Node expect, Node update)
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // 上一步失败则通过enq入队。
    enq(node);
    return node;
}
```

**enq(Node)**

```java
private Node enq(final Node node) {
    // CAS"自旋"，直到成功加入队尾
    for (;;) {
        Node t = tail;
        if (t == null) { // 队列为空，创建一个空的标志结点作为head结点，并将tail也指向它。
            // compareAndSetHead(Node update)，进一步证实 Node 队列有一个哨兵节点
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {	// 正常流程，放入队尾
            node.prev = t;
            // CAS插入队尾
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

**acquireQueued(Node, int)**

通过 `tryAcquire()` 和 `addWaiter()` ，该线程获取资源失败，已经被放入等待队列尾部了。此时线程需要进入等待状态休息，直到其他线程彻底释放资源后唤醒自己。

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true; // 标记是否成功拿到资源
    try {
        boolean interrupted = false; // 标记等待过程中是否被中断过
        
        // 循环直到获取到资源
        for (;;) {
            final Node p = node.predecessor(); // 拿到前驱
            // 如果前驱是head，即该结点是CLH队列的老二（head节点是哨兵），就去尝试获取资源
            if (p == head && tryAcquire(arg)) {
                setHead(node);	// 拿到资源后，将head指向该结点。所以head所指的当前结点，
                				// 就是当前获取到资源的那个结点或null。
                p.next = null;  // setHead中node.prev已置为null，此处再将head.next置为null，
                			    // 此时意味着正在获得资源的线程出队了。
                failed = false;
                return interrupted;	// 返回等待过程中是否被中断过
            }
            
            // 如果自己可以休息了，就进入waiting状态，直到被unpark()
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true; // 如果等待过程中被中断过，哪怕只有那么一次，
            						// 就将interrupted标记为true
        }
    } finally {
        // 如果正常运行会在第16行返回，只有出现异常才能运行到finally块
        // 按照JDK注释所说，撤销状态是由于 timeout 和 interrupt，但是此时是 tryAcquire() 抛异常
        if (failed)
            cancelAcquire(node);
    }
}
```

**shouldParkAfterFailedAcquire(Node, Node)**

此方法主要用于检查状态，看看自己是否真的可以去休息了，万一队列前边的线程都是 CANCELLED 呢。

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus; // 拿到前驱的状态
    if (ws == Node.SIGNAL)
        // 前驱结点是SIGNAL，表示当前结点应该被阻塞。所以返回true。
        return true;
    if (ws > 0) {	// 前驱结点大于 0，表示前驱结点都是 CANCELLED 状态。
        // 一直往前找，直到找到最近一个正常等待的状态，并排在它的后边。
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        // 能运行到此，表示前驱的状态为0或者PROPAGATE。
        // 前驱结点修改为SIGNAL表示上一次SIGNAL的传播只能到前驱
        // 由于新节点的加入会导致前驱为SIGNAL，所以只有末尾的节点和运行的结点为0
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    // 返回false会进行一次自旋
    return false;
}
```

假设pred一开始指向 CANCELLED 结点。从第11行退出的时候结点连接图会发生如下转变：

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/4aeaba81-4f01-43c2-bf5c-eb8032560c1f" /></div>
此时在结点队列中已经没有引用 CANCELLED 结点的了。

**parkAndCheckInterrupt()**

如果线程找好安全休息点后，那就可以安心去休息了。此方法就是让线程去休息，真正进入等待状态。

```java
private final boolean parkAndCheckInterrupt() {
	LockSupport.park(this);			// 调用park()使线程进入waiting状态
	return Thread.interrupted();	// 如果被唤醒，查看自己是不是被中断的。
}
```

`park()` 会让当前线程进入 waiting 状态。在此状态下，有两种途径可以唤醒该线程：

- 被 `unpark()`
- 被 `interrupt()` 

这里使用  `Thread.interrupted()` 会清除线程的中断标志位。同时在清除以后，如果线程在等待过程中是被`interrupt()`唤醒的， `selfInterrupt()` 会补上一个中断信号。这一点与线程池有密切关系。

**cancelAcquire()**

```java
private void cancelAcquire(Node node) {
    // 忽略空结点
    if (node == null) return;

    // 释放线程
    node.thread = null;

    // 跳过前驱为 CANCELLED 的结点
    Node pred = node.prev;
    while (pred.waitStatus > 0)
        node.prev = pred = pred.prev;

    // 获得过滤后结点的后继结点
    Node predNext = pred.next;

    // 设置当前结点的状态
    node.waitStatus = Node.CANCELLED;

    // If we are the tail, remove ourselves.
    // compareAndSetTail(Node expect, Node update)
    if (node == tail && compareAndSetTail(node, pred)) {
        // compareAndSetNext(Node node, Node expect, Node update)
        compareAndSetNext(pred, predNext, null);
    } else {
        // If successor needs signal, try to set pred's next-link
        // so it will get one. Otherwise wake it up to propagate.
        int ws;		// pred 结点的状态
        // 前驱不是head && 前驱线程不空 && ((前驱状态为SIGNAL || 前驱有效且状态替换为SIGNAL成功))
		// 只有被CANCELLED的结点和HEAD的thread为null
        // 进入CAS的时候，ws为0或者PROPAGATE
        // 撤销过程是向前寻找的，而传播过程是向后的，所以pred结点可能已经执行完毕，此时CAS会失败
        // 所以当CAS失败的时候，node实际上就是头结点，需要进入unparkSuccessor()
        if (pred != head && ((ws = pred.waitStatus) == Node.SIGNAL ||
             (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
            pred.thread != null) {
            // 后继有效的时候
            Node next = node.next;
            if (next != null && next.waitStatus <= 0)
                compareAndSetNext(pred, predNext, next);
        } else {
            unparkSuccessor(node);
        }
        node.next = node; // help GC
    }
}
```

先解释一下 predNext，这个指向的是当前有效结点的后一个结点，注意它不是 node。

<div align="center"><img width="70%" src="http://blogfileqiniu.isjinhao.site/764b13cb-4493-4d8f-807a-278fa5c364df" /></div>
在撤销结点的时候一共有三种情况：

*node 是 tail 结点*

此时会将 pred 的后继结点设置为 null。

<div align="center"><img width="70%" src="http://blogfileqiniu.isjinhao.site/e47d7978-9160-4fcf-8f4c-e8acd1f5af11" /></div>
*node 的后继应该等待*

<div align="center"><img width="70%" src="http://blogfileqiniu.isjinhao.site/baeb95c0-205c-4bb3-832d-9b0a04a264ac" /></div>
*node 的后继应该被激活*

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/66063458-3730-423c-bc0e-2134e5349c53" /></div>
**小结**

`acquireQueued()` 分析完之后，我们接下来再回到 `acquire()`，再贴上它的源码吧：

```java
public final void acquire(int arg) {
	if (!tryAcquire(arg) &&
		acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
		selfInterrupt();
}
```

再来总结下它的流程吧：

1. 调用自定义同步器的 `tryAcquire()` 尝试直接去获取资源，如果成功则直接返回；
2. 没成功，则 `addWaiter()` 将该线程加入等待队列的尾部，并标记为独占模式；
3. `acquireQueued()` 使线程在等待队列中休息，有机会时（轮到自己，会被 `unpark()` ）会去尝试获取资源。获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。
4. 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断 `selfInterrupt()` ，将中断信号补上。



### release(int)

此方法是独占模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放了（即state=0）,它会唤醒等待队列里的其他线程来获取资源。这也正是 `unlock()` 的语义，当然不仅仅只限于 `unlock()`。下面是`release()` 的源码：

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;   // 找到头结点
        // 从下文的unparkSuccessor()可以看到，表示正在运行的状态会设置为0，所以此时不需要唤醒
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);  // 唤醒等待队列里的下一个线程
        return true;
    }
    return false;
}
```

逻辑并不复杂。它调用 `tryRelease()` 来释放资源。有一点需要注意的是，它是根据 `tryRelease()` 的返回值来判断该线程是否已经完成释放掉资源了！所以自定义同步器在设计 `tryRelease()` 的时候要明确这一点！

**tryRelease(int)**

> Attempts to set the state to reflect a release in exclusive mode.
>
> This method is always invoked by the thread performing release.

此方法尝试去释放指定量的资源。下面是 `tryRelease()` 的源码：

```java
protected boolean tryRelease(int arg) {
	throw new UnsupportedOperationException();
}
```

跟 `tryAcquire()` 一样，这个方法是需要独占模式的自定义同步器去实现的。正常来说，`tryRelease()` 都会成功的，因为这是独占模式，该线程来释放资源，那么它肯定已经拿到独占资源了，直接减掉相应量的资源即可，也不需要考虑线程安全的问题。但要注意它的返回值，上面已经提到了，`release()` 是根据 `tryRelease()` 的返回值来判断该线程是否已经完成释放掉资源了！所以自义定同步器在实现时，如果已经彻底释放资源（state=0），要返回true，否则返回false。

**unparkSuccessor(Node)**

```java
private void unparkSuccessor(Node node) {
    // 这里，node一般为当前线程所在的结点。
    int ws = node.waitStatus;
    if (ws < 0)	// 置零当前线程所在的结点状态，允许失败。
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;	// 找到下一个需要唤醒的结点s
    if (s == null || s.waitStatus > 0) {	// 如果为空或已取消
        s = null;
        // 循环是从后向前搜寻，因为 CANCELLED 结点中，next指针被破坏了。
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)	// 从这里可以看出，<=0的结点，都是还有效的结点。
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);	// 唤醒
}
```

**小结**

`release()` 方法的目的就是在 Node 队列中找出来第一个应该被释放的结点。但是这个节点不一定是 `head.next`，但是由于在 `cancelAcquire()` 中破坏了 next 链，所以采用从后向前的方式遍历。



### 非公平锁

#### lock()

**lock()**

```java
final void lock() {
    // compareAndSetState(int expect, int update)
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```
**setExclusiveOwnerThread()**

```java
private transient Thread exclusiveOwnerThread;
protected final void setExclusiveOwnerThread(Thread thread) {
    exclusiveOwnerThread = thread;
}
```

非公平锁就是上来就抢，如果抢到了就将独占锁设置为当前线程。如果抢不到就去 `acquire()`。

**tryAcquire()**

```java
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
```

**nonfairTryAcquire()**


```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {	// 可能别的线程释放资源，所以如果为0，就去抢资源，抢到返回 true
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 如果当前线程就是持有锁的线程，就直接加上去
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) 		// overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

#### unlock()

**unlock()**

```java
public void unlock() {
    sync.release(1);
}
```

**release()**

```java
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```

**tryRelease()**

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;	// 获取当前资源
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {	// c=0 表示资源都释放了
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```



### 公平锁

#### lock()

**lock()**

````java
final void lock() {
    acquire(1);
}
````

**tryAcquire()**

```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

公平锁个非公平锁的区别就是第5行多了一个条件`!hasQueuedPredecessors()`。

**hasQueuedPredecessors()**

```java
public final boolean hasQueuedPredecessors() {
    // The correctness of this depends on head being initialized
    // before tail and on head.next being accurate if the current
    // thread is first in queue.
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    return h != t && ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

这个方法主要就是判断当前时候是否有在 Node 队列中排队的线程，当满足如下条件时无排队线程：

- 头指针的指向等于尾指针
- 头指针的后继为空或头指针的后继不空但不是当前线程



## Condition

### await()

**await()**

````java
public final void await() throws InterruptedException {
    // Object::wait() 和 await() 都需要抛出 InterruptedException
    if (Thread.interrupted())
        throw new InterruptedException();
    // 将结点加入到 Condition 队列的末尾
    // 每一个Condition都有一个Node队列
    Node node = addConditionWaiter();
    // 这句表示 await() 操作会导致释放资源。返回释放的资源数
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    // 如果不在 Sync 队列上，就要 park 它。
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        // 这里退出有两种情况：unpark 和 interrupt 。
        // 如果是 unpark 时，说明是 signal 了，Node 会被转移到 Sync 队列上，可以退出循环
        // 如果是 interrupt，进入 checkInterruptWhileWaiting()
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }

    // signal：interruptMode 为 0
    // interrupt 在 signal 之前：interruptMode 为 THROW_IT
    // interrupt 在 signal 之后：interruptMode 为 REINTERRUPT
    
    // 从acquireQueued出来一定是获得资源的
    // 如果在获得资源的时候被中断过，修改interruptMode为REINTERRUPT
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    // 如果在等待的过程中被撤销了，取消 CANCELLED 结点
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
````

**addConditionWaiter()**

```java
private Node addConditionWaiter() {
    Node t = lastWaiter;
    // 能够进入Condition队列的只有CANCELLED和CONDITION，所以这里是清除 CANCELLED 结点
    if (t != null && t.waitStatus != Node.CONDITION) {
        unlinkCancelledWaiters();
        t = lastWaiter;
    }
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    lastWaiter = node;
    return node;
}
```

**unlinkCancelledWaiters()**

```java
private void unlinkCancelledWaiters() {
    Node t = firstWaiter;
    Node trail = null;
    // 整个的流程就是从前到后把结点过滤一遍，如果是CANCELLED状态结点就从Condition队列移除
    while (t != null) {
        Node next = t.nextWaiter;
        if (t.waitStatus != Node.CONDITION) {
            t.nextWaiter = null;
            if (trail == null)
                firstWaiter = next;
            else
                trail.nextWaiter = next;
            if (next == null)
                lastWaiter = trail;
        }
        else
            trail = t;
        t = next;
    }
}
```

**fullyRelease()**

```java
final int fullyRelease(Node node) {
    boolean failed = true;
    try {
        int savedState = getState();
        // 这个release就是我们之前介绍过EXCLUSIVE模式下的释放资源的方法，其可能产生祖苏
        if (release(savedState)) {
            failed = false;
            return savedState;
        } else {
            throw new IllegalMonitorStateException();
        }
    } finally {
        // 释放资源失败，就把当前结点标注为CANCELLED结点
        if (failed)
            node.waitStatus = Node.CANCELLED;
    }
}
```

**checkInterruptWhileWaiting()**

> Checks for interrupt, returning THROW_IE if interrupted before signalled, REINTERRUPT if after signalled, or 0 if not interrupted.

```java
/** Mode meaning to reinterrupt on exit from wait */
private static final int REINTERRUPT =  1;
/** Mode meaning to throw InterruptedException on exit from wait */
private static final int THROW_IE    = -1;

private int checkInterruptWhileWaiting(Node node) {
    return Thread.interrupted() ?
        (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) : 0;
}
```

**transferAfterCancelledWait()**

```java
final boolean transferAfterCancelledWait(Node node) {
    if (compareAndSetWaitStatus(node, Node.CONDITION, 0)) {
        enq(node);
        return true;
    }
    while (!isOnSyncQueue(node))
        Thread.yield();
    return false;
}
```

当这个方法返回 true 时，返回 THROW_IE，此时有两种情况，一是没有线程和它争抢，也就是没有 signal。二是有 signal，但是当前线程抢夺成功。两种情况都表示 interrupt 在 signal 之前。

**reportInterruptAfterWait()**

```java
private void reportInterruptAfterWait(int interruptMode)
    throws InterruptedException {
    if (interruptMode == THROW_IE)
        throw new InterruptedException();
    else if (interruptMode == REINTERRUPT)
        selfInterrupt();
}
```

如果 interrupt 在 signal 之前就抛出 InterruptedException，如果 interrupt 在 signal 之后，补上一个中断状态。



### signal()

**signal()**

```java
public final void signal() {
    // 当前线程自己 signal 是不允许的
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}
```

**isHeldExclusively()**

```java
protected final boolean isHeldExclusively() {
    return getExclusiveOwnerThread() == Thread.currentThread();
}
```

**doSignal()**

```java
private void doSignal(Node first) {
    do {
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
        first.nextWaiter = null;
    } while (!transferForSignal(first) && (first = firstWaiter) != null);
}
```

**transferForSignal()**

```java
final boolean transferForSignal(Node node) {
    // If cannot change waitStatus, the node has been cancelled.
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;
	// 结点加入到 Sync 队列中，返回插入后 node 的前驱，即插入前 Sync 队列 的 tail
    Node p = enq(node);
    int ws = p.waitStatus;
    // 如果返回的结点是 CANCELLED 状态，释放 node 的线程
    // 如果结点状态被改变，说明其已经被执行完成
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);
    return true;
}
```



### signalAll()

**signalAll()**

```java
public final void signalAll() {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignalAll(first);
}
```

**doSignalAll()**

```java
private void doSignalAll(Node first) {
    lastWaiter = firstWaiter = null;
    // 和doSignal的区别就是这里有一个循环
    do {
        Node next = first.nextWaiter;
        first.nextWaiter = null;
        transferForSignal(first);
        first = next;
    } while (first != null);
}
```



## Semaphore

### acquireShared(int)

此方法是共享模式下线程获取共享资源的顶层入口。它会获取指定量的资源，获取成功则直接返回，获取失败则进入等待队列，直到获取到资源为止，整个过程忽略中断。下面是acquireShared()的源码

```java
public final void acquireShared(int arg) {
	if (tryAcquireShared(arg) < 0)
		doAcquireShared(arg);
}
```

这里 `tryAcquireShared()` 依然需要自定义同步器去实现。但是AQS已经把其返回值的语义定义好了：负值代表获取失败；0代表获取成功，但没有剩余资源；正数表示获取成功，还有剩余资源，其他线程还可以去获取。所以这里 `acquireShared()` 的流程就是：

1. `tryAcquireShared()` 尝试获取资源，成功则直接返回；
2. 失败则通过`doAcquireShared()` 进入等待队列，直到获取到资源为止才返回。

**doAcquireShared(int)**

```java
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED); // 加入队列尾部
    boolean failed = true;
    try {
        boolean interrupted = false;	// 等待过程中是否被中断过的标志
        for (;;) {
            final Node p = node.predecessor();
         	// 第一个不是真正的线程结点，只是个哨兵
            if (p == head) {
         
                int r = tryAcquireShared(arg); // 尝试获取资源
                if (r >= 0) {	// 成功
                    // 将head指向自己，还有剩余资源可以再唤醒之后的线程
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)	// 如果等待过程中被打断过，此时将中断补上。
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            
            // 判断状态，寻找安全点，进入waiting状态，等着被unpark()或interrupt()
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

假如老大用完后释放了5个资源，而老二需要6个，老三需要1个，老四需要2个。老大先唤醒老二，老二一看资源不够，他是把资源让给老三呢，还是不让？答案是否定的！老二会继续park()等待其他线程释放资源，也更不会去唤醒老三和老四了。独占模式，同一时刻只有一个线程去执行，这样做未尝不可；但共享模式下，多个线程是可以同时执行的，现在因为老二的资源需求量大，而把后面量小的老三和老四也都卡住了。当然，这并不是问题，只是AQS保证严格按照入队顺序唤醒罢了（保证公平，但降低了并发）。

**setHeadAndPropagate**

```java
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; 
    setHead(node);	// head指向自己
     // 如果还有剩余量，继续唤醒下一个邻居线程
    if (propagate > 0 || h == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            doReleaseShared();
    }
}
```

此方法在 `setHead()` 的基础上多了一步，就是自己苏醒的同时，如果条件符合（比如还有剩余资源），还会去唤醒后继结点，毕竟是共享模式！`doReleaseShared()` 我们留着下一小节的 `releaseShared()` 里来讲。

**小结**

至此，`acquireShared()` 也要告一段落了。让我们再梳理一下它的流程：

- `tryAcquireShared()` 尝试获取资源，成功则直接返回；
- 失败则通过 `doAcquireShared()` 进入等待队列 `park()`，直到被 `unpark()/interrupt()`并成功获取到资源才返回。整个等待过程也是忽略中断的。

其实跟 `acquire()` 的流程大同小异，只不过多了个自己拿到资源后，还会去唤醒后继队友的操作。



### releaseShared(int)

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {	// 尝试释放资源
        doReleaseShared();			// 唤醒后继结点
        return true;
    }
    return false;
}
```

此方法的流程也比较简单，一句话：释放掉资源后，唤醒后继。跟独占模式下的 `release()` 相似，但有一点稍微需要注意：独占模式下的 `tryRelease()` 在完全释放掉资源（state=0）后，才会返回 true 去唤醒其他线程，这主要是基于独占下可重入的考量；而共享模式下的 `releaseShared()` 则没有这种要求，共享模式实质就是控制一定量的线程并发执行，那么拥有资源的线程在释放掉部分资源时就可以唤醒后继等待结点。例如，资源总量是13，A（5）和B（7）分别获取到资源并发运行，C（4）来时只剩1个资源就需要等待。A在运行过程中释放掉2个资源量，然后 `tryReleaseShared(2)` 返回true唤醒C，C一看只有3个仍不够继续等待；随后B又释放2个，`tryReleaseShared(2)` 返回true唤醒C，C一看有5个够自己用了，然后C就可以跟A和B一起运行。而ReentrantReadWriteLock 读锁的 `tryReleaseShared()` 只有在完全释放掉资源（state=0）才返回true，所以自定义同步器可以根据需要决定 `tryReleaseShared()` 的返回值。

**doReleaseShared()**

```java
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;
                unparkSuccessor(h); // 唤醒后继
            }
            else if (ws == 0 && !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;
        }
        if (h == head)	// 到达这里是肯定要经过第九行或者 Node 队列里没有结点了，此时需要退出。
            break;
    }
}
```

**unparkSuccessor**

```java
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```



### 非公平实现

**tryAcquire()**

```java
protected int tryAcquireShared(int acquires) {
    return nonfairTryAcquireShared(acquires);
}
```

```java
final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

上来就用 CAS 抢，抢得到就是我的。

**tryReleaseShared()**

```java
protected final boolean tryReleaseShared(int releases) {
    for (;;) {
        int current = getState();
        int next = current + releases;
        if (next < current) // overflow
            throw new Error("Maximum permit count exceeded");
        if (compareAndSetState(current, next))
            return true;
    }
```

CAS 释放资源。



### 公平实现

```java
protected int tryAcquireShared(int acquires) {
    for (;;) {
        if (hasQueuedPredecessors())
            return -1;
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

唯一的区别就是如果还有正在排队的结点就返回 -1，表示失败。



## AQS细读

> Provides a framework for implementing blocking locks and related synchronizers (semaphores, events, etc) that rely on first-in-first-out (FIFO) wait queues. This class is designed to be a useful basis for most kinds of synchronizers that rely on a single atomic int value to represent state. Subclasses must define the protected methods that change this state, and which define what that state means in terms of this object being acquired or released. Given these, the other methods in this class carry out all queuing and blocking mechanics. Subclasses can maintain other state fields, but only the atomically updated int value manipulated using methods `getState`, `setState` and `compareAndSetState` is tracked with respect to synchronization.

AQS提供了一个模板来帮助我们实现同步器。它使用的的主要思想是一个FIFO队列和一个int类型的状态值。子类继承之后需要重写某些方法来按自己的需求改变状态。状态的改变使用`compareAndSetState` ，`getState`和`setState` 三个方法。

> Subclasses should be defined as non-public internal helper classes that are used to implement the synchronization properties of their enclosing class. Class AbstractQueuedSynchronizer does not implement any synchronization interface. Instead it defines methods such as `acquireInterruptibly` that can be invoked as appropriate by concrete locks and related synchronizers to implement their public methods.

子类应该定义非公共内部类来继承AQS。AQS自己定义了同步器方法而不是实现了相应的接口。

> This class supports either or both a default exclusive mode and a shared mode. When acquired in exclusive mode, attempted acquires by other threads cannot succeed. Shared mode acquires by multiple threads may (but need not) succeed. This class does not "understand" these differences except in the mechanical sense that when a shared mode acquire succeeds, the next waiting thread (if one exists) must also determine whether it can acquire as well. Threads waiting in the different modes share the same FIFO queue. Usually, implementation subclasses support only one of these modes, but both can come into play for example in a `ReadWriteLock`. Subclasses that support only exclusive or only shared modes need not define the methods supporting the unused mode.

AQS提供两种同步模式，独占模式和共享模式。这就类似读者写者模式的读写操作，读操作是可以共享的，写操作不能共享。一般一个AQS实现了一个模式，但是`ReadWriteLock`实现了两种模式。两种模式各自有需要被覆盖的方法，需要实现哪种模式覆盖哪种模式的方法即可。在获取不到资源的时候，两种模式都等在同一个FIFO队列上。

> This class defines a nested `AbstractQueuedSynchronizer.ConditionObject` class that can be used as a `Condition` implementation by subclasses supporting exclusive mode for which method `isHeldExclusively` reports whether synchronization is exclusively held with respect to the current thread, method release invoked with the current `getState` value fully releases this object, and acquire, given this saved state value, eventually restores this object to its previous acquired state. No AbstractQueuedSynchronizer method otherwise creates such a condition, so if this constraint cannot be met, do not use it. The of `AbstractQueuedSynchronizer.ConditionObject` depends of course on the semantics of its synchronizer implementation.

`AbstractQueuedSynchronizer` 内定义了 `Condition` 接口的实现类 `ConditionObject` 。条件对象是使用在独占模式下的，而且需要实现 `isHeldExclusively` 方法，因为在 signal 的时候是不能 signal 自己的，需要用此方法做判断。

> This class provides inspection, instrumentation, and monitoring methods for the internal queue, as well as similar methods for condition objects. These can be exported as desired into classes using an AbstractQueuedSynchronizer for their synchronization mechanics.

对于内部的同步队列，这个类提供了很多方法以至于可以将他们用到同步器中。

> Serialization of this class stores only the underlying atomic integer maintaining state, so deserialized objects have empty thread queues. Typical subclasses requiring serializability will define a `readObject` method that restores this to a known initial state upon deserialization.

序列化此类的时候只能序列化 state，对于 Node 队列需要重新定义 `readObject()` 。

**Usage**

> To use this class as the basis of a synchronizer, redefine the following methods, as applicable, by inspecting and/or modifying the synchronization state using `getState()`, `setState()` and/or `compareAndSetState()`:
>
> - `tryAcquire()`
> - `tryRelease()`
> - `tryAcquireShared()`
> - `tryReleaseShared()`
> - `isHeldExclusively()`
>
> Each of these methods by default throws UnsupportedOperationException. Implementations of these methods must be internally thread-safe, and should in general be short and not block. Defining these methods is the only supported means of using this class. All other methods are declared final because they cannot be independently varied.

实现同步器需要实现上面相应的方法。这些方法的实现应该是线程安全且非阻塞的。

> You may also find the inherited methods from `AbstractOwnableSynchronizer` useful to keep track of the thread owning an exclusive synchronizer. You are encouraged to use them -- this enables monitoring and diagnostic tools to assist users in determining which threads hold locks.
>Even though this class is based on an internal FIFO queue, it does not automatically enforce FIFO acquisition policies. The core of exclusive synchronization takes the form:
> 
> Acquire:
> 
> ```java
> while (!tryAcquire(arg)) {
>    enqueue thread if it is not already queued;
>     possibly block current thread;
>}
> ```
> 
> Release:
> 
> ```java
> if (tryRelease(arg))
>     unblock the first queued thread;
>```
> 
> (Shared mode is similar but may involve cascading signals.)

AQS 继承了 `AbstractOwnableSynchronizer`（AOS），在实现同步器的时候被鼓励实现 AOS 的方法，这有助于分析工具检测代码的运行。

```java
private transient Thread exclusiveOwnerThread;

protected final void setExclusiveOwnerThread(Thread thread) {
    exclusiveOwnerThread = thread;
}

protected final Thread getExclusiveOwnerThread() {
    return exclusiveOwnerThread;
}
```

> Because checks in acquire are invoked before enqueuing, a newly acquiring thread may barge ahead of others that are blocked and queued. However, you can, if desired, define tryAcquire and/or tryAcquireShared to disable barging by internally invoking one or more of the inspection methods, thereby providing a fair FIFO acquisition order. In particular, most fair synchronizers can define tryAcquire to return false if hasQueuedPredecessors (a method specifically designed to be used by fair synchronizers) returns true. Other variations are possible.

`acquire()` 方法中在进队之前会进行权限检查，所以一个新的线程可能比正在阻塞或者正在排队的线程先获得资源。如果需要防止这种事情发生，需要在 `tryAcquire` 或 `tryAcquireShared` 中进行自定义，比如同步锁的 `hasQueuedPredecessors`。

> Throughput and scalability are generally highest for the default barging (also known as greedy, renouncement, and convoy-avoidance) strategy. While this is not guaranteed to be fair or starvation-free, earlier queued threads are allowed to recontend before later queued threads, and each recontention has an unbiased chance to succeed against incoming threads. Also, while acquires do not "spin" in the usual sense, they may perform multiple invocations of tryAcquire interspersed with other computations before blocking. This gives most of the benefits of spins when exclusive synchronization is only briefly held, without most of the liabilities when it isn't. If so desired, you can augment this by preceding calls to acquire methods with "fast-path" checks, possibly prechecking hasContended and/or hasQueuedThreads to only do so if the synchronizer is likely not to be contended.

默认情况下的并发性能是最好的，不过不保证公平。早期的线程在重新竞争锁的时候没有偏向。竞争资源时虽然没有显示的自旋，但是 tryAcquire 会被执行多次在阻塞之前，这具有自旋的特点。如果你想增强 acquire 使其具有快速检查的功能，可以调用 `hasContended `或`hasQueuedThreads `去查看是否目前有竞争。

> This class provides an efficient and scalable basis for synchronization in part by specializing its range of use to synchronizers that can rely on int state, acquire, and release parameters, and an internal FIFO wait queue. When this does not suffice, you can build synchronizers from a lower level using atomic classes, your own custom java.util.Queue classes, and LockSupport blocking support.

**Usage Examples**

> Here is a non-reentrant mutual exclusion lock class that uses the value zero to represent the unlocked state, and one to represent the locked state. While a non-reentrant lock does not strictly require recording of the current owner thread, this class does so anyway to make usage easier to monitor. It also supports conditions and exposes one of the instrumentation methods:
>
> ```java
> class Mutex implements Lock, java.io.Serializable {
> 
>     // Our internal helper class
>     private static class Sync extends AbstractQueuedSynchronizer {
>        // Reports whether in locked state
>         protected boolean isHeldExclusively() {
>            return getState() == 1;
>         }
>         // Acquires the lock if state is zero
>         public boolean tryAcquire(int acquires) {
>             assert acquires == 1; // Otherwise unused
>             if (compareAndSetState(0, 1)) {
>                 setExclusiveOwnerThread(Thread.currentThread());
>                 return true;
>            }
>             return false;
>         }
> 
>        // Releases the lock by setting state to zero
>         protected boolean tryRelease(int releases) {
>             assert releases == 1; // Otherwise unused
>             if (getState() == 0) throw new IllegalMonitorStateException();
>             setExclusiveOwnerThread(null);
>             setState(0);
>             return true;
>        }
> 
>        // Provides a Condition
>         Condition newCondition() { return new ConditionObject(); }
> 
>         // Deserializes properly
>         private void readObject(ObjectInputStream s)
>             throws IOException, ClassNotFoundException {
>             s.defaultReadObject();
>            setState(0); // reset to unlocked state
>         }
>     }
> 
>     // The sync object does all the hard work. We just forward to it.
>     private final Sync sync = new Sync();
> 
>     public void lock()                { sync.acquire(1); }
>     public boolean tryLock()          { return sync.tryAcquire(1); }
>     public void unlock()              { sync.release(1); }
>     public Condition newCondition()   { return sync.newCondition(); }
>     public boolean isLocked()         { return sync.isHeldExclusively(); }
>     public boolean hasQueuedThreads() { return sync.hasQueuedThreads(); }
>     public void lockInterruptibly() throws InterruptedException {
>         sync.acquireInterruptibly(1);
>     }
>     public boolean tryLock(long timeout, TimeUnit unit)
>         throws InterruptedException {
>         return sync.tryAcquireNanos(1, unit.toNanos(timeout));
>     }
> }
> ```
> 
> 
> Here is a latch class that is like a CountDownLatch except that it only requires a single signal to fire. Because a latch is non-exclusive, it uses the shared acquire and release methods.
> 
> ```java
> class BooleanLatch {
>     private static class Sync extends AbstractQueuedSynchronizer {
>        boolean isSignalled() { return getState() != 0; }
>         protected int tryAcquireShared(int ignore) {
>            return isSignalled() ? 1 : -1;
>         }
> 
>        protected boolean tryReleaseShared(int ignore) {
>             setState(1);
>             return true;
>         }
>     }
> 
>     private final Sync sync = new Sync();
>     public boolean isSignalled() { return sync.isSignalled(); }
>     public void signal()         { sync.releaseShared(1); }
>     public void await() throws InterruptedException {
>         sync.acquireSharedInterruptibly(1);
>     }
> }
> ```
> 

