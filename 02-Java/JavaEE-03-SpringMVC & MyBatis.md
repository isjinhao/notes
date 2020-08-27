## 跨域问题和CORS

### 同源策略

如果两个页面的**协议**，**端口（如果有指定）**和**主机**都相同，则两个页面具有相同的**源**。下表给出了同源检测的示例，相对：`http://store.company.com/dir/page.html`:

| URL                                               | 结果 |              原因              |
| :------------------------------------------------ | :--- | :----------------------------: |
| `http://store.company.com/dir2/other.html`        | 成功 |          只有路径不同          |
| `http://store.company.com/dir/inner/another.html` | 成功 |          只有路径不同          |
| `https://store.company.com/secure.html`           | 失败 |    不同协议 ( https和http )    |
| `http://store.company.com:81/dir/etc.html`        | 失败 | 不同端口 ( http:// 80是默认的) |
| `http://news.company.com/dir/other.html`          | 失败 |    不同域名 ( news和store )    |



### 没有同源策略限制的两个危险场景

#### 没有同源策略限制的接口请求

有一个小小的东西叫cookie大家应该知道，一般用来处理登录等场景，目的是让服务端知道谁发出的这次请求。如果你请求了接口进行登录，服务端验证通过后会在响应头加入Set-Cookie字段，然后下次再发请求的时候，浏览器会自动将cookie附加在HTTP请求的头字段Cookie中，服务端就能知道这个用户已经登录过了。知道这个之后，我们来看场景：

- 你准备去清空你的购物车，于是打开了买买买网站`www.maimaimai.com`，然后登录成功，一看，购物车东西这么少，不行，还得买多点。
- 你在看有什么东西买的过程中，你的好基友发给你一个链接`www.nidongde.com`，一脸yin笑地跟你说：“你懂的”，你毫不犹豫打开了。
- 你饶有兴致地浏览着`www.nidongde.com`，谁知这个网站暗地里做了些不可描述的事情！由于没有同源策略的限制，它向`www.maimaimai.com`发起了请求！聪明的你一定想到一句话“服务端验证通过后会在响应头加入Set-Cookie字段，然后下次再发请求的时候，浏览器会自动将cookie附加在HTTP请求的头字段Cookie中”，这样一来，这个不法网站就相当于登录了你的账号，可以为所欲为了！当然，这只是对cookie的一方面限制，想完整的保证cookie的安全性还必须信息安全同学的头发。

#### 没有同源策略限制的Dom查询

- 有一天你刚睡醒，收到一封邮件，说是你的银行账号有风险，赶紧点进`www.yinghang.com`改密码。你吓尿了，赶紧点进去，还是熟悉的银行登录界面，你果断输入你的账号密码，登录进去看看钱有没有少了。
- 睡眼朦胧的你没看清楚，平时访问的银行网站是`www.yinhang.com`，而现在访问的是`www.yinghang.com`，这个钓鱼网站做了什么呢？

```js
// HTML
<iframe name="yinhang" src="www.yinhang.com"></iframe>
// JS
// 由于没有同源策略的限制，钓鱼网站可以直接拿到别的网站的Dom
const iframe = window.frames['yinhang']
const node = iframe.document.getElementById('你输入账号密码的Input')
console.log(`拿到了这个${node}，我还拿不到你刚刚输入的账号密码吗`)
```

<div align="center"><img width="80%" src="http://q0l9qvfyx.bkt.clouddn.com/6189bcc1-8aa4-4d12-921f-93af5a792725"></div>

但是对于如下的请求，会被同源策略放行：

```js
<script src="//static.store.com/jquery.js ></script>
```

这样的话可以保证一些插件能从指定的地址下载放到我们自己的页面中。其实这种“嵌入式”的跨域加载资源的方式还有`<img>`、`<link>`等，相当于我们浏览器发起了一次GET请求，取到相关资源，然后放到本地而已。



### CORS解决跨域问题

CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）。看名字就知道这是处理跨域问题的标准做法。CORS有两种请求，简单请求和非简单请求。

#### 简单请求

只要同时满足以下两大条件，就属于简单请求。

1. 请求方法是以下三种方法之一：
   1. HEAD
   2. GET
   3. POST
2. HTTP的头信息不超出以下几种字段：
   1. Accept
   2. Accept-Language
   3. Content-Language
   4. Last-Event-ID
   5. Content-Type：只限于三个值`application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`

当浏览器发现发起的ajax请求是简单请求时，会在请求头中携带一个字段：`Origin`。

<div align="center"><img width="100%" src="http://q0l9qvfyx.bkt.clouddn.com/73c0a15d-c3c4-4caf-9d06-9e01a9c6bab7"></div>
Origin中会指出当前请求属于哪个域（协议+域名+端口）。服务会根据这个值决定是否允许其跨域。如果服务器允许跨域，需要在返回的响应头中携带下面信息：

```http
Access-Control-Allow-Origin: http://manage.leyou.com
Access-Control-Allow-Credentials: true
Content-Type: text/html; charset=utf-8
```

- Access-Control-Allow-Origin：可接受的域，是一个具体域名或者*（代表任意域名）
- Access-Control-Allow-Credentials：是否允许携带cookie，默认情况下，cors不会携带cookie，除非这个值是true。

服务器想要操作当前页面的cookie，需要满足3个条件：

- 服务的响应头中需要携带Access-Control-Allow-Credentials并且为true。
- 浏览器发起ajax请求需要指定字段withCredentials 为true
- 响应头中的Access-Control-Allow-Origin一定不能为*，必须是指定的域名



#### 特殊请求

特殊请求会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。

浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的`XMLHttpRequest`请求，否则就报错。

```http
OPTIONS /cors HTTP/1.1
Origin: http://manage.leyou.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
Host: api.leyou.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```

与简单请求相比，除了Origin以外，多了两个头：

- Access-Control-Request-Method：接下来会用到的请求方式，比如PUT
- Access-Control-Request-Headers：会额外用到的头信息

服务的收到预检请求，如果许可跨域，会发出响应：

```http
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://manage.leyou.com
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Max-Age: 1728000
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```

除了`Access-Control-Allow-Origin`和`Access-Control-Allow-Credentials`以外，这里又额外多出3个头：

- Access-Control-Allow-Methods：允许访问的方式
- Access-Control-Allow-Headers：允许携带的头
- Access-Control-Max-Age：本次许可的有效时长，单位是秒，**过期之前的ajax请求就无需再次进行预检了



### SpringMVC实现

实现比较简单：

- 浏览器端都有浏览器自动完成，我们无需操心
- 服务端可以通过拦截器统一实现，不必每次都去进行跨域判定的编写。

事实上，SpringMVC已经帮我们写好了CORS的跨域过滤器：CorsFilter，内部已经实现了刚才所讲的判定逻辑，我们直接用就好了。

```java
@Configuration
public class CorsConfig {
    @Bean
    public CorsFilter corsFilter() {
        //1.添加CORS配置信息
        CorsConfiguration config = new CorsConfiguration();
        //1) 允许的域,不要写*，否则cookie就无法使用了
        config.addAllowedOrigin("http://manage.leyou.com");
        //2) 是否发送Cookie信息
        config.setAllowCredentials(true);
        //3) 允许的请求方式
        config.addAllowedMethod("OPTIONS");
        config.addAllowedMethod("HEAD");
        config.addAllowedMethod("GET");
        config.addAllowedMethod("PUT");
        config.addAllowedMethod("POST");
        config.addAllowedMethod("DELETE");
        config.addAllowedMethod("PATCH");
        // 4）允许的头信息
        config.addAllowedHeader("*");

        //2.添加映射路径，我们拦截一切请求
        UrlBasedCorsConfigurationSource configSource = new UrlBasedCorsConfigurationSource();
        configSource.registerCorsConfiguration("/**", config);

        //3.返回新的CorsFilter
        return new CorsFilter(configSource);
    }
}
```



## Mybatis

### #{}和${}的区别是什么

`${}`是变量占位符，属于静态文本替换，比如${driver}会被静态替换为com.mysql.jdbc.Driver。`#{}`是sql的参数占位符，Mybatis会将sql中的`#{}`替换为?号，在sql执行前会使用PreparedStatement的参数设置方法，按序给sql的?号占位符设置参数值，比如`ps.setInt(0, parameterValue)`，`#{item.name}`的取值方式为使用反射从参数对象中获取item对象的name属性值，相当于`param.getItem().getName()`。



### 当实体类中的属性名和表中的字段名不一样 ，怎么办

第1种： 通过在查询的sql语句中定义字段名的别名，让字段名的别名和实体类的属性名一致

```xml
<select id=”selectorder” parametertype=”int” resultetype=”me.gacl.domain.order”> 
    select order_id id, order_no orderno ,order_price price form orders where order_id=#{id}; 
</select> 
```

第2种： 通过`<resultMap></resultMap>`来映射字段名和实体类属性名的一一对应的关系

```xml
<select id="getOrder" parameterType="int" resultMap="orderresultmap">
    select * from orders where order_id=#{id}
</select>

<resultMap type=”me.gacl.domain.order” id=”orderresultmap”> 
    <!–用id属性来映射主键字段–> 
    <id property=”id” column=”order_id” /> 
    <!–用result属性来映射非主键字段，property为实体类属性名，column为数据表中的属性–> 
    <result property = “orderno” column =”order_no”/> 
    <result property=”price” column=”order_price” /> 
</reslutMap>
```



### 模糊查询like语句该怎么写

第1种：在Java代码中添加sql通配符

```
string wildcardname = "%smi%"; 
list<name> names = mapper.selectlike(wildcardname);

<select id=”selectlike”> 
	select * from foo where bar like #{value} 
</select>
```

第2种：在sql语句中拼接通配符，会引起sql注入

```
string wildcardname = "smi"; 
list<name> names = mapper.selectlike(wildcardname);

<select id=”selectlike”> 
	select * from foo where bar like "%"#{value}"%"
</select>
```



### 通常一个Xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗

Dao接口，就是人们常说的Mapper接口，接口的全限名，就是映射文件中的namespace的值，接口的方法名，就是映射文件中MappedStatement的id值，接口方法内的参数，就是传递给sql的参数。Mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为key值，可唯一定位一个MappedStatement，举例：`com.mybatis3.mappers.StudentDao.findStudentById`，可以唯一找到`namespace`为`com.mybatis3.mappers.StudentDao`下面`id = findStudentById`的MappedStatement。在Mybatis中，每一个	`<select>`、`<insert>`、`<update>`、`<delete>`标签，都会被解析为一个MappedStatement对象。

Dao接口里的方法，是不能重载的，因为是全限名+方法名的保存和寻找策略。

Dao接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Dao接口生成代理proxy对象，代理对象proxy会拦截接口方法，转而执行MappedStatement所代表的sql，然后将sql执行结果返回。



### Mybatis是如何进行分页的？分页插件的原理是什么？

Mybatis使用RowBounds对象进行分页，它是针对ResultSet结果集执行的内存分页，而非物理分页，可以在sql内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。

分页插件的基本原理是使用Mybatis提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的sql，然后重写sql，根据dialect方言，添加对应的物理分页语句和物理分页参数。



### Mybatis是如何将sql执行结果封装为目标对象并返回的？都有哪些映射形式？

第一种是使用`<resultMap><resultMap/>`标签，逐一定义列名和对象属性名之间的映射关系。

第二种是使用sql列的别名功能，将列别名书写为对象属性名，比如T_NAME AS NAME，对象属性名一般是name，小写，但是列名不区分大小写，Mybatis会忽略列名大小写，智能找到与之对应对象属性名，你甚至可以写成T_NAME AS NaMe，Mybatis一样可以正常工作。

有了列名与属性名的映射关系后，Mybatis通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。



### 如何执行批量插入

首先，创建一个简单的insert语句：

```xml
<insert id=”insertname”> 
    insert into names (name) values (#{value}) 
</insert>
```

然后在java代码中像下面这样执行批处理插入：

```java
List<String> names = new ArrayList<>();
names.add("fred");
names.add("barney");
names.add("betty");
names.add("wilma");

// 注意这里 executortype.batch
SqlSession sqlsession = sqlSessionFactory.openSession(ExecutorType.BATCH);
try {
    NameMapper mapper = sqlsession.getMapper(NameMapper.class);
    for (String name : names) {
        mapper.insertname(name);
    }
    sqlsession.commit();
} finally {
    sqlsession.close();
}
```



### 如何获取自动生成的(主)键值?

insert 方法总是返回一个int值 ，这个值代表的是插入的行数。

如果采用自增长策略，自动生成的键值在 insert 方法执行完后可以被设置到传入的参数对象中。

```xml
<insert id=”insertname” usegeneratedkeys=”true” keyproperty=”id”>
    insert into names (name) values (#{name})
</insert>
```

```java
name name = new name();
name.setname(“fred”);

int rows = mapper.insertname(name);
// 完成后,id已经被设置到对象中
system.out.println(“rows inserted = ” + rows);
system.out.println(“generated key value = ” + name.getid());
```



### 在mapper中如何传递多个参数

```
（1）第一种：
//DAO层的函数
Public UserselectUser(String name,String area);  

//对应的xml,#{0}代表接收的是dao层中的第一个参数，#{1}代表dao层中第二参数，更多参数一致往后加即可。
<select id="selectUser"resultMap="BaseResultMap">  
    select *  from user_user_t where user_name = #{0} and user_area=#{1}  
</select>  
 
（2）第二种： 使用 @param 注解:
public interface usermapper {
   user selectuser(@param(“username”) string username,@param(“hashedpassword”) string hashedpassword);
}
然后,就可以在xml像下面这样使用(推荐封装为一个map,作为单个参数传递给mapper):
<select id=”selectuser” resulttype=”user”>
         select id, username, hashedpassword
         from some_table
         where username = #{username}
         and hashedpassword = #{hashedpassword}
</select>
 
（3）第三种：多个参数封装成map
https://blog.csdn.net/earthhour/article/details/79635633
```



### 一对一查询

```java
public class Account implements Serializable {
    private Integer id;
    private Integer uid;
    private Double money;
    
    private User user;
    public User getUser() {
        return user;
    }
    public void setUser(User user) {
        this.user = user;
    }
    
    // getter and setter
}
```

```xml
<!-- 建立对应关系 -->
<resultMap type="account" id="accountMap">
    <id column="aid" property="id"/>
    <result column="uid" property="uid"/>
    <result column="money" property="money"/>
    <!-- 它是用于指定从表方的引用实体属性的 -->
    <association property="user" javaType="user">
        <id column="id" property="id"/>
        <result column="username" property="username"/>
        <result column="sex" property="sex"/>
        <result column="birthday" property="birthday"/>
        <result column="address" property="address"/>
    </association>
</resultMap>

<select id="findAll" resultMap="accountMap">
    select u.*, a.id as aid, a.uid, a.money from account a, user u where a.uid=u.id;
</select>
```



### 一对多

```java
public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;
    
    private List<Account> accounts;
    public List<Account> getAccounts() {
        return accounts;
    }
    public void setAccounts(List<Account> accounts) {
        this.accounts = accounts;
    }
    
    // getter and setter
}
```

```xml
<resultMap type="user" id="userMap">
    <id column="id" property="id"></id>
    <result column="username" property="username"/>
    <result column="address" property="address"/>
    <result column="sex" property="sex"/>
    <result column="birthday" property="birthday"/>
    <!-- collection 是用于建立一对多中集合属性的对应关系，ofType 用于指定集合元素的数据类型-->
    <collection property="accounts" ofType="account">
        <id column="aid" property="id"/>
        <result column="uid" property="uid"/>
        <result column="money" property="money"/>
    </collection>
</resultMap>
<!-- 配置查询所有操作 -->
<select id="findAll" resultMap="userMap">
    select u.*, a.id as aid, a.uid, a.money from user u left outer join account
    a on u.id=a.uid
</select>
```



## SpringMVC

### 流程

<div align="center"><img src="http://q0l9qvfyx.bkt.clouddn.com/29c6af83-93f7-4b00-809b-d2692976e48c"></div>
#### SpringMVC的流程

1. 用户发送请求至前端控制器DispatcherServlet；
2. DispatcherServlet收到请求后，调用HandlerMapping处理器映射器，请求获取Handle；
3. 处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器（如果有则生成）一并返回给DispatcherServlet；
4. DispatcherServlet 调用 HandlerAdapter 处理器适配器；
5. HandlerAdapter 经过适配调用具体处理器（Handler，也叫后端控制器）；
6. Handler执行完成返回ModelAndView；
7. HandlerAdapter将Handler执行结果ModelAndView返回给DispatcherServlet；
8. DispatcherServlet将ModelAndView传给ViewResolver视图解析器进行解析；
9. ViewResolver解析后返回具体View；
10. DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）
11. DispatcherServlet响应用户。

#### 组件说明

1. 前端控制器 DispatcherServlet（不需要程序员开发）。作用：接收请求、响应结果，相当于转发器，有了DispatcherServlet 就减少了其它组件之间的耦合度。
2. 处理器映射器HandlerMapping（不需要程序员开发）。作用：根据请求的URL来查找Handler。
3. 处理器适配器HandlerAdapter（不需要程序员开发）。注意：在编写Handler的时候要按照HandlerAdapter要求的规则去编写，这样适配器HandlerAdapter才可以正确的去执行Handler。
4. 处理器Handler（需要程序员开发）
5. 视图解析器 ViewResolver（不需要程序员开发）。作用：进行视图的解析，根据视图逻辑名解析成真正的视图（view）
6. 视图View（需要程序员开发jsp）。View是一个接口， 它的实现类支持不同的视图类型（jsp，freemarker，pdf等等）。



### SpringMvc的控制器是不是单例模式，如果是，有什么问题，怎么解决？

是单例模式，所以在多线程访问的时候有线程安全问题，不要用同步，会影响性能的，解决方案是在控制器里面不能写字段。



## Spring

### Bean的生命周期

在IoC容器启动之后，并不会马上就实例化相应的bean，此时容器仅仅拥有所有对象的BeanDefinition（BeanDefinition：是容器依赖某些工具加载的XML配置信息进行解析和分析，并将分析后的信息编组为相应的BeanDefinition）。只有当getBean()调用时才是有可能触发Bean实例化阶段的活动。

因为当对应某个bean定义的getBean()方法第一次被调用时，不管是显示的还是隐式的，Bean实例化阶段才会被触发，第二次被调用则会直接返回容器缓存的第一次实例化完的对象实例（因为默认是singleton单例，当然，这里的情况prototype类型的bean除外）

#### Bean的一生过程

<div align="center"><img src="http://q0l9qvfyx.bkt.clouddn.com/e61db431-673d-4f2d-be40-4377c65d096a"></div>
**带星号的不用考虑**

1. 实例化bean对象(通过构造方法或者工厂方法)
2. 设置对象属性(setter等)（依赖注入）
3. 如果Bean实现了BeanNameAware接口，工厂调用Bean的setBeanName()方法传递Bean的ID。（和下面的一条均属于检查Aware接口）
4. 如果Bean实现了BeanFactoryAware接口，工厂调用setBeanFactory()方法传入工厂自身
5. 如果这个 Bean 已经实现了 ApplicationContextAware 接口，会调用 setApplicationContext(ApplicationContext)方法，传入 Spring 上下文
6. 将Bean实例传递给Bean的前置处理器的postProcessBeforeInitialization(Object bean, String beanname)方法
7. 调用Bean的初始化方法
8. 将Bean实例传递给Bean的后置处理器的postProcessAfterInitialization(Object bean, String beanname)方法
9. 使用Bean
10. 当 Bean 不再需要时，会经过清理阶段，如果 Bean 实现了 DisposableBean 这个接口，会调用那个其实现的 destroy()方法；
11. 最后，如果这个 Bean 的 Spring 配置中配置了 destroy-method 属性，会自动调用其配置的销毁方法。

#### 基础示例

```java
public class Student implements BeanNameAware {
	private String name;

	//无参构造方法
	public Student() { super(); }

	/** 设置对象属性
	 * @param name the name to set
	 */
	public void setName(String name) {
		System.out.println("设置对象属性setName()..");
		this.name = name;
	}
	
	//Bean的初始化方法
	public void initStudent() {
		System.out.println("Student这个Bean：初始化");
	}
	
	//Bean的销毁方法
	public void destroyStudent() {
		System.out.println("Student这个Bean：销毁");
	}
	
	//Bean的使用
	public void play() {
		System.out.println("Student这个Bean：使用");
	}

	/* 重写toString
	 * @see java.lang.Object#toString()
	 */
	@Override
	public String toString() {
		return "Student [name = " + name + "]";
	}

	//调用BeanNameAware的setBeanName()
	//传递Bean的ID。
	@Override
	public void setBeanName(String name) {
		System.out.println("调用BeanNameAware的setBeanName()..." ); 
	}
}
```

```java
public class CycleTest {
    @Test
 	public void test() {
		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
		Student student = (Student) context.getBean("student");
		//Bean的使用
		student.play();
		System.out.println(student);
		//关闭容器
		((AbstractApplicationContext) context).close();
	}
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
                           http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- init-method：指定初始化的方法
        destroy-method：指定销毁的方法 -->
    <bean id="student" class="com.linjie.cycle.Student" init-method="initStudent" destroy-method="destroyStudent">
        <property name="name" value="LINJIE"></property>
    </bean>
</beans>
```

<div align="center"><img src="http://q0l9qvfyx.bkt.clouddn.com/77ea335f-e5cf-44b6-9e24-2ce8ffa241ea"></div>

#### Bean的前/后置处理器

上面bean的一生其实已经算是对bean生命周期很完整的解释了，然而bean的前/后置处理器，是为了对bean的一个增强。

```java
public class Student implements BeanNameAware {
	private String name;

	// 无参构造方法
	public Student() { super(); }

	/** 设置对象属性
	 * @param name the name to set
	 */
	public void setName(String name) {
		System.out.println("设置对象属性setName()..");
		this.name = name;
	}
	
	// Bean的初始化方法
	public void initStudent() {
		System.out.println("Student这个Bean：初始化");
	}
	
	// Bean的销毁方法
	public void destroyStudent() {
		System.out.println("Student这个Bean：销毁");
	}
	
	// Bean的使用
	public void play() {
		System.out.println("Student这个Bean：使用");
	}

	/* 重写toString
	 * @see java.lang.Object#toString()
	 */
	@Override
	public String toString() {
		return "Student [name = " + name + "]";
	}

	// 调用BeanNameAware的setBeanName()
	// 传递Bean的ID。
	@Override
	public void setBeanName(String name) {
		System.out.println("调用BeanNameAware的setBeanName()..." ); 
	}
}
```

```java
/**
 * bean的后置处理器
 * 分别在bean的初始化前后对bean对象提供自己的实例化逻辑
 * postProcessAfterInitialization：初始化之后对bean进行增强处理
 * postProcessBeforeInitialization：初始化之前对bean进行增强处理
 */
public class MyBeanPostProcessor implements BeanPostProcessor {

	//对初始化之后的Bean进行处理
	//参数：bean：即将初始化的bean
	//参数：beanname：bean的名称
	//返回值：返回给用户的那个bean,可以修改bean也可以返回一个新的bean
	@Override
	public Object postProcessAfterInitialization(Object bean, String beanname) throws BeansException {
		Student stu = null;
		System.out.println("对初始化之后的Bean进行处理，将Bean的成员变量的值修改了");
		if("student".equals(beanname) && bean instanceof Student) {
			stu = (Student) bean;
			stu.setName("Jack");
		}
		return stu;
	}

	//对初始化之前的Bean进行处理
	//参数：bean：即将初始化的bean
	//参数：beanname：bean的名称
	//返回值：返回给用户的那个bean,可以修改bean也可以返回一个新的bean
	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanname) throws BeansException {
		System.out.println("对初始化之前的Bean进行处理,此时我的名字"+bean);
		return bean;
	}
}
```

```java
public class CycleTest {
    @Test
 	public void test() {
		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
		Student student = (Student) context.getBean("student");
		//Bean的使用
		student.play();
		System.out.println(student);
		//关闭容器
		((AbstractApplicationContext) context).close();
	}
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <!-- init-method：指定初始化的方法，destroy-method：指定销毁的方法 -->
    <bean id="student" class="com.linjie.cycle.Student" init-method="initStudent" destroy-method="destroyStudent">
        <property name="name" value="LINJIE"></property>
    </bean>

    <!-- 配置bean的后置处理器,不需要id，IoC容器自动识别是一个BeanPostProcessor -->
    <bean class="com.linjie.cycle.MyBeanPostProcessor"></bean> 
    
</beans>
```

<div align="center"><img src="http://q0l9qvfyx.bkt.clouddn.com/1ba03624-dc68-4a38-b103-e848797d941e"></div>

#### BeanFactoryAware和ApplicationContextAware

这两个接口是为了获得Spring的上下文，Spring的IOC有两个常用接口，BeanFactory和ApplicationContext，这两个接口是为了分别获得这两个上下文。

```java
@Component
public class BeanUtil implements BeanFactoryAware{
    private static BeanFactory beanFactory;  

    @Override
    public void setBeanFactory(BeanFactory factory) throws BeansException {  
        this.beanFactory = factory;  
    }  

    public static <T> T getBean(String beanName) {  
        if (null != beanFactory) {  
            return (T) beanFactory.getBean(beanName);  
        }  
        return null;  
    }
}
```

```java
@Component
public class AppUtil implements ApplicationContextAware {

    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext arg0) throws BeansException{
        applicationContext = arg0;
    }
    public static Object getObject(String id) {
        Object object = null;
        object = applicationContext.getBean(id);
        return object;
    }
}
```



### Spring支持的bean的作用域

Spring容器中的bean可以分为5个范围：

- singleton：默认，每个容器中只有一个bean的实例，单例的模式由BeanFactory自身来维护。


- prototype：为每一个bean请求提供一个实例。


- request：为每一个网络请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。


- session：与request范围类似，确保每个session中有一个bean的实例，在session过期后，bean会随之失效。


- global-session：全局作用域，global-session和Portlet应用相关。当你的应用部署在Portlet容器中工作时，它包含很多portlet。如果你想要声明让所有的portlet共用全局的存储变量的话，那么这全局变量需要存储在global-session中。全局作用域与Servlet中的session作用域效果相同。



### AOP

<div align="center"><img src="http://q0l9qvfyx.bkt.clouddn.com/9a9503ae-8991-41fd-ae00-5bda7be34009"></div>

#### Spring的XML方式配置AOP

```java
public class Logger {
    /**
     * 用于打印日志：计划让其在切入点方法执行之前执行（切入点方法就是业务层方法）
     */
    public  void printLog(){
        System.out.println("Logger类中的pringLog方法开始记录日志了。。。");
    }
}
```

```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"></bean>

<!--spring中基于XML的AOP配置步骤
        1、把通知Bean也交给spring来管理
        2、使用aop:config标签表明开始AOP的配置
        3、使用aop:aspect标签表明配置切面
                id属性：是给切面提供一个唯一标识
                ref属性：是指定通知类bean的Id。
        4、在aop:aspect标签的内部使用对应标签来配置通知的类型
               我们现在示例是让printLog方法在切入点方法执行之前之前：所以是前置通知
               aop:before：表示配置前置通知
                    method属性：用于指定Logger类中哪个方法是前置通知
                    pointcut属性：用于指定切入点表达式，该表达式的含义指的是对业务层中哪些方法增强

            切入点表达式的写法：
                关键字：execution(表达式)
                表达式：
                    访问修饰符  返回值  包名.包名.包名...类名.方法名(参数列表)
                标准的表达式写法：
                    public void com.itheima.service.impl.AccountServiceImpl.saveAccount()
                访问修饰符可以省略
                    void com.itheima.service.impl.AccountServiceImpl.saveAccount()
                返回值可以使用通配符，表示任意返回值
                    * com.itheima.service.impl.AccountServiceImpl.saveAccount()
                包名可以使用通配符，表示任意包。但是有几级包，就需要写几个*.
                    * *.*.*.*.AccountServiceImpl.saveAccount())
                包名可以使用..表示当前包及其子包
                    * *..AccountServiceImpl.saveAccount()
                类名和方法名都可以使用*来实现通配
                    * *..*.*()
                参数列表：
                    可以直接写数据类型：
                        基本类型直接写名称           int
                        引用类型写包名.类名的方式   java.lang.String
                    可以使用通配符表示任意类型，但是必须有参数
                    可以使用..表示有无参数均可，有参数可以是任意类型
                全通配写法：
                    * *..*.*(..)

                实际开发中切入点表达式的通常写法：
                    切到业务层实现类下的所有方法
                        * com.itheima.service.impl.*.*(..)
    -->

<!-- 配置Logger类 -->
<bean id="logger" class="com.itheima.utils.Logger"></bean>

<!--配置AOP-->
<aop:config>
    <!--配置切面 -->
    <aop:aspect id="logAdvice" ref="logger">
        <!-- 配置通知的类型，并且建立通知方法和切入点方法的关联-->
        <aop:before method="printLog" pointcut="execution(* com.itheima.service.impl.*.*(..))"></aop:before>
    </aop:aspect>
</aop:config>
```

#### Spring通知有哪些类型

1. 前置通知（Before advice）：在某连接点（join point）之前执行的通知，但这个通知不能阻止连接点前的执行（除非它抛出一个异常）。
2. 返回后通知（After returning advice）：在某连接点（join point）正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。 
3. 抛出异常后通知（After throwing advice）：在方法抛出异常退出时执行的通知。 
4. 后通知（After (finally) advice）：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。 
5. 环绕通知（Around Advice）：包围一个连接点（join point）的通知，如方法调用。这是最强大的一种通知类型。 环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它们自己的返回值或抛出异常来结束执行。 环绕通知是最常用的一种通知类型。大部分基于拦截的AOP框架，例如Nanning和JBoss4，都只提供环绕通知。 

同一个aspect，不同advice的执行顺序：

- 没有异常情况下的执行顺序：
  - around before advice
  - before advice
  - target method 执行
  - around after advice
  - after advice
  - afterReturning

- 有异常情况下的执行顺序：
  - around before advice
  - before advice
  - target method 执行
  - around after advice
  - after advice
  - afterThrowing：异常发生

```java
/**
 * 用于记录日志的工具类，它里面提供了公共的代码
 */
public class Logger {

    /**
     * 前置通知
     */
    public  void beforePrintLog(){
        System.out.println("前置通知Logger类中的beforePrintLog方法开始记录日志了。。。");
    }

    /**
     * 后置通知
     */
    public  void afterReturningPrintLog(){
        System.out.println("后置通知Logger类中的afterReturningPrintLog方法开始记录日志了。。。");
    }
    /**
     * 异常通知
     */
    public  void afterThrowingPrintLog(){
        System.out.println("异常通知Logger类中的afterThrowingPrintLog方法开始记录日志了。。。");
    }

    /**
     * 最终通知
     */
    public  void afterPrintLog(){
        System.out.println("最终通知Logger类中的afterPrintLog方法开始记录日志了。。。");
    }

    /**
     * 环绕通知
     * 问题：
     *      当我们配置了环绕通知之后，切入点方法没有执行，而通知方法执行了。
     * 分析：
     *      通过对比动态代理中的环绕通知代码，发现动态代理的环绕通知有明确的切入点方法调用，而我们的代码中没有。
     * 解决：
     *      Spring框架为我们提供了一个接口：ProceedingJoinPoint。该接口有一个方法proceed()，此方法就相当于明确调用切入点方法。
     *      该接口可以作为环绕通知的方法参数，在程序执行时，spring框架会为我们提供该接口的实现类供我们使用。
     *
     * spring中的环绕通知：
     *      它是spring框架为我们提供的一种可以在代码中手动控制增强方法何时执行的方式。
     */
    public Object aroundPringLog(ProceedingJoinPoint pjp){
        Object rtValue = null;
        try{
            Object[] args = pjp.getArgs();//得到方法执行所需的参数

            System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。前置");

            rtValue = pjp.proceed(args);//明确调用业务层方法（切入点方法）

            System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。后置");

            return rtValue;
        }catch (Throwable t){
            System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。异常");
            throw new RuntimeException(t);
        }finally {
            System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。最终");
        }
    }
}
```

```xml
<!-- 配置srping的Ioc,把service对象配置进来-->
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"></bean>


<!-- 配置Logger类 -->
<bean id="logger" class="com.itheima.utils.Logger"></bean>

<!--配置AOP-->
<aop:config>
    <!-- 配置切入点表达式 id属性用于指定表达式的唯一标识。expression属性用于指定表达式内容
              此标签写在aop:aspect标签内部只能当前切面使用。
              它还可以写在aop:aspect外面，此时就变成了所有切面可用
          -->
    <aop:pointcut id="pt1" expression="execution(* com.itheima.service.impl.*.*(..))"></aop:pointcut>
    <!--配置切面 -->
    <aop:aspect id="logAdvice" ref="logger">
        <!-- 配置前置通知：在切入点方法执行之前执行
            <aop:before method="beforePrintLog" pointcut-ref="pt1" ></aop:before>-->

        <!-- 配置后置通知：在切入点方法正常执行之后值。它和异常通知永远只能执行一个
            <aop:after-returning method="afterReturningPrintLog" pointcut-ref="pt1"></aop:after-returning>-->

        <!-- 配置异常通知：在切入点方法执行产生异常之后执行。它和后置通知永远只能执行一个
            <aop:after-throwing method="afterThrowingPrintLog" pointcut-ref="pt1"></aop:after-throwing>-->

        <!-- 配置最终通知：无论切入点方法是否正常执行它都会在其后面执行
            <aop:after method="afterPrintLog" pointcut-ref="pt1"></aop:after>-->

        <!-- 配置环绕通知 详细的注释请看Logger类中-->
        <aop:around method="aroundPringLog" pointcut-ref="pt1"></aop:around>
    </aop:aspect>
</aop:config>
```

#### Spring的注解方式配置AOP

```xml
<!-- 配置spring创建容器时要扫描的包-->
<context:component-scan base-package="com.itheima"></context:component-scan>

<!-- 配置spring开启注解AOP的支持 -->
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

```java
/**
 * 用于记录日志的工具类，它里面提供了公共的代码
 */
@Component("logger")
@Aspect//表示当前类是一个切面类
public class Logger {

    @Pointcut("execution(* com.itheima.service.impl.*.*(..))")
    private void pt1(){}

    /**
     * 前置通知
     */
//    @Before("pt1()")
    public  void beforePrintLog(){
        System.out.println("前置通知Logger类中的beforePrintLog方法开始记录日志了。。。");
    }

    /**
     * 后置通知
     */
//    @AfterReturning("pt1()")
    public  void afterReturningPrintLog(){
        System.out.println("后置通知Logger类中的afterReturningPrintLog方法开始记录日志了。。。");
    }
    
    /**
     * 异常通知
     */
//    @AfterThrowing("pt1()")
    public  void afterThrowingPrintLog(){
        System.out.println("异常通知Logger类中的afterThrowingPrintLog方法开始记录日志了。。。");
    }

    /**
     * 最终通知
     */
//    @After("pt1()")
    public  void afterPrintLog(){
        System.out.println("最终通知Logger类中的afterPrintLog方法开始记录日志了。。。");
    }

    /**
     * 环绕通知
     * 问题：
     *      当我们配置了环绕通知之后，切入点方法没有执行，而通知方法执行了。
     * 分析：
     *      通过对比动态代理中的环绕通知代码，发现动态代理的环绕通知有明确的切入点方法调用，而我们的代码中没有。
     * 解决：
     *      Spring框架为我们提供了一个接口：ProceedingJoinPoint。该接口有一个方法proceed()，此方法就相当于明确调用切入点方法。
     *      该接口可以作为环绕通知的方法参数，在程序执行时，spring框架会为我们提供该接口的实现类供我们使用。
     *
     * spring中的环绕通知：
     *      它是spring框架为我们提供的一种可以在代码中手动控制增强方法何时执行的方式。
     */
    @Around("pt1()")
    public Object aroundPringLog(ProceedingJoinPoint pjp){
        Object rtValue = null;
        try{
            Object[] args = pjp.getArgs();//得到方法执行所需的参数

            System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。前置");

            rtValue = pjp.proceed(args);//明确调用业务层方法（切入点方法）

            System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。后置");

            return rtValue;
        }catch (Throwable t){
            System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。异常");
            throw new RuntimeException(t);
        }finally {
            System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。最终");
        }
    }
}
```



### 事务

#### xml方式配置事务

```xml
<!-- 配置业务层-->
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
    <property name="accountDao" ref="accountDao"></property>
</bean>

<!-- 配置账户的持久层-->
<bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!-- 配置数据源-->
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql://localhost:3306/eesy"></property>
    <property name="username" value="root"></property>
    <property name="password" value="1234"></property>
</bean>

<!-- spring中基于XML的声明式事务控制配置步骤
        1、配置事务管理器
        2、配置事务的通知
                此时我们需要导入事务的约束 tx名称空间和约束，同时也需要aop的
                使用tx:advice标签配置事务通知
                    属性：
                        id：给事务通知起一个唯一标识
                        transaction-manager：给事务通知提供一个事务管理器引用
        3、配置AOP中的通用切入点表达式
        4、建立事务通知和切入点表达式的对应关系
        5、配置事务的属性
               是在事务的通知tx:advice标签的内部

     -->
<!-- 配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!-- 配置事务的通知-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <!-- 配置事务的属性
                isolation：用于指定事务的隔离级别。默认值是DEFAULT，表示使用数据库的默认隔离级别。
                propagation：用于指定事务的传播行为。默认值是REQUIRED，表示一定会有事务，增删改的选择。查询方法可以选择SUPPORTS。
                read-only：用于指定事务是否只读。只有查询方法才能设置为true。默认值是false，表示读写。
                timeout：用于指定事务的超时时间，默认值是-1，表示永不超时。如果指定了数值，以秒为单位。
                rollback-for：用于指定一个异常，当产生该异常时，事务回滚，产生其他异常时，事务不回滚。没有默认值。表示任何异常都回滚。
                no-rollback-for：用于指定一个异常，当产生该异常时，事务不回滚，产生其他异常时事务回滚。没有默认值。表示任何异常都回滚。
        -->
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED" read-only="false"/>
        <tx:method name="find*" propagation="SUPPORTS" read-only="true"></tx:method>
    </tx:attributes>
</tx:advice>

<!-- 配置aop-->
<aop:config>
    <!-- 配置切入点表达式-->
    <aop:pointcut id="pt1" expression="execution(* com.itheima.service.impl.*.*(..))"></aop:pointcut>
    <!--建立切入点表达式和事务通知的对应关系 -->
    <aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"></aop:advisor>
</aop:config>
```

#### 注解方式配置事务

```xml
<!-- 配置spring创建容器时要扫描的包-->
<context:component-scan base-package="com.itheima"></context:component-scan>

<!-- 配置JdbcTemplate-->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!-- 配置数据源-->
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql://localhost:3306/eesy"></property>
    <property name="username" value="root"></property>
    <property name="password" value="1234"></property>
</bean>

<!-- spring中基于注解 的声明式事务控制配置步骤
        1、配置事务管理器
        2、开启spring对注解事务的支持
        3、在需要事务支持的地方使用@Transactional注解
     -->
<!-- 配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!-- 开启spring对注解事务的支持-->
<tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
```

```java
/**
 * 账户的业务层实现类
 * 事务控制应该都是在业务层
 */
@Service("accountService")
@Transactional(propagation=Propagation.SUPPORTS, readOnly=true)//只读型事务的配置
public class AccountServiceImpl implements IAccountService{

    @Autowired
    private IAccountDao accountDao;

    @Override
    public Account findAccountById(Integer accountId) {
        return accountDao.findAccountById(accountId);
    }

    //需要的是读写型事务配置
    @Transactional(propagation= Propagation.REQUIRED,readOnly=false)
    @Override
    public void transfer(String sourceName, String targetName, Float money) {
        System.out.println("transfer....");
            //2.1根据名称查询转出账户
            Account source = accountDao.findAccountByName(sourceName);
            //2.2根据名称查询转入账户
            Account target = accountDao.findAccountByName(targetName);
            //2.3转出账户减钱
            source.setMoney(source.getMoney()-money);
            //2.4转入账户加钱
            target.setMoney(target.getMoney()+money);
            //2.5更新转出账户
            accountDao.updateAccount(source);
            int i=1/0;
            //2.6更新转入账户
            accountDao.updateAccount(target);
    }
}
```

#### 事务的传播行为

- PROPAGATION_REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。
- PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。
- PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。
- PROPAGATION_REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务。
- PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。
- PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。



### Spring的设计模式

#### 简单工厂

又叫做静态工厂方法（StaticFactory Method）模式，但不属于 23 种 GOF 设计模式之一。简单工厂模式的实质是由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类。spring 中的 BeanFactory 就是简单工厂模式的体现，根据传入一个唯一的标识来获得 bean 对象，但是否是在传入参数后创建还是传入参数前创建这个要根据具体情况来定。如下配置，就是在 HelloItxxz 类中创建一个 itxxzBean。

#### 单例模式（Singleton）

保证一个类仅有一个实例，并提供一个访问它的全局访问点。spring 中的单例模式完成了后半句话，即提供了全局的访问点 BeanFactory。但没有从构造器级别去控制单例，这是因为 spring 管理的是是任意的 java 对象。
核心提示点：Spring 下默认的 bean 均为 singleton，可以通过 singleton=“true|false” 或者 scope=“？”来指定

#### 适配器（Adapter）

在 Spring 的 Aop 中，使用的 Advice（通知）来增强被代理类的功能。Spring 实现这一 AOP 功能的原理就使用代理模式（1、JDK 动态代理。2、CGLib 字节码生成技术代理。）对类进行方法级别的切面增强，即，生成被代理类的代理类， 并在代理类的方法前，设置拦截器，通过执行拦截器重的内容增强了代理方法的功能，实现的面向切面编程。

#### 代理（Proxy）

为其他对象提供一种代理以控制对这个对象的访问。 从结构上来看和 Decorator 模式类似，但 Proxy 是控制，更像是一种对功能的限制，而 Decorator 是增加职责。spring 的 Proxy 模式在 aop 中有体现，比如 JdkDynamicAopProxy 和 Cglib2AopProxy。

#### 观察者（Observer）

定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。spring 中 Observer 模式常用的地方是 listener 的实现。如 ApplicationListener。

#### 策略（Strategy）

定义一系列的算法，把它们一个个封装起来，并且使它们可相互替换。本模式使得算法可独立于使用它的客户而变化。spring 中在实例化对象的时候用到 Strategy 模式在 SimpleInstantiationStrategy 中有如下代码说明了策略模式的使用情况：

#### 模板方法（Template Method）

定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。Template Method 使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。Template Method 模式一般是需要继承的。这里想要探讨另一种对 Template Method 的理解。spring 中的 JdbcTemplate，在用这个类时并不想去继承这个类，因为这个类的方法太多，但是我们还是想用到 JdbcTemplate 已有的稳定的、公用的数据库连接，那么我们怎么办呢？我们可以把变化的东西抽出来作为一个参数传入 JdbcTemplate 的方法中。但是变化的东西是一段代码，而且这段代码会用到 JdbcTemplate 中的变量。怎么办？那我们就用回调对象吧。在这个回调对象中定义一个操纵 JdbcTemplate 中变量的方法，我们去实现这个方法，就把变化的东西集中到这里了。然后我们再传入这个回调对象到JdbcTemplate，从而完成了调用。这可能是 Template Method 不需要继承的另一种实现方式吧。