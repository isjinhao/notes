## 分割迭代器

> An object for traversing and partitioning elements of a source. The source of elements covered by a Spliterator could be, for example, an array, a Collection, an IO channel, or a generator function.
> A Spliterator may traverse elements individually (tryAdvance()) or sequentially in bulk （散装的）(forEachRemaining()).

**`tryAdvance()`**

If a remaining element exists, performs the given action on it, returning true; else returns false. If this Spliterator is ORDERED the action is performed on the next element in encounter order. Exceptions thrown by the action are relayed to the caller. （为了更好的解释，我们顺便看一下`Collection`的实现，下同）

```java
private final Collection<? extends T> collection; // null OK
private Iterator<? extends T> it;

public boolean tryAdvance(Consumer<? super T> action) {
    if (action == null) throw new NullPointerException();
    if (it == null) {
        it = collection.iterator();
        est = (long) collection.size();
    }
    if (it.hasNext()) {
        action.accept(it.next());
        return true;
    }
    return false;
}
```

**`forEachRemaining()`**

Performs the given action for each remaining element, sequentially in the current thread, until all elements have been processed or the action throws an exception. If this Spliterator is ORDERED, actions are performed in encounter order. Exceptions thrown by the action are relayed to the caller.

```java
private final Collection<? extends T> collection; // null OK
private Iterator<? extends T> it;
private long est;             // size estimate

public void forEachRemaining(Consumer<? super T> action) {
    if (action == null) throw new NullPointerException();
    Iterator<? extends T> i;
    if ((i = it) == null) {
        i = it = collection.iterator();
        est = (long)collection.size();
    }
    i.forEachRemaining(action);
}
```

> A Spliterator may also partition off some of its elements (using trySplit) as another Spliterator, to be used in possibly-parallel operations. Operations using a Spliterator that cannot split, or does so in a highly imbalanced or inefficient manner, are unlikely to benefit from parallelism. Traversal and splitting exhaust elements; each Spliterator is useful for only a single bulk computation.

**`estimateSize()`**

Returns an estimate of the number of elements that would be encountered by a forEachRemaining traversal, or returns Long.MAX_VALUE if infinite, unknown, or too expensive to compute.

If this Spliterator is SIZED and has not yet been partially traversed or split, or this Spliterator is SUBSIZED and has not yet been partially traversed, this estimate must be an accurate count of elements that would be encountered by a complete traversal. Otherwise, this estimate may be arbitrarily inaccurate, but must decrease as specified across invocations of trySplit.

```java
@Override
public long estimateSize() {
    if (it == null) {
        it = collection.iterator();
        return est = (long)collection.size();
    }
    return est;
}
```

**`trySplit()`**

If this spliterator can be partitioned, returns a Spliterator covering elements, that will, upon return from this method, not be covered by this Spliterator.

If this Spliterator is ORDERED, the returned Spliterator must cover a strict prefix of the elements.

Unless this Spliterator covers an infinite number of elements, repeated calls to trySplit() must eventually return null. Upon non-null return:

- the value reported for estimateSize() before splitting, must, after splitting, be greater than or equal to estimateSize() for this and the returned Spliterator; and

- if this Spliterator is SUBSIZED, then estimateSize() for this spliterator before splitting must be equal to the sum of estimateSize() for this and the returned Spliterator after splitting.

This method may return null for any reason, including emptiness, inability to split after traversal has commenced, data structure constraints, and efficiency considerations.

Returns: a Spliterator covering some portion of the elements, or null if this spliterator cannot be split

```java
static final int BATCH_UNIT = 1 << 10;  // batch array size increment
static final int MAX_BATCH = 1 << 25;  // max batch array size;
private long est;             // size estimate
private int batch;            // batch size for splits

public Spliterator<T> trySplit() {
    Iterator<? extends T> i;
    long s;		// collection 的 size
    if ((i = it) == null) {
        i = it = collection.iterator();
        s = est = (long) collection.size();
    } else
        s = est;
    if (s > 1 && i.hasNext()) {
        int n = batch + BATCH_UNIT;
        if (n > s)
            n = (int) s;
        if (n > MAX_BATCH)
            n = MAX_BATCH;
        Object[] a = new Object[n];
        int j = 0;
        do { 
            a[j] = i.next(); 
        } while (++j < n && i.hasNext());
        batch = j;
        if (est != Long.MAX_VALUE)
            est -= j;
        return new ArraySpliterator<>(a, 0, j, characteristics);
    }
    return null;
}
```

对于`Collection`里面`trySplit()`的实现，第一次调用返回的分割迭代器覆盖$2^{10}$个元素，第二次调用返回的覆盖$2^{11}$个元素，第三次调用返回的覆盖$2^{12}$个元素，第n次调用返回的覆盖$2^{9+n}$的元素。



## 构建&执行流

我们下面通过一个例子来剖析，JDK是怎么构建&执行流的。

```java
public static void main(String[] args) {
    List<String> strings = new ArrayList<>();
    strings.add("A123");
    strings.add("A12345");
    strings.add("B123");
    strings.add("B1234");

    strings.stream()
        .filter(s -> s.startsWith("A"))
        .map(item -> item.length())
        .forEach(System.out::println);
}
```

对于这个例子，他是调用Collection的`stream()`生成一个流：

```java
default Spliterator<E> spliterator() {
    return Spliterators.spliterator(this, 0);
}
default Stream<E> stream() {
    // <T> Stream<T> stream(Spliterator<T> spliterator, boolean parallel) {
    return StreamSupport.stream(spliterator(), false);
}
```

分割迭代器方法负责将一个源的数据划分开来。

```java
public static <T> Stream<T> stream(Spliterator<T> spliterator, boolean parallel) {
    Objects.requireNonNull(spliterator);
    return new ReferencePipeline.Head<>(spliterator,
                                        StreamOpFlag.fromCharacteristics(spliterator),
                                        parallel);
}
```

在这段代码中我们会发现，最终我们获得的流是`ReferencePipeline`的内部类`Head`。再深入去看，我们会发现有一个这样的继承体系，[图片来自](https://www.cnblogs.com/Dorae/p/7779246.html)。

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/ec50f054-7296-433b-8bc3-c430be612f02" /></div>
这个继承体系不是很难，很多都是在使用流的时候会被使用的类。

我们之前说过，一个流由一个源，0个或多个中间操作，以及一个终止操作构成。这个体系里的`XXXPipeline`便是中间操作。他们构成了一个管道。在管道中，`Head`，`StatelssOp`，`StatefulOp`三个是真正的代表中间操作的类。其中我们刚才看到的`Head`便表示管道的第一个操作，因为在其他的中间操作中，都有前驱和后继，但是对于`Head`，是只有后继，没有前驱，所以被单独列出来。

我们常用的`Stream`中的方法实际上是在`XXXPipeline`中被实现的，由于我们的例子是使用`ReferencePipeline`，所以我们下面就去看看这个类中是如何实现`filter`、`map`和`forEach`的。

```java
public final Stream<P_OUT> filter(Predicate<? super P_OUT> predicate) {
    Objects.requireNonNull(predicate);
    return new StatelessOp<P_OUT, P_OUT>(this, StreamShape.REFERENCE,
                                         StreamOpFlag.NOT_SIZED) {
        @Override
        Sink<P_OUT> opWrapSink(int flags, Sink<P_OUT> sink) {
            return new Sink.ChainedReference<P_OUT, P_OUT>(sink) {
                @Override
                public void begin(long size) {
                    downstream.begin(-1);
                }
                @Override
                public void accept(P_OUT u) {
                    if (predicate.test(u))
                        downstream.accept(u);
                }
            };
        }
    };
}
```

```java
public final <R> Stream<R> map(Function<? super P_OUT, ? extends R> mapper) {
    Objects.requireNonNull(mapper);
    return new StatelessOp<P_OUT, R>(this, StreamShape.REFERENCE,
                                     StreamOpFlag.NOT_SORTED | StreamOpFlag.NOT_DISTINCT) {
        @Override
        Sink<P_OUT> opWrapSink(int flags, Sink<R> sink) {
            return new Sink.ChainedReference<P_OUT, R>(sink) {
                @Override
                public void accept(P_OUT u) {
                    downstream.accept(mapper.apply(u));
                }
            };
        }
    };
}
```

```java
public void forEach(Consumer<? super P_OUT> action) {
    evaluate(ForEachOps.makeRef(action, false));
}
```

很明显可以看到，中间操作和终止操作的实现有很大的不同。至此，我们可以画一个图来表示这个过程。

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/f841e7ca-5736-4105-b917-11cc2bc05feb" /></div>
但是我们现在还不知道这个流管道是怎么被串联起来的。所以我们接着分析。

我们进入`StatelessOp`的构造方法中，一路向下跟进：

```java
StatelessOp(AbstractPipeline<?, E_IN, ?> upstream,
            StreamShape inputShape,
            int opFlags) {
    super(upstream, opFlags);
    assert upstream.getOutputShape() == inputShape;
}

ReferencePipeline(AbstractPipeline<?, P_IN, ?> upstream, int opFlags) {
    super(upstream, opFlags);
}

AbstractPipeline(AbstractPipeline<?, E_IN, ?> previousStage, int opFlags) {
    if (previousStage.linkedOrConsumed)
        throw new IllegalStateException(MSG_STREAM_LINKED);
    previousStage.linkedOrConsumed = true;
    previousStage.nextStage = this;

    this.previousStage = previousStage;
    this.sourceOrOpFlags = opFlags & StreamOpFlag.OP_MASK;
    this.combinedFlags = StreamOpFlag.combineOpFlags(opFlags, previousStage.combinedFlags);
    this.sourceStage = previousStage.sourceStage;
    if (opIsStateful())
        sourceStage.sourceAnyStateful = true;
    this.depth = previousStage.depth + 1;
}
```

跟到了`AbstractPipeline`的构造方法中，我们才发现，`AbstractPipeline`中有三个属性：

```java
private final AbstractPipeline sourceStage;

/**
  * The "upstream" pipeline, or null if this is the source stage.
  */
private final AbstractPipeline previousStage;

/**
  * The next stage in the pipeline, or null if this is the last stage.
  * Effectively final at the point of linking to the next pipeline.
  */
private AbstractPipeline nextStage;
```

看到这里我们就会发现，管道和管道之间的链接是使用双向链表的方式进行的。同时，每个非`Head`管道，都有一个指向`Head`的引用。这样我们就可以将上面的图再具体一些：

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/1d36c6a1-831c-4d1a-84fa-92a2117f3fcb" /></div>
看到这里我们就知道中间操作的流管道是怎么被串联起来的，但是终止操作是怎么和中间操作联系到一起的呢？所以我们需要看`forEach`方法：

```java
@Override
public void forEach(Consumer<? super P_OUT> action) {
    evaluate(ForEachOps.makeRef(action, false));
}
```

在这个方法中，我们先看内部的方法：

```java
public static <T> TerminalOp<T, Void> makeRef(Consumer<? super T> action,
                                              boolean ordered) {
    Objects.requireNonNull(action);
    return new ForEachOp.OfRef<>(action, ordered);
}
```

这个方法是一个工厂方法，负责构建一个`TerminalOp`类的对象，这个类一看类名就知道他是一个终止操作。我们再向下跟进一层：

```java
static final class OfRef<T> extends ForEachOp<T> {
    final Consumer<? super T> consumer;

    OfRef(Consumer<? super T> consumer, boolean ordered) {
        super(ordered);
        this.consumer = consumer;
    }

    @Override
    public void accept(T t) {
        consumer.accept(t);
    }
}
```

我么会发现，`OfRef`这个类中保存我们传递给`forEach`的行为参数，也就是一个`Consumer`。

然后我们再看`evaluate()`

```java
final <R> R evaluate(TerminalOp<E_OUT, R> terminalOp) {
    assert getOutputShape() == terminalOp.inputShape();
    if (linkedOrConsumed)
        throw new IllegalStateException(MSG_STREAM_LINKED);
    linkedOrConsumed = true;

    return isParallel()
        ? terminalOp.evaluateParallel(this, sourceSpliterator(terminalOp.getOpFlags()))
        : terminalOp.evaluateSequential(this, sourceSpliterator(terminalOp.getOpFlags()));
}
```

这个方法接收一个`TerminalOp`。从他返回的结果来看，我们知道，这里是并行流和串行流执行的分叉的地方。并行流的分析很困难，作为理解`Stream`的原理，而不是真正从性能等角度考虑并行流和串行流的差异，我们便去看看串行流是怎么执行的，进而来理解`Stream`。

```java
<P_IN> R evaluateSequential(PipelineHelper<E_IN> helper, Spliterator<P_IN> spliterator);
```

这个方法是未被实现的方法，它需要`TerminalOp`的子类来决定它的行为到底是什么。同时可以看一下`evaluateParallel()`，这个方法被提供了一个默认实现，同时默认实现就是调用了一下串行的方法。

```java
default <P_IN> R evaluateParallel(PipelineHelper<E_IN> helper,
                                  Spliterator<P_IN> spliterator) {
    if (Tripwire.ENABLED)
        Tripwire.trip(getClass(), 
                      "{0} triggering TerminalOp.evaluateParallel serial default");
    return evaluateSequential(helper, spliterator);
}
```

在找到`evaluateSequential()`的实现之前，我们先去看看`PipelineHelper`，这个类的注释如下：

>Helper class for executing stream pipelines, capturing all of the information about a stream pipeline (output shape, intermediate operations, stream flags, parallelism, etc) in one place.
>
>A PipelineHelper describes the initial segment of a stream pipeline, including its source, intermediate operations, and may additionally incorporate information about the terminal (or stateful) operation which follows the last intermediate operation described by this PipelineHelper. The PipelineHelper is passed to the TerminalOp.evaluateParallel(PipelineHelper, Spliterator), TerminalOp.evaluateSequential(PipelineHelper, Spliterator), and AbstractPipeline.opEvaluateParallel(PipelineHelper, Spliterator, IntFunction), methods, which can use the PipelineHelper to access information about the pipeline such as head shape, stream flags, and size, and use the helper methods such as wrapAndCopyInto(Sink, Spliterator), copyInto(Sink, Spliterator), and wrapSink(Sink) to execute pipeline operations.

注释中说，这个类捕获中间操作、流的flags、是否并行等。但是仅仅看注释其实我们是看不懂的，不过从后面的我们的分析可以得到，它封装了关于求值的方法。同时`AbstractPipeline`继承了`PipelineHelper`。这个类的命名个人很难接受，因为感觉类的功能和类名完全没有联系，如果改为`computeHelper`反而更好理解。不过这已无关紧要。

我们下面来寻找`evaluateSequential()`的实现类。在上面的分析中，我们知道，传递给`evaluate()`方法的实际上是`OfRef`类，而`OfRef`又继承了`ForEachOp`，同时在其中有如下的实现：

```java
public <S> Void evaluateSequential(PipelineHelper<T> helper, Spliterator<S> spliterator) {
    return helper.wrapAndCopyInto(this, spliterator).get();
}
```

很显然，直接跟进去`wrapAndCopyInto()`又是一个待实现的方法。所以我们又要回溯，一致回溯到`evaluate()`，可以发现，我们传递进去的`helper`是`AbstractPipeline`的对象。在这个类中，我们找到了如下实现：

```java
final <P_IN, S extends Sink<E_OUT>> S wrapAndCopyInto(S sink, Spliterator<P_IN> spliterator) {
    copyInto(wrapSink(Objects.requireNonNull(sink)), spliterator);
    return sink;
}
```

在这里我们遇到了之前见过但是没有分析的接口`Sink`。在哪见得呢？在`filter()`和`map()`的实现中。

```java
public final <R> Stream<R> map(Function<? super P_OUT, ? extends R> mapper) {
    Objects.requireNonNull(mapper);
    return new StatelessOp<P_OUT, R>(this, StreamShape.REFERENCE,
                                     StreamOpFlag.NOT_SORTED | StreamOpFlag.NOT_DISTINCT) {
        @Override
        Sink<P_OUT> opWrapSink(int flags, Sink<R> sink) {
            return new Sink.ChainedReference<P_OUT, R>(sink) {
                @Override
                public void accept(P_OUT u) {
                    downstream.accept(mapper.apply(u));
                }
            };
        }
    };
}
```

在创建`StatelessOp`的对象时，我们覆盖了`opWrapSink()`方法，这个方法返回一个`Sink`。什么是`Sink`？

```java
interface Sink<T> extends Consumer<T> {....}
```

从定义上来看，它是继承了`Consumer`的一个接口。我们一般管道的每一段称为一个`Stage`。从我们上一张图中可以看见，`Stage`之间是通过双向链表连接起来的，但是我们却没有说每一段的行为是怎么被保存的。而怎么保存以及后续怎么调用便是由`Sink`来完成。

`Sink`它也是一个`Consumer`，但是比普通的`Comsumer`多了一些功能，JDK注释如下：

> An extension of Consumer used to conduct values through the stages of a stream pipeline, with additional methods to manage size information, control flow, etc. Before calling the accept() method on a Sink for the first time, you must first call the begin() method to inform it that data is coming (optionally informing the sink how much data is coming), and after all data has been sent, you must call the end() method. After calling end(), you should not call accept() without again calling begin(). Sink also offers a mechanism by which the sink can cooperatively signal that it does not wish to receive any more data (the cancellationRequested() method), which a source can poll before sending more data to the Sink.
> A sink may be in one of two states: an initial state and an active state. It starts out in the initial state; the begin() method transitions it to the active state, and the end() method transitions it back into the initial state, where it can be re-used. Data-accepting methods (such as accept() are only valid in the active state.

看到这里，我们知道了`Sink`的作用是进行流中元素的控制的，比如将一个`Consumer`转为两个阶段：初始阶段和激活阶段。但是总感觉还是很迷，这种操作有什么用呢？直接使用`Consumer`不行吗？在我们继续分析之前，我们将上面的那张图再做一下更新。

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/edbd1b1d-0050-426c-a1ce-1f3b86b53240" /></div>
在图中，向下的箭头表示包含的意思。

到了这里，我们便可以继续分析`wrapAndCopyInto()`。同时我们在`ForEachOp`中发现在调用这个方法的时候传的参数是`this`。这个`this`指的是`OfRef`的对象，也就是一个终止操作的对象。

再向下跟进，我们就进入了`wrapSink()`

```java
final <P_IN> Sink<P_IN> wrapSink(Sink<E_OUT> sink) {
    Objects.requireNonNull(sink);

    for (AbstractPipeline p=AbstractPipeline.this; p.depth > 0; p=p.previousStage) {
        sink = p.opWrapSink(p.previousStage.combinedFlags, sink);
    }
    return (Sink<P_IN>) sink;
}
```

其中的`AbstractPipeline.this`就是我们最后一个`ReferencePipeline`，这个对象有指针指向前面的`Stage`。

此时我们就可以分析一下`opWrapSink()`，它是每一个Stream里面供我们使用的操作都会去覆盖的方法，它返回一个`ChainedReference`，这个类里面有一个属性`Sink`，他是下游的`Sink`，也就是说在每个Sink里都会存在一个指向下游`Sink`的引用。

当我们第一次执行上面的代码的时候`sink`是终止操作，`p`是`MapSink`；当我们第而次执行上面的代码的时候`sink`是`MapSink`，`p`是`FileterSink`。所以当这段代码执行完成的时候我们上面的所以`Sink`对象会被关联起来，按照我们的例子，应该有下图：

<div align="center"><img width="40%" src="http://blogfileqiniu.isjinhao.site/d7b67844-4874-4630-82e8-11afc0c402cd" /></div>
Sink组装好了之后便会调用`copyInto()`：

```java
final <P_IN> void copyInto(Sink<P_IN> wrappedSink, Spliterator<P_IN> spliterator) {
    Objects.requireNonNull(wrappedSink);

    if (!StreamOpFlag.SHORT_CIRCUIT.isKnown(getStreamAndOpFlags())) {
        wrappedSink.begin(spliterator.getExactSizeIfKnown());
        spliterator.forEachRemaining(wrappedSink);
        wrappedSink.end();
    } else {
        copyIntoWithCancel(wrappedSink, spliterator);
    }
}
```

在这段代码中我们可以看到，如果不包含短路操作，便执行`spliterator.forEachRemaining(wrappedSink);`，我们以`Collection`的`spliterator`为例，看看最终是怎么执行的。

```java
public void forEachRemaining(Consumer<? super T> action) {
    if (action == null) throw new NullPointerException();
    Iterator<? extends T> i;
    if ((i = it) == null) {
        i = it = collection.iterator();
        est = (long)collection.size();
    }
    i.forEachRemaining(action);
}
```

```java
public boolean tryAdvance(Consumer<? super T> action) {
    if (action == null) throw new NullPointerException();
    if (it == null) {
        it = collection.iterator();
        est = (long) collection.size();
    }
    if (it.hasNext()) {
        action.accept(it.next());
        return true;
    }
    return false;
}
```

如果我们有一个元素，设置item。item首先会进入`FilterSink`，在其中执行：

```java
public void accept(P_OUT u) {
    if (predicate.test(u))
        downstream.accept(u);
}
```

如果`test`为真，调用下游的Sink；下游的Sink是`MapSink`，此时执行：

```java
public void accept(P_OUT u) {
    downstream.accept(mapper.apply(u));
}
```

这段代码中先执行`mapper.apply(u)`，然后执行`downstream.accept`，而此时的`downstream`是`ForEachSink`；然后就会执行`OfRef`的：

```java
public void accept(T t) {
    consumer.accept(t);
}
```

此时的`consumer`是`System.out.println`，最终输出结果。

至此，流的执行分解完成。