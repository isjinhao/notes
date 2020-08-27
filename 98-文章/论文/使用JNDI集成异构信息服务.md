Dirk Corissen, Piotr WVendykier, Dawid Kurzyniec, and Vaidy Sunderam  

埃默里大学

数学与计算机科学系

美国佐治亚州亚特兰大市，邮编：30322



Abstract  

The capability to announce and discover resources is a foundation for heterogenfeous computing systems. Independent projects have adopted Custom implementations of information services, which are  not interoperable and induce substantial maintenance costs. In this paper, we propose an alternative methodology. We sugest that it is possible to reuse existinig naming service deployments and combine them into complex, scalable, hierarchical, distributed federations, by using appropriate client-side integration middlemare that unifies service access and hides heterogeneity behind a common API. We investigate a JNDI-based approach, and describe in detail two newly implemented JNDI service provders, which enable unified access to 1) Jini lookup services, and 2) Harness Distributed Naming Services. We claim that these two technologies, along with others already accessible through JNDI such as e.g. DNS and LDAP, offer features suitable for use in hierarchical heterogeneous information systems.

宣布和发现资源的能力是异构计算系统的基础。独立项目采用了信息服务的自定义实现方式，这些实现方式无法互操作，并且会带来可观的维护成本。在本文中，我们提出了一种替代方法。 我们认为可以通过使用适当的客户端集成中间件来重用现存的命名服务部署，并将它们组合为复杂的，可伸缩的，分层的，分布式的联合身份，这可以统一服务访问并在通用API后面隐藏异质性。 我们研究了一种基于JNDI的方法，并详细描述了两个新实现的JNDI服务提供者，它们可统一访问 1) Jini查找服务和 2) 线束分布式命名服务。 我们声称这两种技术，以及已经可以通过JNDI访问的其他技术，例如 DNS 和LDAP提供适用于分层异构信息系统的功能。



Introduction 

Resource information, registration, and discovery services are of crucial importance in heterogeneous distributed systems, as they provide the necessary bridge between resource providers and consumers. Existing information services, targeting vastly different systems and addressing diverse requirements, vary greatly in the update and query capabilities levels of security and fault tolerance, genericity, scalability, and responsiveness. For instance, DNS provides a name resolution service which scales world-wide but is specialized, lacks strong consistency, and has limited query capabilities. These features make it suitable for managing simple textual data collections for which updates are rare and sophisticated queries are unnecessary. On the other hand, information services targeting more dynamic and diversified data sets, such as Jini (with its leasing and event notification mechanismis), are usually less scalable. Often, a hybrid approach is required to balance these conflicting objectives. Independent projects have developed their own solutions, typically involving complex, distributed, hierarchical informcation services requiring custom installation, configuration, and mainteniance, thus adding to the existing burden of IT software support. Moreover, these specific components are usually not interoperable, which can be a major obstacle to building heterogeneous systeimis. In this paper, we argue that hybrid, large-scale, distributed, hierarchical informcation services can be constructed by assembling existing, off-the-shelf, heterogeneous software components. We claim that access homogeneity can be achieved at the client-side by using an appropriate integration technology, such as Java Naming and Directory Interface (JNDI). We demonstrate how the JNDI model can be used to integrate lookup and discovery mechanisms, such as Jini, HDNS, LDAP, DNS, and others, to build complex, scalable, and powerful information services, while potentially reusing existing middleware deploymeints, thus promoting evolutionary development of the IT infrastructulre and facilitating cross-site integration.

资源信息，注册和发现服务在异构分布式系统中至关重要，因为它们提供了资源提供者和使用者之间的必要桥梁。现有的信息服务以极大不同的系统为目标并满足各种需求，在安全性和容错性，通用性，可伸缩性和响应性的更新和查询功能级别上差异很大。例如，DNS提供了一个可在全球范围内扩展但专业化的名称解析服务，缺乏强大的一致性，并且查询功能有限。这些功能使其适用于管理简单的文本数据集，这些数据集很少进行更新，而无需复杂的查询。另一方面，针对更具动态性和多样性的数据集的信息服务，例如Jini（具有其租赁和事件通知机制），通常可扩展性较差。通常，需要一种混合方法来平衡这些相互矛盾的目标。独立项目已经开发了自己的解决方案，通常涉及复杂的，分布式的，分层的信息服务，需要自定义安装，配置和维护，从而增加了IT软件支持的现有负担。此外，这些特定组件通常不可互操作，这可能是构建异构系统的主要障碍。在本文中，我们认为可以通过组装现有的，现成的，异构的软件组件来构建混合，大规模，分布式，分层的信息服务。我们声称可以通过使用适当的集成技术（例如Java命名和目录接口（JNDI））在客户端实现访问同质性。我们演示了如何使用JNDI模型来集成查找和发现机制（例如Jini，HDNS，LDAP，DNS等），以构建复杂，可扩展且功能强大的信息服务，同时潜在地重用现有的中间件部署方法，从而促进进化开发IT基础设施并促进跨站点集成。



Related Work  	

The choice of an information service implementation depends on a nulmber of factors, incluiding: expected scope (i.e. number and type of relationships between collaborating parties), application requirements (e.g. simple device sharing vs. parallel job submission), existing infrastructure and deployment environment (e.g. firewalls and administrative domains), expertise of the IT personnel, or even the personal preference. Not surprisingly then, different projects have adapted different solutioins to address their particular requirements. Below we list some of the existing grid projects togetlher with the approach they have chosen to implement service discovery and registration. Subsequently, we give examples of integration efforts that, like ours, try to tackle the limitations of these projects.

信息服务实施的选择取决于多种因素，其中包括：预期范围（即协作方之间关系的数量和类型），应用程序要求（例如，简单的设备共享与并行作业提交），现有的基础架构和部署环境（例如防火墙和管理域），IT人员的专业知识，甚至个人喜好。因此，毫不奇怪，不同的项目已经采用了不同的甜菊素来满足其特殊要求。下面我们列出了一些现有的网格项目，以及它们选择的用于实现服务发现和注册的方法。随后，我们提供了一些集成尝试的例子，这些尝试与我们一样，试图解决这些项目的局限性。

ICENI: The Imnperial College e-Science Networked Infrastructure (ICENI) is a mature, service-oriented, integrated grid middleware built around Java and Jini. Its resoulrce discovery and registration tasks are handledc mainly through Jini, augmieilted with higher level capabilities such as XPath queries and seimianitic miiatching. Though predominantly Jini based, it has been demonstrated [13] that ICENI's service-oriented architecture can also be ported to JXTA [17] and OGSA (albeit with some restrictions).

ICENI：帝国大学电子科学网络基础架构（ICENI）是围绕Java和Jini构建的成熟的，面向服务的集成网格中间件。它的资源发现和注册任务主要通过Jini处理，并具有更高的功能，如XPath查询和类似的miiatching。尽管主要基于Jini，但已证明[13] ICENI的面向服务的体系结构也可以移植到JXTA [17]和OGSA（尽管有一些限制）。

JGrid [21], JISGA [30] and ALiCE [26]: these projects are similar in that all use the Jini framework for resource information management: Resources are represenited by Jini services that are stored in the Jini lookup service (LUS). The LUS supports service interface and attribute based matching. In addition JISGA provides an OGSA-compliant interface, and JGrid has extended Jini miodel with a wide-area discovery mechanism.

JGrid [21]，JISGA [30]和ALiCE [26]：这些项目相似，因为它们都使用Jini框架进行资源信息管理：资源由Jini查找服务（LUS）中存储的Jini服务重新呈现。 LUS支持服务接口和基于属性的匹配。另外，JISGA提供了符合OGSA的接口，并且JGrid扩展了Jini miodel的广域发现机制。

Triania [25]: Created at PPARC as part of the GridOneD project the Triana aims to be a pluggable grid middleware implemented as a Grid Application Toolkit (GAT) which provides a portable set of core services. Resource registration and discovery component of the Triaina GAT provides bindiings to JXTA and OGSA (experimental) 

Triania [25]：作为GridOneD项目的一部分在PPARC上创建，Triana的目标是成为实现为可提供一组核心服务的可移植网格应用程序工具包（GAT）的可插拔网格中间件。 Triaina GAT的资源注册和发现组件提供与JXTA和OGSA（实验性）的绑定

Globe [3]: The Globe middleware platform is designed to enable the flexible development of global-scale Internet applications. It is based on a distributed shared objects model from which applicationis are built. These distributed shared objects are automatically replicated across the network and they are registered in a two-tier naming service. At the virtulal level, each object is assigned a human readable, globally unique name, stored in DNS. To maintain the binding between an object replica and its current location (IP Address), a separate, hierarchical on location service is used. Object replica servers are located using the Globe Infrastructure Directory Service which is hierarchy of LDAP servers.

Globe [3]：Globe中间件平台旨在支持灵活开发全球范围的Internet应用程序。它基于构建应用程序的分布式共享对象模型。这些分布式共享对象会在网络上自动复制，并在两层命名服务中注册。在虚拟级别上，为每个对象分配一个人类可读的全局唯一名称，该名称存储在DNS中。为了维持对象副本与其当前位置（IP地址）之间的绑定，使用了单独的分层位置服务。使用LDAP服务器的层次结构Globe Globe Directory Service定位对象副本服务器。

As is apparent in these examples, grid information services are characterized with substantial complexity and large heterogeneity. The complexity places significant burden on resource administrators, who are reponsible for configuring and maintaining the middleware. The heterogeneity, on the other hand, hampers interoperability between grid deployments, and complicates application development. We suggest that it is possible to overcome those issues via an integration middleware that would enable interoperability between service implementations and hiding service heterogeneity behind unified, semi-transparent APIs, providing uniform access to common capabilities while also permitting use of custom, service-specific functionality.

从这些示例中可以明显看出，网格信息服务的特点是具有相当大的复杂性和较大的异构性。复杂性给资源管理员带来了沉重负担，他们负责配置和维护中间件。另一方面，异构性阻碍了网格部署之间的互操作性，并使应用程序开发复杂化。我们建议，有可能通过集成中间件解决这些问题，该中间件将实现服务实现之间的互操作性，并将服务异构性隐藏在统一的半透明API之后，提供对通用功能的统一访问，同时还允许使用自定义的，特定于服务的功能。

This idea of inserting an extra layer of abstraction onto which multiple information services are mrapped is not new. Other projects that have taken up this hourglass niodel include:

插入一个额外的抽象层以映射多个信息服务的想法并不新鲜。采取了这种沙漏策略的其他项目包括：

MonALISA [14]: The goal of the MonALISA framework is to provide service information from large, distributed aiid heterogeneous systemns to a set of loosely coupled higher level services in a flexible, self describing way. These higher level services can then be queried by others and are implemented as Jini services that fully leverage the Jini framework (security, ServiceUl, leasing, ...). For non Jini clients a webservice bridge is available. MonALISA is already in a mature state of development counting a large number of features such as support for SNMP and integration with external monitoring tools like Ganglia. 

MonALISA [14]：MonALISA框架的目标是以一种灵活的自我描述方式，将服务信息从大型的分布式AI异构系统提供给一组松散耦合的更高级别的服务。然后，其他人可以查询这些更高级别的服务，并将其实现为充分利用Jini框架（安全性，ServiceUl，租赁等）的Jini服务。对于非Jini客户，可以使用Web服务桥。 MonALISA已经处于成熟的发展状态，其中包含大量功能，例如对SNMP的支持以及与外部监控工具（例如Ganglia）的集成。

Globus MDSv4 [12]: standard Web Service interfaces form the core abstraction of MDS. Globus currently uses a WSRF-based Monitoring and Discovery System (MDS) to provide information about the current availability and capability of resources. Service data is generated by Service Data Provider programs which are then aggregated into Grid Service instances, each of which provides information through Service Data Elements (SDE). A MDS Index service allows for aggregation capabilities. Data services have their own special registry called a Grid Data Service Registry (GDSR). Previous versions of Globus and MDS, which are still in active use, relied on LDAP-based registries [7, 8].

Globus MDSv4 [12]：标准Web服务接口构成了MDS的核心抽象。 Globus当前使用基于WSRF的监视和发现系统（MDS）来提供有关资源的当前可用性和功能的信息。服务数据由服务数据提供程序生成，然后被聚合到Grid Service实例中，每个实例都通过服务数据元素（SDE）提供信息。 MDS索引服务允许聚合功能。数据服务具有自己的特殊注册表，称为网格数据服务注册表（GDSR）。仍在使用中的Globus和MDS的早期版本依赖于基于LDAP的注册表[7，8]。

R-GMA [11]: The Relational Grid Monitoring Architecture provides a service for information, monitoring and logging in a heterogeneous distributed computing environment. R-GMA makes all the information appear like one large Relational Database, "the hourglass neck", that may be queried to find the information required. It consists of independent Producers which publish informatioi into R-GMA, and Consumers which subscribe. R-GMA uses SOAP messaging over https for all high-level user-to-service and service-to-service comnmuiiications. R-GMA was originally developed under the European DataGrid project buit is now part of the wider European EGEE project.

R-GMA [11]：关系网格监视体系结构为异构分布式计算环境中的信息，监视和日志记录提供服务。 R-GMA使所有信息看起来像一个庞大的关系数据库，即“沙漏脖子”，可以通过它找到所需的信息。它由将信息发布到R-GMA的独立生产者和进行订阅的消费者组成。 R-GMA使用HTTP上的SOAP消息传递进行所有高级用户到服务和服务到服务的通信。 R-GMA最初是在欧洲数据网格项目的支持下开发的，现在已成为更广泛的欧洲EGEE项目的一部分。

R-GMA [11]：关系网格监视体系结构为异构分布式计算环境中的信息，监视和日志记录提供服务。 R-GMA使所有信息看起来像一个庞大的关系数据库，即“沙漏脖子”，可以通过它找到所需的信息。它由将信息发布到R-GMA的独立生产者和进行订阅的消费者组成。 R-GMA使用HTTP上的SOAP消息传递进行所有高级用户到服务和服务到服务的通信。 R-GMA最初是在欧洲数据网格项目的支持下开发的，现在已成为更广泛的欧洲EGEE项目的一部分。

Hawkeye [2]：Hawkeye诞生于Condor项目，它是针对分布式系统的自定义监视和管理工具。 Hawkeye采用可插拔体系结构，可以轻松地插入提供额外功能（例如监视可用磁盘空间）的不同模块。尽管它对Condor保留了很大的偏见，但可以用于监视和集成不同的分布式系统。

While the above mentioned systems provide adequate levels  functionality to support their target application areas, they are not interoperable, and they impose substantial setup and maintenance efforts. In contrast, the approach exploited in this paper emphasizes reuse of existing information service deployments, and offers a possibility to aggregate different, heterogeneous, distributed information services into a single name space that is available to clients via an unified API.

尽管上面提到的系统提供了足够的功能来支持其目标应用程序区域，但它们不是可互操作的，并且会带来大量的设置和维护工作。 相比之下，本文中采用的方法强调了现有信息服务部署的重用，并提供了将不同的，异构的，分布式信息服务聚合到可通过统一API供客户端使用的单个名称空间的可能性。



JNDI  



The JNDI is an application programming interface (API) that provides naming and directory functionality to applications written using the Java programming language. It is designed to be independent of any specific directory service implementation, allowing variety of directory systems (including new, emerging, and already deployed) to be accessed in a common way. The JNDI architecture consists of a client API and a service provider interface (SPI). Java applications use  the JNDI API to access a variety of naming and directory services, while the SPI enables pluggability of directory service implementations [23]. The JNDI architecture is illustrated in Figure 1.

JNDI是一个应用程序编程接口（API），它为使用Java编程语言编写的应用程序提供命名和目录功能。 它被设计为独立于任何特定的目录服务实现，从而允许以通用方式访问各种目录系统（包括新的，正在出现的和已部署的）。 JNDI体系结构由客户端API和服务提供商接口（SPI）组成。 Java应用程序使用JNDI API来访问各种命名和目录服务，而SPI支持目录服务实现的可插入性[23]。 JNDI体系结构如图1所示。

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/881e4422-f452-416f-9c5a-2ff7c09e2e69" /></div>
Currently, JNDI is used mostly in Enterprise Java (J2EE) containers to manage information related to Enterprise Java Beans (EJB). In the grid computing context, virtually all JNDI use cases are related to accessing LDAP directories (e.g. MDS used in Globus v2) or, in some cases, COS naming servers [29]. The JNDI has also been used in agent frameworks [6]. Specialized SPIs have been developed for the smart cardbased Personal Naming and Directory Service [19] and the naming service used in the DREAM distributed services platform [22].

当前，JNDI主要用于企业Java（J2EE）容器中，以管理与企业Java Bean（EJB）有关的信息。在网格计算环境中，几乎所有JNDI用例都与访问LDAP目录（例如Globus v2中使用的MDS）或在某些情况下与COS命名服务器有关[29]。 JNDI也已用于代理框架[6]。已经针对基于智能卡的个人命名和目录服务[19]和DREAM分布式服务平台中使用的命名服务[22]开发了专用SPI。

JNDI exhibits a number of characteristics that make it suitable for heterogeneous information services. First, it features a simple and generic "lowest-common-denominator" base directory interface, so it can support a wide range of service providers. Moreover, JNDI provides uniform but flexible APIs for lookup queries and metadata management. Finally, JNDI provides support for linking multiple, heterogeneous naming services into a single, aggregated name space  (federation), with a simple URL-based naming scheme introduced to identify data entries.

JNDI具有许多特性，使其适合于异构信息服务。首先，它具有简单且通用的“最低公分母”基本目录界面，因此它可以支持广泛的服务提供商。此外，JNDI为查询查询和元数据管理提供了统一但灵活的API。最后，JNDI提供了一种支持，用于将多个异构的命名服务链接到单个聚合的名称空间（联合）中，并引入了一种基于URL的简单命名方案来标识数据条目。

Obviously, a lowest-commondenominator API unifying vastly different services would suffer from oversimplification if it was truly flat and opaque. Therefore, JNDI defines a hierarchy of interfaces, leaving the provider the choice of the supported conformance level. The API is designed to balance the genericity with flexibility. Data entries are represented as <name, object, attributes> tuples. Depending on the underlying service provider implementation, the supported "object" and "attributes" types may be arbitrarily complex; the JNDI specification allows for flexibility while recommending certain minimum conformance levels that every provider should satisfy (e.g. ability to bind any serializable object). Further, JNDI provides APIs for attribute-based queries. Although the syntax of such queries is mandated (and biased towards LDAP), extensions are possible. 

显然，统一了最低限度服务的最低公分母API如果确实是平坦且不透明的，则会过于简化。因此，JNDI定义了接口的层次结构，从而使提供者可以选择支持的一致性级别。该API旨在平衡通用性和灵活性。数据条目表示为<名称，对象，属性>元组。根据基础服务提供商的实现，所支持的“对象”和“属性”类型可能会非常复杂。 JNDI规范提供了灵活性，同时建议了每个提供程序都应满足的某些最低一致性级别（例如，绑定任何可序列化对象的能力）。此外，JNDI提供了用于基于属性的查询的API。尽管此类查询的语法是强制性的（并且偏向LDAP），但可以进行扩展。

The unavoidable tension between genericity and flexibility results in certain tradeoffs. Some specialized capabilities of backend services cannot be easily represented via the API; an example includes the leasing functionality avrailable in Jini. At the same time, JNDI does not fully hide backend service heterogeneity - for instance, clients are sometimes required to supply service-specific configuration parameters or credentials. Finally, since JNDI is a Java technology, the client libraries are not directly accessible from native languages, for which analogous APIs have not been (yet) implemented. Notwithstanding these disadvantages, we feel that they are outweighed by the advantages and can be countered. In particular, we note that access unification can often be achieved to a larger extent than immediately apparent, since certain capabilities missing at the server side can be emulated at the client side. Also, the service-specific configuration stages can usually be isolated from the application layer.

通用性和灵活性之间不可避免的张力会导致某些折衷。后端服务的某些特殊功能无法通过API轻松实现；一个示例包括Jini中可用的租赁功能。同时，JNDI不能完全隐藏后端服务的异构性-例如，有时需要客户端提供特定于服务的配置参数或凭据。最后，由于JNDI是一种Java技术，因此无法从本机语言直接访问客户端库，而本机语言尚未（尚未）实现此类库。尽管存在这些缺点，我们仍认为它们被优点所抵消，可以克服。特别是，我们注意到访问统一通常可以比立即实现的更大程度地实现，因为服务器端缺少的某些功能可以在客户端进行仿真。同样，特定于服务的配置阶段通常可以与应用程序层隔离。



HDNS  

Overview  

HDNS [27] is a fault-tolerant, persistent, distributed naming service initially developed for the Harness project [201. While developing JNDI Service Providers, a completely new version of HDNS (based on JGroups [4]) has been designed and implemented. HDNS establishes a group of naming service nodes which maintain consistent replicas of the registration data. Read requests can be handled entirely by any of the nodes, which facilitates load balancing and reduces access latencies. Write requests, in turn, are propagated to each member of the group, enforcing consistency of the replicas. Each node maintains persistent view of the registration data on a local disk, synchronized in fixed time intervals and upon process exit. The service can thus recover the state after a complete shutdown/restart. To support recovery from partial failures, mechanisms are provided that enable individual crashed/restarted nodes to re-join the group. Furthermore, the service is capable of recovering from network partitions and re-synchronizing the distributed state. These characteristics make HDNS an adequate choice in situations where read requests are predominant but fast update propagation and high failure resilience is required.

HDNS [27]是最初为线束项目[201]开发的容错，持久，分布式命名服务。在开发JNDI服务提供商时，已经设计并实现了一个全新的HDNS版本（基于JGroups [4]）。 HDNS建立了一组命名服务节点，这些节点维护注册数据的一致副本。读取请求可以完全由任何节点处理，这有助于负载平衡并减少访问延迟。依次将写请求传播到组中的每个成员，以确保副本的一致性。每个节点在本地磁盘上维护注册数据的持久视图，并以固定的时间间隔和进程退出时进行同步。因此，服务可以在完全关闭/重新启动后恢复状态。为了支持从部分故障中恢复，提供了使各个崩溃/重新启动的节点重新加入组的机制。此外，该服务能够从网络分区中恢复并重新同步分布式状态。这些特征使HDNS在主要读取请求但要求快速更新传播和高故障恢复能力的情况下成为适当的选择。

JGroups  

JGroups is a toolkit for reliable multicast group communication. It can  be used to create groups of processes whose members can send messages to each other. Members can join and leave the group at any time; the toolkit detects membership changes and propagates them to participants. The most powerful feature of JGroups is a configurable protocol stack, allowing to defer quality-of-service decisions regarding fault tolerance and scalability until run time. For instance, the Virtual Synchrony protocol suite guarantees an atomic broadcast and delivery. However, it comes at the cost of scalability - the entire group is only as fast as its slowest member. An alternative protocol suite uses Bimodal Multicast [5], which improves scalability, for the price of probabilistic message delivery reliability. The latter suite was chosen as the default in HDNS.

JGroups是用于可靠的多播组通信的工具包。 它可用于创建进程组，其成员可以相互发送消息。 成员可以随时加入和退出小组； 该工具包将检测成员资格更改并将其传播给参与者。 JGroups最强大的功能是可配置的协议栈，可以将有关容错和可伸缩性的服务质量决策推迟到运行时为止。 例如，虚拟同步协议套件可保证原子广播和传递。 但是，这是以可伸缩性为代价的-整个组的速度与其最慢的成员一样快。 另一种协议套件使用 Bimodal Multicast[5]，它以概率消息传递可靠性为代价，提高了可伸缩性。 后一种套件被选为HDNS中的默认套件。

Design choices  

HDNS has been implemented as a relatively thin software library, based on existing, well-established underlying technologies - JGroups as a communication substrate and H2O [18, 24, 1] as a hosting environment. Consequently, HDNS exhibits many advanced features inherited from mentioned projects. Owing to dynamic deployment features of H20, HDNS service can be dynamically deployed on participating nodes from a remote network repository, which is particularly useful during the update or maintenance process. Furthermore, H20 provides HDNS with security infrastructure, allowing to control access via user-defined security policies and featuring flexible authentication technologies. Finally, HDNS uses distributed event notification mechanism offered by H20 to implement the JNDI event notification functionality. Similarly, distributed communication and synchronization of replicas is achieved by using appropriate mechanisms of the JGroups toolkit. However, in order to fully support recovery from network partitioning, we needed to implement an additional protocol for the JGroups protocol stack. After a transient network partition, the PRIMARY_PARTITION protocol resolves state conflicts by uniquely selecting the partition deemed to have the valid state, and forcing other partitions to re-synchronize.

HDNS已被实现为一个相对较薄的软件库，它基于现有的，公认的基础技术-作为通信基础的JGroups和作为托管环境的H2O [18，24，1]。因此，HDNS展现了从上述项目继承的许多高级功能。由于H20具有动态部署功能，因此HDNS服务可以从远程网络存储库动态部署在参与的节点上，这在更新或维护过程中特别有用。此外，H20为HDNS提供了安全基础架构，从而允许通过用户定义的安全策略来控制访问并具有灵活的身份验证技术。最后，HDNS使用H20提供的分布式事件通知机制来实现JNDI事件通知功能。同样，通过使用JGroups工具包的适当机制，可以实现副本的分布式通信和同步。但是，为了完全支持从网络分区中进行恢复，我们需要为JGroups协议栈实现附加协议。在临时网络分区之后，PRIMARY_PARTITION协议通过唯一地选择被认为具有有效状态的分区并强制其他分区重新同步来解决状态冲突。



JNDI Service Provilders  

The pluggable architecture and genericity of JNDI, coupled with logical and well-documented APIs, enables relatively easy implementation of custom service providers. This section contains a description of two new service providers: Jini-based and HDNS-based, the choice of which was motivated in part by their potential sefulness in building hierarchical information systems。

JNDI的可插入体系结构和通用性，再加上逻辑性良好且记录良好的API，可以使定制服务提供商的实现相对容易。 本节描述了两个新的服务提供商：基于Jini和基于HDNS的服务提供商，其选择的部分原因在于它们在构建分层信息系统中的潜在自觉性

The Jini Provider  

Jini [16, 9, 28] is a technology developed by Sun Microsystems, providing an environment for building scalable, self-healing and flexible distributed resource sharing systems. Jini enables Java Dynamic Networking, where clients bind dynamically to any network service that is capable of providing needed functionality, regardless of  the name, location, wire protocol, or other implementation details of the particular service instance [9].

Jini [16，9，28]是由Sun Microsystems开发的一项技术，为构建可伸缩，自我修复和灵活的分布式资源共享系统提供了环境。 Jini启用Java动态网络，其中客户端可以动态绑定到能够提供所需功能的任何网络服务，而无需考虑特定服务实例的名称，位置，有线协议或其他实现细节[9]。

Mapping of Jini functionality onto the JNDI abstractions has proven to be a non-trivial task, due to differences in philosophy between the two technologies. In particular, problems arise due to the narrower focus of Jini which is specifically designed to manage services, characterized by capabilities expressed by Java remote interfaces, and not as a general purpose information storage facility. Below we list specific encountered problems and their solutions.

由于两种技术之间的理念差异，将Jini功能映射到JNDI抽象已被证明是一项艰巨的任务。尤其是，由于Jini的关注范围较窄，因此出现了问题，Jini专为管理服务而设计，以Java远程接口表示的功能为特征，而不是作为通用信息存储工具。下面我们列出了遇到的特定问题及其解决方案。

State and Object Factories. To be able to store generic name-value mappings in the Jini registry, a translation mechanism is needed to convert them into fake Jini service stubs upon registration, and back to the original form upon retrieval. We have accomplished this by using the JNDI abstractions of object and state factories, which perform the necessary translations automatically when the appropriate API methods are invoked.

状态和对象工厂。为了能够将通用名称/值映射存储在Jini注册表中，需要一种转换机制，以便在注册时将其转换为伪造的Jini服务存根，并在检索时将其转换回原始形式。我们通过使用对象工厂和状态工厂的JNDI抽象来实现此目的，当调用适当的API方法时，它们将自动执行必要的转换。

Handling leases. To avoid stale references, Jini adopts a notion of leasing [15]. Validity of a data entry in a naming service has to be continuously renewed, typically by the service itself, or else it expires and the entry is removed. No such abstraction exists in the JNDI API, which does not specify any explicit data expiration policy. Hence, Jini leases cannot be easily passed upward to the application layer  through the JNDI API. Therefore, we decided to handle Jini leases entirely in the service provider implementation layer: the provider automatically renews leases of all entries that it has previously bound,  until they are explicitly removed, or until the Java VM exits.

处理租赁。为了避免陈旧的引用，Jini采取了租赁的概念[15]。命名服务中的数据条目的有效性通常必须由服务本身来连续更新，否则它将过期并且该条目将被删除。 JNDI API中不存在此类抽象，该API没有指定任何显式的数据过期策略。因此，Jini租约不能轻易通过JNDI API向上传递到应用程序层。因此，我们决定完全在服务提供者实现层中处理Jini租约：提供者自动续订以前绑定的所有条目的租约，直到明确删除它们，或者直到Java VM退出为止。

Atomicity and consistency. One of the biggest and most unexpected Jini-JNDI mapping issues emerges from the implementation of a bind primitive. The JNDI bind has atomic semnantics: the operation should  succeed if there was no entry bound to the specified name, and otherwise fail throwing an exception. Unfortunately, the Jini LUS does not provide any such primitive that couild be used to implement this functionality strictly; aiming at achieving idempotency, Jini registration methods always overwrite the previous valune. Hence, in order to ensure atomic consistency in concurrent setups, resort to distributed locking was needed. This forced us to adopt Eisenberg and McGuire's algorithim [10], which depends only on the basic read and write primitives, but which is rather costly: it takes 3 reads and 5 writes to enter and leave a critical section in the uncontended case. This means that our strictly compliant implenmentation of JNDI bind on top of Jini induces at least an eight-fold increase in latency in comparison to the basic Jini primitive, which may or may not be a problem, depeniding on whether the application is latency-bound. We note however, that in many applications, binding/ unbinding of a given named entry is performed only by a single writer (or the "owner"), in which case the semantics of bind can be safely relaxed. We thus provide applications with an option to disable the strict semantics, removing the performance penalty by sacrificing the atomicity.

原子性和一致性。最大和最意外的Jini-JNDI映射问题之一来自绑定原语的实现。 JNDI绑定具有原子象征意义：如果没有绑定到指定名称的条目，则该操作应成功，否则将引发异常。不幸的是，Jini LUS没有提供任何可用于严格实现此功能的原语；为了实现幂等，Jini注册方法始终会覆盖以前的域。因此，为了确保并发设置中的原子一致性，需要采用分布式锁定。这迫使我们采用Eisenberg和McGuire的算法[10]，该算法仅依赖于基本的读写基元，但是代价却很高：在无人参与的情况下，需要3次读取和5次写入才能进入和离开关键部分。这意味着我们在Jini上严格遵照的JNDI绑定实现比基本Jini原语至少引起了八倍的延迟增加，这可能会或可能不会成为问题，这取决于应用程序是否受延迟限制。但是，我们注意到，在许多应用程序中，给定命名条目的绑定/解除绑定仅由单个编写者（或“所有者”）执行，在这种情况下，绑定的语义可以安全地放宽。因此，我们为应用程序提供了禁用严格语义的选项，并通过牺牲原子性来消除性能损失。



HDNS Provider  

The control over the source code of HDNS allowed us to avoid certain problems encountered in the context of Jini. HDNS was designed in a way that mapping through JNDI was simple. As a result, a distributed locking algorithm was not needed to implement an atomic bind for HDNS. In fact, all methods from JNDI DirContext interface are atomic in the HDNS service provider. Besides this difference, there is a close affinity between two described providers. HDNS, analogous to Jini utilizes the concept of object and state factories, both of the providers also have a very similar mechanism for handling leases.

对HDNS源代码的控制使我们能够避免在Jini上下文中遇到的某些问题。 HDNS的设计方式使得通过JNDI进行映射非常简单。 结果，不需要分布式锁定算法即可为HDNS实现原子绑定。 实际上，JNDI DirContext接口中的所有方法在HDNS服务提供商中都是原子的。 除了这种差异之外，两个描述的提供者之间还有紧密的联系。 HDNS类似于Jini，它利用对象工厂和状态工厂的概念，这两个提供商都具有非常相似的机制来处理租赁。



Federation

Federation is the process of "hooking" together multiple, distributed, and possibly heterogeneous naming systems so that the composite behaves as a single, possibly hierarchical, aggregate naming service. In JNDI, resources in a naming service federation are identified by composite URL names, which can span multiple substrate naming services [23]. For instance, the URL:

ldap://host.domain/n=jiniServer/jxtaGroup/myObject

links together LDAP, Jini and JXTA services. From the end-user's perspective, accessing an object through federation is fully transparent, and reduces to passing the composite URL to the appropriate API function:

Object val = new InitialDirContexto.lookup("ldap://host/n=jiniServer/jxtaGroup/name");

Behind the scenes, JNDI parses the URL, recognizes it as an LDAP reference, and passes it to the LDAP service provider. That provider, in turn, performs lookup on the string "n-jiniServer", and retrieves a value that is, in this case, interpreted as a reference to the Jini service previously bound to the LDAP service. The request is thus further propagated to the Jini SPI which performs a lookup on the "jxtaGroup"  segment, and identifies it as a reference to the JXTA namiing service.
Finally, the request is delegated to the JXTA service provider, which retrieves the target object data bound to "name".

联合是将多个分布式的，可能是异构的命名系统“挂钩”在一起的过程，以使组合物表现为单个的，可能是分层的聚合命名服务。在JNDI中，命名服务联盟中的资源由复合URL名称标识，该URL可以跨越多个基础命名服务[23]。例如，URL：

ldap：//host.domain/n=jiniServer/jxtaGroup/myObject

将LDAP，Jini和JXTA服务链接在一起。从最终用户的角度来看，通过联合访问对象是完全透明的，并且简化为将复合URL传递给适当的API函数：

对象val = new InitialDirContexto.lookup（“ ldap：// host / n = jiniServer / jxtaGroup / name”）;

在幕后，JNDI解析URL，将其识别为LDAP参考，然后将其传递给LDAP服务提供商。该提供者依次对字符串“ n-jiniServer”执行查找，并检索一个值，在这种情况下，该值被解释为对先前绑定到LDAP服务的Jini服务的引用。因此，该请求进一步传播到Jini SPI，后者在“ jxtaGroup”段上执行查找，并将其标识为对JXTA命名服务的引用。
最后，将请求委托给JXTA服务提供者，后者检索绑定到“名称”的目标对象数据。

From the API perspective, linking naming services into federation is straightforward, and reduces to binding the context interface of one naming service to another naming service:

DirContext intCxt = new InitialDirContext();
DirContext jiniCxt = initCxt.lookup("jini://host1");
DirContext hdnsCtxt = initCxt.lookup("hdns://host2");
hdnsCxt.bind('jiniCxt", jiniCxt);
// now, the URL: hdns://host2/jiniCxt refers
// to the embedded Jini directory

We have implemented the necessarv programmatic support for federations in both providers described in this paper, allowing them to be federated with information services such as DNS, LDAP, or a local filesystem storage, for which the appropriate JNDI providers already exist.

从API的角度来看，将命名服务链接到联盟很简单，并且简化为将一个命名服务的上下文接口绑定到另一个命名服务：

DirContext intCxt = new InitialDirContext();
DirContext jiniCxt = initCxt.lookup(“jini://host1”);
DirContext hdnsCtxt = initCxt.lookup(“ hdns://host2”);
hdnsCxt.bind()'jiniCxt“，jiniCxt);
//now，the URL: hdns://host2/jiniCxt refers to the embedded Jini directory

我们已经在本文描述的两个提供程序中实现了对联合程序的必要程序支持，从而使它们可以与信息服务（例如DNS，LDAP或本地文件系统存储）进行联合，而适当的JNDI提供程序已经存在。

In a wide-scale, hierarchical, distributed information service, the requests originate at the root naming service and are propagated downwards, to be eventually handled by the "leaf" information services. The root namnig service thus receives a massive load of requests, necessitating extremely scalable approaches, with DNS being a good candidate. On the other hand, the "leaf" naming services (e.g. LDAP servers or Jini registries at the department level) can be expected to encounter frequent updates and computationally intensive queries, and require rapid state propagation and event notifications. Suitable technologies for both those extreme cases already exist. The biggest research challenge is related to the intermediate layer, since it has to balance both: (1) distribution and high scalability requirements, to ensure high system throughput and minimize latencies by matching requesters to local nodes, and (2) rapid propagation of updates, with the capability to handle moderate to high update frequencies. We argue that HDNS can be a viable technology to satisfy these requirements. On one hand, state replication allows it to achieve high scalability, whereas its error recovery mechanisms ensure high failure resilience. On the other hand, its state synchronization methodology ensures fast update propagation with configurable consistency levels (examples including bimodal multicast or virtual synchrony).

在大规模，分层的分布式信息服务中，请求起源于根命名服务并向下传播，最终由“叶”信息服务处理。因此，根namnig服务会收到大量请求，这需要极其可扩展的方法，而DNS是很好的选择。另一方面，“叶子”命名服务（例如部门级的LDAP服务器或Jini注册表）可能会遇到频繁的更新和计算量大的查询，并且需要快速的状态传播和事件通知。对于这两种极端情况，已经存在合适的技术。最大的研究挑战与中间层有关，因为它必须兼顾以下两个方面：（1）分发和高可伸缩性要求，以确保高系统吞吐量并通过将请求者与本地节点匹配来最小化延迟，以及（2）快速传播更新，具有处理中到高更新频率的能力。我们认为HDNS可以成为满足这些要求的可行技术。一方面，状态复制允许它实现高可伸缩性，而其错误恢复机制则确保了高故障恢复能力。另一方面，其状态同步方法可确保以可配置的一致性级别（包括双峰多播或虚拟同步的示例）进行快速更新传播。

We thus envision the following, JNDI-based methodology for composing individual, department-level, existing and operational information services, into larger, aggregated name spaces. We propose that a collection of HDNS nodes is deployed into the distribuited system, so that a HDNS node can be found in the proximity of an large group of users. Additional nodes can be deployed dynamically at a later stage as well, while the system is already in operation. The replicated information shared by all HDNS nodes is the set of references to all department-level naming services (e.g. LDAP servers or Jini registries) present in the entire composite system. Since deployment or discontinuation of an informatioin service is a rare occurrence, we expect relatively small update frequency at the HDNS level, unless the managed network is very substaIntial in size. Finally, in order to hide the aspects of distribution, we propose to anchor the federated naming system in DNS, so that a common, well-known service name is resolved to a nearest HDNS node. For instance, when querying the status of an object referred to by the URL "dns://global/emory/niatlics/del/mokey", JNDI client would contact DNS to find the address of a nearest HDNS node belonging to the 'global" federation, then it would use HDNS to query for the address of the "emory/mathics/dcl" LDAP server, and finally, it would issue the "mokey" object query to that LDAP server. In the next section, we present our preliminary experiments and small-scale  stress tests, aiming to evaluate performance of HDNS and Jini providers alongside DNS and LDAP, and to assess viability of the proposed federation sceniario.

因此，我们设想了以下基于JNDI的方法，用于将各个部门级的现有和运营信息服务组合到较大的聚合名称空间中。我们建议将HDNS节点的集合部署到分布式系统中，以便可以在大量用户的附近找到HDNS节点。当系统已经运行时，也可以在以后阶段动态部署其他节点。所有HDNS节点共享的复制信息是对整个组合系统中存在的所有部门级命名服务（例如LDAP服务器或Jini注册表）的引用集。由于信息服务的部署或中断很少发生，因此，除非托管网络的规模非常小，否则我们期望HDNS级别的更新频率相对较小。最后，为了隐藏分发的各个方面，我们建议将联合命名系统锚定在DNS中，以便将通用的知名服务名称解析到最近的HDNS节点。例如，当查询URL“ dns：// global / emory / niatlics / del / mokey”所引用的对象的状态时，JNDI客户端将与DNS联系以查找属于“全局”的最近的HDNS节点的地址。联盟，那么它将使用HDNS查询LDAP服务器的“ emory / mathics / dcl”地址，最后，它将向该LDAP服务器发出“ mokey”对象查询。在下一部分中，我们介绍初步实验和小规模压力测试，旨在评估HDNS和Jini提供程序以及DNS和LDAP的性能，并评估所提议的联盟计划的可行性。



Experimental evaluation  

While adding an additional abstraction or adaptation layer may be elegant and appropriate from the design point of view, care must be taken that the performrance penalty is not too large. To quantify the overhead introduced by the JNDI provider layer, we have run a preliminary series of benchmarks on a local Gigabit Ethernet network. The Jini LUS service, used in the benchmarks, has been running on a Pentihrm 4 2.4 GHz machine with Mandriva Linux 2005 and 1 GB of RAM. Similarly, the HDNS service has been installed on two identical dedicated machines. In addition to the Jini and HDNS tests, we have run the benchmarks for the DNS arid LDAP JNDI providers, using the Bind and OpenLDAP servers installed on similar dedicated machines in the same LAN. In the throughput experiments, a single client machine (2.6 GHz Intel Celeron with 1 GB RAM, running Microsoft Windows XP) issues a series of requests from an increasing number of client threads (between 1 and 100). Each client thread issues consecutive requests (lookup requests for the read test and rebind requests for the write test) with 50 ms pauses between requests (i.e. with the freqtiency of up to 20 Hz). We measured the ability of the service to withstand the increasing load as a number of requests per second that have been successfully handled. In the ideal case (if the request processing time is negligible), the request frequency is 20 Hz per thread, and the number of comlpleted operations per second is 20 times the numnber of client threads.

从设计的角度来看，添加一个额外的抽象层或适应层可能是优雅且合适的，但必须注意，性能损失不会太大。为了量化JNDI提供程序层引入的开销，我们已经在本地千兆以太网上运行了一系列基准测试。基准测试中使用的Jini LUS服务已在配备Mandriva Linux 2005和1 GB RAM的Pentihrm 4 2.4 GHz计算机上运行。同样，HDNS服务已安装在两台相同的专用计算机上。除了Jini和HDNS测试之外，我们还使用安装在同一LAN上类似专用计算机上的Bind和OpenLDAP服务器，为DNS和LDAP JNDI提供程序运行了基准测试。在吞吐量实验中，一台客户端计算机（具有1 GB RAM的2.6 GHz Intel Celeron，运行Microsoft Windows XP）从数量越来越多的客户端线程（介于1到100之间）发出一系列请求。每个客户端线程发出连续的请求（针对读取测试的查找请求和针对写入测试的重新绑定请求），每个请求之间的间隔为50毫秒（即频率高达20 Hz）。我们以每秒成功处理的请求数衡量了服务承受不断增加的负载的能力。在理想情况下（如果请求处理时间可以忽略），每个线程的请求频率为20 Hz，每秒完成的操作数为客户端线程数的20倍。

Figure 2 shows "lookup" results for standalone Jini, and for JNDI-Jini provider with (1) strict and (2) relaxed bind semantics. It can be seen that the standalone Jini can handle up to about 400 requests per second, and its throughput starts decreasing afterwards. The serialization layer introduced by the JNDI-Jini provider reduces the performance by about 25%, yielding the peek throughput of about 300 requests per second. The "strict" versus "relaxed" semantics, which apply only to write operations, did not affect the "lookup" results. The output for "rebind" is shown in Figure 3. The peek Jini write throughput has been measured at about 140 operations/s. The Jini-JNDI provider throughput approaches 80 operations/s for the relaxed semantics, and 20 operations/s for the strict semantics. This 7-fold decrease, caused by the need for extra comnunication, indicates that strict bind semantics should be disabled whenever possible, and otherwise a proxybased solutions should be adapted so that the necessary locking is performed locally (near the Jini LUS, e.g. on the same host), exposing the atomic interface to the client.

图2显示了针对独立Jini以及具有（1）严格和（2）宽松绑定语义的JNDI-Jini提供程序的“查找”结果。可以看出，独立的Jini每秒最多可以处理400个请求，此后吞吐量开始下降。 JNDI-Jini提供程序引入的序列化层将性能降低了约25％，从而产生了每秒约300个请求的透视吞吐量。仅适用于写操作的“严格”与“宽松”语义不影响“查找”结果。 “ rebind”的输出如图3所示。偷看的Jini写吞吐量已测量为约140次操作/秒。 Jini-JNDI提供程序吞吐量对于宽松的语义接近80个操作/秒，对于严格的语义接近20个操作/秒。由于需要额外的通信而导致的这种7倍的减少表明，应尽可能禁用严格的绑定语义，否则应修改基于代理的解决方案，以便在本地执行必要的锁定（例如，在Jini LUS附近）同一主机），将原子接口暴露给客户端。

Figure 4 presents 'lookup" tests for HDNS and JNDI HDNS provider. In this test, all lookup requests are sent to the same HDNS node, so the results show a per-node throughput. As it can be seen, HDNS demonstrates excellent scalability; we have not been able to identify the peak throughput as it exceeds 1800 read operations per second. The HDNS JNDI provider layer does not introduce a noticeable overhead (due to the fact that HDNS server has been implemented with the JNDI support in mind). The 'rebind" results for HDNS are shown in Figure 5. Write operations in HDNS impose a substantial overhead, due to the necessity to propagate the distributed state to all HDNS nodes. In our experiments, we have observed the peak write throughput at about 200 operations/s. The tests exposed an issue with the HDNS implementation: the service response time does not degrade gracefully as the number of clients increases; we observe a rapid throughput decline (instead of levelling off) for number of clients exceeding 20. We have traced the problem to the buffer management in the underlying JGroups implementation; flooding the server with requests cause internal JGroups message queues to grow without bounds eventually causing memory exhaustion and server crash. We are currently investigating possible approaches to address this issue.

图4展示了针对HDNS和JNDI HDNS提供程序的“查找”测试，在该测试中，所有查找请求均发送到同一HDNS节点，因此结果显示了每个节点的吞吐量。我们无法确定峰值吞吐量，因为它每秒超过1800次读取操作。HDNSJNDI提供程序层没有引入明显的开销（因为考虑到HDNS服务器是在考虑到JNDI支持的情况下实现的）。 HDNS的“重新绑定”结果如图5所示。由于必须将分布式状态传播到所有HDNS节点，因此HDNS中的写入操作会产生大量开销。在我们的实验中，我们观察到峰值写入吞吐量约为200次操作/秒。这些测试暴露了HDNS实施的问题：服务响应时间不会随着客户端数量的增加而适当减少；对于超过20个的客户端，我们观察到吞吐量快速下降（而不是趋于平稳）。我们已将问题追溯到底层JGroups实现中的缓冲区管理；向服务器发送大量请求，导致内部JGroups消息队列不断增长，最终导致内存耗尽和服务器崩溃。我们目前正在研究解决此问题的可能方法。

Figures 6 and 7 show throughput of Bind (DNS) and OpenLDAP services, respectively, when accessed via standard JNDI providers. As could be expected, DNS exhibits excellent scalability, with peak throughput per node exceeding 1800 lookup operations/s. Similarly, very good write throughput has been observed for the LDAP server. Surprisingly, the read throughput of OpenLDAP plateaus at about 800 operations per second, leaving server resources (CPU, network, mem[eLory) Unsaturated. We are currently investigating causes of this phenomenon; we suspect that the anomaly is due to some automatic slowdown mechanism, such as a countermeasure against Denial-of-Service attacks. 

图6和7分别显示了通过标准JNDI提供程序访问Bind（DNS）和OpenLDAP服务时的吞吐量。可以预料，DNS具有出色的可伸缩性，每个节点的峰值吞吐量超过1800个查找操作/秒。同样，对于LDAP服务器，观察到非常好的写入吞吐量。令人惊讶的是，OpenLDAP平台的读取吞吐量约为每秒800次操作，从而使服务器资源（CPU，网络，内存）不饱和。我们目前正在调查这种现象的原因；我们怀疑该异常是由于某种自动减速机制引起的，例如针对“拒绝服务”攻击的对策。

These preliminary experiments lead us to conclude that the proposed HDNS service is a viable techinology for implemnenting very scalable distributed lookup services, which - unlike DNS - support remote updates and ensure their rapid propagation. However, the implementation needs improvement to be able to gracefully handle update overload. Also, we note that OpenLDAP service exhibits excellent responsiveness to update requests, and is therefore feasible for management of dynamic data sets. The scalability of Jini scores somewhat lower; yet, in the environments with Jini already deployed, the JNDI interface can provide standardized access to it for the cost of some extra overhead.

这些初步的实验使我们得出结论，即所提议的HDNS服务是实现非常可扩展的分布式查找服务的可行技术，与DNS不同，该服务支持远程更新并确保其快速传播。但是，实现需要改进，以便能够正常处理更新过载。另外，我们注意到OpenLDAP服务对更新请求具有出色的响应能力，因此对于动态数据集的管理是可行的。 Jini的可伸缩性得分较低；但是，在已经部署了Jini的环境中，JNDI接口可以提供对其的标准化访问，但需要付出一些额外的开销。
我们的初步测试表明，当所讨论的JNDI提供程序组合到联合名称空间中时，它们的个人性能特征得以保留。因此，鉴于其出色的查找可扩展性和快速的更新传播，我们认为HDNS是在大规模联合信息服务中实现中间层的有前途的技术。需要进一步的研究和实验来验证所提出的方法在大规模网络环境中的可行性。

Our preliminiary tests indicate that the individual performance characteristics of the discussed JNDI providers are preserved when they are combined into a federated name space. We thus believe that HDNS is a promising technology for implementing intermediate layers  in the large scale federated information service, given its excellent lookup scalability and fast update propagation. Further research anid experimnentation is required to validate feasibility of the proposed approach in large-scale network settings.

我们的初步测试表明，当所讨论的JNDI提供程序组合到联合名称空间中时，它们的各个性能特征得以保留。 因此，我们认为HDNS具有出色的查找可扩展性和快速更新传播能力，因此它是在大规模联合信息服务中实现中间层的有前途的技术。 需要进一步的研究和实验来验证所提出的方法在大规模网络环境中的可行性。

Conclusions  

In this paper, we suggested a JNDI-based methodology of merging individual niamning services into aggregate, hierarchical information systems. We described two new implementations of JNDI service providers, enabling seamless access to Jini lookup services, and to the fault tolerant Harness Distributed Naming Service. Extrapolating from these two cases, we argue that unification of the naming service    access methodology is indeed possible and beneficial, despite the differences in design models and implemenitation strategies. We present our preliminary experimenital results that sggest that HDNS may be a viable candidate for organizing isolated information service deployments into a DNS-anchored, aggregated name space. We believe that the approach enabling API standardization and seamless service federations, can result in a substantial decrease of deploymnent and maintenance costs, and in improved scalability and interoperability, without a significant sacrifice in functionality.

在本文中，我们提出了一种基于JNDI的方法，将单个漫游服务合并到聚合的分层信息系统中。我们描述了JNDI服务提供程序的两个新实现，它们实现了对Jini查找服务和容错的线束分布式命名服务的无缝访问。从这两种情况推断，尽管设计模型和实现策略存在差异，但统一命名服务访问方法确实是可能且有益的。我们提供的初步实验结果表明，HDNS可能是将孤立的信息服务部署组织到DNS锚定的聚合名称空间中的可行选择。我们相信，实现API标准化和无缝服务联盟的方法可以大大降低部署和维护成本，并改善可伸缩性和互操作性，而不会在功能上做出重大牺牲。

Even though our initial results are promising, further investigation and larger scale experiments are necessary to validate feasibility of the proposed approach in real-world settings Building a large scale information service federation, aind its thorough experimeintal evaluation, will therefore be the focus of our future work.

尽管我们的初步结果令人鼓舞，但仍需要进行进一步的研究和更大规模的实验，以验证所提出方法在现实环境中的可行性。建立大规模的信息服务联盟，并对其进行全面的实验评估，因此将成为我们未来的重点工作。