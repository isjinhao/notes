## 集合概览

<div align="center"><img width="70%" src="http://q0l9qvfyx.bkt.clouddn.com/f6aa99af-bd85-4f36-a113-c999a4eac0d4"></div>
### List

- **Arraylist：** Object数组
- **Vector：** Object数组，线程安全。
- **LinkedList：** 双向链表



### Map

- **HashMap：** JDK1.8之前HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。JDK1.8以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间
- **LinkedHashMap：** LinkedHashMap 继承自 HashMap，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，LinkedHashMap 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。
- **TreeMap：** 红黑树



### Set

- **HashSet（无序，唯一）:** 基于 HashMap 实现的，底层采用 HashMap 来保存元素
- **LinkedHashSet：** LinkedHashSet 继承与 HashSet，并且其内部是通过 LinkedHashMap 来实现的。
- **TreeSet（有序，唯一）：** 红黑树（自平衡的排序二叉树)



## ArrayList分析

**ArrayList有三种方式来初始化，构造方法源码如下：**

```java
/**
 * 默认初始容量大小
 */
private static final int DEFAULT_CAPACITY = 10;

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * 默认构造函数，使用初始容量10构造一个空列表(无参数构造)
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**
 * 带初始容量参数的构造函数。（用户自己指定容量）
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {	// 初始容量大于0
        // 创建initialCapacity大小的数组
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {	// 初始容量等于0
        //创建空数组
        this.elementData = EMPTY_ELEMENTDATA;
    } else {	// 初始容量小于0，抛出异常
        throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
    }
}

/**
* 构造包含指定collection元素的列表，这些元素利用该集合的迭代器按顺序返回
* 如果指定的集合为null，throws NullPointerException。 
*/
public ArrayList(Collection<? extends E> c) {
	elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

细心的同学一定会发现 ：**以无参数构造方法创建 ArrayList 时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为10。** 下面在我们分析 ArrayList 扩容时会讲到这一点内容！



### add

```java
/**
 * 将指定的元素追加到此列表的末尾。 
 */
public boolean add(E e) {
	// 添加元素之前，先调用ensureCapacityInternal方法
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 这里看到ArrayList添加元素的实质就相当于为数组赋值
    elementData[size++] = e;
    return true;
}
```



### ensureCapacityInternal()

```java
// 获得容量
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

- 当我们要 add 进第1个元素到 ArrayList 时，minCapacity为1，在Math.max()方法比较后，minCapacity 为10。elementData.length 为0 （因为还是一个空的 list），因为执行了 `ensureCapacityInternal()` 方法 ，所以 minCapacity 此时为10。此时，`minCapacity - elementData.length > 0 `成立，所以会进入 `grow(minCapacity)` 方法。
- 当add第2个元素时，minCapacity 为2，此时e lementData.length(容量)在添加第一个元素后扩容成 10 了。此时，`minCapacity - elementData.length > 0 `不成立，所以不会进入 （执行）`grow(minCapacity)` 方法。
- 添加第3、4···到第10个元素时，依然不会执行grow方法，数组容量都为10。

直到添加第11个元素，minCapacity(为11)比elementData.length（为10）要大。进入grow方法进行扩容。



### `grow()`

```java
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

private void grow(int minCapacity) {
    // oldCapacity为旧容量，newCapacity为新容量
    int oldCapacity = elementData.length;
    // 将oldCapacity 右移一位，其效果相当于oldCapacity /2，
    // 我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    // 若新容量大于MAX_ARRAY_SIZE，执行hugeCapacity()方法来比较minCapacity和MAX_ARRAY_SIZE，
    // 若minCapacity大于最大容量，则新容量则为Integer.MAX_VALUE，否则，新容量大小则为 
    // MAX_ARRAY_SIZE 即为Integer.MAX_VALUE - 8。
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
}
```

所以 ArrayList 每次扩容之后容量都会变为原来的 1.5 倍！

> ">>"（移位运算符）：>>1 右移一位相当于除2，右移n位相当于除以 2 的 n 次方。这里 oldCapacity 明显右移了1位所以相当于oldCapacity /2。对于大数据的2进制运算，位移运算符比那些普通运算符的运算要快很多,因为程序仅仅移动一下而已，不去计算，这样提高了效率，节省了资源 　

**我们再来通过例子探究一下grow() 方法 ：**

- 当add第1个元素时，oldCapacity 为0，经比较后第一个if判断成立，newCapacity = minCapacity(为10)。但是第二个if判断不会成立，即newCapacity 不比 MAX_ARRAY_SIZE大，则不会进入 `hugeCapacity` 方法。数组容量为10，add方法中 return true,size增为1。
- 当add第11个元素进入grow方法时，newCapacity为15，比minCapacity（为11）大，第一个if判断不成立。新容量没有大于数组最大size，不会进入hugeCapacity方法。数组容量扩为15，add方法中return true，size增为11。
- 以此类推······

如果新容量大于 MAX_ARRAY_SIZE,进入(执行) `hugeCapacity()` 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，如果minCapacity大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 `Integer.MAX_VALUE - 8`。

**这里补充一点比较重要，但是容易被忽视掉的知识点：**

- java 中的 `length `属性是针对数组说的,比如说你声明了一个数组,想知道这个数组的长度则用到了 length 这个属性.
- java 中的 `length()` 方法是针对字符串说的,如果想看这个字符串的长度则用到 `length()` 这个方法.
- java 中的 `size()` 方法是针对泛型集合说的,如果想看这个泛型有多少个元素,就调用此方法来查看!



### System.arraycopy()

```java
/**
 * 在此列表中的指定位置插入指定的元素。 
 * 先调用 rangeCheckForAdd 对index进行界限检查；然后调用 ensureCapacityInternal 方法保证
 * capacity足够大；再将从index开始之后的所有成员后移一个位置；将element插入index位置；最后size加1。
 */
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // arraycopy()方法实现数组自己复制自己
    // elementData:源数组;index:源数组中的起始位置;elementData：目标数组；index + 1：目标数组中的
    // 起始位置； size - index：要复制的数组元素的数量；
    System.arraycopy(elementData, index, elementData, index + 1, size - index);
    elementData[index] = element;
    size++;
}
```



### Arrays.copyOf()

```java
public static <T> T[] copyOf(T[] original, int newLength) {
    return (T[]) copyOf(original, newLength, original.getClass());
}

public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0, Math.min(original.length, newLength));
    return copy;
}
```

个人觉得使用 `Arrays.copyOf()`方法主要是为了给原有数组扩容。



### ensureCapacity

ArrayList 源码中有一个 `ensureCapacity` 方法不知道大家注意到没有，这个方法 ArrayList 内部没有被调用过，所以很显然是提供给用户调用的，那么这个方法有什么作用呢？

```java
/**
 * 如有必要，增加此 ArrayList 实例的容量，以确保它至少可以容纳由minimum capacity参数指定的元素数。
 * @param   minCapacity   所需的最小容量
 */
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
        // any size if not default element table
        ? 0
        // larger than default for default empty table. It's already
        // supposed to be at default size.
        : DEFAULT_CAPACITY;

    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}
```

**最好在 add 大量元素之前用 ensureCapacity 方法，以减少增量重新分配的次数**

我们通过下面的代码实际测试以下这个方法的效果：

```java
public class EnsureCapacityTest {
	public static void main(String[] args) {
		ArrayList<Object> list = new ArrayList<Object>();
		final int N = 10000000;
		long startTime = System.currentTimeMillis();
		for (int i = 0; i < N; i++) {
			list.add(i);
		}
		long endTime = System.currentTimeMillis();
		System.out.println("使用ensureCapacity方法前："+(endTime - startTime));

		list = new ArrayList<Object>();
		long startTime1 = System.currentTimeMillis();
		list.ensureCapacity(N);
		for (int i = 0; i < N; i++) {
			list.add(i);
		}
		long endTime1 = System.currentTimeMillis();
		System.out.println("使用ensureCapacity方法后："+(endTime1 - startTime1));
	}
}
```

运行结果：

```
使用ensureCapacity方法前：4637
使用ensureCapacity方法后：241
```

通过运行结果，我们可以很明显的看出向 ArrayList 添加大量元素之前最好先使用`ensureCapacity` 方法，以减少增量重新分配的次数。



## LinkedList分析

LinkedList是一个实现了List接口和Deque接口的双端链表。 LinkedList底层的链表结构使它支持高效的插入和删除操作，另外它实现了Deque接口，使得LinkedList类也具有队列的特性；LinkedList不是线程安全的，如果想使LinkedList变成线程安全的，可以调用静态类Collections类中的synchronizedList方法：

```java
List list = Collections.synchronizedList(new LinkedList(...));
```



### 内部结构分析

```java
private static class Node<E> {
	E item;//节点值
    Node<E> next;//后继节点
    Node<E> prev;//前驱节点
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

这个类就代表双端链表的节点Node。这个类有三个属性，分别是前驱节点，本节点的值，后继结点。



### 构造方法

**空构造方法：**

```java
public LinkedList() { }
```

**用已有的集合创建链表的构造方法：**

```java
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```



### add方法

**add(E e)** 方法：将元素添加到链表尾部

```java
public boolean add(E e) {
    linkLast(e); // 这里就只调用了这一个方法
    return true;
}
/**
* 链接使e作为最后一个元素。
*/
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode; // 新建节点
    if (l == null)
    	first = newNode;
    else
    	l.next = newNode; // 指向后继元素也就是指向下一个元素
    size++;
    modCount++;
}
```

**add(int index,E e)**：在指定位置添加元素

```java
public void add(int index, E element) {
    checkPositionIndex(index); // 检查索引是否处于[0-size]之间

    if (index == size) // 添加在链表尾部
        linkLast(element);
    else // 添加在链表中间
        linkBefore(element, node(index));
}
```

linkBefore方法需要给定两个参数，一个插入节点的值，一个指定的node，所以我们又调用了Node(index)去找到index对应的node

**addAll(Collection c )：将集合插入到链表尾部**

```java
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}
```

**addAll(int index, Collection c)：** 将集合从指定位置开始插入

```java
public boolean addAll(int index, Collection<? extends E> c) {
    // 1:检查index范围是否在size之内
    checkPositionIndex(index);

    // 2:toArray()方法把集合的数据存到对象数组中
    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;

    // 3：得到插入位置的前驱节点和后继节点
    Node<E> pred, succ;
    // 如果插入位置为尾部，前驱节点为last，后继节点为null
    if (index == size) {
        succ = null;
        pred = last;
    }
    // 否则，调用node()方法得到后继节点，再得到前驱节点
    else {
        succ = node(index);
        pred = succ.prev;
    }

    // 4：遍历数据将数据插入
    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        //创建新节点
        Node<E> newNode = new Node<>(pred, e, null);
        //如果插入位置在链表头部
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }

    //如果插入位置在尾部，重置last节点
    if (succ == null) {
        last = pred;
    }
    //否则，将插入的链表与先前链表连接起来
    else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}    
```

上面可以看出addAll方法通常包括下面四个步骤：

1. 检查index范围是否在size之内
2. toArray()方法把集合的数据存到对象数组中
3. 得到插入位置的前驱和后继节点
4. 遍历数据，将数据插入到指定位置

**addFirst(E e)：** 将元素添加到链表头部

```java
 public void addFirst(E e) {
        linkFirst(e);
    }
private void linkFirst(E e) {
    final Node<E> f = first;
    final Node<E> newNode = new Node<>(null, e, f); // 新建节点，以头节点为后继节点
    first = newNode;
    // 如果链表为空，last节点也指向该节点
    if (f == null)
        last = newNode;
    // 否则，将头节点的前驱指针指向新节点，也就是指向前一个元素
    else
        f.prev = newNode;
    size++;
    modCount++;
}
```

**addLast(E e)：** 将元素添加到链表尾部，与 **add(E e)** 方法一样

```java
public void addLast(E e) {
    linkLast(e);
}
```



### 根据位置取数据的方法

**get(int index)：** 根据指定索引返回数据

```java
public E get(int index) {
    //检查index范围是否在size之内
    checkElementIndex(index);
    //调用Node(index)去找到index对应的node然后返回它的值
    return node(index).item;
}
```

**获取头节点（index=0）数据方法:**

```java
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}
public E element() {
    return getFirst();
}
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}

public E peekFirst() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}
```

**区别：** getFirst()，element()，peek()，peekFirst() 这四个获取头结点方法的区别在于对链表为空时的处理，是抛出异常还是返回null，其中**getFirst()** 和**element()** 方法将会在链表为空时，抛出异常

element()方法的内部就是使用getFirst()实现的。它们会在链表为空时，抛出NoSuchElementException
**获取尾节点（index=-1）数据方法:**

```java
 public E getLast() {
     final Node<E> l = last;
     if (l == null)
         throw new NoSuchElementException();
     return l.item;
 }
 public E peekLast() {
     final Node<E> l = last;
     return (l == null) ? null : l.item;
 }
```

**两者区别：** **getLast()** 方法在链表为空时，会抛出**NoSuchElementException**，而**peekLast()** 则不会，只是会返回 **null**。



### 根据对象得到索引的方法

**int indexOf(Object o)：** 从头遍历找

```java
public int indexOf(Object o) {
    int index = 0;
    if (o == null) {
        // 从头遍历
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    } else {
        // 从头遍历
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}
```

**int lastIndexOf(Object o)：** 从尾遍历找

```java
public int lastIndexOf(Object o) {
    int index = size;
    if (o == null) {
        // 从尾遍历
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (x.item == null)
                return index;
        }
    } else {
        // 从尾遍历
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (o.equals(x.item))
                return index;
        }
    }
    return -1;
}
```



### 检查链表是否包含某对象的方法

**contains(Object o)：** 检查对象o是否存在于链表中

```java
 public boolean contains(Object o) {
     return indexOf(o) != -1;
 }
```



### 删除方法

**remove()** ,**removeFirst(),pop():** 删除头节点

```java
public E pop() {
    return removeFirst();
}
public E remove() {
    return removeFirst();
}
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}
```

**removeLast(),pollLast():** 删除尾节点

```java
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}
public E pollLast() {
    final Node<E> l = last;
    return (l == null) ? null : unlinkLast(l);
}
```

**区别：** removeLast()在链表为空时将抛出NoSuchElementException，而pollLast()方法返回null。

**remove(Object o):** 删除指定元素

```java
public boolean remove(Object o) {
    // 如果删除对象为null
    if (o == null) {
        // 从头开始遍历
        for (Node<E> x = first; x != null; x = x.next) {
            // 找到元素
            if (x.item == null) {
                // 从链表中移除找到的元素
                unlink(x);
                return true;
            }
        }
    } else {
        //从头开始遍历
        for (Node<E> x = first; x != null; x = x.next) {
            //找到元素
            if (o.equals(x.item)) {
                //从链表中移除找到的元素
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

当删除指定对象时，只需调用remove(Object o)即可，不过该方法一次只会删除一个匹配的对象，如果删除了匹配对象，返回true，否则false。

unlink(Node x) 方法：

```java
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next; // 得到后继节点
    final Node<E> prev = x.prev; // 得到前驱节点

    // 删除前驱指针
    if (prev == null) {
        first = next; // 如果删除的节点是头节点,令头节点指向该节点的后继节点
    } else {
        prev.next = next; // 将前驱节点的后继节点指向后继节点
        x.prev = null;
    }

    // 删除后继指针
    if (next == null) {
        last = prev; // 如果删除的节点是尾节点,令尾节点指向该节点的前驱节点
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```

**remove(int index)**：删除指定位置的元素

```java
public E remove(int index) {
    // 检查index范围
    checkElementIndex(index);
    // 将节点删除
    return unlink(node(index));
}
```



### LinkedList类常用方法测试

```java
public class LinkedListDemo {
    public static void main(String[] srgs) {
        // 创建存放int类型的linkedList
        LinkedList<Integer> linkedList = new LinkedList<>();
        /************************** linkedList的基本操作 ************************/
        linkedList.addFirst(0); // 添加元素到列表开头
        linkedList.add(1);     // 在列表结尾添加元素
        linkedList.add(2, 2); // 在指定位置添加元素
        linkedList.addLast(3); // 添加元素到列表结尾
        
        System.out.println("LinkedList（直接输出的）: " + linkedList);

        System.out.println("getFirst()获得第一个元素: " + 
                           linkedList.getFirst()); // 返回此列表的第一个元素
        System.out.println("getLast()获得第最后一个元素: " + 
                           linkedList.getLast()); // 返回此列表的最后一个元素
        System.out.println("removeFirst()删除第一个元素并返回: " + 
                           linkedList.removeFirst()); // 移除并返回此列表的第一个元素
        System.out.println("removeLast()删除最后一个元素并返回: " + 
                           linkedList.removeLast()); // 移除并返回此列表的最后一个元素
        System.out.println("After remove:" + linkedList);
        System.out.println("contains()方法判断列表是否包含1这个元素:" + 
                           linkedList.contains(1)); // 判断此列表包含指定元素，如果是则返回真
        System.out.println("该linkedList的大小 : " +
                           linkedList.size()); // 返回此列表的元素个数

        /************************** 位置访问操作 ************************/
        System.out.println("-----------------------------------------");
        linkedList.set(1, 3); // 将此列表中指定位置的元素替换为指定的元素
        System.out.println("After set(1, 3):" + linkedList);
        System.out.println("get(1)获得指定位置（这里为1）的元素: " +
                           linkedList.get(1)); // 返回此列表中指定位置处的元素

        /************************** Search操作 ************************/
        System.out.println("-----------------------------------------");
        linkedList.add(3);
        System.out.println("indexOf(3): " +
                           linkedList.indexOf(3)); // 返回此列表中首次出现的指定元素的索引
        System.out.println("lastIndexOf(3): " +
                           linkedList.lastIndexOf(3));// 返回此列表中最后出现的指定元素的索引

        /************************** Queue操作 ************************/
        System.out.println("-----------------------------------------");
        System.out.println("peek(): " + linkedList.peek()); // 获取但不移除此列表的头
        System.out.println("element(): " + linkedList.element()); // 获取但不移除此列表的头
        linkedList.poll(); // 获取并移除此列表的头
        System.out.println("After poll():" + linkedList);
        linkedList.remove();
        System.out.println("After remove():" + linkedList); // 获取并移除此列表的头
        linkedList.offer(4);
        System.out.println("After offer(4):" + linkedList); // 将指定元素添加到此列表的末尾

        /************************** Deque操作 ************************/
        System.out.println("-----------------------------------------");
        linkedList.offerFirst(2); // 在此列表的开头插入指定的元素
        System.out.println("After offerFirst(2):" + linkedList);
        linkedList.offerLast(5); // 在此列表末尾插入指定的元素
        System.out.println("After offerLast(5):" + linkedList);
        System.out.println("peekFirst(): " +
                           linkedList.peekFirst()); // 获取但不移除此列表的第一个元素
        System.out.println("peekLast(): " +
                           linkedList.peekLast()); // 获取但不移除此列表的第一个元素
        linkedList.pollFirst(); // 获取并移除此列表的第一个元素
        System.out.println("After pollFirst():" + linkedList);
        linkedList.pollLast(); // 获取并移除此列表的最后一个元素
        System.out.println("After pollLast():" + linkedList);
        linkedList.push(2); // 将元素推入此列表所表示的堆栈（插入到列表的头）
        System.out.println("After push(2):" + linkedList);
        linkedList.pop(); // 从此列表所表示的堆栈处弹出一个元素（获取并移除列表第一个元素）
        System.out.println("After pop():" + linkedList);
        linkedList.add(3);
        linkedList.removeFirstOccurrence(3); // 移除第一次出现的指定元素（从头到尾遍历）
        System.out.println("After removeFirstOccurrence(3):" + linkedList);
        linkedList.removeLastOccurrence(3); // 移除最后一次出现的指定元素（从尾到头遍历）
        System.out.println("After removeFirstOccurrence(3):" + linkedList);

        /************************** 遍历操作 ************************/
        System.out.println("-----------------------------------------");
        linkedList.clear();
        for (int i = 0; i < 100000; i++) {
            linkedList.add(i);
        }
        // 迭代器遍历
        long start = System.currentTimeMillis();
        Iterator<Integer> iterator = linkedList.iterator();
        while (iterator.hasNext()) {
            iterator.next();
        }
        long end = System.currentTimeMillis();
        System.out.println("Iterator：" + (end - start) + " ms");

        // 顺序遍历(随机遍历)
        start = System.currentTimeMillis();
        for (int i = 0; i < linkedList.size(); i++) {
            linkedList.get(i);
        }
        end = System.currentTimeMillis();
        System.out.println("for：" + (end - start) + " ms");

        // 另一种for循环遍历
        start = System.currentTimeMillis();
        for (Integer i : linkedList)
            ;
        end = System.currentTimeMillis();
        System.out.println("for2：" + (end - start) + " ms");

        // 通过pollFirst()或pollLast()来遍历LinkedList
        LinkedList<Integer> temp1 = new LinkedList<>();
        temp1.addAll(linkedList);
        start = System.currentTimeMillis();
        while (temp1.size() != 0) {
            temp1.pollFirst();
        }
        end = System.currentTimeMillis();
        System.out.println("pollFirst()或pollLast()：" + (end - start) + " ms");

        // 通过removeFirst()或removeLast()来遍历LinkedList
        LinkedList<Integer> temp2 = new LinkedList<>();
        temp2.addAll(linkedList);
        start = System.currentTimeMillis();
        while (temp2.size() != 0) {
            temp2.removeFirst();
        }
        end = System.currentTimeMillis();
        System.out.println("removeFirst()或removeLast()：" + (end - start) + " ms");
    }
}
```



## Arrays.asList

`Arrays.asList()`将数组转换为集合后,底层其实还是数组。**传递的数组必须是对象数组，而不是基本类型。**`Arrays.asList()`是泛型方法，传入的对象必须是对象数组。

```java
int[] myArray = { 1, 2, 3 };
List myList = Arrays.asList(myArray);
System.out.println(myList.size()); // 1
System.out.println(myList.get(0)); // 数组地址值
System.out.println(myList.get(1)); // 报错：ArrayIndexOutOfBoundsException
int [] array=(int[]) myList.get(0);
System.out.println(array[0]); // 1
```

当传入一个原生数据类型数组时，`Arrays.asList()` 的真正得到的参数就不是数组中的元素，而是数组对象本身！此时List 的唯一元素就是这个数组，这也就解释了上面的代码。我们使用包装类型数组就可以解决这个问题。

```java
Integer[] myArray = { 1, 2, 3 };
```

**使用集合的修改方法:add()、remove()、clear()会抛出异常。**`Arrays.asList()` 方法返回的并不是 `java.util.ArrayList` ，而是 `java.util.Arrays` 的一个内部类,这个内部类并没有实现集合的修改方法或者说并没有重写这些方法。

```java
private static class ArrayList<E> extends 
	AbstractList<E> implements RandomAccess, java.io.Serializable {
	...
    @Override
    public E get(int index) {
      ...
    }
    @Override
    public E set(int index, E element) {
      ...
    }
    @Override
    public int indexOf(Object o) {
      ...
    }
    @Override
    public boolean contains(Object o) {
       ...
    }
    @Override
    public void forEach(Consumer<? super E> action) {
      ...
    }
    @Override
    public void replaceAll(UnaryOperator<E> operator) {
      ...
    }
    @Override
    public void sort(Comparator<? super E> c) {
      ...
    }
}
```

我们再看一下`java.util.AbstractList`的`remove()`方法，这样我们就明白为啥会抛出`UnsupportedOperationException`。

```java
public E remove(int index) {
    throw new UnsupportedOperationException();
}
```

正确的数组转集合的方法：`List list = new ArrayList<>(Arrays.asList("a", "b", "c"))`



## Comparable & Comparator

### Comparable

> This interface imposes a total ordering on the objects of each class that implements it. This ordering is referred to as the class’s natural ordering, and the class’s compareTo method is referred to as its natural comparison method.
>
> Lists (and arrays) of objects that implement this interface can be sorted automatically by Collections.sort (and Arrays.sort). Objects that implement this interface can be used as keys in a sorted map or as elements in a sorted set, without the need to specify a comparator.

compareTo 方法的返回值有三种情况：

- e1.compareTo(e2) > 0 即 e1 > e2
- e1.compareTo(e2) = 0 即 e1 = e2
- e1.compareTo(e2) < 0 即 e1 < e2

满足上述规则的是升序排序，与上诉规则相反则是降序排序。

````java
public class Test {
    public static void main(String[] args) throws IOException  {
    	Set<Person> set = new TreeSet<>();
    	set.add(new Person(50));
    	set.add(new Person(30));
    	set.add(new Person(90));
    	Iterator<Person> iterator = set.iterator();
    	while(iterator.hasNext())
    		System.out.println(iterator.next());
    } 
}
class Person implements Comparable<Person>{
	private Integer age;
	public Person(Integer age) { this.age = age; }
	@Override
	public int compareTo(Person o) {
		if(o == null)
			throw new NullPointerException();
		if(age > o.age)
			return 1;
		else if(age < o.age)
			return -1;
		return 0;
	}
	public String toString() { return "Person [age=" + age + "]"; }
}
````

> Note that null is not an instance of any class, and e.compareTo(null) should throw a NullPointerException even though e.equals(null) returns false.

> The natural ordering for a class C is said to be consistent with equals if and only if e1.compareTo(e2) == 0 has the same boolean value as e1.equals(e2) for every e1 and e2 of class C. It is strongly recommended (though not required) that natural orderings be consistent with equals. This is so because sorted sets (and sorted maps)without explicit comparators behave “strangely” when they are used with elements (or keys) whose natural ordering is inconsistent with equals. In particular, such a sorted set (or sorted map) violates（违反） the general contract for set (or map), which is defined in terms of（依据） the equals method.
>
> For example, if one adds two keys a and b such that (!a.equals(b) && a.compareTo(b) == 0) to a sorted set that does not use an explicit comparator, the second add operation returns false (and the size of the sorted set does not increase) because a and b are equivalent from the sorted set’s perspective.

这段话的意思是如果compareTo规则和equals规则不同就会发生奇怪的问题，即在`!a.equals(b) && a.compareTo(b) == 0`这种情况下，不能插入集合中，因为从排序集合判断是否相等使用compareTo的规则。

```java
public class Test {
    public static void main(String[] args) throws IOException {
    	Set<Person> set = new TreeSet<>();
    	set.add(new Person(50));
    	set.add(new Person(50));
    	set.add(new Person(90));
    	Iterator<Person> iterator = set.iterator();
    	while(iterator.hasNext())
    		System.out.println(iterator.next());
    }
}
class Person implements Comparable<Person>{
	private Integer age;
	public Person(Integer age) { this.age = age; }
	@Override
	public int compareTo(Person o) {
		if(o == null)
			throw new NullPointerException();
		if(age == o.age)	// 模仿 a.compareTo(b) == 0
			return 0;
		if(age > o.age)
			return 1;
		return -1;
	}
	@Override
	public String toString() { return "Person [age=" + age + "]"; }
	@Override
	public boolean equals(Object obj) {
		if(obj == null || !(obj instanceof Person))
			return false;
		Person p = (Person)obj;
		if(this.age == 50 || p.age == 50) {			// 模仿 !a.equals(b)
			return false;
		}
		return (this.age == p.age) ? true : false;
	}
	@Override
	public int hashCode() { return age.hashCode(); }
}
// Person [age=50]
// Person [age=90]
```



### Comparator

使用自然排序需要类实现 Comparable，并且在内部重写 comparaTo 方法。而 Comparator 则是在外部制定排序规则，然后作为排序策略参数传递给某些类，比如 Collections.sort(), Arrays.sort(), 或者一些内部有序的集合（比如 SortedSet，SortedMap 等）。**同时存在时采用 Comparator（定制排序）的规则进行比较。**

使用方式主要分三步：

1. 创建一个 Comparator 接口的实现类，并赋值给一个对象。在 compare 方法中针对自定义类写排序规则。
2. 将 Comparator 对象作为参数传递给 排序类的某个方法
3. 向排序类中添加 compare 方法中使用的自定义类

```java
public class Test {
    public static void main(String[] args) throws IOException  {
    	Comparator<Person> com = new Comparator<Person>() {
			@Override
			public int compare(Person o1, Person o2) {
				if(o1.getAge() > o2.getAge())
					return 1;
				else if(o1.getAge() < o2.getAge())
					return -1;
				return 0;
			}
		};
    	Set<Person> set = new TreeSet<>(com);
    	set.add(new Person(50));
    	set.add(new Person(20));
    	set.add(new Person(90));
    	Iterator<Person> iterator = set.iterator();
    	while(iterator.hasNext())
    		System.out.println(iterator.next());
    } 
}

class Person{
	private Integer age;
	public Person(Integer age) { this.age = age; }
	@Override
	public String toString() { return "Person [age=" + age + "]"; }
	public Integer getAge() { return age; }
	public void setAge(Integer age) { this.age = age; }
}
```

需要注意的一点是，compare的相等规则需要个equals相同。



## Collections和Arrays常见方法

### Collections

#### 排序操作

```java
void reverse(List list)	//反转
void shuffle(List list)	//随机排序
void sort(List list)	//按自然排序的升序排序
void sort(List list, Comparator c)	//定制排序，由Comparator控制排序逻辑
void swap(List list, int i , int j)	//交换两个索引位置的元素
void rotate(List list, int distance)//旋转。当distance为正数时，将list后distance个元素整体移到前面。当distance为负数时，将 list的前distance个元素整体移到后面。
```

#### 查找，替换操作

```java
int binarySearch(List list, Object key)//对List进行二分查找，返回索引，注意List必须是有序的
    
int max(Collection coll)//根据元素的自然顺序，返回最大的元素。
int min(Collection coll)

int max(Collection coll, Comparator c)//根据定制排序，返回最大元素，排序规则由Comparatator类控制。类比
int min(Collection coll, Comparator c)

void fill(List list, Object obj)//用指定的元素代替指定list中的所有元素。

int frequency(Collection c, Object o)//统计元素出现次数

int indexOfSubList(List list, List target)//统计target在list中第一次出现的索引，找不到则返回-1
int lastIndexOfSubList(List source, list target)

boolean replaceAll(List list, Object oldVal, Object newVal) //用新元素替换旧元素

```



### Arrays

#### 排序 : `sort()`

```java
int a[] = { 1, 3, 2, 7, 6, 5, 4, 9 };
// sort(int[] a)方法按照数字顺序排列指定的数组。
Arrays.sort(a);
System.out.println("Arrays.sort(a):");
for (int i : a) {
	System.out.print(i);
}
// 换行
System.out.println();

// sort(int[] a,int fromIndex,int toIndex)按升序排列数组的指定范围
int b[] = { 1, 3, 2, 7, 6, 5, 4, 9 };
Arrays.sort(b, 2, 6);
System.out.println("Arrays.sort(b, 2, 6):");
for (int i : b) {
	System.out.print(i);
}

// 换行
System.out.println();

int c[] = { 1, 3, 2, 7, 6, 5, 4, 9 };
// parallelSort(int[] a) 按照数字顺序排列指定的数组(并行的)。同sort方法一样也有按范围的排序
Arrays.parallelSort(c);
System.out.println("Arrays.parallelSort(c)：");
for (int i : c) {
	System.out.print(i);
}

// 换行
System.out.println();

// parallelSort给字符数组排序，sort也可以
char d[] = { 'a', 'f', 'b', 'c', 'e', 'A', 'C', 'B' };
Arrays.parallelSort(d);
System.out.println("Arrays.parallelSort(d)：");
for (char d2 : d) {
	System.out.print(d2);
}
```

#### 查找 : `binarySearch()`

```java
char[] e = { 'a', 'f', 'b', 'c', 'e', 'A', 'C', 'B' };
// 排序后再进行二分查找，否则找不到
Arrays.sort(e);
System.out.println("Arrays.sort(e)" + Arrays.toString(e));
System.out.println("Arrays.binarySearch(e, 'c')：");
int s = Arrays.binarySearch(e, 'c');
System.out.println("字符c在数组的位置：" + s);
```

#### 比较: `equals()`

```java
char[] e = { 'a', 'f', 'b', 'c', 'e', 'A', 'C', 'B' };
char[] f = { 'a', 'f', 'b', 'c', 'e', 'A', 'C', 'B' };
/*
* 元素数量相同，并且相同位置的元素相同。 另外，如果两个数组引用都是null，则它们被认为是相等的 。
*/
// 输出true
System.out.println("Arrays.equals(e, f):" + Arrays.equals(e, f));
```

#### 填充 : `fill()`

```java
int[] g = { 1, 2, 3, 3, 3, 3, 6, 6, 6 };
// 数组中所有元素重新分配值
Arrays.fill(g, 3);
System.out.println("Arrays.fill(g, 3)：");
// 输出结果：333333333
for (int i : g) {
	System.out.print(i);
}
// 换行
System.out.println();

int[] h = { 1, 2, 3, 3, 3, 3, 6, 6, 6, };
// 数组中指定范围元素重新分配值
Arrays.fill(h, 0, 2, 9);
System.out.println("Arrays.fill(h, 0, 2, 9);：");
// 输出结果：993333666
for (int i : h) {
	System.out.print(i);
}
```

#### 转列表 `asList()`

```java
/*
* 返回由指定数组支持的固定大小的列表。
* （将返回的列表更改为“写入数组”。）该方法作为基于数组和基于集合的API之间的桥梁，与Collection.toArray()相结合 。
* 返回的列表是可序列化的，并实现RandomAccess 。
* 此方法还提供了一种方便的方式来创建一个初始化为包含几个元素的固定大小的列表如下：
*/
List<String> stooges = Arrays.asList("Larry", "Moe", "Curly");
System.out.println(stooges);
```

#### 转字符串 `toString()`

```java
/*
* 返回指定数组的内容的字符串表示形式。
*/
char[] k = { 'a', 'f', 'b', 'c', 'e', 'A', 'C', 'B' };
System.out.println(Arrays.toString(k));// [a, f, b, c, e, A, C, B]
```

#### 复制 `copyOf()`

```java
// copyOf 方法实现数组复制,h为数组，6为复制的长度
int[] h = { 1, 2, 3, 3, 3, 3, 6, 6, 6, };
int i[] = Arrays.copyOf(h, 6);
System.out.println("Arrays.copyOf(h, 6);：");
// 输出结果：123333
for (int j : i) {
	System.out.print(j);
}
// 换行
System.out.println();
// copyOfRange将指定数组的指定范围复制到新数组中
int j[] = Arrays.copyOfRange(h, 6, 11);
System.out.println("Arrays.copyOfRange(h, 6, 11)：");
// 输出结果66600(h数组只有9个元素这里是从索引6到索引11复制所以不足的就为0)
for (int j2 : j) {
	System.out.print(j2);
}
// 换行
System.out.println();
```



