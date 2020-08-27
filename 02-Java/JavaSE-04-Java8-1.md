## 初识Lambda

在Java中，我们只能传递和返回值，而无法将函数作为参数传递给一个方法，也无法声明返回一个函数的方法。所以在代码中，我们不能传递一个行为，只能将行为包装成为一个对象来进行传递。比如下面的Swing：

```java
public static void main(String[] args) {
    JFrame jFrame = new JFrame("My JFrame");
    JButton jButton = new JButton("JButton");
    jButton.addActionListener(new ActionListener() {
        @Override
        public void actionPerformed(ActionEvent e) {
            System.out.println("Button Pressed! ");
        }
    });
    jFrame.add(jButton);
    jFrame.pack();
    jFrame.setVisible(true);
    jFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
}
```

对于上面的代码，我们实际上不需要`new ActionListener()`、`actionPerformed`等等，我们只需要sout这个行为。传统的匿名内部类却必须需要构建一个对象传入。而Lambda表达式便可以直接的传递一个行为进去。

```java
public static void main(String[] args) {
    JFrame jFrame = new JFrame("My JFrame");
    JButton jButton = new JButton("JButton");

    // 单行直接写，多行使用花括号{}括起来
    // event 全写是 ActionEvent envent，即全写：(ActionEvent event) -> sout
    // 这里只写event是因为Java的编译系统能推断出来这个地方的event就是Action Event
    jButton.addActionListener(event -> System.out.println("Button Pressed! "));
    jFrame.add(jButton);
    jFrame.pack();
    jFrame.setVisible(true);
    jFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
}
```



### 函数式接口

> 一个接口中只有一个抽象方法就被叫做抽象接口。

- 如果一个接口只有一个抽象方法，他就是一个函数式接口。
- 如果我们在接口上声明了`@FunctionalInterface`，编译期会按函数式接口的定义来要求他，如果不满足函数式接口的定义，编译期会报错。
- 如果某个接口只有一个函数式接口，即使没有加`@FunctionalInterface`，编译器也将其看成函数式接口。
- 如果某个接口覆盖了`Object`中的方法，那么此接口的抽象方法不会加一，如以下代码：

```java
@FunctionalInterface
public interface MyTest2 {
    void test();

    // 不会报错，
    @Override
    boolean equals(Object obj);
}
```

注意：在函数作为一等公民的语言中，函数被看做类型，而在Java中，Lambda表达式是对象，他们必须依附一类特别的对象类型 - 函数式接口。



### Lambda的一个小例子

```java
// 使用Lambda完成集合的遍历
public class MyTest1 {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("abc", "test", "hello");
        
        // 匿名内部类完成遍历
        list.forEach(new Consumer<String>() {
            @Override
            public void accept(String string) {
                System.out.println(string);
            }
        });
        
        // lambda expressions
        list.forEach(item -> System.out.println(item));

        // method references
        list.forEach(System.out::println);
    }
}
```



## Lambda基本语法

Lambda 表达式的语法格式如下：

```
(parameters) -> expression
或
(parameters) ->{ statements; }
```

以下是lambda表达式的重要特征：

- 可选类型声明：不需要声明参数类型，编译器可以统一识别参数值。
- 可选的参数圆括号：一个参数无需定义圆括号，但多个参数需要定义圆括号。
- 可选的大括号：如果主体包含了一个语句，就不需要使用大括号。
- 可选的返回关键字：如果主体只有一个表达式返回值则编译器会自动返回值，如果使用了大括号需要指定表达式返回了一个数值。

```java
public static void main(String args[]) {
    Java8Tester tester = new Java8Tester();

    // 类型声明
    MathOperation addition = (int a, int b) -> a + b;

    // 不用类型声明
    MathOperation subtraction = (a, b) -> a - b;

    // 大括号中的返回语句
    MathOperation multiplication = (int a, int b) -> {
        return a * b;
    };

    // 没有大括号及返回语句
    MathOperation division = (int a, int b) -> a / b;

    System.out.println("10 + 5 = " + tester.operate(10, 5, addition));
    System.out.println("10 - 5 = " + tester.operate(10, 5, subtraction));
    System.out.println("10 x 5 = " + tester.operate(10, 5, multiplication));
    System.out.println("10 / 5 = " + tester.operate(10, 5, division));

    // 不用括号
    GreetingService greetService1 = message -> System.out.println("Hello " + message);

    // 用括号
    GreetingService greetService2 = (message) -> System.out.println("Hello " + message);

    greetService1.sayMessage("Runoob");
    greetService2.sayMessage("Google");
}

interface MathOperation {
    int operation(int a, int b);
}
interface GreetingService {
    void sayMessage(String message);
}
private int operate(int a, int b, MathOperation mathOperation) {
    return mathOperation.operation(a, b);
}
```



### 变量作用域

Lambda 表达式只能引用标记了 final 的外层局部变量，这就是说不能在 Lambda 内部修改定义在域外的局部变量，否则会编译错误。

```java
public class Java8Tester {
    final static String salutation = "Hello! ";

    public static void main(String args[]) {
        GreetingService greetService1 = 
            message -> System.out.println(salutation + message);
        greetService1.sayMessage("Runoob");

        int num = 1;
        Converter<Integer, String> s = (param) -> String.valueOf(param + num);
        System.out.println(s.convert(2));

//        num = 5;   
//        此行导致上面的Lambda表达式报错，因为Lambda表达式引用的外面的变量必须是effective final的

//     在 Lambda 表达式当中不允许声明一个与局部变量同名的参数或者局部变量
        String first = "";
//        Comparator<String> comparator = (first, second) -> 
//		        Integer.compare(first.length(), second.length());  // 编译会出错
    }

    interface GreetingService {
        void sayMessage(String message);
    }
    public interface Converter<T1, T2> {
        String convert(int i);
    }
}
```



## 四个重要的函数式接口

JDK8提供了四个重要的函数式接口：`函数型接口`、`断言型接口`、`消费型接口`、`提供型接口`。



### Function<T, R>

> Represents a function that accepts one argument and produces a result. This is a functional interface whose functional method is `R apply(T t)`.

**apply(Object)**

```java
public class FunctionTest {
    private static FunctionTest functionTest = new FunctionTest();
    public static void main(String[] args) {
        // 传入的是一个行为，这个行为就是一个映射，从输入参数到返回值的映射
        System.out.println(functionTest.compute(1, integer -> integer + 1));
        System.out.println(functionTest.convert(1, integer -> String.valueOf(integer)));
    }
    int compute(int a, Function<Integer, Integer> function) {
        return function.apply(a);
    }
    public String convert(int a, Function<Integer, String> function) {
        return function.apply(a);
    }
}
```

**compose & andThen**

```java
default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
    Objects.requireNonNull(before);
    return (V v) -> apply(before.apply(v));
}
```

> Returns a composed function that first applies the before function to its input, and then applies this function to the result. If evaluation of either function throws an exception, it is relayed to the caller of the composed function.

```java
default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t) -> after.apply(apply(t));
}
```

>Returns a composed function that first applies this function to its input, and then applies the after function to the result. If evaluation of either function throws an exception, it is relayed to the caller of the composed function.

测试代码：

```java
public class FunctionTest2 {
    static FunctionTest2 functionTest2 = new FunctionTest2();
    public static void main(String[] args) {
        System.out.println(functionTest2.compute1
                           (2, value -> value * 3, value -> value * value));	// 12
        System.out.println(functionTest2.compute2
                           (2, value -> value * 3, value -> value * value));	// 36
    }
    int compute1(int a, Function<Integer, Integer> f1, Function<Integer, Integer> f2) {
        return f1.compose(f2).apply(a);
    }
    int compute2(int a, Function<Integer, Integer> f1, Function<Integer, Integer> f2){
        return f1.andThen(f2).apply(a);
    }
}
```

compose分析：

```java
// 我们先把这个扩充一下compose方法扩充一下
default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
    Objects.requireNonNull(before);
    // 我们先把这个扩充一下，这样便可以清楚的看见，先调用before的apply，再调用自身的apply
    return (V v) -> Function.this.apply(before.apply(v));
}
```

```java
// 我们再改写一下调用方
public static void main(String[] args) {
    System.out.println(functionTest2.compute1
                       (2, value -> value * 3, value -> value * value));
}

int compute1(int a, Function<Integer, Integer> f1, Function<Integer, Integer> f2) {
    System.out.println("f1: " + f1);
    System.out.println("f2: " + f2);
    Function<Integer, Integer> compose = f1.compose(f2);
    System.out.println("compose: " + compose);
    return compose.apply(a);
}

// f1: jdk8.FunctionTest2$$Lambda$1/471910020@816f27d
// f2: jdk8.FunctionTest2$$Lambda$2/303563356@87aac27
// compose: jdk8.Function$1@3e3abc88

// 我们最终应用的是apply是compose里面的apply方法
// 而apply方法是由compose方法获得的：return (V v) -> Function.this.apply(before.apply(v));
// 这个before很明显就是f2，但是Function.this是什么？compose还是f1？
```

```java
// 我们改写一个Function
public interface Function<T, R> {
    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        System.out.println(this);
        // Lambda表达式可以由内部类替换
        return new Function<V, R>() {
            @Override
            public R apply(V v) {
                System.out.println(this);
                System.out.println(Function.this);
                return Function.this.apply(before.apply(v));
            }
        };
    }
}
// jdk8.FunctionTest2$$Lambda$1/471910020@816f27d
// jdk8.Function$1@3e3abc88
// jdk8.FunctionTest2$$Lambda$1/471910020@816f27d

// 从输出中可以看到Function.this是我们的f1
// new Function的时候创建的是一个内部类，而Function.this是一个外部类对象
// 实际上，new Function的外部类对象就是第8行的this，也就是f1
```

```java
return (V v) -> apply(before.apply(v));
// 如果上面的expression形式的Lambda表达式太绕，换成下面statements形式的的就好理解了。
return (V v) -> {
    return apply(before.apply(v));
};
```

`andThen`方法的分析和`compose`一致。

#### BiFunction\<T, U, R\>

> Represents a function that accepts two arguments and produces a result. `R apply(T t, U u);`

T表示第一个输入的参数的类型，U表示第二个输入的参数的类型。R表示返回类型。

```java
public class BiFunctionTest {
    public static void main(String[] args) {
        System.out.println(compute1(2, 1, (a, b) -> a + b)); // 3
        System.out.println(compute1(2, 1, (a, b) -> a - b)); // 1
        System.out.println(compute1(2, 1, (a, b) -> a * b)); // 2
        System.out.println(compute1(2, 1, (a, b) -> a / b)); // 2

        System.out.println(compute2(2, 3, (a, b) -> a + b, a -> a * a)); // 25
    }

    static int compute1(int a, int b, BiFunction<Integer, Integer, Integer> bf) {
        return bf.apply(a, b);
    }

    static int compute2(int a, int b, 
    		BiFunction<Integer, Integer, Integer> bf, Function<Integer, Integer> f) {
        return f.apply(bf.apply(a, b));
    }
}
```

```java
public class BiFunctionTest {
    public static void main(String[] args) {
        Person p1 = new Person("zhangsan", 12);
        Person p2 = new Person("lisi", 13);
        Person p3 = new Person("wangwu", 14);

        List<Person> list = Arrays.asList(p1, p2, p3);
        // 过滤年龄等于13的
        System.out.println(getPersonList(13, list,
                (age, persons) -> persons.stream().filter(
                        person -> age == person.getAge()).collect(Collectors.toList())));
    	// 过滤年龄大于13的
        System.out.println(getPersonList(13, list,
                (age, persons) -> persons.stream().filter(
                        person -> age < person.getAge()).collect(Collectors.toList())));
        // 过滤年龄小于13的
        System.out.println(getPersonList(13, list,
                (age, persons) -> persons.stream().filter(
                        person -> age > person.getAge()).collect(Collectors.toList())));
    }

    static List<Person> getPersonList(
        int age, List<Person> persons, BiFunction<Integer, List<Person>, List<Person>> bf){
        return bf.apply(age, persons);
    }
}

class Person {
    private String name;
    private int age;
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public String getName() {   return name;    }
    public void setName(String name) {  this.name = name;   }
    public int getAge() {   return age; }
    public void setAge(int age) {   this.age = age; }
    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
// person = Person{name='wangwu', age=34}
// [Person{name='lisi', age=13}, Person{name='wangwu', age=14}]
// [Person{name='lisi', age=13}]
// [Person{name='wangwu', age=14}]
// [Person{name='zhangsan', age=12}]
```

#### BinaryOperator\<T\> extends BiFunction\<T,T,T\>

> Represents an operation upon two operands of the same type, producing a result of the same type as the operands. This is a specialization of BiFunction for the case where the operands and the result are all of the same type.

- `<T> BinaryOperator<T> minBy(Comparator<? super T> comparator)`：Returns a BinaryOperator which returns the lesser of two elements according to the specified Comparator.
- `<T> BinaryOperator<T> maxBy(Comparator<? super T> comparator)`：Returns a BinaryOperator which returns the greater of two elements according to the specified Comparator.

```java
public class BinaryOperatorTest {
    public static void main(String[] args) {
        System.out.println(getShort("hello123", "world", 
        	(a, b) -> a.length() - b.length()));
        System.out.println(getShort("hello123", "world", 
        	(a, b) -> a.charAt(0) - b.charAt(0)));
        System.out.println("--------------------");
        System.out.println(getBig("hello123", "world", 
        	(a, b) -> a.length() - b.length()));
        System.out.println(getBig("hello123", "world", 
        	(a, b) -> a.charAt(0) - b.charAt(0)));
    }

    static String getShort(String a, String b, Comparator<String> c){
        // c的结果小于0返回a，否则返回b
        return BinaryOperator.minBy(c).apply(a, b);
    }

    static String getBig(String a, String b, Comparator<String> c){
        // c的结果大于0返回a，否则返回b
        return BinaryOperator.maxBy(c).apply(a, b);
    }
}
// world
// hello123
// --------------------
// hello123
// world
```



### Predicate\<T\>

> Represents a predicate (boolean-valued function) of one argument. `boolean test(T t);`

```java
public class PredicateTest {

    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
        conditionFilter(list, item -> item % 2 == 0);
        System.out.println("-----------");
        conditionFilter(list, item -> item % 2 != 0);
        System.out.println("-----------");
        conditionFilter(list, item -> item > 5);
        System.out.println("-----------");
        conditionFilter(list, item -> true);
    }

    static void conditionFilter(List<Integer> list, Predicate<Integer> pd) {
        list.forEach(integer -> {
            if (pd.test(integer))
                System.out.println(integer);
        });
    }
}
// 2 4 6 8
// -----------
// 1 3 5 7 9
// -----------
// 6 7 8 9
// -----------
// 1 2 3 4 5 6 7 8 9
```

**与或非**

```java
public class PredicateTest {

    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
        conditionFilterAnd(list, item -> item % 2 == 0, item -> item > 5);
        System.out.println("-------------");
        conditionFilterOr(list, item -> item % 2 == 0, item -> item > 5);
        System.out.println("-------------");
        conditionFilterNegate(list, item -> item % 2 == 0);
    }

    static void conditionFilterAnd(
        	List<Integer> list, Predicate<Integer> pd1, Predicate<Integer> pd2) {
        list.forEach(integer -> {
            if (pd1.and(pd2).test(integer))
                System.out.println(integer);
        });
    }

    static void conditionFilterOr(
        	List<Integer> list, Predicate<Integer> pd1, Predicate<Integer> pd2) {
        list.forEach(integer -> {
            if (pd1.or(pd2).test(integer))
                System.out.println(integer);
        });
    }
    static void conditionFilterNegate(
        	List<Integer> list, Predicate<Integer> pd1) {
        list.forEach(integer -> {
            if (pd1.negate().test(integer))
                System.out.println(integer);
        });
    }
}
// 6 8
// -------------
// 2 4 6 7 8 9
// -------------
// 1 3 5 7 9
```



### Consumer\<T\>

> Represents an operation that accepts a single input argument and returns no result. Unlike most other functional interfaces, Consumer is expected to operate via side-effects. `void accept(T t);`

**andThen**

> Returns a composed Consumer that performs, in sequence, this operation followed by the after operation. If performing either operation throws an exception, it is relayed to the caller of the composed operation. If performing this operation throws an exception, the after operation will not be performed.

```java
public class ConsumerTest {
    public static void main(String[] args) {
        testConsumer();
        testAndThen();
    }

    public static void testConsumer() {
        Consumer<Integer> square = x -> System.out.println("print square : " + x * x);
        square.accept(2);
    }

    public static void testAndThen() {
        Consumer<Integer> consumer1 = x -> System.out.println("first x : " + x);
        Consumer<Integer> consumer2 = x -> {
            System.out.println("second x : " + x);
            throw new NullPointerException("throw exception test");
        };
        Consumer<Integer> consumer3 = x -> System.out.println("third x : " + x);
        consumer1.andThen(consumer2).andThen(consumer3).accept(1);
    }
}
/*
print square : 4
first x : 1
second x : 1
Exception in thread "main" java.lang.NullPointerException: throw exception test
	at jdk8.ConsumerTest.lambda$testAndThen$2(ConsumerTest.java:21)
	at java.util.function.Consumer.lambda$andThen$0(Consumer.java:65)
	at java.util.function.Consumer.lambda$andThen$0(Consumer.java:65)
	at jdk8.ConsumerTest.testAndThen(ConsumerTest.java:25)
	at jdk8.ConsumerTest.main(ConsumerTest.java:9)
*/
```

**Consumer的应用**

Consumer在经典应用是`java.lang.Iterable`里面的`forEach`方法。

```java
default void forEach(Consumer<? super T> action) {
    Objects.requireNonNull(action);
    for (T t : this) {
        action.accept(t);
    }
}
```

```java
Person p1 = new Person("zhangsan", 12);
Person p2 = new Person("lisi", 13);
Person p3 = new Person("wangwu", 14);

List<Person> list = Arrays.asList(p1, p2, p3);
List<Person> list2 = new ArrayList<>();

list.forEach(person -> {
    if(person.getAge() > 12)
        list2.add(person);
});
// [Person{name='lisi', age=13}, Person{name='wangwu', age=14}]
```

#### BiConsumer\<T, U\>

> Represents an operation that accepts two input arguments and returns no result. This is the two-arity specialization of Consumer. Unlike most other functional interfaces, BiConsumer is expected to operate via side-effects. `void accept(T t, U u);`

```java
Map<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
map.forEach((k, v) -> System.out.println(k + "=" + v));

// 1=one
// 2=two
// 3=three
```



### Supplier\<T\>

>Represents a supplier of results. There is no requirement that a new or distinct result be returned each time the supplier is invoked. `T get();`

``` java
public static void testConsumerToSupplier() {
    Consumer<Person> consumer = person -> {
        person.setName("wangwu");
        person.setAge(34);
    };
    Person person = new Person("lisi", 40);
    consumer.accept(person);
    System.out.println("person = " + person);
}
// person = Person{name='wangwu', age=34}
```



## Lambda不是语法糖

Lambda表达式看上去就是匿名内部类的一个语法糖，但是实际上其它的实现方式不和匿名内部类一致，匿名内部类在编译后会生成两个字节码文件，而Lambda只是在原有的类中增加一个私有的方法：

```java
public class MainLambda {
    public static void main(String[] args) {
        new Thread(() -> System.out.println("Lambda Thread run()")).start();
    }
}
```



**this引用的意义**

既然Lambda表达式不是内部类的简写，那么Lambda内部的`this`引用也就跟内部类对象没什么关系了。在Lambda表达式中`this`的意义跟在表达式外部完全一样。因此下列代码将输出四遍`Hello`。

```java
public class Hello {
    public Hello(){
        System.out.println(this);
    }
    Runnable r1 = () -> System.out.println(this);
    Runnable r2 = () -> System.out.println(toString());
    public static void main(String[] args) {
        new Hello().r1.run();
        System.out.println("-----------");
        new Hello().r2.run();
    }
    public String toString() {
        return "Hello";
    }
}
// Hello
// Hello
// -----------
// Hello
// Hello
```



## Optional

> A container object which may or may not contain a non-null value. If a value is present, isPresent() will return true and get() will return the value.
> Additional methods that depend on the presence or absence of a contained value are provided, such as orElse() (return a default value if value not present) and ifPresent() (execute a block of code if the value is present).
> This is a value-based class; use of identity-sensitive operations (including reference equality (==), identity hash code, or synchronization) on instances of Optional may have unpredictable results and should be avoided.

**Value-based Classes**

> Some classes, such as `java.util.Optional` and `java.time.LocalDateTime`, are *value-based*. Instances of a value-based class:
>
> - are final and immutable (though may contain references to mutable objects);
> - have implementations of `equals`, `hashCode`, and `toString` which are computed solely from the instance's state and not from its identity or the state of any other object or variable;
> - make no use of identity-sensitive operations such as reference equality (`==`) between instances, identity hash code of instances, or synchronization on an instances's intrinsic lock;
> - are considered equal solely based on `equals()`, not based on reference equality (`==`);
> - do not have accessible constructors, but are instead instantiated through factory methods which make no commitment as to the identity of returned instances;
> - are *freely substitutable* when equal, meaning that interchanging any two instances `x` and `y` that are equal according to `equals()` in any computation or method invocation should produce no visible change in behavior.
>
> A program may produce unpredictable results if it attempts to distinguish two references to equal values of a value-based class, whether directly via reference equality or indirectly via an appeal to synchronization, identity hashing, serialization, or any other identity-sensitive mechanism. Use of such identity-sensitive operations on instances of value-based classes may have unpredictable effects and should be avoided.

```java
public class Employee {
    private String name;
    public Employee(String name) {  this.name = name;   }
    public void setName(String name) {  this.name = name;   }
    public String getName() {   return name;    }
}
```

```java
public class Company {
    private String name;
    private List<Employee> employees;
    public Company(String name) {   this.name = name;   }

    public List<Employee> getEmployees() {  return employees;   }
    public void setEmployees(List<Employee> employees) {    this.employees = employees; }
}
```

```java
Employee e1 = new Employee("zhangsan");
Employee e2 = new Employee("lisi");

Company company = new Company("test");

List<Employee> employees = Arrays.asList(e1, e2);
company.setEmployees(employees);	// 注释掉此句，最终输出的是[]。不是null

Optional<Company> optional = Optional.ofNullable(company);
System.out.println(
    optional.map(comp -> comp.getEmployees()).orElse(Collections.emptyList()));
// [jdk8.Employee@53d8d10a, jdk8.Employee@e9e54c2]
```

需要注意，`Optional`是一个未实现序列化的类，不应当将其作为函数的参数或者类的属性。



## 方法引用

方法引用是Lambda的一个语法糖。我们可以将其看作是一个函数指针。方法引用有四种：

- 类名::静态方法名
- 对象::实例方法名
- 类名::实例方法名
- 构造方法引用

```java
public class Student {
    private String name;
    private int score;

    public Student(String name, int score) {
        this.name = name;
        this.score = score;
    }

    public String getName() {   return name;    }
    public void setName(String name) {  this.name = name;   }
    public int getScore() { return score;   }
    public void setScore(int score) {   this.score = score; }

    // 此方法违背了面向对象的设计原则，实际使用中不推荐
    public static int compareByScore1(Student s1, Student s2) {
        return s1.score - s2.score;
    }
    // 此方法违背了面向对象的设计原则，实际使用中不推荐
    public int compareByName2(Student s1, Student s2) {
        return s1.name.compareTo(s2.name);
    }
    public int compareByScore3(Student s){
        return this.score - s.score;
    }
    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", score=" + score +
                '}';
    }
}
```

```java
public class MethodReferenceTest {
    public static void main(String[] args) {
        Student s1 = new Student("zhangsan", 10);
        Student s2 = new Student("lisi", 20);
        Student s3 = new Student("wangwu", 30);
        Student s4 = new Student("zhaoliu", 40);

        List<Student> students = Arrays.asList(s1, s2, s4, s3);

        // sort方法需要一个Comparator，Comparator的抽象方法是：int compare(T o1, T o2);
        // 我们传入的参数是两个Student类型的对象，返回的是一个int型数据
//        students.sort((s1p, s2p) -> Student.compareByScore(s1p, s2p));

        // compareByScore这个方法接收两个Student类型的对象，
        // 返回一个int型数据，可以满足我们的需要，所以编译器能识别成功
        students.sort(Student::compareByScore1);

        // forEach方法需要一个Consumer，Consumer的抽象方法是：void accept(T t);
        // System.out.println(...) 满足一个参数，没有返回值的要求，所以可以被编译器识别
        students.forEach(System.out::println);
        System.out.println("----------------");

        students.sort(s1::compareByName2);
        students.forEach(System.out::println);
        System.out.println("----------------");

        // Comparator的抽象方法是：int compare(T o1, T o2);
        // 在这种写法中，第一个参数会去调用compareByScore3，将第二个参数传入
        // 如果函数式接口的抽象方法有三个以上的参数，第一个参数用于调用方法，后面的参数都会用于传入
        students.sort(Student::compareByScore3);
        students.forEach(System.out::println);
        System.out.println("----------------");

        // 调用String的无参构造方法，返回一个String对象
        System.out.println(getString(String::new));
        // 调用public String(String original) 构造方法
        System.out.println(getString("123", String::new));

    }
    static String getString(Supplier<String> s){
        return s.get() + "abc";
    }
    static String getString(String str, Function<String, String> f){
        return f.apply(str);
    }
}
// Student{name='zhangsan', score=10}
// Student{name='lisi', score=20}
// Student{name='wangwu', score=30}
// Student{name='zhaoliu', score=40}
// ----------------
// Student{name='lisi', score=20}
// Student{name='wangwu', score=30}
// Student{name='zhangsan', score=10}
// Student{name='zhaoliu', score=40}
// ----------------
// Student{name='zhangsan', score=10}
// Student{name='lisi', score=20}
// Student{name='wangwu', score=30}
// Student{name='zhaoliu', score=40}
// ----------------
// abc
// 123
```



## Lambda and Collections

由于引入了 Lambda 表达式，Java8新增了`java.util.funcion`包，里面包含常用的**函数接口**，这是Lambda表达式的基础，Java集合框架也新增部分接口，以便与Lambda表达式对接。我们等下也会介绍其中的一部分方法。

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/f78851ae-08f9-4920-b093-dd5115c4a413" /></div>


### List中的新方法

#### forEach()

该方法的签名为`void forEach(Consumer action)`，作用是对容器中的每个元素执行`action`指定的动作，其中`Consumer`是个函数接口，里面只有一个待实现方法`void accept(T t)`（后面我们会看到，这个方法叫什么根本不重要，你甚至不需要记忆它的名字）。

需求：*假设有一个字符串列表，需要打印出其中所有长度大于3的字符串.*

Java7及以前我们可以用增强的for循环实现：

```java
// 使用曾强for循环迭代
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
for(String str : list){
    if(str.length()>3)
        System.out.println(str);
}
```

现在使用`forEach()`方法结合匿名内部类，可以这样实现：

```java
// 使用forEach()结合匿名内部类迭代
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.forEach(new Consumer<String>(){
    @Override
    public void accept(String str){
        if(str.length()>3)
            System.out.println(str);
    }
});
```

上述代码调用`forEach()`方法，并使用匿名内部类实现`Comsumer`接口。到目前为止我们没看到这种设计有什么好处，但是不要忘记Lambda表达式，使用 Lambda 表达式实现如下：

```java
// 使用forEach()结合Lambda表达式迭代
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.forEach( str -> {
        if(str.length()>3)
            System.out.println(str);
    });
```

上述代码给`forEach()`方法传入一个 Lambda 表达式，我们不需要知道`accept()`方法，也不需要知道`Consumer`接口，类型推导帮我们做了一切。

#### removeIf()

该方法签名为`boolean removeIf(Predicate filter)`，作用是**删除容器中所有满足`filter`指定条件的元素**，其中`Predicate`是一个函数接口，里面只有一个待实现方法`boolean test(T t)`，同样的这个方法的名字根本不重要，因为用的时候不需要书写这个名字。

需求：*假设有一个字符串列表，需要删除其中所有长度大于3的字符串。*

我们知道如果需要在迭代过程冲对容器进行删除操作必须使用迭代器，否则会抛出`ConcurrentModificationException`，所以上述任务传统的写法是：

```java
// 使用迭代器删除列表元素
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
Iterator<String> it = list.iterator();
while(it.hasNext()) {
    if(it.next().length()>3) // 删除长度大于3的元素
        it.remove();
}
```

现在使用`removeIf()`方法结合匿名内部类，我们可是这样实现：

```java
// 使用removeIf()结合匿名名内部类实现
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.removeIf(new Predicate<String>(){ // 删除长度大于3的元素
    @Override
    public boolean test(String str){
        return str.length()>3;
    }
});
```

上述代码使用`removeIf()`方法，并使用匿名内部类实现`Precicate`接口。相信你已经想到用Lambda表达式该怎么写了：

```java
// 使用removeIf()结合Lambda表达式实现
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.removeIf(str -> str.length()>3); // 删除长度大于3的元素
```

使用Lambda表达式不需要记忆`Predicate`接口名，也不需要记忆`test()`方法名，只需要知道此处需要一个返回布尔类型的Lambda表达式就行了。

#### replaceAll()

该方法签名为`void replaceAll(UnaryOperator operator)`，作用是**对每个元素执行`operator`指定的操作，并用操作结果来替换原来的元素**。其中`UnaryOperator`是一个函数接口，里面只有一个待实现函数`T apply(T t)`。

需求：*假设有一个字符串列表，将其中所有长度大于3的元素转换成大写，其余元素不变。*

Java7及之前似乎没有优雅的办法：

```java
// 使用下标实现元素替换
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
for(int i=0; i<list.size(); i++){
    String str = list.get(i);
    if(str.length()>3)
        list.set(i, str.toUpperCase());
}
```

使用`replaceAll()`方法结合匿名内部类可以实现如下：

```java
// 使用匿名内部类实现
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.replaceAll(new UnaryOperator<String>(){
    @Override
    public String apply(String str){
        if(str.length()>3)
            return str.toUpperCase();
        return str;
    }
});
```

上述代码调用`replaceAll()`方法，并使用匿名内部类实现`UnaryOperator`接口。我们知道可以用更为简洁的Lambda表达式实现：

```java
// 使用Lambda表达式实现
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.replaceAll(str -> {
    if(str.length()>3)
        return str.toUpperCase();
    return str;
});
```

#### sort()

该方法定义在`List`接口中，方法签名为`void sort(Comparator c)`，该方法**根据`c`指定的比较规则对容器元素进行排序**。`Comparator`接口我们并不陌生，其中有一个方法`int compare(T o1, T o2)`需要实现，显然该接口是个函数接口。

需求：*假设有一个字符串列表，按照字符串长度增序对元素排序。*

由于Java7以及之前`sort()`方法在`Collections`工具类中，所以代码要这样写：

```java
// Collections.sort()方法
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
Collections.sort(list, new Comparator<String>(){
    @Override
    public int compare(String str1, String str2){
        return str1.length()-str2.length();
    }
});
```

现在可以直接使用`List.sort()方法`，结合Lambda表达式，可以这样写：

```java
// List.sort()方法结合Lambda表达式
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.sort((str1, str2) -> str1.length()-str2.length());
```



### Map中的新方法

#### forEach()

该方法签名为`void forEach(BiConsumer action)`，作用是**对`Map`中的每个映射执行`action`指定的操作**，其中`BiConsumer`是一个函数接口，里面有一个待实现方法`void accept(T t, U u)`。`BinConsumer`接口名字和`accept()`方法名字都不重要，请不要记忆他们。

需求：*假设有一个数字到对应英文单词的Map，请输出Map中的所有映射关系．*

Java7以及之前经典的代码如下：

```java
// Java7以及之前迭代Map
HashMap<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
for(Map.Entry<Integer, String> entry : map.entrySet()){
    System.out.println(entry.getKey() + "=" + entry.getValue());
}
```

使用`Map.forEach()`方法，结合匿名内部类，代码如下：

```java
// 使用forEach()结合匿名内部类迭代Map
HashMap<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
map.forEach(new BiConsumer<Integer, String>(){
    @Override
    public void accept(Integer k, String v){
        System.out.println(k + "=" + v);
    }
});
```

上述代码调用`forEach()`方法，并使用匿名内部类实现`BiConsumer`接口。当然，实际场景中没人使用匿名内部类写法，因为有Lambda表达式：

```java
// 使用forEach()结合Lambda表达式迭代Map
HashMap<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
map.forEach((k, v) -> System.out.println(k + "=" + v));
```

#### replaceAll()

该方法签名为`replaceAll(BiFunction function)`，作用是对`Map`中的每个映射执行`function`指定的操作，并用`function`的执行结果替换原来的`value`，其中`BiFunction`是一个函数接口，里面有一个待实现方法`R apply(T t, U u)`．不要被如此多的函数接口吓到，因为使用的时候根本不需要知道他们的名字．

需求：*假设有一个数字到对应英文单词的Map，请将原来映射关系中的单词都转换成大写．*

Java7以及之前经典的代码如下：

```java
// Java7以及之前替换所有Map中所有映射关系
HashMap<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
for(Map.Entry<Integer, String> entry : map.entrySet()){
    entry.setValue(entry.getValue().toUpperCase());
}
```

使用`replaceAll()`方法结合匿名内部类，实现如下：

```java
// 使用replaceAll()结合匿名内部类实现
HashMap<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
map.replaceAll(new BiFunction<Integer, String, String>(){
    @Override
    public String apply(Integer k, String v){
        return v.toUpperCase();
    }
});
```

上述代码调用`replaceAll()`方法，并使用匿名内部类实现`BiFunction`接口。更进一步的，使用Lambda表达式实现如下：

```java
// 使用replaceAll()结合Lambda表达式实现
HashMap<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
map.replaceAll((k, v) -> v.toUpperCase());
```

简洁到让人难以置信．

#### merge()

该方法签名为`merge(K key, V value, BiFunction remappingFunction)`，作用是：

1. 如果`Map`中`key`对应的映射不存在或者为`null`，则将`value`（不能是`null`）关联到`key`上；
2. 否则执行`remappingFunction`，如果执行结果非`null`则用该结果跟`key`关联，否则在`Map`中删除`key`的映射．

```java
default V merge(
    	K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    Objects.requireNonNull(value);
    V oldValue = get(key);
    V newValue = (oldValue == null) ? value :
    remappingFunction.apply(oldValue, value);
    if(newValue == null) {
        remove(key);
    } else {
        put(key, newValue);
    }
    return newValue;
}
```

`merge()`方法虽然语义有些复杂，但该方法的用方式很明确，让我们从最基本的例子开始：计算唯一的单词出现次数。在Java8之前的时候，代码非常混乱，实际的实现其实已经失去了本质层面的设计意义。

```java
Map map = new HashMap<String, Integer>();
words.forEach(word -> {
    var prev = map.get(word);
    if (prev == null) {
        map.put(word, 1);
    } else {
        map.put(word, prev + 1);
    }
});
```

在Java8中实现就优雅很多了

```java
Map map = new HashMap<String, Integer>();
words.forEach(word ->
        map.merge(word, 1, (prev, one) -> prev + one)
);
```

实际上如果给定的`key`不存在，它就变成了`put(key, value)`。如果key已经存在，我们  remappingFunction 可以选择合并的方式。这个功能是完美契机上面的场景：

- 只需返回新值即可覆盖旧值： `(old, new) -> new`
- 只需返回旧值即可保留旧值：`(old, new) -> old`
- 以某种方式合并两者，例如：`(old, new) -> old + new`
- 甚至删除旧值：`(old, new) -> null`

#### computeIfAbsent()

该方法签名为`V computeIfAbsent(K key, Function mappingFunction)`，作用是：只有在当前`Map`中**不存在`key`值的映射或映射值为`null`时**，才调用`mappingFunction`，并在`mappingFunction`执行结果非`null`时，将结果跟`key`关联。

```java
default V computeIfAbsent(
    	K key, Function<? super K, ? extends V> mappingFunction) {
    Objects.requireNonNull(mappingFunction);
    V v;
    if ((v = get(key)) == null) {
        V newValue;
        if ((newValue = mappingFunction.apply(key)) != null) {
            put(key, newValue);
            return newValue;
        }
    }
    return v;
}
```

示例代码：

```java
String ret;
Map<String, String> map = new HashMap<>();
ret = map.putIfAbsent("a", "aaa"); //ret 为"aaa", map 为 {"a":"aaa"}
ret = map.putIfAbsent("a", "bbb"); //ret 为 "aaa", map 还是 {"a":"aaa"}

map.put("b", null);
ret = map.putIfAbsent("b", "bbb"); //ret 为 "bbb", map 为 {"a":"aaa","b":"bbb"}
```

#### computeIfPresent()

该方法签名为`V computeIfPresent(K key, BiFunction remappingFunction)`，作用跟`computeIfAbsent()`相反，即，只有在当前`Map`中**存在`key`值的映射且非`null`时**，才调用`remappingFunction`，如果`remappingFunction`执行结果为`null`，则删除`key`的映射，否则使用该结果替换`key`原来的映射．

```java
default V computeIfPresent(
    	K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    V oldValue;
    if ((oldValue = get(key)) != null) {
        V newValue = remappingFunction.apply(key, oldValue);
        if (newValue != null) {
            put(key, newValue);
            return newValue;
        } else {
            remove(key);
            return null;
        }
    } else {
        return null;
    }
}
```

示例代码：

```java
String ret;
Map<String, String> map = new HashMap<>();
ret = map.computeIfPresent("a", (key, value) -> key + value); // ret null, map 为 {}
map.put("a", null); // map 为 ["a":null]
ret = map.computeIfPresent("a", (key, value) -> key + value); // ret null, map为 {"a":null}
map.put("a", "+aaa");

// ret "a+aaa", map 为 {"a":"a+aaa"}
ret = map.computeIfPresent("a", (key, value) -> key + value); 

// ret 为 null, map 为 {}，计算出的 null 把 key 删除了
ret = map.computeIfPresent("a", (key, value) -> null); 
```

#### compute()

该方法签名为`compute(K key, BiFunction remappingFunction)`，作用是把`remappingFunction`的计算结果关联到`key`上，如果计算结果为`null`，则在`Map`中删除`key`的映射．

```java
default V compute(
    	K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    V oldValue = get(key);

    V newValue = remappingFunction.apply(key, oldValue);
    if (newValue == null) {
        // delete mapping
        if (oldValue != null || containsKey(key)) {
            // something to remove
            remove(key);
            return null;
        } else {
            // nothing to do. Leave things as they were.
            return null;
        }
    } else {
        // add or replace old mapping
        put(key, newValue);
        return newValue;
    }
}
```

示例如下：

```java
String ret;
Map<String, String> map = new HashMap<>() ;
ret = map.compute("a", (key, value) -> "a" + value); // ret="anull", map={"a":"anull"}
ret = map.compute("a", (key, value) -> "a" + value); // ret="aanull", map={"a":"aanull"}
ret = map.compute("a", (key, value) -> null); // ret=null, map={}
```

