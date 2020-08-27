## 字节码文件结构

使用`javap-verbose`命令分析一个字节码文件时将会分析该字节码文件的魔数、版本号、常量池、类信息、类的构造方法、类中的方法信息、类变量与成员变量等信息。我们先来编写一个非常简单的类。

```java
public interface MyInterface1 {
}
```

反编译指令：`javap`，参数：`-p`、`-verbose`。


```java
public class MyTest1 implements MyInterface1 {
    private String str = "1000";
    public String getStr() {	return str;		}
    public void setStr(String str) {	this.str = str;	 }
}
```

```class
Classfile /D:/workspace/java/project-workspace/backend-development-summary/codes/target/classes/two/jvm/bytecodes/MyTest1.class
  Last modified 2019-11-26; size 587 bytes
  MD5 checksum a6fe9bd13568b3724178d222e01b213c
  Compiled from "MyTest1.java"
public class two.jvm.bytecodes.MyTest1 implements two.jvm.bytecodes.MyInterface1
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #5.#22         // java/lang/Object."<init>":()V
   #2 = String             #23            // 1000
   #3 = Fieldref           #4.#24         // two/jvm/bytecodes/MyTest1.str:Ljava/lang/String;
   #4 = Class              #25            // two/jvm/bytecodes/MyTest1
   #5 = Class              #26            // java/lang/Object
   #6 = Class              #27            // two/jvm/bytecodes/MyInterface1
   #7 = Utf8               str
   #8 = Utf8               Ljava/lang/String;
   #9 = Utf8               <init>
  #10 = Utf8               ()V
  #11 = Utf8               Code
  #12 = Utf8               LineNumberTable
  #13 = Utf8               LocalVariableTable
  #14 = Utf8               this
  #15 = Utf8               Ltwo/jvm/bytecodes/MyTest1;
  #16 = Utf8               getStr
  #17 = Utf8               ()Ljava/lang/String;
  #18 = Utf8               setStr
  #19 = Utf8               (Ljava/lang/String;)V
  #20 = Utf8               SourceFile
  #21 = Utf8               MyTest1.java
  #22 = NameAndType        #9:#10         // "<init>":()V
  #23 = Utf8               1000
  #24 = NameAndType        #7:#8          // str:Ljava/lang/String;
  #25 = Utf8               two/jvm/bytecodes/MyTest1
  #26 = Utf8               java/lang/Object
  #27 = Utf8               two/jvm/bytecodes/MyInterface1
{
  public two.jvm.bytecodes.MyTest1();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String 1000
         7: putfield      #3                  // Field str:Ljava/lang/String;
        10: return
      LineNumberTable:
        line 3: 0
        line 5: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Ltwo/jvm/bytecodes/MyTest1;

  public java.lang.String getStr();
    descriptor: ()Ljava/lang/String;
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #3                  // Field str:Ljava/lang/String;
         4: areturn
      LineNumberTable:
        line 8: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Ltwo/jvm/bytecodes/MyTest1;

  public void setStr(java.lang.String);
    descriptor: (Ljava/lang/String;)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: aload_1
         2: putfield      #3                  // Field str:Ljava/lang/String;
         5: return
      LineNumberTable:
        line 12: 0
        line 13: 5
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Ltwo/jvm/bytecodes/MyTest1;
            0       6     1   str   Ljava/lang/String;
}
SourceFile: "MyTest1.java"
```

<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/ccc5f00c-1a6d-44a8-adf5-12031c0a6ffe" /></div>
### 字节码文件解析

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/09d5eba7-9f22-44d0-9540-5724a2fa2d73" /></div>
#### **魔数** & 版本号

**魔数**

所有的class字节码文件的前4个字节都是魔数，魔数值为固定值：0xCAFEBABE。翻译为“咖啡宝贝”，emmm，和Java的图标有异曲同工之妙。

**版本号**

魔数之后的4个字节为版本信息，前两个字节表示`minor version（次版本号）`，后两个字节表示`major version（主版本号）`。这里的版本号为00000034，换算成十进制，表示次版本号为0，主版本号为52。所以，该文件的版本号为：1.8.0。可以通过`java-version`命令来查看当前JDK的版本。

```
// JDK版本号与字节码文件版本号对应表
J2SE 8 = 52
J2SE 7 = 51
J2SE 6.0 = 50
J2SE 5.0 = 49
JDK 1.4 = 48
JDK 1.3 = 47
JDK 1.2 = 46
JDK 1.1 = 45
```

#### **常量池（constant pool）**

紧接着主版本号之后的就是常量池入口。一个Java类中定义的很多信息都是由常量池来维护和描述的，可以将常量池看作是class文件的资源仓库，比如说Java类中定义的方法与变量信息，都是存储在常量池中。常量池中主要存储两类常量：字面量与符号引用。字面量如文本字符串，Java中声明为final的常量值等，而符号引用如类和接口的全局限定名，字段的名称和描述符，方法的名称和描述符等。

##### **常量池的总体结构**

Java类所对应的常量池主要由常量池数量与常量池数组（常量表）这两部分共同构成。常量池数量紧跟在主版本号后面，占据2个字节。常量池数组则紧跟在常量池数量之后。值得注意的是：`常量池数组中元素的个数=常量池数-1`，目的是满足某些常量池索引值的数据在特定情况下需要表达不引用任何一个常量池的含义，也就是说索引为0的位置也是一个常量（保留常量），只不过它不位于常量表中，这个常量就对应null值。如我们上面的示例中，001C表示有28个常量，常量池中有27个常量，加上一个null，正好是28个。

常量池数组与一般的数组不同的是，常量池数组中不同的元素的类型、结构都是不同的，长度当然也就不同；但是，每一种元素的第一个数据都是一个u1类型，该字节是个标志位，占据1个字节。JVM在解析常量池时，会根据这个u1类型来获取元素的具体类型。

<div align="center"><img width="95%" src="http://q0l9qvfyx.bkt.clouddn.com/9184d919-94e9-49ce-935d-2c1d9742510d" /></div>
在JVM规范中，每个变量/字段都有描述信息，描述信息主要的作用是描述字段的数据类型、方法的参数列表（包括数量、类型与顺序）与返回值。根据描述符规则，基本数据类型和代表无返回值的void类型都用一个`大写字符`来表示，对象类型则使用`字符L加对象的全限定名称`来表示。目的是压缩字节码文件的体积，如下：`B-byte`、`C-char`、`D-double`、`F-float`、`I-int`、`J-long`、`S-short`、`Z-boolean`、`V-void`、`L-对象类型`，如
`Ljava/lang/String;`。

对于数组类型来说，每一个维度使用一个前置的`[`来表示，如`int[`被记录为`[`，`String[[`被记录为`[Ljava/lang/string`。

**描述方法**

用描述符描述方法时，按照先参数列表，后返回值的顺序来描述。参数列表按照参数的严格顺序放在一组`()`内，如方法 `String getRealNameByIdAndNickNName(int id, String name);`的描述符为：`(I, Ljava/lang/String;)Ljava/lang/String;`。

<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/9bb27ad2-7978-4dbf-ad79-b4832a7c4a31" /></div>
**描述属性**

<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/d0c46382-591d-4099-bf3c-72bb8ed42e40" /></div>
#### 类访问标志&父类&接口

**access flags**

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/515a0b61-cb5a-4026-8bb0-af01a74a54b7" /></div>
access flags中一共有16个标志位可以使用，当前只定义了其中8个，没有使用到的标志位要求一律为0。我们的代码是一个普通Java类，不是接口、枚举或者注解，被public关键字修饰但没有被声明为final和abstract，并且它使用了JDK1.2之后的编译器进行编译，因此它的ACC_PUBLIC、 ACC_SUPER标志应当为真，而ACC_FINAL、ACC_INTERFACE、ACC_ABSTRACT、ACC_SYNTHETIC、ACC_ANNOTATION、ACC_ENUM这6个标志应当为假，因此它的access flags的值应为:0x0001 & 0x0020 = 0x0021。16进制显示也确实如此。

**当前类名 & 父类名**

<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/5982fb78-f848-4628-963a-003dbd3c120e" /></div>
**接口个数 & 接口名**

<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/4ba25311-a2fc-4c8c-ba6a-708323eb7476" /></div>
#### 字段表集合

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/e677e76c-daf2-4dbd-9da7-bbca7ba0a59b" /></div>
**字段访问标志**

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/d4060b33-d570-42f0-aa71-0611fb3d5792" /></div>
<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/ef8b649d-c035-40a2-97e3-7d8e7cf9a405" /></div>
#### 方法表集合

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/52748985-025f-47c0-ba60-6f42c284497c" /></div>
**方法访问标志**

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/64435e52-d4da-4444-8719-3458df27a56a" /></div>
<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/d51a8538-d479-4a09-b1f4-1eb26badea21" /></div>
#### 属性表集合

属性表（attribute_info）在前面的讲解之中已经出现过数次，在Class、字段表方法表都可以携带自己的属性表集合，以用于描述某些场景专有的信息与Class文件中其他的数据项目要求严格的顺序、长度和内容不同，属性表集合的限制稍微宽松了一些。不再要求各个属性表具有严格顺序，并且只要不与已有属性名重复，任何人实现的编译器都可以向属性表中写入自己定义的属性信息。下面是属性表名称及其对应的位置。

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/5e250661-f22f-4405-9955-106157dc7a70" /></div>
**Code属性**

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/832f4c6f-7807-44b8-8829-c63678f1f383" /></div>
<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/44069c3d-6a1c-4c5b-8508-b3ca86c1e7e1" /></div>
**LineNumberTable属性**

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/9a440455-7d7e-4bbf-beaf-ec44b25fc1ea" /></div>
line_number_table是一个数量为 line_number_table_length、类型为 line_number_info的集合，line_number_info表包括了 start_pc 和 line_number两个u2类型的数据项，前者是字节码行号，后者是Java源码行号。

<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/34e0c3c5-359f-4f23-87f1-fd1796fefc56" /></div>
**LocalVariableTable**

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/b348cec4-abbe-49a0-adb5-ea230d55ea27" /></div>
其中，local_variable_info项目代表了一个栈帧与源码中的局部变量的关联。

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/b3d30433-e5e6-4a0a-b32c-53ccf40c17f3" /></div>
start_pc和 length属性分别代表这个局部变量的生命周期开始的字节码偏移量及其作用范围覆盖的长度，两者结合起来就是这个局部变量在字节码之中的作用域范围。name_index和 descriptor_index都是指向常量池中 CONSTANT_Utf8_info型常量的索引，分别代表了局部变量的名称以及这个局部变量的描述符。index是这个局部变量在栈帧局部变量表中slot的位置。当这个变量数据类型是64位类型时（double和long），它占用的Slot为 index 和 index+1两个。

<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/35f73683-b82b-46ef-b0b8-0d19499c9487" /></div>
**ConstantValue属性**

Constant Value属性的作用是通知虚拟机自动为静态变量赋值。只有被 static 关键字修饰的变量（类变量）才可以使用这项属性。类似“int x=123”和“static int=123”这样的变量定义在Java程序中是非常常见的事情，但虚拟机对这两种变量赋值的方式和时刻都有所不同。对手非 static 类型的变量（也就是实例变量）的赋值是在实例构造器`<init>`方法中进行的；而对于类变量则有两种方式可以选择：在类构造器`<clini>`方法中或者使用 Constant Value属性。目前 Sun Java 编译器的选择是：如果同时使用 fina l和 static 来修饰一个变量（按照习惯，这里称“常量”更贴切），并且这个变量的数据类型是基本类型或者 java.lang.String的话，就生成 Constant Value 属性来进行初始化，如果这个变量没有被 final 修饰，或者非基本类型及字符串，则将会选择在`<ckinit>`方法中进行初始化。

虽然有 final 关键字才更符合“Constant Value”的语义，但虚拟机规范中并没有强制要求字段必须设置了 ACC_FINAL标志只要求了有 Constant Value 属性的字段必须设置 ACC_STATIC 标志而已，对 final 关键字的要求是 Javac编译器自己加入的限制。而对 Constant Value 的属性值只能限于基本类型和 String，不过其实这不是什么限制，因为此属性的属性值只是一个常量池的索引号，由于 Class 文件格式的常量类型中只有与基本属性和字符串相对应的字面量，所以就算 Constant Value 属性想支持别的类型也无能为力。
从数据结构中可以看出， Constant Value 属性是一个定长属性，它的 attribute_length 数据项值必须固定为2。 constantvalue_index 数据项代表了常量池中一个字面量常量的引用，根据字段类型的不同，字面量可以是 CONSTANT_Long_info、 CONSTANT_Float_info、CONSTANT_Double_info、 CONSTANT_Integer_info、CONSTANT_ String_info常量中的一种。

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/440d82bb-e622-4911-bcf4-6e76b6a10e94" /></div>
#### SourceFile属性

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/c683dbdc-c969-45d4-98ec-a495587a8b21"/></div>
<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/a8cb668e-54ff-48c1-9ab0-dd6b1aefd1eb" /></div>


## 实例分析

### 静态代码块

<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/6627201a-3244-42cf-bdf4-095c4630e0dd" /></div>
静态属性的初始化和静态代码块的执行都是在JDK自动生成的`cinit`方法中完成的。其实是指令重排序的体现。



### 构造方法

<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/8d998e66-d9c6-41ca-96e3-debc61f7a69a" /></div>
<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/d9acc670-6c4f-4854-ac1b-8f16f3136a4c" /></div>
实例变量的赋值本质上都是在构造方法里进行的。



### 分析异常

```java
public static void test() throws IOException, ArrayIndexOutOfBoundsException {
    try {
        InputStream inputStream = new FileInputStream("test.txt");
        ServerSocket serverSocket = new ServerSocket(9999);
        serverSocket.accept();
    } catch (FileNotFoundException ex) {
        throw ex;
    } catch (NullPointerException ex) {
        throw ex;
    } finally {
        System.out.println("finally");
    }
}

public static void main(String[] args){
    try {
        test();
    } catch (IOException e) {

    }
}
```

```
Classfile /D:/workspace/java/project-workspace/backend-development-summary/codes/target/classes/two/jvm/bytecodes/MyTest4.class
  Last modified 2019-11-28; size 1347 bytes
  MD5 checksum d2e6f4cb6a617878f8c9757b8cc2c6e6
  Compiled from "MyTest4.java"
public class two.jvm.bytecodes.MyTest4
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #16.#45        // java/lang/Object."<init>":()V
   #2 = Class              #46            // java/io/FileInputStream
   #3 = String             #47            // test.txt
   #4 = Methodref          #2.#48         // java/io/FileInputStream."<init>":(Ljava/lang/String;)V
   #5 = Class              #49            // java/net/ServerSocket
   #6 = Methodref          #5.#50         // java/net/ServerSocket."<init>":(I)V
   #7 = Methodref          #5.#51         // java/net/ServerSocket.accept:()Ljava/net/Socket;
   #8 = Fieldref           #52.#53        // java/lang/System.out:Ljava/io/PrintStream;
   #9 = String             #54            // finally
  #10 = Methodref          #55.#56        // java/io/PrintStream.println:(Ljava/lang/String;)V
  #11 = Class              #57            // java/io/FileNotFoundException
  #12 = Class              #58            // java/lang/NullPointerException
  #13 = Methodref          #15.#59        // two/jvm/bytecodes/MyTest4.test:()V
  #14 = Class              #60            // java/io/IOException
  #15 = Class              #61            // two/jvm/bytecodes/MyTest4
  #16 = Class              #62            // java/lang/Object
  #17 = Utf8               <init>
  #18 = Utf8               ()V
  #19 = Utf8               Code
  #20 = Utf8               LineNumberTable
  #21 = Utf8               LocalVariableTable
  #22 = Utf8               this
  #23 = Utf8               Ltwo/jvm/bytecodes/MyTest4;
  #24 = Utf8               test
  #25 = Utf8               inputStream
  #26 = Utf8               Ljava/io/InputStream;
  #27 = Utf8               serverSocket
  #28 = Utf8               Ljava/net/ServerSocket;
  #29 = Utf8               ex
  #30 = Utf8               Ljava/io/FileNotFoundException;
  #31 = Utf8               Ljava/lang/NullPointerException;
  #32 = Utf8               StackMapTable
  #33 = Class              #57            // java/io/FileNotFoundException
  #34 = Class              #58            // java/lang/NullPointerException
  #35 = Class              #63            // java/lang/Throwable
  #36 = Utf8               Exceptions
  #37 = Class              #64            // java/lang/ArrayIndexOutOfBoundsException
  #38 = Utf8               main
  #39 = Utf8               ([Ljava/lang/String;)V
  #40 = Utf8               args
  #41 = Utf8               [Ljava/lang/String;
  #42 = Class              #60            // java/io/IOException
  #43 = Utf8               SourceFile
  #44 = Utf8               MyTest4.java
  #45 = NameAndType        #17:#18        // "<init>":()V
  #46 = Utf8               java/io/FileInputStream
  #47 = Utf8               test.txt
  #48 = NameAndType        #17:#65        // "<init>":(Ljava/lang/String;)V
  #49 = Utf8               java/net/ServerSocket
  #50 = NameAndType        #17:#66        // "<init>":(I)V
  #51 = NameAndType        #67:#68        // accept:()Ljava/net/Socket;
  #52 = Class              #69            // java/lang/System
  #53 = NameAndType        #70:#71        // out:Ljava/io/PrintStream;
  #54 = Utf8               finally
  #55 = Class              #72            // java/io/PrintStream
  #56 = NameAndType        #73:#65        // println:(Ljava/lang/String;)V
  #57 = Utf8               java/io/FileNotFoundException
  #58 = Utf8               java/lang/NullPointerException
  #59 = NameAndType        #24:#18        // test:()V
  #60 = Utf8               java/io/IOException
  #61 = Utf8               two/jvm/bytecodes/MyTest4
  #62 = Utf8               java/lang/Object
  #63 = Utf8               java/lang/Throwable
  #64 = Utf8               java/lang/ArrayIndexOutOfBoundsException
  #65 = Utf8               (Ljava/lang/String;)V
  #66 = Utf8               (I)V
  #67 = Utf8               accept
  #68 = Utf8               ()Ljava/net/Socket;
  #69 = Utf8               java/lang/System
  #70 = Utf8               out
  #71 = Utf8               Ljava/io/PrintStream;
  #72 = Utf8               java/io/PrintStream
  #73 = Utf8               println
{
  public two.jvm.bytecodes.MyTest4();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 9: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Ltwo/jvm/bytecodes/MyTest4;

  public static void test() throws java.io.IOException, java.lang.ArrayIndexOutOfBoundsException;
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=3, args_size=0
         0: new           #2                  // class java/io/FileInputStream
         3: dup
         4: ldc           #3                  // String test.txt
         6: invokespecial #4                  // Method java/io/FileInputStream."<init>":(Ljava/lang/String;)V
         9: astore_0
        10: new           #5                  // class java/net/ServerSocket
        13: dup
        14: sipush        9999
        17: invokespecial #6                  // Method java/net/ServerSocket."<init>":(I)V
        20: astore_1
        21: aload_1
        22: invokevirtual #7                  // Method java/net/ServerSocket.accept:()Ljava/net/Socket;
        25: pop
        26: getstatic     #8                  // Field java/lang/System.out:Ljava/io/PrintStream;
        29: ldc           #9                  // String finally
        31: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        34: goto          54
        37: astore_0
        38: aload_0
        39: athrow
        40: astore_0
        41: aload_0
        42: athrow
        43: astore_2
        44: getstatic     #8                  // Field java/lang/System.out:Ljava/io/PrintStream;
        47: ldc           #9                  // String finally
        49: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        52: aload_2
        53: athrow
        54: return
      Exception table:
         from    to  target type
             0    26    37   Class java/io/FileNotFoundException
             0    26    40   Class java/lang/NullPointerException
             0    26    43   any
            37    44    43   any
      LineNumberTable:
        line 13: 0
        line 14: 10
        line 15: 21
        line 21: 26
        line 22: 34
        line 16: 37
        line 17: 38
        line 18: 40
        line 19: 41
        line 21: 43
        line 22: 52
        line 23: 54
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
           10      16     0 inputStream   Ljava/io/InputStream;
           21       5     1 serverSocket   Ljava/net/ServerSocket;
           38       2     0    ex   Ljava/io/FileNotFoundException;
           41       2     0    ex   Ljava/lang/NullPointerException;
      StackMapTable: number_of_entries = 4
        frame_type = 101 /* same_locals_1_stack_item */
          stack = [ class java/io/FileNotFoundException ]
        frame_type = 66 /* same_locals_1_stack_item */
          stack = [ class java/lang/NullPointerException ]
        frame_type = 66 /* same_locals_1_stack_item */
          stack = [ class java/lang/Throwable ]
        frame_type = 10 /* same */
    Exceptions:
      throws java.io.IOException, java.lang.ArrayIndexOutOfBoundsException

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=1, locals=2, args_size=1
         0: invokestatic  #13                 // Method test:()V
         3: goto          7
         6: astore_1
         7: return
      Exception table:
         from    to  target type
             0     3     6   Class java/io/IOException
      LineNumberTable:
        line 27: 0
        line 30: 3
        line 28: 6
        line 31: 7
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       8     0  args   [Ljava/lang/String;
      StackMapTable: number_of_entries = 2
        frame_type = 70 /* same_locals_1_stack_item */
          stack = [ class java/io/IOException ]
        frame_type = 0 /* same */
}
SourceFile: "MyTest4.java"
```

从字节码的角度是不区分受查异常和非受查异常的，但是区分try-catch异常处理和throws的异常处理。而对于`throw new Exception()`语法来说，仅仅是抛出了一个异常，需要调用者接收。

**Exceptions属性**

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/fecffe8f-0131-4258-a2e9-8d56b8cc91ba" /></div>

Exceptions属性是在方法表中与Code属性平级的一项属性，作用是列举出方法中可能抛出的受查异常（Checked Excepitons），也就是方法描述时在 throws 关键字后面列举的异常。结构如上表。 

**exception_index_table**

Exceptions属性中的 number_of_exceptions项表示方法可能抛出 number_of_exceptions 种受查异常，每一种受查异常使用一个exception_index_table项表示，exception_index_table是一个指向常量池中 CONSTANT_Class_info 型常量的索引，代表了该受查异常的类型。

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/3bc00f7f-2c11-4150-a52b-5c3494833723" /></div>

**exception_table**

exception_table是存在Code属性中的，也是try_catch在字节码层次的体现。

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/e572d50e-f212-4ab5-8587-812cfa6be59a" /></div>

如果当字节码在第 start_pc 行到第 end_pc 行之间（不含第 end 行出现了类型为 catch_type或者其子类的异常（catch_type）为指向一个 CONSTANF_Classinfo 型常量的索引，则转到第 handler_pc 行继续处理。当 catch_type的值为0时，代表任意异常情况都需要转向到 handler_pc 处进行处理。

使用jclasslib分析上面的异常：

```
 0 new #2 <java/io/FileInputStream>
 3 dup
 4 ldc #3 <test.txt>
 6 invokespecial #4 <java/io/FileInputStream.<init>>
 9 astore_0
10 new #5 <java/net/ServerSocket>
13 dup
14 sipush 9999
17 invokespecial #6 <java/net/ServerSocket.<init>>
20 astore_1
21 aload_1
22 invokevirtual #7 <java/net/ServerSocket.accept>
25 pop
26 getstatic #8 <java/lang/System.out>
29 ldc #9 <finally>
31 invokevirtual #10 <java/io/PrintStream.println>
34 goto 54 (+20)
37 astore_0
38 aload_0
39 athrow
40 astore_0
41 aload_0
42 athrow
43 astore_2
44 getstatic #8 <java/lang/System.out>
47 ldc #9 <finally>
49 invokevirtual #10 <java/io/PrintStream.println>
52 aload_2
53 athrow
54 return
```

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/237f2320-1c29-4304-b319-dfc689047f15" /></div>

从上面反编译的助记符可以知道，finally的处理是在每个 catch 后面复制一份finally里面的指令来完成的。



### synchronized

```java
void test(){
    synchronized (o){
        System.out.println(o);
    }
}
```

```java
void test();
    descriptor: ()V
    flags:
    Code:
      stack=2, locals=3, args_size=1
         0: aload_0
         1: getfield      #3     // Field o:Ljava/lang/Object;
         4: dup
         5: astore_1
         6: monitorenter			// 加锁
         7: getstatic     #4     // Field java/lang/System.out:Ljava/io/PrintStream;
        10: aload_0
        11: getfield      #3     // Field o:Ljava/lang/Object;
        14: invokevirtual #5     // Method java/io/PrintStream.println:(Ljava/lang/Object;)V
        17: aload_1
        18: monitorexit				// 正常解锁
        19: goto          27		// 正常解锁之后return
        22: astore_2
        23: aload_1
        24: monitorexit				// 保证在athrow之前锁被释放
        25: aload_2
        26: athrow
        27: return
      Exception table:
         from    to  target type
             7    19    22   any
            22    25    22   any
      LineNumberTable:
        line 16: 0
        line 17: 7
        line 18: 17
        line 19: 27
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      28     0  this   Ltwo/jvm/bytecodes/MyTest2;
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 22
          locals = [ class two/jvm/bytecodes/MyTest2, class java/lang/Object ]
          stack = [ class java/lang/Throwable ]
        frame_type = 250 /* chop */
          offset_delta = 4
```

**异常时退出的处理**

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/ca28d1cf-c96f-4ec5-960d-91582a5561e1" /></div>
如果[7, 19)行的任何异常转到22行，[22, 25)行的任何异常转到22行。

**monitorenter**

The *objectref* must be of type `reference`.

Each object is associated with a monitor. A monitor is locked if and only if it has an owner. The thread that executes *monitorenter* attempts to gain ownership of the monitor associated with *objectref*, as follows:

- If the entry count of the monitor associated with *objectref* is zero, the thread enters the monitor and sets its entry count to one. The thread is then the owner of the monitor.
- If the thread already owns the monitor associated with *objectref*, it reenters the monitor, incrementing its entry count.
- If another thread already owns the monitor associated with *objectref*, the thread blocks until the monitor's entry count is zero, then tries again to gain ownership.

Run-time Exceptions

-  If *objectref*  is `null`, monitorenter throws a `NullPointerException`**.** 

**monitorexit** 

- The *objectref* must be of type `reference`.
- The thread that executes *monitorexit* must be the owner of the monitor associated with the instance referenced by *objectref*.
- The thread decrements the entry count of the monitor associated with *objectref*. If as a result the value of the entry count is zero, the thread exits the monitor and is no longer its owner. Other threads that are blocking to enter the monitor are allowed to attempt to do so.

Run-time Exceptions

- If *objectref* is `null`, *monitorexit* throws a `NullPointerException`.
- Otherwise, if the thread that executes *monitorexit* is not the owner of the monitor associated with the instance referenced by *objectref*, *monitorexit* throws an `IllegalMonitorStateException`.
- Otherwise, if the Java Virtual Machine implementation enforces the rules on structured locking described in [§2.11.10](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.10) and if the second of those rules is violated by the execution of this *monitorexit* instruction, then *monitorexit* throws an `IllegalMonitorStateException`.



### 分析this

<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/254a4352-87c0-483f-8221-0aa44e45520f" /></div>
对于Java类中的每一个实例方法（非static方法），其在编译后所生成的字节码当中，方法参数的数量总是会比源代码中方法参数的数量多一个，即参数this，它位于方法的第一个参数位置处；这样，我们就可以在Java的实例方法中使用this来去访问当前对象的属性以及其他方法。

这个操作是在编译期间完成的，即由 javac 编译器在编译的时候将对this的访问转化为对一个普通实例方法参数的访问;接下来在运行期间。由JVM在调用实例方法时，自动向实例方法传入该this。所以在实例方法的局部变量表中，至少会有一个指向当前对象的局部变量。

**上图的4个局部变量**

- this
- inputStream
- serverSocket
- 第四个局部变量是两个catch中的某一个异常。因为两个异常最多只能有一个生效，所以局部变量最多是4个。