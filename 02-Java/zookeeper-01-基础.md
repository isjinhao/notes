##  概述

**命名**

Zookeeper是一个开源的分布式的，为分布式应用提供协调服务的Apache项目。大数据生态系统里很多组件的命名都是某种动物的昆虫，入Hadoop就是🐘，hive就是🐝。Zookeeper即动物园管理者，故名思义就算管理大数据生态系统各组件的管理员，如图所示。

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/0fdb6208-4e4b-497e-b68e-9f42f872dd53" /></div>
**工作机制**

Zookeeper从设计模式角度来理解：是一个基于观察者模式设计的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper就将负责通知已经在Zookeeper上注册的那些观察者做出 相应的反应。从而实现集群中类似Master/Slave管理模式。

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/feadb43a-aff8-4080-a0e3-b382dd273008" /></div>
**特点**

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/dcd831c8-131e-43d9-b80a-973b22dc4013" /></div>
1. Zookeeper：一个领导者（Leader），多个跟随者（Follwer）组成的集群。

2. - Leader负责进行投票的发起和决议，更新系统状态。
   - Follwer用于接收客户请求并向客户端返回结果，在选举Leader过程中参与投票。

3. 集群中只要有半数以上的节点存活，Zookeeper集群就能正常提供服务。

4. 全局数据一致：每一个Server保存一份相同的数据副本，Client无论连接到那个Server，数据都是一致的。

5. 更新请求顺序进行，来自同一个Client的更新请求按其发送顺序依次执行。

6. 数据更新原子性，一次数据更新要么成功要么失败。

7. 实时性，在一定的时间范围内，Client能读到最新的数据。

**数据结构**

- Zookeeper的数据结构与Unix文件系统很类似，整体上可以看成一棵树，每个节点称作一个ZNode。每一个ZNode默认能够存储1MB的数据，每个ZNode都可以通过其路径唯一标识。
- ZNode兼具文件和目录两种节点，既像文件一样维护者数据、元信息、ACL、时间戳等数据结构，又像目录一样可以作为路径标识的一部分。

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/e4d663a0-6ef3-4be3-b409-b54910b0f105" /></div>
**ZNode**

一个Znode大致分为三部分：

- 节点的数据：即Znode data（节点path，节点data）的关系就像是Java中的Map集合的(key, value)的关系。

- 节点的子节点：children。
- 节点的状态：stat，用来描述当前节点的创建，修改记录，包括cZxid、ctime等。

*属性说明*

- 在Zookeeper中使用get命令查看指定节点的data、stat信息：

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/868d048f-9568-4155-a1aa-a83a2e38f3a9" /></div>
- cZxid：数据节点创建时的事务ID
- ctime：数据节点创建时间
- mZxid：数据节点最后一次更新时的时间
- ptime：数据节点最后一次更新的时间
- Pzxid：数据节点最后一次被修改时的事务ID
- cversion：子节点的更改次数
- dataVersion：节点数据的更改次数
- aclVersion：节点的ACL的更改次数
- ephemeralOwner：如果节点是临时节点，则表示创建该节点的会话的SessionID，如果节点是持久节点，则该属性值为0
- dataLength：数据内容的长度
- numChildren：数据节点子节点的个数

*节点类型*

- 临时节点：该节点生命周期依赖于创建它们的会话，一但会话（Session）结束，临时节点将被自动删除，当然也可以手动删除。虽然每个临时的Znode都会绑定到一个客户端会话，但他们对所有的客户端还是可见的，另外Zookeeper的临时节点不允许拥有子节点。
- 持久化节点：该节点的生命周期不依赖于会话，并且只有在客户端显示执行删除操作的时候，它们才能被删除。

**应用场景**

*统一命名服务*

在分布式环境下，经常需要对应用服务进行统一命名，便于识别。（例如IP不容易记住，而域名容易记住）

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/dd5d4556-7d43-4410-9392-4669bbb128d5" /></div>
*统一配置管理*

java编程经常会遇到配置项， 比如数据库的url、 schema、 user和password等。 通常这些配置项我们会放置在配置文件中， 再将配置文件放置在服务器上当需要更改配置项时， 需要去服务器上修改对应的配置文件。 但是随着分布式系统的兴起， 由于许多服务都需要使用到该配置文件， 因此有必须保证该配置服务的高可用性（high
availability） 和各台服务器上配置数据的一致性。 通常会将配置文件部署在一个集群上，然而一个集群动辄上千台服务器， 此时如果再一台台服务器逐个修改配置文件那将是非常繁琐且危险的的操作， 因此就需要一种服务， 能够高效快速且可靠地完成配置项的更改等操作， 并能够保证各配置项在每台服务器上的数据一致性。

zookeeper就可以提供这样一种服务， 其使用Zab这种一致性协议来保证一致性。 现在有很多开源项目使用zookeeper来维护配置， 比如在hbase中， 客户端就是连接一个zookeeper， 获得必要的hbase集群的配置信息， 然后才可以进一步操作。 还有在开源的消息队列kafka中， 也使用zookeeper来维护broker的信息。 在alibaba开源的soa框架dubbo中也广泛的使用zookeeper管理一些配置来实现服务治理。  

简易步骤：

- 可将配置信息写入Zookeeper上的一个ZNode。
- 各个客户端服务器监听这个ZNode。
- 一旦ZNode中的数据被修改，Zookeeper将通知各个客户端服务器。

<div align="center"><img width="40%" src="http://blogfileqiniu.isjinhao.site/ae951e55-c484-4418-8132-5df813647a93" /></div>
*分布式锁服务*

一个集群是一个分布式系统， 由多台服务器组成。 为了提高并发度和可靠性，多台服务器上运行着同一种服务。 当多个服务在运行时就需要协调各服务的进度， 有时候需要保证当某个服务在进行某个操作时， 其他的服务都不能进行该操作， 即对该操作进行加锁， 如果当前机器挂掉后， 释放锁并fail over 到其他的机器继续执行该服务。

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/6c79fcca-9c66-43f9-ba10-d2459570f0ca" /></div>
*统一集群管理*

一个集群有时会因为各种软硬件故障或者网络故障， 出现某些服务器挂掉而被移除集群， 而某些服务器加入到集群中的情况， zookeeper会将这些服务器加入/移出的情况通知给集群中的其他正常工作的服务器， 以及时调整存储和计算等任务的分配和执行等。 此外zookeeper还会对故障的服务器做出诊断并尝试修复。  

简易步骤：

- 将节点信息写入Zookeeper上的一个ZNode。
- 监听这个ZNode可以获取它的实时状态变化。

<div align="center"><img width="40%" src="http://blogfileqiniu.isjinhao.site/9aec6b29-1497-40e1-ab41-e3fe6d1c17a0" /></div>
*生成分布式唯一ID*  

在过去的单库单表型系统中， 通常可以使用数据库字段自带的auto_increment属性来自动为每条记录生成一个唯一的ID。 但是分库分表后， 就无法在依靠数据库的auto_increment属性来唯一标识一条记录了。 此时我们就可以用zookeeper在分布式环境下生成全局唯一ID。 做法如下： 每次要生成一个新Id时， 创建一个持久顺序节点， 创建操作返回的节点序号， 即为新Id， 然后把比自己节点小的删除即可。



## 常用Shell命令

### 新增节点 

```shell
create [-s] [-e] path data 	# 其中-s 为有序节点， -e 临时节点
```

创建持久化节点并写入数据：

```shell
create /hadoop "123456"
```

创建持久化有序节点， 此时创建的节点名为指定节点名 + 自增序号。

```shell
[zk: localhost:2181(CONNECTED) 2] create -s /a "aaa"
Created /a0000000000
[zk: localhost:2181(CONNECTED) 3] create -s /b "bbb"
Created /b0000000001
[zk: localhost:2181(CONNECTED) 4] create -s /c "ccc"
Created /c0000000002
```

创建临时节点， 临时节点会在会话过期后被删除：

```shell
[zk: localhost:2181(CONNECTED) 5] create -e /tmp "tmp"
Created /tmp
```

创建临时有序节点， 临时节点会在会话过期后被删除：

```shell
[zk: localhost:2181(CONNECTED) 6] create -s -e /aa 'aaa'
Created /aa0000000004
[zk: localhost:2181(CONNECTED) 7] create -s -e /bb 'bbb'
Created /bb0000000005
[zk: localhost:2181(CONNECTED) 8] create -s -e /cc 'ccc'
Created /cc0000000006
```



### 更新节点

更新节点的命令是 `set`， 可以直接进行修改， 如下：  

```shell
[zk: localhost:2181(CONNECTED) 3] set /hadoop "345"
cZxid = 0x4
ctime = Thu Dec 12 14:55:53 CST 2019
mZxid = 0x5
mtime = Thu Dec 12 15:01:59 CST 2019
pZxid = 0x4
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
```

也可以基于版本号进行更改， 此时类似于乐观锁机制， 当你传入的数据版本号（dataVersion）和当前节点的数据版本号不符合时， zookeeper会拒绝本次修改：

```shell
[zk: localhost:2181(CONNECTED) 10] set /hadoop "3456" 0
version No is not valid : /hadoop
```



### 删除节点

删除节点的语法如下：

```shell
delete path [version]
```

和更新节点数据一样， 也可以传入版本号， 当你传入的数据版本号（dataVersion）和当前节点的数据版本号不符合时， zookeeper 不会执行删除操作。  

```shell
[zk: localhost:2181(CONNECTED) 36] delete /hadoop 0
version No is not valid : /hadoop #无效的版本号
[zk: localhost:2181(CONNECTED) 37] delete /hadoop 1
[zk: localhost:2181(CONNECTED) 38]
```

要想删除某个节点及其所有后代节点， 可以使用递归删除， 命令为：

```shell
rmr path
```



### 查看节点

```shell
get path
```

```shell
[zk: localhost:2181(CONNECTED) 1] get /hadoop
123456
cZxid = 0x4
ctime = Thu Dec 12 14:55:53 CST 2019
mZxid = 0x4
mtime = Thu Dec 12 14:55:53 CST 2019
pZxid = 0x4
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
```

节点各个属性如下表。其中一个重要的概念是 Zxid（ZooKeeper TransactionId），ZooKeeper节点的每一次更改都具有唯一的 Zxid， 如果 Zxid1 小于 Zxid2， 则Zxid1 的更改发生在 Zxid2 更改之前 。

| 状态属性       | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| cZxid          | 数据节点创建时的事务 ID                                      |
| ctime          | 数据节点创建时的时间                                         |
| mZxid          | 数据节点最后一次更新时的事务 ID                              |
| mtime          | 数据节点最后一次更新时的时间                                 |
| pZxid          | 数据节点的子节点最后一次被修改时的事务 ID                    |
| cversion       | 子节点的更改次数                                             |
| dataVersion    | 节点数据的更改次数                                           |
| aclVersion     | 节点的 ACL 的更改次数                                        |
| ephemeralOwner | 如果节点是临时节点， 则表示创建该节点的会话的 SessionID； 如果节点是持久节点， 则该属性值为 0 |
| dataLength     | 数据内容的长度                                               |
| numChildren    | 数据节点当前的子节点个数                                     |



### 查看节点状态

可以使用 `stat` 命令查看节点状态， 它的返回值和 `get` 命令类似， 但不会返回节点数据  

```shell
[zk: localhost:2181(CONNECTED) 2] stat /hadoop
cZxid = 0x4
ctime = Thu Dec 12 14:55:53 CST 2019
mZxid = 0x4
mtime = Thu Dec 12 14:55:53 CST 2019
pZxid = 0x4
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
```



### 查看节点列表

查看节点列表有 `ls path` 和 `ls2 path` 两个命令， 后者是前者的增强， 不仅可以查看指定路径下的所有节点， 还可以查看当前节点的信息。（ls2 = ls + stat）

```shell
[zk: localhost:2181(CONNECTED) 0] ls /
[cluster, controller_epoch, brokers, storm, zookeeper, admin, ...]
[zk: localhost:2181(CONNECTED) 1] ls2 /
[cluster, controller_epoch, brokers, storm, zookeeper, admin, ....]
cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x130
cversion = 19
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 11
```



### 监听器 - get path [watch]

使用 `get path [watch]` 注册的监听器能够在节点内容发生改变的时候， 向客户端发出通知。 需要注意的是触发器是一次性的 (One-time trigger)， 即触发一次后就会立即失效。

```shell
[zk: localhost:2181(CONNECTED) 4] get /hadoop watch
[zk: localhost:2181(CONNECTED) 5] set /hadoop 45678
WATCHER::
WatchedEvent state:SyncConnected type:NodeDataChanged path:/hadoop
```



### 监听器 - stat path [watch]

使用 `stat path [watch]` 注册的监听器能够在节点状态发生改变的时候， 向客户端发出通知。  

```shell
[zk: localhost:2181(CONNECTED) 7] stat /hadoop watch
[zk: localhost:2181(CONNECTED) 8] set /hadoop 112233
WATCHER::
WatchedEvent state:SyncConnected type:NodeDataChanged path:/hadoop
```



### 监听器 - ls\ls2 path [watch]

使用 `ls path [watch]` 或 `ls2 path [watch]` 注册的监听器能够监听该节点下所有子节点的增加和删除操作。  

```shell
[zk: localhost:2181(CONNECTED) 9] ls /hadoop watch
[]
[zk: localhost:2181(CONNECTED) 10] create /hadoop/yarn "aaa"
WATCHER::
WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/hadoop
```



## zookeeper的acl权限控制

zookeeper 类似文件系统， client 可以创建节点、 更新节点、 删除节点， 那么如何做到对节点权限的控制呢？ 所以 zookeeper 的 access control list（acl） 访问控制列表可以做到这一点。使用 `scheme:id:permission`来标识，主要涵盖 3 个方面：

- 权限模式（scheme）：授权的策略

- 授权对象（id） ： 授权的对象

- 权限（permission） ： 授予的权限  

特性如下：

- zookeeper 的权限控制是基于每个 znode 节点的， 需要对每个节点设置权限
- 每个 znode 支持设置多种权限控制方案和多个权限
- 子节点不会继承父节点的权限， 客户端无权访问某节点， 但可能可以访问它的子节点

**权限模式**

| 方案   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| world  | 此模式下的id只有一个：anyone，代表登录zookeeper所有人（默认） |
| ip     | 对客户端使用IP地址认证                                       |
| auth   | 使用已添加认证的用户认证                                     |
| digest | 使用“用户名:密码”方式认证                                    |

**授予的权限**

| 权限   | ACL简写 | 描述                             |
| ------ | ------- | -------------------------------- |
| create | c       | 可以创建子节点                   |
| delete | d       | 可以删除子节点（仅下一级节点）   |
| read   | r       | 可以读取节点数据及显示子节点列表 |
| write  | w       | 可以设置节点数据                 |
| admin  | a       | 可以设置节点访问控制列表权限     |

**关于授权的命令**

| 命令    | 使用方式            | 描述         |
| ------- | ------------------- | ------------ |
| getAcl  | getAcl path         | 读取ACL权限  |
| setAcl  | setAcl path acl     | 设置ACL权限  |
| addauth | addauth scheme auth | 添加认证用户 |



### world授权模式

```shell
setAcl <path> world:anyone:<acl>
```

```shell
[zk: localhost:2181(CONNECTED) 1] create /node1 "node1"
Created /node1
[zk: localhost:2181(CONNECTED) 2] getAcl /node1
'world,'anyone # world方式对所有用户进行授权
: cdrwa #增、 删、 改、 查、 管理
[zk: localhost:2181(CONNECTED) 3] setAcl /node1 world:anyone:cdrwa
cZxid = 0x2
ctime = Fri Dec 13 22:25:24 CST 2019
mZxid = 0x2
mtime = Fri Dec 13 22:25:24 CST 2019
pZxid = 0x2
cversion = 0
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
```



### IP授权模式

```
setAcl <path> ip:<ip>:<acl>
```

```shell
[zk: localhost:2181(CONNECTED) 18] create /node2 "node2"
Created /node2
[zk: localhost:2181(CONNECTED) 23] setAcl /node2 ip:192.168.60.129:cdrwa
cZxid = 0xe
ctime = Fri Dec 13 22:30:29 CST 2019
mZxid = 0x10
mtime = Fri Dec 13 22:33:36 CST 2019
pZxid = 0xe
cversion = 0
dataVersion = 2
aclVersion = 1
ephemeralOwner = 0x0
dataLength = 20
numChildren = 0
[zk: localhost:2181(CONNECTED) 25] getAcl /node2
'ip,'192.168.60.129
: cdrwa

# 使用IP非 192.168.60.129 的机器
[zk: localhost:2181(CONNECTED) 0] get /node2
Authentication is not valid : /node2 # 没有权限
```



### Auth授权模式

```shell
addauth digest <user>:<password> # 添加认证用户
setAcl <path> auth:<user>:<acl>
```

每一个 zookeeper 客户端都会存在一些 auth 信息。如果服务器结点指定了 auth。那么连接服务器的客户端必须有对应的 auth，否则便是权限不足。

```shell
[zk: localhost:2181(CONNECTED) 2] create /node3 "node3"
Created /node3
[zk: localhost:2181(CONNECTED) 4] addauth digest itcast:123456 		# 添加认证用户
[zk: localhost:2181(CONNECTED) 1] setAcl /node3 auth:itcast:cdrwa
cZxid = 0x15
ctime = Fri Dec 13 22:41:04 CST 2019
mZxid = 0x15
mtime = Fri Dec 13 22:41:04 CST 2019
pZxid = 0x15
cversion = 0
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
[zk: localhost:2181(CONNECTED) 0] getAcl /node3
'digest,'itcast:673OfZhUE8JEFMcu0l64qI8e5ek=
: cdrwa

# 添加认证用户后可以访问
[zk: localhost:2181(CONNECTED) 3] get /node3
node3
cZxid = 0x15
ctime = Fri Dec 13 22:41:04 CST 2019
mZxid = 0x15
mtime = Fri Dec 13 22:41:04 CST 2019
pZxid = 0x15
cversion = 0
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
```



### Digest授权模式

Digest 授权模式和 Auth 授权模式一样的。只是在Auth模式中，由于事先添加了 auth，所以不再需要手动设置密码。在Digest 模式中，需要自己计算密码，再作为 `setAcl` 的参数。

```shell
setAcl <path> digest:<user>:<password>:<acl>
```

这里的密码是经过 SHA1 及 BASE64 处理的密文， 在SHELL中可以通过以下命令计算：  

```shell
echo -n <user>:<password> | openssl dgst -binary -sha1 | openssl base64
```

先来计算一个密文：

```shell
echo -n itheima:123456 | openssl dgst -binary -sha1 | openssl base64
```

案例：

```shell
[zk: localhost:2181(CONNECTED) 4] create /node4 "node4"
Created /node4

# 使用是上面算好的密文密码添加权限：
[zk: localhost:2181(CONNECTED) 5] setAcl /node4 digest:itheima:qlzQzCLKhBROghkooLvb+Mlwv4A=:cdrwa
cZxid = 0x1c
ctime = Fri Dec 13 22:52:21 CST 2019
mZxid = 0x1c
mtime = Fri Dec 13 22:52:21 CST 2019
pZxid = 0x1c
cversion = 0
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0

[zk: localhost:2181(CONNECTED) 6] getAcl /node4 
'digest,'itheima:qlzQzCLKhBROghkooLvb+Mlwv4A=
: cdrwa

[zk: localhost:2181(CONNECTED) 3] get /node4
Authentication is not valid : /node4 # 没有权限
[zk: localhost:2181(CONNECTED) 4] addauth digest itheima:123456 # 添加认证用户
[zk: localhost:2181(CONNECTED) 5] get /node4
1 # 成功读取数据
cZxid = 0x1c
ctime = Fri Dec 13 22:52:21 CST 2019
mZxid = 0x1c
mtime = Fri Dec 13 22:52:21 CST 2019
pZxid = 0x1c
cversion = 0
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
```



### 多种模式授权

```shell
[zk: localhost:2181(CONNECTED) 0] create /node5 "node5"
Created /node5
[zk: localhost:2181(CONNECTED) 1] addauth digest itcast:123456 # 添加认证用户
[zk: localhost:2181(CONNECTED) 2] setAcl /node5
ip:192.168.60.129:cdra,auth:itcast:cdrwa,digest:itheima:qlzQzCLKhBROghkooLvb+Mlwv4A=:cdrwa
```



### 超级管理员  

zookeeper 的权限管理模式有一种叫做super， 该模式提供一个超管可以方便的访问任何权限的节点

 假设这个超管是： super:admin， 需要先为超管生成密码的密文  

```shell
echo -n super:admin | openssl dgst -binary -sha1 | openssl base64
```

那么打开 zookeeper 目录下的 /bin/zkServer.sh 服务器脚本文件， 找到如下一行  

```shell
nohup $JAVA "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}"
```

这就是脚本中启动zookeeper的命令， 默认只有以上两个配置项， 我们需要加一个超管的配置项  

```shell
"-Dzookeeper.DigestAuthenticationProvider.superDigest=super:xQJmxLMiHGwaqBvst5y6rkB6HQs="
```

那么修改以后这条完整命令变成了  

```shell
nohup $JAVA "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-
Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" "-
Dzookeeper.DigestAuthenticationProvider.superDigest=super:xQJmxLMiHGwaqBv
st5y6rkB6HQs="\
-cp "$CLASSPATH" $JVMFLAGS $ZOOMAIN "$ZOOCFG" > "$_ZOO_DAEMON_OUT"
2>&1 < /dev/null &
```

之后启动zookeeper,输入如下命令添加权限  

```shell
addauth digest super:admin # 添加认证用户
```



## zookeeper javaAPI

znode 是 zooKeeper 集合的核心组件， zookeeper API 提供了一小组方法使用 zookeeper 集合来操纵 znode 的所有细节。客户端应该遵循以步骤， 与 zookeeper 服务器进行清晰和干净的交互。

- 连接到 zookeeper 服务器。 zookeeper 服务器为客户端分配会话ID。
- 定期向服务器发送心跳。 否则， zookeeper 服务器将过期会话ID， 客户端需要重新连接。
- 只要会话ID处于活动状态， 就可以获取/设置 znode。
- 所有任务完成后， 断开与 zookeeper 服务器的连接。 如果客户端长时间不活动， 则 zookeeper 服务器将自动断开客户端。  



### 连接到 ZooKeeper

```java
ZooKeeper(String connectionString, int sessionTimeout, Watcher watcher);
```

- connectionString - zookeeper主机
- sessionTimeout - 会话超时（以毫秒为单位）
- watcher - 实现“监视器”对象。 zookeeper 集合通过监视器对象返回连接状态。  

```java
public class ZookeeperConnection {
    public static void main(String[] args) {
        try {
            // 计数器对象
            CountDownLatch countDownLatch=new CountDownLatch(1);
            // arg1:服务器的ip和端口
            // arg2:客户端与服务器之间的会话超时时间  以毫秒为单位的
            // arg3:监视器对象
            ZooKeeper zooKeeper=new ZooKeeper("59.110.xxx.xxx:2181", 5000, new Watcher() {
                @Override
                public void process(WatchedEvent event) {
                    if(event.getState() == Event.KeeperState.SyncConnected) {
                        System.out.println("连接创建成功!");
                        countDownLatch.countDown();
                    }
                }
            });
            // 主线程阻塞等待连接对象的创建成功
            countDownLatch.await();
            // 会话编号
            System.out.println(zooKeeper.getSessionId());
            zooKeeper.close();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
}
```



### 新增节点

```java
// 同步方式
create(String path, byte[] data, List<ACL> acl, CreateMode createMode)

// 异步方式
create(String path, byte[] data, List<ACL> acl, CreateMode createMode，
AsyncCallback.StringCallback callBack, Object ctx)
```

- path：znode路径。 例如，`/node1 /node1/node11`
- data：要存储在指定 znode 路径中的数据
- acl：要创建的节点的访问控制列表。 `zookeeper API` 提供了一个静态接口 `ZooDefs.Ids`，用于获取一些基本的 acl 列表。 例如， `ZooDefs.Ids.OPEN_ACL_UNSAFE` 返回打开 znode 的 acl 列表。
- createMode：节点的类型，这是一个枚举。
- callBack：异步回调接口
- ctx：传递上下文参数  

测试：

```java
public class ZKCreate {

    String IP = "192.168.60.130:2181";
    ZooKeeper zooKeeper;

    @Before
    public void before() throws Exception {
        // 计数器对象
        CountDownLatch countDownLatch = new CountDownLatch(1);
        // arg1:服务器的ip和端口
        // arg2:客户端与服务器之间的会话超时时间  以毫秒为单位的
        // arg3:监视器对象
        zooKeeper = new ZooKeeper(IP, 5000, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                if (event.getState() == Event.KeeperState.SyncConnected) {
                    System.out.println("连接创建成功!");
                    countDownLatch.countDown();
                }
            }
        });
        // 主线程阻塞等待连接对象的创建成功
        countDownLatch.await();
    }

    @After
    public void after() throws Exception {
        zooKeeper.close();
    }

    @Test
    public void create1() throws Exception {
        // arg1:节点的路径
        // arg2:节点的数据
        // arg3:权限列表  world:anyone:cdrwa
        // arg4:节点类型  持久化节点
        zooKeeper.create("/create/node1", "node1".getBytes(), 
                         ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    }

    @Test
    public void create2() throws Exception {
        // Ids.READ_ACL_UNSAFE：world:anyone:r
        zooKeeper.create("/create/node2", "node2".getBytes(), 
                         ZooDefs.Ids.READ_ACL_UNSAFE, CreateMode.PERSISTENT);
    }

    @Test
    public void create3() throws Exception {
        /**
         * world 授权模式
         */
        // 权限列表
        List<ACL> acls = new ArrayList<>();
        // 授权模式 和 授权对象
        Id id = new Id("world", "anyone");
        // 权限设置
        acls.add(new ACL(ZooDefs.Perms.READ, id));
        acls.add(new ACL(ZooDefs.Perms.WRITE, id));
        zooKeeper.create("/create/node3", "node3".getBytes(), acls, 
                         CreateMode.PERSISTENT);
    }

    @Test
    public void create4() throws Exception {
        // ip 授权模式
        // 权限列表
        List<ACL> acls = new ArrayList<>();
        // 授权模式和授权对象
        Id id = new Id("ip", "192.168.60.130");
        // 权限设置
        acls.add(new ACL(ZooDefs.Perms.ALL, id));
        zooKeeper.create("/create/node4", "node4".getBytes(), acls, 
                         CreateMode.PERSISTENT);
    }

    @Test
    public void create5() throws Exception {
        // auth授权模式
        // 添加授权用户
        zooKeeper.addAuthInfo("digest", "itcast:123456".getBytes());
        zooKeeper.create("/create/node5", "node5".getBytes(), 
                         ZooDefs.Ids.CREATOR_ALL_ACL, CreateMode.PERSISTENT);
    }

    @Test
    public void create6() throws Exception {
        // auth 授权模式
        // 添加授权用户
        zooKeeper.addAuthInfo("digest", "itcast:123456".getBytes());
        // 权限列表
        List<ACL> acls = new ArrayList<ACL>();
        // 授权模式和授权对象
        Id id = new Id("auth", "itcast");
        // 权限设置
        acls.add(new ACL(ZooDefs.Perms.READ, id));
        zooKeeper.create("/create/node6", "node6".getBytes(), acls, 
                         CreateMode.PERSISTENT);
    }

    @Test
    public void create7() throws Exception {
        // digest 授权模式
        // 权限列表
        List<ACL> acls = new ArrayList<ACL>();
        // 授权模式和授权对象
        Id id = new Id("digest", "itheima:qlzQzCLKhBROghkooLvb+Mlwv4A=");
        // 权限设置
        acls.add(new ACL(ZooDefs.Perms.ALL, id));
        zooKeeper.create("/create/node7", "node7".getBytes(), acls, 
                         CreateMode.PERSISTENT);
    }

    @Test
    public void create8() throws Exception {
        // 持久化顺序节点
        // Ids.OPEN_ACL_UNSAFE world:anyone:cdrwa
        String result = zooKeeper.create("/create/node8", "node8".getBytes(), 
                                         ZooDefs.Ids.OPEN_ACL_UNSAFE, 
                                         CreateMode.PERSISTENT_SEQUENTIAL);
        System.out.println(result);
    }

    @Test
    public void create9() throws Exception {
        //  临时节点
        // Ids.OPEN_ACL_UNSAFE world:anyone:cdrwa
        String result = zooKeeper.create("/create/node9", "node9".getBytes(), 
                                         ZooDefs.Ids.OPEN_ACL_UNSAFE, 
                                         CreateMode.EPHEMERAL);
        System.out.println(result);
    }

    @Test
    public void create10() throws Exception {
        // 临时顺序节点
        // Ids.OPEN_ACL_UNSAFE world:anyone:cdrwa
        String result = zooKeeper.create("/create/node10", "node10".getBytes(), 
                                         ZooDefs.Ids.OPEN_ACL_UNSAFE, 
                                         CreateMode.EPHEMERAL_SEQUENTIAL);
        System.out.println(result);
    }

    @Test
    public void create11() throws Exception {
        // 异步方式创建节点
        zooKeeper.create("/create/node11", "node11".getBytes(), 
                         ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT, 
                         new AsyncCallback.StringCallback() {
            @Override
            public void processResult(int rc, String path, Object ctx, String name) {
                // 0 代表创建成功
                System.out.println(rc);
                // 节点的路径
                System.out.println(path);
                // 节点的路径
                System.out.println(name);
                // 上下文参数
                System.out.println(ctx);
            }
        }, "I am context");
        Thread.sleep(10000);
        System.out.println("结束");
    }
}
```



### 更新节点

```java
// 同步方式
setData(String path, byte[] data, int version);
// 异步方式
setData(String path, byte[] data, int version，
	AsyncCallback.StatCallback callBack, Object ctx);
```

- path：znode路径
- data：要存储在指定znode路径中的数据。
- version：znode的当前版本。 每当数据更改时， ZooKeeper会更新znode的版本号。
- callBack：异步回调接口
- ctx：传递上下文参数  

测试：

```java
public class ZKSet {

    String IP = "192.168.60.130:2181";
    ZooKeeper zooKeeper;

    @Before
    public void before() throws Exception {
       ...
    }

    @After
    public void after() throws Exception {
        zooKeeper.close();
    }

    @Test
    public void set1() throws Exception {
        // arg1:节点的路径
        // arg2:修改的数据
        // arg3:数据版本号 -1代表版本号不参与更新
        Stat stat = zooKeeper.setData("/set/node1", "node13".getBytes(), -1);
        // 当前节点的版本号
        System.out.println(stat.getVersion());

    }

    @Test
    public void set2() throws Exception {
        zooKeeper.setData("/set/node1", "node14".getBytes(), -1, 
                          new AsyncCallback.StatCallback() {
            @Override
            public void processResult(int rc, String path, Object ctx, Stat stat) {
                // 0代表修改成功
                System.out.println(rc);
                // 节点的路径
                System.out.println(path);
                // 上下文参数对象
                System.out.println(ctx);
                // 属性描述对象
                System.out.println(stat.getVersion());
            }
        }, "I am Context");
        Thread.sleep(10000);
        System.out.println("结束");
    }
}
```



### 删除节点

```java
// 同步方式
delete(String path, int version);
// 异步方式
delete(String path, int version, AsyncCallback.VoidCallback callBack, Object ctx);
```

- path：znode路径
- version：znode的当前版本
- callBack：异步回调接口
- ctx：传递上下文参数  

测试：

```java
public class ZKDelete {
    
    String IP = "192.168.60.130:2181";
    ZooKeeper zooKeeper;

    @Before
    public void before() throws Exception {
        ...
    }

    @After
    public void after() throws Exception {
        zooKeeper.close();
    }

    @Test
    public void delete1() throws Exception {
        // arg1:删除节点的节点路径
        // arg2:数据版本信息 -1代表删除节点时不考虑版本信息
        zooKeeper.delete("/delete/node1",-1);
    }

    @Test
    public void delete2() throws Exception {
        // 异步使用方式
        zooKeeper.delete("/delete/node2", -1, new AsyncCallback.VoidCallback() {
            @Override
            public void processResult(int rc, String path, Object ctx) {
                // 0代表删除成功
                System.out.println(rc);
                // 节点的路径
                System.out.println(path);
                // 上下文参数对象
                System.out.println(ctx);
            }
        },"I am Context");
        Thread.sleep(10000);
        System.out.println("结束");
    }
}
```



### 查看节点

```java
// 同步方式
getData(String path, boolean b, Stat stat)
// 异步方式
getData(String path, boolean b, AsyncCallback.DataCallback callBack, Object ctx)
```

- path：znode路径。
- b：是否使用连接对象中注册的监视器。
- stat：返回znode的元数据。
- callBack：异步回调接口
- ctx：传递上下文参数  

测试：

```java
public class ZKGet {

    String IP = "192.168.60.130:2181";
    ZooKeeper zooKeeper;

    @Before
    public void before() throws Exception {
        ...
    }

    @After
    public void after() throws Exception {
        zooKeeper.close();
    }

    @Test
    public void get1() throws Exception {
        // arg1:节点的路径
        // arg3:读取节点属性的对象
        Stat stat=new Stat();
        byte []bys = zooKeeper.getData("/get/node1", false, stat);
        // 打印数据
        System.out.println(new String(bys));
        // 版本信息
        System.out.println(stat.getVersion());
    }

    @Test
    public void get2() throws Exception {
        // 异步方式
        zooKeeper.getData("/get/node1", false, new AsyncCallback.DataCallback() {
            @Override
            public void processResult(int rc, String path, Object ctx, 
                                      byte[] data, Stat stat) {
                // 0代表读取成功
                System.out.println(rc);
                // 节点的路径
                System.out.println(path);
                // 上下文参数对象
                System.out.println(ctx);
                // 数据
                System.out.println(new String(data));
                // 属性对象
                System.out.println(stat.getVersion());
            }
        },"I am Context");
        Thread.sleep(10000);
        System.out.println("结束");
    }
}
```



### 查看子节点

```java
// 同步方式
getChildren(String path, boolean b)
// 异步方式
getChildren(String path, boolean b, AsyncCallback.ChildrenCallback callBack, Object ctx)
```

- path：Znode路径。
- b：是否使用连接对象中注册的监视器。
- callBack：异步回调接口。
- ctx：传递上下文参数  

测试：

```java
public class ZKGetChid {
    String IP = "192.168.60.130:2181";
    ZooKeeper zooKeeper;

    @Before
    public void before() throws Exception {
        ...
    }

    @After
    public void after() throws Exception {
        zooKeeper.close();
    }

    @Test
    public void get1() throws Exception {
        // arg1:节点的路径
        List<String> list = zooKeeper.getChildren("/get", false);
        for (String str : list) {
            System.out.println(str);
        }
    }

    @Test
    public void get2() throws Exception {
        // 异步用法
        zooKeeper.getChildren("/get", false, new AsyncCallback.ChildrenCallback() {
            @Override
            public void processResult(int rc, String path, 
                                      Object ctx, List<String> children) {
                // 0代表读取成功
                System.out.println(rc);
                // 节点的路径
                System.out.println(path);
                // 上下文参数对象
                System.out.println(ctx);
                // 子节点信息
                for (String str : children) {
                    System.out.println(str);
                }
            }
        },"I am Context");
        Thread.sleep(10000);
        System.out.println("结束");
    }
}
```



### 检查节点是否存在

```java
// 同步方法
exists(String path, boolean b);
// 异步方法
exists(String path, boolean b, AsyncCallback.StatCallback callBack, Object ctx);
```

- path：znode路径。
- b：是否使用连接对象中注册的监视器。
- callBack：异步回调接口。
- ctx：传递上下文参数  

测试：

```java
public class ZKExists {
    String IP = "192.168.60.130:2181";
    ZooKeeper zooKeeper;

    @Before
    public void before() throws Exception {
		...
    }

    @After
    public void after() throws Exception {
        zooKeeper.close();
    }

    @Test
    public void exists1() throws Exception {
        // arg1:节点的路径
        Stat stat=zooKeeper.exists("/exists1",false);
        // 节点的版本信息
        System.out.println(stat.getVersion());
    }

    @Test
    public void exists2() throws Exception {
        // 异步方式
        zooKeeper.exists("/exists1", false, new AsyncCallback.StatCallback() {
            @Override
            public void processResult(int rc, String path, Object ctx, Stat stat) {
                // 0 代表方式执行成功
                System.out.println(rc);
                // 节点的路径
                System.out.println(path);
                // 上下文参数
                System.out.println(ctx);
                // 节点的版本信息
                System.out.println(stat.getVersion());
            }
        },"I am Context");
        Thread.sleep(10000);
        System.out.println("结束");
    }
}
```



## Watcher

zookeeper提供了数据的发布/订阅功能，多个订阅者可同时监听某一特定主题对象，当该主题对象的自身状态发生变化时(例如节点内容改变、节点下的子节点列表改变等)，会实时、主动通知所有订阅者。

zookeeper采用了Watcher机制实现数据的发布/订阅功能。该机制在被订阅对象发生变化时会异步通知客户端，因此客户端不必在Watcher注册后轮询阻塞，从而减轻了客户端压力。

watcher机制实际上与观察者模式类似，也可看作是一种观察者模式在分布式场景下的实现方式。



### watcher架构

Watcher 实现由三个部分组成：

- Zookeeper 服务端
- Zookeeper 客户端
- 客户端的 ZKWatchManager 对象  

客户端首先将 Watcher 注册到服务端，同时将 Watcher 对象保存到客户端的 Watch 管理器中。当 ZooKeeper 服务端监听的数据状态发生变化时，服务端会主动通知客户端，接着客户端的 Watch 管理器会触发相关 Watcher 来回调相应处理逻辑，从而完成整体的数据发布/订阅流程。  

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/b96ffd42-dee9-4e6e-b084-b940fc7cbd9f" /></div>



### watcher特性

| 特性           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| 一次性         | watcher是一次性的，一旦被触发就会移除，再次使用时需要重新注册 |
| 客户端顺序回调 | watcher回调是顺序串行化执行的，只有回调后客户端才能看到最新的数 据状态。一个watcher回调逻辑不应该太多，以免影响别的watcher执行 |
| 轻量级         | WatchEvent是最小的通信单元，结构上只包含通知状态、事件类型和节点路径，并不会告诉数据节点变化前后的具体内容； |
| 时效性         | watcher只有在当前session彻底失效时才会无效，若在session有效期内快速重连成功，则watcher依然存在，仍可接收到通知； |



### Watcher接口设计

`Watcher` 是一个接口，任何实现了 `Watcher` 接口的类就是一个新的 `Watcher`。`Watcher` 内部包含了两个枚举类：`KeeperState`、`EventType`。

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/7f8ee620-44e9-4714-8373-72c9cd054728" /></div>

**Watcher通知状态（KeeperState）**

`KeeperState` 是客户端与服务端连接状态发生变化时对应的通知类型。枚举属性如下：  

| 枚举属性      | 说明                     |
| ------------- | ------------------------ |
| SyncConnected | 客户端与服务器正常连接时 |
| Disconnected  | 客户端与服务器断开连接时 |
| Expired       | 会话session失效时        |
| AuthFailed    | 身份认证失败时           |

**Watcher事件类型（EventType）**

`EventType` 是数据节点（znode）发生变化时对应的通知类型。`EventType` 变化时 `KeeperState` 永远处于`SyncConnected` 通知状态下；当 `KeeperState` 发生变化时，`EventType` 永远为 `None`。枚举属性如下：  

| 枚举属性            | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| None                | 无                                                           |
| NodeCreated         | Watcher 监听的数据节点被创建时                               |
| NodeDeleted         | Watcher 监听的数据节点被删除时                               |
| NodeDataChanged     | Watcher 监听的数据节点内容发生变更时（无论内容数据是否变化） |
| NodeChildrenChanged | Watcher 监听的数据节点的子节点列表发生变更时                 |

注：客户端接收到的相关事件通知中只包含状态及类型等信息，不包括节点变化前后的具体内容，变化前的数据需业务自身存储，变化后的数据需调用get等方法重新获取。



### 事件的捕获

可以注册watcher的方法：`get()`、`exists()`、`getChildren()`。

可以触发watcher的方法：`create()`、`delete()`、`set()`。连接断开的情况下触发的 `Watcher` 会丢失。

`new ZooKeeper()` 时注册的是 default watcher，它不是一次性的，只对client的连接状态变化作出反应。

**操作与事件类型的对应**

|                       | event For “/path”      | event For “/path/child” |
| :-------------------- | :--------------------- | :---------------------- |
| create(“/path”)       | ET.NodeCreated         | 无                      |
| delete(“/path”)       | ET.NodeDeleted         | 无                      |
| set(“/path”)          | ET.NodeDataChanged     | 无                      |
| create(“/path/child”) | ET.NodeChildrenChanged | ET.NodeCreated          |
| delete(“/path/child”) | ET.NodeChildrenChanged | ET.NodeDeleted          |
| set(“/path/child”)    | 无                     | ET.NodeDataChanged      |

**事件类型与 Watcher 的对应关系**

| event For “/path”      | Default | exists(“/path”) | getData(“/path”) | getChildren(“/path”) |
| ---------------------- | ------- | --------------- | ---------------- | -------------------- |
| ET.None                | √       | √               | √                | √                    |
| ET.NodeCreated         |         | √               | √                |                      |
| ET.NodeDeleted         |         | √               | √                |                      |
| ET.NodeDataChanged     |         | √               | √                |                      |
| ET.NodeChildrenChanged |         |                 |                  | √                    |

**操作与 Watcher 的对应**

|                        | exits("/path") | getData(“/path”) | getChildren(“/path”) | exits("/path/child") | getData(“/path/child”) | getChildren(“/path/child”) |
| ---------------------- | -------------- | ---------------- | -------------------- | -------------------- | ---------------------- | -------------------------- |
| create(“/path”)        | √              | √                | 会报错               |                      |                        |                            |
| delete(“/path”)        | √              | √                | √（这个要注意）      |                      |                        |                            |
| setData(“/path”)       | √              | √                |                      |                      |                        |                            |
| create(“/path/child”)  |                |                  | √                    | √                    | √                      |                            |
| delete(“/path/child”)  |                |                  | √                    | √                    | √                      | √                          |
| setData(“/path/child”) |                |                  |                      | √                    | √                      |                            |



### Default

```java
public class ZKConnectionWatcher implements Watcher {

    // 计数器对象
    static CountDownLatch countDownLatch = new CountDownLatch(1);
    // 连接对象
    static ZooKeeper zooKeeper;

    @Override
    public void process(WatchedEvent event) {
        try {
            // 事件类型
            if (event.getType() == Event.EventType.None) {
                if (event.getState() == Event.KeeperState.SyncConnected) {
                    System.out.println("连接创建成功!");
                    countDownLatch.countDown();
                } else if (event.getState() == Event.KeeperState.Disconnected) {
                    System.out.println("断开连接！");
                } else if (event.getState() == Event.KeeperState.Expired) {
                    System.out.println("会话超时!");
                    zooKeeper = new ZooKeeper("192.168.60.130:2181", 5000, 
                                              new ZKConnectionWatcher());
                } else if (event.getState() == Event.KeeperState.AuthFailed) {
                    System.out.println("认证失败！");
                }
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    public static void main(String[] args) {
        try {
            zooKeeper = new ZooKeeper("192.168.60.130:2181", 5000, 
                                      new ZKConnectionWatcher());
            // 阻塞线程等待连接的创建
            countDownLatch.await();
            // 会话id
            System.out.println(zooKeeper.getSessionId());
            // 添加授权用户
            zooKeeper.addAuthInfo("digest1","itcast1:1234561".getBytes());
            byte [] bs=zooKeeper.getData("/node1", false, null);
            System.out.println(new String(bs));
            Thread.sleep(50000);
            zooKeeper.close();
            System.out.println("结束");
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
}
```



### Exists

```java
public class ZKWatcherExists {

    String IP = "192.168.60.130:2181";
    ZooKeeper zooKeeper = null;

    @Before
    public void before() throws IOException, InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(1);
        // 连接zookeeper客户端
        zooKeeper = new ZooKeeper(IP, 6000, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println("连接对象的参数!");
                // 连接成功
                if (event.getState() == Event.KeeperState.SyncConnected) {
                    countDownLatch.countDown();
                }
                System.out.println("path = " + event.getPath());
                System.out.println("eventType = " + event.getType());
            }
        });
        countDownLatch.await();
    }

    @After
    public void after() throws InterruptedException {
        zooKeeper.close();
    }

    @Test
    public void watcherExists1() throws KeeperException, InterruptedException {
        // arg1:节点的路径
        // arg2:使用连接对象中的watcher
        zooKeeper.exists("/watcher1", true);
        Thread.sleep(50000);
        System.out.println("结束");
    }

    @Test
    public void watcherExists2() throws KeeperException, InterruptedException {
        // arg1:节点的路径
        // arg2:自定义watcher对象
        zooKeeper.exists("/watcher1", new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println("自定义watcher");
                System.out.println("path = " + event.getPath());
                System.out.println("eventType = " + event.getType());
            }
        });
        Thread.sleep(50000);
        System.out.println("结束");
    }

    @Test
    public void watcherExists3() throws KeeperException, InterruptedException {
        // watcher一次性
        Watcher watcher = new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                try {
                    System.out.println("自定义watcher");
                    System.out.println("path = " + event.getPath());
                    System.out.println("eventType = " + event.getType());
                    zooKeeper.exists("/watcher1", this);
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
        };
        zooKeeper.exists("/watcher1", watcher);
        Thread.sleep(80000);
        System.out.println("结束");
    }

    @Test
    public void watcherExists4() throws KeeperException, InterruptedException {
        // 注册多个监听器对象
        zooKeeper.exists("/watcher1", new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println("1");
                System.out.println("path=" + event.getPath());
                System.out.println("eventType=" + event.getType());
            }
        });
        zooKeeper.exists("/watcher1", new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println("2");
                System.out.println("path = " + event.getPath());
                System.out.println("eventType = " + event.getType());
            }
        });
        Thread.sleep(80000);
        System.out.println("结束");
    }
}
```



### Get

```java
public class ZKWatcherGetData {
    String IP = "192.168.60.130:2181";
    ZooKeeper zooKeeper = null;

    @Before
    public void before() throws IOException, InterruptedException {
		...
    }

    @After
    public void after() throws InterruptedException {
        zooKeeper.close();
    }

    @Test
    public void watcherGetData1() throws KeeperException, InterruptedException {
        // arg1:节点的路径
        // arg2:使用连接对象中的watcher
        zooKeeper.getData("/watcher2", true, null);
        Thread.sleep(50000);
        System.out.println("结束");
    }

    @Test
    public void watcherGetData2() throws KeeperException, InterruptedException {
        // arg1:节点的路径
        // arg2:自定义watcher对象
        zooKeeper.getData("/watcher2", new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println("自定义watcher");
                System.out.println("path=" + event.getPath());
                System.out.println("eventType=" + event.getType());
            }
        }, null);
        Thread.sleep(50000);
        System.out.println("结束");
    }

    @Test
    public void watcherGetData3() throws KeeperException, InterruptedException {
        // 一次性
        Watcher watcher = new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                try {
                    System.out.println("自定义watcher");
                    System.out.println("path=" + event.getPath());
                    System.out.println("eventType=" + event.getType());
                    if(event.getType()==Event.EventType.NodeDataChanged) {
                        zooKeeper.getData("/watcher2", this, null);
                    }
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
        };
        zooKeeper.getData("/watcher2", watcher, null);
        Thread.sleep(50000);
        System.out.println("结束");
    }

    @Test
    public void watcherGetData4() throws KeeperException, InterruptedException {
        // 注册多个监听器对象
        zooKeeper.getData("/watcher2", new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                try {
                    System.out.println("1");
                    System.out.println("path = " + event.getPath());
                    System.out.println("eventType = " + event.getType());
                    if(event.getType() == Event.EventType.NodeDataChanged) {
                        zooKeeper.getData("/watcher2", this, null);
                    }
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
        }, null);
        zooKeeper.getData("/watcher2", new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                try {
                    System.out.println("2");
                    System.out.println("path=" + event.getPath());
                    System.out.println("eventType=" + event.getType());
                    if(event.getType()==Event.EventType.NodeDataChanged) {
                        zooKeeper.getData("/watcher2", this, null);
                    }
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
        }, null);
        Thread.sleep(50000);
        System.out.println("结束");
    }
}
```



### GetChild

```java
public class ZKWatcherGetChild {
    String IP = "192.168.60.130:2181";
    ZooKeeper zooKeeper = null;

    @Before
    public void before() throws IOException, InterruptedException {
		...
    }

    @After
    public void after() throws InterruptedException {
        zooKeeper.close();
    }

    @Test
    public void watcherGetChild1() throws KeeperException, InterruptedException {
        // arg1:节点的路径
        // arg2:使用连接对象中的watcher
        zooKeeper.getChildren("/watcher3", true);
        Thread.sleep(50000);
        System.out.println("结束");
    }

    @Test
    public void watcherGetChild2() throws KeeperException, InterruptedException {
        // arg1:节点的路径
        // arg2:自定义watcher
        zooKeeper.getChildren("/watcher3", new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println("自定义watcher");
                System.out.println("path = " + event.getPath());
                System.out.println("eventType = " + event.getType());
            }
        });
        Thread.sleep(50000);
        System.out.println("结束");
    }

    @Test
    public void watcherGetChild3() throws KeeperException, InterruptedException {
        // 一次性
        Watcher watcher = new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                try {
                    System.out.println("自定义watcher");
                    System.out.println("path = " + event.getPath());
                    System.out.println("eventType = " + event.getType());
                    if (event.getType() == Event.EventType.NodeChildrenChanged) {
                        zooKeeper.getChildren("/watcher3", this);
                    }
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
        };
        zooKeeper.getChildren("/watcher3", watcher);
        Thread.sleep(50000);
        System.out.println("结束");
    }

    @Test
    public void watcherGetChild4() throws KeeperException, InterruptedException {
        // 多个监视器对象
        zooKeeper.getChildren("/watcher3", new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                try {
                    System.out.println("1");
                    System.out.println("path = " + event.getPath());
                    System.out.println("eventType = " + event.getType());
                    if (event.getType() == Event.EventType.NodeChildrenChanged) {
                        zooKeeper.getChildren("/watcher3", this);
                    }
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
        });

        zooKeeper.getChildren("/watcher3", new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                try {
                    System.out.println("2");
                    System.out.println("path = " + event.getPath());
                    System.out.println("eventType = " + event.getType());
                    if (event.getType() == Event.EventType.NodeChildrenChanged) {
                        zooKeeper.getChildren("/watcher3", this);
                    }
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
        });
        Thread.sleep(50000);
        System.out.println("结束");
    }
}
```





