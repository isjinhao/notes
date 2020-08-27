## zookeeper 应用场景

### 配置中心案例

工作中有这样的一个场景: 数据库用户名和密码信息放在一个配置文件中，应用读取该配置文件，配置文件信息放入缓存。

若数据库的用户名和密码改变时候，还需要重新加载缓存，比较麻烦，通过 ZooKeeper 可以轻松完成，当数据库发生变化时自动完成缓存同步。

设计思路：

1. 连接zookeeper服务器
2. zookeeper中的配置信息，注册watcher监听器，存入本地变量
3. zookeeper中的配置信息发生变化时，通过watcher的回调方法捕获数据变化事件
4. 重新获取配置信息  

```java
public class MyConfigCenter implements Watcher {

    //  zk的连接串
    String IP = "192.168.60.130:2181";
    //  计数器对象
    CountDownLatch countDownLatch = new CountDownLatch(1);
    // 连接对象
    static ZooKeeper zooKeeper;

    // 用于本地化存储配置信息
    private String url;
    private String username;
    private String password;

    @Override
    public void process(WatchedEvent event) {
        try {
            // 捕获事件状态
            if (event.getType() == Event.EventType.None) {
                if (event.getState() == Event.KeeperState.SyncConnected) {
                    System.out.println("连接成功");
                    countDownLatch.countDown();
                } else if (event.getState() == Event.KeeperState.Disconnected) {
                    System.out.println("连接断开!");
                } else if (event.getState() == Event.KeeperState.Expired) {
                    System.out.println("连接超时!");
                    // 超时后服务器端已经将连接释放，需要重新连接服务器端
                    zooKeeper = new ZooKeeper("192.168.60.130:2181", 6000,
                            new ZKConnectionWatcher());
                } else if (event.getState() == Event.KeeperState.AuthFailed) {
                    System.out.println("验证失败!");
                }
                // 当配置信息发生变化时
            } else if (event.getType() == EventType.NodeDataChanged) {
                initValue();
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    // 构造方法
    public MyConfigCenter() {
        initValue();
    }


    // 连接zookeeper服务器，读取配置信息
    public void initValue() {
        try {
            // 创建连接对象
            zooKeeper = new ZooKeeper(IP, 5000, this);
            // 阻塞线程，等待连接的创建成功
            countDownLatch.await();
            // 读取配置信息
            this.url = new String(zooKeeper.getData("/config/url", true, null));
            this.username = new String(zooKeeper.getData("/config/username", true, null));
            this.password = new String(zooKeeper.getData("/config/password", true, null));
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    public static void main(String[] args) {
        try {
            MyConfigCenter myConfigCenter = new MyConfigCenter();
            for (int i = 1; i <= 20; i++) {
                Thread.sleep(5000);
                System.out.println("url:"+myConfigCenter.getUrl());
                System.out.println("username:"+myConfigCenter.getUsername());
                System.out.println("password:"+myConfigCenter.getPassword());
                System.out.println("########################################");
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```



### 生成分布式唯一ID

在过去的单库单表型系统中，通常可以使用数据库字段自带的 auto_increment 属性来自动为每条记录生成一个唯一的ID。但是分库分表后，就无法在依靠数据库的 auto_increment 属性来唯一标识一条记录了。此时我们就可以用zookeeper在分布式环境下生成全局唯一ID。

设计思路：

1. 连接zookeeper服务器
2. 指定路径生成临时有序节点
3. 取序列号及为分布式环境下的唯一ID  

```java
public class GloballyUniqueId implements Watcher {
    //  zk的连接串
    String IP = "59.110.143.226:2181";
    //  计数器对象
    CountDownLatch countDownLatch = new CountDownLatch(1);
    //  用户生成序号的节点
    String defaultPath = "/uniqueId";
    //  连接对象
    ZooKeeper zooKeeper;

    @Override
    public void process(WatchedEvent event) {
        try {
            // 捕获事件状态
            if (event.getType() == Watcher.Event.EventType.None) {
                if (event.getState() == Watcher.Event.KeeperState.SyncConnected) {
                    System.out.println("连接成功");
                    countDownLatch.countDown();
                } else if (event.getState() == Watcher.Event.KeeperState.Disconnected) {
                    System.out.println("连接断开!");
                } else if (event.getState() == Watcher.Event.KeeperState.Expired) {
                    System.out.println("连接超时!");
                    // 超时后服务器端已经将连接释放，需要重新连接服务器端
                    zooKeeper = new ZooKeeper(IP, 6000,
                            new ZKConnectionWatcher());
                } else if (event.getState() == Watcher.Event.KeeperState.AuthFailed) {
                    System.out.println("验证失败!");
                }
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    // 构造方法
    public GloballyUniqueId() {
        try {
            //打开连接
            zooKeeper = new ZooKeeper(IP, 5000, this);
            // 阻塞线程，等待连接的创建成功
            countDownLatch.await();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    // 生成id的方法
    public String getUniqueId() {
        String path = "";
        try {
            //创建临时有序节点
            path = zooKeeper.create(defaultPath, new byte[0], Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        // /uniqueId0000000001
        return path.substring(9);
    }

    public static void main(String[] args) {
        GloballyUniqueId globallyUniqueId = new GloballyUniqueId();
        for (int i = 1; i <= 5; i++) {
            String id = globallyUniqueId.getUniqueId();
            System.out.println(id);
        }
    }
}
```



### 分布式锁

设计思路：

1. 每个客户端往 /Locks 下创建临时有序节点 /Locks/Lock000000001
2. 客户端取得 /Locks 下子节点，并进行排序，判断排在最前面的是否为自己，如果自己的锁节点在第一位，代表获取锁成功
3. 如果自己的锁节点不在第一位，则监听自己前一位的锁节点。然后自己进入等待。
4. 当前一位锁节点释放锁时，会通知当前结点，此时重新执行第2步逻辑，判断自己是否获得了锁。

```java
public class MyLock {
    //  zk的连接串
    String IP = "192.168.60.130:2181";
    //  计数器对象
    CountDownLatch countDownLatch = new CountDownLatch(1);
    // ZooKeeper配置信息
    ZooKeeper zooKeeper;
    private static final String LOCK_ROOT_PATH = "/Locks";
    private static final String LOCK_NODE_NAME = "Lock_";
    private String lockPath;

    // 打开zookeeper连接
    public MyLock() {
        try {
            zooKeeper = new ZooKeeper(IP, 5000, new Watcher() {
                @Override
                public void process(WatchedEvent event) {
                    if (event.getType() == Event.EventType.None) {
                        if (event.getState() == Event.KeeperState.SyncConnected) {
                            System.out.println("连接成功!");
                            countDownLatch.countDown();
                        }
                    }
                }
            });
            countDownLatch.await();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    // 获取锁
    public void acquireLock() throws Exception {
        // 创建锁节点
        createLock();
        // 尝试获取锁
        attemptLock();
    }

    // 创建锁节点
    private void createLock() throws Exception {
        // 判断Locks是否存在，不存在创建
        Stat stat = zooKeeper.exists(LOCK_ROOT_PATH, false);
        if (stat == null) {
            zooKeeper.create(LOCK_ROOT_PATH, new byte[0], 
                             ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        }
        // 创建临时有序节点
        lockPath = zooKeeper.create(LOCK_ROOT_PATH + "/" + LOCK_NODE_NAME, new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
        System.out.println("节点创建成功:" + lockPath);
    }

    // 监视器对象，监视上一个节点是否被删除
    Watcher watcher = new Watcher() {
        @Override
        public void process(WatchedEvent event) {
            if (event.getType() == Event.EventType.NodeDeleted) {
                synchronized (this) {
                    notifyAll();
                }
            }
        }
    };

    // 尝试获取锁
    private void attemptLock() throws Exception {
        // 获取Locks节点下的所有子节点
        List<String> list = zooKeeper.getChildren(LOCK_ROOT_PATH, false);
        // 对子节点进行排序
        Collections.sort(list);
        // /Locks/Lock_000000001
        int index = list.indexOf(lockPath.substring(LOCK_ROOT_PATH.length() + 1));
        if (index == 0) {
            System.out.println("获取锁成功!");
            return;
        } else {
            // 上一个节点的路径
            String path = list.get(index - 1);
            Stat stat = zooKeeper.exists(LOCK_ROOT_PATH + "/" + path, watcher);
            if (stat != null) {
                synchronized (watcher) {
                    watcher.wait();
                }
            }
            attemptLock();
        }
    }

    // 释放锁
    public void releaseLock() throws Exception {
        // 删除临时有序节点
        zooKeeper.delete(this.lockPath, -1);
        zooKeeper.close();
        System.out.println("锁已经释放:" + this.lockPath);
    }

    public static void main(String[] args) {
        MyLock myLock = null;
        try {
            myLock = new MyLock();
            myLock.createLock();
            ...
        } catch (Exception ex) {
            ex.printStackTrace();
        } finally {
            try {
                myLock.releaseLock();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```



## Curator

Curator 是 Netflix 公司开源的一个 zookeeper 客户端， 后捐献给apache，curator 框架在 zookeeper 原生API接口上进行了包装， 解决了很多 zooKeeper 客户端非常底层的细节开发。 提供 zooKeeper 各种应用场景（比如： 分布式锁服务、 集群领导选举、共享计数器、 缓存机制、 分布式队列等）的抽象封装， 实现了Fluent风格的API接口， 是最好用， 最流行的 zookeeper 的客户端。  

原生zookeeperAPI的不足：

- 连接对象异步创建， 需要开发人员自行编码等待
- 连接没有自动重连超时机制
- Watcher一次注册生效一次
- 不支持递归创建树形节点  

Curator特点：

- 解决session会话超时重连
- Watcher反复注册
- 简化开发api
- 遵循Fluent风格的API
- 提供了分布式锁服务、 共享计数器、 缓存机制等机制  

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.7</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>2.6.0</version>
    <type>jar</type>
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.10</version>
    <type>jar</type>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>2.6.0</version>
    <type>jar</type>
</dependency>
```



### 连接到 ZooKeeper

```java
public class CuratorConnection {
    public static void main(String[] args) {
//         3秒后重连一次，只重连1次
//         RetryPolicy retryPolicy = new RetryOneTime(3000);

//         每3秒重连一次，重连3次
//         RetryPolicy retryPolicy = new RetryNTimes(3, 3000);

//         每3秒重连一次，总等待时间超过10秒后停止重连
//         RetryPolicy retryPolicy = new RetryUntilElapsed(10000, 3000);

        // baseSleepTimeMs * Math.max(1, random.nextInt(1 << (retryCount + 1)))
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);

        // 创建连接对象
        // ip1:port1,ip2:port2
        CuratorFramework client = CuratorFrameworkFactory.builder()
                // IP地址端口号
                .connectString("xxx:2181")
                // 会话超时时间
                .sessionTimeoutMs(5000)
                // 重连机制
                .retryPolicy(retryPolicy)
                // 命名空间：在根节点下创建一个create结点。后续的操作都对此节点下进行
                .namespace("create")
                // 构建连接对象
                .build();
        // 打开连接
        client.start();
        // 关闭连接
        client.close();
    }
}
```



### 新增节点

```java
public class CuratorCreate {

    String IP = "xxxx:2181";
    CuratorFramework client;

    @Before
    public void before() {
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        client = CuratorFrameworkFactory.builder()
                .connectString(IP)
                .sessionTimeoutMs(5000)
                .retryPolicy(retryPolicy)
                .namespace("create")
                .build();
        client.start();
        // 即使这里输出连接成功，也不一定是真正的连接成功。是不是真正连接成功需要看操作能否正常执行
        System.out.println("连接成功");
    }

    @After
    public void after() {
        client.close();
    }

    @Test
    public void create1() throws Exception {
        // 新增节点
        client.create()
                // 节点的类型
                .withMode(CreateMode.PERSISTENT)
                // 节点的权限列表 world:anyone:cdrwa
                .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
                // arg1:节点的路径
                // arg2:节点的数据
                .forPath("/node2", "node2".getBytes());

        System.out.println("结束");
        Thread.sleep(100000);
    }


    @Test
    public void create2() throws Exception {
        // 自定义权限列表
        // 权限列表
        List<ACL> list = new ArrayList<ACL>();
        // 授权模式和授权对象
        Id id = new Id("ip", "192.168.60.130");
        list.add(new ACL(ZooDefs.Perms.ALL, id));
        client.create().withMode(CreateMode.PERSISTENT).withACL(list).
            		forPath("/node2", "node2".getBytes());
        System.out.println("结束");
    }

    @Test
    public void create3() throws Exception {
        // 递归创建节点树
        client.create()
                // 递归节点的创建
                .creatingParentsIfNeeded()
                .withMode(CreateMode.PERSISTENT)
                .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
                .forPath("/node3/node31", "node31".getBytes());
        System.out.println("结束");
    }


    @Test
    public void create4() throws Exception {
        // 异步方式创建节点
        client.create()
                .creatingParentsIfNeeded()
                .withMode(CreateMode.PERSISTENT)
                .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
                // 异步回调接口
                .inBackground(new BackgroundCallback() {
                     public void processResult(CuratorFramework curatorFramework, 
                                     		CuratorEvent curatorEvent) throws Exception {
                         // 节点的路径
                         System.out.println(curatorEvent.getPath());
                         // 时间类型
                         System.out.println(curatorEvent.getType());
                     }
        })
                .forPath("/node4","node4".getBytes());
        Thread.sleep(5000);
        System.out.println("结束");
    }
}
```



### 更新节点

```java
public class CuratorSet {

    String IP = "192.168.60.130:2181,192.168.60.130:2182,192.168.60.130:2183";
    CuratorFramework client;

    @Before
    public void before() {
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        client = CuratorFrameworkFactory.builder()
                .connectString(IP)
                .sessionTimeoutMs(5000)
                .retryPolicy(retryPolicy)
                .namespace("set").build();
        client.start();
    }

    @After
    public void after() {
        client.close();
    }

    @Test
    public void set1() throws Exception {
        // 更新节点
        client.setData()
                // arg1:节点的路径
                // arg2:节点的数据
                .forPath("/node1", "node11".getBytes());
        System.out.println("结束");
    }

    @Test
    public void set2() throws Exception {
        client.setData()
                // 指定版本号
                .withVersion(2)
                .forPath("/node1", "node1111".getBytes());
        System.out.println("结束");
    }

    @Test
    public void set3() throws Exception {
        // 异步方式修改节点数据
        client.setData()
                .withVersion(-1).inBackground(new BackgroundCallback() {
            public void processResult(CuratorFramework curatorFramework, CuratorEvent curatorEvent) throws Exception {
                // 节点的路径
                System.out.println(curatorEvent.getPath());
                // 事件的类型
                System.out.println(curatorEvent.getType());
            }
        }).forPath("/node1", "node1".getBytes());
        Thread.sleep(5000);
        System.out.println("结束");
    }
}
```



### 删除节点

```java
public class CuratorDelete {

    String IP = "192.168.60.130:2181,192.168.60.130:2182,192.168.60.130:2183";
    CuratorFramework client;

    @Before
    public void before() {
		...
	}

    @After
    public void after() {
        client.close();
    }

    @Test
    public void delete1() throws Exception {
       // 删除节点
        client.delete()
                // 节点的路径
                .forPath("/node1");
        System.out.println("结束");
    }

    @Test
    public void delete2() throws Exception {
        client.delete()
                .withVersion(0)	// 版本号
                .forPath("/node1");
        System.out.println("结束");
    }

    @Test
    public void delete3() throws Exception {
        //删除包含子节点的结点
        client.delete()
                .deletingChildrenIfNeeded()
                .withVersion(-1)
                .forPath("/node1");
        System.out.println("结束");
    }

    @Test
    public void delete4() throws Exception {
        // 异步方式删除节点
        client.delete()
                .deletingChildrenIfNeeded()
                .withVersion(-1)
                .inBackground(new BackgroundCallback() {
                    public void processResult(CuratorFramework curatorFramework, 
                                           CuratorEvent curatorEvent) throws Exception {
                        // 节点路径
                        System.out.println(curatorEvent.getPath());
                        // 事件类型
                        System.out.println(curatorEvent.getType());
                    }
                })
                .forPath("/node1");
        Thread.sleep(5000);
        System.out.println("结束");
    }
}
```



### 查看节点

```java
public class CuratorGet {

    String IP = "192.168.60.130:2181,192.168.60.130:2182,192.168.60.130:2183";
    CuratorFramework client;

    @Before
    public void before() {
		...
    }

    @After
    public void after() {
        client.close();
    }

    @Test
    public void get1() throws Exception {
        // 读取节点数据
        byte [] bys = client.getData()
                .forPath("/node1");      // 节点的路径
        System.out.println(new String(bys));
    }

    @Test
    public void get2() throws Exception {
        // 读取数据时读取节点的属性
        Stat stat = new Stat();
        byte [] bys = client.getData()
                // 读取属性
                .storingStatIn(stat)
                .forPath("/node1");
        System.out.println(new String(bys));
        System.out.println(stat.getVersion());
    }

    @Test
    public void get3() throws Exception {
        // 异步方式读取节点的数据
        client.getData()
                 .inBackground(new BackgroundCallback() {
                     public void processResult(CuratorFramework curatorFramework, 
                                           CuratorEvent curatorEvent) throws Exception {
                         // 节点的路径
                         System.out.println(curatorEvent.getPath());
                         // 事件类型
                         System.out.println(curatorEvent.getType());
                         // 数据
                         System.out.println(new String(curatorEvent.getData()));
                     }
                 })
                .forPath("/node1");
        Thread.sleep(5000);
        System.out.println("结束");
    }
}
```



### 查看子节点

```java
public class CuratorGetChild {

    String IP = ":2183";
    CuratorFramework client;

    @Before
    public void before() {
		...
    }

    @After
    public void after() {
        client.close();
    }

    @Test
    public void getChild1() throws Exception {
        // 读取子节点数据
        List<String> list = client.getChildren()
                // 节点路径
                .forPath("/get");
        for (String str : list) {
            System.out.println(str);
        }
    }

    @Test
    public void getChild2() throws Exception {
        // 异步方式读取子节点数据
        client.getChildren()
                .inBackground(new BackgroundCallback() {
                    public void processResult(CuratorFramework curatorFramework, CuratorEvent curatorEvent) throws Exception {
                        // 节点路径
                        System.out.println(curatorEvent.getPath());
                        // 事件类型
                        System.out.println(curatorEvent.getType());
                        // 读取子节点数据
                        List<String> list = curatorEvent.getChildren();
                        for (String str : list) {
                            System.out.println(str);
                        }
                    }
                })
                .forPath("/get");
        Thread.sleep(5000);
        System.out.println("结束");
    }
}
```



### 检查节点是否存在

```java
public class CuratorExists {

    String IP = "192.168.60.130:2181,192.168.60.130:2182,192.168.60.130:2183";
    CuratorFramework client;

    @Before
    public void before() {
		...
    }

    @After
    public void after() {
        client.close();
    }

    @Test
    public void exists1() throws Exception {
        // 判断节点是否存在
       Stat stat = client.checkExists()
                 // 节点路径
                .forPath("/node2");	// 如果不存在，stat为null
        System.out.println(stat.getVersion());	
    }

    @Test
    public void exists2() throws Exception {
        // 异步方式判断节点是否存在
        client.checkExists()
                 .inBackground(new BackgroundCallback() {
                     public void processResult(CuratorFramework curatorFramework, 
                                           CuratorEvent curatorEvent) throws Exception {
                         // 节点路径
                         System.out.println(curatorEvent.getPath());
                         // 事件类型
                         System.out.println(curatorEvent.getType());
                         System.out.println(curatorEvent.getStat().getVersion());
                     }
                 })
                .forPath("/node2");
        Thread.sleep(5000);
        System.out.println("结束");
    }
}
```



### Watcher

Curator 提供了两种 Watcher(Cache) 来监听结点的变化

- Node Cache：只是监听某一个特定的节点，监听节点的新增和修改
- PathChildren Cache：监控一个ZNode的子节点. 当一个子节点增加， 更新， 删除时， Path Cache会改变它的状态， 会包含最新的子节点， 子节点的数据和状态。

```java
public class CuratorWatcher {

    String IP = "xxx:2181";
    CuratorFramework client;

    @Before
    public void before() {
		...
    }

    @After
    public void after() {
        client.close();
    }


    @Test
    public void watcher1() throws Exception {
        // 监视某个节点的数据变化
        // arg1:连接对象
        // arg2:监视的节点路径
        final NodeCache nodeCache = new NodeCache(client, "/dubbo");
        // 启动监视器对象
        nodeCache.start();
        nodeCache.getListenable().addListener(new NodeCacheListener() {
            // 节点变化时回调的方法
            public void nodeChanged() throws Exception {
                System.out.println(nodeCache.getCurrentData().getPath());
                System.out.println(new String(nodeCache.getCurrentData().getData()));
            }
        });
        Thread.sleep(100000);
        System.out.println("结束");
        // 关闭监视器对象
        nodeCache.close();
    }

    @Test
    public void watcher2() throws Exception {
        // 监视子节点的变化
        // arg1:连接对象
        // arg2:监视的节点路径
        // arg3:事件中是否可以获取节点的数据
        PathChildrenCache pathChildrenCache = 
            	new PathChildrenCache(client, "/dubbo", true);
        // 启动监听
        pathChildrenCache.start();
        pathChildrenCache.getListenable().addListener(new PathChildrenCacheListener() {
            // 当子节点方法变化时回调的方法
            public void childEvent(CuratorFramework curatorFramework, 
                     	PathChildrenCacheEvent pathChildrenCacheEvent) throws Exception {
                // 节点的事件类型
                System.out.println(pathChildrenCacheEvent.getType());
                // 节点的路径
                System.out.println(pathChildrenCacheEvent.getData().getPath());
                // 节点数据
                System.out.println(new String(pathChildrenCacheEvent.getData().getData()));
            }
        });
        Thread.sleep(100000);
        System.out.println("结束");
        // 关闭监听
        pathChildrenCache.close();
    }
}
```



### 事务

```java
public class CuratorTransaction {

    String IP = "192.168.60.130:2181,192.168.60.130:2182,192.168.60.130:2183";
    CuratorFramework client;

    @Before
    public void before() {
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        client = CuratorFrameworkFactory.builder()
                .connectString(IP)
                .sessionTimeoutMs(10000).retryPolicy(retryPolicy)
                .namespace("create").build();
        client.start();
    }

    @After
    public void after() {
        client.close();
    }

    @Test
    public void tra1() throws Exception {
        // 开启事务
        client.inTransaction()
                .create().forPath("/node1","node1".getBytes())
                .and()
                .create().forPath("/node2","node2".getBytes())
                .and()
                //事务提交
                .commit();
    }
}
```



### 分布式锁

```java
public class CuratorLock {

    String IP = "192.168.60.130:2181,192.168.60.130:2182,192.168.60.130:2183";
    CuratorFramework client;

    @Before
    public void before() {
		...
    }

    @After
    public void after() {
        client.close();
    }


    @Test
    public void lock1() throws Exception {
        // 排他锁
        // arg1:连接对象
        // arg2:节点路径
        InterProcessLock interProcessLock = new InterProcessMutex(client, "/lock1");
        System.out.println("等待获取锁对象!");
        // 获取锁
        interProcessLock.acquire();
        for (int i = 1; i <= 10; i++) {
            Thread.sleep(3000);
            System.out.println(i);
        }
        // 释放锁
        interProcessLock.release();
        System.out.println("等待释放锁!");
    }

    @Test
    public void lock2() throws Exception {
        // 读写锁
        InterProcessReadWriteLock interProcessReadWriteLock = new InterProcessReadWriteLock(client, "/lock1");
        // 获取读锁对象
        InterProcessLock interProcessLock = interProcessReadWriteLock.readLock();
        System.out.println("等待获取锁对象!");
        // 获取锁
        interProcessLock.acquire();
        for (int i = 1; i <= 10; i++) {
            Thread.sleep(3000);
            System.out.println(i);
        }
        // 释放锁
        interProcessLock.release();
        System.out.println("等待释放锁!");
    }

    @Test
    public void lock3() throws Exception {
        // 读写锁
        InterProcessReadWriteLock interProcessReadWriteLock = new InterProcessReadWriteLock(client, "/lock1");
        // 获取写锁对象
        InterProcessLock interProcessLock = interProcessReadWriteLock.writeLock();
        System.out.println("等待获取锁对象!");
        // 获取锁
        interProcessLock.acquire();
        for (int i = 1; i <= 10; i++) {
            Thread.sleep(3000);
            System.out.println(i);
        }
        // 释放锁
        interProcessLock.release();
        System.out.println("等待释放锁!");
    }

}
```



## zookeeper四字监控命令

安装nc：

```shell
# root用户安装
# 下载安装包
wget http://vault.centos.org/6.6/os/x86_64/Packages/nc-1.84-22.el6.x86_64.rpm
# rpm安装
rpm -iUv nc-1.84-22.el6.x86_64.rpm
```

使用：

```
echo <四字命令> | nc <ip> <端口号>
```



### conf

输出相关服务配置的详细信息。

| 属性              | 含义                                                         |
| ----------------- | ------------------------------------------------------------ |
| clientPort        | 客户端端口号                                                 |
| dataDir           | 数据快照文件目录，默认情况下100000次事务操作生成一次快照     |
| dataLogDir        | 事物日志文件目录，生产环境中放在独立的磁盘上                 |
| tickTime          | 服务器之间或客户端与服务器之间维持心跳的时间间隔（以毫秒为单位） |
| maxClientCnxns    | 最大连接数                                                   |
| minSessionTimeout | 最小session超时，minSessionTimeout=tickTime*2                |
| maxSessionTimeout | 最大session超时，maxSessionTimeout=tickTime*20               |
| serverId          | 服务器编号                                                   |
| initLimit         | 集群中的 follower 服务器与leader服务器之间初始连接时能容忍的最多心跳数。此配置表示，允许 follower（相对于 *leader* 而言的“客户端”）连接并同步到 leader 的初始化连接时间，它以 tickTime 的倍数来表示。当超过设置倍数的 tickTime 时间，则连接失败。 |
| syncLimit         | 集群中的follower服务器与leader服务器之间请求和应答之间能容忍的最多心跳数。此配置表示， leader 与 follower 之间发送消息，请求和应答时间长度。如果 follower 在设置的时间内不能与 leader 进行通信，那么此 follower 将被丢弃。 |
| electionAlg       | 0:基于UDP的LeaderElection 1:基于UDP的 FastLeaderElection 2:基于UDP和认证的 FastLeaderElection 3:基于TCP的FastLeaderElection。默认值为3，另外三种算法已经被弃用， 并且有计划在之后的版本中将它们彻底删除而不再支持 |
| electionPort      | 选举端口                                                     |
| quorumPort        | 数据通信端口                                                 |
| peerType          | 是否为观察者 1为观察者                                       |



### cons

列出所有连接到这台服务器的客户端连接/会话的详细信息。

| 属性     | 含义                                                         |
| -------- | ------------------------------------------------------------ |
| ip       | ip地址                                                       |
| port     | 端口号                                                       |
| queued   | 等待被处理的请求数， 请求缓存在队列中                        |
| received | 收到的包数                                                   |
| sent     | 发送的包数                                                   |
| sid      | 会话id                                                       |
| lop      | 最后的操作：GETD-读取数据 DELE-删除数据 CREA-创建数据 PING-心跳包 |
| est      | 连接时间戳                                                   |
| to       | 超时时间                                                     |
| lcxid    | 当前会话的操作id（每一次读/写操作都会使事务增加1）           |
| lzxid    | 最大事务id                                                   |
| lresp    | 最后响应时间戳                                               |
| llat     | 最后/最新 延时                                               |
| minlat   | 最小延时                                                     |
| maxlat   | 最大延时                                                     |
| avglat   | 平均延时                                                     |



### crst

重置当前这台服务器所有连接/会话的统计信息。



### dump

列出未经处理的会话和临时节点

```shell
[zk: localhost:2181(CONNECTED) 2] create -e /temp "temp"
Created /temp
[zk: localhost:2181(CONNECTED) 3] create -e /temp1 "temp"
Created /temp1

# 另一个客户端
SessionTracker dump:
Session Sets (7):
0 expire at Wed Apr 29 22:00:00 CST 2020:
0 expire at Wed Apr 29 22:00:06 CST 2020:
0 expire at Wed Apr 29 22:00:10 CST 2020:
0 expire at Wed Apr 29 22:00:12 CST 2020:
0 expire at Wed Apr 29 22:00:16 CST 2020:
1 expire at Wed Apr 29 22:00:22 CST 2020:
        0x170e2c8f16100b5
1 expire at Wed Apr 29 22:00:26 CST 2020:
        0x170e2c8f16100a1
ephemeral nodes dump:
Sessions with Ephemerals (1):
0x170e2c8f16100b5:
        /temp1
        /temp
```



### envi

输出关于服务器的环境配置信息。

| 属性              | 含义                                          |
| ----------------- | --------------------------------------------- |
| zookeeper.version | 版本                                          |
| host.name         | host信息                                      |
| java.version      | java版本                                      |
| java.vendor       | 供应商                                        |
| java.home         | 运行环境所在目录                              |
| java.class.path   | classpath                                     |
| java.library.path | 第三方库指定非java类包的位置（如： dll， so） |
| java.io.tmpdir    | 默认的临时文件路径                            |
| java.compiler     | JIT 编译器的名称                              |
| os.name           | Linux                                         |
| os.arch           | amd64                                         |
| os.version        | 3.10.0-514.el7.x86_64                         |
| user.name         | zookeeper                                     |
| user.home         | /home/zookeeper                               |
| user.dir          | /home/zookeeper/zookeeper2181/bin             |



### ruok

测试服务是否处于正确运行状态。ruok -> are you ok?



### srvr

输出服务器的详细信息

| 属性                | 含义       |
| ------------------- | ---------- |
| Zookeeper version   | 版本       |
| Latency min/avg/max | 延时       |
| Received            | 收包       |
| Sent                | 发包       |
| Connections         | 连接数     |
| Outstanding         | 堆积数     |
| Zxid                | 最大事物id |
| Mode                | 服务器角色 |
| Node count          | 节点数     |



### stat

和srvr类似， 但是会包含每个连接的会话信息。



### srst

重置server状态



### wchs  

列出服务器watches的简洁信息  

| connectsions | 连接数      |
| ------------ | ----------- |
| watch-paths  | watch节点数 |
| watchers     | watcher数量 |



### wchc  

通过session分组， 列出watch的所有节点， 它的输出的是一个与 watch 相关的会话的节点列表  

问题：

```shell
wchc is not executed because it is not in the whitelist
```

解决方法：

```shell
# 修改启动指令 zkServer.sh
# 注意找到这个信息
else
    echo "JMX disabled by user request" >&2
    ZOOMAIN="org.apache.zookeeper.server.quorum.QuorumPeerMain"
fi

# 下面添加如下信息
ZOOMAIN="-Dzookeeper.4lw.commands.whitelist=* ${ZOOMAIN}"
```



### wchp

通过路径分组， 列出所有的 watch 的session id信息  

问题：

```shell
wchp is not executed because it is not in the whitelist
```

解决方法：

```shell
# 修改启动指令 zkServer.sh
# 注意找到这个信息
else
    echo "JMX disabled by user request" >&2
    ZOOMAIN="org.apache.zookeeper.server.quorum.QuorumPeerMain"
fi

# 下面添加如下信息
ZOOMAIN="-Dzookeeper.4lw.commands.whitelist=* ${ZOOMAIN}"
```



### mntr  

列出服务器的健康状态

| 属性                          | 含义                 |
| ----------------------------- | -------------------- |
| zk_version                    | 版本                 |
| zk_avg_latency                | 平均延时             |
| zk_max_latency                | 最大延时             |
| zk_min_latency                | 最小延时             |
| zk_packets_received           | 收包数               |
| zk_packets_sent               | 发包数               |
| zk_num_alive_connections      | 连接数               |
| zk_outstanding_requests       | 堆积请求数           |
| zk_server_state               | leader/follower 状态 |
| zk_znode_count                | znode数量            |
| zk_watch_count                | watch数量            |
| zk_ephemerals_count           | 临时节点（znode）    |
| zk_approximate_data_size      | 数据大小             |
| zk_open_file_descriptor_count | 打开的文件描述符数量 |
| zk_max_file_descriptor_count  | 最大文件描述符数量   |