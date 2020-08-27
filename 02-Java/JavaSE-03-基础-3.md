## 泛型

### 泛型类

```java
// 此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
// 在实例化泛型类时，必须指定T的具体类型
public class Generic<T> {
    // key这个成员变量的类型为T，T的类型由外部指定  
    private T key;

    // 泛型构造方法形参key的类型也为T，T的类型由外部指定
    public Generic(T key) { this.key = key; }

    // 泛型方法getKey的返回值类型为T，T的类型由外部指定
    public T getKey(){ return key; }
    
    public static void main(String[] args) {
    	// 泛型的类型参数只能是类类型（包括自定义类），不能是简单类型
    	// 传入的实参类型需与泛型的类型参数类型相同，即为Integer
    	Generic<Integer> genericInteger = new Generic<Integer>(123456);

    	// 传入的实参类型需与泛型的类型参数类型相同，即为String
    	Generic<String> genericString = new Generic<String>("key_vlaue");
    	System.out.println("key is " + genericInteger.getKey().getClass());
    	System.out.println("key is " + genericString.getKey().getClass());
	}
}
```



### 泛型接口的实现

```java
// 定义一个泛型接口
public interface Generator<T> {
    public T next();
}
```

**当实现泛型接口的类，未传入泛型实参时：**

```java
/**
 * 未传入泛型实参时，与泛型类的定义相同，在声明类的时候，需将泛型的声明也一起加到类中
 * 即：class FruitGenerator<T> implements Generator<T>
 * 如果不声明泛型，如：class FruitGenerator implements Generator<T>，会报错："Unknown class"
 */
class FruitGenerator<T> implements Generator<T>{
    @Override
    public T next() { return null; }
}
```

**当实现泛型接口的类，传入泛型实参时：**

```java
/**
 * 传入泛型实参时：
 * 定义一个生产器实现这个接口，虽然我们只创建了一个泛型接口Generator<T>
 * 但是我们可以为T传入无数个实参，形成无数种类型的Generator接口。
 * 在实现类实现泛型接口时，如已将泛型类型传入实参类型，则所有使用泛型的地方都要替换成传入的实参类型
 * 即：Generator<T>，public T next(); 中的的T都要替换成传入的String类型。
 */
public class FruitGenerator implements Generator<String> {
    private String[] fruits = new String[]{"Apple", "Banana", "Pear"};

    @Override
    public String next() {
        Random rand = new Random();
        return fruits[rand.nextInt(3)];
    }
}
```



### 泛型方法

泛型类，是在实例化类的时候指明泛型的具体类型；**泛型方法，是在调用方法的时候指明泛型的具体类型** 。

```java
/**
 * 泛型方法的基本介绍
 * @param tClass 传入的泛型实参
 * @return T 返回值为T类型
 * 说明：
 *   1）public 与 返回值中间<T>非常重要，可以理解为声明此方法为泛型方法。
 *   2）只有声明了<T>的方法才是泛型方法，泛型类中的使用了泛型的成员方法并不是泛型方法。
 *   3）<T>表明该方法将使用泛型类型T，此时才可以在方法中使用泛型类型T。
 *   4）与泛型类的定义一样，此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型。
 *	 5）如果泛型类也声明了一个T，泛型方法的T会覆盖泛型类的T
 */
public <T> T genericMethod(Class<T> tClass)
    		throws InstantiationException, IllegalAccessException{
        T instance = tClass.newInstance();
        return instance;
}
```

#### 泛型"方法"的详细举例

```java
public class GenericTest {
   // 这个类是个泛型类，在上面已经介绍过
   public class Generic<T> {
        private T key;

        public Generic(T key) { this.key = key; }

        // 我想说的其实是这个，虽然在方法中使用了泛型，但是这并不是一个泛型方法。
        // 这只是类中一个普通的成员方法，只不过他的返回值是在声明泛型类已经声明过的泛型。
        // 所以在这个方法中才可以继续使用 T 这个泛型。
        public T getKey(){ return key; }

        /**
         * 这个方法显然是有问题的，在编译器会给我们提示这样的错误信息"cannot reslove symbol E"
         * 因为在类的声明中并未声明泛型E，所以在使用E做形参和返回值类型时，编译器会无法识别。
            public E setKey(E key){
                 this.key = keu
            }
        */
    }

    /** 
     * 这才是一个真正的泛型方法。
     * 首先在public与返回值之间的<T>必不可少，这表明这是一个泛型方法，并且声明了一个泛型T
     * 这个T可以出现在这个泛型方法的任意位置.
     * 泛型的数量也可以为任意多个 
     *    如：public <T,K> K showKeyName(Generic<T> container){
     *        ...
     *        }
     */
    public <T> T showKeyName(Generic<T> container){
        System.out.println("container key :" + container.getKey());
        // 当然这个例子举的不太合适，只是为了说明泛型方法的特性。
        T test = container.getKey();
        return test;
    }

    /**
     * 这个方法是有问题的，编译器会为我们提示错误信息："UnKnown class 'E' "
     * 虽然我们声明了<T>,也表明了这是一个可以处理泛型的类型的泛型方法。
     * 但是只声明了泛型类型T，并未声明泛型类型E，因此编译器并不知道该如何处理E这个类型。
        public <T> T showKeyName(Generic<E> container) {
            ...
        }
     */
}
```

#### 类中的泛型方法

```java
public class Generic {
    class Fruit {
        public String toString() { return "fruit"; }
    }
    class Apple extends Fruit {
        public String toString() { return "apple"; }
    }
    class Person {
        public String toString() { return "Person"; }
    }

    class GenerateTest<T> {
        // 方法中的T和类上声明的T一致
        public void show_1(T t){
            System.out.println(t.toString());
        }

        // 泛型类中声明了一个泛型方法，使用泛型E，泛型E可以为任意类型。可以类型与T相同，也可以不同。
        // 由于泛型方法在声明的时候会声明泛型<E>，因此即使在泛型类中并未声明泛型，编译器也能够正确识
        // 别泛型方法中识别的泛型。
        public <E> void show_3(E t) {
            System.out.println(t.toString());
        }

        // 泛型类中声明了一个泛型方法，使用泛型T，这个T是一种全新的类型，即泛型方法的T覆盖了泛型类的T
        // 编译器会报警告：The type parameter T is hiding the type T
        public <T> void show_2(T t) {
            System.out.println(t.toString());
        }
    }

    public static void main(String[] args) {
        Apple apple = new Generic().new Apple();
        Person person = new Generic().new Person();

        GenerateTest<Fruit> generateTest = new Generic().new GenerateTest<Fruit>();
        
        // apple是Fruit的子类，所以这里可以，此时实际上是多态的性质
        generateTest.show_1(apple);
        // 编译器会报错，因为泛型类型实参指定的是Fruit，而传入的实参类是Person
        // generateTest.show_1(person);

        // 使用这两个方法都可以成功
        generateTest.show_2(apple);
        generateTest.show_2(person);

        // 使用这两个方法也都可以成功
        generateTest.show_3(apple);
        generateTest.show_3(person);
    }
}
```

 

### 静态方法与泛型

类中的静态方法使用泛型：**静态方法无法访问类上定义的泛型；如果静态方法操作的引用数据类型不确定的时候，必须要将泛型定义在方法上。**

```java
public class StaticGenerator<T> {
    /**
     * 如果在类中定义使用泛型的静态方法，需要添加额外的泛型声明（将这个方法定义成泛型方法）
     * 即使静态方法要使用泛型类中已经声明过的泛型也不可以。
     * 如：public static void show(T t){..},此时编译器会提示错误信息：
     *    "StaticGenerator cannot be refrenced from static context"
     */
    public static <T> void show(T t){

    }
}
```

为什么要有这样的限定呢？其实是因为Java中的泛型是在编译期实现的，也就是说如果我们有一个泛型类，如`Stream<T>`，它里面有一个静态方法`of(T ... t)`，我们不能使用`Stream<String>.of(...)`，因为压根就没有`Stream<String>`这个类型，只能使用`Stream.of()`，这样一来我们就无法给参数T实例化一个类型。也就是说类中的静态属性和静态方法中的T没有办法被实例化的。



### 泛型通配符

泛型类和泛型方法都是定义了一个类或者方法的模板。能构造不同类型的属性或不同参数类型的方法。我们假设：

```java
abstract class Fruit {
    abstract int getPrice();
    abstract int getNumber();
}
class Apple extends Fruit {
    @Override
    int getPrice() {
        return 1;
    }
    @Override
    int getNumber() {
        return 10;
    }
}
class Orange extends Fruit {
    @Override
    int getPrice() {
        return 2;
    }
    @Override
    int getNumber() {
        return 20;
    }
}
public class Goods<T> {
    private T t;
    private int number;
}
class Shopping{
    void payMoney(Goods<Fruit> fruitGoods){ }
    public static void main(String[] args) {
        Shopping shopping = new Shopping();
        shopping.payMoney(new Goods<Apple>());
    }
}
```

上面代码的33行会出错。本来我们的想法是计算应该付多少钱，所以使用了抽象类`Fruit`。但是虽然`Apple`是`Fruit`的子类，但是`Goods<Apple>`却不是`Goods<Fruit>`的子类，所以这里会报错。怎么解决呢？便是使用通配符`?`，将上面代码的30行换为`void payMoney(Goods<?> fruitGoods){ }`便不会报错。

通配符是一种类型，一种可以代表多种类型的类型。它和`T`不一样，`T`是为了声明此类有一个类型参数需要去实例化，而`?`是一种实例化`T`的类型。

#### 上下界通配符

下面代码就是“上界通配符（Upper Bounds Wildcards）”：

```java
Goods<? extends Fruit>
```

上界通配符表示传入的只能是Fruit的子类及自己。

相对应的，“下界通配符（Lower Bounds Wildcards）”：

```java
Plate<? super Fruit>
```

下界通配符表示传入的只能是Fruit的父类及自己。

#### 通配符在实例T时的限制

假如我们有两个方法：

```java
public class Goods<T> {
    private T t;
    private int number;
    public void set(T t){ }
    public T get(){ return null; }
    public static void main(String[] args) {
        Goods<? extends Fruit> fruitGoods = new Goods<>();
        fruitGoods.set(new Orange());	// 这段代码会出错
        Fruit fruit = fruitGoods.get();
    }
}
```

我们分析一下，第7行代码中`? extends Fruit`替换了`T`，这个时候编译器知道`T`是`Fruit`或者其子类，但是它却不知道具体是哪一个，假如它编译器设置为了`Apple`，那么传入`Orange`对象在编译期是不会出错的，因为`Orange`是符合`? extends Fruit`的，但是我们知道`Apple`的引用根本无法引用`Orange`的对象，就会出错。所以编译器就拒绝传递它不确定的类型（null 除外，因为任何对象都能引用null）。

第9行不会出错，因为返回的类型是`Fruit`或者其子类，这样我们在接收的时候使用`Fruit`接收就行了。

这里有人会问，那为什么不会将`private T t`的`T`设置为`Fruit`呢，其实这是由于在第9行中的`Fruit`是我们认为确定的，对于编译器来说这是不可变得，而`private T t`不是我们能确定的，它是一个不确定的值，编译器为了防止出现我们刚才的问题拒绝传递不确定的值。

```java
public class Goods<T> {
    private T t;
    private int number;
    public void set(T t){ }
    public T get(){ return null; }
    public static void main(String[] args) {
        Goods<? super Fruit> fruitGoods = new Goods<>();
        fruitGoods.set(new Orange());
        Object fruit = fruitGoods.get();
    }
}
```

如果使用了下界通配符，我们就可以传入`Orange`对象，因为`T`会被实例化为`Fruit`或其父类，而无论是`Fruit`还是其父类都能接收`Orange`对象，但是不能接受`Fruit`的父类。

而在返回的时候只能用`Object`接收，因为在`super`定义的类型区间内只有`Object`可以接收所有的类型。



### 泛型的缺陷

#### 运行时类型查询只适用于原始类型

```java
if(a instanceof Pair<String>);	 	// error
Pair<String> p = (Pair<String>) a;	// error
ArrayList<String>.class == ArrayList<Integer> // true
```

#### **不能创建一个确切的泛型类型的数组**

也就是说下面的这个例子是不可以的：

```java
List<String>[] ls = new ArrayList<String>[10];
```

而使用通配符创建泛型数组是可以的，如下面这个例子：

```java
List<?>[] ls = new ArrayList<?>[10];
```

这样也是可以的：

```java
List<String>[] ls = new ArrayList[10];
```

下面使用[Sun](http://docs.oracle.com/javase/tutorial/extra/generics/fineprint.html)[的一篇文档](http://docs.oracle.com/javase/tutorial/extra/generics/fineprint.html)的一个例子来说明这个问题：

```java
List<String>[] lsa = new List<String>[10]; // Not really allowed.    
Object o = lsa;
Object[] oa = (Object[]) o;    
List<Integer> li = new ArrayList<Integer>();    
li.add(new Integer(3));    
oa[1] = li; // Unsound, but passes run time store check    
String s = lsa[1].get(0); // Run-time error: ClassCastException.

/**
这种情况下，由于JVM泛型的擦除机制，在运行时JVM是不知道泛型信息的，所以可以给oa[1]赋上一个ArrayList而不会出现异常，但是在取出数据的时候却要做一次类型转换，所以就会出现ClassCastException，如果可以进行泛型数组的声明，上面说的这种情况在编译期将不会出现任何的警告和错误，只有在运行时才会出错。而对泛型数组的声明进行限制，对于这样的情况，可以在编译期提示代码有类型安全问题，比没有任何提示要强很多。
*/
```

下面采用通配符的方式是被允许的:**数组的类型不可以是类型变量，除非是采用通配符的方式**，因为对于通配符的方式，最后取出数据是要做显式的类型转换的。

```java
List<?>[] lsa = new List<?>[10]; // OK, array of unbounded wildcard type.    
Object o = lsa;
Object[] oa = (Object[]) o;
List<Integer> li = new ArrayList<Integer>();
li.add(new Integer(3));
oa[1] = li; // Correct.
Integer i = (Integer) lsa[1].get(0); // OK
```

#### 可变参参数警告

```java
public static <T> void addAll(Collection<T> coll, T... ts) {
	for (T : ts)
		coll. add(t);
}
```

应该记得，实际上参数ts是一个数组，包含提供的所有实参，现在考虑一下调用：

```java
Collection<Pair<String>> table ...;
Pair<String> pairl = ...;
Pair<String> pair2 = ...;
addAll(table, pairl, pair2);
```

为了调用这个方法，Java虚拟机必须建立一个`Pair <String>`数组,这就违反了前面的规则。不过，对于这种情况,规则有所放松，你只会得到一个警告，而不是错误。可以采用两种方法来抑制这个警告。一种方法是为包含addA1调用的方法增加注解`@Suppress Warnings("unchecked")`或者在Java SE7中，还可以用`@Safe Varargs`直接标注`addAll`方法：

```java
@SafeVarargs
public static <T> void addAl1 (Collection<T> coll, T... ts)
```


现在就可以提供泛型类型来调用这个方法了。对于只需要读取参数数组元素的所有方法，都可以使用这个注解。

#### 不能实例化类型变量

不能使用像`new T(...)`，`new T[...]`或`T.class`这样的表达式中的类型变量。例如，下面的`Pair<T>`构造器就是非法的：

```java
public Pair { 	first= new T(); second = new T();	}		// Error
```

##### 反射解决

```java
public class Pair<T> {
    public T first;
    public T second;
    public Pair(T first, T second) {
        this.first = first;
        this.second = second;
    }
    public static <T> Pair<T> makePair(Class<T> cla){
        try {
            return new Pair<>(cla.newInstance(), cla.newInstance());
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

#### 不能实例化类型数组

```java
//    T [] ts = new T[100];     error
```

##### 反射解决

```java
public class Pair<T> {
    T[] ts;
    public Pair(T[] ts) {	this.ts = ts;	}
    public static <T> Pair<T> makePair(T... a) {
        return new Pair<>((T[]) Array.newInstance(a.getClass().getComponentType(), 2));
    }
    public static void main(String[] args) {
        Pair<String> p = Pair.makePair(value -> new String[value]);
    }
}
```

#### 数组作为类的私有实例域

如果数组仅仅作为一个类的私有实例域，就可以将这个数组声明为`Object []`，并且在取元素时进行类型转换。 例如ArrayList类这样实现：

```java
transient Object[] elementData;

public E get(int index) {
    rangeCheck(index);
    return elementData(index);
}
@SuppressWarnings("unchecked")
E elementData(int index) {
    return (E) elementData[index];
}

public E set(int index, E element) {
    rangeCheck(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```

#### 不能抛出或捕获泛型类的实例

既不能抛出也不能捕获泛型类对象。 实际上，甚至泛型类扩展`Throwable`都是不合法的。例如，以下定义就不能正常编译：

```java
public class Problem <T> extends Exception { /*...*/ } // Error can't extend Throwable
```

catch 子句中不能使用类型变量。 例如，以下方法将不能编译：

```java
public static <T extends Throwable> void doWork(Class<T> t) {
	try	{
		// do work
	} catch (T e) { // Error can't catch type variable
		// ...
	}
}
```

不过，在异常规范中使用类型变量是允许的。以下方法是合法的：

```java
public static <T extends Throwable> void doWork(T t) throws T { // OK
	try { 
		// do work
	} catch (Throwable realCause) {
		throw t ;
	}
}
```

#### 可以消除对受查异常的检查

Java 异常处理的一个基本原则是，必须为所有受查异常提供一个处理器。不过可以利用泛型消除这个限制。关键在于以下方法：

```java
@SuppressWamings("unchecked")
public static <T extends Throwable> void throwAs(Throwable e) throws T {
	throw (T) e ;
}
```

假设这个方法包含在类`Block`中 ， 如果调用`Block.<RuntimeException> throwAs(t);`，编译器就会认为t是一个非受查异常。以下代码会把所有异常都转换为编译器所认为的非受查异常：

```java
try {
	// do work
} catch (Throwable t) {
	Block.<RuntimeException> throwAs(t);
}
```

下面把这个代码包装在一个抽象类中。用户可以覆盖body方法来提供一个具体的动作。调用toThread时，会得到`Thread`类的一个对象，它的`run`方法不会介意受查异常。

```java
public abstract class Block {
	public abstract void body() throws Exception;
	public Thread toThread() {
        return new Thread(){
			public void run(){
                try {
                    body();
                } catch (Throwable t) {
                    Block.<RuntimeException> throwAs(t);
                }
            }
        };
    }

    @SuppressWamings("unchecked")
	public static <T extends Throwable> void throwAs(Throwable e) throws T {
		throw (T) e;
    }
}
```

例如，以下程序运行了一个线程，它会拋出一个受查异常。

```java
public class Test {
	public static void main(String []args) {
		new Block() {
			public void body() throws Exception {
				Scanner in = new Scanner(new File("ququx"), "UTF-8");
				while(in.hasNext())
					System.out.println(in.next());
			}
		}.toThread().start();
	}
}
```

运行这个程序时，会得到一个栈轨迹，其中包含一个`FileNotFoundException`。这有什么意义呢？正常情况下，你必须捕获线程run方法中的所有受查异常 ， 把它们“包装”到非受查异常中，因为`run`方法声明为不抛出任何受查异常。

不过在这里并没有做这种“包装”。我们只是抛出异常，并“哄骗”编译器，让它认为这不是一个受查异常。通过使用泛型类、 擦除和`@SuppressWamings`注解，就能消除 Java 类型系统的部分基本限制。



### <T extends Comparable<? super T>>

```java
 public class Test	{
     //	第一种声明：简单，灵活性低
     public static <T extends Comparable<T>> void mySort1(List<T> list)	{
         Collections.sort(list);
     }
     //	第二种声明：复杂，灵活性高
     public static <T extends Comparable<? super T>> void mySort2(List<T> list) {
         Collections.sort(list);
     }
     public static void main(String[] args) {
     	//主函数中将分别创建Animal和Dog两个序列，然后调用排序方法对其进行测试
　　  	//main函数中具体的两个版本代码将在下面具体展示
     }
 }

 class Animal implements Comparable<Animal> {
     protected int age;
     public Animal(int age) {
         this.age = age;
     }
     // 使用年龄与另一实例比较大小
     @Override
     public int compareTo(Animal other) {
         return this.age - other.age;
     }
 }

 class Dog extends Animal {
     public Dog(int age) {
         super(age);
     }
 }
```

**对mySort1()进行测试，main方法代码如下所示** 

```java
// 创建一个 Animal List
List<Animal> animals = new ArrayList<Animal>();
animals.add(new Animal(25));
animals.add(new Dog(35));

// 创建一个 Dog List
List<Dog> dogs = new ArrayList<Dog>();
dogs.add(new Dog(5));
dogs.add(new Dog(18));

// 测试 mySort1() 方法
mySort1(animals);
mySort1(dogs);		// error
```

 结果编译出错，报错信息为： 

```java
The method mySort1(List<T>) in the type TypeParameterTest is not applicable for the arguments (List<Dog>)
```

如果传入的是`List<Animal>`程序将正常执行，因为`Animal`实现了接口`Comparable<Animal>`。但是，如果传入的参数是`List<Dog>`程序将报错，因为`Dog`类中没有实现接口`Comparable<Dog>`，它只从`Animal`继承了一个`Comparable<Animal>`接口。

**对mySort12()进行测试，main方法代码如下所示** 

```java
// 创建一个 Animal List
List<Animal> animals = new ArrayList<Animal>();
animals.add(new Animal(25));
animals.add(new Dog(35));

// 创建一个 Dog List
List<Dog> dogs = new ArrayList<Dog>();
dogs.add(new Dog(5));
dogs.add(new Dog(18));

// 测试  mySort2() 方法
mySort2(animals);
mySort2(dogs);
```

这时候我们发现该程序可以正常运行。它不但能够接受`Animal implements Comparable<Animal>`这样的参数，也可以接收`Dog implements Comparable<Animal>`这样的参数。 

- 在添加animals时，T是Animal，Comparable的泛型是Animal，`Animal super Animal`成立。
- 在添加dogs时，T是Dog，Comparable的泛型是Animal，`Animal super Dog`成立。



## ThreadLocal

### ThreadLocal使用

ThreadLocal是一个线程局部变量，我们都知道全局变量和局部变量的区别，拿Java举例就是定义在类中的是全局的变量，各个方法中都能访问得到（静态方法不能获得实例属性），而局部变量定义在方法中，只能在方法内访问。那线程局部变量（ThreadLocal）就是每个线程都会有一个局部变量，独立于变量的初始化副本，而各个副本是通过线程唯一标识相关联的。

```java
public class TaskThread extends Thread {
	private UniqueThreadIdGenerator t;

	public TaskThread(String threadName, UniqueThreadIdGenerator t) {
		this.setName(threadName);
		this.t = t;
	}

	@Override
	public void run() {
		for (int i = 0; i < 4; i++) {
			try {
				int value = t.getUniqueId();
				System.out.println("thread[ " + Thread.currentThread().getName() + 
                	" ] --> uniqueId[ " + value + " ]");
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}

	public static void main(String[] args) {
		UniqueThreadIdGenerator uniqueThreadId = new UniqueThreadIdGenerator();
		// 为每个线程生成一个唯一的局部标识
		TaskThread t1 = new TaskThread("custom-thread-1", uniqueThreadId);
		TaskThread t2 = new TaskThread("custom-thread-2", uniqueThreadId);
		TaskThread t3 = new TaskThread("custom-thread-3", uniqueThreadId);
		t1.start();
		t2.start();
		t3.start();
	}
}

class UniqueThreadIdGenerator {

	// 线程局部整型变量
	private final ThreadLocal<Integer> uniqueNum = new ThreadLocal<Integer>() {
		@Override
		protected Integer initialValue() {
			return 0;
		}
	};

	// 变量值
	public int getUniqueId() {
		uniqueNum.set(uniqueNum.get() + 1);
		return uniqueNum.get();
	}
}

// thread[ custom-thread-2 ] --> uniqueId[ 1 ]
// thread[ custom-thread-1 ] --> uniqueId[ 1 ]
// thread[ custom-thread-3 ] --> uniqueId[ 1 ]
// thread[ custom-thread-1 ] --> uniqueId[ 2 ]
// thread[ custom-thread-2 ] --> uniqueId[ 2 ]
// thread[ custom-thread-1 ] --> uniqueId[ 3 ]
// thread[ custom-thread-3 ] --> uniqueId[ 2 ]
// thread[ custom-thread-1 ] --> uniqueId[ 4 ]
// thread[ custom-thread-2 ] --> uniqueId[ 3 ]
// thread[ custom-thread-3 ] --> uniqueId[ 3 ]
// thread[ custom-thread-2 ] --> uniqueId[ 4 ]
// thread[ custom-thread-3 ] --> uniqueId[ 4 ]
// 每个线程之间的uniqueId是互不干扰的
```



### ThreadLocal源码分析

每个线程内部有一个`ThreadLocalMap`，`get()`的时候就是获得当前线程的`ThreadLocalMap`，并且将当前`ThreadLocal`对象传入`map.getEntry(this);`

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/be8d7461-5ec5-41c7-be7f-4b7c30f454a1"></div>
#### get()

```java
// ThreadLocal.java
public T get() {
    Thread t = Thread.currentThread();
    // 拿到每个线程内部的ThreadLocalMap
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        // this指的是这个ThreadLocal对象，每个ThreadLocalMap可以有多个ThreadLocal对象
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

ThreadLocalMap getMap(Thread t) {
	return t.threadLocals;
}

private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}

// 默认初始化为null
protected T initialValue() {
    return null;
}

void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}

// ThreadLocalMap.java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}
```

#### set()

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

#### remove()

```java
// 从当前线程的ThreadLocalMap中删除当前的ThreadLocal
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);
}
```



### ThreadLocalMap

ThreadLocalMap是ThreadLocal的内部类，没有实现Map接口，用独立的方式实现了Map的功能，其内部的Entry也独立实现。

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/ad6080da-3850-463c-94a3-00f13be71c3d"></div>
在ThreadLocalMap中，也是用Entry来保存K-V结构数据的。但是Entry中key只能是ThreadLocal对象，这点被Entry的构造方法已经限定死了。

Entry继承自WeakReference（弱引用，生命周期只能存活到下次GC前），但只有Key是弱引用类型的，Value并非弱引用。

```java
// Entry.java
static class Entry extends WeakReference<ThreadLocal> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal k, Object v) {
        super(k);
        value = v;
    }
}
```

#### Hash冲突怎么解决

和HashMap的最大的不同在于，ThreadLocalMap结构非常简单，没有next引用，也就是说ThreadLocalMap中解决Hash冲突的方式并非链表的方式，而是采用线性探测的方式，所谓线性探测，就是根据初始key的hashcode值确定元素在table数组中的位置，如果发现这个位置上已经有其他key值的元素被占用，则利用固定的算法寻找一定步长的下个位置，依次判断，直至找到能够存放的位置。

ThreadLocalMap解决Hash冲突的方式就是简单的步长加1或减1，寻找下一个相邻的位置。

```java
/**
 * Increment i modulo len.
 */
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}

/**
 * Decrement i modulo len.
 */
private static int prevIndex(int i, int len) {
    return ((i - 1 >= 0) ? i - 1 : len - 1);
}
```

显然ThreadLocalMap采用线性探测的方式解决Hash冲突的效率很低，如果有大量不同的ThreadLocal对象放入map中时发送冲突，或者发生二次冲突，则效率很低。**所以这里引出的良好建议是：每个线程只存一个变量，需要多个变量 这个时候需要把这些对象封装成变量对象**。



### 内存泄漏

ThreadLocal在ThreadLocalMap中是以一个弱引用身份被Entry中的Key引用的，因此如果ThreadLocal没有外部强引用来引用它，那么ThreadLocal会在下次JVM垃圾收集时被回收。这个时候就会出现Entry中Key已经被回收，出现一个null Key的情况，外部读取ThreadLocalMap中的元素是无法通过null Key来找到Value的。因此如果当前线程的生命周期很长（比如线程池中的线程），一直存在，那么其内部的ThreadLocalMap对象也一直生存下来，这些null key就存在一条强引用链的关系一直存在：Thread --> ThreadLocalMap-->Entry-->Value，这条强引用链会导致Entry不会回收，Value也不会回收，但Entry中的Key却已经被回收的情况，造成内存泄漏。

但是JVM团队已经考虑到这样的情况，并做了一些措施来保证ThreadLocal尽量不会内存泄漏：在ThreadLocal的get()、set()、remove()方法调用的时候会清除掉线程ThreadLocalMap中所有Entry中Key为null的Value，并将整个Entry设置为null，利于下次内存回收。

ThreadLocal的get()方法在调用map.getEntry(this)时，内部会判断key是否为null

```java
private Entry getEntry(ThreadLocal key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

private Entry getEntryAfterMiss(ThreadLocal key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        ThreadLocal k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);		// 清除空结点的方法
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}

private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;

    // expunge entry at staleSlot（意思是，删除value，设置为null便于下次回收）
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;

    // Rehash until we encounter null
    // 将当前Entry删除后，会继续循环往下检查是否有key为null的节点，如果有则一并删除，防止内存泄漏。
    Entry e;
    int i;
    for (i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal k = e.get();
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;

                // Unlike Knuth 6.4 Algorithm R, we must scan until
                // null because multiple entries could have been stale.
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
```

但这样也并不能保证ThreadLocal不会发生内存泄漏，例如：

- 使用static的ThreadLocal，延长了ThreadLocal的生命周期，可能导致的内存泄漏。
- 分配使用了ThreadLocal又不再调用get()、set()、remove()方法，那么就会导致内存泄漏

#### 为什么使用弱引用？

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/5ea4a817-8e0f-4ee0-9602-4fdd20fac2bf" /></div>
有一些文章说内存泄漏是因为key的弱引用，但是实际上key使用弱引用不仅不是内存泄漏的原因，反而可以减少内存泄漏的发生。官方文档的说法：

> To help deal with very large and long-lived usages, the hash table entries use WeakReferences for keys.
> 为了处理非常大和生命周期非常长的线程，哈希表使用弱引用作为 key。

下面我们分两种情况讨论：

- key 使用强引用：引用的ThreadLocal的对象被回收了，但是ThreadLocalMap还持有ThreadLocal的强引用，如果没有手动删除，ThreadLocal不会被回收，导致Entry内存泄漏。
- key 使用弱引用：引用的ThreadLocal的对象被回收了，由于ThreadLocalMap持有ThreadLocal的弱引用，即使没有手动删除，ThreadLocal也会被回收。value在下一次ThreadLocalMap调用set，get，remove的时候会被清除。

比较两种情况，我们可以发现：由于ThreadLocalMap的生命周期跟Thread一样长，如果都没有手动删除对应key，都会导致内存泄漏，但是使用弱引用可以多一层保障：弱引用ThreadLocal不会内存泄漏，对应的value在下一次ThreadLocalMap调用set，get，remove的时候会被清除。

因此，ThreadLocal内存泄漏的根源是：由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key的value就会导致内存泄漏，而不是因为弱引用。

所以：**每次使用完ThreadLocal，都调用它的remove()方法，清除数据。**尤其是在使用线程池的情况下，没有及时清理ThreadLocal，不仅是内存泄漏的问题，更严重的是可能导致业务逻辑出现问题。所以，使用ThreadLocal就跟加锁完要解锁一样，用完就清理。



## 匿名内部类不能修改外部变量

想必这个问题也曾经困扰过很多人，在讨论这个问题之前，先看下面这段代码：

```java
public class Outer {
    public Outer test(int x) {
        int y = 100;
        return new Outer() {
            int z = y;
        };
    }
}
```

<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/2ab13184-1697-4430-a45a-2ca7b1044fff" /></div>
从反编译的结果来看，我们会创建一个内部类，并且执行构造方法。我们再反编译一个`Outer$1.class`

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/448f0e22-4474-438e-a487-d5ac9b613a12" /></div>
从构造方法我们可以看出了这个构造方法有两个参数，一个是外部类的引用，一个是一个int。

<div align="center"><img width="40%" src="http://q0l9qvfyx.bkt.clouddn.com/981c3249-77bf-402a-aade-cc5c3814ed9a" /></div>
从字段结果中可以看到，有三个字段：z、final y和Outer。也就是说我们虽然没有把y声明为final的，在传递到内部类中的时候也会是final的，这时候就知道为什么在内部类中不能修改y了，因为内部类中的y实际上是外部类数据的一份拷贝，这份拷贝在传递到内部类后会变成一个final的值。而且即使不是final的，我们把内部类的值进行修改实际上对外部类也没有任何影响。



## 深入分析JDK动态代理

```java
public interface Subject {
    void request();
}
```

```java
public class RealSubject implements Subject {
    @Override
    public void request() {
        System.out.println("request is called");
    }
}
```

```java
public class DynamicSubject implements InvocationHandler {
    private Object object;
    public DynamicSubject(Object object) {
        this.object = object;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("before calling: "+ method);
        method.invoke(this.object, args);
        System.out.println("after calling: " + method);
        return null;
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        RealSubject rs = new RealSubject();
        InvocationHandler ds = new DynamicSubject(rs);
        Class<? extends RealSubject> aClass = rs.getClass();

        Subject subject = (Subject)Proxy.newProxyInstance(
            aClass.getClassLoader(), aClass.getInterfaces(), ds);
        subject.request();
        System.out.println(subject.getClass());
        System.out.println(subject.getClass().getSuperclass());
        System.out.println(subject.getClass().getSuperclass().getSuperclass());
        System.out.println(Arrays.toString(subject.getClass().getInterfaces()));
    }
}
/*
    before calling: public abstract void two.jvm.bytecodes.Subject.request()
    request is called
    after calling: public abstract void two.jvm.bytecodes.Subject.request()
    class com.sun.proxy.$Proxy0
    class java.lang.reflect.Proxy
    class java.lang.Object
    [interface two.jvm.bytecodes.Subject]
*/
```

从上面的输我们可以看到`subject`的类型是运行期动态生成的字节码对象。

返回字节码对象的方法是Client的第7行，所以我们进入此方法。

```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,
		InvocationHandler h)	throws IllegalArgumentException {
    Objects.requireNonNull(h);

    final Class<?>[] intfs = interfaces.clone();
    final SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
    }

    /*
     * Look up or generate the designated proxy class.
     */
    Class<?> cl = getProxyClass0(loader, intfs);

    /*
     * Invoke its constructor with the designated invocation handler.
     */
   // ...
}
```

在此方法中，我们会发现，获得生成Class对象的是滴14行。

```java
private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
        proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());

private static Class<?> getProxyClass0(ClassLoader loader,
                                       Class<?>... interfaces) {
    // 限制接口数量不能多于65535，因为在字节码中使用两个字节表示接口数量
    if (interfaces.length > 65535) {
        throw new IllegalArgumentException("interface limit exceeded");
    }

    // If the proxy class defined by the given loader implementing
    // the given interfaces exists, this will simply return the cached copy;
    // otherwise, it will create the proxy class via the ProxyClassFactory
    return proxyClassCache.get(loader, interfaces);
}
```

进入`WeakCache.get()`方法：

```java
public V get(K key, P parameter) {
    Objects.requireNonNull(parameter);

    expungeStaleEntries();

    Object cacheKey = CacheKey.valueOf(key, refQueue);

    // lazily install the 2nd level valuesMap for the particular cacheKey
    ConcurrentMap<Object, Supplier<V>> valuesMap = map.get(cacheKey);
    if (valuesMap == null) {
        ConcurrentMap<Object, Supplier<V>> oldValuesMap
            = map.putIfAbsent(cacheKey,
                              valuesMap = new ConcurrentHashMap<>());
        if (oldValuesMap != null) {
            valuesMap = oldValuesMap;
        }
    }

    // create subKey and retrieve the possible Supplier<V> stored by that
    // subKey from valuesMap
    Object subKey = Objects.requireNonNull(subKeyFactory.apply(key, parameter));
    Supplier<V> supplier = valuesMap.get(subKey);
    Factory factory = null;

    while (true) {
        if (supplier != null) {
            // supplier might be a Factory or a CacheValue<V> instance
            V value = supplier.get();
            if (value != null) {
                return value;
            }
        }
        // else no supplier in cache
        // or a supplier that returned null (could be a cleared CacheValue
        // or a Factory that wasn't successful in installing the CacheValue)

        // lazily construct a Factory
        if (factory == null) {
            factory = new Factory(key, parameter, subKey, valuesMap);
        }

        if (supplier == null) {
            supplier = valuesMap.putIfAbsent(subKey, factory);
            if (supplier == null) {
                // successfully installed Factory
                supplier = factory;
            }
            // else retry with winning supplier
        } else {
            if (valuesMap.replace(subKey, supplier, factory)) {
                // successfully replaced
                // cleared CacheEntry / unsuccessful Factory
                // with our Factory
                supplier = factory;
            } else {
                // retry with current supplier
                supplier = valuesMap.get(subKey);
            }
        }
    }
}
```

在这个方法中，我们可以发现最终返回`value`的是第30行。但`value`的值是通过函数式接口`Supplier`获得的，而`supplier`的赋值是在第46行完成的而`factory`对象是在第39行被构建的。

```java
private final BiFunction<K, P, V> valueFactory;

public WeakCache(BiFunction<K, P, ?> subKeyFactory,
                 BiFunction<K, P, V> valueFactory) {
    this.subKeyFactory = Objects.requireNonNull(subKeyFactory);
    this.valueFactory = Objects.requireNonNull(valueFactory);
}

// Factory是WeakCache的一个内部类
private final class Factory implements Supplier<V> {

    private final K key;
    private final P parameter;
    private final Object subKey;
    private final ConcurrentMap<Object, Supplier<V>> valuesMap;

    Factory(K key, P parameter, Object subKey,
            ConcurrentMap<Object, Supplier<V>> valuesMap) {
        this.key = key;
        this.parameter = parameter;
        this.subKey = subKey;
        this.valuesMap = valuesMap;
    }

    @Override
    public synchronized V get() { // serialize access
        // re-check
        Supplier<V> supplier = valuesMap.get(subKey);
        if (supplier != this) {
            // something changed while we were waiting:
            // might be that we were replaced by a CacheValue
            // or were removed because of failure ->
            // return null to signal WeakCache.get() to retry
            // the loop
            return null;
        }
        // else still us (supplier == this)

        // create new value
        V value = null;
        try {
            value = Objects.requireNonNull(valueFactory.apply(key, parameter));
        } finally {
            if (value == null) { // remove us on failure
                valuesMap.remove(subKey, this);
            }
        }
        // the only path to reach here is with non-null value
        assert value != null;

        // wrap value with CacheValue (WeakReference)
        CacheValue<V> cacheValue = new CacheValue<>(value);

        // put into reverseMap
        reverseMap.put(cacheValue, Boolean.TRUE);

        // try replacing us with CacheValue (this should always succeed)
        if (!valuesMap.replace(subKey, this, cacheValue)) {
            throw new AssertionError("Should not reach here");
        }

        // successfully replaced us with new CacheValue -> return the value
        // wrapped by it
        return value;
    }
}
```

而`Factory`是`Supplier`的一个实现类，对于Factory的`key`和`parameter`，我们传入的是`ClassLoader loader`和`Class<?>... interfaces`。在`get()`方法中给`value`赋值的是第33行的`apply()`方法，而`valueFactory`是一个函数式接口`BiFunction`，此接口传入两个参数，获得一个结果。现在我们传入的就是`loader`和`interfaces`，而`apply`的具体实现是在构建`WeakCache`传入的。所以我们再回到`Proxy`类。

```java
private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
    proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());


/**
 * A factory function that generates, defines and returns the proxy class given
 * the ClassLoader and array of interfaces.
 */
private static final class ProxyClassFactory
    implements BiFunction<ClassLoader, Class<?>[], Class<?>> {
    // prefix for all proxy class names
    private static final String proxyClassNamePrefix = "$Proxy";

    // next number to use for generation of unique proxy class names
    private static final AtomicLong nextUniqueNumber = new AtomicLong();

    @Override
    public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

        Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
        for (Class<?> intf : interfaces) {
            /*
             * Verify that the class loader resolves the name of this
             * interface to the same Class object.
             */
            Class<?> interfaceClass = null;
            try {
                interfaceClass = Class.forName(intf.getName(), false, loader);
            } catch (ClassNotFoundException e) {
            }
            if (interfaceClass != intf) {
                throw new IllegalArgumentException(
                    intf + " is not visible from class loader");
            }
            /*
             * Verify that the Class object actually represents an
             * interface.
             */
            if (!interfaceClass.isInterface()) {
                throw new IllegalArgumentException(
                    interfaceClass.getName() + " is not an interface");
            }
            /*
             * Verify that this interface is not a duplicate.
             */
            if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
                throw new IllegalArgumentException(
                    "repeated interface: " + interfaceClass.getName());
            }
        }

        String proxyPkg = null;     // package to define proxy class in
        int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

        /*
         * Record the package of a non-public proxy interface so that the
         * proxy class will be defined in the same package.  Verify that
         * all non-public proxy interfaces are in the same package.
         */
        for (Class<?> intf : interfaces) {
            int flags = intf.getModifiers();
            if (!Modifier.isPublic(flags)) {
                accessFlags = Modifier.FINAL;
                String name = intf.getName();
                int n = name.lastIndexOf('.');
                String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                if (proxyPkg == null) {
                    proxyPkg = pkg;
                } else if (!pkg.equals(proxyPkg)) {
                    throw new IllegalArgumentException(
                        "non-public interfaces from different packages");
                }
            }
        }

        if (proxyPkg == null) {
            // if no non-public proxy interfaces, use com.sun.proxy package
            proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
        }

        /*
         * Choose a name for the proxy class to generate.
         */
        long num = nextUniqueNumber.getAndIncrement();
        String proxyName = proxyPkg + proxyClassNamePrefix + num;

        /*
         * Generate the specified proxy class.
         */
        byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
            proxyName, interfaces, accessFlags);
        try {
            return defineClass0(loader, proxyName,
                                proxyClassFile, 0, proxyClassFile.length);
        } catch (ClassFormatError e) {
            /*
             * A ClassFormatError here means that (barring bugs in the
             * proxy class generation code) there was some other
             * invalid aspect of the arguments supplied to the proxy
             * class creation (such as virtual machine limitations
             * exceeded).
             */
            throw new IllegalArgumentException(e.toString());
        }
    }
}
```

再第90行是真正的生成字节码对象的字节数组的地方。

```java
private static final boolean saveGeneratedFiles = (Boolean)AccessController.doPrivileged(new GetBooleanAction("sun.misc.ProxyGenerator.saveGeneratedFiles"));

public static byte[] generateProxyClass(final String var0, Class<?>[] var1, int var2) {
    ProxyGenerator var3 = new ProxyGenerator(var0, var1, var2);
    final byte[] var4 = var3.generateClassFile();
    if (saveGeneratedFiles) {
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
                try {
                    int var1 = var0.lastIndexOf(46);
                    Path var2;
                    if (var1 > 0) {
                        Path var3 = Paths.get(var0.substring(0, var1).replace('.', File.separatorChar));
                        Files.createDirectories(var3);
                        var2 = var3.resolve(var0.substring(var1 + 1, var0.length()) + ".class");
                    } else {
                        var2 = Paths.get(var0 + ".class");
                    }

                    Files.write(var2, var4, new OpenOption[0]);
                    return null;
                } catch (IOException var4x) {
                    throw new InternalError("I/O exception saving generated file: " + var4x);
                }
            }
        });
    }
    return var4;
}
```

可以看见在第6行有个开关，这个开关为`true`会将字节数组写入文件中。所以我们`Client`的`main`方法中加入：

```java
System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
```

这样在根目录的`com/sun/proxy`文件下就会生成一个class文件，在IDEA中双击点开：

```java
public final class $Proxy0 extends Proxy implements Subject {
    private static Method m1;
    private static Method m2;
    private static Method m3;
    private static Method m0;

    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final void request() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m3 = Class.forName("two.jvm.bytecodes.Subject").getMethod("request");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```

我们会发现这个类继承了`Proxy`类，实现了`Subject`。然后从第33行便可以发现代理对象的`request`执行的便是`InvocationHandler`的`invoke`方法。传入的方法是`two.jvm.bytecodes.Subject`类中的`request`方法，也就是最终执行到了`DynamicSubject`中的invoke方法：

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    System.out.println("before calling: "+ method);
    method.invoke(this.object, args);
    System.out.println("after calling: " + method);
    return null;
}
```


