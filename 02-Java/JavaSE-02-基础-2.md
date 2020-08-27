## 内部类

### 成员内部类

成员内部类是最普通的内部类，它的定义为位于另一个类的内部，形如下面的形式：

```java
class Circle {
    double radius = 0;
    
    public Circle(double radius) { this.radius = radius; }
    
    class Draw {     // 内部类
        public void drawSahpe() {
            System.out.println("drawshape");
        }
    }
}
```

这样看起来，类Draw像是类Circle的一个成员，Circle称为外部类。成员内部类可以无条件访问外部类的所有成员属性和成员方法（包括private成员和静态成员）。

```java
public class Outer {
    private double radius = 0;
    private int test = 1;
    public static int count =1;
    public Outer(double radius) {
        this.radius = radius;
    }
    
    class Draw {     // 内部类
    	private int test = 2;
        public void drawSahpe() {
            System.out.println(radius);  // 外部类的private成员
            System.out.println(count);   // 外部类的静态成员
            System.out.println(Outer.this.test);
        }
    }
    
    public static void main(String[] args) {
    	Draw draw = new Outer(20).new Draw();
    	draw.drawSahpe();
	}
}
```

不过要注意的是，当成员内部类拥有和外部类同名的成员变量或者方法时，会发生隐藏现象，即默认情况下访问的是成员内部类的成员。如果要访问外部类的同名成员，需要以下面的形式进行访问：

```
外部类.this.成员变量
外部类.this.成员方法
```

虽然成员内部类可以无条件地访问外部类的成员，而外部类想访问成员内部类的成员却不是这么随心所欲了。在外部类中如果要访问成员内部类的成员，必须先创建一个成员内部类的对象，再通过指向这个对象的引用来访问：

```java
class Circle {
    private double radius = 0;

    public Circle(double radius) {
        this.radius = radius;
        getDrawInstance().drawSahpe();   // 必须先创建成员内部类的对象，再进行访问
    }

    private Draw getDrawInstance() {
        return new Draw();
    }
     
    class Draw {     // 内部类
        public void drawSahpe() {
            System.out.println(radius);  // 外部类的private成员
        }
    }
}
```

成员内部类是依附外部类而存在的，也就是说，如果要创建成员内部类的对象，前提是必须存在一个外部类的对象。创建成员内部类对象的一般方式如下：

```java
public class Test {
    public static void main(String[] args)  {
        // 第一种方式：
        Outter outter = new Outter();
        Outter.Inner inner = outter.new Inner();  // 必须通过Outter对象来创建
         
        // 第二种方式：
        Outter.Inner inner1 = outter.getInnerInstance();
    }
}
 
class Outter {
    private Inner inner = null;
    public Outter() { }
     
    public Inner getInnerInstance() {
        if(inner == null)
            inner = new Inner();
        return inner;
    }
      
    class Inner {
        public Inner() { }
    }
}
```

内部类可以拥有private访问权限、protected访问权限、public访问权限及包访问权限。如上面的例子：

- private：则只能在外部类的内部访问；
- public：则任何地方都能访问；
- protected：只能在同一个包下或者继承外部类的情况下访问；
- 默认访问权限：则只能在同一个包下访问。

这一点和外部类有一点不一样，外部类只能被public和包访问两种权限修饰。我个人是这么理解的，由于成员内部类看起来像是外部类的一个成员，所以可以像类的成员一样拥有多种权限修饰。



### 局部内部类

局部内部类是定义在一个方法或者一个作用域里面的类，它和成员内部类的区别在于局部内部类的访问仅限于方法内或者该作用域内。

```java
class People{
    public People() { }
}
 
class Man{
    public Man(){ }
    public People getWoman(){
        class Woman extends People{   // 局部内部类
            int age =0;
        }
        return new Woman();
    }
}
```



### 匿名内部类

匿名内部类应该是平时我们编写代码时用得最多的，在编写事件监听的代码时使用匿名内部类不但方便，而且使代码更加容易维护。匿名内部类的语法：

```java
new ClassName(){
	// 匿名内部类的body
}
```

下面这段代码是一段线程代码：

```java
public static void main(String[] args) {
    System.out.println(Thread.currentThread());
    new Thread("匿名内部类"){
        {
            System.out.println("匿名内部类的body");
        }
        @Override
        public void run() {
            System.out.println(this);
        }
    }.start();
}
```

匿名内部类是**唯一一种没有构造器的类**。正因为其没有构造器，所以匿名内部类的使用范围非常有限，大部分匿名内部类用于接口回调。匿名内部类在编译的时候由系统自动起名为Outter$数字.class。一般来说，匿名内部类用于继承其他类或是实现接口，并不需要增加额外的方法，只是对继承方法的实现或是重写。但是匿名内部类也是可以书写其他方法的，因为本质上他就是继承或实现。



### 静态内部类

静态内部类也是定义在另一个类里面的类，只不过在类的前面多了一个关键字static。静态内部类是不需要依赖于外部类的，这点和类的静态成员属性有点类似，并且它不能使用外部类的非static成员变量或者方法，这点很好理解，因为在没有外部类的对象的情况下，可以创建静态内部类的对象，如果允许访问外部类的非static成员就会产生矛盾，因为外部类的非static成员必须依附于具体的对象。

```java
public class Test {
    public static void main(String[] args)  {
        // 静态内部类是不依赖于外部类的，也就说可以在不创建外部类对象的情况下创建内部类的对象。
        Outter.Inner inner = new Outter.Inner();
    }
}
 
class Outter {
    public Outter() { }
     
    static class Inner {
        public Inner() { }
    }
}
```



## 枚举和switch

```java
public class Test {
	public static void main(String[] args) {
		System.out.println(TestEnum.FRI.getClass()); 	// class TestEnum
		System.out.println(TestEnum.FRI);		// FRI
	}
}

enum TestEnum {
	 MON, TUE, WED, THU, FRI, SAT, SUN;
}
```

switch中可以跟的类型有byte、short、char、int、String、枚举。**没有long**。

```java
String str = "200";

switch (str) {
	default:		// default 只能定义一次
		System.out.println(100);
	case "100":
		System.out.println(1000);
	case "1000":
		System.out.println(1000);
}

//str = 200;
//输出：
//	100
//	1000
//	1000
//因为不能匹配100或1000，所以从defalut口进入。
//
//str = 100;
//输出：
//	1000
//	1000
//因为可以匹配100，从第二个口进入。
```



## 随机数和round

```java
// 获得随机数：greater than or equal to 0.0 and less than 1.0
double iRandom = Math.random();
System.out.println(iRandom);

// 获得区间在[0, n)之间的随机整数
Random random2 = new Random();
int nextInt = random2.nextInt(10);
System.out.println(nextInt);
```

```java
// 负数：加0.5之后向小的取整
System.out.println(Math.round(-11.6));
System.out.println(Math.round(-11.5));
// 正数：加0.5之后向大的取整
System.out.println(Math.round(11.6));
System.out.println(Math.round(11.5));
```



## String、StringBuilder & StringBuffer

### **可变性**

简单的来说：String类中使用final关键字修饰字符数组来保存字符串，`private　final　char　value[]`，所以String对象是不可变的。而`StringBuilder`与`StringBuffer`都继承自`AbstractStringBuilder`类，在 `AbstractStringBuilder`中也是使用字符数组保存字符串`char[]value` 但是没有用final关键字修饰，所以这两种对象都是可变的。	

`StringBuilde`r与`StringBuffer`的构造方法都是调用父类构造方法也就是`AbstractStringBuilder`实现的。

`AbstractStringBuilder.java`

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    char[] value;
    int count;
    AbstractStringBuilder() {
    }
    AbstractStringBuilder(int capacity) {
        value = new char[capacity];
    }
}
```



### 线程安全性

`String`中的对象是不可变的，也就可以理解为常量，线程安全。`AbstractStringBuilder`是`StringBuilder`与`StringBuffer`的公共父类，定义了一些字符串的基本操作，如`expandCapacity`、`append`、`insert`、`indexOf`等公共方法。`StringBuffer`对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。`StringBuilder`并没有对方法进行加同步锁，所以是非线程安全的。　



### **性能**

每次对`String`类型进行改变的时候，都会生成一个新的`String`对象，然后将指针指向新的`String`对象。`StringBuffer`每次都会对`StringBuffer`对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用`StringBuilder`相比使用`StringBuffer`仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。



### **对于三者使用的总结：**

1. 操作少量的数据: 适用String
2. 单线程操作字符串缓冲区下操作大量数据: 适用`StringBuilder`
3. 多线程操作字符串缓冲区下操作大量数据: 适用`StringBuffer`

`Java`中字符串相加，编译的时候会创建`StringBuilder`类，然后利用`StringBuilder`的`append()`方法相加。所以不能在循环里使用`+`，需要在循环外面使用创建`StringBuilder`的对象，在循环内进行`append()`操作。



## 日期类

### LocalDate

```java
// 当前日期
LocalDate today = LocalDate.now();
System.out.println("Current Date=" + today);

// 根据年、月、日创建日期
LocalDate firstDay_2014 = LocalDate.of(2014, Month.JANUARY, 1);
System.out.println("Specific Date=" + firstDay_2014);

// 不合法的参数会报 java.time.DateTimeException
// LocalDate feb29_2014 = LocalDate.of(2014, Month.FEBRUARY, 29);

// Current date in "Asia/Kolkata", you can get it from ZoneId javadoc
LocalDate todayKolkata = LocalDate.now(ZoneId.of("Asia/Kolkata"));
System.out.println("Current Date in IST=" + todayKolkata);

// java.time.zone.ZoneRulesException: Unknown time-zone ID: IST
// LocalDate todayIST = LocalDate.now(ZoneId.of("IST"));

// Getting date from the base date: 01/01/1970
LocalDate dateFromBase = LocalDate.ofEpochDay(365);
System.out.println("365th day from base date= " + dateFromBase);

// 得到某年的多少天
LocalDate hundredDay2014 = LocalDate.ofYearDay(2014, 100);
System.out.println("100th day of 2014=" + hundredDay2014);
```



### LocalTime

```java
// Current Time
LocalTime time = LocalTime.now();
System.out.println("Current Time = " + time);

// Creating LocalTime by providing input arguments
LocalTime specificTime = LocalTime.of(12, 20, 25, 40);
System.out.println("Specific Time of Day = " + specificTime);

// Try creating time by providing invalid inputs
// LocalTime invalidTime = LocalTime.of(25,20);
// Exception in thread "main" java.time.DateTimeException:

// Invalid value for HourOfDay (valid values 0 - 23): 25
// Current date in "Asia/Kolkata", you can get it from ZoneId javadoc
LocalTime timeKolkata = LocalTime.now(ZoneId.of("Asia/Kolkata"));

System.out.println("Current Time in IST = " + timeKolkata);
// java.time.zone.ZoneRulesException: Unknown time-zone ID: IST
// LocalTime todayIST = LocalTime.now(ZoneId.of("IST"));

// Getting time from the base date: 00:00:00
LocalTime specificSecondTime = LocalTime.ofSecondOfDay(10000);
System.out.println("10000th second time = " + specificSecondTime);
```



### LocalDateTime

```java
// Current Date
LocalDateTime today = LocalDateTime.now();
System.out.println("Current DateTime1 = " + today);

// Current Date using LocalDate and LocalTime
today = LocalDateTime.of(LocalDate.now(), LocalTime.now());
System.out.println("Current DateTime2 = " + today);

// Creating LocalDateTime by providing input arguments
LocalDateTime specificDate = LocalDateTime.of(2014, Month.JANUARY, 1, 10, 10, 30);
System.out.println("Specific Date = " + specificDate);

// Try creating date by providing invalid inputs
// LocalDateTime feb29_2014 = LocalDateTime.of(2014, Month.FEBRUARY, 28, 25, 1, 1);
// Exception in thread "main" java.time.DateTimeException:

// Invalid value for HourOfDay (valid values 0 - 23): 25
// Current date in "Asia/Kolkata", you can get it from ZoneId javadoc
LocalDateTime todayKolkata = LocalDateTime.now(ZoneId.of("Asia/Kolkata"));
System.out.println("Current Date in IST = " + todayKolkata);

// java.time.zone.ZoneRulesException: Unknown time-zone ID: IST
// LocalDateTime todayIST = LocalDateTime.now(ZoneId.of("IST"));

// Getting date from the base date: 01/01/1970
LocalDateTime dateFromBase = LocalDateTime.ofEpochSecond(10000, 0, ZoneOffset.UTC);
System.out.println("10000th second time from 01/01/1970= " + dateFromBase);
```



### Instant

```java
// Current timestamp
Instant timestamp = Instant.now();
System.out.println("Current Timestamp = " + timestamp);

// 转换为 milliseconds from 01/01/1970
long epochMilli = timestamp.toEpochMilli();
System.out.println(epochMilli);
```



### Date & SimpleDateFormat

```java
SimpleDateFormat oldFormatter = new SimpleDateFormat("yyyy/MM/dd");
Date date1 = new Date();
System.out.println(oldFormatter.format(date1));
```



### 相互转换

```java
Date now = new Date();

LocalDateTime localDateTime = now.toInstant().atZone(ZoneId.systemDefault()).toLocalDateTime();
LocalTime localTime = now.toInstant().atZone(ZoneId.systemDefault()).toLocalTime();
LocalDate localDate = now.toInstant().atZone(ZoneId.systemDefault()).toLocalDate();

System.out.println(localDateTime);
System.out.println(localTime);
System.out.println(localDate);

// LocalTime 和 LocalDate 没有atZone方法
Date oldLocalDateTime = 
	Date.from(localDateTime.atZone(ZoneId.systemDefault()).toInstant());
System.out.println(oldLocalDateTime);

long millis = 
	localDateTime.atZone(ZoneId.systemDefault()).toInstant().toEpochMilli();
long time = now.getTime();
System.out.println(millis);
System.out.println(time);

LocalDateTime parseLocalDateTime = 
	Instant.ofEpochMilli(millis).atZone(ZoneId.systemDefault()).toLocalDateTime();
LocalDate parseLocalDate = 
	Instant.ofEpochMilli(millis).atZone(ZoneId.systemDefault()).toLocalDate();

System.out.println(parseLocalDateTime);
System.out.println(parseLocalDate);
```



### 获取年月日时钟秒

```java
LocalDateTime dt = LocalDateTime.now();
System.out.println(dt.getYear());
System.out.println(dt.getMonthValue()); // 1 - 12
System.out.println(dt.getDayOfMonth());
System.out.println(dt.getHour());
System.out.println(dt.getMinute());
System.out.println(dt.getSecond());
```



### 获得某月第一天/最后一天

```java
LocalDate today = LocalDate.now();
//本月的第一天
LocalDate firstday = LocalDate.of(today.getYear(), today.getMonth(), 1);
//本月的最后一天
LocalDate lastDay =today.with(TemporalAdjusters.lastDayOfMonth());
System.out.println("本月的第一天       " + firstday);
System.out.println("本月的最后一天   " + lastDay);
```



### 格式化日期

```java
DateTimeFormatter dtf1 = DateTimeFormatter.ofPattern("yyyy/MM/dd  HH:mm:ss");
DateTimeFormatter dtf2 = DateTimeFormatter.ofPattern("yyyy/MM/dd");

LocalDateTime ldt = LocalDateTime.now();
LocalDate ld = LocalDate.now();

System.out.println(ldt);
System.out.println(ld);

String format1 = ldt.format(dtf1);		// 日期格式化为字符串
String format2 = ld.format(dtf2);		// 日期格式化为字符串
System.out.println(format1);
System.out.println(format2);

LocalDateTime parse1 = LocalDateTime.parse(format1, dtf1);	
LocalDate parse2 = LocalDate.parse(format2, dtf2);

System.out.println(parse1);		// 字符串转化为日期
System.out.println(parse2);		// 字符串转化为日期
```



## BigInteger&BigDecimal

### BigInteger

理论上可以表示无限大的数字。常用方法：

```java
BigInteger abs()  返回大整数的绝对值
BigInteger add(BigInteger val) 返回两个大整数的和
BigInteger and(BigInteger val)  返回两个大整数的按位与的结果
BigInteger andNot(BigInteger val) 返回两个大整数与非的结果
BigInteger divide(BigInteger val)  返回两个大整数的商
double doubleValue()   返回大整数的double类型的值
float floatValue()   返回大整数的float类型的值
BigInteger gcd(BigInteger val)  返回大整数的最大公约数
int intValue() 返回大整数的整型值
long longValue() 返回大整数的long型值
BigInteger max(BigInteger val) 返回两个大整数的最大者
BigInteger min(BigInteger val) 返回两个大整数的最小者
BigInteger mod(BigInteger val) 用当前大整数对val求模
BigInteger multiply(BigInteger val) 返回两个大整数的积
BigInteger negate() 返回当前大整数的相反数
BigInteger not() 返回当前大整数的非
BigInteger or(BigInteger val) 返回两个大整数的按位或
BigInteger pow(int exponent) 返回当前大整数的exponent次方
BigInteger remainder(BigInteger val) 返回当前大整数除以val的余数
BigInteger leftShift(int n) 将当前大整数左移n位后返回
BigInteger rightShift(int n) 将当前大整数右移n位后返回
BigInteger subtract(BigInteger val)返回两个大整数相减的结果
byte[] toByteArray(BigInteger val)将大整数转换成二进制反码保存在byte数组中
String toString() 将当前大整数转换成十进制的字符串形式
BigInteger xor(BigInteger val) 返回两个大整数的异或
```



### BigDecimal

浮点数之间的等值判断，基本数据类型不能用==来比较，包装数据类型不能用 equals 来判断。具体原理和浮点数的编码方式有关。

```java
float a = 1.0f - 0.9f;
float b = 0.9f - 0.8f;
System.out.println(a);// 0.100000024
System.out.println(b);// 0.099999964
System.out.println(a == b);// false
```

具有基本计算机组原知识的我们很清楚的知道输出并不是我们想要的结果（**精度丢失**），我们如何解决这个问题呢？一种很常用的方法是：**使用使用 BigDecimal 来定义浮点数的值，再进行浮点数的运算操作。**

```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");
BigDecimal x = a.subtract(b);// 0.1
BigDecimal y = b.subtract(c);// 0.1
System.out.println(x.equals(y));// true 
```

#### API

```java
BigDecimal(int)       创建一个具有参数所指定整数值的对象。      
BigDecimal(double)    创建一个具有参数所指定双精度值的对象。     
BigDecimal(long)      创建一个具有参数所指定长整数值的对象。     
BigDecimal(String)    创建一个具有参数所指定以字符串表示的数值的对象。


add(BigDecimal)       BigDecimal对象中的值相加，然后返回这个对象。
subtract(BigDecimal)  BigDecimal对象中的值相减，然后返回这个对象。
multiply(BigDecimal)  BigDecimal对象中的值相乘，然后返回这个对象。
divide(BigDecimal)    BigDecimal对象中的值相除，然后返回这个对象。
toString()            将BigDecimal对象的数值转换成字符串。    
doubleValue()         将BigDecimal对象中的值以双精度数返回。   
floatValue()          将BigDecimal对象中的值以单精度数返回。   
longValue()           将BigDecimal对象中的值以长整数返回。    
intValue()            将BigDecimal对象中的值以整数返回。
```

我们在使用BigDecimal时，为了防止精度丢失，推荐使用它的 **BigDecimal(String)** 构造方法来创建对象。

```java
BigDecimal a = new BigDecimal(1.01);
BigDecimal b = new BigDecimal(1.02);
BigDecimal c = new BigDecimal("1.01");
BigDecimal d = new BigDecimal("1.02");
System.out.println(a.add(b));
System.out.println(c.add(d));

输出：
2.0300000000000000266453525910037569701671600341796875
2.03
```



### 整型包装类值的比较

所有整形包装类对象值得比较必须使用equals方法。

```java
Integer x = 3;
Integer y = 3;
System.out.println(x == y);// true
Integer a = new Integer(3);
Integer b = new Integer(3);
System.out.println(a == b);//false
System.out.println(a.equals(b));//false
```



## 序列化

把变量从内存中变成可存储或传输的过程称之为序列化，把字节序列恢复为Java对象的过程称为对象的反序列化。对象的序列化主要有两种用途：

1. 把对象的字节序列永久的保存到硬盘上，通常存放在一个文件中。
2. 在网络上传送对象的字节序列。

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/137588ce-f62f-4bb9-9298-2c27daa4b0ef"></div>
### 默认序列化

```java
public class Server {
	public static void main(String[] args) throws Exception {
		Path path = Paths.get("test");
		Person temp = new Person("陈钰琪", 19, "411x2x19xx1x2x6x1x", 20000d);
		
		ObjectOutputStream out = 
			new ObjectOutputStream(new FileOutputStream(path.toFile()));
		out.writeObject(temp);
		
		ObjectInputStream in = 
			new ObjectInputStream(new FileInputStream(path.toFile()));
		Person readObject = (Person)in.readObject();
		System.out.println(readObject);
	}
}
class Person implements Serializable{
	
	private static final long serialVersionUID = -485348963313276072L;
	private String name;
	private Integer age;
	private String id;
	private Double money;
	
	public Person(String name, Integer age, String id, Double money) {
		this.name = name;
		this.age = age;
		this.id = id;
		this.money = money;
	}
	
	// getter和setter
	
	@Override
	public String toString() {
		return "Person [name=" + name + ", 
			age=" + age + ", id=" + id + ", money=" + money + "]";
	}
}
```



### 版本号

版本号是为了控制对象的版本而存在的，版本号一致才可认为可以进行对应的序列化和反序列化。比如我们使用上面的代码把一个Person写入temp文件了，然后修改`serialVersionUID = 485348963313276072L;`，会发现爆出`java.io.InvalidClassException`。



### 单例的处理

```java
public final class MySingleton implements Serializable{ 
	private static final long serialVersionUID = 1L;
	
	private MySingleton() { } 
	
	private static final MySingleton INSTANCE = new MySingleton(); 
	
	public static MySingleton getInstance() { return INSTANCE; } 
	
	private Object readResolve() throws ObjectStreamException { 
	    // instead of the object we're on, return the class variable INSTANCE 
		return INSTANCE; 
  	}
}
```

反序列化”组装”一个新对象时，就会自动调用这个`readResolve`方法来返回我们指定好的对象了，单例规则也就得到了保证。



### 序列化与类加载器

```java
protected Class<?> resolveClass(ObjectStreamClass desc)
    throws IOException, ClassNotFoundException {
    String name = desc.getName();
    try {
        return Class.forName(name, false, latestUserDefinedLoader();());
    } catch (ClassNotFoundException ex) {
        Class<?> cl = primClasses.get(name);
        if (cl != null) {
            return cl;
        } else {
            throw ex;
        }
    }
}
/**
latestUserDefinedLoader();
Returns first non-privileged class loader on the stack (excluding reflection generated frames) or the extension class loader if only class loaded by the boot class loader and extension class loader are found on the stack. This method is also called via reflection by the following RMI-IIOP class: com.sun.corba.se.internal.util.JDKClassLoader This method should not be removed or its signature changed without corresponding modifications to the above class.

返回堆栈上的第一个非特权类加载器（不包括反射生成的帧）或扩展类加载器（如果堆栈上只找到引导类加载器和扩展类加载器加载的类）。此方法也由以下RMI-IIOP类通过反射调用：com.sun.corba.se.internal.util.JDKClassLoader在没有对上述类进行相应修改的情况下，不应删除此方法或更改其签名。
**/
```

默认序列化的时候采用的是一个XXX类加载器。这个类加载器是笔者并不知道是什么。不过可以确定它使用的不是当前类的加载器（本人踩过坑）。如果需要采用自定义的类加载器，比如当前类的类加载器，可以继承`ObjectInputStream`来覆盖`resolveClass(ObjectStreamClass desc)`方法。

```java
public class CurrentThreadLoaderObjectInputStream extends ObjectInputStream {
    public CurrentThreadLoaderObjectInputStream(InputStream in) throws IOException {
        super(in);
    }
    protected CurrentThreadLoaderObjectInputStream() 
        throws IOException, SecurityException {
    }
    // 覆盖原有的加载类的机制，从当前线程的loader进行加载
    @Override
    protected Class<?> resolveClass(ObjectStreamClass desc)
            throws IOException, ClassNotFoundException {
        String name = desc.getName();
        return Thread.currentThread().getContextClassLoader().loadClass(name);
    }

}
```



## 反射
通常能够分析类能力的程序称为反射（reflective）。程序员通过反射库（包括Class类，Constructor类等）完成业务功能，便是使用到了反射技术。反射的主要功能是在程序运行时分析类。
运用反射技术来实现功能一般分为四步：获得Class对象、获得构造器、获得类的实例、运行。反射技术甚至可以获得私有信息。



### 获得Class对象
Class对象是在类加载时由Java虚拟机创建的封装某类型信息的对象。有三种获得方式：

```java
public class TestClazz {
	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws ClassNotFoundException {
		// 1、通过Object类的getClass()方法
		TestClazz tc = new TestClazz();
		Class clazz1 = tc.getClass();
		
		// 2、通过类的静态class属性。
		Class clazz2 = TestClazz.class;
		
		// 3、通过Class类中的方法构造。这种可拓展性更强，根本不需要知道类型，通过字符串就能获得。
		// 但是也正是这个原因，可能发生ClassNotFoundException（main函数的此异常与前两者无关）
		Class clazz3 = Class.forName("com.first.TestClazz");
	}
}
```



### 获得构造器

- 获得所有公共权限的构造方法（包括继承的）：`clazz.getConstructors();`

```java
public class TestClazz {
	public TestClazz(String msg) { System.out.println(msg); }
	public TestClazz() { }
	private TestClazz(int i) { System.out.println(i); }
	
	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		
		Constructor[] constructors = clazz.getConstructors();
		for(Constructor e:constructors)
			System.out.println(e);
		/*Console：
		 	public com.first.TestClazz(java.lang.String)
			public com.first.TestClazz()		 */
	}
}
```
- 获得指定参数的公共构造器（可以获得继承的）：`clazz.getConstructor(Class... parameterTypes);`
```java
public class TestClazz {
	public TestClazz(String msg) { System.out.println(msg); }
	public TestClazz() { }
	private TestClazz(int i) { System.out.println(i); }
	
	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		
		Constructor con1 = clazz.getConstructor();
		Constructor con2 = clazz.getConstructor(String.class);
		System.out.println(con1);
		System.out.println(con2);
		/*Console:
		  		public com.first.TestClazz()
				public com.first.TestClazz(java.lang.String)		*/
	}
}
```
- 获得所有的构造器（不包括继承的）：`clazz.getDeclaredConstructors();`
```java
public class TestClazz {
	private TestClazz(int i) { System.out.println(i); }
	public TestClazz(String msg) { System.out.println(msg); }
	public TestClazz() { }
	
	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");

		Constructor []cons = clazz.getDeclaredConstructors();
		for(Constructor e : cons)
			System.out.println(e);
		/*Console:
		  		public com.first.TestClazz()
				public com.first.TestClazz(java.lang.String)
				private com.first.TestClazz(int)		*/
	}
}
```
- 获得指定的构造器（不包括继承的）：`clazz.getDeclaredConstructor((Class... parameterTypes);`
```java
public class TestClazz {
	private TestClazz(int i) { System.out.println(i); }
	public TestClazz(String msg) { System.out.println(msg); }
	public TestClazz() { }
	
	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		
		Constructor con = clazz.getDeclaredConstructor(int.class);
		System.out.println(con);
		/*Console:
		  		private com.first.TestClazz(int)			*/
	}
}
```


### 创建对象

- 通过公共构造器创建对象：`constructor.newInstance(Class... parameterTypes)`方法。传入的参数类型和构造器的参数类型一致。
```java
public class TestClazz {
	private TestClazz(int i) { System.out.println(i); }
	public TestClazz(String msg) { System.out.println(msg); }
	public TestClazz() { }
	
	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		Constructor con1 = clazz.getConstructor(String.class);
		Object obj1 = con1.newInstance("HelloWorld!");
		System.out.println(obj1);
		
		System.out.println("-----------------------");
		Constructor con2 = clazz.getConstructor();
		Object obj2 = con2.newInstance();
		System.out.println(obj2);
		/*Console:
		 		HelloWorld!
				com.first.TestClazz@7852e922
				-----------------------
				com.first.TestClazz@4e25154f			 */
	}
}
```

- 通过当前类不可访问的构造器创建对象：`constructor.setAccessible(true);`
```java
public class TestClazz {
	private TestClazz(int i) { System.out.println(i); }
	public TestClazz(String msg) { System.out.println(msg); }
	TestClazz() { }

	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws   {
		Class clazz = Class.forName("com.first.TestClazz");
		Constructor constructor = clazz.getDeclaredConstructor(int.class);

		//在本例中不写这句也能执行，因为被反射的类和当前类是同一个类，private的构造方法是可以被访问的
		//如果某个构造器是当前类不可访问的，此方法可以使其变成可访问的类型
		constructor.setAccessible(true);
		Object object = constructor.newInstance(1);
		System.out.println(object);
		/*Console:
		 		1
		 		com.first.TestClazz@7852e922		 */
	}
}
```
#### 快速获得对象
如果被反射的类有被当前类可访问的无参构造函数，可以直接使用`clazz.newInstance();`

```java
public class TestClazz {
	private TestClazz(int i) { System.out.println(i); }
	public TestClazz(String msg) { System.out.println(msg); }
	TestClazz() { }
	
	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		
		Object object = clazz.newInstance();
		System.out.println(object);
		/*Console:
		 		com.first.TestClazz@7852e922		 */
	}
}
```



### 获得成员变量

- 获得所有公共类型的属性（包括继承的）：`clazz.getFields();`
```java
public class TestClazz {
	public String msg;
	private int i;

	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		Field[] fields = clazz.getFields();
		for(Field f : fields)
			System.out.println(f);
		/*Console:
		  		public java.lang.String com.first.TestClazz.msg			*/
	}
}
```

- 获得指定公共类型的属性（包括继承的）：`clazz.getField(String name);`
```java
public class TestClazz {
	public String msg;
	private int i;

	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		Field field = clazz.getField("msg");
		System.out.println(field);
		/*Console:
		  		public java.lang.String com.first.TestClazz.msg			*/
	}
}
```

- 获得所有的属性（不包括继承的）：`clazz.getDeclaredFields();`
```java
public class TestClazz {
	public String msg;
	private int i;

	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		Field []fields = clazz.getDeclaredFields();
		for(Field f : fields)
			System.out.println(f);
		/*Console:
		  		public java.lang.String com.first.TestClazz.msg
				private int com.first.TestClazz.i						*/
	}
}
```

- 获得指定的属性（不包括继承的）：`clazz.getDeclaredField(String name);`
```java
public class TestClazz {
	public String msg;
	private int i;

	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		Field field = clazz.getDeclaredField("i");
		System.out.println(field);
		/*Console:
				private int com.first.TestClazz.i						*/
	}
}
```

#### 设置成员变量的值

- 设置可访问的变量的值：`field.set(Object obj, Object value);`
```java
public class TestClazz {
	public String msg;
	private int i;
	TestClazz() { }
	
	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		TestClazz object = (TestClazz)clazz.newInstance();
		Field field = clazz.getDeclaredField("msg");
		field.set(object, "HelloWorld!");
		System.out.println(object.msg);
		/*Console:
				HelloWorld!						*/
	}
}
```

- 设置不可访问的变量的值：`field.setAccessible(true);`
```java
public class TestClazz {
	public String msg;
	private int i;
	TestClazz() { }
	
	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		TestClazz object = (TestClazz)clazz.newInstance();
		Field field = clazz.getDeclaredField("i");
		// 在本例中不写这句也能执行，因为被反射的类和当前类是同一个类，private的属性是可以被访问的
		// 如果某个属性是当前类不可访问的，此方法可以使其变成可访问的类型
		field.setAccessible(true);
		field.set(object, 65535);
		System.out.println(object.i);
		/*Console:
				65535						*/
	}
}
```



### 获得成员方法

- 获得所有公共的方法（包括继承的和构造器）：`clazz.getMethods();`
```java
public class TestClazz {
	private int getI() { return i; }
	TestClazz() { }
	
	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		Method[] methods = clazz.getMethods();
		for(Method m : methods)
			System.out.println(m);
		/*Console:
				public static void com.first.TestClazz.main(java.lang.String[]) throws java.lang.Exception
				public final void java.lang.Object.wait() throws java.lang.InterruptedException
				public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
				public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
				public boolean java.lang.Object.equals(java.lang.Object)
				public java.lang.String java.lang.Object.toString()
				public native int java.lang.Object.hashCode()
				public final native java.lang.Class java.lang.Object.getClass()
				public final native void java.lang.Object.notify()
				public final native void java.lang.Object.notifyAll()						*/
	}
}
```

- 获得指定的公共方法（包括继承的，不包括构造器）：`clazz.getMethod(String name, Class<?>... parameterTypes);`
```java
public class TestClazz{
	public void show(String msg) { System.out.println(msg); }
	TestClazz() { }
	
	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		Method m = clazz.getMethod("show", String.class);
		System.out.println(m);
		/*Console:
				public void com.first.TestClazz.show(java.lang.String)					*/
	}
}
```

- 获得所有的方法（不包括继承的，不包括构造器）：`clazz.getDeclaredMethods();`
```java
public class TestClazz{
	private int getI() { return i; }
	TestClazz() { }
	
	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		Method[] methods = clazz.getDeclaredMethods();
		for(Method m : methods)
			System.out.println(m);
		/*Console:
				public static void com.first.TestClazz.main(java.lang.String[]) throws java.lang.Exception
				private int com.first.TestClazz.getI()					*/
	}
}
```

- 获得指定的方法（不包括继承的，不包括构造器）：`clazz.getDeclaredMethod(String name, Class<?>... parameterTypes);`

```java
public class TestClazz{
	private void show(String msg) { System.out.println(msg); }
	TestClazz() { }
	
	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		Method m = clazz.getDeclaredMethod("show", String.class);
		System.out.println(m);
		/*Console:
				public void com.first.TestClazz.show(java.lang.String)							*/
	}
}
```

#### 执行方法

- 执行公共方法：`method.invoke(Object obj, Object... args);`
```java
public class TestClazz{
	public void show(String msg) { System.out.println(msg); }
	TestClazz() { }

	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		Object object = clazz.newInstance();
		Method m = clazz.getDeclaredMethod("show", String.class);
		m.invoke(object, "HelloWorld!");
		/*Console:
				HelloWorld					*/
	}
}
```

- 执行私有方法：`method.invoke(Object obj, Object... args);`
```java
public class TestClazz{
	private void show(String msg) { System.out.println(msg); }
	TestClazz() { }

	@SuppressWarnings("rawtypes")
	public static void main(String[] args) throws Exception {
		Class clazz = Class.forName("com.first.TestClazz");
		Object object = clazz.newInstance();
		Method m = clazz.getDeclaredMethod("show", String.class);
		m.setAccessible(true);
		m.invoke(object, "HelloWorld!");
		/*Console:
				HelloWorld					*/
	}
}
```



## 正则表达式
正则表达式（regular expression）描述了一种字符串匹配的模式（pattern），可以用来检查一个串是否含有某种子串、将匹配的子串替换或者从某个串中取出符合某个条件的子串等。但是上面的叙述，对于之前没有接触过正则表达式的人还是很迷，我们打个比方，有一串字符：`123xyz234`和一个模式：`*^*`，我们假设`*`表示任意长度的由数字组成的字符串，`^`表示任意长度的由英文字符表示的字符串，那么我们就可以说这个字符串能匹配上这个模式。因为`123`可以匹配上`*`，`xyz`可以匹配上`^`，`234`可以匹配上`*`。同样的，假如我们再有一个模式：`*^`，我们用这个模式在字符串                                            中提取，可以提取出来：`123`、`123x`、`123xyz`、`234` 等等，但是不能提取出来`z234`、`123xyz234`。因为我们能提取出来的都是符合这个模式的，这个模式就是正则表达式。

不同的语言在正则表达式上的语法是有差距的，但是相同点远远大于不同点。我们在这使用Java语言中正则表达式。不过正则表达式的语法非常难记，在这也是举例出一些常用的语法，具体使用还是得查API文档。

**最简单的正则表达式**

在我们刚才的举例中可以看出正则表达式其实就是一种匹配规则，那么每个字符串也都是一种匹配规则。比如下面的`split()`方法是按照正则表达式把字符串分割，我们传入的一个字符串就是一个正则表达式。
```java
public class RegTest {
	public static void main(String[] args) {
		
		String testStr = "1234567890";
		/**
		 * 	Splits this string around matches of the given regular expression. 
		 */
		String[] split = testStr.split("67");
		System.out.println(Arrays.deepToString(split));
		
	}
}
```

**和正则表达式有关的类**

虽然上个例子我们并没有使用到正则表达式相关的类，但这并不表明正则表达式就是一个字符串这么简单，在`split()`方法的内部还是调用了正则表达式相关的方法。
- `Pattern`类：Pattern 对象是一个正则表达式的编译表示。Pattern 类没有公共构造方法。要创建一个 Pattern 对象，你必须首先调用其公共静态编译方法，它返回一个 Pattern 对象。该方法接受一个正则表达式作为它的第一个参数。
- `Matcher`类：Matcher 对象是对输入字符串进行解释和匹配操作的引擎。与Pattern 类一样，Matcher 也没有公共构造方法。你需要调用 Pattern 对象的 matcher 方法来获得一个 Matcher 对象。
- `PatternSyntaxException`：PatternSyntaxException 是一个非强制异常类，它表示一个正则表达式模式中的语法错误。

现在我们使用正则表达式有关的类来完成上面的例子：
```java
public class RegTest {
	public static void main(String[] args) {
		
		String testStr = "1234567890";
		Pattern pattern = Pattern.compile("67");
		String[] split = pattern.split(testStr);
		System.out.println(Arrays.deepToString(split));
		
	}
}
```
PatternSyntaxException 式一个异常类，但是不强制处理正则表达式的异常，所以这里可加可不加。Matcher的用法请继续看下去。

**匹配**

在上面我们测试的是正则表达式的分割作用，但是这并不是一个很好的学习正则表达式的例子。所以下面我们将采用`匹配方法`来学习正则表达式，先给一个小例子。
```java
public class RegTest {
	public static void main(String[] args) {
		
		String testStr = "1234567890";
		boolean matches = Pattern.matches("67", testStr);
		System.out.println(matches);
	}
}
```
这里的输出结果肯定是`false`了，因为`12345`和`890`都无法在模式中被匹配。

**字符类**

- `[abc]` ：a、b 或 c（简单类） 
- `[^abc]` ：任何字符，除了 a、b 或 c（否定） 
- `[a-zA-Z]` ：a 到 z 或 A 到 Z，两头的字母包括在内（范围） 
- `[a-d[m-p]]` ：a 到 d 或 m 到 p：[a-dm-p]（并集） 
- `[a-z&&[def]]` ：d、e 或 f（交集） 
- `[a-z&&[^bc]]` ：a 到 z，除了 b 和 c：[ad-z]（减去） 
- `[a-z&&[^m-p]]` ：a 到 z，而非 m 到 p：[a-lq-z]（减去） 
```java
public class RegTest {
	public static void main(String[] args) {
		
		String testStr = "123";
		Pattern.matches("[123][123][123]", testStr); 	//true
		Pattern.matches("[^123][123][123]", testStr); 	//false
		
	}
}
```

**预定义字符类**

- `.` ：任何字符（与行结束符可能匹配也可能不匹配） 
- `\d` ：数字：[0-9] 
- `\D` ：非数字： [^0-9] 
- `\s` ：空白字符：[ \t\n\x0B\f\r] 
- `\S` ：非空白字符：[^\s] 
- `\w` ：单词字符：[a-zA-Z_0-9] 
- `\W` ：非单词字符：[^\w] 
```java
public class RegTest {
	public static void main(String[] args) {
		
		String testStr = "123";
		System.out.println(Pattern.matches("\\d\\d\\d", testStr)); 	//true
		System.out.println(Pattern.matches("\\w\\w\\w", testStr));	//true
	}
}
```

**数量词** 

- `X?`： X存在一次或一次也没有 
- `X*` ：X存在零次或多次 
- `X+` ：X存在一次或多次 
- `X{n}` ：X存在恰好 n 次 
- `X{n,}` ：X存在至少 n 次 
- `X{n,m}` ：X存在至少 n 次，但是不超过 m 次 
```java
public class RegTest {
	public static void main(String[] args) {
		
		String testStr = "123";
		boolean matches = Pattern.matches("[123]{3}", testStr); //true
		System.out.println(matches);
		
	}
}
```

**查找子串**

查找子串需要使用到Pattern和Mather
`[flid=1415279, ffid=BK-2898-20180922-A, frtt=20180922210700, frlt=20180923000300][flid=1417032, ffid=OD-689-20180923-D, fatt=2401, stat=BOR, ista=BOR]`
```java
public class RegTest {
	public static final String FFID = "((ffid=){1})\\w{2}-\\w{3,6}-\\d{8}-\\w";
	public static void main(String[] args) {
		String str = "[flid=1415279, ffid=BK-2898-20180922-A, frtt=20180922210700, frlt=20180923000300][flid=1417032, ffid=OD-689-20180923-D, fatt=2401, stat=BOR, ista=BOR]";
		Pattern pattern = Pattern.compile(FFID);
		Matcher matcher = pattern.matcher(str);
		//循环找出全部的匹配子串
		while(matcher.find()) {
			System.out.println(matcher.group(0));
		}
	}
}
```

**group方法**

group是针对正则表达式中的`()`来说的，`group(0)`就是指的整个串，`group(1)`指的是第一个括号里的东西，`group(2)`指的第二个括号里的东西。 

```java
public static void main(String[] args) throws Exception {
    String regEx = "count(\\d+)(df)";
    String s = "count000dfdfsdffaaaa1";
    Pattern pat = Pattern.compile(regEx);
    Matcher mat = pat.matcher(s);
    if (mat.find()) {
        System.out.println(mat.group());
        System.out.println(mat.group(0));
        System.out.println(mat.group(1));
        System.out.println(mat.group(2));
    }
}
/*
    count000df
    count000df
    000
    df
*/
```

> 