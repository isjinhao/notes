## JVM & JDK & JRE

### JVM

Java虚拟机（JVM）是运行 Java 字节码的虚拟机。JVM有针对不同系统的特定实现（Windows，Linux，macOS），目的是使用相同的字节码，它们都会给出相同的结果。

**什么是字节码?采用字节码的好处是什么?**

> 在 Java 中，JVM可以理解的代码就叫做`字节码`（即扩展名为 `.class` 的文件），它不面向任何特定的处理器，只面向虚拟机。Java 语言通过字节码的方式，在一定程度上解决了传统解释型语言执行效率低的问题，同时又保留了解释型语言可移植的特点。所以 Java 程序运行时比较高效，而且，由于字节码并不针对一种特定的机器，因此，Java程序无须重新编译便可在多种不同操作系统的计算机上运行。

**Java 程序从源代码到运行一般有下面3步：**

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/009ba5ad-77cf-43f4-abed-80bb6a5f1437" /></div>
我们需要格外注意的是`.class->机器码`这一步。在这一步 JVM 类加载器首先加载字节码文件，然后通过解释器逐行解释执行，这种方式的执行速度会相对比较慢。而且，有些方法和代码块是经常需要被调用的（也就是所谓的热点代码），所以后面引进了 JIT 编译器，而JIT 属于运行时编译。当 JIT 编译器完成第一次编译后，其会将字节码对应的机器码保存下来，下次可以直接使用。而我们知道，机器码的运行效率肯定是高于 Java 解释器的。这也解释了我们为什么经常会说 Java 是编译与解释共存的语言。

> HotSpot采用了惰性评估（Lazy Evaluation）的做法，根据二八定律，消耗大部分系统资源的只有那一小部分的代码（热点代码），而这也就是JIT所需要编译的部分。JVM会根据代码每次被执行的情况收集信息并相应地做出一些优化，因此执行的次数越多，它的速度就越快。JDK 9引入了一种新的编译模式AOT（Ahead of Time Compilation），它是直接将字节码编译成机器码，这样就避免了JIT预热等各方面的开销。JDK支持分层编译和AOT协作使用。但是 ，AOT编译器的编译质量是肯定比不上JIT编译器的。

**总结：**

Java虚拟机（JVM）是运行Java字节码的虚拟机。JVM有针对不同系统的特定实现（Windows、Linux、macOS），目的是使用相同的字节码，它们都会给出相同的结果。字节码和不同系统的 JVM 实现是Java语言”一次编译，随处可以运行“的关键所在。



### JDK 和 JRE

JDK是Java Development Kit，它是功能齐全的Java SDK。它拥有JRE所拥有的一切，还有编译器（javac）和工具（如javadoc和jdb）。它能够创建和编译程序。

JRE是Java运行时环境。它是运行已编译 Java 程序所需的所有内容的集合，包括Java虚拟机（JVM），Java类库，Java命令和其他的一些基础构件。但是，它不能用于创建新程序。

如果只是为了运行一下Java程序的话，那么只需要安装JRE。但如果需要进行一些Java编程方面的工作，那么就需要安装JDK了。但是，这不是绝对的。有时，即使不打算在计算机上进行任何Java开发，仍然需要安装JDK。例如，如果要部署JSP-Web应用程序，需要JDK来将JSP编译成为Servlet。



## 异常

异常是程序在运行时出现的会导致程序运行终止的错误。这种错误是不能通过编译系统检查出来的。常见的异常的发生原因为系统资源错误和用户操所错误两种。

Java把异常信息封装成一个类。当发生某种异常时将某种对应的类作为异常信息抛出。异常的根类时Throwable，其有两个直接子类 Error和Exception。Error是描述系统资源错误的类。Exception是描述用户操作错误的类。

发生了Error就是必须修改代码或调整外部环境问题，程序肯定会终止。比如需要开辟一个内存大小为99999999个int的数组。所以一般来说Error很少发生。发生了Exception就要进行处理，使程序运行下去，如数组越界异常，文件不存在异常等。而Exception又可以分为两种，一种是程序本身存在的问题引发的异常（健壮性不够），即：RuntimeException；一种是程序本身可能没有问题，但遇到诸如文件不存在所导致的错误，Excption中除了RuntimeException外都是此种异常，此种异常被称为受查异常（受查异常在写代码的时候会提示抛出还是处理）。`Error+RuntimeException`构成了非受查异常。

<div align="center"><img width="65%" src="http://blogfileqiniu.isjinhao.site/3bdd4fc9-2ed9-4daa-b1fd-bed4e7004347"></div>
**受查异常、非受查异常**

Java语言规范规定派生于Error类或RuntimeException类的异常都称为非受查异常，其余异常都被成为受查异常。受查异常就是必须告诉它的调用者可能会出现异常，让其调用者抛出或者捕获处理，这类异常如果没有在程序中进行异常处理，编译不通过。非受查异常则不需要。

**声明受查异常（throws）**

除了Error和RuntimeException的异常都是受查异常，这些异常如：IOException、SQLException等，在可能发生的方法中需要被声明。同时非受查异常最好不要被声明。需要使用throws声明异常后的情况如下：

- 调用一个抛出受查异常的方法：`public int read() throws IOException`
- 方法中使用throw语句抛出了受查异常。

**抛出异常（throw）**

throw可以用于抛出异常对象。格式如下例：

```java
public static void demo() throws Exception{    
	throw new Exception("出现异常");
}
```



### 常见的运行时异常

- `java.lang.NullPointerException` ：空指针异常；
  - 出现原因：调用了未经初始化的对象或者是不存在的对象。

- `java.lang.ClassNotFoundException`：指定的类找不到；
  - 出现原因：类的名称和路径加载错误；通常都是程序试图通过字符串来加载某个类时可能引发异常。

- `java.lang.NumberFormatException`：字符串转换为数字异常；
  - 出现原因：字符型数据中包含非数字型字符。

- `java.lang.IndexOutOfBoundsException`：数组角标越界异常，常见于操作数组对象时发生。

- `java.lang.ClassCastException`：数据类型转换异常

- `SQLException`：SQL 异常，常见于操作数据库时的 SQL 语句错误。



### finally和return的执行顺序

finally在return语句执行后，return语句返回前执行。

```java
public class Main {
   public static void main(String[] args) {
       int j = query();
       System.out.println(j);
   }
   public static int query() {
       int i = 0;
       try {
           System.out.print("try\n");
           return i += 10;
       } catch (Exception e) {
           System.out.print("catch\n");
           i += 20;
       } finally {
           System.out.print("finally-i:"+i + "\n");
           i += 10;
           System.out.print("finally\n");
           //return i;
       }
       System.out.print("finish");
       return 200;
   }
}
/**
	try
    finally-i:10
    finally
    10
*/
```

代码中try语句块中，return i+=10; 这个时候i已经是10了，这个可以从输出的打印结果看出来，因为进入到finally语句的时候，有一个打印语句，打印结果中i就是10，就说明了return语句中的i+=10是已经执行了。 

```java
public class Main {
   public static void main(String[] args) {
       int j = query();
       System.out.println(j);
   }
   public static int query() {
       int i = 0;
       try {
           System.out.print("try\n");
           return i += 10;
       } catch (Exception e) {
           System.out.print("catch\n");
           i += 20;
       } finally {
           System.out.print("finally-i:"+i + "\n");
           System.out.print("finally\n");
           return i += 10;
       }
   }
}
/*
	try
    finally-i:10
    finally
    20
*/
```

try里`return i+=10`的`i+=10`被执行之后，return语句还没有返回，因为return语句返回代表该方法执行结束。在finally里`return i+=10`的`i+=10`被执行之后，i变成了20，此时的return会覆盖之前的return。

#### 由字节码执行顺序深入理解

**正常情况下指令**

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/72922b73-75e0-4f3e-b693-823d2b6fdb0a" /></div>
<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/e31ff637-674d-4915-96c2-844ce959b76a" /></div>
**异常情况下的处理**

```
Exception table:
	from    to  target type
		2    15    58   Class java/lang/Exception
		2    15   113   any
	58    70   113   any
```

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/286a6feb-ccb0-4f28-bad2-e69b9372624b" /></div>
如果2到15行指令出现了异常，会进入我们的catch中。



## final

**修饰类**

当用final修饰一个类时，表明这个类不能被继承。也就是说，如果一个类你永远不会让他被继承，就可以用final进行修饰。final类中的成员变量可以根据需要设为final，但是要注意final类中的所有成员方法都会被隐式地指定为final方法。

**修饰方法**

只有在想明确禁止该方法在子类中被覆盖的情况下才将方法设置为final的。即父类的final方法是不能被子类所覆盖的。

重写的前提是子类可以从父类中继承此方法，如果父类中final修饰的方法同时访问控制权限为private，将会导致子类中不能直接继承到此方法，因此，此时可以在子类中定义方法签名相同的方法，此时不再产生重写与final的矛盾，而是在子类中重新定义了新的方法。（注：类的private方法会隐式地被指定为final方法。）

**修饰变量**

final成员变量表示常量，只能被赋值一次，赋值后值不再改变。

- 当final修饰一个基本数据类型时，表示该基本数据类型的值一旦在初始化后便不能发生变化；
- 如果final修饰一个引用类型时，则在对其初始化之后便不能再让其指向其他对象了，但该引用所指向的对象的内容是可以发生变化的。本质上是一回事，因为引用的值是一个地址，final要求值，即地址的值不发生变化。

final修饰一个成员变量，必须要显示初始化。这里有两种初始化方式，一种是在变量声明的时候初始化；第二种方法是在声明变量的时候不赋初值，但是要在这个变量所在的类的所有的构造函数中对这个变量赋初值。

**finalize**

Object 类的一个方法，在垃圾回收器执行的时候会调用被回收对象的此方法，可以覆盖此方法提供垃圾收集时的其他资源回收，例如关闭文件等。该方法更像是一个对象生命周期的临终方法，当该方法被系统调用则代表该对象即将“死亡”，但是需要注意的是，我们主动行为上去调用该方法并不会导致该对象“死亡”，这是一个被动的方法（其实就是回调方法），不需要我们调用。

**effective final**

A variable is considered effective final if it is not modified after initialization in the local block. This means you can now use the local variable without final keyword inside an anonymous class or lambda expression, provided they must be effectively final.



## static

**静态方法不可以覆盖，但是可以被重新定义**（注意，静态方法可以被继承）

```java
public class Bks01 extends Dad {
    public static void main(String[] args) {
        test();             // son
        Dad.test();         // dad
    }
    // static方法可以被重新定义
    static void test() {	System.out.println("son");	}
}

class Dad {
    static void test() {	System.out.println("dad");	}
}
```

**static执行顺序**

- 静态先于普通
- 父类先于子类

```java
public class Bsk02 {
    static {	System.out.println("Bsk02 static");	}
    public Bsk02() {	System.out.println("Bsk02 constructor");	}
    public static void main(String[] args) {	
        new Son();	
    }
}

class Son extends Father {
    Bsk02 bsk02 = new Bsk02();
    static {	System.out.println("Son static");	}
    public Son() {	System.out.println("Son constructor");	}
}

class Father {
    static {	System.out.println("Father static");	}
    public Father() {	System.out.println("Father constructor");	}
}
/**
    Bsk02 static
    Father static
    Son static
    Father constructor
    Bsk02 constructor
    Son constructor
**/
```

1. 加载Bsk02类，执行静态代码块，执行main方法里的`new Son()`
3. 初始化子类之前先初始化父类Father，执行静态代码块
4. 初始化父类的静态变量/属性/代码块之后再执行子类Son的静态代码块
5. 再构建父类对象
6. 再初始化子类成员变量
7. 再构建子类对象



## 重载和重写

### 重写

- 参数列表必须完全与被重写方法的相同（方法签名一致）
- 返回类型与被重写方法的返回类型可以不相同，但是必须是父类返回值的派生类
- 访问权限不能比父类中被重写的方法的访问权限更低
- 父类的成员方法只能被它的子类重写
- 声明为final的方法不能被重写
- 声明为static的方法**不能被重写**，但是能够被再次声明
- 子类和父类在同一个包中，那么子类可以重写父类所有方法，除了声明为private和final的方法。
- 子类和父类不在同一个包中，那么子类只能够重写父类的声明为public和protected的非final方法。
- 重写的方法（子类）不能抛出新的受查异常，或者比被重写方法（父类）声明的更广泛的受查异常。非受查异常没有约束。
- 构造方法不能被重写。
- 如果不能继承一个方法，则不能重写这个方法。



### 重载

- 被重载的方法必须改变参数列表（参数个数或类型或顺序不一样）；
- 被重载的方法可以改变返回类型；
- 被重载的方法可以改变访问修饰符；
- 被重载的方法可以声明新的或更广的受查异常；
- 方法能够在同一个类中或者在一个子类中被重载。
- 无法以返回值类型作为重载函数的区分标准。



### 区别

|  区别点  |   重载方法   |                      重写方法                      |
| :------: | :----------: | :------------------------------------------------: |
| 参数列表 |   区分标准   |                    一定不能修改                    |
| 返回类型 | 不是区分标准 |                   相同或者派生类                   |
| 受查异常 | 不是区分标准 | 可以缩小范围或删除，一定不能抛出新的或者更广的异常 |
|   访问   | 不是区分标准 |       一定不能做更严格的限制（可以降低限制）       |



## 抽象类 & 接口

### 抽象类

- 抽象类中可以定义构造器
- 可以有抽象方法和具体方法
- 抽象类中可以定义成员变量
- 有抽象方法的类必须被声明为抽象类，而抽象类未必要有抽象方法
- 一个抽象类只能继承一个抽象类（和普通类一致）
- 抽象类可以包含静态方法

#### 抽象方法

- 抽象方法**不能同时是静态的**：抽象方法需要子类重写，而静态的方法是无法被重写的（只能被重新定义），因此二者是矛盾的。
- 抽象方法**不能同时是本地的**：本地方法是由本地代码（如C代码）实现的方法，而抽象方法是没有实现的，也是矛盾的。
- 抽象方法**不能同时被synchronized修饰**：synchronized加载普通方法上，拿的是对象的锁。但是抽象方法必须属于抽象类，抽象类本身不能存在实例。



### 接口

- 接口中不能定义构造器
- 方法全部都是抽象方法
- 抽象类中的成员可以是private、默认、protected、public
- 接口中定义的成员变量实际上都是常量
- 接口中不能有静态方法
- 一个类可以实现多个接口



### 接口和抽象类的区别是什么

1. 接口的方法默认是 public，接口中的方法不能有实现（Java 8开始接口方法可以有默认实现），而抽象类可以有非抽象的方法。
2. 接口中除了`static final`变量，不能有其他变量，而抽象类中则不一定。
3. 一个类可以实现多个接口，但只能实现一个抽象类。接口自己本身可以通过extends关键字扩展多个接口。
4. 接口方法默认修饰符是public，抽象方法可以有public、protected和default这些修饰符（抽象方法就是为了被重写所以不能使用private关键字修饰！）。

备注：在JDK8中，接口也可以定义静态方法，可以直接用接口名调用。实现类和实现是不可以调用的。如果同时实现两个接口，接口中定义了一样的默认方法，则必须重写，不然会报错。



## 对象克隆

### 多层浅拷贝

```java
public class CloneTest{
	public static void main(String[] args) throws CloneNotSupportedException {
		Body body = new Body(new Head(new Face(new String("丑"))));
		Body body1 = (Body) body.clone();
		System.out.println("body == body1 : " + (body == body1));
		System.out.println("body.head == body1.head : " + (body.head == body1.head));
		System.out.println(body.head.face == body1.head.face);
		System.out.println(body.head.face.name == body1.head.face.name);
	}
}

class Body implements Cloneable {
	public Head head;
	public Body(Head head) { this.head = head; }

	@Override
	protected Object clone() throws CloneNotSupportedException {
		Body newBody = (Body) super.clone();
		newBody.head = (Head) head.clone();
		return newBody;
	}
}

class Head implements Cloneable {
	public Face face;
	public Head(Face face) { this.face = face; }

	@Override
	protected Object clone() throws CloneNotSupportedException {
		Head newHead = (Head)super.clone();
		newHead.face = (Face) face.clone();
		return newHead;
	}
}

class Face implements Cloneable{
	
	public Face(String name) { this.name = name; }
	public String name;

	@Override
	protected Object clone() throws CloneNotSupportedException {
		Face newFace = (Face)super.clone();
		newFace.name = new String(this.name);
		return newFace;
	}
}
```



### 序列化

```java
public class CloneTest {
	public static void main(String[] args) {
		try {
			Person p1 = new Person("Hao LUO", 33, new Car("Benz", 300));
			Person p2 = CloneUtils.clone(p1); // 深度克隆
			p2.getCar().setBrand("BYD");
			System.out.println(p1);
			System.out.println(p2);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}

class CloneUtils {
	private CloneUtils() {
		throw new AssertionError();
	}

	@SuppressWarnings("unchecked")
	public static <T extends Serializable> T clone(T obj) throws Exception {
		ByteArrayOutputStream bout = new ByteArrayOutputStream();
		ObjectOutputStream oos = new ObjectOutputStream(bout);
		oos.writeObject(obj);
		ByteArrayInputStream bin = new ByteArrayInputStream(bout.toByteArray());
		ObjectInputStream ois = new ObjectInputStream(bin);
		return (T) ois.readObject();
	}
}

class Person implements Serializable {
	private static final long serialVersionUID = -9102017020286042305L;
	private String name; // 姓名
	private int age; // 年龄
	private Car car; // 座驾

	public Person(String name, int age, Car car) {
		this.name = name;
		this.age = age;
		this.car = car;
	}
    
	// ... getter方法和setter方法

	@Override
	public String toString() {
		return "Person [name=" + name + ", age=" + age + ", car=" + car + "]";
	}
}

class Car implements Serializable {
	private static final long serialVersionUID = -5713945027627603702L;
	private String brand; // 品牌
	private int maxSpeed; // 最高时速

	public Car(String brand, int maxSpeed) {
		this.brand = brand;
		this.maxSpeed = maxSpeed;
	}

	// ... getter和setter方法

	@Override
	public String toString() {
		return "Car [brand=" + brand + ", maxSpeed=" + maxSpeed + "]";
	}
}
```



## 基本类型和包装类型

```java
public static void testInter() {
    Integer a = new Integer(200);
    Integer b = new Integer(200);
    Integer c = 200;
    Integer e = 200;
    int d = 200;
    Object o = 200;
    System.out.println("object和包装类型                 " + (o == c));
    System.out.println("两个new出来的对象                 " + (a == b));
    System.out.println("new出的对象和用int赋值的Integer   " + (a == c));
    System.out.println("两个用int赋值的Integer            " + (c == e));
    System.out.println("基本类型和new出的对象             " + (d == a));
    System.out.println("基本类型和自动装箱的对象           " + (d == c));

    Integer f = 100;
    Integer g = 100;
    System.out.println("-128到127之间                    " + (f == g));
}
/**
    object和包装类型                 false
    两个new出来的对象                 false
    new出的对象和用int赋值的Integer   false
    两个用int赋值的Integer            false
    基本类型和new出的对象             true
    基本类型和自动装箱的对象           true
    -128到127之间                    true
*/
```

自动装箱发生的过程是：基本类型转换为包装类型。自动拆箱发生的过程是：包装类型转换为基本类型。

```java
Integer i1 = 40;
Integer i2 = 40;
Integer i3 = 0;
Integer i4 = new Integer(40);
Integer i5 = new Integer(40);
Integer i6 = new Integer(0);

System.out.println("i1 = i2   " + (i1 == i2));
System.out.println("i1 = i2+i3   " + (i1 == i2 + i3));
System.out.println("i1 = i4   " + (i1 == i4));
System.out.println("i4 = i5   " + (i4 == i5));
System.out.println("i4 == i5+i6   " + (i4 == i5 + i6));   
System.out.println("40 == i5+i6   " + (40 == i5 + i6));  
/*
	i1 == i2   true
    i1 == i2+i3   true
    i1 == i4   false
    i4 == i5   false
    
    语句i4 == i5 + i6，因为+这个操作符不适用于Integer对象，首先i5和i6进行自动拆箱操作，进行数
    值相加，即i4 == 40。然后Integer对象无法与数值进行直接比较，所以i4自动拆箱转为int值40，最终
    这条语句转为40 == 40进行数值比较。
    i4 == i5+i6   true
    40 == i5+i6   true
*/
```

Java基本类型的包装类中：Byte，Short，Integer，Long，Character，Boolean实现了常量池技术；Float，Double 并没有实现常量池技术。

Character缓存了[0,127]之间的数据，Boolean缓存了true和false，其他4种包装类默认创建了数值[-128,127]的相应类型的缓存数据，但是超出此范围仍然会去创建新的对象。



### int的范围

```java
// 补码会进行循环
System.out.println(Integer.MAX_VALUE);		// 2147483647
System.out.println(Integer.MAX_VALUE + 1);	// -2147483648
System.out.println(Integer.MAX_VALUE + 2);	// -2147483647

System.out.println(Integer.MIN_VALUE);		// -2147483648
System.out.println(Integer.MIN_VALUE - 1);	// 2147483647
System.out.println(Integer.MIN_VALUE - 2);	// 2147483646
```



### 基本类型之间的转换

- byte、short、int、long：小范围向大范围的可以自动转，大范围向小范围需要强制转。
- char可以自动转换为int、long，可以强制转换为short、byte。byte、short、int、long都需要强制转换才能转为char。
- float自动转换为double，double强制转换为float。
- byte、short、int、long可以自动float和double。反之需要强制转换。
- `+=`和`-=`包含自动转换，即`short s1 = 1; s1 += 1;`不会报错。



## 常量池

### 三个关于池子的名词

#### 全局字符串池（string pool也有叫做string literal pool）

全局字符串池里的内容是在类加载完成，经过验证，准备阶段之后在堆中生成字符串对象实例，然后将该字符串对象实例的引用值存到string pool中（**记住：string pool中存的是引用值而不是具体的实例对象，具体的实例对象是在堆中开辟的一块空间存放的。**）。

在HotSpot VM里实现的string pool功能的是一个StringTable类，它是一个哈希表，里面存的是驻留字符串（也就是我们常说的用双引号括起来的）的引用（而不是驻留字符串实例本身），也就是说在堆中的某些字符串实例被这个StringTable引用之后就等同被赋予了”驻留字符串”的身份。这个StringTable在每个HotSpot VM的实例只有一份，被所有的类共享。（字符串常量池本身也是存在堆中的）

#### class文件常量池（class constant pool）

我们都知道，class文件中除了包含类的版本、字段、方法、接口等描述信息外，还有一项信息就是常量池（constant pool table），用于存放编译器生成的**各种字面量（Literal）和符号引用（Symbolic References）**。字面量就是我们所说的常量概念，如文本字符串、被声明为final且编译期能被确定的常量值等。符号引用是一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可（它与直接引用区分一下，直接引用一般是指向方法区的本地指针，相对偏移量或是一个能间接定位到目标的句柄）。一般包括下面三类常量：

- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符

常量池的每一项常量都是一个表，一共有如下表所示的11种各不相同的表结构数据，这每个表开始的第一位都是一个字节的标志位（取值1-12），代表当前这个常量属于哪种常量类型。

#### 运行时常量池（runtime constant pool）

当java文件被编译成class文件之后，也就是会生成class常量池。jvm在执行某个类的时候，必须经过**加载、连接、初始化**，而连接又包括验证、准备、解析三个阶段。而当类加载到内存中后，jvm就会将class常量池中的内容存放到运行时常量池中，由此可知，运行时常量池也是每个类都有一个。在上面我也说了，class常量池中存的是字面量和符号引用，也就是说他们存的并不是对象的实例，而是对象的符号引用值。而经过解析（resolve）之后，也就是把符号引用替换为直接引用，解析的过程会去查询全局字符串池，也就是我们上面所说的StringTable，以保证运行时常量池所引用的字符串与全局字符串池中所引用的是一致的。JDK8的运行时常量池存放在堆中。



### String和字符串常量池

#### String对象的两种创建方式

```java
// 直接使用双引号声明出来的 String 对象会直接存储在常量池中。
// 使用双引号声明时会调用ldc命令，所以其检查的是字符串常量池。
String str1 = "abcd";

// 对于new String()则会在堆中创建一个String对象，并返回该对象的引用。
// Initializes a newly created String object so that it represents the same sequence 
// of characters as the argument; in other words, the newly created string is a copy 
// of the argument string. Unless an explicit copy of original is needed, use of this 
// constructor is unnecessary since Strings are immutable.
String str2 = new String("abcd");

System.out.println(str1 == str2);	//false
```

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/cf985854-486b-48b2-b22a-d0f9542a4756"></div>
*上图中的常量池指的是运行时常量池。*

#### intern()

String.intern() 是一个 Native 方法，JDK对它的解释是：

> A pool of strings, initially empty, is maintained privately by the class String. When the intern method is invoked, if the **string** pool already contains a string equal to this String object as determined by the equals(Object) method, then the string from the pool is returned. Otherwise, this String object is added  to the **string** pool and a reference to this String object is returned.（笔者注：这里的添加到字符串常量池实际上是在**首次解析**发现字符串常量池中没有才会添加到字符串常量池中）
> It follows that for any two strings s and t, s.intern() == t.intern() is true if and only if s.equals(t) is true.
> **All literal strings and string-valued constant expressions are interned**.

深入理解java虚拟机里面有一点解释的不是很规范，书里的解释字符串常量池里存的是“某个字符串实例首次出现的引用”，实际上应该说是“某个字符串实例首次被解析时的引用”，因为在类型的生命周期中，解析阶段需要将符号引用转为直接引用，这个阶段所使用到的字符串才符合书里说的“首次出现”这个概念，但问题是解析这个过程的发生只保证在new，invokespecial等12个指令出现之前。所以在面试题中，我一般将首次使用所为首次解析。

#### 题

**1**

```java
String s1 = new String("计算机");
String s2 = s1.intern();
String s3 = "计算机";
System.out.println(s2);
System.out.println(s1 == s2);	// false
System.out.println(s3 == s2);	// true
```

<div align="center"><img width="90%" src="http://blogfileqiniu.isjinhao.site/2dc48da6-9627-4e34-af33-47080e1b9ea3" /></div>
**2**

```java
String s1 = "计算机";
String s2 = new String("计算机");
String s3 = new String("计算机").intern();

System.out.println(s1 == s2); // false
System.out.println(s1 == s3); // true
```

<div align="center"><img width="90%" src="http://blogfileqiniu.isjinhao.site/07bbbec7-4f23-440a-b261-ef0ecdb6ca41" /></div>
**3**

```java
String s3 = new String("1") + new String("2");
System.out.println(s3 == s3.intern());	// true
```

<div align="center"><img width="90%" src="http://blogfileqiniu.isjinhao.site/2df8478d-173b-432c-96e7-64c255cf3fdd" /></div>
**4**

```java
String str = "12";	// 首次解析 12
String s3 = new String("1") + new String("2");
System.out.println(s3 == s3.intern());	// false
```

<div align="center"><img width="90%" src="http://blogfileqiniu.isjinhao.site/c5c7356d-e5e3-4439-8d91-1df7c09c8bd9" /></div>
**5**


```java
String s1 = new String("xy") + "z";
String s2 = s1.intern();	// 首次解析xyz
System.out.println(s1 == s1.intern());    // true 
System.out.println(s2 == s1.intern()); 		// true 
```

<div align="center"><img width="90%" src="http://blogfileqiniu.isjinhao.site/f3a7ec71-a668-42fb-bfdc-e60636bacb7a" /></div>
**6**

```java
String s1 = new String("xy") + "z";
String s3 = new String("xyz");	// 首次解析xyz
String s2 = s1.intern();
System.out.println(s2 == s3);	// false
System.out.println(s1 == s1.intern());    // false
System.out.println(s2 == s1.intern());  // true
```

<div align="center"><img width="90%" src="http://blogfileqiniu.isjinhao.site/cdd18456-b1c4-4325-a2da-a3d1c3c72554" /></div>
**7**

```java
String s1 = new String("xy") + "z";
String s2 = s1.intern();
String s3 = new String("xyz");
System.out.println(s2 == s3);	// false
System.out.println(s1 == s1.intern());    // true
System.out.println(s2 == s1.intern());	// true
```

<div align="center"><img width="90%" src="http://blogfileqiniu.isjinhao.site/5e8eeffa-c9f8-46cf-b5dc-9fec4a6e5106" /></div>
**8**

```java
String s1 = new String("xy") + "z";
String s2 = s1.intern();
String s0 = "xyz";
String s3 = new String(s0);
System.out.println(s2 == s3);  // false
System.out.println(s1 == s2);  // true
System.out.println(s1 == s3);  // false
System.out.println(s0 == s1);  // true
System.out.println(s2 == s3);  // false
```

<div align="center"><img width="90%" src="http://blogfileqiniu.isjinhao.site/9edb0838-a0ea-4e00-a1c6-e4cfbeee6c18" /></div>
#### 深入分析运行时常量池

深入理解Java虚拟机第二版又这么一句话：JDK1.7（以及部分其他虚拟机，例如JRockit）的 intern 实现不会再复制实例，只是在常量池中记录首次出现的实例引用。

刚开始看到这句话其实是有点迷的，因为笔者之前理解的常量池中存放的应该是对象，为何引用也能被放进去？不过分析一下之后发现其实在运行时常量池里面应该也是使用键值对形式来保存数据的。否则诸如ldc等指令从常量池中拿数据时就只能采用遍历的方式，这个时间复杂度是不能忍受的，所以笔者暂且将运行时常量池保存数据的结果画作下图。

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/0e33c618-286b-46d7-a240-479f91d3c7f1" /></div>
## 文件的表示

### File

**构造**

```java
// public File(String filepath);
// 绝对路径:以盘符开头的路径
// 相对路径:相对当前项目的根目录

public File(String parent, String child);
public File(File parent,String child);
public File(URI uri);
```

```java
File aFile;
try {
    aFile = new File(new URI("file:///https://isjinhao.github.io/images/avatar.gif"));
    System.out.println(aFile.getName());
} catch (URISyntaxException e) {
    e.printStackTrace();
}
```

**获取方法**

```java
public String getAbsolutePath();  // 获取绝对路径
public String getName();  // 获取当前File对象的名字
public String getPath();  // 获取创建File对象时 传递的路径

public long length();
// 获取表示文件的File对象的占用的字节数
// 如果是文件夹的File对象,返回目录本身的大小,不是目录及其所有孩子的大小
```

**创建文件**

```java
public boolean createNewFile(); // 创建一个新的文件（只能是文件，不能是文件夹），返回是否创建成功
```

**创建文件夹**

```java
public boolean mkdir();  // 创建一个新的文件夹，返回是否创建成功
```

**判断File对象所表示的文件在OS中是否存在**

```java
public boolean exists();   // 返回该File对象是否存在
```

**判断是否是文件**

```java
public boolean isFile();   // 返回是否是文件
```

**判断是否是文件夹**

```java
public boolean isDirectory();  // 返回是否是文件夹
```

**删除**

```java
public boolean delete();  //  删除文件或者文件夹。可以删除的是单个文件，或者空文件夹
```

**list和listFiles**

```java
public String[] list();
public File[] listFiles(); 	// 只能列出当前文件夹下的一级子文件或者子文件夹
```

**文件过滤**

```java
public class Demo2 {
	public static void main(String[] args) {
		File fileDir = new File("D:\\blog\\isjinhao\\source\\_posts\\04-进程管理");
		// 列出file下所有file对象
		MyFileFilter ff = new MyFileFilter();
		File[] files = fileDir.listFiles(ff);
		for (File file : files) {
			System.out.println(file);
		}
	}
}
class MyFileFilter implements FileFilter{
	@Override
	public boolean accept(File pathname) {
		String name = pathname.getName();
		if(name.endsWith(".png") || name.endsWith(".PNG"))
			return true;
		return false;
	}
}
```



### Path

Path是JDK7中表达路径的一个新方式，在Path中，它把文件的路径看做几个**部件**组成的，比如`/usr/develop/tomcat`可以被看出两个部件组成：`/usr`和`/develop/tomcat`，当然也可以看做三个部件`/usr`、`/develop`和`/tomcat`组成的。以根部件开始的是绝对路径，在类Unix系统中是`\`，在Windows系统中是`C:\`等。

**获得Path**

通过Paths的静态方法：

1. `static Path get(String first, String ... more);`
2. `public static Path get(URI uri);`

通过连接给定的字符串创建一个路径。

**按当前路径解析路径**

1. `Path resolve(Path other);`
2. `Path resolve(String other);`

如果other是绝对路径，那么返回other；否则，返回通过连接this和other获得路径。

```java
Path path1 = Paths.get("D:\\", "data.csv");
Path path2 = Paths.get("test\\test", "选修课数据修改.csv");

Path path3 = path1.resolve(path2);
Path path4 = path2.resolve(path1);

System.out.println(path3);	// D:\data.csv\test\test\选修课数据修改.csv
System.out.println(path4);	// D:\data.csv
```

**按当前路径解析路径**

1. `Path resolveSibling(Path other);`
2. `Path resolveSibling(String other);`

如果other是绝对路径，那么返回other；否则，返回通过连接this的父路径和other获得路径。

```java
//define the fixed path
 Path base = Paths.get("C:/rafaelnadal/tournaments/2009/BNP.txt");
 
//resolve sibling AEGON.txt file
 Path path = base.resolveSibling("AEGON.txt");
 //output: C:\rafaelnadal\tournaments\2009\AEGON.txt
 System.out.println(path.toString());
```

**按相对路径进行解析**

```java
Path path01 = Paths.get("BNP.txt");
Path path02 = Paths.get("AEGON.txt");
System.out.println(path01);     // BNP.txt
Path path01_to_path02 = path01.relativize(path02);      // ..\AEGON.txt
System.out.println(path01_to_path02);
System.out.println(path02);     // AEGON.txt
Path path02_to_path01 = path02.relativize(path01);      // ..\BNP.txt
System.out.println(path02_to_path01);
```

**其他API**

1. 移除诸如.和..等的冗余元素：`Path normalize();`
2. 返回和当前路径相等价的绝对路径：`Path toAbsolutePath();`
3. 返回父路径（没有时返回null）：`Path getParent();`
4. 返回该路径的最后一个部件：`Path getFileName();`
5. 返回该路径的根部件（没有时返回null）：`Path getRoot();`
6. 由Path创建一个File对象：`File toFile();`



### Files

**处理小型文本文件**

- `public static byte[] readAllBytes(Path path) throws IOException`

```java
public static void main(String[] args) {	
	try {
		byte[] bytes = Files.readAllBytes(Paths.get("filestest"));
		String string = new String(bytes, "UTF-8");
		System.out.println(string);
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}
}
```

- `public static List<String> readAllLines(Path path) throws IOException`

```java
public static void main(String[] args) throws Exception {
	List<String> lines = Files.readAllLines(Paths.get("filestest"));
	Iterator<String> iterator = lines.iterator();
	while(iterator.hasNext())
		System.out.println(iterator.next());
}
```

- `public static Path write(Path path, byte[] bytes, OpenOption... options) throws IOException`

```java
public static void main(String[] args) throws Exception {
	Files.write(Paths.get("filestest"), "深陷琪中，钰罢不能".getBytes(), 	
                StandardOpenOption.APPEND);
}
```

- 获得IO流
  - `public static InputStream newInputStream(Path path, OpenOption... options)`
  - `public static OutputStream newOutputStream(Path path, OpenOption... options)`
  - `public static BufferedReader newBufferedReader(Path path, Charset cs)`
  - `public static BufferedReader newBufferedReader(Path path)`

**Files.exists()**

`Files.exists()`方法检查给定的`Path`在文件系统中是否存在。

可以创建在文件系统中不存在的`Path`实例。例如，如果您计划创建一个新目录，您首先要创建相应的`Path`实例，然后创建目录。

由于`Path`实例可能指向，也可能没有指向文件系统中存在的路径，你可以使用`Files.exists()`方法来确定它们是否存在（如果需要检查的话）。

这里是一个`Files.exists()`的例子：

```java
Path path = Paths.get("data/logging.properties");
boolean pathExists = Files.exists(path, new LinkOption[]{ LinkOption.NOFOLLOW_LINKS });
```

这个例子首先创建一个`Path`实例指向一个路径，我们想要检查这个路径是否存在。然后，这个例子调用`Files.exists()`方法，然后将`Path`实例作为第一个参数。

注意`Files.exists()`方法的第二个参数。这个参数是一个选项数组，它影响`Files.exists()`如何确定路径是否存在。在上面的例子中的数组包含`LinkOption.NOFOLLOW_LINKS`，这意味着`Files.exists()`方法不应该在文件系统中跟踪符号链接，以确定文件是否存在。

**Files.createDirectory()**

`Files.createDirectory()`方法，用于根据`Path`实例创建一个新目录：

```java
Path path = Paths.get("data/subdir");
try {
    Path newDir = Files.createDirectory(path);
} catch(FileAlreadyExistsException e){
    // 目录已经存在
} catch (IOException e) {
    // 其他发生的异常
    e.printStackTrace();
}
```

第一行创建表示要创建的目录的`Path`实例。在`try-catch`块中，用路径作为参数调用`Files.createDirectory()`方法。如果创建目录成功，将返回一个`Path`实例，该实例指向新创建的路径。

如果该目录已经存在，则是抛出一个`java.nio.file.FileAlreadyExistsException`。如果出现其他错误，可能会抛出`IOException`。例如，如果想要的新目录的父目录不存在，则可能会抛出`IOException`。父目录是您想要创建新目录的目录。因此，它表示新目录的父目录。

**Files.copy()**

`Files.copy()`方法从一个路径拷贝一个文件到另外一个目录，这里是一个Java `Files.copy()`例子：

```java
Path sourcePath      = Paths.get("data/logging.properties");
Path destinationPath = Paths.get("data/logging-copy.properties");

try {
    Files.copy(sourcePath, destinationPath);
} catch(FileAlreadyExistsException e) {
    // 目录已经存在
} catch (IOException e) {
    // 其他发生的异常
    e.printStackTrace();
}
```

首先，该示例创建一个源和目标`Path`实例。然后，这个例子调用`Files.copy()`，将两个`Path`实例作为参数传递。这可以让源路径引用的文件被复制到目标路径引用的文件中。

如果目标文件已经存在，则抛出一个`java.nio.file.FileAlreadyExistsException`异常。如果有其他错误，则会抛出一个`IOException`。例如，如果将该文件复制到不存在的目录，则会抛出`IOException`。

**重写已存在的文件**

可以强制`Files.copy()`覆盖现有的文件。这里有一个示例，演示如何使用`Files.copy()`覆盖现有文件。

```java
Path sourcePath      = Paths.get("data/logging.properties");
Path destinationPath = Paths.get("data/logging-copy.properties");
try {
    Files.copy(sourcePath, destinationPath, StandardCopyOption.REPLACE_EXISTING);
} catch(FileAlreadyExistsException e) {
    // 目标文件已存在
} catch (IOException e) {
    // 其他发生的异常
    e.printStackTrace();
}
```

请注意`Files.copy()`方法的第三个参数。如果目标文件已经存在，这个参数指示`copy()`方法覆盖现有的文件。

**Files.move()**

 `Files`还包含一个函数，用于将文件从一个路径移动到另一个路径。移动文件与重命名相同，但是移动文件既可以移动到不同的目录，也可以在相同的操作中更改它的名称。是的，`java.io.File`类也可以使用它的`renameTo()`方法来完成这个操作，但是现在已经在`java.nio.file.Files`中有了文件移动功能。

这里有一个`Files.move()`例子：

```java
Path sourcePath      = Paths.get("data/logging-copy.properties");
Path destinationPath = Paths.get("data/subdir/logging-moved.properties");
try {
    Files.move(sourcePath, destinationPath, StandardCopyOption.REPLACE_EXISTING);
} catch (IOException e) {
    // 移动文件失败
    e.printStackTrace();
}
public enum StandardCopyOption implements CopyOption {
    /**
     * Replace an existing file if it exists.
     */
    REPLACE_EXISTING,
    /**
     * Copy attributes to the new file.
     */
    COPY_ATTRIBUTES,
    /**
     * Move the file as an atomic file system operation.
     */
    ATOMIC_MOVE;
}
```

首先创建源路径和目标路径。源路径指向要移动的文件，而目标路径指向文件应该移动到的位置。然后调用`Files.move()`方法。这会导致文件被移动。

请注意传递给`Files.move()`的第三个参数。这个参数告诉`Files.move()`方法来覆盖目标路径上的任何现有文件。这个参数实际上是可选的。

如果移动文件失败，`Files.move()`方法可能抛出一个`IOException`。例如，如果一个文件已经存在于目标路径中，并且您已经排除了`StandardCopyOption.REPLACE_EXISTING`选项，或者被移动的文件不存在等等。

**Files.delete()**

`Files.delete()`方法可以删除一个文件或者目录。下面是一个Java `Files.delete()`例子：

```java
Path path = Paths.get("data/subdir/logging-moved.properties");

try {
    Files.delete(path);
} catch (IOException e) {
    // 删除文件失败
    e.printStackTrace();
}
```

首先，创建指向要删除的文件的`Path`。然后调用`Files.delete()`方法。如果`Files.delete()`由于某种原因不能删除文件（例如，文件或目录不存在），会抛出一个`IOException`。

#### 文件搜索

**Files.walkFileTree()**

`Files.walkFileTree()`方法包含递归遍历目录树的功能。`walkFileTree()`方法将`Path`实例和`FileVisitor`作为参数。`Path`实例指向您想要遍历的目录。`FileVisitor`在遍历期间被调用。

在我解释遍历是如何工作之前，这里我们先了解`FileVisitor`接口:

```java
public interface FileVisitor<T> {
    FileVisitResult preVisitDirectory(T dir, BasicFileAttributes attrs)
        throws IOException;

    FileVisitResult visitFile(T file, BasicFileAttributes attrs)
        throws IOException;

    FileVisitResult visitFileFailed(T file, IOException exc)
        throws IOException;

    FileVisitResult postVisitDirectory(T dir, IOException exc)
        throws IOException;
}
```

您必须自己实现`FileVisitor`接口，并将实现的实例传递给`walkFileTree()`方法。在目录遍历过程中，您的`FileVisitor`实现的每个方法都将被调用。如果不需要实现所有这些方法，那么可以扩展`SimpleFileVisitor`类，它包含`FileVisitor`接口中所有方法的默认实现。

这里是一个`walkFileTree()`的例子：

```java
Files.walkFileTree(path, new FileVisitor<Path>() {
    @Override
    public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs)
        	throws IOException {
        System.out.println("pre visit dir:" + dir);
        return FileVisitResult.CONTINUE;
    }
    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs)
        	throws IOException {
        System.out.println("visit file: " + file);
        return FileVisitResult.CONTINUE;
    }
    @Override
    public FileVisitResult visitFileFailed(Path file, IOException exc)
        	throws IOException {
        System.out.println("visit file failed: " + file);
        return FileVisitResult.CONTINUE;
    }
    @Override
    public FileVisitResult postVisitDirectory(Path dir, IOException exc)
        	throws IOException {
        System.out.println("post visit directory: " + dir);
        return FileVisitResult.CONTINUE;
    }
});
```

`FileVisitor`实现中的每个方法在遍历过程中的不同时间都被调用:

在访问任何目录之前调用`preVisitDirectory()`方法。在访问一个目录之后调用`postVisitDirectory()`方法。

调用`visitFile()`在文件遍历过程中访问的每一个文件。它不会访问目录-只会访问文件。在访问文件失败时调用`visitFileFailed()`方法。例如，如果您没有正确的权限，或者其他什么地方出错了。

这四个方法中的每个都返回一个`FileVisitResult`枚举实例。`FileVisitResult`枚举包含以下四个选项:

- CONTINUE：继续，意味着文件的执行应该像正常一样继续。
- TERMINATE：终止，意味着文件遍历现在应该终止。
- SKIP_SIBLING：跳过同级，意味着文件遍历应该继续，但不需要访问该文件或目录的任何同级。
- SKIP_SUBTREE：意味着文件遍历应该继续，但是不需要访问这个目录中的子目录。这个值只有从`preVisitDirectory()`返回时才是一个函数。如果从任何其他方法返回，它将被解释为一个`CONTINUE`继续。

通过返回其中一个值，调用方法可以决定如何继续执行文件。

**文件搜索**

这里是一个用于扩展`SimpleFileVisitor`的`walkFileTree()`，以查找一个名为`README.txt`的文件:

```java
Path rootPath = Paths.get("data");
String fileToFind = File.separator + "README.txt";
try {
    Files.walkFileTree(rootPath, new SimpleFileVisitor<Path>() {

        @Override
        public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) 
            	throws IOException {
            String fileString = file.toAbsolutePath().toString();
            // System.out.println("pathString = " + fileString);

            if(fileString.endsWith(fileToFind)){
                System.out.println("file found at path: " + file.toAbsolutePath());
                return FileVisitResult.TERMINATE;
            }
            return FileVisitResult.CONTINUE;
        }
    });
} catch(IOException e){
    e.printStackTrace();
}
```

**递归删除目录**

`Files.walkFileTree()`也可以用来删除包含所有文件和子目录的目录。`Files.delete()`方法只会删除一个目录，如果它是空的。通过遍历所有目录并删除每个目录中的所有文件(在`visitFile()`)中，然后删除目录本身(在`postVisitDirectory()`中)，您可以删除包含所有子目录和文件的目录。下面是一个递归目录删除示例:

```java
Path rootPath = Paths.get("data/to-delete");
try {
    Files.walkFileTree(rootPath, new SimpleFileVisitor<Path>() {
        @Override
        public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
            System.out.println("delete file: " + file.toString());
            Files.delete(file);
            return FileVisitResult.CONTINUE;
        }

        @Override
        public FileVisitResult postVisitDirectory(Path dir, IOException exc) 
            	throws IOException {
            Files.delete(dir);
            System.out.println("delete dir: " + dir.toString());
            return FileVisitResult.CONTINUE;
        }
    });
} catch(IOException e){
    e.printStackTrace();
}
```
