## 基础

### synchronized对某个对象加锁

```java
public class T {
	private int count = 10;
	private Object o = new Object();
	public void m() {
		synchronized(o) { // 任何线程要执行下面的代码，必须先拿到o的锁
			count--;
			System.out.println(Thread.currentThread().getName() + " count = " + count);
		}
	}
}
```

```java
public class T {
    private int count = 10;
    public void m() {
        synchronized (this) { // 任何线程要执行下面的代码，必须先拿到this的锁
            count--;
            System.out.println(Thread.currentThread().getName() + " count = " + count);
        }
    }
}
```



### synchronized对方法加锁

```java
public class T {
	private int count = 10;
	public synchronized void m() { //等同于在方法的代码执行时要synchronized(this)
		count--;
		System.out.println(Thread.currentThread().getName() + " count = " + count);
	}
}
```

```java
public class T {
    private static int count = 10;
    public synchronized static void m() { // 这里等同于synchronized(T.class)
        count--;
        System.out.println(Thread.currentThread().getName() + " count = " + count);
    }
    public static void mm() {
        synchronized (T.class) {
            count--;
        }
    }
}
```



### volatile

```java
// volatile不能保证原子性
public class T implements Runnable {
    private /*volatile*/ int count = 10000;
    public /*synchronized*/ void run() {
        for (int i = 0; i < 100; i++)
            count--;
        System.out.println(Thread.currentThread().getName() + " count = " + count);
    }
    public static void main(String[] args) {
        T t = new T();
        for (int i = 0; i < 100; i++) {
            new Thread(t, "THREAD" + i).start();
        }
    }
}
```

```java
// 保证线程可见性
public class T01_HelloVolatile {
	volatile boolean running = true; // 对比一下有无volatile的情况下，整个程序运行结果的区别
	void m() {
		System.out.println("m start");
		while(running) { }
		System.out.println("m end!");
	}
	public static void main(String[] args) {
		T01_HelloVolatile t = new T01_HelloVolatile();
		new Thread(t::m, "t1").start();
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		t.running = false;
	}
}
```

```java
// volatile 引用类型（包括数组）只能保证引用本身的可见性，不能保证内部字段的可见性
public class T02_VolatileReference1 {
    boolean running = true;
    volatile static T02_VolatileReference1 T = new T02_VolatileReference1();
    void m() {
        System.out.println("m start");
        while(running) {
//			try {
//				TimeUnit.MILLISECONDS.sleep(10);
//			} catch (InterruptedException e) {
//				e.printStackTrace();
//			} 
//			这段代码被放开后程序可以正常执性，可能是JVM对volatile有优化，但是没有考证过具体的细节
        }
        System.out.println("m end!");
    }
    public static void main(String[] args) {
        new Thread(T::m, "t1").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        T.running = false;
    }
}
```



### synchronized特点

```java
// synchronized是可重入的，也就是说他是基于线程分配的
public class T {
    synchronized void m1() {
        System.out.println("m1 start");
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        m2();
        System.out.println("m1 end");
    }
    synchronized void m2() {
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("m2");
    }
    public static void main(String[] args) {
        new T().m1();
    }
}
```

```java
// 继承并覆盖方法也可以冲入
public class T {
	synchronized void m() {
		System.out.println("m start");
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("m end");
	}
	public static void main(String[] args) {
		new TT().m();
	}
}
class TT extends T {
	@Override
	synchronized void m() {
		System.out.println("child m start");
		super.m();
		System.out.println("child m end");
	}
}
```

```java
// 遇到异常释放锁
public class T {
	int count = 0;
	synchronized void m() {
		System.out.println(Thread.currentThread().getName() + " start");
		while(true) {
			count ++;
			System.out.println(Thread.currentThread().getName() + " count = " + count);
			try {
				TimeUnit.SECONDS.sleep(1);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			if(count == 5) {
				int i = 1/0; // 此处抛出异常，锁将被释放
				System.out.println(i);
			}
		}
	}
	public static void main(String[] args) {
		T t = new T();
		Runnable r = () -> t.m();
		new Thread(r, "t1").start();
		try {
			TimeUnit.SECONDS.sleep(3);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		new Thread(r, "t2").start();
	}
}
```

```java
// 锁定某对象o，如果o的属性发生改变，不影响锁的使用但是如果o变成另外一个对象，则锁定的对象发生改变
// 应该避免将锁定对象的引用变成另外的对象
public class SyncSameObject {
    /*final*/ Object o = new Object();
    void m() {
        synchronized (o) {
            while (true) {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName());
            }
        }
    }
    public static void main(String[] args) {
        SyncSameObject t = new SyncSameObject();
        // 启动第一个线程
        new Thread(t::m, "t1").start();
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 创建第二个线程
        Thread t2 = new Thread(t::m, "t2");
        t.o = new Object(); // 锁对象发生改变，所以t2线程得以执行，如果注释掉这句话，线程2将永远得不到执行机会
        t2.start();
    }
}
```



## 面试题

### 1

实现一个容器，提供两个方法，add，size。写两个线程，线程1添加10个元素到容器中，线程2实现监控元素的个数，当个数到5个时，线程2给出提示并结束。

**WaitNotify**

```java
public class WaitNotify {
    
    List lists = new ArrayList();
    public void add(Object o) { lists.add(o); }
    public int size() { return lists.size(); }

    static volatile boolean t2Start = false;

    public static void main(String[] args) throws InterruptedException {
        WaitNotify c = new WaitNotify();

        final Object lock = new Object();

        new Thread(() -> {
            try {
                System.out.println("t1 start");
                for (int i = 0; i < 10; i++) {
                    c.add(new Object());
                    System.out.println("add " + i);
                    synchronized (lock) {
                        if (c.size() == 5) {
                            if(t2Start)
                                lock.notify();
                            lock.wait();
                        }
                    }
                    TimeUnit.SECONDS.sleep(1);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "t1").start();

        TimeUnit.SECONDS.sleep(1);

        new Thread(() -> {
            t2Start = true;
            try {
                System.out.println("t2 start");
                while (true) {
                    synchronized (lock) {
                        if (c.size() != 5) {
                            lock.wait();
                        } else {
                            System.out.println("size == 5");
                            System.out.println("t2 end");
                            lock.notify();
                            break;
                        }
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            }

        }, "t2").start();
    }
}
```

**AwaitSignal**

```java
public class AwaitSignal {

    List lists = new ArrayList();
    public void add(Object o) { lists.add(o); }
    public int size() { return lists.size(); }

    static volatile boolean t2Start = false;

    public static void main(String[] args) throws InterruptedException {
        AwaitSignal c = new AwaitSignal();

        ReentrantLock lock = new ReentrantLock();
        Condition condition = lock.newCondition();

        new Thread(() -> {

            try {
                System.out.println("t1 start");
                for (int i = 0; i < 10; i++) {
                    c.add(new Object());
                    System.out.println("add " + i);
                    lock.lock();
                    if (c.size() == 5) {
                        if (t2Start) {
                            condition.signal();
                        }
                        condition.await();
                    }
                    lock.unlock();
                    TimeUnit.SECONDS.sleep(1);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "t1").start();

        TimeUnit.SECONDS.sleep(7);

        new Thread(() -> {
            t2Start = true;
            try {
                System.out.println("t2 start");
                while (true) {
                    lock.lock();
                    if (c.size() != 5) {
                        condition.await();
                    } else {
                        System.out.println("size == 5");
                        System.out.println("t2 end");
                        // 通知t1继续执行
                        condition.signalAll();
                        lock.unlock();
                        break;
                    }
                    lock.unlock();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }

        }, "t2").start();
    }
}
```



### 2

写一个固定容量同步容器，拥有put和get方法，能够支持2个生产者线程以及10个消费者线程的阻塞调用。

```java
public class MyContainer2<T> {
    final private LinkedList<T> lists = new LinkedList<>();
    final private int MAX = 10; // 最多10个元素
    private int count = 0;

    private Lock lock = new ReentrantLock();
    private Condition producer = lock.newCondition();
    private Condition consumer = lock.newCondition();

    public void put(T t) {
        try {
            lock.lock();
			/**
			 * 如果这个地方使用if不使用while，考虑一种情况
			 * t1和t2两个线程在此等待，突然一个signal将他俩都唤醒了
			 * t1抢到了资源，所以继续运行，又把队列加满了，然后释放了资源
			 * t2终于获得了资源，但此时队列实际上满的，t2不应该去再往队列里加一个
			 * 所以解决方案就是t2判断队列是不是满的，他发现是满的，就继续await
			 */
			while (lists.size() == MAX) {
                producer.await();	// 一定要注意，这个方法会释放临界资源
            }

            lists.add(t);
            ++count;
            consumer.signalAll(); // 通知消费者线程进行消费
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public T get() {
        T t = null;
        try {
            lock.lock();
            while (lists.size() == 0) {
                consumer.await();
            }
            t = lists.removeFirst();
            count--;
            producer.signalAll(); // 通知生产者进行生产
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
        return t;
    }
    
    public static void main(String[] args) {
        MyContainer2<String> c = new MyContainer2<>();
        // 启动消费者线程
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                for (int j = 0; j < 5; j++)
                	System.out.println(c.get());
            }, "c" + i).start();
        }

        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 启动生产者线程
        for (int i = 0; i < 2; i++) {
            new Thread(() -> {
                for (int j = 0; j < 25; j++)
                	c.put(Thread.currentThread().getName() + " " + j);
            }, "p" + i).start();
        }
    }
}
```



### 3

用线程顺序打印A1B2C3....Z26。

```java
public class WaitNotify {

    private static volatile boolean t2Started = false;

    public static void main(String[] args) {

        final Object o = new Object();
        char[] aI = "1234567".toCharArray();
        char[] aC = "ABCDEFG".toCharArray();

        new Thread(() -> {
            try {
                synchronized (o) {
                    if (!t2Started) {
                        o.wait();
                    }
                    for (char c : aI) {
                        System.out.print(c);
                        o.notify();
                        o.wait();
                    }
                    o.notify();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "t1").start();

        new Thread(() -> {
            t2Started = true;
            try {
                synchronized (o) {
                    for (char c : aC) {
                        System.out.print(c);
                        o.notify();
                        o.wait();
                    }
                    o.notify();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "t2").start();
    }
}
```





















