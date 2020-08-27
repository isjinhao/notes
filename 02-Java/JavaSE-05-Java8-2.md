## 初识Stream

在最开始接触到这个名词的时候，我们应该都是有一个问题，此流和IO流有什么关系？实际上它俩一点关系都没有，IO流描述的是对IO的操作如同流水线操作一样，比如我们读一个字节处理一个字节或者读一行处理一行。而Stream这个流也是这个意思，它将对数据源的处理也描述的向流水线操作一样。

一个Stream包含一个源、0个或多个中间操作和一个终止操作。由于Stream遵循惰性求值的规范，所以中间操作不会产生任何结果，只有遇到终止操作才会产生结果。

```java
// 构建源
public static void main(String[] args) {
    Stream<String> stream = Stream.of("hello", "world", "hello world");
    Stream<String> stream1 = Arrays.stream(new String[]{"hello", "world", "hello world"});
    Stream<String> stream2 = Arrays.asList("hello", "world", "hello world").stream();
}
```

```java
// 构建源
public static void main(String[] args) {
    IntStream intStream = IntStream.of(new int[]{5, 6, 7});
    intStream.forEach(System.out::println);
    System.out.println("-------------------");
    // 输出 [3,8)返回的数据
    IntStream.range(3, 8).forEach(System.out::println);
    System.out.println("-------------------");
    // 输出 [3,8]返回的数据
    IntStream.rangeClosed(3, 8).forEach(System.out::println);
}
```

```java
public static void main(String[] args) {
    List<Integer> list = Arays.asList(1, 2, 3, 4, 5, 6, 7);
    //        System.out.println(list.stream().map(new Function<Integer, Integer>() {
    //            @Override
    //            public Integer apply(Integer integer) {
    //                return integer * 2;
    //            }
    //        }).reduce(0, new BinaryOperator<Integer>() {
    //            @Override
    //            public Integer apply(Integer integer, Integer integer2) {
    //                return integer + integer2;
    //            }
    //        }));
    System.out.println(list.stream().map(i -> i * 2).reduce(0, Integer::sum));
    // map是中间过程，reduce是终止操作
}
```

```java
// 通过源构建数组和集合
public static void main(String[] args) {
    Stream<String> stream = Stream.of("hello", "world", "hello world");

    // Returns an array containing the elements of this stream,
    // using the provided generator function to allocate the returned array,
    // as well as any additional arrays that might be required for a partitioned 
    // execution or for resizing. This is a terminal operation.s
    String[] strings = stream.toArray(length -> new String[length]);

    System.out.println(Arrays.deepToString(strings));
    
//    相同的效果，但是不能在这里和上面的的代码同时出现，因为toArray是一个终止操作，stream已经终止了
//	  这里就像一个IO流被关闭了就不能再使用一样
//    List<String> collect = stream.collect(Collectors.toList());
//    System.out.println(collect);
}
```

```java
// Returns an infinite sequential unordered stream where each element is generated 
// by the provided Supplier. This is suitable for generating constant streams, 
// streams of random elements, etc.
Stream<String> generate = Stream.generate(UUID.randomUUID()::toString);
Stream<String> limit = generate.limit(3);
List<String> collect = limit.collect(Collectors.toList());
System.out.println(collect);

// Returns an infinite sequential ordered Stream produced by iterative application of a 
// function f to an initial element seed, producing a Stream consisting of seed, f(seed), 
// f(f(seed)), etc. The first element (position 0) in the Stream will be the provided seed. 
// For n > 0, the element at position n, will be the result of applying the function f to 
// the element at position n - 1.
Stream<Integer> iterate = Stream.iterate(5, (item) -> item + item);

Stream<Integer> limit1 = iterate.limit(10);
List<Integer> collect1 = limit1.collect(Collectors.toList());
System.out.println(collect1);

// [70e641b3-75d7-4561-ad54-fefe77fa6d63, 70e641b3-75d7-4561-ad54-fefe77fa6d63, 70e641b3-75d7-4561-ad54-fefe77fa6d63]
// [5, 10, 20, 40, 80, 160, 320, 640, 1280, 2560]
```



## 基础API

流具有如下特点：

1. 不存储数据。流是基于数据源的对象，它本身不存储数据元素，而是通过管道将数据源的元素传递给操作。
2. 函数式编程。流的操作不会修改数据源，例如`filter`不会将数据源中的数据删除。
3. 延迟操作。流的很多操作如filter，map等中间操作是延迟执行的，只有到终点操作才会将操作顺序执行。
4. 可以解绑。对于无限数量的流，有些操作是可以在有限的时间完成的，比如`limit(n)` 或 `findFirst()`，这些操作可是实现"短路"（Short-circuiting），访问到有限的元素后就可以返回。
5. 纯消费。流的元素只能访问一次，类似Iterator，操作没有回头路，如果你想从头重新访问流的元素，对不起，你得重新生成一个新的流。

**forEach()**

方法签名为`void forEach(Consumer action)`，作用是对容器中的每个元素执行`action`指定的动作。

```java
// 使用Stream.forEach()迭代
Stream<String> stream = Stream.of("I", "love", "you", "too");
stream.forEach(str -> System.out.println(str));
```

**filter()**

函数原型为`Stream filter(Predicate predicate)`，作用是返回一个只包含满足`predicate`条件元素的`Stream`。

```java
// 保留长度等于3的字符串
Stream<String> stream= Stream.of("I", "love", "you", "too");
stream.filter(str -> str.length()==3).forEach(str -> System.out.println(str));
```

**distinct()**

函数原型为`Stream distinct()`，作用是返回一个去除重复元素之后的`Stream`。

```java
// 去掉一个too之后的其余字符串
Stream<String> stream= Stream.of("I", "love", "you", "too", "too");
stream.distinct().forEach(str -> System.out.println(str));
```

**sorted()**

排序函数有两个，一个是用自然顺序排序，一个是使用自定义比较器排序，函数原型分别为`Stream　sorted()`和`Stream　sorted(Comparator comparator)`。

```java
// 输出按照长度升序排序后的字符串
Stream<String> stream= Stream.of("I", "love", "you", "too");
stream.sorted((str1, str2) -> str1.length()-str2.length()).forEach(System.out::println);
```

**map()**

函数原型为` Stream map(Function mapper)`，作用是返回一个对当前所有元素执行执行`mapper`之后的结果组成的`Stream`。直观的说，就是对每个元素按照某种操作进行转换，转换前后`Stream`中元素的个数不会改变，但元素的类型取决于转换之后的类型。

```java
// 输出原字符串的大写形式
Stream<String> stream　= Stream.of("I", "love", "you", "too");
stream.map(str -> str.toUpperCase()).forEach(str -> System.out.println(str));
```

**flatMap()**

函数原型为` Stream flatMap(Function> mapper)`，作用是对每个元素执行`mapper`指定的操作，并用所有`mapper`返回的`Stream`中的元素组成一个新的`Stream`作为最终返回结果。说起来太拗口，通俗的讲`flatMap()`的作用就相当于把原`stream`中的所有元素都”摊平”之后组成的`Stream`，转换前后元素的个数和类型都可能会改变。

```java
// 原来的stream中有两个元素，分别是两个List，执行flatMap()之后，将每个List都“摊平”成了一个个的数字，
// 所以会新产生一个由5个数字组成的Stream。所以最终将输出1~5这5个数字
Stream<List<Integer>> stream = Stream.of(Arrays.asList(1,2), Arrays.asList(3, 4, 5));
stream.flatMap(list -> list.stream()).forEach(i -> System.out.println(i));
```

**peek**

`peek` 操作接收的是一个 `Consumer` 函数。会按照 `Consumer` 函数提供的逻辑去消费流中的每一个元素，但是返回的流还是包含原来的流中的元素。

```java
// peek操作一般用于不想改变流中元素本身的类型或者只想操作元素的内部状态时；而map则用于改变流中元素本身
// 类型，即从元素中派生出另一种类型的操作。这是他们之间的最大区别。 
String[] arr = new String[]{"a","b","c","d"};
Arrays.stream(arr)
        .peek(System.out::println) // a,b,c,d
        .count();
```

**匹配**

```java
public boolean 	allMatch(Predicate<? super T> predicate)
public boolean 	anyMatch(Predicate<? super T> predicate)
public boolean 	noneMatch(Predicate<? super T> predicate)
```

- `allMatch`只有在所有的元素都满足断言时才返回true，否则false，流为空时总是返回true
- `anyMatch`只有在任意一个元素满足断言时就返回true，否则false
- `noneMatch`只有在所有的元素都不满足断言时才返回true，否则false

```java
System.out.println(Stream.of(1,2,3,4,5).allMatch( i -> i > 0)); // true
System.out.println(Stream.of(1,2,3,4,5).anyMatch( i -> i > 0)); // true
System.out.println(Stream.of(1,2,3,4,5).noneMatch( i -> i > 0)); // false
System.out.println(Stream.<Integer>empty().allMatch( i -> i > 0)); // true
System.out.println(Stream.<Integer>empty().anyMatch( i -> i > 0)); // false
System.out.println(Stream.<Integer>empty().noneMatch( i -> i > 0)); // true
```



## reduce

reduce操作符是及早求值操作符，接受一个元素序列为输入，反复使用某个合并操作，把序列中的元素合并成一个汇总的结果，其生成的值不是随意的，而是根据指定的计算模型。reduce方法有三个override的方法。

```java
Optional<T> reduce(BinaryOperator<T> accumulator);
T reduce(T identity, BinaryOperator<T> accumulator);
<U> U reduce(U identity, 
             BiFunction<U, ? super T, U> accumulator, 
             BinaryOperator<U> combiner);
```

**第一种形式**

```java
Optional<T> reduce(BinaryOperator<T> accumulator);
```

> Performs a reduction on the elements of this stream, using an associative accumulation function, and returns an Optional describing the reduced value, if any. This is equivalent to:
>
> ```java
>  boolean foundAny = false;
>  T result = null;
>  for (T element : this stream) {
>      if (!foundAny) {
>          foundAny = true;
>          result = element;
>      }
>      else
>          result = accumulator.apply(result, element);
>  }
>  return foundAny ? Optional.of(result) : Optional.empty();
> ```
>
> but is not constrained to execute sequentially. The accumulator function must be an associative function. This is a terminal operation.

```java
public static void main(String[] args) {
    Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10).reduce((sum, item) -> {
        System.out.println(sum);
        return sum - item;
    }).ifPresent(System.out::println);
}
// 1 -1 -4 -8 -13 -19 -26 -34 -43 -53
// 第一轮的sum是索引为0位置的数据，item是索引为1位置的数据。
// 后面轮次的sum是上一轮次计算的结果。item的索引依次加一。
```

**第二种形式**

```java
T reduce(T identity, BinaryOperator<T> accumulator);
```

> Performs a reduction on the elements of this stream, using the provided identity value and an associative accumulation function, and returns the reduced value. This is equivalent to:
>
> ```java
>  T result = identity;
>  for (T element : this stream)
>      result = accumulator.apply(result, element)
>  return result;
> ```
>
> but is not constrained to execute sequentially. The identity value must be an identity for the accumulator function. This means that for all t, accumulator.apply(identity, t) is equal to t. The accumulator function must be an associative function. This is a terminal operation.

```java
public static void main(String[] args) {
    Integer reduce = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10).reduce(1, (sum, item) -> {
        System.out.println(sum);
        return sum - item;
    });
    System.out.println(reduce);
}
// 1 0 -2 -5 -9 -14 -20 -27 -35 -44 -54
// 第一轮的sum是identity，item是索引为0位置的数据。
// 后面轮次的sum是上一轮次计算的结果。item的索引依次加一。
```

**第三种形式**

```java
<U> U reduce(U identity, 
             BiFunction<U, ? super T, U> accumulator,
             BinaryOperator<U> combiner);
```

> Performs a reduction on the elements of this stream, using the provided identity, accumulation and combining functions. This is equivalent to:
>
> ```java
>  U result = identity;
>  for (T element : this stream)
>      result = accumulator.apply(result, element)
>  return result;
> ```
>
> but is not constrained to execute sequentially. The identity value must be an identity for the combiner function. This means that for all u, combiner(identity, u) is equal to u. Additionally, the combiner function must be compatible with the accumulator function; for all u and t, the following must hold:
>
> ```java
>  combiner.apply(u, accumulator.apply(identity, t)) == accumulator.apply(u, t)
> ```
>
> This is a terminal operation.

第三种形式相对于第二种形式多了一个参数，这个参数是`BinaryOperator`接收两个参数，返回一个值，这三个数据是相同类型的。实际上这个参数是用于支持并发操作的。



## collect

**API介绍**

`reduce()`擅长的是生成一个值，如果想要从Stream生成一个集合或者Map等复杂的对象该怎么办呢？就需要用到`collect()`了，也就是说它可以把Stream中的要有元素收集到一个结果容器中。其有两种重载的方法：

```java
<R> R collect(Supplier<R> supplier,
                  BiConsumer<R, ? super T> accumulator,
                  BiConsumer<R, R> combiner);
```

> Performs a mutable reduction operation on the elements of this stream. A mutable reduction is one in which the reduced value is a mutable result container, such as an ArrayList, and elements are incorporated（合并） by updating the state of the result rather than by replacing the result. This produces a result equivalent to:
>
> ```java
>  R result = supplier.get();
>  for (T element : this stream)
>      accumulator.accept(result, element);
>  return result;
> ```
>
> Like reduce(Object, BinaryOperator), collect operations can be parallelized without requiring additional synchronization. This is a terminal operation.

```java
<R, A> R collect(Collector<? super T, A, R> collector);
```

> Performs a mutable reduction operation on the elements of this stream using a Collector. A Collector encapsulates the functions used as arguments to collect(Supplier, BiConsumer, BiConsumer), allowing for reuse of collection strategies and composition of collect operations such as multiple-level grouping or partitioning.

在不使用API之前我们先考虑一下将一个`Stream`转换成一个容器（或者`Map`）需要做哪些工作？至少需要两样东西吧：

1. 目标容器是什么？是`ArrayList`还是`HashSet`，或者是个`TreeMap`。
2. 新元素如何添加到容器中？是`List.add()`还是`Map.put()`。
3. 如果并行的进行规约，还需要告诉`collect()`多个部分结果如何合并成一个。

结合以上分析，`collect()`方法定义为` R collect(Supplier supplier, BiConsumer accumulator, BiConsumer combiner)`，三个参数依次对应上述三条分析。不过每次调用`collect()`都要传入这三个参数太麻烦，收集器`Collector`就是对这三个参数的简单封装，所以`collect()`的另一定义为` R collect(Collector collector)`。`Collectors`工具类可通过静态方法生成各种常用的`Collector`。举例来说，如果要将`Stream`规约成`List`可以通过如下两种方式实现：

```java
//　将Stream规约成List
Stream<String> stream = Stream.of("I", "love", "you", "too");
List<String> list = 
    stream.collect(ArrayList::new, ArrayList::add, ArrayList::addAll);  // 方式１
// List<String> list = stream.collect(Collectors.toList());  // 方式2
System.out.println(list);
```

通常情况下我们不需要手动指定`collect()`的三个参数，而是调用`collect(Collector collector)`方法，并且参数中的`Collector`对象大都是直接通过`Collectors`工具类获得。实际上传入的收集器的行为决定了`collect()`的行为。

**使用collect()生成Collection**

前面已经提到通过`collect()`方法将`Stream`转换成容器的方法，这里再汇总一下。将`Stream`转换成`List`或`Set`是比较常见的操作，所以`Collectors`工具已经为我们提供了对应的收集器，通过如下代码即可完成：

```java
// 将Stream转换成List或Set
Stream<String> stream = Stream.of("I", "love", "you", "too");
List<String> list = stream.collect(Collectors.toList()); // 1
Set<String> set = stream.collect(Collectors.toSet()); // 2
```

上述代码能够满足大部分需求，但由于返回结果是接口类型，我们并不知道类库实际选择的容器类型是什么，有时候我们可能会想要人为指定容器的实际类型，这个需求可通过`Collectors.toCollection(Supplier collectionFactory)`方法完成。

```java
// 使用toCollection()指定规约容器的类型
ArrayList<String> arrayList = stream.collect(Collectors.toCollection(ArrayList::new)); // 3
HashSet<String> hashSet = stream.collect(Collectors.toCollection(HashSet::new)); // 4
```

上述代码第2行处指定规约结果是`ArrayList`，而第三行处指定规约结果为`HashSet`。一切如你所愿。

我们下面看一个笛卡尔积的例子。

```java
public static void main(String[] args) {
    List<String> list1 = Arrays.asList("Hi", "Hello");
    List<String> list2 = Arrays.asList("zhangsan", "lisi", "wangwu");
    List<String> result =
        list1.stream()
        .flatMap(item -> list2.stream().map(item2 -> item + " " + item2))
        .collect(Collectors.toList());
    result.forEach(System.out::println);
}
// Hi zhangsan
// Hi lisi
// Hi wangwu
// Hello zhangsan
// Hello lisi
// Hello wangwu
```

分析这个例子我们先看`map`和`flatMap`的区别：

```java
Stream<Stream<String>> streamStream = 
			list1.stream().map(item -> list2.stream().map(item2 -> item + " " + item2));
Stream<String> stream = 
			list1.stream().flatMap(item -> list2.stream().map(item2 -> item + " " + item2));
```

我们会发现，`flatMap()`的作用就是我们刚才说的将多个`Stream`摊平。然后我们再分析就会发现，其实这就是一个双层循环，我们再内层循环中，返回了一个`Stream`，在外层循环中将多个`Stream`合在一起。

我们类比着，做一个三层循环：

```java
List<String> result =
    list1.stream()
    .flatMap(item1 -> list2.stream()
             .flatMap(item2 -> list3.stream()
                      .map(item3 -> item1 + "" + item2 + " " + item3)))
    .collect(Collectors.toList());

result.forEach(System.out::println);
//    Hizhangsan a
//    Hizhangsan b
//    Hilisi a
//    Hilisi b
//    Hiwangwu a
//    Hiwangwu b
//    Hellozhangsan a
//    Hellozhangsan b
//    Hellolisi a
//    Hellolisi b
//    Hellowangwu a
//    Hellowangwu b
```

**使用collect()生成Map**

前面已经说过`Stream`背后依赖于某种数据源，数据源可以是数组、容器等，但不能是`Map`。反过来从`Stream`生成`Map`是可以的，但我们要想清楚`Map`的`key`和`value`分别代表什么，根本原因是我们要想清楚要干什么。通常在三种情况下`collect()`的结果会是`Map`：

1. 使用`Collectors.toMap()`生成的收集器，用户需要指定如何生成`Map`的`key`和`value`。
2. 使用`Collectors.partitioningBy()`生成的收集器，对元素进行二分区操作时用到。
3. 使用`Collectors.groupingBy()`生成的收集器，对元素做`group`操作时用到。

```java
public class Student {

    private String name;
    private int score;
    private int age;

    public Student(String name, int score) {
        this.name = name;
        this.score = score;
    }

    public Student(String name, int score, int age) {
        this.name = name;
        this.score = score;
        this.age = age;
    }

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getScore() {
        return score;
    }
    public void setScore(int score) {
        this.score = score;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", score=" + score +
                ", age=" + age +
                '}';
    }
}
```

情况1：使用`toMap()`生成的收集器，这种情况是最直接的，这是和`Collectors.toCollection()`并列的方法。如下代码展示将学生列表转换成由`<学生，分数>`组成的`Map`。非常直观，无需多言。

```java
// 使用toMap()统计学生成绩
Map<Student, Integer> studentToScore =
    students.stream().collect(Collectors.toMap(Function.identity(),     // 如何生成key
                                     student -> student.getScore()));   // 如何生成value
```

情况2：使用`partitioningBy()`生成的收集器，这种情况适用于将`Stream`中的元素依据某个二值逻辑（满足条件，或不满足）分成互补相交的两部分，比如男女性别、成绩及格与否等。

```java
Student student1 = new Student("zhangsan", 80);
Student student2 = new Student("lisi", 90);
Student student3 = new Student("wangwu", 100);
Student student4 = new Student("zhaoliu", 90);
Student student5 = new Student("zhaoliu", 90);
List<Student> students = Arrays.asList(student1, student2, student3, student4);

Map<Boolean, List<Student>> map = students.stream().
    collect(Collectors.partitioningBy(student -> student.getScore() >= 90));

// 对于结果为真部分的数据再次进行分区
Map<Boolean, Map<Boolean, List<Student>>> map3 = 
    students.stream().collect(partitioningBy(student -> student.getScore() > 80, 
                              partitioningBy(student -> student.getScore() > 90)));
// {false={false=[Student{name='zhangsan', score=80, age=0}], 
//         true=[]}, 
// true={false=[Student{name='lisi', score=90, age=0}, Student{name='zhaoliu', score=90, age=0}, Student{name='zhaoliu', score=90, age=0}], 
//       true=[Student{name='wangwu', score=100, age=0}]}}
```

情况3：使用`groupingBy()`生成的收集器，这是比较灵活的一种情况。跟SQL中的`group by`语句类似，这里的`groupingBy()`也是按照某个属性对数据进行分组，属性相同的元素会被对应到`Map`的同一个`key`上。

```java
Map<String, List<Student>> map = students.stream().
			collect(Collectors.groupingBy(Student::getName));
Map<Integer, List<Student>> map = students.stream().
			collect(Collectors.groupingBy(Student::getScore));
```

以上只是分组的最基本用法，有些时候仅仅分组是不够的。在SQL中使用`group by`是为了协助其他查询，比如

1. 先将员工按照部门分组
2. 然后统计每个部门员工的人数

Java类库设计者也考虑到了这种情况，增强版的`groupingBy()`能够满足这种需求。增强版的`groupingBy()`允许我们对元素分组之后再执行某种运算，比如求和、计数、平均值、类型转换等。这种先将元素分组的收集器叫做**上游收集器**，之后执行其他运算的收集器叫做**下游收集器**(*downstream Collector*)。

```java
Map<String, Long> map = students.stream().
			collect(Collectors.groupingBy(Student::getName, 
                                          Collectors.counting())); // 下游收集器

Map<String, Double> map = students.stream().collect(Collectors.groupingBy(
    		Student::getName, 
    		Collectors.averagingDouble(Student::getScore)));   // 下游收集器
```

上面代码的逻辑是不是越看越像SQL？高度非结构化。还有更狠的，下游收集器还可以包含更下游的收集器。

```java
Map<Integer, Map<String, List<Student>>> map = students.stream().
    collect(groupingBy(Student::getScore, groupingBy(Student::getName)));
//	{80={zhangsan=[Student{name='zhangsan', score=80, age=0}]}, 
//	 100={wangwu=[Student{name='wangwu', score=100, age=0}]}, 
//   90={lisi=[Student{name='lisi', score=90, age=0}], 
//      zhaoliu=[Student{name='zhaoliu', score=90, age=0}, Student{name='zhaoliu', score=90, 
//																	age=0}]}}
```

**使用collect()做字符串join**

这个肯定是大家喜闻乐见的功能，字符串拼接时使用`Collectors.joining()`生成的收集器，从此告别*for*循环。`Collectors.joining()`方法有三种重写形式，分别对应三种不同的拼接方式。无需多言，代码过目难忘。

```java
// 使用Collectors.joining()拼接字符串
Stream<String> stream = Stream.of("I", "love", "you");
//String joined = stream.collect(Collectors.joining());  // "Iloveyou"
//String joined = stream.collect(Collectors.joining(","));  // "I,love,you"
String joined = stream.collect(Collectors.joining(",", "{", "}"));  // "{I,love,you}"
```

**collectingAndThen**

```java
// 获得每个姓名下的最低分，返回的是Optional，但实际上此时的最低分一定不为空
Map<String, Optional<Student>> collect = students.stream().collect(groupingBy(
    			Student::getName, minBy(Comparator.comparingInt(Student::getScore))));

// 所以可以只用collectingAndThen，传入一个 Function
Map<String, Student> map5 = students.stream().collect(groupingBy(Student::getName, 	            collectingAndThen(minBy(Comparator.comparingInt(Student::getScore)), Optional::get)));

// {lisi=Student{name='lisi', score=90, age=0}, 
//  zhaoliu=Student{name='zhaoliu', score=90, age=0}, 
//  zhangsan=Student{name='zhangsan', score=80, age=0}, 
//  wangwu=Student{name='wangwu', score=100, age=0}}
```



## Collector

> A mutable reduction operation that accumulates input elements into a mutable result container, optionally transforming the accumulated result into a final representation after all input elements have been processed. Reduction operations can be performed either sequentially or in parallel.
>
> Examples of mutable reduction operations include: accumulating elements into a Collection; concatenating strings using a StringBuilder; computing summary information about elements such as sum, min, max, or average; computing "pivot table" summaries such as "maximum valued transaction by seller", etc. The class Collectors provides implementations of many common mutable reductions.
>
> A reduction operation (also called a fold) takes a sequence of input elements and combines them into a single summary result by repeated application of a combining operation, such as finding the sum or maximum of a set of numbers, or accumulating elements into a list. 
>
> A mutable reduction operation accumulates input elements into a mutable result container, such as a Collection or StringBuilder, as it processes the elements in the stream.
>
> A Collector is specified by four functions that work together to accumulate entries into a mutable result container, and optionally perform a final transform on the result. They are:
>
> - creation of a new result container (supplier())
> - incorporating a new data element into a result container (accumulator())
> - combining two result containers into one (combiner())
> - performing an optional final transform on the container (finisher())

对于函数`combiner()`，JDK对其注释如下：

A function that accepts two partial results and merges them. The combiner function may fold state from one argument into the other and return that, or may return a new result container.

我们举个例子来看这个函数：

假如有四个线程同时执行一次聚合，此次聚合的结果是生成一个集合，那么四个线程就会产生四个结果集合。此时这个函数便要发挥作用，他需要将四个结果集合合并成为一个，那么此时可以有两种做法，一是再生成一个新的集合，将四个结果集合都加入这个集合，再返回。二是将第二个结果集合加入第一个结果集合，然后返回第一个集合再去，然后再去和第三个集合合并，此时我们就说第二个结果集合被fold了。

这个方法会在线程安全的情况下被调用，即使开发者没有执行并发安全操作。它的核心用处就是在并行流的环境中将多个部分结果合成一个结果返回。

> Collectors also have a set of characteristics, such as Collector.Characteristics.CONCURRENT, that provide hints that can be used by a reduction implementation to provide better performance.
>
> A sequential implementation of a reduction using a collector would create a single result container using the supplier function, and invoke the accumulator function once for each input element. A parallel implementation would partition the input, create a result container for each partition, accumulate the contents of each partition into a subresult for that partition, and then use the combiner function to merge the subresults into a combined result.
>
> To ensure that sequential and parallel executions produce equivalent results, the collector functions must satisfy an identity and an associativity constraints.
>
> The identity constraint says that for any partially accumulated result, combining it with an empty result container must produce an equivalent result. That is, for a partially accumulated result a that is the result of any series of accumulator and combiner invocations, a must be equivalent to combiner.apply(a, supplier.get()).

identity可以翻译为”同一性“，这个性质就是说对于任何一个中间结果，当`combiner.apply(a, supplier.get())`被调用后时必须等价于a（等价于可能是=，也可能是equal，需要根据业务来算）。

> The associativity constraint says that splitting the computation must produce an equivalent result. That is, for any input elements t1 and t2, the results r1 and r2 in the computation below must be equivalent:
>
> ```java
>  A a1 = supplier.get();
>  accumulator.accept(a1, t1);
>  accumulator.accept(a1, t2);
>  R r1 = finisher.apply(a1);  // result without splitting
> 
>  A a2 = supplier.get();
>  accumulator.accept(a2, t1);
>  A a3 = supplier.get();
>  accumulator.accept(a3, t2);
>  R r2 = finisher.apply(combiner.apply(a2, a3));  // result with splitting
> ```

上面的代码就是在说将一系列的操作串行执行和将操作分割执行之后再合并应该保持一致。

> For collectors that do not have the UNORDERED characteristic, two accumulated results a1 and a2 are equivalent if finisher.apply(a1).equals(finisher.apply(a2)). For unordered collectors, equivalence is relaxed to allow for non-equality related to differences in order. (For example, an unordered collector that accumulated elements to a List would consider two lists equivalent if they contained the same elements, ignoring order.)

两个结果`a1`，`a2`是否等价的逻辑值应该和`finisher.apply(a1).equals(finisher.apply(a2))`一致。但是对于具有无序特征的情况下，不考虑容器中的元素顺序。

> Libraries that implement reduction based on Collector, such as Stream.collect(Collector), must adhere to the following constraints:
>
> - The first argument passed to the accumulator function, both arguments passed to the combiner function, and the argument passed to the finisher function must be the result of a previous invocation of the result supplier, accumulator, or combiner functions.

这个实际上是在说累加器的第一个参数是容器的类型，组合器参数的类型是容器的类型。

> - The implementation should not do anything with the result of any of the result supplier, accumulator, or combiner functions other than to pass them again to the accumulator, combiner, or finisher functions, or return them to the caller of the reduction operation.

这个实际上是在说在collect的执行过程中所有的操作都应该来自`supplier`、`accumulator`、`combiner `。

> - If a result is passed to the combiner or finisher function, and the same object is not returned from that function, it is never used again.
> - Once a result is passed to the combiner or finisher function, it is never passed to the accumulator function again.

这个又有点fold的意思了，也就是旧的对象如果不作为返回值返回，他们的状态就会被折叠。

> - For non-concurrent collectors, any result returned from the result supplier, accumulator, or combiner functions must be serially thread-confined. This enables collection to occur in parallel without the Collector needing to implement any additional synchronization. The reduction implementation must manage that the input is properly partitioned, that partitions are processed in isolation, and combining happens only after accumulation is complete.
> - For concurrent collectors, an implementation is free to (but not required to) implement reduction concurrently. A concurrent reduction is one where the accumulator function is called concurrently from multiple threads, using the same concurrently-modifiable result container, rather than keeping the result isolated during accumulation. A concurrent reduction should only be applied if the collector has the Collector.Characteristics.UNORDERED characteristics or if the originating data is unordered.

这两条很绕，分析之后会发现。它是在说非并发的收集器需要保证收三个函数的结果不被其他线程调用，这样就可以在并发的条件下无须额外的同步就可以使用。对于并发的收集器，并发操作应该是使用线程安全的容器在调用累加操作的时候保证的，而不是在累加操作的内部实现中保证的。

> In addition to the predefined implementations in Collectors, the static factory methods of(Supplier, BiConsumer, BinaryOperator, Characteristics...) can be used to construct collectors. For example, you could create a collector that accumulates widgets into a TreeSet with:
>
> ```java
>  Collector<Widget, ?, TreeSet<Widget>> intoSet =
>      Collector.of(TreeSet::new, TreeSet::add,
>                   (left, right) -> { left.addAll(right); return left; });
> ```
>
> (This behavior is also implemented by the predefined collector Collectors.toCollection(Supplier)).

apiNote：

> Performing a reduction operation with a Collector should produce a result equivalent to:
>
> ```java
>  R container = collector.supplier().get();
>  for (T t : data)
>      collector.accumulator().accept(container, t);
>  return collector.finisher().apply(container);
> ```
>
> However, the library is free to partition the input, perform the reduction on the partitions, and then use the combiner function to combine the partial results to achieve a parallel reduction. (Depending on the specific reduction operation, this may perform better or worse, depending on the relative cost of the accumulator and combiner functions.)
>
> Collectors are designed to be composed; many of the methods in Collectors are functions that take a collector and produce a new collector. For example, given the following collector that computes the sum of the salaries of a stream of employees:
>
> ```java
>  Collector<Employee, ?, Integer> summingSalaries
>      = Collectors.summingInt(Employee::getSalary))
> ```
>
> If we wanted to create a collector to tabulate the sum of salaries by department, we could reuse the "sum of salaries" logic using Collectors.groupingBy(Function, Collector):
>
> ```java
>  Collector<Employee, ?, Map<Department, Integer>> summingSalariesByDept
>      			= Collectors.groupingBy(Employee::getDepartment, summingSalaries);
> ```
>



### 自定义收集器

自定义一个收集器，我们需要实现Collector接口。Collector接口的三个泛型参数分别表示集合中元素的类型，集合的类型和返回值的类型。我们再看五个待实现的方法。

```java
Supplier<A> supplier();
BiConsumer<A, T> accumulator();
BinaryOperator<A> combiner();
Function<A, R> finisher();
Set<Characteristics> characteristics();
```

`supplier`、`accumulator`、`combiner`之前已经介绍的很详细了。`finisher`的作用是将中间容器的类型转为最终的结果类型。中间容器的类型也就是三个泛型中的集合的类型，结果类型也就是三个泛型中的返回值的类型。

而对于`characteristics`是标识这个收集器的特性。它是使用枚举来做的。

```java
enum Characteristics {
    /**
     * Indicates that this collector is <em>concurrent</em>, meaning that
     * the result container can support the accumulator function being
     * called concurrently with the same result container from multiple
     * threads.
     *
     * <p>If a {@code CONCURRENT} collector is not also {@code UNORDERED},
     * then it should only be evaluated concurrently if applied to an
     * unordered data source.
     */
    CONCURRENT,

    /**
     * Indicates that the collection operation does not commit to preserving
     * the encounter order of input elements.  (This might be true if the
     * result container has no intrinsic order, such as a {@link Set}.)
     */
    UNORDERED,

    /**
     * Indicates that the finisher function is the identity function and
     * can be elided.  If set, it must be the case that an unchecked cast
     * from A to R will succeed.
     */
    IDENTITY_FINISH
}
```

- `UNORDERED`：如果流中输入的元素和最终返回的元素顺序不一致，需要标志上这个特性。比如讲一个List转为一个Set，Set中元素的顺序不保证和List一致。
- `IDENTITY_FINISH`：如果中间结果的类型和最终返回的类型一致，标志上这个特性之`finisher`就不会被调用，而是使用强制转换的方式来将中间结果的容器转为最终结果。

下面是一个使用自定义收集器将List转为Set的案例，中间容器的类型和结果容器的类型一致。

```java
public class MySetCollector<T> implements Collector<T, Set<T>, Set<T>>{
    @Override
    public Supplier<Set<T>> supplier() {
        System.out.println("supplier invoked!");
        return HashSet::new;
    }

    @Override
    public BiConsumer<Set<T>, T> accumulator() {
        System.out.println("accumulator invoked!");
        return Set::add;
    }

    @Override
    public BinaryOperator<Set<T>> combiner() {
        System.out.println("combiner invoked!");
        return (set1, set2) -> {
            set1.addAll(set2);
            return set1;
        };
    }

    @Override
    public Function<Set<T>, Set<T>> finisher() {
//        System.out.println("finisher invoked!");
//        return Function.identity();
        throw new UnsupportedOperationException();
    }

    @Override
    public Set<Characteristics> characteristics() {
        System.out.println("characteristics invoked!");
        return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH, UNORDERED));
    }

    public static void main(String[] args) {
        List<String> list = Arrays.asList("hello", "world", "welcome", "hello");
        Set<String> set = list.stream().collect(new MySetCollector<>());

        System.out.println(set);
    }
}
```

下面是一个中间容器和最终结果容器不一致的情况：

```java
public class MySetCollector2<T> implements Collector<T, Set<T>, Map<T, T>> {

    @Override
    public Supplier<Set<T>> supplier() {
        System.out.println("supplier invoked!");
        return HashSet<T>::new;
    }

    @Override
    public BiConsumer<Set<T>, T> accumulator() {
        System.out.println("accumulator invoked!");
        return (set, item) -> {
            set.add(item);
        };
    }

    @Override
    public BinaryOperator<Set<T>> combiner() {
        System.out.println("combiner invoked!");
        return (set1, set2) -> {
            set1.addAll(set2);
            return set1;
        };
    }

    @Override
    public Function<Set<T>, Map<T, T>> finisher() {
        System.out.println("finisher invoked!");
        return set -> {
            Map<T, T> map = new TreeMap<>();
            set.stream().forEach(item -> map.put(item, item));
            return map;
        };
    }

    @Override
    public Set<Characteristics> characteristics() {
        System.out.println("characteristics invoked!");
        return Collections.unmodifiableSet(EnumSet.of(Characteristics.UNORDERED));
    }

    public static void main(String[] args) {
        System.out.println(Runtime.getRuntime().availableProcessors());
        for(int i = 0; i < 1; ++i) {
            List<String> list = Arrays.asList(
                "hello", "world", "welcome", "hello", "a", "b", "c", "d", "e", "f", "g");
            Set<String> set = new HashSet<>();
            set.addAll(list);
            System.out.println("set: " + set);

            Map<String, String> map = 
                set.stream().collect(new MySetCollector2<>());
            System.out.println(map);
        }
    }
}
```

对于`CONCURRENT`，是指在并行流的情况下多个结果会去操作一个容器。如果不加上这个特性，并行流的情况下是多个线程各自操作一个容器。

```java
public class MySetCollector2<T> implements Collector<T, Set<T>, Map<T, T>> {

    @Override
    public Supplier<Set<T>> supplier() {
        System.out.println("supplier invoked!");
        return () -> {
            System.out.println("-----------");
            return new HashSet<T>();
        };
    }

    @Override
    public BiConsumer<Set<T>, T> accumulator() {
        System.out.println("accumulator invoked!");
        return (set, item) -> {
            System.out.println("accumulator: " + 
            		set  + ", " + Thread.currentThread().getName());
            set.add(item);
        };
    }

    @Override
    public BinaryOperator<Set<T>> combiner() {
        System.out.println("combiner invoked!");
        return (set1, set2) -> {
            System.out.println("set1: " + set1);
            System.out.println("set2: " + set2);
            set1.addAll(set2);
            return set1;
        };
    }

    @Override
    public Function<Set<T>, Map<T, T>> finisher() {
        System.out.println("finisher invoked!");
        return set -> {
            Map<T, T> map = new TreeMap<>();
            set.stream().forEach(item -> map.put(item, item));
            return map;
        };
    }

    @Override
    public Set<Characteristics> characteristics() {
        System.out.println("characteristics invoked!");
        return Collections.unmodifiableSet(
        	EnumSet.of(Characteristics.UNORDERED, Characteristics.CONCURRENT));
    }

    public static void main(String[] args) {
        for(int i = 0; i < 1; ++i) {
            List<String> list = Arrays.asList(
            	"hello", "world", "welcome", "hello", "a", "b", "c", "d", "e", "f", "g");
            Set<String> set = new HashSet<>();
            set.addAll(list);
            System.out.println("set: " + set);
            Map<String, String> map = 
            	set.parallelStream().collect(new MySetCollector2<>());
            System.out.println(map);
        }
    }
}
```

所以在并行流的环境中如果加上了`CONCURRENT`，只会生成一个容器，而且此时后期不需要调用`combiner`，上面的代码在不抛出并行修改异常的时候会作如下输出：

```
8
set: [a, b, world, c, d, e, f, g, hello, welcome]
characteristics invoked!
supplier invoked!
-----------
accumulator invoked!
accumulator: [], main
accumulator: [hello], main
accumulator: [hello], ForkJoinPool.commonPool-worker-1
accumulator: [hello, welcome], ForkJoinPool.commonPool-worker-4
accumulator: [hello, welcome], ForkJoinPool.commonPool-worker-3
accumulator: [hello, welcome], main
accumulator: [a, b, d, f, hello, welcome], main
accumulator: [a, b, d, hello, welcome], ForkJoinPool.commonPool-worker-3
accumulator: [d, hello, welcome], ForkJoinPool.commonPool-worker-1
accumulator: [a, b, world, d, f, g, hello, welcome], ForkJoinPool.commonPool-worker-3
characteristics invoked!
finisher invoked!
{a=a, b=b, c=c, d=d, e=e, f=f, g=g, hello=hello, welcome=welcome, world=world}
```

如果不加上`CONCURRENT`，由于操作的是多个容器，会调用多次`combiner`。

```java
8
set: [a, b, world, c, d, e, f, g, hello, welcome]
characteristics invoked!
supplier invoked!
accumulator invoked!
combiner invoked!
characteristics invoked!
-----------
-----------
-----------
-----------
accumulator: [], main
accumulator: [], ForkJoinPool.commonPool-worker-3
-----------
accumulator: [], ForkJoinPool.commonPool-worker-4
accumulator: [], ForkJoinPool.commonPool-worker-1
-----------
-----------
accumulator: [], ForkJoinPool.commonPool-worker-5
accumulator: [b], ForkJoinPool.commonPool-worker-3
accumulator: [f], ForkJoinPool.commonPool-worker-5
set1: []
accumulator: [], ForkJoinPool.commonPool-worker-4
-----------
accumulator: [d], ForkJoinPool.commonPool-worker-1
set1: [welcome]
set2: []
set2: [hello]
accumulator: [b, world], ForkJoinPool.commonPool-worker-3
set1: [hello]
set1: [d, e]
set2: [welcome]
set1: [a]
set2: [f, g]
set2: [b, world, c]
set1: [a, b, world, c]
set2: [d, e, f, g]
set1: [a, b, world, c, d, e, f, g]
set2: [hello, welcome]
characteristics invoked!
finisher invoked!
{a=a, b=b, c=c, d=d, e=e, f=f, g=g, hello=hello, welcome=welcome, world=world}
```

从结果中可以看出，生成了多个中间容器，执行了多次`combiner`。



## Stream

> A sequence of elements supporting sequential and parallel aggregate operations. The following example illustrates an aggregate operation using Stream and IntStream:
>
> ```java
>  int sum = widgets.stream()
>                   .filter(w -> w.getColor() == RED)
>                   .mapToInt(w -> w.getWeight())
>                   .sum();
> ```
>
> In this example, widgets is a `Collection<Widget>`. We create a stream of Widget objects via Collection.stream(), filter it to produce a stream containing only the red widgets, and then transform it into a stream of int values representing the weight of each red widget. Then this stream is summed to produce a total weight.
>
> In addition to Stream, which is a stream of object references, there are primitive specializations for IntStream, LongStream, and DoubleStream, all of which are referred to as "streams" and conform to the characteristics and restrictions described here.
>
> To perform a computation, stream operations are compose d into a stream pipeline. A stream pipeline consists of a source (which might be an array, a collection, a generator function, an I/O channel, etc), zero or more intermediate operations (which transform a stream into another stream, such as filter(Predicate)), and a terminal operation (which produces a result or side-effect, such as count() or forEach(Consumer)). Streams are lazy; computation on the source data is only performed when the terminal operation is initiated, and source elements are consumed only as needed.

side-effect指的是在流执行过程中可对源数据的状态做出修改。side-effect是不被推荐使用的。

```java
ArrayList<String> results = new ArrayList<>();
stream.filter(s -> pattern.matcher(s).matches())
    .forEach(s -> results.add(s));  // Unnecessary use of side-effects!
```

> Collections and streams, while bearing some superficial similarities, have different goals. Collections are primarily concerned with the efficient management of, and access to, their elements. By contrast, streams do not provide a means to directly access or manipulate their elements, and are instead concerned with declaratively describing their source and the computational operations which will be performed in aggregate on that source. However, if the provided stream operations do not offer the desired functionality, the iterator() and spliterator() operations can be used to perform a controlled traversal.

JDK8的Collection中多了一个方法：`Spliterator<E> spliterator()`。翻译为分割迭代器，顾名思义，这个方法是获得一个迭代器，而为什么不是返回`iterator`，而是要新创建一个类呢？这是为流的并行做准备，如果使用`iterator`，它是使用`hasNext`和`next`顺序遍历的，不方便我们将流直接分割成差不多的两部分。所以就引入了`Spliterator`。

> A stream pipeline, like the "widgets" example above, can be viewed as a query on the stream source. Unless the source was explicitly designed for concurrent modification (such as a ConcurrentHashMap), unpredictable or erroneous behavior may result from modifying the stream source while it is being queried.
>
> Most stream operations accept parameters that describe user-specified behavior, such as the lambda expression w -> w.getWeight() passed to mapToInt in the example above. To preserve correct behavior, these behavioral parameters:
>
> - must be non-interfering (they do not modify the stream source); and 
> - in most cases must be stateless (their result should not depend on any state that might change during execution of the stream pipeline).

如果源数据不是支持并发的容器，不要在流的执行期间修改源数据。否则会导致异常。但是在流执行之前修改是可以的。

```java
List<String> l = new ArrayList(Arrays.asList("one", "two"));
Stream<String> sl = l.stream();
l.add("three");
String s = sl.collect(joining(" "));
```

> Such parameters are always instances of a functional interface such as Function, and are often lambda expressions or method references. Unless otherwise specified these parameters must be non-null.
>
> A stream should be operated on (invoking an intermediate or terminal stream operation) only once. This rules out, for example, "forked" streams, where the same source feeds two or more pipelines, or multiple traversals of the same stream. A stream implementation may throw IllegalStateException if it detects that the stream is being reused. However, since some stream operations may return their receiver rather than a new stream object, it may not be possible to detect reuse in all cases.
>
> Streams have a close() method and implement AutoCloseable, but nearly all stream instances do not actually need to be closed after use. Generally, only streams whose source is an IO channel (such as those returned by Files.lines(Path, Charset)) will require closing. Most streams are backed by collections, arrays, or generating functions, which require no special resource management. (If a stream does require closing, it can be declared as a resource in a try-with-resources statement.)
>
> Stream pipelines may execute either sequentially or in parallel. This execution mode is a property of the stream. Streams are created with an initial choice of sequential or parallel execution. (For example, Collection.stream() creates a sequential stream, and Collection.parallelStream() creates a parallel one.) This choice of execution mode may be modified by the sequential() or parallel() methods, and may be queried with the isParallel() method.