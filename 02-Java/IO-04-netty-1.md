## Reactor IO

### BIO编程

在开发网络应用的时候，对于每一个请求，都会做如下的几个步骤：

- Read request
- Decode request
- Process service
- Encode reply
- Send reply

在传统的网络编程模型中，每一个Handler处理一个请求，阻塞式的完成全部的流程。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/6f0a1234-d7f1-4bfd-9cff-4163e4bdfff4" /></div>
```java
class Server implements Runnable {
    public void run() {
        try {
            ServerSocket ss = new ServerSocket(PORT);
            while (!Thread.interrupted())
                new Thread(new Handler(ss.accept())).start();
            // or, single-threaded, or a thread pool
        } catch (IOException ex) { /* ... */ }
    }
    static class Handler implements Runnable {
        final Socket socket;
        Handler(Socket s) { socket = s; }
        public void run() {
            try {
                byte[] input = new byte[MAX_INPUT];
                socket.getInputStream().read(input);
                byte[] output = process(input);
                socket.getOutputStream().write(output);
            } catch (IOException ex) { /* ... */ }
        }
        private byte[] process(byte[] cmd) { /* ... */ }
    }
}
```

IO设备与CPU速度不匹配是拖慢整个系统的原因。比如在我们的系统中



4G

13505：普通字节流时间	1K
78467：NIO时间

0.172



7788：普通字节流时间	1M
9445：NIO时间

0.825



9290：普通字节流时间	10M
10691：NIO时间

0.869



12516：普通字节流时间	50M
11350：NIO时间

1.103





370M

1243：普通字节流时间	50M
784：NIO时间

1.585



787：普通字节流时间	10M
580：NIO时间

1.357



805：普通字节流时间	1M
590：NIO时间

1.364



1343：普通字节流时间	1K
6811：NIO时间

0.197







|          | 100  | 1000 | 10000 | 100000 | 1000000 | 10000000 |
| :------: | :--: | :--: | :---: | :----: | :-----: | :------: |
| 直接内存 |  1   |  5   |  17   |  111   |  1426   |   9371   |
|  堆内存  |  0   |  2   |  21   |  170   |  1023   |   7764   |
|          |      |      |       |        |         |          |



### Basic Reactor Design

```java
class Reactor implements Runnable {
    final Selector selector;
    final ServerSocketChannel serverSocket;

    Reactor(int port) throws IOException {
        selector = Selector.open();
        serverSocket = ServerSocketChannel.open();
        serverSocket.socket().bind(new InetSocketAddress(port));
        serverSocket.configureBlocking(false);
        SelectionKey sk = serverSocket.register(selector, SelectionKey.OP_ACCEPT);
        sk.attach(new Acceptor());
    }

    public void run() {
        try {
            while (!Thread.interrupted()) {
                selector.select();
                Set selected = selector.selectedKeys();
                Iterator it = selected.iterator();
                while (it.hasNext()) {
                    dispatch((SelectionKey) (it.next()));
                }
                selected.clear();
            }
        } catch (IOException ex) { }
    }

    void dispatch(SelectionKey k) {
        Runnable r = (Runnable) (k.attachment());
        if (r != null) {
            r.run();
        }
    }

    // accept是阻塞
    class Acceptor implements Runnable { // inner
        public void run() {
            try {
                SocketChannel c = serverSocket.accept();
                if (c != null)
                    new Handler(selector, c);
            } catch (IOException ex) { /* ... */ }
        }
    }
}
```

```java
final class Handler implements Runnable {
    final SocketChannel socket;
    final SelectionKey sk;
    ByteBuffer input = ByteBuffer.allocate(MAXIN);
    ByteBuffer output = ByteBuffer.allocate(MAXOUT);
    static final int READING = 0, SENDING = 1;
    int state = READING;

    Handler(Selector sel, SocketChannel c) throws IOException {
        socket = c;
        c.configureBlocking(false);
        // Optionally try first read now
        sk = socket.register(sel, 0);
        sk.attach(this);
        sk.interestOps(SelectionKey.OP_READ);
        sel.wakeup();
    }

    boolean inputIsComplete() { /* ... */ }
    boolean outputIsComplete() { /* ... */ }
    void process() { /* ... */ }

    public void run() {
        try {
            if (state == READING)
                read();
            else if (state == SENDING)
                send();
        } catch (IOException ex) { /* ... */ }
    }

    void read() throws IOException {
        socket.read(input);
        if (inputIsComplete()) {
            process();
            state = SENDING;
            // Normally also do first write now
            sk.interestOps(SelectionKey.OP_WRITE);
        }
    }

    void send() throws IOException {
        socket.write(output);
        if (outputIsComplete())
            sk.cancel();
    }
}
```



### Worker Threads

```java
class Handler implements Runnable {
    
    final SocketChannel socket;
    final SelectionKey sk;
    ByteBuffer input = ByteBuffer.allocate(MAXIN);
    ByteBuffer output = ByteBuffer.allocate(MAXOUT);
    static final int READING = 0, SENDING = 1, int PROCESSING = 3;
    int state = READING;
    static PooledExecutor pool = new PooledExecutor(...);

    Handler(Selector sel, SocketChannel c) throws IOException {
        socket = c;
        c.configureBlocking(false);
        // Optionally try first read now
        sk = socket.register(sel, 0);
        sk.attach(this);
        sk.interestOps(SelectionKey.OP_READ);
        sel.wakeup();
    }
    
    synchronized void read() { // ...
        socket.read(input);
        if (inputIsComplete()) {
            state = PROCESSING;
            pool.execute(new Processer());
        }
    }
    
    synchronized void processAndHandOff() {
        process();
        state = SENDING; // or rebind attachment
        sk.interest(SelectionKey.OP_WRITE);
    }
    
    boolean inputIsComplete() { /* ... */ }
    boolean outputIsComplete() { /* ... */ }
    void process() { /* ... */ }
    
    class Processer implements Runnable {
        public void run() { 
            processAndHandOff(); 
        }
    }
}
```









## Netty介绍

Netty是一个IO框架，提供了对Java原生网络编程API的封装，使得用户可以快速开发高效的网络应用程序。Netty提供了对客户端和服务器的支持，我们在分析时主要分析服务器的执行流程，服务器的基本开发代码如下：

```java
public class MyServer {
    public static void main(String[] args) throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new MyServerInitializer());
            ChannelFuture channelFuture = serverBootstrap.bind(13001).sync();
            channelFuture.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```



## 服务器启动









## Future机制









## 接收请求如何调用





## WorkAround





## 粘包问题





## Netty的zero-copy







## InBound





## OutBound







## Channel







## Socket属性配置









## ByteBuf





