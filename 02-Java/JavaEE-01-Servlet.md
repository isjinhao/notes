## Tomcat结构

<div align='center'><img src="http://blogfileqiniu.isjinhao.site/db71dc1e-b4ac-4f06-a3f0-be144ffad048" /></div>
**bin**

用来存放Tomcat的可执行文件：

- `startup.bat`是windows系统下启动Tomcat的可执行文件；
- `shutdown.bat`是windows系统下关闭Tomca的可执行文件。
- `startup.sh`是Linux下的启动Tomcat的可执行文件；
- `shutdown.sh`是linux下关闭Tomca的可执行文件。

**conf**

存放Tomcat服务器全局配置的各种文件。

- `web.xml`：给动态Web工程提供相应的配置，比如Session的过期时间，如果在工程的`web.xml`中覆盖了同种配置，以工程配置优先。
- `server.xml`：配置和服务器本身相关的信息，如用什么编码集解析URL，

**lib**

存放的是tomcat运行时和项目运行时必须的jar包。如果我们想把某个jar包让所有工程都能使用而不用每个工程都导入，直接将其放入lib文件夹下即可。

**logs**

存放的是日志文件

**webapps**

存放要发布的Web项目。将Web项目打包成War包，放在此目录下，在Tomcat启动时会将War解压并发布。

**work**

用来存放jsp文件文件在运行时产生的java文件和class文件。



## Dynamic Web项目结构

### Web项目结构

```
myweb(目录名:项目名)
			|
			|---资源文件  html img css js   可以存放到文件夹下
			|---WEB-INF(目录:特点,通过浏览器直接访问不到)
			|		|
			|		|---lib(目录:项目运行的jar包)
			|		|---classes(目录:存放的class文件)
			|		|---web.xml(核心配置文件,在web2.5版本中必须有,web3.0版本不是必须的)
```



### 手动创建一个Web项目并发布

<div align='center'><img src="http://blogfileqiniu.isjinhao.site/afbd3139-45fa-4703-9468-6aee8c75ebcb" /></div>
- `http://localhost:8080/test-web/test.html`



<div align='center'><img src="http://blogfileqiniu.isjinhao.site/b5b0758c-0ec7-4260-ad1d-0463dde0f0fa" /></div>
- `http://localhost:8080/test-web/WEB-INF/test.html`

<div align='center'><img src="http://blogfileqiniu.isjinhao.site/d82744f8-a973-4342-b504-157719b93d61" /></div>

## HTTP请求

HTTP请求共有DELETE、HEAD、GET、OPTIONS、POST、PUT、TRACE和CONNECT八种请求方式。

**请求行**

`请求的方式 请求的资源 协议/版本`。

**请求头**

key-value类型的数据。

- `Accept`：浏览器可接受的mime类型，如：`text/html,image/*`。
- `Accept-Charset`：浏览器解析所用哪个的字符集，如：` ISO-8859-1`。
- `Accept-Encoding`：浏览器能够进行解码的数据编码方式，比如`gzip`。Servlet能够向支持gzip的浏览器返回经gzip编码的HTML页面。
- `Accept-Language`：浏览器所希望的语言种类，当服务器能够提供一种以上的语言版本时要用到。这个指的是中文、英语这种语言。
- `Host`：被访问的主机。
- `If-Modified-Since`：在发送HTTP请求时，把浏览器端缓存页面的最后修改时间一起发到服务器去，服务器会把这个时间与服务器上实际文件的最后修改时间进行比较。如果时间一致，那么返回HTTP状态码304（不返回文件内容），客户端接到之后，就直接把本地缓存文件显示到浏览器中。如果时间不一致，就返回HTTP状态码200和新的文件内容，客户端接到之后，会丢弃旧文件，把新文件缓存起来，并显示到浏览器中。
- `Referer`：告诉服务器我是从哪个页面链接过来的，服务器基此可以获得一些信息用于处理。
- `User-Agent`：浏览器内核。客户端浏览器的信息， 如。`Mozilla/4.0 (compatible; MSIE 5.5; Windows NT 5.0) `
- `Cookie`：客户端会话技术。

**请求体**

`post`请求的参数。只有表单提交或异步提交时明确指定`method="post"`这时候是post请求，其他的都是get请求。格式：参数名称=值&参数名称=值。

**图解**

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/7095f0a3-e929-405c-a8d8-e068816d6b82"></div>

## HTTP响应

**响应行**

`版本/协议 响应的状态码 状态码说明`。常见的状态码：

- `200`：响应成功
- `302`：重定向
- `304`：读缓存
- `404`：用户访问的数据不存在
- `500`：服务器内部错误

**响应头**

- `Location`：跳转方向，仅配合状态码302使用才有作用，如 `https://www.baicu.com `。

- `Server`：服务器型号
- `Content-Encoding`：Servlet应该通过查看`Accept-Encoding`头（即`request.getHeader("Accept-Encoding")`）检查浏览器是否支持gzip，为支持gzip的浏览器返回经gzip压缩的HTML页面，为其他浏览器返回普通页面。
- `Content-Length`：数据长度
- Content-Type: text/html; charset=GB2312  	 --数据类型
- `Last-Modified`：客户可以通过`If-Modified-Since`请求头提供一个日期，只有改动时间迟于指定时间的文档才会返回，否则返回一个304（Not Modified）状态。`Last-Modified`也可用setDateHeader方法来设置。
- `Refresh`：表示浏览器应该在多少时间之后刷新文档，以秒计。但只刷新一次。除了刷新当前文档之外，还可以通过`setHeader("Refresh", "5; URL=http://host/path")`让浏览器读取指定的页面。 
- `Content-Disposition`：指示浏览器不要解析文档，而是以附加形式下载。如`attachment; filename=aaa.zip`。
- `Set-Cookie`：设置cookie。

**响应体**

浏览器解析的内容。

**图解**

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/627b176d-f048-46c7-9ff1-59f87f34e2e4"></div>

## Servlet

Servlet是指任何实现了Servlet接口的类。一般情况下Servlet用来扩展基于HTTP协议的Web服务器，它可以接受和响应通过HTTP协议从客户端发过来的信息。

Servlet是一个类，但Servlet类的对象是由Web服务器创建的，不是由开发者创建的。并且多个客户端访问同一个Servlet时只会创建一个Servlet对象。



### Servlet的方法

Servlet有5个方法：

1. void init(ServletConfig config)：在服务器创建 Servlet对象时执行。
2. void destroy() ：在服务器关闭时调用。
3. ServletConfig getServletConfig() ：返回一个ServletConfig对象。
4. String getServletInfo() ：得到Servlet的信息。如作者、版本等。
5. void service(ServletRequest req, ServletResponse res) ：被服务器调用去获得和响应从服务器发来的请求。



### Servlet对象的生命周期

1. Servlet何时创建：默认第一次访问Servlet时创建该对象。
2. Servlet何时销毁：服务器关闭Servlet就销毁了。
3. 每次访问必然执行的方法：`service(ServletRequest req, ServletResponse res)`方法



### Servlet访问过程

浏览器中输入的URL会被浏览器封装成HTTP请求发送给服务器。Tomcat收到从客户端发来的时候解析HTTP请求中的资源地址，然后创建代表请求的requset和代表响应的response对象，再把这两个对象作为参数去调用service()方法。



## ServletConfig

代表当前Servlet在web.xml中的配置信息。当Servlet配置了初始化参数后，web容器在创建Servlet实例对象时，会自动将这些初始化参数封装到ServletConfig对象中，并在调用Servlet的init方法时，将ServletConfig对象传递给Servlet。进而，程序员通过ServletConfig对象就可以得到当前Servlet的初始化参数信息。这样做的好处是：如果将数据库信息、编码方式等配置信息放在web.xml中，如果以后数据库的用户名、密码改变了，则直接很方便地修改web.xml就行了，避免了直接修改源代码的麻烦。**最大的用处是获得ServletContext对象。**



## HttpServlet开发

在开发的时候一般不直接实现Servlet来获取和响应从客户端发来的信息，而是继承Servlet接口的实现类HttpServlet。因为从客户端发来的请求和响应都是基于HTTP协议的，HttpServlet就是用于HTTP协议请求和响应的实现类，继承httpServlet后再覆盖某些方法进行开发即可。



### 继承HttpServlet进行开发

服务器会调用Servlet接口的service()方法，由于多态原则，真正被调用的方法是HttpServlet类的service()方法。HttpServlet类为每种请求方式都设置单独的处理方法，但我们使用的一般使用的是GET和POST请求方式，所以我们只需要覆盖doGet()和doPost()方法即可。这是因为HttpServlet()的service()方法中可以调用每种请求方式的方法，我们覆盖doGet()和doPost()就可以达到功能性需求。

```java
@WebServlet("/TestServlet")
public class TestServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;
       
    public TestServlet() {
        super();
    }

	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// ...
	}

	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		doGet(request, response);
	}
}
```



### web.xml配置

**必须配置的参数**

有Servlet还不行，Web服务器不能直接通过类名访问Servlet，必须通过一个虚拟路径来访问这个Servlet，所以要对Servlet进行配置。配置在web.xml文件中进行。举例说明：

```xml
<servlet>
	<servlet-name>Begin</servlet-name>
	<servlet-class>com.servlet.begin.Begin</servlet-class>
</servlet>
<servlet-mapping>
	<servlet-name>Begin</servlet-name>
	<url-pattern>/begin</url-pattern>
</servlet-mapping>
```

当浏览器中地址输入为：`…/begin`，然后Tomcat去找此项目下虚拟路径为`/begin`对应的`servlet-name`，找到`servlet-name`后映射到servlet配置中找到它所对应的类`com.servlet.begin.Begin`。mapping在集合里指的是映射，在此指的也是映射。servlet-mapping里的Begin相当于Key。servlet里的servlet-class相当于Value。

总结来说，`servlet`和`servlet-mapping`里的`servlet-name`必须一致，`url-pattern`是浏览器输入的地址，`servlet-class`是对访问做出处理的类。

**可选配置的参数**

1. `load-on-startup`：`<load-on-startup>x</load-on-startup>`，x大于等于1的时候表示在服务器启动的时候就创建Servlet对象。
2. 缺省Servlet：可以将url-pattern配置为’/’，代表该servlet是缺省的servlet，也就是当访问资源地址与所有的Servlet都不匹配时，有缺省的servlet负责处理，就是传说中的404界面。



### url-pattern的三种匹配

1. 完全匹配：访问的资源与配置的资源完全相同才能访问到。
2. 目录匹配：格式：`/虚拟的目录../*`。如：`…/abc/*`表示当输入路径`…/abc/xxx`的xxx为任意的时候都可以访问的到。
3. 拓展名匹配：格式：`*.扩展名`。如：`*.abcd`表示输入路径为`…/xxx.abcd`的xxx为任意的时候都能访问。在某路径下进行拓展名匹配是错误的，如：`/aaa/bbb/*.abcd`（错误的）



## HttpServletResponse

HttpServletResponse是一个接口，封装了向客户端输出信息的方法，部分方法如下：

1. void setStatus(int sc)：设置响应状态码；

2. void setHeader(String name, String data)：设置响应头；

3. ServletOutputStream getOutputStream()：返回一个输出流对象；

4. PrintWriter getWriter()：返回一个打印流对象。



### HttpServletResponse的执行过程

HttpServletResponse是向客户端输出信息的接口，但它的对象（设为response）不是直接把信息输出给浏览器，也不是直接输出给服务器。response会把要输出的信息输出到response的缓冲区，然后服务器拿到缓冲区的数据后再添加一些数据组成HTTP响应发送给客户端。



### 重定向

重定向的过程需要两次访问Servlet。当浏览器访问到重定向的Servlet时此Servlet会返回一个302的状态码和重定向的地址。代码会有两句：

1. `response.setStatus(int sc);`
2. `response.setHeader(“location”, “要跳转的地址”);`

但Java把这两步进行了封装，`response.sendRedirect(“要跳转的地址”);`。



### 定时刷新

使用setHeader()。response.setHeader(“refresh”, “3”);表示这个页面3秒后会被刷新。

还可以指定URL跳转到其他页面（URL可以带数据，所以是GET方式跳转）：

```
response.setHeader(“refresh”, “3;url=https:www.baidu.com”)
```



### 输出乱码的解决

1. 设置response缓冲区：response缓冲区的默认编码是iso8859-1，设置为UTF-8编码需要使用：`response.setCharacterEncoding(String charset);`
2. 设置浏览器编码：浏览器默认使用GBK编码，但缓冲区设置的是UTF-8编码，仍然有乱码问题存在：`response.setContentType("text/html;charset=UTF-8");`
3. 一键解决缓冲区和浏览器问题：仍然是`response.setContentType("text/html;charset=UTF-8");`此方法默认包含了`response.setCharacterEncoding(String charset);`。



##  ServletContext域对象

ServletContext类的对象（后续servletContext特指ServletContext类对象）是域对象，就是说这个对象可以进行数据的存取操作，它的作用范围是所有的Servlet。方法如下：

1. void setAttribute(String name, Object o)

2. String getAttribute(String name)

3. void removeAttribute(String name)




## HttpServletRequest

HttpServletRequest是一个接口，封装了获取浏览器发来的信息的方法。部分方法如下：

1. String getMethod()：获得请求方式；

2. String getHeader(String name)：获取某请求头的Value；

3. String getParameter(String name)：获得提交时的某参数的Value；

4. String getRemoteAddr()：获得客户端的IP地址；

5. String getContextPath()：获得web应用的名称；



### 提交乱码解决

1. POST方式：`request.setCharacterEncoding("UTF-8");`

2. GET方式：

   - 假设接收到的参数为param，在使用之前加上：

     ```
     param = new String(param.getBytes("iso8859-1"),"utf-8");
     ```

   - 修改server.xml文件：补上此句

     <div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/ca865411-5bf4-411f-bd91-74b47fd5ef06"></div>



### HttpServletRequest域对象

HttpServletRequest接口的对象引用（request特指HttpServletRequest接口的对象引用）同时也是域对象，就是说这个对象可以进行数据的存取操作，它的作用范围是一次http请求，当此次请求结束（返回一个response时）。方法如下：

1. `void setAttribute(String name, Object o)`
2. `String getAttribute(String name)`
3. `void removeAttribute(String name)`



### 请求转发

当客户端请求某个Servlet的时候，此Servlet交给别的Servlet解决叫做请求转发。请求转发分为两步：

1. 获得转发器：`RequestDispatcher rd = request.getRequestDispatcher(String path)`
2. 转发的时候携带请求和响应：`rd.forward(ServletRequest request, ServletResponse response)`



### 辨析重定向和请求转发

重定向是两次请求服务器，请求转发只有一次请求服务器。重定向可以访问外部资源，转发只能访问内部资源。

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/fd8650c5-1232-4922-bca5-f3e5464c44dd" /></div>
1. 转发的路径：项目中的`url-pattern`
2. 重定向的路径：项目名+项目中的`url-pattern`



### request.get*()方法测试

```java
@WebServlet("/test")
public class Test extends HttpServlet {
	private static final long serialVersionUID = 1L;

	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	
		System.out.println(request.getRequestURL().toString());
		System.out.println(request.getRequestURI().toString());
		System.out.println(request.getServletPath());
		System.out.println(request.getServerPort());
		System.out.println(request.getScheme());
		System.out.println(request.getRemoteUser());
		System.out.println(request.getRemotePort());
		System.out.println(request.getRemoteHost());
		System.out.println(request.getRemoteAddr());
		System.out.println(request.getQueryString());
		System.out.println(request.getProtocol());
		System.out.println(request.getPathTranslated());
		System.out.println(request.getAuthType());
		System.out.println(request.getCharacterEncoding());
		System.out.println(request.getContentLength());
		System.out.println(request.getContentLengthLong());
		System.out.println(request.getContentType());
		System.out.println(request.getContextPath());
		System.out.println(request.getLocalAddr());
		System.out.println(request.getLocalName());
		System.out.println(request.getLocalPort());
		System.out.println(request.getMethod());
		System.out.println(request.getPathInfo());
	
	}
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub
		doGet(request, response);
	}
}
```

客户端IP：10.5.70.87

服务器IP：10.5.69.204

```
getAuthType : null
getCharacterEncoding : null
getContentLength : -1
getContentLengthLong : -1
getContentType : null
getContextPath : /Test-URI		//webapps下的项目文件夹名称
getLocalAddr : 10.5.69.204		//相对Web应用来说是Local，对B/S结构来说是S
getLocalName : DESKTOP-OI1K2LH  //S的名称
getLocalPort : 80				//S的端口号
getMethod : GET
getPathInfo : null
getPathTranslated : null
getProtocol : HTTP/1.1
getQueryString : id=10
getRemoteAddr : 10.5.70.87		//B/S的B的地址
getRemoteHost : 10.5.70.87
getRemotePort : 52897			//B的端口号
getRemoteUser : null
getRequestedSessionId : null
getRequestURI : /Test-URI/test	//相对于项目的地址
getScheme : http				//协议类型
getServerName : 10.5.69.204
getServerPort : 80
getServletPath : /test			//Servlet映射的地址
getAttributeNames : java.util.Collections$3@1994a730
getRequestURL : http://10.5.69.204/Test-URI/test		//浏览器中输入地址
```



## 会话技术

客户端访问服务器的时候，服务器并不知道客户端是谁，因为HTTP协议是无状态的。也就是说A把商品a加入购物车后关闭浏览器之后再来访问服务器时服务器分不出来是不是A访问了服务器。而会话技术就是帮助服务器区分客户端的。用户打开浏览器，访问Web服务器上多个资源，然后关闭浏览器，整个过程称之为一次会话。会话技术分为：Cookie和Session。



### Cookie

Cookie技术是将用户状态的数据存储到客户端的技术。主要方法如下：

- 创建Cookie（不能保存中文信息）：`Cookie c = new Cookie(String name, String value);`

- 设置Cookie在客户端的持久化时间，无持久化时间Cookie在关闭浏览器时信息会销毁： `c.setMaxAge(int seconds);`，时间秒。

- 设置Cookie的携带路径，如果不设置携带路径，那么该Cookie信息会在访问产生该Cookie的 web资源所在的路径都携带Cookie信息，如：

  - `cookie.setPath("/WEB");`		代表访问WEB应用中的任何资源都携带该Cookie
  - `cookie.setPath("/WEB/cookieServlet");` 代表访问WEB中的cookieServlet时才携带Cookie信息

- 向客户端发送Cookie：`response.addCookie(Cookie cookie);`

- 删除客户端的Cookie：使用**同名同路径且持久化时间为0**的Cookie覆盖。

- 服务器获得客户端携带的Cookie：满足条件的Cookie会被自动以请求头的方式发送到服务器端。获得某一Cookie的内容需要两步：

  - 通过request获得所有的Cookie：`Cookie[] cookies = request.getCookies();`

  - 遍历Cookie数组，通过Cookie的名称获得我们想要的Cookie：

    ```java
    for(Cookie cookie : cookies){
    	if(cookie.getName().equal(cookieName)){
    		String cookieValue = cookie.getValue();
    	}
    }
    ```



### Session

Session技术是将用户状态的数据存储到服务器端的技术。服务器会为每个客户端都创建一块内存空间   存储客户的数据，但客户端需要每次都携带一个标识ID（通过Cookie储存在客户端，在Tomcat中叫做JSESSIONID）去服务器中寻找属于自己的内  存空间。同时由于Session具有存储数据的功能，也是一个域对象。主要方法如下：

1. 获得Session对象：`HttpSession session = request.getSession();`。此方法会获得专属于当前会话的Session对象，如果服务器端没有该会话的Session对象会创建一个新的Session返回，如果已经有了属于该会话的Session直接将已有的Session返回（实质就是根据JSESSIONID判断该客户端是否在服务器上已经存在session了）
2. 向Session中存储数据：`session.setAttribute(String name, Object obj);`
3. 从Session中获得数据：`session.getAttribute(String name);`
4. 移除Session中某名称的值：`session.removeAttribute(String name);`



### Session域的声明周期

- 创建：第一次执行request.getSession()时创建。

- 销毁：默认情况下在客户端不再操作服务器端资源时30分钟后session过期。

  1. 手动销毁session： `session.invalidate();`

  2. 也可以在工程的web.xml中进行配置来修改默认过期时间：

     ```xml
     <session-config>
     	<session-timeout>30</session-timeout>
     </session-config>
     ```

- 作用范围：一次会话中任何资源公用一个session对象。



## 获得各种路径

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/6b9e42bf-8e22-400b-97d1-3fa6f9d17ac1"></div>
## 过滤器

```java
@WebFilter(urlPatterns = "/api/*", filterName = "loginFilter")
public class LoginFilter implements Filter {

	/**
	 * 容器加载的时候调用
	 */
	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
		System.out.println("init loginFilter");
	}

	/**
	 * 请求被拦截的时候进行调用
	 */
	@Override
	public void doFilter(ServletRequest servletRequest, 
         ServletResponse servletResponse, FilterChain filterChain)
			throws IOException, ServletException {
		System.out.println("doFilter loginFilter");

		HttpServletRequest req = (HttpServletRequest) servletRequest;
		HttpServletResponse resp = (HttpServletResponse) servletResponse;
		String username = req.getParameter("username");

		if ("xdclass".equals(username)) {
			filterChain.doFilter(servletRequest, servletResponse);
		} else {
			resp.sendRedirect("/index.html");
			return;
		}
	}

	/**
	 * 容器被销毁的时候被调用
	 */
	@Override
	public void destroy() {
		System.out.println("destroy loginFilter");
	}
}
```

多个过滤器的执行顺序是按照filterName按字典排序执行。

| **属性名**      | **类型**       | **描述**                                                     |
| --------------- | -------------- | ------------------------------------------------------------ |
| filterName      | String         | 指定过滤器的 name 属性，等价于`<filter-name>`                |
| value           | String[]       | 该属性等价于 urlPatterns 属性。但是两者不应该同时使用。      |
| urlPatterns     | String[]       | 指定一组过滤器的 URL 匹配模式。等价于`<url-pattern>`标签。   |
| servletNames    | String[]       | 指定过滤器将应用于哪些 Servlet。取值是 @WebServlet 中的 name 属性的取值，或者是 web.xml 中`<servlet-name>`的取值。 |
| dispatcherTypes | DispatcherType | 指定过滤器的转发模式。具体取值包括： ASYNC、ERROR、FORWARD、INCLUDE、REQUEST。 |
| initParams      | WebInitParam[] | 指定一组过滤器初始化参数，等价于`<init-param>`标签。         |
| asyncSupported  | boolean        | 声明过滤器是否支持异步操作模式，等价于`<async-supported>` 标签。 |
| description     | String         | 该过滤器的描述信息，等价于`<description>`标签。              |
| displayName     | String         | 该过滤器的显示名，通常配合工具使用，等价于`<display-name>`标签。 |



## 监听器

常用的监听器 ServletContextListener、HttpSessionListener、ServletRequestListener。都是接口。

```java
@WebListener
public class RequestListener implements ServletRequestListener {
	@Override
	public void requestDestroyed(ServletRequestEvent sre) {
		System.out.println("======requestDestroyed========");
	}

	@Override
	public void requestInitialized(ServletRequestEvent sre) {
		System.out.println("======requestInitialized========");	
	}
}
```



## JDBC

Java Database Connectivity，Java数据库连接。是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。JDBC提供了一种基准，据此可以构建更高级的工具和接口，使数据库开发人员能够编写数据库应用程序，所以说，JDBC对Java程序员而言是API，对实现与数据库连接的服务提供商而言是接口模型。作为API，JDBC为程序开发提供标准的接口，并为数据库厂商及第三方中间件厂商实现与数据库的连接提供了标准方法。

数据库并不是Java提供的，所以在Java中如果想连接数据库，肯定需要使用第三方jar包，这些jar包是数据库厂商根据JDBC接口模型开发的自己数据库的连接包。文章里使用的数据库是mysql，所以使用mysql-connector。



### 基本使用

使用JDBC需要六步：注册驱动、建立连接、创建Statement、执行查询、获得结果集、处理结果集、释放资源。

```java
public class Test {
    public static void main(String[] args) throws Exception {
        Connection con = null;
        Statement statement = null;
        ResultSet query = null;
        try {

            /**
             * 1、注册驱动：在使用JDBC连接连接数据库的时候，Java程序并不知道自己是否连接上了相应的
             *            数据库，所以在第一步需要注册驱动，如果连接正常则可以进行接下来的操作。也
             *            就是说注册驱动这一步是通过我们导入的jar包测试能否正常连接数据库。
             */
            DriverManager.registerDriver(new com.mysql.jdbc.Driver());
    
            /**
             * 2、获得连接：数据库服务器中可能存在多个数据库，我们需要连接上我们即将使用的数据库。
             */
            con = DriverManager.getConnection
                ("jdbc:mysql://localhost/jdbc-study", "root", "root");
    
            /**
             * 3、创建Statement：如果想和数据库进行交互，一定需要使用这个类。JDK对他的解释是：
             *    The object used for executing a static SQL statement and 
             *    returning the results it produces. 
             */
            statement = con.createStatement();
    
            /**
             * 4、执行查询，获得结果集：想数据库中注入SQL语句，是我们能够进行操作
             */
            String sql = "select * from students";
    
            /**
             * 5、处理结果集：query是数据库中的元组集合，可以通过循环获得每个元组
             */
            query = statement.executeQuery(sql);
            while(query.next()){
                String id = query.getString("id");
                String name = query.getString("name");
                String clazz = query.getString("clazz");
                System.out.println(id + "  " + name + "  " + clazz);
            }
        }catch (Exception e) {
            e.printStackTrace();
        }finally {
            /**
             * 6、释放资源：操作结束后进行操作
             */
            if(query != null)
                query.close();
            if(statement != null)
                statement.close();
            if(con != null)
                con.close();
        }
    }
}
```



### 封装工具类

```java
public class JDBCUtil {
	
	static String driverClass = null;
	static String url = null;
	static String name = null;
	static String password= null;
	
	static{
		try {
			//1. 创建一个属性配置对象
			Properties properties = new Properties();
			InputStream is = new FileInputStream("jdbc.properties");
			
			// 使用类加载器，去读取src底下的资源文件。 后面在servlet
			// InputStream is = JDBCUtil.class.getClassLoader().
            //				getResourceAsStream("jdbc.properties");
			//导入输入流。
			properties.load(is);

			//读取属性
			driverClass = properties.getProperty("driverClass");
			url = properties.getProperty("url");
			name = properties.getProperty("name");
			password = properties.getProperty("password");
			
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	private static ThreadLocal<Connection> 
        connectionHolder = new ThreadLocal<Connection>() {
		Connection conn = null;
		public Connection initialValue() {
		    try {
		    	conn = DriverManager.getConnection(url, name, password);
			} catch (SQLException e) {
				e.printStackTrace();
			}
		    return conn;
		}
	};
	
	/**
	 * 获取连接对象
	 * @return
	 */
	public static Connection getConn(){
		Connection conn = null;
		try {
			Class.forName(driverClass);
			conn = connectionHolder.get();
		} catch (Exception e) {
			e.printStackTrace();
		}
		return conn;
	}
	
    // 释放资源
	public static void release(Connection conn , Statement st , ResultSet rs){
		closeRs(rs);
		closeSt(st);
		closeConn(conn);
	}

	//开启事务	
	public static void startTransaction(){
		try{
			Connection conn =  connectionHolder.get();
			conn.setAutoCommit(false);
		}catch (Exception e) {
			throw new RuntimeException(e);
		}
	}
	
	//回滚事务
	public static void rollback(){
		try{
			Connection conn =  connectionHolder.get();
			if(conn != null){
				conn.rollback();
			}
		}catch (Exception e) {
			throw new RuntimeException(e);
		}
	}
    
	//提交事务
	public static void commit(){
		
		try{
			Connection conn =  connectionHolder.get();
			if(conn!=null){
				conn.commit();
			}
		}catch (Exception e) {
			throw new RuntimeException(e);
		}
	}
	
	private static void closeRs(ResultSet rs){
		try {
			if(rs != null){
				rs.close();
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}finally{
			rs = null;
		}
	}
	
	private static void closeSt(Statement st){
		try {
			if(st != null){
				st.close();
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}finally{
			st = null;
		}
	}
	
	private static void closeConn(Connection conn){
		try {
			if(conn != null){
				conn.close();
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}finally{
			conn = null;
		}
	}
}
```

```properties
driverClass=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost/jdbc-study
name=root
password=root
```



### CURD

```java
public class CRUD {

    static Connection con;
    static {
        con = JDBCUtil.getConn();
    }

    @Test
    public void query() throws Exception {
        Statement statement = con.createStatement();
        String sql = "select * from students";
        ResultSet query = statement.executeQuery(sql);
        while(query.next()){
            String id = query.getString("id");
            String name = query.getString("name");
            String clazz = query.getString("clazz");
            System.out.println(id + "  " + name + "  " + clazz);
        }
    }

    @Test
    public void insert() throws Exception {
        Statement statement = con.createStatement();
        String sql = "insert into students(id, name, clazz) values ('160341238', '赵承阳', '160341B')";
        int row = statement.executeUpdate(sql);
    }

    @Test
    public void update() throws Exception {
        Statement statement = con.createStatement();
        String sql = "update students set id = 'helloworld' where id = '160341237'";
        int row = statement.executeUpdate(sql);
    }

    @Test
    public void delete() throws Exception {
        Statement statement = con.createStatement();
        String sql = "delete from students where id = '160341238'";
        //返回处理的行数
        int row = statement.executeUpdate(sql); 
    }
}
```



## PrepareStatement

```java
public static void main(String[] args) throws Exception {
    Statement statement = con.createStatement();
    String qid = "160341238 or 1 = 1";
    String sql = "select * from students where id = " + qid;
    ResultSet query = statement.executeQuery(sql);
    while(query.next()){
    String id = query.getString("id");
    String name = query.getString("name");
    String clazz = query.getString("clazz");
    System.out.println(id + "  " + name + "  " + clazz);
}
    /**Console:
    *      160341238  赵承阳  160341B
    	   aaa  詹金浩  160341B            */
}
```

在上面这段代码中，查询的qid后面添加上了or 1 = 1就可以把表中所有信息都查询出来，因为or 1 = 1这句话是一定为真，而我们刚才使用的Statement又使用的是拼接字符串的方式，在字符串中or会被认为是关键字，所以sql语句的条件永远为真。可以采用PrepareStatement类来解决这个问题。



### PrepareStatement

```java
public static void main(String[] args) throws Exception {
    String sql = "select * from students where id=?";
    PreparedStatement ps = con.prepareStatement(sql);
    /**
    * 从1开始，把字符串填到匹配的?里。关键字也被认为是是字符串
    */
    ps.setString(1, "160341238 or 1 = 1");
    ResultSet query = ps.executeQuery();
    while(query.next()){
    String id = query.getString("id");
    String name = query.getString("name");
    String clazz = query.getString("clazz");
    System.out.println(id + "  " + name + "  " + clazz);
}
	//无结果
```



### 改进的CURD

```java
public class AdvancedCURD {

    static Connection con;
    static {
        con = JDBCUtil.getConn();
    }

    @Test
    public void query() throws Exception {
        String sql = "select * from students where id=?";
        PreparedStatement ps = con.prepareStatement(sql);
        ps.setString(1, "160341238");
        ResultSet query = ps.executeQuery();
        while(query.next()){
            String id = query.getString("id");
            String name = query.getString("name");
            String clazz = query.getString("clazz");
            System.out.println(id + "  " + name + "  " + clazz);
        }
    }

    @Test
    public void insert() throws Exception {
        String sql = "insert into students(id, name, clazz) values (?, ?, ?)";
        PreparedStatement ps = con.prepareStatement(sql);
        ps.setString(1, "160341244");
        ps.setString(2, "qwe");
        ps.setString(3, "160341B");
        int row = ps.executeUpdate();
        System.out.println(row);
    }

    @Test
    public void update() throws Exception {
        String sql = "update students set id = ? where id = ?";
        PreparedStatement ps = con.prepareStatement(sql);
        ps.setString(1, "helloworld");
        ps.setString(2, "160341243");
        int row = ps.executeUpdate();
    }

    @Test
    public void delete() throws Exception {
        String sql = "delete from students where id = ?";
        PreparedStatement ps = con.prepareStatement(sql);
        ps.setString(1, "160341238");
        int row = ps.executeUpdate();
        //返回处理的行数
        System.out.println(row);
    }
}
```



## 数据库连接池

### 自定义数据库连接池

数据库连接池的概念本来就是sun公司提出来的额，所以sun公司针对数据库连接池也提供了一套规范，一个简单的数据库连接池如下，只有获取连接和归还连接的方法：

```java
public class MyDataSource implements DataSource {

    List <Connection> list = new ArrayList<Connection>();
    public  MyDataSource() {
        for (int i = 0; i < 10; i++) {
            Connection conn = JDBCUtil.getConn();
            list.add(conn);
        }
    }

//  该连接池对外公布的获取连接的方法
    @Override
    public Connection getConnection() throws SQLException {
        //来拿连接的时候，先看看，池子里面还有没有。
        if(list.size() == 0 ){
            for (int i = 0; i < 5; i++) {
                Connection conn = JDBCUtil.getConn();
                list.add(conn);
            }
        }
        //remove(0) ---> 移除第一个。 移除的是集合中的第一个。  移除的是开始的那个元素
        Connection conn = list.remove(0);
        return conn;
    }

    /**
     * 用完之后，记得归还。
     * @param conn
     */
    public void addBack(Connection conn){
        list.add(conn);
    }

    //----------------------------
	// other method
}
```

#### 问题

1. AddBack()：这个方法不是接口中的方法，不能使用面向接口的编程。
2. 连接池不是单例：在一个程序中，连接池应该只存在一个，每new一个都会产生一个连接池和n个连接。
3. 扩容：当连接数大于我们设置的数量时需要对连接池中的连接扩容，否则就会产生问题。

#### 解决

1. 使用装饰者模式装饰Connection；
2. 把连接池设为单例；
3. 连接池空时自动增加机制。



### 改进的数据库连接池

```java
public class MyDataSource implements DataSource {
    private MyDataSource() {}
    private static MyDataSource mds = new MyDataSource();
    static List <Connection> list = new ArrayList<Connection>();
    static{
        for (int i = 0; i < 10; i++) {
            Connection conn = JDBCUtil.getConn();
            list.add(conn);
        }
    }

    public static MyDataSource getMyDataSource() {
        return mds;
    }

//  该连接池对外公布的获取连接的方法，扩容
    @Override
    public Connection getConnection() throws SQLException {
        //来拿连接的时候，先看看，池子里面还有没有。
        if(list.size() == 0 ){
            for (int i = 0; i < 5; i++) {
                Connection conn = JDBCUtil.getConn();
                list.add(conn);
            }
        }
        Connection conn = list.remove(0);
        Connection connection = new ConnectionWrap(conn, list);
        return connection;
    }

    /**
     * 用完之后，记得归还。
     * @param conn
     */
    public void addBack(Connection conn){
        list.add(conn);
    }

    //----------------------------
	// other method
}
```

```java
public class ConnectionWrap implements Connection{

    private Connection connection = null;
    private List <Connection> list ;
    public ConnectionWrap(Connection connection, List <Connection> list) {
        this.connection = connection;
        this.list = list;
    }

    @Override
    public void close() throws SQLException {
        list.add(connection);
    }

    @Override
    public PreparedStatement prepareStatement(String sql) throws SQLException {
        return connection.prepareStatement(sql);
    }

    //====================================================================
    //之后的方法没有被装饰，需要使用时再装饰

}
```

```java
public static void main(String[] args) throws SQLException {
	
	MyDataSource myDataSource = MyDataSource.getMyDataSource();
	
	Connection connection = new ConnectionWrap(myDataSource.getConnection(), myDataSource.list);
	
	// ...
	
}
```



### C3P0

```xml
<?xml version="1.0" encoding="UTF-8"?>
<c3p0-config>

    <!-- default-config 默认的配置，  -->
  <default-config>
    <property name="driverClass">com.mysql.jdbc.Driver</property>
    <property name="jdbcUrl">jdbc:mysql://localhost/jdbc-study</property>
    <property name="user">root</property>
    <property name="password">root</property>


    <property name="initialPoolSize">10</property>
    <property name="maxIdleTime">30</property>
    <property name="maxPoolSize">100</property>
    <property name="minPoolSize">10</property>
    <property name="maxStatements">200</property>
  </default-config>

   <!-- This app is massive! -->
  <named-config name="oracle"> 
    <property name="acquireIncrement">50</property>
    <property name="initialPoolSize">100</property>
    <property name="minPoolSize">50</property>
    <property name="maxPoolSize">1000</property>

    <!-- intergalactoApp adopts a different approach to configuring statement caching -->
    <property name="maxStatements">0</property> 
    <property name="maxStatementsPerConnection">5</property>

    <!-- he's important, but there's only one of him -->
    <user-overrides user="master-of-the-universe"> 
      <property name="acquireIncrement">1</property>
      <property name="initialPoolSize">1</property>
      <property name="minPoolSize">1</property>
      <property name="maxPoolSize">5</property>
      <property name="maxStatementsPerConnection">50</property>
    </user-overrides>
  </named-config>
</c3p0-config>
```

```java
public class Test {
    public static void main(String[] args) {

        Connection connection = null;
        PreparedStatement ps = null;
        try {
            //1、构建数据源
            ComboPooledDataSource cpds = new ComboPooledDataSource();

            //2、得到连接对象
            connection = cpds.getConnection();

            //3、执行sql语句
            String sql = "select * from students";
            ps = connection.prepareStatement(sql);

            //4、获得结果、处理结果
            ResultSet query = ps.executeQuery();
            while(query.next()) {
                String id = query.getString("id");
                String name = query.getString("name");
                String clazz = query.getString("clazz");
                System.out.println(id + " " + name + " " + clazz);
            }
        }catch (Exception e) {
            e.printStackTrace();
        }finally {
            try {
                ps.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```



## Dbutils

### 增删改

```java
import org.apache.commons.dbutils.QueryRunner;

import com.mchange.v2.c3p0.ComboPooledDataSource;

public class Test {

    public static void main(String[] args) throws Exception {

        //1、指定数据库连接池
        ComboPooledDataSource dataSource = new ComboPooledDataSource();

        //2、增删改都使用update方法
        QueryRunner qr = new QueryRunner(dataSource);
        String insert = "insert into students(id, name, clazz) values(?, ?, ?)";
        qr.update(insert, "123", "123", "123");

        String update = "update students set name = ? where id = ?";
        qr.update(update, "789", "123");

        String delete = "delete from students where id = ?";
        qr.update(delete, "123");
    }
}
```



### 查询

#### 自定义封装-返回值类型是单个对象

```java
String query = "select * from students where id = ?";

Student student = qr.query(query, new ResultSetHandler<Student>() {
    public Student handle(ResultSet rs) throws SQLException {
        Student s = new Student();
        while(rs.next()) {
            String id = rs.getString("id");
            String name = rs.getString("name");
            String clazz = rs.getString("clazz");
            s.setId(id);
            s.setName(name);
            s.setClazz(clazz);
        }
        return s;
    }
}, "helloworld");

System.out.println(student);
```

#### 自定义封装-返回值类型是集合

```java
String query = "select * from students";
List<Student> list = qr.query(query, new ResultSetHandler<List<Student>>() {
    public List<Student> handle(ResultSet rs) throws SQLException {
        List<Student> l = new ArrayList<>();
        Student s = new Student();
        while(rs.next()) {
            s.setId(rs.getString("id"));
            s.setName(rs.getString("name"));
            s.setClazz(rs.getString("clazz"));
            l.add(s);
        }
        return l;
    }
});
System.out.println(list);
```

快速封装-返回值类型是单个对象

```java
String query = "select * from students where id = ?";
Student q = qr.query(query, new BeanHandler<>(Student.class), "helloworld");
System.out.println(q);
/**Console:
*   Student [id=helloworld, name=qwe, clazz=160341B]
*/
```

#### 快速封装-返回值类型是多个对象

```java
String query = "select * from students";
List<Student> l = qr.query(query, new BeanListHandler<>(Student.class));
System.out.println(l);
/**
 * [Student [id=160341240, name=qwe, clazz=160341B], 
 *      Student [id=160341244, name=qwe, clazz=160341B], 
 *          Student [id=aaa, name=詹金浩, clazz=160341B], 
 *              Student [id=helloworld, name=qwe, clazz=160341B]]
 */
```



