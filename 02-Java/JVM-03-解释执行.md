## 运行时的栈帧结构

栈帧（Stack Frame）是用于支持虚拟机进行方法调用和方法执行的数据结构，它是虚拟机运行时数据区中的虚拟机栈（Virtual Machine Stack）的栈元素。栈帧存储了方法的局部变量表、操作数栈、动态连接和方法返回地址等信息。每一个方法从调用开始至执行完成的过程，都对应着一个栈帧在虚拟机栈里面从入栈到出栈的过程。也就是说虚拟机每调用一个方法都会在当前调用栈上压入一个栈帧，调用完了删除调用栈栈顶的栈帧。

每一个栈帧都包括了局部变量表、操作数栈、动态连接、方法返回地址和一些额外的附加信息。在编译程序代码的时候，栈帧中需要多大的局部变量表，多深的操作数栈都已经完全确定了，并且写入到方法表的Code属性之中，因此一个栈帧需要分配多少内存，不会受到程序运行期变量数据的影响，而仅仅取决于具体的虚拟机实现。一个线程中的方法调用链可能会很长，很多方法都同时处于执行状态。对于执行引擎来说，在活动线程中，只有位于栈顶的栈帧才是有效的，称为当前栈帧（Current Stack Frame），与这个栈帧相关联的方法称为当前方法（Current Method）。执行引擎运行的所有字节码指令都只针对当前栈帧进行操作，在概念模型上，典型的栈帧结构如图。

<div align="center"><img width="50%" src="http://q0l9qvfyx.bkt.clouddn.com/7db50cc5-8d27-4f47-b789-bfdb44c82a5d" /></div>
### 局部变量表

局部变量表（Local Variable Table）是一组变量值存储空间，用于存放**方法参数**和**方法内部定义的局部变量**。在Java程序编译为 Class 文件时，就在方法的Code属性的 max locals 数据项中确定了该方法所需要分配的局部变量表的最大容量。局部变量表的容量以变量槽（Variable Slot， 下称Slot）为最小单位，同时每个 Slot 都应该能存放一个boolean、byte、char、 short、int、float、 reference 或 returnAddress 类型的数据（这8种数据类型，都可以使用32位或更小的物理内存来存放），但它不是指每个Slot占用32位长度的内存空间，实际上Slot的长度可以随着处理器、操作系统或虚拟机的不同而发生变化。比如64位虚拟机中完全可以用64位的物理空间去实现一个Slot，只要在后期使用对齐和补白的手段让Slot在外观上看起来与32位虚拟机中的一致就可以了。

一个Slot可以存放一个32位以内的数据类型，Java中占用32位以内的数据类型有 boolean、byte、char、short、int、float、reference 和 returnAddress 8种类型。对于64位的数据类型，虚拟机会以高位对其的方式为其分配两个连续的Slot空间。Java语言中明确的（reference类型则可能是32位也可能是64位）64位的数据类型只有long 和 double两种。

其中 reference 类型表示对一个对象实例的引用，它具有两个功能。一是从此引用中直接或间接地查找到对象在Java堆中的数据存放的起始地址索引，二是此引用中直接或间接地查找到对象所属数据类型在方法区中的存储的类型信息。returnAddress 类型目前已经很少见了，它是为很古老的Java虚拟机实现异常处理服务的。

虚拟机通过索引定位的方式使用局部变量表，索引值的范围是从0开始至局部变量表最大的Slot数量如果访问的是32位数据类型的变量,索引n就代表了使用第n个Slot 如果是64位数据类型的变量，则说明会同时使用n和n+1两个Slot。对于两个相邻的共同存放一个64位数据的两个Slot，不允许采用任何方式单独访问其中的某一个，Java虚拟机规范中明确要求了如果遇到进行这种操作的字节码序列，虚拟机应该在类加载的校验阶段出异常。

#### 复用

在方法执行时，虚拟机是使用局部变量表完成参数值到参数变量列表的传递过程的，如果执行的是实例方法（非static的方法），那局部变量表中第0位索引的Slot默认是用于传递方法所属对象实例的引用，在方法中可以通过关键字“this”来访问到这个隐含的参数。其余参数则按照参数表顺序排列，占用从1开始的局部变量Slot，参数表分配完毕后，再根据方法体内部定义的变量顺序和作用域分配其余的Slot。为了尽可能节省栈帧空间，局部变量表中的Slot是可以重用的，方法体中定义的变量，其作用域并不一定会覆盖整个方法体，如果当前字节码PC计数器的值已经超出了某个变量的作用域，那这个变量对应的Slot就可以交给其他变量使用。不过，这样的设计除了节省栈帧空间以外，还会伴随一些额外的副作用，例如，在某些情况下，Slot的复用会直接影响到系统的垃圾收集行为。

```java
public static void main(String[] args) {
    {
        byte[] placeHolder = new byte[64 * 1024 *1024];
    }
    System.gc();
}
/*
[GC (System.gc())  68403K->66336K(110080K), 0.0012697 secs]
[Full GC (System.gc())  66336K->66160K(110080K), 0.0054959 secs]
*/
```

```java
public class MyTest1 {
    public static void main(String[] args) {
        {
            byte[] placeHolder = new byte[64 * 1024 *1024];
        }
        int a = 0;
        System.gc();
    }
}
/*
[GC (System.gc())  68403K->66336K(110080K), 0.0010283 secs]
[Full GC (System.gc())  66336K->624K(110080K), 0.0059838 secs]
*/
```

placeHolder能否被回收的根本原因是：局部变量表中的Slot是否还存有关于 placeholder 数组对象的引用。第一次修改中，代码虽然已经离开了 placeholder 的作用域，但在此之后，没有任何对局部变量表的读写操作，placeholder 原本所占用的Slot还没有被其他变量所复用，所以作为 GC Roots 一部分的局部变量表仍然保持着对它的关联。这种关联没有被及时打断，在绝大部分情况下影响都很轻微。但如果遇到一个方法，其后面的代码有一些耗时很长的操作，而前面又定义了占用了大量内存、实际上已经不会再使用的变量，手动将其设置为null值（用来代替那句int a=0，把变量对应的局部变量表Slot清空）便不见得是一个绝对无意义的操作，这种操作可以作为一种在极特殊情形（对象占用内存大、此方法的栈帧长时间不能被回收、方法调用次数达不到JIT的编译条件）下的“奇技”来使用。

#### 局部变量需要初始化

关于局部变量表，还有一点可能会对实际开发产生影响，就是局部变量不像前面介绍的类变量那样存在“准备阶段”。类变量有两次赋初始值的过程，一次在准备阶段，赋予系统初始值；另外一次在初始化阶段，赋予程序员定义的初始值。因此，即使在初始化阶段程序员没有为类变量赋值也没有关系，类变量仍然具有一个确定的初始值。但局部变量就不一样如果一个局部变量定义了但没有赋初始值是不能使用的，而且这个错误会在被编译期间就被发现。

<div align="center"><img width="50%" src="http://q0l9qvfyx.bkt.clouddn.com/b8fcb7e8-5a04-45ce-9a94-51a09066614b" /></div>
### 操作数栈

同局部变量表一样，操作数栈的最大深度也在编译的时候写入到Code属性的 max_stacks 数据项中。操作数栈的每一个元素可以是任意的Java数据类型，包括 long 和 double。32位数据类型所占的栈容量为1，64位数据类型所占的栈容量为2。在方法执行的任何时候，操作数栈的深度都不会超过在 max stacks 数据项中设定的最大值。当一个方法刚刚开始执行的时候，这个方法的操作数栈是空的，在方法的执行过程中，会有各种字节码指令往操作数栈中写入和提取内容，也就是出栈/入栈操作。例如，在做算术运算的时候是通过操作数栈来进行的，又或者在调用其他方法的时候是通过操作数栈来进行参数传递的。

举个例子，整数加法的字节码指令iadd在运行的时候操作数栈中最接近栈顶的两个元素已经存入了两个int型的数值，当执行这个指令时，会将这两个int值出栈并相加，然后将相加的结果入栈。操作数栈中元素的数据类型必须与字节码指令的序列严格匹配，在编译程序代码的时候，编译器要严格保证这一点，在类校验阶段的数据流分析中还要再次验证这一点。再以上面的iadd指令为例，这个指令用于整型数加法，它在执行时，最接近栈顶的两个元素的数据类型必须为int型，不能出现一个long和一个float使用iadd命令相加的情况。



### 动态连接

每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的动态连接（Dynamic Linking）。我们知道 Class 文件的常量池中存有大量的符号引用，字节码中的方法调用指令就以常量池中指向方法的符号引用作为参数。这些符号引用一部分会在类加载阶段或者第一次使用的时候就转化为直接
引用，这种转化称为静态解析。另外一部分将在每一次运行期间转化为直接引用，这部分称为动态连接。这一部分在后面的方法调用详细解释。



### 方法返回地址

当一个方法开始执行后，只有两种方式可以退出这个方法。第一种方式是执行引擎遇到任意一个方法返回的字节码指令，这时候可能会有返回值传递给上层的方法调用者（调用当前方法的方法称为调用者），是否有返回值和返回值的类型将根据遇到何种方法返回指令来决定，这种退出方法的方式称为正常完成出口（Normal Method Invocation Completion），另外一种退出方式是，在方法执行过程中遇到了异常，并且这个异常没有在方法体内得到处理，无论是Java虚拟机内部产生的异常，还是代码中使用athrow字节码指令产生的异常，只要在本方法的异常表中没有搜索到匹配的异常处理器，就会导致方法退出，这种退出方法的方式称为异常完成出口（Abrupt Method Invocation Completion）。一个方法使用异常完成出口的方式退出，是不会给它的上层调用者产生任何返回值的。

无论采用何种退出方式，在方法退出之后，都需要返回到方法被调用的位置，程序才能继续执行，方法返回时可能需要在栈帧中保存一些信息，用来帮助恢复它的上层方法的执行状态。一般来说，方法正常退出时，调用者的PC计数器的值可以作为返回地址，栈帧中很可能会保存这个计数器值。而方法异常退出时，返回地址是要通过异常处理器表来确定的，栈帧中一般不会保存这部分信息。

方法退出的过程实际上就等同于把当前栈帧出栈，因此退出时可能执行的操作有：恢复上层方法的局部变量表和操作数栈，把返回值（如果有的话）压入调用者栈帧的操作数栈中，调整PC计数器的值以指向方法调用指令后面的一条指令等。



## 方法调用

方法调用并不等同于方法执行，方法调用阶段唯一的任务就是确定被调用方法的版本（即调用哪一个方法），暂时还不涉及方法内部的具体运行过程。在程序运行时，进行方法调用是最普遍、最频繁的操作，但前面提到，Class文件的编译过程中不包含传统编译中的连接步骤，一切方法调用在 Class 文件里面存储的都只是符号引用，而不是方法在实际运行时内存布局中的人口地址（相当于之前说的直接引用），这个特性给Java带来了更强大的动态扩展能力，但也使得Java方法调用过程变得相对复杂起来，需要在类加载期间，甚至到运行期间才能确定目标方法的直接引用。



### 解析

所有方法调用中的目标方法在 Class 文件里面都是一个常量池中的符号引用，在连接时期的解析阶段，会将其中的一部分符号引用转化为直接引用。这种解析能成立的前提是：方法在程序真正运行之前就有一个可确定的调用版本，并且这个方法的调用版本在运行期是不可改变的。换句话说，调用目标在程序代码写好、编译器进行编译时就必须确定下来。这类方法的调用称为解析（Resolution），在Java语言中符合“编译期可知，运行期不可变”这个要求的方法，主要包括静态方法和私有方法两大类，前者与类型直接关联，后者在外部不可被访问，这两种方法各自的特点决定了它们都不可能通过继承或别的方式重写其他版本，因此它们都适合在类加载阶段进行解析。

在Java虚拟机里面提供了5条方法调用字节码指令：

- invokestatic：调用静态方法。
- invokespecial：调用实例构造器`<init>`方法、私有方法和父类方法。
- invokevirtual：调用所有的虚方法。
- invokeinterface：调用接口方法，会在运行时再确定一个实现此接口的对象。
- invokedynamic：支持动态语言而存在的，文章的后续有详细分析。JDK7出现，JDK8真正派上用场。

只要能被 invokestatic 和 invokespecial 指令调用的方法，都可以在解析阶段中确定唯一的调用版本，符合这个条件的有静态方法、私有方法、实例构造器、父类方法4类，它们在类加载的时候就会把符号引用解析为该方法的直接引用。这些方法可以称为非虚方法，与之相反，其他方法称为虚方法。而final方法虽然也是通过 invokevirtual 调用的，但是实际上在编译期间就可以被确定，所以也是非虚方法。



### 静态分派（重载）

```java
public class MyTest2 {

    void test(Human human){
        System.out.println("human");
    }

    void test(Men men){
        System.out.println("men");
    }

    void test(Women women){
        System.out.println("women");
    }

    public static void main(String[] args) {
        MyTest2 myTest2 = new MyTest2();
        Human human1 = new Women();
        Human human2 = new Men();
        myTest2.test(human1);
        myTest2.test(human2);
    }

}
abstract class Human {  }
class Women extends Human {  }
class Men extends Human {    }
/*
human
human
*/
```

我们把上面代码中的“Human”称为变量的静态类型（Static Type），或者叫做的外观类型（Apparent Type），后面的“Man”则称为变量的实际类型（Actual Type），静态类型和实际类型在程序中都可以发生一些变化，区别是静态类型的变化仅仅在使用时发生，变量本身的静态类型不会被改变，并且最终的静态类型是在编译期可知的；而实际类型变化的结果在运行期才可确定，编译器在编译程序的时候并不知道一个对象的实际类型是什么。例如下
面的代码：

```java
// 实际类型变化
Human human = new Man();
human = new Women();
// 静态类型变化
test((Men) human);
test((Women) human);
```

使用哪个重载版本完全取决于传入参数的数量和数据类型。代码中刻意地定义了两个静态类型相同但实际类型不同的变量，但编译器在重载时是通过参数的静态类型而不是实际类型作为判定依据的。并且静态类型是编译期可知的，因此，在编译阶段，Javac编译器会根据参数的静态类型决定使用哪个重载版本。

所有依赖静态类型来定位方法执行版本的分派动作称为静态分派。静态分派的典型应用是方法重载。静态分派发生在编译阶段，因此确定静态分派的动作实际上不是由虚拟机来执行的。另外，编译器虽然能确定出方法的重载版本，但在很多情况下这个重载版本并不是“唯一的”，往往只能确定一个“更加合适的”版本。这种模糊的结论在由0和1构成的计算机世界中算是比较“稀罕”的事情，这种模糊结论的主要原因是字面量不需要定义，所以字面量没有显式的静态类型，它的静态类型只能通过语言上的规则去理解和推断。

##### 重载方法匹配优先级测试

```java
public class MyTest3 {
    public static void test(Object arg) {
        System.out.println("object");
    }
    public static void test(int arg) {
        System.out.println("int");
    }
    public static void test(long arg) {
        System.out.println("long");
    }
    public static void test(Character arg) {
        System.out.println("Character");
    }
    public static void test(char arg) {
        System.out.println("char");
    }
    public static void test(char... arg) {
        System.out.println("可变参数");
    }
    public static void test(Serializable arg) {
        System.out.println("Serializable");
    }
    public static void main(String[] args) {
        test('a');
    }
}
```

- 保留全部方法，运行后输出：char。
- 注释参数类型为char的方法，运行后输出：int。
- 再注释参数类型为int的方法，运行后输出：long。
- 再注释参数类型为Character的方法，运行后输出：Serializable。
- 再注释参数类型为Serializable的方法，运行后输出：object。
- 再注释参数类型为Object的方法，运行后输出：可变参数。



### 动态分配（重写）

```java
static abstract class Human {
    protected abstract void sayHello();
}
static class Man extends Human {
    @Override
    protected void sayHello() {
        System.out.println("man");
    }
}
static class Women extends Human {
    @Override
    protected void sayHello() {
        System.out.println("women");
    }
}

public static void main(String[] args) {
    Human human1 = new Man();
    Human human2 = new Women();
    human1.sayHello();
    human2.sayHello();
}
/*
man
women
*/
```

<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/e1124e8a-62f4-466b-89a7-0d04792f2589" /></div>
0~15行的字节码是准备动作，作用是建立 human1 和 human2 的内存空间，调用Man和 Woman类型的实例构造器，将这两个实例的引用存放在第1、2个局部变量表Slot之中，这个动作也就对应了代码中的这两句：

```java
Human human1 = new Man()
Human human2 new Woman()
```

接下来的16~21句是关键部分，16、20两句分别把刚刚创建的两个对象的引用压到栈顶，这两个对象是将要执行的 sayHello 方法的所有者，称为接收者（Receiver）；17和21句是方法调用指令，这两条调用指令单从字节码角度来看是完全一样的，但是这两句指令最终执行的目标方法并不相同。原因就需要从 invokevirtual 指令的多态查找过程开始说起，invokevirtual指令的运行时解析过程大致分为以下几个步骤：

1. 找到操作数栈顶的第一个元素所指向的对象的实际类型，记作C。
2. 如果在类型C中找到与常量中的描述符和简单名称都相符的方法，则进行访问权限校验，如果通过则返回这个方法的直接引用，查找过程结束；如果不通过，则返回java.lang.IllegalAccessError异常。
3. 否则，按照继承关系从下往上依次对C的各个父类进行第2步的搜索和验证过程。
4. 如果始终没有找到合适的方法，则抛出java.ang.AbstractMethodError异常。

由于 invokevirtual 指令执行的第一步就是在运行期确定接收者的实际类型，所以两次调用中的 invokevirtual 指令把常量池中的类方法符号引用解析到了不同的直接引用上，这个过程就是Java语言中方法重写的本质。我们把这种在运行期根据实际类型确定方法执行版本的分派过程称为动态分派。



### 单分派和多分派

方法的接收者与方法的参数统称为方法的宗量，这个定义最早应该来源于《Java与模式》一书。根据分派基于多少种宗量，可以将分派划分为单分派和多分派两种。单分派是根据一个宗量对目标方法进行选择，多分派则是根据多于一个宗量对目标方法进行选择。

```java
public class Dispatch {
    static class QQ {}
    static class _360 {}

    public static class Father {
        public void hardChoice(QQ arg) {
            System.out.println("father choose qq");
        }
        public void hardChoice(_360 arg) {
            System.out.println("father choose 360");
        }
    }
    public static class Son extends Father {
        public void hardChoice(QQ arg) {
            System.out.println("son choose qq");
        }
        public void hardChoice(_360 arg) {
            System.out.println("son choose 360");
        }
    }
     public static void main(String[] args) {
         Father father = new Father();
         Father son = new Son();
         father.hardChoice(new _360());
         son.hardChoice(new QQ());
     }
}
/*
father choose 360
son choose qq
*/
```

我们来看看编译阶段编译器的选择过程，也就是静态分派的过程。这时选择目标方法的依据有两点：一是静态类型是 `Father` 还是 `Son` ，二是方法参数是`QQ`还是`_360`。这次选择结果的最终产物是产生了两条 `invokevirtual` 指令，两条指令的参数分别为常量池中指向 `Father.hardChoice(_360)`及 `Father.hardChoice(QQ)`方法的符号引用。因为是根据两个宗量进行选择，所以Java语言的静态分派属于多分派类型。

看看运行阶段虚拟机的选择，也就是动态分派的过程。在执行`son.hardChoice(new QQ())`这句代码时，更准确地说，是在执行这句代码所对应的 `invokevirtual` 指令时，由于编译期已经决定目标方法的签名必须为`hardChoice(QQ)`，虚拟机此时不会关心传递过来的参数`QQ`到底是“腾讯QQ”还是“奇瑞QQ”，因为这时参数的静态类型、实际类型都对方法的选择不会构成任何影响，唯一可以影响虚拟机选择的因素只有此方法的接受者的实际类型是 `Father` 还是 `Son` 。因为只有一个宗量作为选择依据，所以Java语言的动态分派属于单分派类型。



### 虚方法表

由于动态分派是非常频繁的动作，而且动态分派的方法版本选择过程需要运行时在类的方法元数据中搜索合适的目标方法，因此在虚拟机的实际实现中基于性能的考虑，大部分实现都不会真正地进行如此频繁的搜索面对这种情况，最常用的“稳定优化”手段就是为类在方法区中建立一个虚方法表（Virtual Methed Table，也称为 vtable，与此对应的，在 invokeinterface 执行时也会用到接方法表 Inteface Method Table，称 itable），使用虚方法表索引来代替元数据查找以提高性能。

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/456cc571-b4e5-4d9b-8b48-92e7392beb85" /></div>
虚方法表中存放着各个方法的实际入口地址，如果某个方法在类中没有被重写，那子类的虚方法表里面的地址入口和父类相同方法的地址入口是一致的，都指向父类的实现入口。如果子类中重写了这个方法，子类方法表中的地址将会替换为指向子类实现版本的入口地址。



## 基于栈的字节码解释执行引擎

Java编译器输出的指令流，基本上是一种基于栈的指令集架构（Instruction Set Architecture，ISA），指令流中的指令大部分都是零地址指令，它们依赖操作数栈进行工作。与之相对的另外一套常用的指令集架构是基于寄存器的指令集。最典型的就是x86的二地址指令集，说得通俗一些，就是现在我们主流PC机中直接支持的指令集架构，这些指令依赖寄存器进行工作。前者的可移植性好，后者的性能高。



### 一个例子

```java
public int calc(){
	int a = 100;
	int b = 200;
	int c = 300;
	return (a + b) * c;
}
```

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/715bfa04-6d93-482f-b1ba-4ba52f24cee3" /></div>
<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/125fe7e2-85c2-48be-bfa2-da481d506512" /></div>
<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/f1153e3c-5572-4639-ae51-cee3f9587920" /></div>
<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/7903e7af-723c-43c0-94e1-ac5b06110546" /></div>
<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/6a114b15-b47e-45fc-b99d-b08790d8c5cb" /></div>
<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/beef1f48-f201-4a97-a146-dbcdfcb8dbad" /></div>
<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/764d9e06-89cc-4187-a335-7c0afd91464c" /></div>
## 反射的执行

在发射技术中，一般有两个过程，先找到某个方法，再执行方法。找到某个方法使用的是遍历的手段完成的。

```java
// 先查找本类中的方法，再查找父类的方法，再查找父接口的方法
private Method privateGetMethodRecursive(String name,
                                         Class<?>[] parameterTypes,
                                         boolean includeStaticMethods,
                                         MethodArray allInterfaceCandidates) {
    // Note: the intent is that the search algorithm this routine
    // uses be equivalent to the ordering imposed by
    // privateGetPublicMethods(). It fetches only the declared
    // public methods for each class, however, to reduce the
    // number of Method objects which have to be created for the
    // common case where the method being requested is declared in
    // the class which is being queried.
    //
    // Due to default methods, unless a method is found on a superclass,
    // methods declared in any superinterface needs to be considered.
    // Collect all candidates declared in superinterfaces in {@code
    // allInterfaceCandidates} and select the most specific if no match on
    // a superclass is found.

    // Must _not_ return root methods
    Method res;
    // Search declared public methods
    if ((res = searchMethods(privateGetDeclaredMethods(true),
                             name,
                             parameterTypes)) != null) {
        if (includeStaticMethods || !Modifier.isStatic(res.getModifiers()))
            return res;
    }
    // Search superclass's methods
    if (!isInterface()) {
        Class<? super T> c = getSuperclass();
        if (c != null) {
            if ((res = c.getMethod0(name, parameterTypes, true)) != null) {
                return res;
            }
        }
    }
    // Search superinterfaces' methods
    Class<?>[] interfaces = getInterfaces();
    for (Class<?> c : interfaces)
        if ((res = c.getMethod0(name, parameterTypes, false)) != null)
            allInterfaceCandidates.add(res);
    // Not found
    return null;
}
```

```java
// 用遍历的方式查找方法
private static Method searchMethods(Method[] methods, String name,
                                    Class<?>[] parameterTypes) {
    Method res = null;
    String internedName = name.intern();
    for (int i = 0; i < methods.length; i++) {
        Method m = methods[i];
        if (m.getName() == internedName
            && arrayContentsEq(parameterTypes, m.getParameterTypes())
            && (res == null
                || res.getReturnType().isAssignableFrom(m.getReturnType())))
            res = m;
    }
    return (res == null ? res : getReflectionFactory().copyMethod(res));
}
```

对于这个过程，在大规模使用反射技术时其实可以自己缓存一遍，比如使用哈希表的方式，这样就可以将时间复杂度降为`O(1)`。而对于方法的执行，我们再来看看`Method.invoke`方法。

```java
public Object invoke(Object obj, Object... args)
    throws IllegalAccessException, IllegalArgumentException, InvocationTargetException {
    if (!override) {
        if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
            Class<?> caller = Reflection.getCallerClass();
            checkAccess(caller, clazz, obj, modifiers);
        }
    }
    MethodAccessor ma = methodAccessor;             // read volatile
    if (ma == null) {
        ma = acquireMethodAccessor();
    }
    return ma.invoke(obj, args);
}
```

```java
private MethodAccessor acquireMethodAccessor() {
    // First check to see if one has been created yet, and take it if so
    MethodAccessor tmp = null;
    if (root != null) tmp = root.getMethodAccessor();
    if (tmp != null) {
        methodAccessor = tmp;
    } else {
        // Otherwise fabricate one and propagate it up to the root
        tmp = reflectionFactory.newMethodAccessor(this);
        setMethodAccessor(tmp);
    }
    return tmp;
}
```

从上面的代码可以看出，真正执行反射的对象是`MethodAccessor`接口的子类对象，此接口的继承图：

<div align="center"><img width="40%" src="http://blogfileqiniu.isjinhao.site/54130b64-9029-4bf9-bc8b-812bb7b06cf7" /></div>
`invoke`方法是在接口`MethodAccessor`中提供的，其中`MagicAccessorImpl`没有任何作用，`MethodAccessorImpl`也仅仅是提供了一个构造方法。

```java
private static boolean noInflation = false;

public MethodAccessor newMethodAccessor(Method var1) {
    checkInitted();
    if (noInflation && !ReflectUtil.isVMAnonymousClass(var1.getDeclaringClass())) {
        return (new MethodAccessorGenerator()).generateMethod(
            var1.getDeclaringClass(), var1.getName(), var1.getParameterTypes(), 
            var1.getReturnType(), var1.getExceptionTypes(), var1.getModifiers());
    } else {
        NativeMethodAccessorImpl var2 = new NativeMethodAccessorImpl(var1);
        DelegatingMethodAccessorImpl var3 = new DelegatingMethodAccessorImpl(var2);
        var2.setParent(var3);
        return var3;
    }
}
```

而对于`DelegatingMethodAccessorImpl`，它只是将`invoke`方法委托给别人：

```java
class DelegatingMethodAccessorImpl extends MethodAccessorImpl {
    private MethodAccessorImpl delegate;
    DelegatingMethodAccessorImpl(MethodAccessorImpl var1) { this.setDelegate(var1); }
    public Object invoke(Object var1, Object[] var2) 
        	throws IllegalArgumentException, InvocationTargetException {
        return this.delegate.invoke(var1, var2);
    }
    void setDelegate(MethodAccessorImpl var1) { this.delegate = var1; }
}
```

在`noInflation`为假时（初始值为假），真正执行方法的是`NativeMethodAccessorImpl`：

```java
class NativeMethodAccessorImpl extends MethodAccessorImpl {
    private final Method method;
    private DelegatingMethodAccessorImpl parent;
    private int numInvocations;

    NativeMethodAccessorImpl(Method var1) { this.method = var1; }

    public Object invoke(Object var1, Object[] var2) 
        	throws IllegalArgumentException, InvocationTargetException {
        if (++this.numInvocations > ReflectionFactory.inflationThreshold() && 
            	!ReflectUtil.isVMAnonymousClass(this.method.getDeclaringClass())) {
            MethodAccessorImpl var3 = (MethodAccessorImpl)(new MethodAccessorGenerator())
                .generateMethod(this.method.getDeclaringClass(), this.method.getName(), 
                         this.method.getParameterTypes(), this.method.getReturnType(), 
                         this.method.getExceptionTypes(), this.method.getModifiers());
            this.parent.setDelegate(var3);
        }
        return invoke0(this.method, var1, var2);
    }

    void setParent(DelegatingMethodAccessorImpl var1) { this.parent = var1; }
    private static native Object invoke0(Method var0, Object var1, Object[] var2);
}
```

而`NativeMethodAccessorImpl`中真正执行`invoke`的方法是本地方法，也就是说此时会将请求由Java平台转向C++平台，执行完毕后再返回Java平台。而什么时候`noInflation`为真呢？这是由调用次数决定的，当某个方法的调用次数超过阈值（默认为15）的时候就会生成该对象的字节码，然后就能向正常的方法一样执行了。这样做是由于生成字节码是一个耗时的操作，如果方法的调用次数很少就去生成字节码是不明智的做法。

- `noInflation`的默认值由`sun.reflect.noInflation`决定
- 阈值由`sun.reflect.inflationThreshold`决定



## 动态语言支持

### 什么是动态语言

仅从讲道理的角度说，动态语言指的是类型检查是在运行期进行而静态语言的类型检查是编译期进行。对于Java语言，简单来说，编译期就是源代码生成字节码的过程，运行期就是字节码执行的过程。那么什么是类型检查呢？对于一个方法调用，我们最终是要找到方法体里面的指令，所以必须先确定它的类型，此时我们来看下面的代码：

```java
obj.println("hello world");
```

对于一个有Java开发经验的人员来说，他脑子里一定会出现概念：`obj`是一个`PrintStream`类型的对象。诚然，如果`obj`是一个`PrintStream`类型的对象，这段代码可以达到我们想要的结果。不过如果`obj`不确定是不是`PrintStream`类型的对象，但是保证`obj`这个对象里有一个方法签名完全一致的方法`println()`，这段代码就不能达到我我们想要的结果吗？自然在Java中是不可以的，因为Java在编译的时候需要确定符号引用，比如上面的代码在编译是可能生成如下的指令：

```
invokevirtual #4;  //Method java/io/PrintStream.println:(Ljava/lang/String;)V
```

如果不确定类型，就无法生成完整的符号引用。但是对于动态语言，比如`JavaScript`，因为其不需要生成完整的符号引用，所以只要在运行的时候确定方法里面有符合要求的方法就可以了。比如下面的代码在动态语言中是可以实现的，因为只要保证所有的对象都有`sayHello()`这个方法就行了：

```
var arrays = {"abc", new ObjectX(), 123, Dog, Cat, Car..}
for(item in arrays){
	item.sayHello();
}
```

我们也可以考虑一下，如果Java想实现相同的功能可以怎么做呢？一个显而易见的办法就是编译时建立一个从数组index到符号引用的映射，执行的时候通过映射间接的寻找方法而不是通过符号引用直接寻找方法。



### invokedynamic

我们上面说道如果想要在Java中也能实现动态语言的功能，需要间接寻找方法而不是通过符号引用直接寻找，但是在JDK7之前的四个`invoke*`指令后面跟着的第一个参数都是被调用方法的符号引用（`CONSTANT_Methodref_info`和`CONSTANT_InterfaceMethodref_info`）。所以想要完成动态调用的功能，JDK7引入了一个新的指令：`invokedynamic`。其后跟的参数不再是符号引用，而是一个新的属性：`CONSTANT_InvokeDynamic_info`。这个属性主要包含三个参数： 

- bootstrap method
- the dynamic invocation name
- the argument and return types of the call

其中第一个参数最终会指向一个`MethodHandle`，这个名词直译过来就是方法句柄，它可以唯一确定一个方法。也就是说Java在实现动态方法调动时比我们刚才提出的映射到符号引用更近一步，它直接映射到了某个具体的方法。

同时每一处含有`invokeDynamic`指令的位置都被称为动态调用点，可以用`CallSite`类表示。下面我们看一段Lambda表达式的代码。

```java
public class Test {
    public static void main(String[] args) throws Exception {
    	Arrays.stream(new int[]{1}).forEach(i -> i = i + 1);
    }
}
```

反编译的结果：

```
Constant pool:
   #1 = Methodref          #6.#27         // java/lang/Object."<init>":()V
   #2 = Methodref          #28.#29        // java/util/Arrays.stream:([I)Ljava/util/stream/IntStream;
   #3 = InvokeDynamic      #0:#34         // #0:accept:()Ljava/util/function/IntConsumer;
   #4 = InterfaceMethodref #35.#36        // java/util/stream/IntStream.forEach:(Ljava/util/function/IntConsumer;)V
   #5 = Class              #37            // sb/core/Test
   #6 = Class              #38            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Lsb/core/Test;
  #14 = Utf8               main
  #15 = Utf8               ([Ljava/lang/String;)V
  #16 = Utf8               args
  #17 = Utf8               [Ljava/lang/String;
  #18 = Utf8               Exceptions
  #19 = Class              #39            // java/lang/Exception
  #20 = Utf8               MethodParameters
  #21 = Utf8               lambda$main$0
  #22 = Utf8               (I)V
  #23 = Utf8               i
  #24 = Utf8               I
  #25 = Utf8               SourceFile
  #26 = Utf8               Test.java
  #27 = NameAndType        #7:#8          // "<init>":()V
  #28 = Class              #40            // java/util/Arrays
  #29 = NameAndType        #41:#42        // stream:([I)Ljava/util/stream/IntStream;
  #30 = Utf8               BootstrapMethods
  #31 = MethodHandle       #6:#43         // invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
  #32 = MethodType         #22            //  (I)V
  #33 = MethodHandle       #6:#44         // invokestatic sb/core/Test.lambda$main$0:(I)V
  #34 = NameAndType        #45:#46        // accept:()Ljava/util/function/IntConsumer;
  #35 = Class              #47            // java/util/stream/IntStream
  #36 = NameAndType        #48:#49        // forEach:(Ljava/util/function/IntConsumer;)V
  #37 = Utf8               sb/core/Test
  #38 = Utf8               java/lang/Object
  #39 = Utf8               java/lang/Exception
  #40 = Utf8               java/util/Arrays
  #41 = Utf8               stream
  #42 = Utf8               ([I)Ljava/util/stream/IntStream;
  #43 = Methodref          #50.#51        // java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
  #44 = Methodref          #5.#52         // sb/core/Test.lambda$main$0:(I)V
  #45 = Utf8               accept
  #46 = Utf8               ()Ljava/util/function/IntConsumer;
  #47 = Utf8               java/util/stream/IntStream
  #48 = Utf8               forEach
  #49 = Utf8               (Ljava/util/function/IntConsumer;)V
  #50 = Class              #53            // java/lang/invoke/LambdaMetafactory
  #51 = NameAndType        #54:#58        // metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/
invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
  #52 = NameAndType        #21:#22        // lambda$main$0:(I)V
  #53 = Utf8               java/lang/invoke/LambdaMetafactory
  #54 = Utf8               metafactory
  #55 = Class              #60            // java/lang/invoke/MethodHandles$Lookup
  #56 = Utf8               Lookup
  #57 = Utf8               InnerClasses
  #58 = Utf8               (Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
  #59 = Class              #61            // java/lang/invoke/MethodHandles
  #60 = Utf8               java/lang/invoke/MethodHandles$Lookup
  #61 = Utf8               java/lang/invoke/MethodHandles
{
  public sb.core.Test();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 5: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lsb/core/Test;

  public static void main(java.lang.String[]) throws java.lang.Exception;
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=4, locals=1, args_size=1
         0: iconst_1
         1: newarray       int
         3: dup
         4: iconst_0
         5: iconst_1
         6: iastore
         7: invokestatic  #2                  // Method java/util/Arrays.stream:([I)Ljava/util/stream/IntStream;
        10: invokedynamic #3,  0              // InvokeDynamic #0:accept:()Ljava/util/function/IntConsumer;
        15: invokeinterface #4,  2            // InterfaceMethod java/util/stream/IntStream.forEach:(Ljava/util/function/IntConsumer;)V
        20: return
      LineNumberTable:
        line 10: 0
        line 12: 20
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      21     0  args   [Ljava/lang/String;
    Exceptions:
      throws java.lang.Exception
    MethodParameters:
      Name                           Flags
      args
}
SourceFile: "Test.java"
InnerClasses:
     public static final #56= #55 of #59; //Lookup=class java/lang/invoke/MethodHandles$Lookup of class java/lang/invoke/MethodHandles
BootstrapMethods:
  0: #31 invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #32 (I)V
      #33 invokestatic sb/core/Test.lambda$main$0:(I)V
      #32 (I)V
```

我们顺着第91行的`invokedynamic`向下看，其后面的参数`#3, 0`中的0是一个占位符，真正有助于分析的是`#3`。其表示常量池中的第三个常量：

```
#3 = InvokeDynamic      #0:#34         // #0:accept:()Ljava/util/function/IntConsumer;
```

这是`CONSTANT_InvokeDynamic_info`类型的常量的表示方法。`#0`表示bootstrap method是引导方法表里面的第0个索引的方法。此属性表在内部类中。`#34`是一个`NameAndType`类型的常量值，其实是函数式接口`intConsumer`的`accept`方法。

而从110行的属性表中可以看出，引导方法是`java.lang.invoke.LambdaMetafactory.metafactory`。返回值是`java.lang.invoke.CallSite;`。

通过Lambda表达式我们可以了解关于`invokedynamic`的一些特点，但是仍然不是很直接，所以我们来手工模拟一下指令的执行过程。不过在解释指令之前，我们先看`MethodHandle`的用法。

```
var arrays = {"abc", new ObjectX(), 123, Dog, Cat, Car..}
for(item in arrays){
	item.sayHello();
}
```

我们上面有一个这样的例子，说只要每个`item`都有`sayHello`方法就可以在动态语言中运行。我们不能直接模拟上面的代码，因为Java中没有这样的语法。但是也可以体现只要对象有指定方法签名的方法就可以调用的思想。

```java
public class MethodHandleTest {
    static class ClassA {
        public void println(String s) {  System.out.println("ClassA: " + s);  }
    }

    public static void main(String[] args) throws Throwable {
        for(int i = 0; i < 10; i++){
            Object object = 
            	System.currentTimeMillis() % 2 == 0 ? System.out : new ClassA();
            // 无论object是哪个实现类，下面这句都能正确调用到println方法
            getPrintlnMH(object).invokeExact("2020");
        }
    }

    private static MethodHandle getPrintlnMH(Object receiver) throws Exception {
        // MethodType: 代表“方法类型”，包含了方法的返回值(methodType()的第一个参数)
        // 和具体参数(methodType()第二个及以后的参数)。
        MethodType methodType = MethodType.methodType(void.class, String.class);
        // lookup()方法来自MethodHandles.lookup, 这句的作用是在指定类中查找符合给定的方法名称、
        // 方法类型，并且符合调用权限的方法句柄。
        // 因为这里调用的是一个虚方法，按照Java语言的规则，方法第一个参数是隐式的，
        // 代表该方法的接收者，也即是this指向的对象，这个参数以前是放在参数列表中进行传递的，
        // 而现在提供了bindTo()方法来完成这件事情
        return lookup().findVirtual(receiver.getClass(), 
        	"println", methodType).bindTo(receiver);
    }
}
```

我们刚才还说了一个名词，动态调用点（CallSite），但是现在看来不需要CallSite我们也能完成动态调用，那么它的作用是什么呢？我们看看JDK对它的解释。

> A CallSite is a holder for a variable MethodHandle, which is called its target. An invokedynamic instruction linked to a CallSite delegates all calls to the site's current target. A CallSite may be associated with several invokedynamic instructions, or it may be "free floating", associated with none. In any case, it may be invoked through an associated method handle called its dynamic invoker.

也就是说一个`CallSite`应该关联一个或多个`invokedynamic`指令，每个`CallSite`都关联一个`MethodHandle`。

```java
public class InvokeDynamicTest {
    public static void main(String[] args) throws Throwable {
        // 引导方法
        MethodType methodType = MethodType.methodType(void.class, String.class);
        MethodHandle methodHandleA = lookup().findStatic(A.class, "testMethod", methodType);

        // 动态调用点
        CallSite cs = new MutableCallSite(methodHandleA);

        // 按照jdk文档，如果没有关联至invokedynamic指令，就需要生成一个dynamic invoker。
        MethodHandle methodHandle = cs.dynamicInvoker();
        methodHandle.invokeExact("123");

        // 更换target。
        MethodHandle methodHandleB = lookup().findStatic(B.class, "testMethod", methodType);
        cs.setTarget(methodHandleB);
        methodHandle.invokeExact("456");
    }
}

class A {
    public static void testMethod(String s) {
        System.out.println("hello A：" + s);
    }
}
class B {
    public static void testMethod(String s) {
        System.out.println("hello B：" + s);
    }
}
```