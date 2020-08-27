## NIO介绍

NIO指的是新的IO，目的是为了和面向流的IO对比。但是随着时间的发展，这种IO也不再新了（JDK4引入），所以人们会给它叫做非阻塞IO，但是我们需要知道，它其实是IO复用和同步非阻塞的结合。和传统BIO相比，最明显的特点就是可以使用一个服务器线程完成对多个客户端的响应，在请求次数多但每次传输的信息量较小的环境中可以显著增加服务器的并发数量。

<div align='center'><img width="60%" src="http://blogfileqiniu.isjinhao.site/86ededba-ae33-495c-b368-45b3e9669dbb"></div>
图中体现了NIO的三大组件：

- selector：用于阻塞线程并监听事件。它在不同的平台上的实现方式不同。
- 通道：BIO是面向流的，NIO是面向通道的。
- buffer：使用流处理IO的时候，我们可以一个个字节的读，也可以一次读一串字节。而使用NIO的时候，我们是将数据从通道读出至buffer或者从buffer读入到通道。



## Buffer

Java NIO中的Buffer用于和NIO通道进行交互。如你所知，数据是从通道读入缓冲区，从缓冲区写入到通道中的。缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。一个简单地例子如下：

```java
public static void main(String[] args) {
    // 指定buffer长度
    IntBuffer buffer = IntBuffer.allocate(10);
    SecureRandom secureRandom = new SecureRandom();
    // 加入随机数
    for (int i = 0; i < buffer.capacity(); i++) {
        int random = secureRandom.nextInt(20);
        buffer.put(random);
    }
    // 将buffer转换一下，读写切换
    buffer.flip();
    // 读取
    while (buffer.hasRemaining()) {
        System.out.println(buffer.get());
    }
}
```

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。为了理解Buffer的工作原理，需要熟悉它的三个属性：

- capacity：作为一个内存块，Buffer有一个固定的大小值，叫“capacity”。你只能往对应的Buffer里写capacity个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。
- position：
  - 当你写数据到Buffer中时，position表示当前的位置。初始的position值为0，当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1。
  - 当读取数据时，也是从某个特定位置读。当将Buffer进程模式切换时，position会被重置为0。当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。
- limit：
  - 写模式下，Buffer的limit表示最大能写入的位置，如果写入的位置>=limit，报异常。
  - 当切换Buffer到读模式时，limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，这个值在写模式下就是position）

**Buffer的继承图**

Buffer分为两种类型，堆内内存和堆外内存。我们等下先看堆内Buffer，堆外Buffer和零拷贝在下一篇文章聊。

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/f8a0a517-d675-4c2c-8406-fbe892301bde" /></div>
**ByteBuffer存储其他数据类型的数据**

```java
public static void main(String[] args) {
    ByteBuffer byteBuffer = ByteBuffer.allocate(64);
    byteBuffer.putInt(123);
    byteBuffer.putLong(123213123L);
    byteBuffer.putChar('我');
    byteBuffer.putShort((short) 2);

    byteBuffer.flip();
    
    // 需要一一对应的读
    System.out.println(byteBuffer.getInt());
    System.out.println(byteBuffer.getLong());
    System.out.println(byteBuffer.getChar());
    System.out.println(byteBuffer.getShort());
}
```

**只读Buffer**

一般的Buffer都是可读可写的，但是还有一种Buffer是只读Buffer。

```java
public static void main(String[] args) {

    //创建一个buffer
    ByteBuffer buffer = ByteBuffer.allocate(64);
    for(int i = 0; i < 64; i++) {
        buffer.put((byte)i);
    }
    //读取
    buffer.flip();
    //得到一个只读的Buffer
    ByteBuffer readOnlyBuffer = buffer.asReadOnlyBuffer();
    System.out.println(readOnlyBuffer.getClass());
    //读取
    while (readOnlyBuffer.hasRemaining()) {
        System.out.println(readOnlyBuffer.get());
    }
    readOnlyBuffer.put((byte)100); 	// ReadOnlyBufferException
}
```

他们的实现方式是将写操作都覆盖并直接抛出异常。

**equals()**

当满足下列条件时，表示两个Buffer相等：

1. 有相同的类型（byte、char、int等）。
2. Buffer中剩余的byte、char等的个数相等。
3. Buffer中所有剩余的byte、char等都相同。

如你所见，equals只是比较Buffer的一部分，不是每一个在它里面的元素都比较。实际上，它只比较Buffer中的剩余元素。

**compareTo()方法**

compareTo()方法比较两个Buffer的剩余元素(byte、char等)， 如果满足下列条件，则认为一个Buffer“小于”另一个Buffer：

1. 第一个不相等的元素小于另一个Buffer中对应的元素 。
2. 所有元素都相等，但第一个Buffer比另一个先耗尽（第一个Buffer的元素个数比另一个少）。

**Scatter/Gather**

- scatter：分散，从Channel中读取是指在读操作时将读取的数据写入多个buffer中。因此，Channel将从Channel中读取的数据“分散（scatter）”到多个Buffer中。

Scattering Reads是指数据从一个channel读取到多个buffer中。如下图描述：

<div align='center'><img width="30%" src="http://q0l9qvfyx.bkt.clouddn.com/fa76ce44-d13b-49b6-a3cf-35dbf91fb053"></div>
```java
public class Demo3 {
	private static Charset charset = Charset.forName("UTF-8");
	
	public static void main(String[] args) throws Exception {
		RandomAccessFile file = new RandomAccessFile("test_in", "rw");
		FileChannel channel = file.getChannel();
		ByteBuffer header = ByteBuffer.allocate(6);
		ByteBuffer body   = ByteBuffer.allocate(64);
		ByteBuffer[] bufferArray = { header, body };
		long read = channel.read(bufferArray);
		System.out.println(read);
		header.flip();
		body.flip();
		System.out.println(charset.decode(header).toString());
		System.out.println(charset.decode(body).toString());		
	}
}
```

`read()`方法按照buffer在数组中的顺序将从channel中读取的数据写入到buffer，当一个buffer被写满后，channel紧接着向另一个buffer中写。

Scattering Reads在移动下一个buffer前，必须填满当前的buffer，这也意味着它不适用于动态消息（消息大小不固定）。换句话说，如果存在消息头和消息体，消息头必须完成填充（例如 128byte），Scattering Reads才能正常工作。

- gather：聚集，写入Channel是指在写操作时将多个buffer的数据写入同一个Channel，因此，Channel 将多个Buffer中的数据“聚集（gather）”后发送到Channel。

Gathering Writes是指数据从多个buffer写入到同一个channel。如下图描述：

<div align='center'><img width="30%" src="http://blogfileqiniu.isjinhao.site/1d24ea68-6ffc-40fc-b3c9-342290aca18e"></div>
```java
public class Demo4 {
	public static void main(String[] args) throws Exception {
		
		RandomAccessFile fileOut = new RandomAccessFile("test_out", "rw");
		FileChannel channelOut = fileOut.getChannel();
		
		ByteBuffer header = ByteBuffer.allocate(6);
		ByteBuffer body   = ByteBuffer.allocate(64);

		header.put("陈".getBytes());
		body.put("钰琪是个大可爱".getBytes());
		ByteBuffer[] bufferArray = { header, body };
		
		header.flip();
		body.flip();
		
		channelOut.write(bufferArray);
		fileOut.close();
	}
}
```



### 堆内Buffer

**堆内分配的Buffer**

在我们刚才的说明中提到了写模式和读模式，但是实际上这只是被强行赋予的，即JDK中并没有说法，这么说只是为了更方便的理解，所以下面我们来解读一下Buffer的API及怎么在两种模式之间进行切换。需要指出：写模式指的是向Buffer写入，读模式是从Buffer里读出。

Buffer的分配

- 要想获得一个Buffer对象首先要进行分配。 每一个Buffer类都有一个allocate方法。下面是一个分配48字节capacity的ByteBuffer的例子。`ByteBuffer buf = ByteBuffer.allocate(48);`

- 这是分配一个可存储1024个字符的CharBuffer：`CharBuffer buf = CharBuffer.allocate(1024);`

刚获得的Buffer默认是写模式。

<div align='center'><img width="50%" src="http://blogfileqiniu.isjinhao.site/c3dcf69d-3917-496f-9db4-7a82a47aa7d3">



**向Buffer中写数据**


写数据到Buffer有两种方式：

- 从Channel写到Buffer：`int bytesRead = inChannel.read(buf); //read into buffer`
  - bytesRead指读出的数据大小。当bytesRead为-1时表示缓存区中不再有数据。
- 通过Buffer的put()方法写到Buffer里：
  - put()：相对写。向缓冲区的当前位置写入一个单元的数据，写完后把位置加1。
  - put(int index)：绝对写。向参数index指定的位置写入一个单元的数据。

```java
public class Demo1 {
	private static Charset charset = Charset.forName("UTF-8");
	
	public static void main(String[] args) throws Exception {
		// test_in是有数据的文件，用于被读入至Java程序
		RandomAccessFile fileIn = new RandomAccessFile("test_in", "rw");
		FileChannel inChannel = fileIn.getChannel();
		ByteBuffer buf = ByteBuffer.allocate(48);
		int bytesRead = inChannel.read(buf);
		while (bytesRead != -1) {
			System.out.println("Read " + bytesRead);
			buf.flip();		// 写模式切换为读模式
			System.out.println(charset.decode(buf).toString());
			buf.clear();
			bytesRead = inChannel.read(buf);
		}
		fileIn.close();
	}
}
```

`inChannel.read(buf);`之后，`buf.flip();`之前Buffer的状态：

<div align='center'><img width="50%" src="http://blogfileqiniu.isjinhao.site/de56c6a7-a774-45c0-8d0d-f7b437c1ad97"></div>
**向Channel中写数据**

```java
public class Demo2 {
    private static Charset charset = Charset.forName("UTF-8");
    public static void main(String[] args) throws Exception {
        /**
		 * test_in是有数据的文件，用于被读入至Java程序
		 * test_out中没有数据，用于Java程序的写出
		 */
        RandomAccessFile fileIn = new RandomAccessFile("test_in", "rw");
        RandomAccessFile fileOut = new RandomAccessFile("test_out", "rw");

        FileChannel inChannel = fileIn.getChannel();
        FileChannel outChannel = fileOut.getChannel();

        ByteBuffer buf = ByteBuffer.allocate(48);
        int bytesRead = -1;
        do {
            bytesRead = inChannel.read(buf);
            System.out.println("Read " + bytesRead);
            buf.flip();		// 写模式切换为读模式
            outChannel.write(buf);
            buf.clear();
        } while(bytesRead != -1);

        fileIn.close();
        fileOut.close();
    }
}
```

`buf.flip();`之后，`outChannel.write(buf);`之前Buffer的状态。

<div align='center'><img width="50%" src="http://blogfileqiniu.isjinhao.site/a302e97f-91db-4acd-9bb7-ab6a8e7c8f0e"></div>
#### **API**

**limit**

```java
public class Demo2 {
	private static Charset charset = Charset.forName("UTF-8");
	
	public static void main(String[] args) throws Exception {
		/**
		 * test_in是有数据的文件，用于被读入至Java程序
		 * test_out中没有数据，用于Java程序的写出
		 */
		RandomAccessFile fileIn = new RandomAccessFile("test_in", "rw");
		RandomAccessFile fileOut = new RandomAccessFile("test_out", "rw");
		
		FileChannel inChannel = fileIn.getChannel();
		FileChannel outChannel = fileOut.getChannel();
		
		ByteBuffer buf = ByteBuffer.allocate(48);
		int bytesRead = -1;
		do {
			bytesRead = inChannel.read(buf);
			System.out.println("写模式下： " + buf.limit());
			buf.flip();		//写模式切换为读模式
			System.out.println("读模式下： " + buf.limit());
			outChannel.write(buf);
			buf.clear();
		}while(bytesRead != -1);
		
		fileIn.close();
		fileOut.close();
	}
}
```

**rewind()方法**

`rewind()`将position设回0，所以你可以重读Buffer中的所有数据。limit保持不变，仍然表示能从Buffer中读取多少个元素（byte、char等）。

```java
public final Buffer rewind() {
    position = 0;
    mark = -1;
    return this;
}
```

**clear()与compact()方法**

一旦读完Buffer中的数据，需要让Buffer准备好再次被写入。可以通过clear()或compact()方法来完成。

如果调用的是clear()方法，position将被设回0，limit被设置成 capacity的值。换句话说，Buffer被清空了。Buffer中的数据并未清除。如果Buffer中有一些未读的数据，调用clear()方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。

```java
public final Buffer clear() {
    position = 0;
    limit = capacity;
    mark = -1;
    return this;
}
```

如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用`compact()`方法。`compact()`方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像`clear()`方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。

**mark()与reset()方法**

通过调用Buffer.mark()方法，可以标记Buffer中的一个特定位置。之后可以通过调用Buffer.reset()方法将position置于这个位置。

```java
public final Buffer mark() {
    mark = position;
    return this;
}
```

```java
public final Buffer reset() {
    int m = mark;
    if (m < 0)
        throw new InvalidMarkException();
    position = m;
    return this;
}
```

**slice**

相当于做一个buffer的快照，特点：

- slice前，设置positon和limit，那么复制一个[positon, limit)的快照。
- 在这个快照上修改值，会实际改变buffer的值。但是slicebuffer的位置不会影响buffer。

```java
public static void main(String[] args) {
    ByteBuffer byteBuffer = ByteBuffer.allocate(10);
    for (int i = 0; i < byteBuffer.capacity(); i++) {
        byteBuffer.put((byte) i);
    }
    //设置postition和limit位置
    byteBuffer.position(2);
    byteBuffer.limit(6);

    ByteBuffer sliceBufeer = byteBuffer.slice();
    byteBuffer.clear();
    for (int i = 0; i < sliceBufeer.capacity(); i++) {
        sliceBufeer.put(i, (byte) (sliceBufeer.get(i) * 2));
    }
    while (byteBuffer.hasRemaining()) {
        System.out.println(byteBuffer.get());
    }
}
```

**文件复制**

```java
public static void main(String[] args) throws IOException {
    FileInputStream in = new FileInputStream("input.txt");
    FileOutputStream out = new FileOutputStream("out.txt");
    FileChannel fileInChannel = in.getChannel();
    FileChannel fileOutChannel = out.getChannel();

    ByteBuffer byteBuffer = ByteBuffer.allocate(512);

    while (true) {
        /**
             *  这里必须使用clear()复原，如果不复原，positon和limit位置相同
             *  此时不会读取数据，因为可以认为buffer没有空间了，所以read=0,
             *  read都没有等于-1的机会，等于-1是因为输入流那边文件到头了返回-1,
             *  这种情况根本没有回去读，所以为0，然后之后flip把postition刷新回了0位置
             *  开始循环读出数据。
             */
        byteBuffer.clear();
        int read = fileInChannel.read(byteBuffer);
        System.out.println("read:" + read);
        if (-1 == read) {
            break;
        }
        byteBuffer.flip();
        fileOutChannel.write(byteBuffer);

    }
    fileInChannel.close();
    fileOutChannel.close();
}
```



## Channel

Java NIO的通道类似流，但又有些不同：

- 既可以从通道中读取数据，又可以写数据到通道。但流的读写通常是单向的。
- 通道可以异步地读写。
- 通道中的数据总是要先读到一个Buffer，或者总是要从一个Buffer中写入。

java.nio.channels.Channel接口只声明了两个方法：

- close()：关闭通道。
- isOpen()：判断通道是否打开。

通道在创建时被打开，一旦关闭通道，就不能重新打开它。

**Channel的实现**

这些是Java NIO中最重要的通道的实现：

- FileChannel：从文件中读写数据。
- DatagramChannel：能通过UDP读写网络中的数据。
- SocketChannel：能通过TCP读写网络中的数据。
- ServerSocketChannel：可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。



### 数据传输

#### **通道之间的数据传输**

在Java NIO中，如果两个通道中有一个是FileChannel，那你可以直接将数据从一个channel传输到另外一个channel。

**transferFrom()**

FileChannel的transferFrom()方法可以将数据从源通道传输到FileChannel中（这个方法在JDK文档中的解释为将字节从给定的可读取字节通道传输到此通道的文件中）。下面是一个简单的例子：

```java
public class Demo5 {
	public static void main(String[] args) throws Exception {
		
		RandomAccessFile fromFile = new RandomAccessFile("test_in", "rw");
		FileChannel fromChannel = fromFile.getChannel();

		RandomAccessFile toFile = new RandomAccessFile("test_out", "rw");
		FileChannel toChannel = toFile.getChannel();

		long position = 0;
		long count = fromChannel.size();

		toChannel.transferFrom(fromChannel, position, count);
	
		fromFile.close();
		toFile.close();
	}
}
```

**transferTo()**

将数据从FileChannel传输到其他的channe中。下面是一个简单的例子：

```java
public class Demo6 {
	public static void main(String[] args) throws Exception {
		
		RandomAccessFile fromFile = new RandomAccessFile("test_in", "rw");
		FileChannel      fromChannel = fromFile.getChannel();

		RandomAccessFile toFile = new RandomAccessFile("test_out", "rw");
		FileChannel      toChannel = toFile.getChannel();

		long position = 0;
		long count = fromChannel.size();
		fromChannel.transferTo(position, count, toChannel);
		fromFile.close();
		toFile.close();
	}
}
```

#### 通道和Buffer之间的数据传输

**Buffer主动**

```java
public static void main(String[] args) throws Exception{
    String str = "hello world";
    //创建一个输出流->channel
    FileOutputStream fileOutputStream = new FileOutputStream("d:\\file01.txt");

    //通过 fileOutputStream 获取 对应的 FileChannel
    //这个 fileChannel 真实 类型是  FileChannelImpl
    FileChannel fileChannel = fileOutputStream.getChannel();
    System.out.println(fileChannel.getClass());

    //创建一个缓冲区 ByteBuffer
    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

    byteBuffer.put(str.getBytes());		//将 str 放入 byteBuffer

    //对byteBuffer 进行flip
    byteBuffer.flip();

    //将byteBuffer 数据写入到 fileChannel
    fileChannel.write(byteBuffer);
    fileOutputStream.close();
}
```

**Channel主动**

```java
public static void main(String[] args) throws Exception {
    //创建文件的输入流
    File file = new File("d:\\file01.txt");
    FileInputStream fileInputStream = new FileInputStream(file);

    //通过fileInputStream 获取对应的FileChannel -> 实际类型  FileChannelImpl
    FileChannel fileChannel = fileInputStream.getChannel();

    //创建缓冲区
    ByteBuffer byteBuffer = ByteBuffer.allocate((int) file.length());

    //将通道的数据读入到Buffer
    fileChannel.read(byteBuffer);

    //将byteBuffer 的 字节数据 转成String
    System.out.println(new String(byteBuffer.array()));
    fileInputStream.close();
}
```



## Selector

Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。

**Selector的创建**

通过调用`Selector.open()`方法创建一个Selector，如下：

```java
Selector selector = Selector.open();
```

**向Selector注册通道**

为了将Channel和Selector配合使用，必须将Channel注册到Selector上。通过`SelectableChannel.register()`方法来实现，如下：

```java
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, Selectionkey.OP_READ);
```

与Selector一起使用时，Channel必须处于非阻塞模式下。这意味着不能将`FileChannel`与Selector一起使用，因为`FileChannel`不能切换到非阻塞模式。而套接字通道都可以。

注意`register()`方法的第二个参数。这是一个“interest集合”，意思是在通过Selector监听Channel时对什么事件感兴趣。可以监听四种不同类型的事件：

- `SelectionKey.OP_ACCEPT`：接收连接就绪事件，表示服务器监听到了客户连接，服务器可以接收这个连接了。常量值为16
- `SelectionKey.OP_CONNECT`：连接就绪事件，表示客户与服务器的连接已经建立成功。常量值为8。
- `SelectionKey.OP_READ`：读就绪事件，表示通道中已经有了可读数据，可以执行读操作了。常量值为1。
- `SelectionKey.OP_WRITE`：写就绪事件，表示已经可以向通道写数据了。常量值为4。

以上常量分别占居不同的二进制位，因此可以通过二进制的或运算“|”，来将它们进行任意组合。

总结来说：通道触发了一个事件意思是该事件已经就绪。所以，某个Channel成功连接到另一个服务器称为“连接就绪”。一个`ServerSocketChannel`准备好接收新进入的连接称为“接收就绪”。一个有数据可读的通道可以说是“读就绪”。等待写数据的通道可以说是“写就绪”。

**SelectionKey**

在上一小节中，当向Selector注册Channel时，`register()`方法会返回一个`SelectionKey`对象。这个对象包含了一些你感兴趣的属性：

- interest集合
- ready集合
- Channel
- Selector
- 附加的对象（可选）

**interest集合**

就像向Selector注册通道一节中所描述的，interest集合是你所选择的感兴趣的事件集合。可以通过`SelectionKey`读写interest集合，像这样：

```java
int interestSet = selectionKey.interestOps();
boolean isInterestedInAccept  = 
	(interestSet & SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT；
//  其他的也类似...
```

**ready集合**

ready 集合是通道已经准备就绪的操作的集合。在一次选择（Selection）之后，你会首先访问这个ready set。Selection将在下一小节进行解释。可以这样访问ready集合：`intreadySet = selectionKey.readyOps();`

可以用像检测interest集合那样的方法，来检测channel中什么事件或操作已经就绪。但是，也可以使用以下四个方法，它们都会返回一个布尔类型：

```java
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
```

**Channel + Selector**

从`SelectionKey`访问Channel和Selector很简单。如下：

```java
Channel channel = selectionKey.channel();
Selector selector = selectionKey.selector();
```

**附加的对象**

可以将一个对象或者更多信息附着到SelectionKey上，这样就能方便的识别某个给定的通道。例如，可以附加 与通道一起使用的Buffer，或是包含聚集数据的某个对象。使用方法如下：

```java
selectionKey.attach(theObject);
Object attachedObj = selectionKey.attachment();
```

还可以在用register()方法向Selector注册Channel的时候附加对象。如：

```java
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
```

**通过Selector选择通道**

一旦向Selector注册了一或多个通道，就可以调用几个重载的`select()`方法。这些方法返回你所感兴趣的事件（如连接、接受、读或写）已经准备就绪的那些通道。换句话说，如果你对“读就绪”的通道感兴趣，`select()`方法会返回读事件已经就绪的那些通道。

下面是`select()`方法：

- `int select()`：阻塞到至少有一个通道在你注册的事件上就绪了。
- `int select(long timeout)`：和`select()`一样，除了最长会阻塞timeout毫秒。
- `int selectNow()`：不会阻塞，不管什么通道就绪都立刻返回（此方法执行非阻塞的选择操作。如果自从前一次选择操作后，没有通道变成可选择的，则此方法直接返回零。）。

`select()`方法返回的int值表示有多少通道已经就绪。亦即，自上次调用`select()`方法后有多少通道变成就绪状态。如果调用select()方法，因为有一个通道变成就绪状态，返回了1，若再次调用`select()`方法，如果另一个通道就绪了，它会再次返回1。如果对第一个就绪的channel没有做任何操作，现在就有两个就绪的通道，但在每次`select()`方法调用之间，只有一个通道就绪了。

**selectedKeys()**

一旦调用了`select()`方法，并且返回值表明有一个或更多个通道就绪了，然后可以通过调用selector的`selectedKeys()`方法，访问“已选择键集（selected key set）”中的就绪通道。如下所示：

```java
Set selectedKeys = selector.selectedKeys();
```

当像Selector注册Channel时，`Channel.register()`方法会返回一个`SelectionKey` 对象。这个对象代表了注册到该Selector的通道。可以通过`SelectionKey`的`selectedKeySet()`方法访问这些对象。

可以遍历这个已选择的键集合来访问就绪的通道。如下：

```java
Set selectedKeys = selector.selectedKeys();
Iterator keyIterator = selectedKeys.iterator();
while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
    } else if (key.isReadable()) {
        // a channel is ready for reading
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }
    keyIterator.remove();
}
```

这个循环遍历已选择键集中的每个键，并检测各个键所对应的通道的就绪事件。注意每次迭代末尾的`keyIterator.remove()`调用。Selector不会自己从已选择键集中移除`SelectionKey`实例。必须在处理完通道时自己移除。下次该通道变成就绪时，Selector会再次将其放入已选择键集中。

`SelectionKey.channel()`方法返回的通道需要转型成你要处理的类型，如`ServerSocketChannel`或`SocketChannel`等。



## SocketChannel

**打开 SocketChannel**

下面是`SocketChannel`的打开方式：

```java
SocketChannel socketChannel = SocketChannel.open();
socketChannel.connect(new InetSocketAddress("127.0.0.1", 20000));
```

**关闭 SocketChannel**

```java
socketChannel.close();
```

**ServerSocketChannel**

Java NIO中的 `ServerSocketChannel` 是一个可以监听新进来的TCP连接的通道，就像BIO中的`ServerSocket`一样。`ServerSocketChannel`类在`java.nio.channels` 包中。

**打开 ServerSocketChannel**

通过调用 `ServerSocketChannel.open()` 方法来打开`ServerSocketChannel`，如：

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
```

**关闭 ServerSocketChannel**

通过调用 `ServerSocketChannel.close()` 方法来关闭 `ServerSocketChannel`，如：

```java
serverSocketChannel.close();
```

**监听新进来的连接**

通过 `ServerSocketChannel.accept()` 方法监听新进来的连接。当 `accept()` 方法返回的时候，它返回一个包含新进来的连接的 `SocketChannel`。因此，`accept()`方法会一直阻塞到有新连接到达。

通常不会仅仅只监听一个连接,在while循环中调用 `accept()` 方法，如下面的例子：

```java
while(true){
    SocketChannel socketChannel = serverSocketChannel.accept();
    // do something with socketChannel...
}
```

当然，也可以在while循环中使用除了true以外的其它退出准则。

**非阻塞模式**

`ServerSocketChannel` 可以设置成非阻塞模式。在非阻塞模式下，`accept()` 方法会立刻返回，如果还没有新进来的连接,返回的将是null。 因此，需要检查返回的 `SocketChannel` 是否是null，如：

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));
serverSocketChannel.configureBlocking(false);

while(true){
    SocketChannel socketChannel = serverSocketChannel.accept();

    if(socketChannel != null){
        // do something with socketChannel...
    }
}
```

**群聊系统的实现**

```java
public class GroupChatServer {
    // 定义属性
    private Selector selector;
    private ServerSocketChannel listenChannel;
    private static final int PORT = 6667;

    // 构造器，初始化工作
    public GroupChatServer() {
        try {
            selector = Selector.open();
            listenChannel = ServerSocketChannel.open();
            listenChannel.socket().bind(new InetSocketAddress(PORT));
            listenChannel.configureBlocking(false);  // 设置非阻塞模式
            // 将该 listenChannel 注册到 selector
            listenChannel.register(selector, SelectionKey.OP_ACCEPT);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 监听
    public void listen() {
        System.out.println("监听线程: " + Thread.currentThread().getName());
        try {
            while (true) {
                int count = selector.select();
                if (count > 0) {
                    Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                    while (iterator.hasNext()) {
                        SelectionKey key = iterator.next();
                        if (key.isAcceptable()) {
                            SocketChannel sc = listenChannel.accept();
                            sc.configureBlocking(false);
                            // 将该 sc 注册到seletor
                            sc.register(selector, SelectionKey.OP_READ);

                            // 提示
                            System.out.println(sc.getRemoteAddress() + " 上线 ");
                        }
                        if (key.isReadable()) { // 通道发送read事件，即通道是可读的状态
                            readData(key);
                        }
                        iterator.remove();
                    }
                } else {
                    System.out.println("等待....");
                }
            }

        } catch (Exception e) {
            e.printStackTrace();

        } finally {
            //发生异常处理....
        }
    }

    //读取客户端消息
    private void readData(SelectionKey key) {
        SocketChannel channel = null;
        try {
            channel = (SocketChannel) key.channel();
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            int count = channel.read(buffer);
            if (count > 0) {
                String msg = new String(buffer.array());
                System.out.println("form 客户端: " + msg);
                //向其它的客户端转发消息, 专门写一个方法来处理
                sendInfoToOtherClients(msg, channel);
            }
        } catch (IOException e) {
            try {
                System.out.println(channel.getRemoteAddress() + " 离线了..");
                key.cancel();
                channel.close();
            } catch (IOException e2) {
                e2.printStackTrace();
            }
        }
    }

    // 转发消息给其它客户(通道)
    private void sendInfoToOtherClients(String msg, SocketChannel self) throws IOException {
        System.out.println("服务器转发数据给客户端线程: "
                           + Thread.currentThread().getName());
        for (SelectionKey key : selector.keys()) {
            // 通过 key 取出对应的 SocketChannel
            Channel targetChannel = key.channel();
            if (targetChannel instanceof SocketChannel && targetChannel != self) {
                SocketChannel dest = (SocketChannel) targetChannel;
                ByteBuffer buffer = ByteBuffer.wrap(msg.getBytes());
                dest.write(buffer);
            }
        }
    }

    public static void main(String[] args) {
        //创建服务器对象
        GroupChatServer groupChatServer = new GroupChatServer();
        groupChatServer.listen();
    }
}
```

**BIO连接NIO服务器**

```java
public class EchoSeverZJH {
	private Selector selector;
	private ServerSocketChannel serverSocketChannel;
	private int port = 8000;
	private Charset charset = Charset.forName("utf-8");
	private List<String> list = new ArrayList<>();
	private int num;
	
	public EchoSeverZJH(){
		try {
			selector = Selector.open();
			serverSocketChannel = ServerSocketChannel.open();
			serverSocketChannel.configureBlocking(false);
			serverSocketChannel.socket().setReuseAddress(true);
			serverSocketChannel.socket().bind(new InetSocketAddress(port));
		} catch (IOException e1) {
			e1.printStackTrace();
		}
		System.out.println("服务器开启成功... : " + new Date());
		list.add("一共有18018条航班数据");
		try (Scanner cin = new Scanner(new BufferedInputStream(new FileInputStream("copy")))) {
			String line = null;
			while(cin.hasNext()) {
				line = cin.nextLine();
				list.add(line);
			}
			list.add("no data");
		} catch (IOException e) {
			e.printStackTrace();
		}
		num = list.size();
		System.out.println("数据已准备好... : " + new Date());
	}

	public void service(){
		try {
			serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
		} catch (ClosedChannelException e) {
			e.printStackTrace();
		}
		while (true) {
			//没有连接就会阻塞
			int n = 0;
			try {
				n = selector.select();
			} catch (IOException e) {
				e.printStackTrace();
			}
			if (n == 0)
				continue;
			Set<SelectionKey> readkeys = selector.selectedKeys();
			Iterator<SelectionKey> iterator = readkeys.iterator();
			while (iterator.hasNext()) {
				SelectionKey key = iterator.next();
				if(!key.isValid()) {
					iterator.remove();
					continue;
				}
				if (key.isAcceptable()) {
					accept(key);
				}
				if (key.isWritable()) {
					send(key);
				}
				iterator.remove();
			}
		}
	}

	// 处理接收连接就绪事件
	private void accept(SelectionKey key){
		// 返回一个对象
		SocketChannel socketChannel;
		try {
			socketChannel = serverSocketChannel.accept();
			System.out.println("收到了客户端连接，来自 ： " + socketChannel.getRemoteAddress());
			socketChannel.configureBlocking(false);
			//注册到selector
			socketChannel.register(selector, SelectionKey.OP_WRITE, 0);
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	// 处理写就绪事件
	private void send(SelectionKey key){
		SocketChannel socketChannel = (SocketChannel) key.channel();
		int index = (int)key.attachment();
		key.attach((index + 1) % num);
		String line = list.get(index);
		ByteBuffer outputBuffer = charset.encode(line + "\r\n");
		while (outputBuffer.hasRemaining()) {
			try {
				socketChannel.write(outputBuffer);
			} catch (IOException e) {
				key.cancel();
				try {
					System.out.println(socketChannel.getRemoteAddress() + "   断开连接----");
					socketChannel.socket().close();
					socketChannel.close();
					outputBuffer.clear();
					break;
				} catch (IOException e1) {
					e1.printStackTrace();
				}
				e.printStackTrace();
			}
		}
	}

	public static void main(String[] args){
		new EchoSeverZJH().service();
	}
}
```

```java
public class ReadFromSocket{
	public static void main(String[] args) {
		Socket socket = null;
		Scanner sc = null;
		try {
			socket = new Socket(InetAddress.getByName("127.0.0.1"), 8000);
			String line = null;
			sc = new Scanner(socket.getInputStream());
			line = sc.nextLine();
			System.out.println(line);
			while (sc.hasNext()) {
				Thread.sleep(1000);
				line = sc.nextLine();
				System.out.println(line);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}finally {
			try {
				socket.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
			sc.close();
		}
	}
}
```



## Selector细讲

> A multiplexor of SelectableChannel objects.
>
> A selector may be created by invoking the open method of this class, which will use the system's default selector provider to create a new selector. A selector may also be created by invoking the openSelector method of a custom selector provider. A selector remains open until it is closed via its close method. 
>
> A selectable channel's registration with a selector is represented by a SelectionKey object. A selector maintains three sets of selection keys:
>
> 1. The *key set* contains the keys representing the current channel registrations of this selector. This set is returned by the keys method.
> 2. The *selected-key set* is the set of keys such that each key's channel was detected to be ready for at least one of the operations identified in the key's interest set during a prior selection operation. This set is returned by the selectedKeys method. The selected-key set is always a subset of the key set.
> 3. The *cancelled-key set* is the set of keys that have been cancelled but whose channels have not yet been deregistered. This set is not directly accessible. The cancelled-key set is always a subset of the key set.

每个Selector维护着三个selection key set：

- key set：全部的注册的key
- selected-key set：产生了事件的key
- cancelled-key set：被撤销但是Channel还处于注册状态的key。被撤销的key不是一个合法的key，即`SelectionKey.isValid()`返回false。

> All three sets are empty in a newly-created selector.
>
> A key is added to a selector's key set as a side effect of registering a channel via the channel's register method. Cancelled keys are removed from the key set during selection operations. The key set itself is not directly modifiable.

key加入key set是通道向selector注册的时候进行的。被撤销的key会在selection过程被移除。

> A key is added to its selector's cancelled-key set when it is cancelled, whether by closing its channel or by invoking its cancel method. Cancelling a key will cause its channel to be deregistered during the next selection operation, at which time the key will removed from all of the selector's key sets. 

key被撤销的时候通道不会被立即deregistered，而是在下一次selection操作时才会被deregistered，同时key会被从所有的key sets中移除。

> Keys are added to the selected-key set by selection operations. A key may be removed directly from the selected-key set by invoking the set's remove method or by invoking the remove method of an iterator obtained from the set. Keys are never removed from the selected-key set in any other way; they are not, in particular, removed as a side effect of selection operations. Keys may not be added directly to the selected-key set.

key是在selection过程被加入selected-key set的。从selected-key set移除key可以通过集合的或者迭代器的`remove()`方法。

**Selection**

> During each selection operation, keys may be added to and removed from a selector's selected-key set and may be removed from its key and cancelled-key sets. Selection is performed by the select(), select(long), and selectNow() methods, and involves three steps:
>
> - Each key in the cancelled-key set is removed from each key set of which it is a member, and its channel is deregistered. This step leaves the cancelled-key set empty.

清空cancelled-key set，并从其他两个集合中移除对应的key，并且将对应的Channel都deregistered。

> - The underlying operating system is queried for an update as to the readiness of each remaining channel to perform any of the operations identified by its key's interest set as of the moment that the selection operation began. For a channel that is ready for at least one such operation, one of the following two actions is performed:
>   - If the channel's key is not already in the selected-key set then it is added to that set and its ready-operation set is modified to identify exactly those operations for which the channel is now reported to be ready. Any readiness information previously recorded in the ready set is discarded.
>   - Otherwise the channel's key is already in the selected-key set, so its ready-operation set is modified to identify any new operations for which the channel is reported to be ready. Any readiness information previously recorded in the ready set is preserved; in other words, the ready set returned by the underlying system is bitwise-disjoined into the key's current ready set.
>
> If all of the keys in the key set at the start of this step have empty interest sets then neither the selected-key set nor any of the keys' ready-operation sets will be updated.

selector 会根据之前Channel注册的事件来询问操作系统并更新剩下的准备就绪的通道，此时分两种清空：

- 如果channel的key不存在selected-key set中，加入并且ready操作集合会被修改。ready set记录的之前的就绪事件的信息会被丢弃。
- 如果channel的key已经存在selected-key set中，修改ready操作集合。ready set之前记录的就绪事件信息会被保留。

> - If any keys were added to the cancelled-key set while step (2) was in progress then they are processed as in step (1).
>
> Whether or not a selection operation blocks to wait for one or more channels to become ready, and if so for how long, is the only essential difference between the three selection methods.

**Concurrency**

> Selectors are themselves safe for use by multiple concurrent threads; their key sets, however, are not.
> The selection operations synchronize on the selector itself, on the key set, and on the selected-key set, in that order. They also synchronize on the cancelled-key set during steps (1) and (3) above.

selector 的方法是线程安全的，但是key sets不是。

> Changes made to the interest sets of a selector's keys while a selection operation is in progress have no effect upon that operation; they will be seen by the next selection operation.

被注册到Channel的新的感兴趣的事件在下一轮selection才会生效。

> Keys may be cancelled and channels may be closed at any time. Hence the presence of a key in one or more of a selector's key sets does not imply that the key is valid or that its channel is open. Application code should be careful to synchronize and check these conditions as necessary if there is any possibility that another thread will cancel a key or close a channel.

应用需要关注key是不是合法的，因为key的存在不能保证他的channel是的开启的。

> A thread blocked in one of the select() or select(long) methods may be interrupted by some other thread in one of three ways:
> 
>- By invoking the selector's wakeup method,
> - By invoking the selector's close method, or
>- By invoking the blocked thread's interrupt method, in which case its interrupt status will be set and the selector's wakeup method will be invoked.

打断一个正在selection过程的selector有三种方式：执行selector的wakeup方法，执行selector的close方法，执行线程的interrupt方法。

> The close method synchronizes on the selector and all three key sets in the same order as in a selection operation. 
> 
>A selector's key and selected-key sets are not, in general, safe for use by multiple concurrent threads. If such a thread might modify one of these sets directly then access should be controlled by synchronizing on the set itself. The iterators returned by these sets' iterator methods are fail-fast: If the set is modified after the iterator is created, in any way except by invoking the iterator's own remove method, then a java.util.ConcurrentModificationException will be thrown.

直接对key集合进行修改不是线程安全的，而且一旦iterator被创建，只能通过迭代器对集合进行修改。



## SelectionKey细讲

> A token representing the registration of a SelectableChannel with a Selector.
>
> A selection key is created each time a channel is registered with a selector. A key remains valid until it is cancelled by invoking its cancel method, by closing its channel, or by closing its selector. Cancelling a key does not immediately remove it from its selector; it is instead added to the selector's cancelled-key set for removal during the next selection operation. The validity of a key may be tested by invoking its isValid method. 

SelectionKey在其撤销方法没被调用且通道未关闭且selector未关闭的时候是合法的。

> A selection key contains two operation sets represented as integer values. Each bit of an operation set denotes a category of selectable operations that are supported by the key's channel.
>
> The interest set determines which operation categories will be tested for readiness the next time one of the selector's selection methods is invoked. The interest set is initialized with the value given when the key is created; it may later be changed via the interestOps(int) method.
>
> The ready set identifies the operation categories for which the key's channel has been detected to be ready by the key's selector. The ready set is initialized to zero when the key is created; it may later be updated by the selector during a selection operation, but it cannot be updated directly.

一个SelectorKey包括两个操作集合。一个是interest set，一个是ready set。

> That a selection key's ready set indicates that its channel is ready for some operation category is a hint, but not a guarantee, that an operation in such a category may be performed by a thread without causing the thread to block. A ready set is most likely to be accurate immediately after the completion of a selection operation. It is likely to be made inaccurate by external events and by I/O operations that are invoked upon the corresponding channel.

SelectionKey的就绪事件集合在selection完成后是立即精确的，但是外部数据和IO操作会使其变得不精确。

> This class defines all known operation-set bits, but precisely which bits are supported by a given channel depends upon the type of the channel. Each subclass of SelectableChannel defines an validOps() method which returns a set identifying just those operations that are supported by the channel. An attempt to set or test an operation-set bit that is not supported by a key's channel will result in an appropriate run-time exception.

操作集合位的精确定义是由SelectableChannel决定的。它的`validOps()`方法会返回操作集合位。

> It is often necessary to associate some application-specific data with a selection key, for example an object that represents the state of a higher-level protocol and handles readiness notifications in order to implement that protocol. Selection keys therefore support the attachment of a single arbitrary object to a key. An object can be attached via the attach method and then later retrieved via the attachment method.

一个SelectionKey可以带一个附带对象，使用`attachment()`完成。

> Selection keys are safe for use by multiple concurrent threads. The operations of reading and writing the interest set will, in general, be synchronized with certain operations of the selector. Exactly how this synchronization is performed is implementation-dependent: In a naive implementation, reading or writing the interest set may block indefinitely if a selection operation is already in progress; in a high-performance implementation, reading or writing the interest set may block briefly, if at all. In any case, a selection operation will always use the interest-set value that was current at the moment that the operation began.

Selectionkey是线程安全的。而且读写interest set也是线程安全的。