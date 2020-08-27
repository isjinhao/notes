## JVM

JVM则是JRE中的核心组成部分，承担分析和执行Java字节码的工作。在Java历史上有很多发行的Java虚拟机，但目前一般学习使用的都是`HotSpot`。查看本机JVM：`java -version`

<div align='center'><img src="http://q0l9qvfyx.bkt.clouddn.com/f4ef67b6-c40f-4f57-a5e0-6108da3a9c85" style="width:80%;" /></div>
**运行时数据区**

Java虚拟机在执行Java程序的时候会把它所管理的内存区域划分为若干个不同的数据区域。根据JVM规范，JVM 内存共分为虚拟机栈、堆、方法区、程序计数器、本地方法栈五个部分。

**HotSpot 虚拟机**

HotSpot虚拟机把本地方法栈和虚拟机栈合并在了一起。所以只有虚拟机栈、堆、方法区、程序计数器四个部分。



## 运行时数据区划分

<div align='center'><img src="http://blogfileqiniu.isjinhao.site/ac670268-0509-4a50-ae34-06706b6bf87c" style="width:50%;" /></div>
### 线程私有

#### 程序计数器

程序计数器是一块较小的内存空间，可以看作是当前线程所执行的字节码的行号指示器。字节码解释器工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等功能都需要依赖这个计数器来完。

另外，**为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各线程之间计数器互不影响，独立存储，所以其是线程私有的。**

从上面的介绍中我们知道程序计数器主要有两个作用：

- 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。
- 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

**注意**：程序计数器是唯一一个不会出现 OutOfMemoryError 的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。

#### Java虚拟机栈

与程序计数器一样，Java虚拟机栈也是线程私有的，它的生命周期和线程相同，描述的是 Java 方法执行的内存模型，每次方法调用的数据都是通过栈传递的。

Java 内存可以粗糙的区分为堆内存（Heap）和栈内存(Stack)，其中栈就是现在说的虚拟机栈，或者说是虚拟机栈中局部变量表部分。（实际上，Java虚拟机栈是由一个个栈帧组成，而每个栈帧中都拥有：局部变量表、操作数栈、动态链接、方法出口信息）

局部变量表主要存放了编译器可知的各种数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）和returnAddress类型（在古老的虚拟机中用于异常处理）。

Java 虚拟机栈会出现两种异常：StackOverFlowError 和 OutOfMemoryError。

- **StackOverFlowError：** 若Java虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前Java虚拟机栈的最大深度的时候，就抛出StackOverFlowError异常。

- **OutOfMemoryError：** 若 Java 虚拟机栈的内存大小允许动态扩展，且当线程请求栈时内存用完了，无法再动态扩展了，此时抛出OutOfMemoryError异常。

Java 虚拟机栈也是线程私有的，每个线程都有各自的Java虚拟机栈，而且随着线程的创建而创建，随着线程的死亡而死亡。

**扩展：那么方法/函数如何调用？**

Java 栈可用类比数据结构中栈，Java 栈中保存的主要内容是栈帧，每一次函数调用都会有一个对应的栈帧被压入Java栈，每一个函数调用结束后，都会有一个栈帧被弹出。

Java方法有两种返回方式：

- return 语句。

- 抛出异常。

不管哪种返回方式都会导致栈帧被弹出。



### 线程共享

#### Java堆

Java 虚拟机所管理的内存中最大的一块，Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。

#### 方法区

方法区与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做 Non-Heap（非堆），目的应该是与 Java 堆区分开来。

HotSpot 虚拟机中方法区在JDK8之前被称为 **“永久代”**，本质上两者并不等价。仅仅是因为 HotSpot 虚拟机设计团队用永久代来实现方法区而已，这样 HotSpot 虚拟机的垃圾收集器就可以像管理 Java 堆一样管理这部分内存了。但是这并不是一个好主意，因为这样更容易遇到内存溢出问题。

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。因为虽然JVM规范中表示不强制在此区域进行垃圾回收，但是目前主流的商业虚拟机，如HotSpot都实现了方法区的垃圾回收。

JDK8 的时候，方法区被移动到了元空间，元空间使用的是直接内存。

我们可以使用参数： `-XX:MetaspaceSize` 来指定元数据区的大小。与永久区很大的不同就是，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。

##### 运行时常量池

运行时常量池是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有常量池信息（用于存放编译期生成的各种字面量和符号引用）

- 字面量就是指这个量本身，比如字面量3。也就是指3。再比如 string类型的字面量"ABC"， 这个"ABC" 通过字来描述。 可以理解成一眼就能知道的量。
- 符号引用（Symbolic References）：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能够无歧义的定位到目标即可。例如，在Class文件中它以CONSTANT_Class_info、CONSTANT_Fieldref_info、CONSTANT_Methodref_info等类型的常量出现。

既然运行时常量池时方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 OutOfMemoryError 异常。

JDK8及之后版本的 JVM 已经将运行时常量池从方法区中移了出来，在 Java 堆（Heap）中开辟了一块区域存放运行时常量池。



## 对象的创建

<div align='center'><img src="http://q0l9qvfyx.bkt.clouddn.com/b3252c77-0466-41a1-8ab7-b98cf1f4a92d" style="width:80%;" /></div>
**类加载检查**

虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程。

**分配内存**

在**类加载检查**通过后，接下来虚拟机将为新生对象**分配内存**。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从 Java 堆中划分出来。**分配方式**有 **“指针碰撞”** 和 **“空闲列表”** 两种，选择那种分配方式由 Java 堆是否规整决定，而Java堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定。

**初始化零值**

内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。

**设置对象头**

初始化零值完成之后，**虚拟机要对对象进行必要的设置**，例如这个对象是那个类的实例、如何才能找到类的元数据信息、对象的哈希吗、对象的 GC 分代年龄等信息。 **这些信息存放在对象头中。** 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。

**执行 init 方法**

在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，`<init>` 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 `<init>` 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。



### **内存分配的两种方式**

**指针碰撞**

假设Java堆中的内存时绝对规整的，所有用过的内存都放在一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲空间那边挪动一段与对象大小相等的距离，这种分配方式成为“指针碰撞”。

**空闲链表**

如果JAVA堆中的内存并不是规整的，已使用的内存和空闲的内存相互交错，那就没有办法简单地进行指针碰撞了，虚拟机就必须维护一个列表，记录上哪些内存块是可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录，这种分配方式成为“空闲列表”。



### 内存分配并发问题

在创建对象的时候有一个很重要的问题，就是线程安全，因为在实际开发过程中，创建对象是很频繁的事情，作为虚拟机来说，必须要保证线程是安全的，否则比如当虚拟机正在给对象A分配内存，指针还没来得及修改，对象B又同时使用了原来的指针来分配内存就会引发严重的问题。通常来讲，虚拟机采用两种方式来保证线程安全：

**CAS+失败重试**

CAS 是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。

**TLAB**

为每一个线程预先在Eden区分配一块儿内存，称为本地线程分配缓存（Thread Local Allocation Buffer，TLAB），JVM在给线程中的对象分配内存时，首先在TLAB分配，当对象大于TLAB中的剩余内存或TLAB的内存已用尽时，再采用上述的CAS进行内存分配。



## 对象的访问定位

建立对象就是为了使用对象，我们的Java程序通过栈上的 reference 数据来操作堆上的具体对象。对象的访问方式有虚拟机实现而定，目前主流的访问方式有**使用句柄**和**直接指针**两种（HotSpot虚拟机使用直接指针方式）：

- **句柄：** 如果使用句柄的话，那么Java堆中将会划分出一块内存来作为句柄池，reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息；

<div align='center'><img src="http://q0l9qvfyx.bkt.clouddn.com/76b1fdaa-1749-4064-85da-d7eff45862d2" style="width:80%;" /></div>
- **直接指针**：如果使用直接指针访问，那么 Java 堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而reference 中存储的直接就是对象的地址。

<div align='center'><img src="http://q0l9qvfyx.bkt.clouddn.com/e8566da3-5d49-48a2-8d34-747d1677b7be" style="width:80%;" /></div>
这两种对象访问方式各有优势。使用句柄来访问的最大好处是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改。使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销。



## 错误演示

### OutOfMemoryError

> Thrown when the Java Virtual Machine cannot allocate an object because it is out of memory, and no more memory could be made available by the garbage collector. OutOfMemoryError objects may be constructed by the virtual machine as if suppression were disabled and/or the stack trace was not writable.

```java
// VMOptions: -Xms5m -Xmx5m -XX:+HeapDumpOnOutOfMemoryError
// 设置最小堆空间、设置最大堆空间、发生OutOfMemoryError时Dump堆
public static void main(String[] args) {
    List<MyTest1> list = new ArrayList<>();
    while(true)
        list.add(new MyTest1());
}
/*
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid21184.hprof ...
Heap dump file created [9351863 bytes in 0.098 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3210)
	at java.util.Arrays.copyOf(Arrays.java:3181)
	at java.util.ArrayList.grow(ArrayList.java:265)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:239)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:231)
	at java.util.ArrayList.add(ArrayList.java:462)
	at two.jvm.runtimememory.MyTest1.main(MyTest1.java:11)
*/
```



### 线程栈溢出

```java
// 设置线程栈的大小：-Xss160k
public class MyTest2 {
    static int length = 0;
    public static void main(String[] args) {
        try {
            test();
        } catch (Throwable t) {
            t.printStackTrace();
        }
    }

    static void test() {
        length++;
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(length);
        test();
    }
}
```

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/3563316d-3f94-4c1e-9fe2-9f3a4dabf5dd" /></div>
### 元空间溢出

```java
// CGLIB测试
public class MyTest3 {

    // -XX:MaxMetaspaceSize=10m
    public static void main(String[] args) {
        while (true) {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(MyTest3.class);
            enhancer.setUseCache(false);
            enhancer.setCallback((MethodInterceptor) (obj, method, args1, proxy) -> proxy.invoke(obj, args1));
            System.out.println("Hello World");
            enhancer.create();
        }
    }
}
```

<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/fba7083a-058c-4ef6-abb1-e968512a542c" /></div>

## JDK监控工具

### 命令行工具

#### jps

```
jps [options] [hostid]
```

 如果不指定hostid就默认为当前主机或服务器。命令行参数选项说明如下： 

- `-q`：不输出类名、Jar名和传入main方法的参数

- `-l`：·输出main类或Jar的全限名
- `-m`：输出传入main方法的参数

- `-v`：输出传入JVM的参数

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/4de6d879-2198-486d-8782-bf1f339600a4" /></div>
#### jstat

```
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
```

选项option可以由以下值构成：

- class：显示 ClassLoader的相关信息。
- compiler：显示JT编译的相关信息。
- gc：显示与GC相关的堆信息。
- gccapacity：显示各个代的容量及使用情况。

- gccause：显示垃圾收集相关信息（同gcutil），同时显示最后一次或当前正在发生的垃圾收集的诱发原因。

- gcnew：显示新生代信息。

- gcnewcapacity：显示新生代大小与使用情况。
- gcold：显示老年代和永久代的信息。
- gcoldcapacity：显示老年代的大小。
- gcmetacapacity：显示元空间的大小。

- gcutil：显示垃圾收集信息。
- printcompilation：输出JIT编译的方法信息。

`-t`参数可以在输出信息前加上一个 Timestamp列，显示程序的运行时间。

`-h`参数可以在周期性数据输出时，输出多少行数据后，跟着输出一个表头信息。

`interval`参数用于指定输出统计数据的周期，单位为毫秒。

`count`用于指定一共输出多少次数据

**class**

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/81fea933-ede0-4782-9218-52dd5a5d86e6" /></div>
在class的输出中，Loaded表示载入了类的数量，Bytes表示载入类的合计大小，Unloaded表示卸载类的数量，第2个Bytes表示卸载类的大小，Time表示在加载和卸载类上所花的时间。

**compiler**

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/31b011c9-8c29-4239-80c2-4590914d7337" /></div>
Compiled表示编译任务执行的次数，Failed表示编译失败的次数，Invalid表示编译不可用的次数，Time表示编译的总耗时，FailedType表示最后一次编译失败的类型，FailedMethod表示最后一次编译失败的类名和方法名。

**gcnewcapacity**

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/da450e02-ff07-4fab-8405-52105ee4417c" /></div>
- NGCMN：新生代最小值
- NGCMX：新生代最大值
- NGC：新生代当前大小
- S0MAX：S0区最大容量
- S0C：S0区当前容量
- S1CMX：S1区最大容量
- S1C：S1区当前容量
- ECMX：Eden区最大值
- EC：Eden区当前容量
- YGC：新生代GC的次数
- FGC：Full GC次数

**gcoldcapacity**

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/da263c7b-5303-4169-b397-7e408e4fe1da" /></div>
- OGCMN：老年代最小容量
- OGCMX：老年代最大容量
- OGC：老年代当前容量
- OC：老年代当前容量
- FGCT：Full GC总耗时
- GCT：GC总耗时

**gcmetacapcaity**

<div align="center"><img width="90%" src="http://blogfileqiniu.isjinhao.site/cfd51f1c-83b1-4b30-9147-b413e348567c" /></div>
- MCMN：最小元数据容量
- MCMX：最大元数据容量
- MC：当前元数据空间大小
- CCSMN：最小压缩类空间大小
- CCSMX：最大压缩类空间大小
- CCSC：当前压缩类空间大小

**gcutil**

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/2a556c4c-5998-4998-b6cb-d6cc875ed815" /></div>
- S0：S0已使用的占当前容量百分比
- S1：S1已使用的占当前容量百分比
- E：Eden区已使用的占当前容量百分比
- O：老年代已使用的占当前容量百分比
- M：元空间已使用的占当前容量百分比
- CCS：压缩类使用的空间

更详细参考：https://www.jianshu.com/p/213710fb9e40

#### jinfo

```
jinfo <option> <pid>
```

选项option可以由以下值构成：

- `-flag<name>`：打印指定的参数值
- `-flag [+|-]<name>`：设置布尔类型的参数值
- `-flag <name>=<value>`：设置虚拟机参数值

修改只对少数虚拟机参数有效。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/cdb6d8dd-b4de-43c4-9769-7df97f7039bc" /></div>
#### jmap

jmap命令用于生产堆转存快照。打印出某个java进程（使用pid）内存内的，所有对象的情况（如：产生那些对象，及其数量）。

```
jmap [option] pid
jmap [option] executable core
jmap [option] [server-id@]remote-hostname-or-IP
```

选项option可以由以下值构成：

- `-dump:[live,]format=b,file=<filename>`：使用hprof二进制形式，输出jvm的heap内容到文件。live子选项是可选的，假如指定live选项，那么只输出活的对象到文件。

- `-histo[:live]`：打印每个Class的实例数目，内存占用，类全名信息。如果live子参数加上后，只统计活的对象数量。

- `-clstats` ：打印类加载器的统计信息

**dump**

<div align="center"><img width="70%" src="http://blogfileqiniu.isjinhao.site/eb4df7a5-ccaf-47ba-a0fe-fad038a62bb5" /></div>
**histo**

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/0afb2fbb-e583-49af-be87-43fc4777c383" /></div>
**clstats**

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/aa2e9d60-318e-4c2a-9316-e09b8d73308a" /></div>
#### jstack

```
jstack [option] pid
jstack [option] executable core
jstack [option] [server-id@]remote-hostname-or-IP
```

- `-l`：长列表。打印关于锁的附加信息。
- `-F`：当`jstack [-l] pid`没有相应的时候强制打印栈信息。
- `-m`：打印java和native c/c++框架的所有栈信息。

### 可视化工具

**jvisualvm**

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/f783a362-39df-47d4-b4a6-5c41dc3b969e" /></div>
**jconsole**

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/0b9a9d73-eee4-4aca-bdc8-1e1e8d2cbe17" /></div>
更详细：https://juejin.im/post/59e6c1f26fb9a0451c397a8c#heading-9

