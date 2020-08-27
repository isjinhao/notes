## 类型的生命周期

### 生命周期

- 加载：查找并加载类的二进制数据到JVM中（从外存到内存的过程）
- 连接：
  - 验证 : 确保被加载的类的正确性
  - 准备：为类的**静态变量**分配内存，并将其**初始化为默认值**，但是到达初始化之前类变量都没有初始化为真正的初始值
  - 解析：把类中的符号引用转换为直接引用，就是在类型的常量池中寻找类、接口、字段和方法的符号引用，把这些符号引用替换成直接引用的过程
- 初始化：为类的静态变量赋予正确的初始值



### 初始化

 Java程序对类的使用方式可分为**主动使用**和**被动使用**两种。当且仅当主动使用时才会触发初始化过程。

#### 主动使用

- 创建类的实例（助记符：`new`）
- 访问某个类或接口的静态变量，或者对该静态变量赋值（助记符：`getstatic`、`putstatic`）
- 调用类的静态方法（助记符：`invokestatic`）
- 反射（`Class.forName(xxx)`）
- 初始化一个类的子类时其父类需要现行初始化
- Java虚拟机启动时被标明启动类的类
- JDK1.7开始提供的动态语言支持（了解）

#### 被动使用

除了上面七种情况外，其他使用Java类的方式都被看做是对类的被动使用，都不会导致类的初始化。比如：

```java
Thread.currentThread().getContextClassLoader().loadClass("xxx");	// 不会初始化
```


```java
/**
        对于静态字段来说，只有直接定义了该字段的类才会被初始化
        当一个类在初始化时，要求父类全部都已经初始化完毕
        -XX:+TraceClassLoading，用于追踪类的加载信息并打印出来

        -XX:+<option>，表示开启option选项
        -XX:-<option>，表示关闭option选项
        -XX:<option>=<value>，表示将option的值设置为value
*/
public class MyTest{
    public static void main(String[] args){
        System.out.println(MyChild.str);    
        // 输出：MyParent static block、 hello world   （因为对MyChild不是主动使用）
        System.out.println(MyChild.str2);   
        // 输出：MyParent static block、MyChild static block、welcome
    }
}
class MyParent{
    public static String str = "hello world";
    static {
        System.out.println("MyParent static block");
    }
}
class MyChild extends MyParent{
    public static String str2 = "welcome";
    static {
        System.out.println("MyChild static block");
    }
}
```

#### final

```java
/*
        常量在编译阶段会存入到调用这个常量的方法所在的类的常量池中。所以本质上，调用类并没有直接调用到定义常量的
        类，因此并不会触发定义常量的类的初始化。注意：这里指的是将常量存到MyTest2的常量池中，之后MyTest2和
        MyParent就没有任何关系了。甚至我们可以将MyParent2的class文件删除

        助记符 ldc：        表示将int、float或者String类型的常量值从常量池中推送至栈顶
        助记符 bipush：     表示将单字节[128-127]的常量值推送到栈顶，byte int push
        助记符 sipush：     表示将一个短整型值[-32768-32369]推送至栈顶，short int push
        助记符 iconst_1：   表示将int型的1推送至栈顶（iconst_m1到iconst_5）
*/
public class MyTest2 {
    public static void main(String[] args) {
        System.out.println(MyParent2.str);    //输出 hello world
        System.out.println(MyParent2.s);
        System.out.println(MyParent2.i);
        System.out.println(MyParent2.j);
    }
}

class MyParent2 {
    public static final String str = "hello world";
    public static final short s = 7;
    public static final int i = 129;
    public static final int j = 1;

    static {
        System.out.println("MyParent static block");
    }
}
```

```java
/*
 * 当一个常量的值并非编译期间可以确定的，那么其值就不会放到调用类的常量池中
 * 这时在程序运行时，会导致主动使用这个常量所在的类，显然会导致这个类被初始化
 */
public class MyTest3 {
    public static void main(String[] args) {
        System.out.println(MyParent3.str);  
        // 输出MyParent static block、kjqhdun-baoje21w-jxqioj1-2jwejc9029
    }
}

class MyParent3 {
    public static final String str = UUID.randomUUID().toString();

    static {
        System.out.println("MyParent static block");
    }
}
```

#### 接口

```java
/*
	使用接口中的变量的时候不需要初始化其父接口
*/
public class ClassLoadTest5 {
    public static void main(String[] args) {
        System.out.println(MyChild2.thread);
        // thread 初始化了
		// Thread[Thread-0,5,main]
    }
}

interface Student6 {
    Thread thread = new Thread() {
        {
            System.out.println("thread 初始化了");  //如果父接口初始化了这句应该输出
        }
    };
}

interface MyChild2 extends Student6 {
    // thread2是final变量，但是不会在编译期被放在ClassLoadTest5的常量池中
    Thread thread2 = new Thread() {
        {
            System.out.println("thread2 初始化了");
        }
    };
}
```

#### 类+接口

```java
// 类初始化的时候不需要其接口完成初始化
public class MyTest8 {
    public static void main(String[] args) {
        System.out.println(MyChild8.t2);
    }
}

interface MyGrandpa {
    Thread t = new Thread() {
        {
            System.out.println("MyGrandpa init");
        }
    };
}

interface MyParent1 extends MyGrandpa {
    Thread t1 = new Thread() {
        {
            System.out.println("MyParent1 init");
        }
    };
}

class MyChild8 implements MyParent1{
    public static Thread t2 = new Thread() {
        {
            System.out.println("MyChild8 init");
        }
    };
}
// MyChild8 init
// Thread[Thread-0,5,main]
```

```java
public class MyTest8 {
    public static void main(String[] args) {
        System.out.println(MyChild8.t2);
    }
}

interface MyGrandpa {
    Runnable t = new Thread() {
        {
            System.out.println("MyGrandpa init");
        }
    };
}

class MyParent1 implements MyGrandpa {
    public static Runnable t1 = new Thread() {
        {
            System.out.println("MyParent1 init");
        }
    };
}

class MyChild8 extends MyParent1 {
    public static Runnable t2 = new Thread() {
        {
            System.out.println("MyChild8 init");
        }
    };
}
/*
	MyParent1 init
    MyChild8 init
    Thread[Thread-1,5,main]
*/
```

#### 准备阶段和初始化的顺序

```java
/*
 * 准备阶段会把静态变量附上默认值
 * 初始化的时候按从上到下的顺序把变量附上初始值
 */
public class MyTest6 {
    public static void main(String[] args) {
        Singleton singleton = Singleton.getInstance();
        System.out.println(Singleton.counter1);     // 1
        System.out.println(Singleton.counter2);     // 0
    }
}

class Singleton {
    public static int counter1;
    private static Singleton singleton = new Singleton();

    private Singleton() {
        counter1++;
        counter2++;
    }

    public static int counter2 = 0;
    public static Singleton getInstance() {
        return singleton;
    }
}
```

#### 数组

```java
/*
 * 对于数组实例来说，其类型是由JVM在运行期动态生成的，表示为 `[L数组元素全限定名` 这种形式。
 * 对于数组来说，JavaDoc将构成数据的元素称为Component，实际上是将数组降低一个维度后的类型。
 *
 * 助记符：anewarray：表示创建一个引用类型（如类、接口）的数组，并将其引用值压入栈顶
 * 助记符：newarray：表示创建一个指定原始类型（int boolean float double）d的数组，并将其引用值压入栈顶
 */
public class MyTest4 {
    public static void main(String[] args) {
//        MyParent4 myParent4 = new MyParent4();        // 创建类的实例，属于主动使用，会导致类的初始化
        MyParent4[] myParent4s = new MyParent4[1];      // 不是主动使用，不会导致类的初始化
        System.out.println(myParent4s.getClass());   // 输出 [[Ltwo.jvm.classloader.MyParent4;
        System.out.println(myParent4s.getClass().getSuperclass());    // 输出Object

        int[] i = new int[1];
        System.out.println(i.getClass());          // 输出 [I
        System.out.println(i.getClass().getSuperclass());    // 输出Object

        // 实际上数组类型是运行期动态加载的，也就是说数组类型是JVM在运行过程中创建的
    }
}

class MyParent4 {
    static {
        System.out.println("MyParent static block");
    }
}
```

#### 类的实例化过程

- 为新的对象分配内存
- 为实例变量赋默认值
- 为实例变量赋正确的初始值
- 执行构造方法

```java
public class MyTest7 {
    public MyTest7() {    }

    public int i = 10;
    {
        i = 100;
    }	// 这种代码块会在每次创建一个实例的时候都执行一次

    public static void main(String[] args) {
        System.out.println(new MyTest7().i); // 100
    }
}
```



### 生命周期的几个时机

**类加载**

类加载器并不需要等到某个类被“首次主动使用”时再加载它。JVM规范允许类加载器在预料某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到了.class文件缺失或存在错误，类加载器不会立即报告错误，而是在程序首次主动使用该类才报告错误（LinkageError错误），如果这个类没有被程序主动使用，那么类加载器就不会报告错误。在使用loadClass的时候只进行类加载过程，不会进行连接。

**连接**

连接的前两个过程笔者没有找到进行的依据，但是第三个过程解析的触发条件是遇到如下字节码：anewarray、checkcast、getfiled、getstatic、instanceof、invokedynamic、invokeinterface、invokespecial、invokestatic、invokevirtual、ldc、ldc_c、multianewarray、new、putfield、putstatic。

**初始化时机**

主动使用

**生命周期的顺序**

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/c0e21dca-6e8a-4abc-ad89-ce78d41e5c3f" /></div>
规范规定，加载、验证、准备、初始化、卸载这五个阶段的是要按顺序进行，但是解析阶段是遇到如上字节码指令才会进行。比如解析可能再初始化之后。



## 类加载器

- Java虚拟机自带的加载器
  - 根类加载器（Bootstrap）：该加载器没有父加载器，它负责加载虚拟机中的核心类库。根类加载器从系统属性`sun.boot.class.path`所指定的目录中加载类库。类加载器的实现依赖于底层操作系统，属于虚拟机的实现的一部分，它并没有继承`java.lang.ClassLoader`类。
  - 扩展类加载器（Extension）：它的父加载器为根类加载器。它从`java.ext.dirs`系统属性所指定的目录中加载类库，或者从JDK的安装目录的`jre\lib\ext`子目录（扩展目录）下加载类库，如果把用户创建的jar文件放在这个目录下，也会自动由扩展类加载器加载，扩展类加载器是纯`java`类，是`java.lang.ClassLoader`的子类。
  - 系统应用类加载器（System）：也称为应用类加载器，它的父加载器为扩展类加载器，它从环境变量`classpath`或者系统属性`java.class.path`所指定的目录中加载类，他是用户自定义的类加载器的默认父加载器。系统类加载器时纯Java类，是`java.lang.ClassLoader`的子类。
- 用户自定义的类加载器
  - `java.lang.ClassLoader`的子类
  - 用户可以定制类的加载方式

根类加载器–>扩展类加载器–>系统应用类加载器–>自定义类加载器

**路径**

```java
System.out.println(System.getProperty("sun.boot.class.path"));  // 根加载器路径
System.out.println(System.getProperty("java.ext.dirs"));        // 扩展类加载器路径
System.out.println(System.getProperty("java.class.path"));      // 应用类加载器路径
```



### ClassLoader

> A class loader is an object that is responsible for loading classes. The class ClassLoader is an abstract class. Given the binary name of a class, a class loader should attempt to locate or generate data that constitutes a definition for the class. A typical strategy is to transform the name into a file name and then read a "class file" of that name from a file system.
> Every Class object contains a reference to the ClassLoader that defined it.
> **Class objects for array classes are not created by class loaders, but are created automatically as required by the Java runtime.** The class loader for an array class, as returned by Class.getClassLoader() is the same as the class loader for its element type; if the element type is a primitive type, then the array class has no class loader.
> Applications implement subclasses of ClassLoader in order to extend the manner in which the Java virtual machine dynamically loads classes.
> Class loaders may typically be used by security managers to indicate security domains.
> The ClassLoader class uses a delegation model to search for classes and resources. Each instance of ClassLoader has an associated parent class loader. When requested to find a class or resource, a ClassLoader instance will delegate the search for the class or resource to its parent class loader before attempting to find the class or resource itself. The virtual machine's built-in class loader, called the "bootstrap class loader", does not itself have a parent but may serve as the parent of a ClassLoader instance. 
> Class loaders that support concurrent loading of classes are known as parallel capable class loaders and are required to register themselves at their class initialization time by invoking the ClassLoader.registerAsParallelCapable method. Note that the ClassLoader class is registered as parallel capable by default. However, its subclasses still need to register themselves if they are parallel capable. In environments in which the delegation model is not strictly hierarchical, class loaders need to be parallel capable, otherwise class loading can lead to deadlocks because the loader lock is held for the duration of the class loading process (see loadClass methods).
> Normally, the Java virtual machine loads classes from the local file system in a platform-dependent manner. For example, on UNIX systems, the virtual machine loads classes from the directory defined by the CLASSPATH environment variable.
> However, some classes may not originate from a file; they may originate from other sources, such as the network, or they could be constructed by an application. The method defineClass converts an array of bytes into an instance of class Class. Instances of this newly defined class can be created using Class.newInstance.
> The methods and constructors of objects created by a class loader may reference other classes. To determine the class(es) referred to, the Java virtual machine invokes the loadClass method of the class loader that originally created the class.
> For example, an application could create a network class loader to download class files from a server. Sample code might look like:

```java
ClassLoader loader = new NetworkClassLoader(host, port);
Object main = loader.loadClass("Main", true).newInstance();
// . . .
```

The network class loader subclass must define the methods findClass and loadClassData to load a class from the network. Once it has downloaded the bytes that make up the class, it should use the method defineClass to create a class instance. A sample implementation is:

```java
class NetworkClassLoader extends ClassLoader {
    String host;
    int port;
    public Class findClass(String name) {
        byte[] b = loadClassData(name);
        return defineClass(name, b, 0, b.length);
    }

    private byte[] loadClassData(String name) {
        // load the class data from the connection
        // . . .
    }
}
```

**Binary names**
Any class name provided as a String parameter to methods in ClassLoader must be a binary name as defined by The Java™ Language Specification. Examples of valid class names include:

```java
"java.lang.String"
"javax.swing.JSpinner$DefaultEditor"
"java.security.KeyStore$Builder$FileBuilder$1"
"java.net.URLClassLoader$3$1"
```

#### 数组

```java
public static void main(String[] args) {

    String[] strings = new String[2];
    System.out.println(strings.getClass().getClassLoader());
    // 输出null，因为String是BootStrap类加载器加载的

    int[] ints = new int[2];
    System.out.println(ints.getClass().getClassLoader());
    // 输出null，因为int是原生类型，没有类加载器

}
```



#### 双亲委派机制

如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行，如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器，如果父类加载器可以完成类加载任务，就加载后返回，否则交给子类加载器完成。（扩展类加载器只会加载jar包）

<div align="center"><img width="70%" src="http://q0l9qvfyx.bkt.clouddn.com/8dfa5630-d565-410f-b279-3edbb89666b7" /></div>
若有一个类能够成功加载Test类，那么这个类加载器被称为**定义类加载器**，所有能成功返回Class对象引用的类加载器（包括定义类加载器）称为**初始类加载器**。 



### 自定义类加载器

#### loadClass

```java
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException{
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        Class<?> c = findLoadedClass(name);
        // 如果没有被记载
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                // 有爹就找他爹去加载
                if (parent != null) {
                    c = parent.loadClass(name, false);
                // 没爹就找BootStrap加载器去加载
                } else {
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }

            // 父亲不能加载的情况下自己加载
            if (c == null) {
                // If still not found, then invoke findClass in order
                // to find the class.
                long t1 = System.nanoTime();
                
                // 一个满足双亲委派原则的自定义类加载器需要覆盖此方法
                c = findClass(name);

                // this is the defining class loader; record the stats
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}
```

#### findClass

> Finds the class with the specified binary name. This method should be overridden by class loader implementations that follow the delegation model for loading classes, and will be invoked by the loadClass method after checking the parent class loader for the requested class. The default implementation throws a ClassNotFoundException.

这个方法是上面第29行被调用的方法。如果想保持双亲委派机制（实际推荐），在自定义类加载器的时候不去覆盖loadClass，去覆盖findClass就可以了。

#### 自定义一个类加载器

```java
/**
 * 创建自定义加载器，继承ClassLoader
 */
public class MyClassLoader extends ClassLoader {
    private String classLoaderName;
    private String path;
    private final String fileExtension = ".class";

    public MyClassLoader(String classLoaderName) {
        super();        // 将系统类当做该类的父加载器
        this.classLoaderName = classLoaderName;
    }
    public void setPath(String path) {
        this.path = path;
    }

    @Override
    public Class<?> findClass(String className) {
        System.out.println("className=" + className);
        byte[] data = loadClassData(className);
        return defineClass(className, data, 0, data.length);  // define方法为父类方法
    }

    private byte[] loadClassData(String name) {
        InputStream is = null;
        byte[] data = null;
        ByteArrayOutputStream baos = null;
        try {
            File file = new File(this.path +  name.replace(".", "\\") + this.fileExtension);
            System.out.println(file.getAbsolutePath());
            is = new FileInputStream(file);
            baos = new ByteArrayOutputStream();
            int ch;
            while (-1 != (ch = is.read())) {
                baos.write(ch);
            }
            data = baos.toByteArray();

        } catch (Exception e) {
        } finally {
            try {
                is.close();
                baos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            return data;
        }
    }

    public static void test(ClassLoader classLoader) throws Exception {
        Class<?> clazz = classLoader.loadClass("two.jvm.classloader.MyTest");
        //loadClass是父类方法，在方法内部调用findClass
        System.out.println(clazz.hashCode());
        Object object = clazz.newInstance();
        System.out.println(object);
    }

    public static void main(String[] args) throws Exception {
        // 父亲是系统类加载器，根据父类委托机制，MyTest1由AppClassLoader加载了
//        MyClassLoader loader1 = new MyClassLoader("loader1");
//        test(loader1);

        // 仍然是系统类加载器进行加载的，因为路径正好是classpath
//        MyClassLoader loader2 = new MyClassLoader("loader2");
//        loader2.path = "D:\\workspace\\java\\project-workspace\\backend-development-summary\\backend-development-summary\\target\\classes\\";
//        test(loader2);

        // 自定义的类加载器被执行，findClass方法下的输出被打印。前提是当前classpath下不存在MyTest1.class，MyTest16的父加载器-系统类加载器会尝试从classpath中寻找MyTest1。
        MyClassLoader loader3 = new MyClassLoader("loader3");
        loader3.path = "C:\\Users\\ISJINHAO\\Desktop\\";
        test(loader3);

        // 与3同时存在，输出两个class的hash不同，findClass方法下的输出均被打印，因为类加载器的命名空间不同
        MyClassLoader loader4 = new MyClassLoader("loader4");
        loader4.path = "C:\\Users\\ISJINHAO\\Desktop\\";
        test(loader4);

        // 将loader3作为父加载器，此次输出的字节码和loader3的一致
        MyClassLoader loader5 = new MyClassLoader(loader3, "loader3");
        loader3.path = "C:\\Users\\ISJINHAO\\Desktop\\";
        test(loader5);
    }
}

/*
    className=two.jvm.classloader.MyTest
    C:\Users\ISJINHAO\Desktop\two\jvm\classloader\MyTest.class
    1554874502
    two.jvm.classloader.MyTest@6e0be858
    className=two.jvm.classloader.MyTest
    C:\Users\ISJINHAO\Desktop\two\jvm\classloader\MyTest.class
    1360875712
    two.jvm.classloader.MyTest@60e53b93
    1554874502
    two.jvm.classloader.MyTest@5e2de80c
*/
```

**命名空间**

- 每个类加载器都有自己的命名空间，命名空间由该加载器及所有父加载器所加载的类构成；
- 在同一个命名空间中，不会出现类的完整名字（包括类的包名）相同的两个类；
- 在不同的命名空间中，有可能会出现类的完整名字（包括类的包名）相同的两个类；
- 同一命名空间内的类是互相可见的，非同一命名空间内的类是不可见的；
- 子加载器可以见到父加载器加载的类，父加载器也能见到子加载器加载的类。



### 类的卸载

- 当一个类被加载、连接和初始化之后，它的生命周期就开始了。当此类的Class对象不再被引用，即不可触及时，Class对象就会结束生命周期，类在方法区内的数据也会被卸载。
- 一个类何时结束生命周期，取决于代表它的Class对象何时结束生命周期。
- 由Java虚拟机自带的类加载器所加载的类，在虚拟机的生命周期中，始终不会被卸载。Java虚拟机本身会始终引用这些加载器，而这些类加载器则会始终引用他们所加载的类的Class对象，因此这些Class对象是可触及的。
- 由用户自定义的类加载器所加载的类是可以被卸载的。

```java
/**
    自定义类加载器加载类的卸载
    -XX:+TraceClassUnloading
    
    jvisualvm 可以查看Java进程的信息
*/
public static void main(String[] args) throws Exception {
    MyClassLoader loader2 = new MyClassLoader("loader2");
    loader2.path = "C:\\Users\\ISJINHAO\\Desktop\\";
    test(loader2);
    loader2 = null;
    System.gc();   // 让系统去显式执行垃圾回收

    // 输出的两个对象hashcode值不同，因为前面加载的已经被卸载了
    loader2 = new MyClassLoader("loader6");
    loader2.path = "C:\\Users\\ISJINHAO\\Desktop\\";
    test(loader2);
}

/*
    className=two.jvm.classloader.MyTest
    C:\Users\ISJINHAO\Desktop\two\jvm\classloader\MyTest.class
    1554874502
    two.jvm.classloader.MyTest@6e0be858
    [Unloading class two.jvm.classloader.MyTest 0x0000000100061028]
    className=two.jvm.classloader.MyTest
    C:\Users\ISJINHAO\Desktop\two\jvm\classloader\MyTest.class
    1360875712
    two.jvm.classloader.MyTest@60e53b93
    [Unloading class two.jvm.classloader.MyTest 0x0000000100061028]
*/
```



### 命名空间解析

**定于两个辅助类**

```java
public class MySample {
    public MySample() {
        System.out.println("MySample is loaded... " + this.getClass().getClassLoader());
        new MyCat();
        System.out.println("from MySample: " + MyCat.class);
    }
}
```

```java
public class MyCat {
    public MyCat() {
        System.out.println("MyCat is loaded... " + this.getClass().getClassLoader());
//        System.out.println("from MyCat: " + MySample.class);
    }
}
```

**操作1：**

1. 外部文件有MySample和MyCat字节码
2. classpath下只有MySample字节码

```java
/**
 * 创建自定义加载器，继承ClassLoader
 */
public class MyClassLoader extends ClassLoader {
    private String classLoaderName;
    private String path;
    private final String fileExtension = ".class";

    public MyClassLoader(String classLoaderName) {
        super();        //将系统类当做该类的父加载器
        this.classLoaderName = classLoaderName;
    }
    public void setPath(String path) {
        this.path = path;
    }

    @Override
    public Class<?> findClass(String className) {
        System.out.println("className=" + className);
        byte[] data = loadClassData(className);
        return defineClass(className, data, 0, data.length); //define方法为父类方法
    }

    private byte[] loadClassData(String name) {
        InputStream is = null;
        byte[] data = null;
        ByteArrayOutputStream baos = null;
        try {
            File file = new File(this.path + name.replace(".", "\\") + this.fileExtension);
            System.out.println(file.getAbsolutePath());
            is = new FileInputStream(file);
            baos = new ByteArrayOutputStream();
            int ch;

            while (-1 != (ch = is.read())) {
                baos.write(ch);
            }
            data = baos.toByteArray();

        } catch (Exception e) {
        } finally {
            try {
                is.close();
                baos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            return data;
        }
    }

    public static void test(ClassLoader classLoader) throws Exception {
        Class<?> clazz = classLoader.loadClass("two.jvm.classloader.MyTest");
        //loadClass是父类方法，在方法内部调用findClass
        System.out.println(clazz.hashCode());
        Object object = clazz.newInstance();
        System.out.println(object);
    }

    public static void main(String[] args) throws Exception {
        MyClassLoader loader1 = new MyClassLoader("loader1");
        loader1.setPath("C:\\Users\\ISJINHAO\\Desktop\\");
        Class<?> clazz = loader1.loadClass("two.jvm.classloader.MySample");
        System.out.println(clazz.hashCode());
        clazz.newInstance();
    }

}

/*
312714112
MySample is loaded... sun.misc.Launcher$AppClassLoader@18b4aac2
Exception in thread "main" java.lang.NoClassDefFoundError: two/jvm/classloader/MyCat
	at two.jvm.classloader.MySample.<init>(MySample.java:6)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at java.lang.Class.newInstance(Class.java:442)
	at two.jvm.classloader.MyClassLoader.main(MyClassLoader.java:83)
Caused by: java.lang.ClassNotFoundException: two.jvm.classloader.MyCat
	at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	... 7 more
	
由于MySample是由AppClassLoader加载的，所以new MyCat()使用的是MySample的的类加载器，但是AppClassLoader的命名空间下无法加载MyCat，所以报错。
*/
```

**操作2：**

1. 外部文件有MySample和MyCat字节码
2. classpath下只有MyCat字节码

```java
/*
className=two.jvm.classloader.MySample
C:\Users\ISJINHAO\Desktop\two\jvm\classloader\MySample.class
1554874502
MySample is loaded... two.jvm.classloader.MyClassLoader@12a3a380
MyCat is loaded... sun.misc.Launcher$AppClassLoader@18b4aac2
from MySample: class two.jvm.classloader.MyCat

由于MySample是由MyClassLoader加载的，所以new MyCat()使用的是MySample的的类加载器，但是MyClassLoader的父加载器是可以加载MyCat的，所以MyCat是由AppClassLoader加载的。
*/
```

**操作3**

1. 外部文件有MySample和MyCat字节码
2. classpath下只有MyCat字节码
3. 修改MyCat为

```java
public class MyCat {
    public MyCat() {
        System.out.println("MyCat is loaded... " + this.getClass().getClassLoader());
        System.out.println("from MyCat: " + MySample.class);
    }
}
/*
className=two.jvm.classloader.MySample
C:\Users\ISJINHAO\Desktop\two\jvm\classloader\MySample.class
1554874502
MySample is loaded... two.jvm.classloader.MyClassLoader@12a3a380
MyCat is loaded... sun.misc.Launcher$AppClassLoader@18b4aac2
Exception in thread "main" java.lang.NoClassDefFoundError: two/jvm/classloader/MySample
	at two.jvm.classloader.MyCat.<init>(MyCat.java:6)
	at two.jvm.classloader.MySample.<init>(MySample.java:6)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at java.lang.Class.newInstance(Class.java:442)
	at two.jvm.classloader.MyClassLoader.main(MyClassLoader.java:83)
Caused by: java.lang.ClassNotFoundException: two.jvm.classloader.MySample
	at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	... 8 more

MySample是由MyClassLoader加载的，MyCat是由AppClassLoader加载的，在AppClassLoader中不能访问子加载器加载的类。
*/
```

**操作4**

MySample和MyCat如下：

```java
public class MySample {
    public MySample() {
        System.out.println("MySample is loaded... " + this.getClass().getClassLoader());
        new MyCat();
        System.out.println("from MySample: " + MyCat.class);
    }
}
```

```java
public class MyCat {
    public MyCat() {
        System.out.println("MyCat is loaded... " + this.getClass().getClassLoader());
//        System.out.println("from MyCat: " + MySample.class);
    }
}
```

```java
className=two.jvm.classloader.MySample
C:\Users\ISJINHAO\Desktop\two\jvm\classloader\MySample.class
1554874502
MySample is loaded... two.jvm.classloader.MyClassLoader@12a3a380
MyCat is loaded... sun.misc.Launcher$AppClassLoader@18b4aac2
from MySample: class two.jvm.classloader.MyCat
```

正常。因为MySample是由MyClassLoader加载的，MyCat是由AppClassLoader加载的，在MyClassLoader中可以访问父加载器加载的类。

**命名空间与强转问题**

```java
public class MyPerson {
    private MyPerson person;

    public void setMyPerson(Object object) {
        this.person = (MyPerson) object;
    }
}
```

```java
public class MyTest21 {
    public static void main(String[] args) throws Exception {
        MyClassLoader loader1 = new MyClassLoader("loader1");
        MyClassLoader loader2 = new MyClassLoader("loader2");
        loader1.setPath("C:\\Users\\ISJINHAO\\Desktop\\");
        loader2.setPath("C:\\Users\\ISJINHAO\\Desktop\\");
        // 删掉classpath下的MyPerson类
        Class<?> clazz1 = loader1.loadClass("two.jvm.classloader.MyPerson");
        Class<?> clazz2 = loader2.loadClass("two.jvm.classloader.MyPerson");

        // clazz1和clazz2由loader1和loader2加载，结果为false
        System.out.println(clazz1 == clazz2);
        Object object1 = clazz1.newInstance();
        Object object2 = clazz2.newInstance();

        Method method = clazz1.getMethod("setMyPerson", Object.class);
        // 此处报错，如图所示，loader1和loader2所处不用的命名空间
        method.invoke(object1, object2);
    }
}
/*
className=two.jvm.classloader.MyPerson
C:\Users\ISJINHAO\Desktop\two\jvm\classloader\MyPerson.class
className=two.jvm.classloader.MyPerson
C:\Users\ISJINHAO\Desktop\two\jvm\classloader\MyPerson.class
false
Exception in thread "main" java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at two.jvm.classloader.MyTest21.main(MyTest21.java:22)
Caused by: java.lang.ClassCastException: two.jvm.classloader.MyPerson cannot be cast to two.jvm.classloader.MyPerson
	at two.jvm.classloader.MyPerson.setMyPerson(MyPerson.java:7)
	... 5 more
*/
```

<div align="center"><img width="40%" src="http://q0l9qvfyx.bkt.clouddn.com/24a9845e-bf60-4cdd-b956-156cedb97dcf" /></div>
#### 双亲委托机制的好处

- 可以确保Java和核心库的安全：所有的Java应用都会引用java.lang中的类，也就是说在运行期java.lang中的类会被加载到虚拟机中，如果这个加载过程如果是由自己的类加载器所加载，那么很可能就会在JVM中存在多个版本的java.lang中的类，而且这些类是相互不可见的（命名空间的作用）。借助于双亲委托机制，Java核心类库中的类的加载工作都是由启动根加载器去加载，从而确保了Java应用所使用的的都是同一个版本的Java核心类库，他们之间是相互兼容的；
- 确保Java核心类库中的类不会被自定义的类所替代；
- 不同的类加载器可以为相同名称的类（binary name）创建额外的命名空间。相同名称的类可以并存在Java虚拟机中，只需要用不同的类加载器去加载即可。相当于在Java虚拟机内部建立了一个又一个相互隔离的Java类空间。



### getSystemClassLoader()

>Returns the system class loader for delegation. This is the default delegation parent for new ClassLoader instances, and is typically the class loader used to start the application.
>**This method is first invoked early in the runtime's startup sequence, at which point it creates the system class loader and sets it as the context class loader of the invoking Thread.**
>The default system class loader is an implementation-dependent instance of this class.
>If the system property "java.system.class.loader" is defined when this method is first invoked then the value of that property is taken to be the name of a class that will be returned as the system class loader. The class is loaded using the default system class loader and must define a public constructor that takes a single parameter of type ClassLoader which is used as the delegation parent. An instance is then created using this constructor with the default system class loader as the parameter. The resulting class loader is defined to be the system class loader.

自定义类加载器的双亲默认是`getSystemClassLoader()返回的实例`，启动应用（运行main方法）的类加载器亦是。从文档可以看出此方法会很早就被JVM调用一次，此次调用的时候初始化`AppClasLoader`。实际上，拓展类加载器也是在这个阶段。



### 指定SystemClassLoader

```
在classes文件夹下运行，指定系统类加载器

java -Djava.system.class.loader=two.jvm.classloader.MyClassLoader two.jvm.cla
ssloader.MyTest23
```

```java
/*
    在运行期，一个Java类是由该类的完全限定名（binary name）和用于加载该类的定义类加载器所共同决定的。如果同样名字（完全相同限定名）是由两个不同的加载器所加载，那么这些类就是不同的，即便.class文件字节码相同，并且从相同的位置加载亦如此。
    在oracle的hotspot，系统属性sun.boot.class.path如果修改错了，则运行会出错：
    Error occurred during initialization of VM
    java/lang/NoClassDeFoundError: java/lang/Object
*/
public class MyTest23 {
    public static void main(String[] args) {
        System.out.println(System.getProperty("sun.boot.class.path"));
        System.out.println(System.getProperty("java.ext.dirs"));
        System.out.println(System.getProperty("java.class.path"));
        System.out.println("-----------");
        System.out.println(ClassLoader.class.getClassLoader());
        System.out.println(Launcher.class.getClassLoader());

        // 下面的系统属性指定系统类加载器，默认是AppClassLoader
        System.out.println(System.getProperty("java.system.class.loader"));
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
		// 上面官方文档的解释，会将AppClassLoader设置为指定系统类加载器的父亲
        System.out.println(systemClassLoader.getParent());
    }
}
/*
需要加入一个新的构造方法：
    public MyClassLoader(ClassLoader parent) {
        super(parent);      //显式指定该类的父加载器
    }

从终端启动：>java -Djava.system.class.loader=two.jvm.classloader.MyClassLoader two.jvm.cla
ssloader.MyTest23

D:\Java\jdk\jre\lib\resources.jar;D:\Java\jdk\jre\lib\rt.jar;D:\Java\jdk\jre\lib\sunrsasign.jar;D:\Java\jdk\jre\lib\jsse.jar;D:\Java\jdk\jre\lib\jce.jar;D:\Java\jdk\jre\lib\charsets.ja
r;D:\Java\jdk\jre\lib\jfr.jar;D:\Java\jdk\jre\classes
D:\Java\jdk\jre\lib\ext;C:\WINDOWS\Sun\Java\lib\ext
.
-----------
null
null
two.jvm.classloader.MyClassLoader
sun.misc.Launcher$AppClassLoader@18b4aac2
*/
```

#### 三个自带的类加载器初始化流程分析

由于三个自带的类加载器初始化涉及的代码较多，所以下面只是按照方法的执行顺序来进行分析。而方法的起始便是我们之前提过的`getSystemClassLoader()`，这个方法在JVM后很早期就会被调用。

```java
// The class loader for the system
// @GuardedBy("ClassLoader.class")
private static ClassLoader scl;

// 如果没有初始化，初始化，否则返回SystemClassLoader，默认是AppClassLoader
public static ClassLoader getSystemClassLoader() {
    initSystemClassLoader();
    if (scl == null) {
        return null;
    }
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        checkClassLoaderPermission(scl, Reflection.getCallerClass());
    }
    return scl;
}
```

```java
// Set to true once the system class loader has been set
// @GuardedBy("ClassLoader.class")
private static boolean sclSet;

// 初始化系统类加载器
private static synchronized void initSystemClassLoader() {
    // 没有被初始化过
    if (!sclSet) {
        if (scl != null)
            throw new IllegalStateException("recursive invocation");
        // 获得Launcher，在Launcher的构造方法中初始化拓展类加载器和AppClassLoader
        sun.misc.Launcher l = sun.misc.Launcher.getLauncher();
        if (l != null) {
            Throwable oops = null;
            // 获得AppClassLoader
            scl = l.getClassLoader();
            try {
                // 如果类加载器是被-Djava.system.class.loader指定的，通过这一步设置给scl
                scl = AccessController.doPrivileged(new SystemClassLoaderAction(scl));
            } catch (PrivilegedActionException pae) {
                oops = pae.getCause();
                if (oops instanceof InvocationTargetException) {
                    oops = oops.getCause();
                }
            }
            if (oops != null) {
                if (oops instanceof Error) {
                    throw (Error) oops;
                } else {
                    // wrap the exception
                    throw new Error(oops);
                }
            }
        }
        sclSet = true;
    }
}
```

```java
private static Launcher launcher = new Launcher();

public static Launcher getLauncher() { return launcher; }

public Launcher() {
    Launcher.ExtClassLoader var1;
    try {
        var1 = Launcher.ExtClassLoader.getExtClassLoader();  // 获得拓展类加载器
    } catch (IOException var10) {
        throw new InternalError("Could not create extension class loader", var10);
    }

    try {
        this.loader = Launcher.AppClassLoader.getAppClassLoader(var1);
    } catch (IOException var9) {
        throw new InternalError("Could not create application class loader", var9);
    }
	// 设置线程上下文类加载器
    Thread.currentThread().setContextClassLoader(this.loader);
    String var2 = System.getProperty("java.security.manager");
    if (var2 != null) {
        SecurityManager var3 = null;
        if (!"".equals(var2) && !"default".equals(var2)) {
            try {
                var3 = (SecurityManager)this.loader.loadClass(var2).newInstance();
            } catch (Exception xxx) {            }
        } else {   var3 = new SecurityManager();       }
        if (var3 == null) {
            throw new InternalError("Could not create SecurityManager: " + var2);
        }
        System.setSecurityManager(var3);
    }
}
```

```java
public static Launcher.ExtClassLoader getExtClassLoader() throws IOException {
    if (instance == null) {
        Class var0 = Launcher.ExtClassLoader.class;
        synchronized(Launcher.ExtClassLoader.class) {
            if (instance == null) { instance = createExtClassLoader(); }
        }
    }
    return instance;
}

// 构建一个拓展类加载器
private static Launcher.ExtClassLoader createExtClassLoader() throws IOException {
    try {
        return (Launcher.ExtClassLoader)AccessController.doPrivileged(new PrivilegedExceptionAction<Launcher.ExtClassLoader>() {
            // 获得ExtDirs并且构建 拓展类加载器
            // 由于拓展类加载器继承了URLClassLoader，给定File便可以进行加载
            public Launcher.ExtClassLoader run() throws IOException {
                File[] var1 = Launcher.ExtClassLoader.getExtDirs();
                int var2 = var1.length;
                for(int var3 = 0; var3 < var2; ++var3) {
                    MetaIndex.registerDirectory(var1[var3]);
                }
                return new Launcher.ExtClassLoader(var1);
            }
        });
    } catch (PrivilegedActionException var1) {
        throw (IOException)var1.getException();
    }
}

// 获得java.ext.dirs属性对应的jar包
private static File[] getExtDirs() {
    String var0 = System.getProperty("java.ext.dirs");
    File[] var1;
    if (var0 != null) {
        StringTokenizer var2 = new StringTokenizer(var0, File.pathSeparator);
        int var3 = var2.countTokens();
        var1 = new File[var3];
        for(int var4 = 0; var4 < var3; ++var4) { var1[var4] = new File(var2.nextToken()); }
    } else { var1 = new File[0];  }
    return var1;
}
```

```java
// 获取AppClassLoader
public static ClassLoader getAppClassLoader(final ClassLoader var0) throws IOException {
    // jars由java.class.path属性指定
    final String var1 = System.getProperty("java.class.path");
    final File[] var2 = var1 == null ? new File[0] : Launcher.getClassPath(var1);
    return (ClassLoader)AccessController.doPrivileged(new PrivilegedAction<Launcher.AppClassLoader>() {
        public Launcher.AppClassLoader run() {
            URL[] var1x = var1 == null ? new URL[0] : Launcher.pathToURLs(var2);
            return new Launcher.AppClassLoader(var1x, var0);
        }
    });
}
```

```java
class SystemClassLoaderAction implements PrivilegedExceptionAction<ClassLoader> {
    private ClassLoader parent;

    SystemClassLoaderAction(ClassLoader parent) {  this.parent = parent;  }

    public ClassLoader run() throws Exception {
        String cls = System.getProperty("java.system.class.loader");
        // 如果没有设置java.system.class.loader，返回parent
        if (cls == null) {   return parent;   }
        // 调用自定义类加载器中能接受一个 parentClassLoader 的构造方法
        Constructor<?> ctor = Class.forName(cls, true, parent)
            .getDeclaredConstructor(new Class<?>[] { ClassLoader.class });
        ClassLoader sys = (ClassLoader) ctor.newInstance(
            new Object[] { parent });
        // 更新线程上下文类加载器
        Thread.currentThread().setContextClassLoader(sys);
        return sys;
    }
}
```



### Class.forName

```java
/*
name：class name
initialize: 是否初始化
loader: 类加载器
*/
public static Class<?> forName(String name, boolean initialize, ClassLoader loader)
```

> Returns the Class object associated with the class or interface with the given string name, using the given class loader. Given the fully qualified name for a class or interface (in the same format returned by getName) this method attempts to locate, load, and link the class or interface. The specified class loader is used to load the class or interface. If the parameter loader is null, the class is loaded through the bootstrap class loader. The class is initialized only if the initialize parameter is true and if it has not been initialized earlier.
> If name denotes a primitive type or void, an attempt will be made to locate a user-defined class in the unnamed package whose name is name. Therefore, this method cannot be used to obtain any of the Class objects representing primitive types or void.
> If name denotes an array class, the component type of the array class is loaded but not initialized.
> For example, in an instance method the expression:
>
> ```java
> Class.forName("Foo")
> is equivalent to:
> Class.forName("Foo", true, this.getClass().getClassLoader())
> ```
>
>
> Note that this method throws errors related to loading, linking or initializing as specified in Sections 12.2, 12.3 and 12.4 of The Java Language Specification. Note that this method does not check whether the requested class is accessible to its caller.
> If the loader is null, and a security manager is present, and the caller's class loader is not null, then this method calls the security manager's checkPermission method with a RuntimePermission("getClassLoader") permission to ensure it's ok to access the bootstrap class loader.

```java
// 默认初始化
// 默认的类加载器是调用者的类加载器
public static Class<?> forName(String className)
    throws ClassNotFoundException {
    Class<?> caller = Reflection.getCallerClass();
    return forName0(className, true, ClassLoader.getClassLoader(caller), caller);
}
```



### 线程上下文类加载器

#### **当前类加载器**

每个类都会使用自己的类加载器（即加载自身的类加载器）来去加载其他类（指的是所依赖的类），如果classX引用了classY，那么加载classX的类加载器就会去加载classY。

```java
public class MyTest24 {

    public static void main(String[] args) throws Exception {
        MyClassLoader myClassLoader = new MyClassLoader("myclassloader");
        myClassLoader.setPath("C:\\Users\\ISJINHAO\\Desktop\\");
        Class.forName("two.jvm.classloader.Test1", true, myClassLoader);
    }
}

class Test1 {
    static Test2 test2 = new Test2();
    static {
        // Test2是由MyClassLoader加载的
        System.out.println(test2.getClass().getClassLoader());
    }
}

class Test2 {   }
/*
从外部加载class文件：
    className=two.jvm.classloader.Test1
    C:\Users\ISJINHAO\Desktop\two\jvm\classloader\Test1.class
    className=two.jvm.classloader.Test2
    C:\Users\ISJINHAO\Desktop\two\jvm\classloader\Test2.class
    two.jvm.classloader.MyClassLoader@4554617c
*/
```

这时候我们有一个问题，就是如果有一个类加载器委托模型如图：

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/e5d0c474-d9c1-41d9-9206-e4d57c18659a" /></div>
如果我们用loader1加载classX，loader1的父加载器是AppClassLoader，在classX中有一个classY成员变量。JVM是使用AppClassLoader加载classY，还是使用loader1加载classY，再委托给AppClassLoader？测试如下

```java
public class MyTest26 {
    MyTest27 myTest27 = new MyTest27();
}
```

```java
public class MyTest27 {  }
```

```java
public class MyTest25 {
    public static void main(String[] args) throws Exception {

        MyClassLoader loader1 = new MyClassLoader("loader1");
        loader1.setPath("C:\\Users\\ISJINHAO\\Desktop\\");

        Class<?> aClass = loader1.loadClass("two.jvm.classloader.MyTest26");
        aClass.newInstance();
    }
}
```

```java
/**
 * 创建自定义加载器，继承ClassLoader
 */
public class MyClassLoader extends ClassLoader {
    private String classLoaderName;
    private String path = "";
    private final String fileExtension = ".class";

    public MyClassLoader(String classLoaderName) {
        super();        // 将系统类当做该类的父加载器
        this.classLoaderName = classLoaderName;
    }
    public void setPath(String path) { this.path = path; }

    @Override
    public Class<?> findClass(String className) {
        System.out.println("className=" + className);
        System.out.println("classLoaderName=" + classLoaderName);
        byte[] data = loadClassData(className);
        return defineClass(className, data, 0, data.length); //define方法为父类方法
    }

    private byte[] loadClassData(String name) {
        InputStream is = null;
        byte[] data = null;
        ByteArrayOutputStream baos = null;
        try {
            File file = new File(this.path + "\\" + name.replace(".", "\\") + this.fileExtension);
            System.out.println(file.exists() + " " + file.getAbsolutePath());
            is = new FileInputStream(file);
            baos = new ByteArrayOutputStream();
            int ch;

            while (-1 != (ch = is.read())) {
                baos.write(ch);
            }
            data = baos.toByteArray();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (is != null)
                    is.close();
                if (baos != null)
                    baos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            return data;
        }
    }
}
```

**操作步骤1**

1. 删除当前类路径下的MyTest27
2. "C:\\Users\\ISJINHAO\\Desktop\\"路径下有MyTest26和MyTest27

**结果**

```java
Exception in thread "main" java.lang.NoClassDefFoundError: two/jvm/classloader/MyTest27
	at two.jvm.classloader.MyTest26.<init>(MyTest26.java:5)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at java.lang.Class.newInstance(Class.java:442)
	at two.jvm.classloader.MyTest25.main(MyTest25.java:10)
Caused by: java.lang.ClassNotFoundException: two.jvm.classloader.MyTest27
	at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	... 7 more
```

**操作步骤2**

1. 删除当前类路径下的MyTest27和MyTest26
2. "C:\\Users\\ISJINHAO\\Desktop\\"路径下有MyTest26和MyTest27

**结果**

```
className=two.jvm.classloader.MyTest26
classLoaderName=loader1
true C:\Users\ISJINHAO\Desktop\two\jvm\classloader\MyTest26.class
className=two.jvm.classloader.MyTest27
classLoaderName=loader1
true C:\Users\ISJINHAO\Desktop\two\jvm\classloader\MyTest27.class
```

**分析**

操作1里面loader1加载MyTest26，loader1委托给AppClassLoader，所以MyTest26是被AppClassLoader加载的，如果MyTest27是由被loader1加载，再委托给AppClassLoader的，AppClassLoader无法加载，再交由loader1，如此一来，MyTest27可以顺利加载。实际上无法顺利加载，说明MyTest27是直接被交给AppClassLoader的，AppClassLoader无法加载，所以有ClassNotFoundException。

操作2里面loader1加载MyTest26，loader1委托给AppClassLoader，AppClassLoader无法加载，再交给loader1，loader1从“C:\\Users\\ISJINHAO\\Desktop\\”路径下加载MyTest26，加载MyTest27的时候也是从外部路径加载的。

##### **总结**

每个类都会使用自己的类加载器（即加载自身的类加载器）来去加载其他类（指的是所依赖的类），如果classX引用了classY，那么classX的定义类加载器就会去加载classY。

#### **线程上下文类加载器**

线程上下文类加载器是从JDK1.2开始引入的类，Thread类中的`getContextClassLoader()`与`setContextClassLoader(ClassLoader cl)`分别用来获取和设置上下文类加载器。如果没有通过set方法进行设置的话，线程将继承其父线程的上下文类加载器。Java应用运行时的初始线程的上下文类加载器是AppClassLoader。在线程中运行的代码可以通过该类加载器来加载类与资源。

线程上下文类加载器的重要性：在SPI（Service Provider Interface）机制下，ServiceLoader是由Bootstrap类加载器加载的，但是外包部导入的包是由AppClassLoader加载的。按照我们刚才的验证知道，这样其实是不可能的，因为在Bootstrap类加载器中根本无法看见当前类路径下的jar包。解决这个问题的就是线程上下文类加载器。

```java
public class MyTest28 {
    public static void main(String[] args) {
        ServiceLoader<Driver> loader = ServiceLoader.load(Driver.class);
        Iterator<Driver> iterator = loader.iterator();

        while(iterator.hasNext()){
            Driver next = iterator.next();
            System.out.println("driver: " + next + ", loader" 
                               + next.getClass().getClassLoader());
        }
        System.out.println("ServiceLoader类加载器: " + ServiceLoader.class.getClassLoader());
        System.out.println("当前类加载器：" + Thread.currentThread().getContextClassLoader());
    }
}
/*
driver: com.mysql.jdbc.Driver@5e481248, loadersun.misc.Launcher$AppClassLoader@18b4aac2
driver: com.mysql.fabric.jdbc.FabricMySQLDriver@66d3c617, loadersun.misc.Launcher$AppClassLoader@18b4aac2
ServiceLoader类加载器: null
当前类加载器：sun.misc.Launcher$AppClassLoader@18b4aac2
*/
```

**跟踪分析**

```java
public static <S> ServiceLoader<S> load(Class<S> service) {
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    // 传入当前线程类加载器
    return ServiceLoader.load(service, cl);
}
public static <S> ServiceLoader<S> load(Class<S> service, ClassLoader loader) {
    return new ServiceLoader<>(service, loader);
}
```

```java
// The class loader used to locate, load, and instantiate providers
private final ClassLoader loader;

private ServiceLoader(Class<S> svc, ClassLoader cl) {
    service = Objects.requireNonNull(svc, "Service interface cannot be null");
    loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
    acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
    reload();
}
```

hasNext的源码省略...其目的是确定`META-INF/services`中的文件的全限定名。下面是next代码：

```java
public S next() {
    if (acc == null) {
        return nextService();
    } else {
        PrivilegedAction<S> action = new PrivilegedAction<S>() {
            public S run() { return nextService(); }
        };
        return AccessController.doPrivileged(action, acc);
    }
}
private S nextService() {
    if (!hasNextService())
        throw new NoSuchElementException();
    String cn = nextName;
    nextName = null;
    Class<?> c = null;
    try {
        // 在这里便是使用之前从线程获得类加载器进行类的加载
        c = Class.forName(cn, false, loader);
    } catch (ClassNotFoundException x) {
        fail(service, "Provider " + cn + " not found");
    }
    if (!service.isAssignableFrom(c)) {
        fail(service, "Provider " + cn  + " not a subtype");
    }
    try {
        S p = service.cast(c.newInstance());
        providers.put(cn, p);
        return p;
    } catch (Throwable x) {
        fail(service, "Provider " + cn + " could not be instantiated", x);
    }
    throw new Error();          // This cannot happen
}
```

