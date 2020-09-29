## IOC容器

### IOC

假如有这么一个需要，查询并展示指定导演拍摄的电影。设计流程如下：

1、首先有一个类是提供给客户的，这个类是一个UI类，为了简便，Demo里使用控制台。

```java
public class MovieLister {
    private MovieFinder finder = new ColonDelimitedMovieFinder();
    public void showMoviesDirectedBy(String director) {
        List allMovies = finder.findAll();
        for (Iterator it = allMovies.iterator(); it.hasNext(); ) {
            Movie movie = (Movie) it.next();
            if (!movie.getDirector().equals(director)) {
                it.remove();
            }
        }
        System.out.println(allMovies.toArray());
    }
}
```

2、第二个类是一个实体类，代表电影。

```java
public class Movie {
    public String getDirector(){
        // 具体实现略
        return "";
    }
}
```

3、第三个类就是查询类，它应当是一个接口，这样的话我们才可以从不同的存储介质里获取电影的信息。

```java
public interface MovieFinder {
    List findAll();
}
```

4、我们给出一个实现类，假设电影信息保存在逗号分隔的文本文件中。

```java
public class ColonDelimitedMovieFinder implements MovieFinder {
    @Override
    public List findAll() {
        return null;
    }
}
```

这样的话，当我们在使用这个功能的时候客户端可以这么编写。

```java
public class Client {
    public static void main(String[] args) {
        MovieLister movieLister = new MovieLister();
        movieLister.ShowMoviesDirectedBy("wangwu");
    }
}
```

我们可以看见，在整个功能的实现中，`MovieLister` 类依赖了 `MovieFinder` 接口，这时候符合依赖倒置的原则。此时我们分析一下三者之间的依赖关系。

<div align="center"><img width="30%" src="http://blogfileqiniu.isjinhao.site/ae8e695a-b5fa-4883-b979-59ddac839f4b" /></div>

从图中可以看见，依赖关系非常的复杂，如果所有的类都是同一个人编写，那么没有什么问题，但是如果是由两个并不认识的人写这个功能，比如王五写了主要的结构，赵六写了一个通过爬虫来获取电影的 `CrawlerFinder`，`CrawlerFinder` 和其他模块该怎么工作呢？有人会说直接new一个 `CrawlerFinder` 的实例赋值给finder不就行了吗？注意，请不要站在上帝视角看待问题，假设王五已经使用`ColonDelimitedMovieFinder` 完成了这个功能，然后才发现了赵六的 `CrawlerFinder`。此时用 `new CrawlerFinder()` 替换代码里的`new ColonDelimitedMovieFinder()` 会违反开闭原则。

**控制反转中反转的是依赖的获取过程。**对于Demo来说，目前是高层模块通过new一个低层模块的实现得到的。这样就会引发一个问题：对于你不知道的实现就无法处理。所以我们要留下一个拓展点，并能通过修改拓展点的实现来更换低层模块的实现。很容易想到的方法就是通过构造器或setter方法设置依赖。此时 `MovieLister` 的设计如下。

```java
public class MovieLister {
    private MovieFinder finder = new ColonDelimitedMovieFinder();
    public MovieLister() { }
    public MovieLister(MovieFinder finder) {
        this.finder = finder;
    }
    public void setFinder(MovieFinder finder) {
        this.finder = finder;
    }
}
```

但是我们客户端不能向下面这样修改。因为这样违反了最小知道原则，即客户端不应该知道 `MovieFinder` 的实现，所以此时需要一个第三方组件来实现这个调用构造器或者setter方法的功能。这个第三方组件就是IOC容器。

```java
public class Client {
    public static void main(String[] args) {
        MovieLister movieLister = new MovieLister(new CrawlerFinder());
        movieLister.ShowMoviesDirectedBy("wangwu");
    }
}
```

引入IOC容器之后的依赖图：

<div align="center"><img width="40%" src="http://blogfileqiniu.isjinhao.site/5d662654-3613-48df-bf22-f3c4df6ba7df" /></div>

引入IOC容器之后就会有两种获取依赖的方式，一种是模块向IOC容器要依赖；一种是IOC容器给模块注入依赖，当然，这种方式需要给模块编写一个配置文件，文件会规定注入什么依赖。前者就是依赖查找（Dependency Lookup），后者是依赖注入（Dependency Injection）。

对于使用Spring的人来说，经常编写bean的配置文件：

```xml
<beans>
    <bean id="MovieLister" class="spring.MovieLister">
        <!-- 依赖注入 -->
        <property name="finder" ref="MovieFinder" />
    </bean>
    <bean id="MovieFinder" class="spring.ColonDelimitedMovieFinder" />
</beans>
```

或者使用@Autowired：

```java
@Autowired
private MovieFinder finder = new ColonDelimitedMovieFinder();
```

这两种都是依赖注入。

当然Spring也提供了对依赖查找的支持。下面便是一个拓展ApplicationContextAware接口实现的查找bean的工具类。

```java
@Component
public class SpringUtils implements ApplicationContextAware {
    private static ApplicationContext applicationContext; 
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        SpringUtils.applicationContext = applicationContext;
    }
    public static <T> T getBean(String beanName) {
        if(applicationContext.containsBean(beanName)){
            return (T) applicationContext.getBean(beanName);
        }else{
            return null;
        }
    }
    public static <T> Map<String, T> getBeansOfType(Class<T> baseType){
        return applicationContext.getBeansOfType(baseType);
    }
}
```

MartinFowler关于IOC和DI的解释：https://martinfowler.com/articles/injection.html

### DIP

依赖倒置原则的定义是：

1. 高层模块不应该依赖底层模块，两者都应该依赖抽象接口。
2. 抽象接口不应该依赖具体实现，而具体实现需要依赖抽象接口。

但是这个定义没有说出来依赖倒置的根本思想。我们设想一下，如果两个模块有两个团队进行开发，那么势必会有高低之分，依赖倒置的重心是在讨论接口由谁来定义，是两个模块中的低层模块定义还是高层模块定义？按照传统的开发思维，自然是低层模块定义，因为低层模块是实现这些接口，而依赖倒置原则倒置的便是接口的定义，其将接口的定义交给了高层模块。

为什么要这么做呢？因为底层模块并不知道高层模块的需求，如果让低层模块定义接口，势必会造成定义的接口很难被高层模块使用的问题。所以高层模块定义完接口后，底层模块去实现才是好的开发模式。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/f11688fd-0214-4cf0-8451-5acdb2ece3fe" /></div>

那么，引入IOC容器之后，接口是谁定义呢？自然是IOC容器定义，因为引入IOC容器之后，IOC容器便是系统的高级模块，其他的都是底层模块。比如在Spring和Mybatis的整合中，Mybatis为了接入Spring，需要引入mybatis-spring项目才可以。

#### DIP，DI，IOC的关系

DIP和IOC都是设计的基本思想。DI是IOC的实现手段。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/799552bb-eb80-442b-b4b5-7bc8990cb0dd" /></div>



## Spring IOC入门

### Dependency Lookup

spring配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="user" class="ico.domain.User">
        <property name="id" value="1" />
        <property name="name" value="isjinhao" />
    </bean>

    <!-- 不加primary="true"在按类型查找时会报错，因为SuperUser也是User类型，
		加上primary="true"后Spring就知道同类型有多个时寻找这个 -->
    <bean id="superUser" class="ico.domain.SuperUser" parent="user" primary="true">
        <property name="address" value="深圳" />
    </bean>

    <bean id="objectFactoryUser" 
          class="org.springframework.beans.factory.config.ObjectFactoryCreatingFactoryBean">
        <property name="targetBeanName" value="user"/>
    </bean>

</beans>
```

测试用的注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface SuperUserAnnotation {

}
```

实体类

```java
public class User {
    private Long id;
    private String name;
    // getter and setter
}

@SuperUserAnnotation
public class SuperUser extends User {
    private String address;
    // getter and setter
}
```

依赖查找Demo

```java
public class DependencyLookupDemo {
    public static void main(String[] args) {
        BeanFactory beanFactory = 
        	new ClassPathXmlApplicationContext("META-INF/dependency-lookup-context.xml");
        lookupRealTimeByName(beanFactory);
        System.out.println("-----------------------");
        lookupLazyByName(beanFactory);
        System.out.println("-----------------------");
        lookupByType(beanFactory);
        System.out.println("-----------------------");
        lookupCollectionByType(beanFactory);
        System.out.println("-----------------------");
        lookupByAnnotation(beanFactory);
    }

    private static void lookupByAnnotation(BeanFactory beanFactory) {
        if (beanFactory instanceof ListableBeanFactory) {
            ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
            Map<String, User> users = 
                (Map) listableBeanFactory.getBeansWithAnnotation(SuperUserAnnotation.class);
            System.out.println("查找标注 @SuperUserAnnotation 所有的 User 集合对象：" + users);
        }
    }
    private static void lookupCollectionByType(BeanFactory beanFactory) {
        if (beanFactory instanceof ListableBeanFactory) {
            ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
            Map<String, User> users = listableBeanFactory.getBeansOfType(User.class);
            System.out.println("查找到的所有的 User 集合对象：" + users);
        }
    }
    private static void lookupByType(BeanFactory beanFactory) {
        User user = beanFactory.getBean(User.class);
        System.out.println("loadByType: " + user);
    }
    private static void lookupLazyByName(BeanFactory beanFactory) {
        ObjectFactory<User> userObjectFactory = 
            (ObjectFactory<User>) beanFactory.getBean("objectFactoryUser");
        User user = userObjectFactory.getObject();
        System.out.println("lazyLoadByName: " + user);
    }
    private static void lookupRealTimeByName(BeanFactory beanFactory) {
        User user = (User) beanFactory.getBean("user");
        System.out.println("realTimeLoadByName: " + user);
    }
}
```



### Dependency Injection

spring配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/util
        https://www.springframework.org/schema/util/spring-util.xsd">

    <import resource="dependency-lookup-context.xml"/>

    <!-- 使用autowire就是开启注入了 -->
    <bean id="userRepository" class="ico.repository.UserRepository" autowire="byType">
<!--        <property name="userList">-->
<!--            <util:list>-->
<!--                <ref bean="user"/>-->
<!--                <ref bean="superUser"/>-->
<!--            </util:list>-->
<!--        </property>-->
    </bean>
</beans>
```

被注入的bean

```java
public class UserRepository {
    private List<User> userList;
    private BeanFactory beanFactory;
    private ApplicationContext applicationContext;
    private ObjectFactory<User> objectFactory;
    private Environment environment;
}
```

依赖注入Demo

```java
public class DependencyInjectionDemo {
    public static void main(String[] args) {
        /**
         * Spring的bean来源有三种情况
         */
        BeanFactory beanFactory = 
            new ClassPathXmlApplicationContext("META-INF/dependency-injection-context.xml");
        UserRepository userRepository = beanFactory.getBean("userRepository", UserRepository.class);

        // 注入集合类型（用户自定义的spring bean）
        System.out.println(userRepository.getUserList());
        // 注入spring内置的bean
        System.out.println(userRepository.getEnvironment());
        // 注入非bean类型的依赖
        System.out.println(userRepository.getBeanFactory());
        // 延迟加载对象
        System.out.println(userRepository.getObjectFactory().getObject());
    }
}
```



### BeanFactory和ApplicationContext

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/71ddef69-f5ae-47ea-b088-f7089864ff9f" /></div>

从类图上看，ApplicationContext是BeanFactory的子接口，那么Demo里的ApplicationContext和BeanFactory就是同一个对象吗？我们测试一下：

```java
public class DifferenceBetweenBeanFactoryAndApplicationContext {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext classPathXmlApplicationContext = 
        	new ClassPathXmlApplicationContext("META-INF/dependency-injection-context.xml");
        UserRepository userRepository = 
        	classPathXmlApplicationContext.getBean("userRepository", UserRepository.class);

        System.out.println("classPathXmlApplicationContext：" + classPathXmlApplicationContext);
        // 底层的IOC容器
        System.out.println("beanFactory" + userRepository.getBeanFactory());
        // 注入的ApplicationContext
        System.out.println(userRepository.getApplicationContext());
        System.out.println(classPathXmlApplicationContext.getBeanFactory());
    }
}
```

实际上是，AbstractRefreshableApplicationContext里组合一个beanFactory。这个beanFactory是注入到UserRepository里的beanFactory。

```java
public abstract class AbstractRefreshableApplicationContext extends AbstractApplicationContext {
    private DefaultListableBeanFactory beanFactory;
    public final ConfigurableListableBeanFactory getBeanFactory() {
        synchronized (this.beanFactoryMonitor) {
            if (this.beanFactory == null) {
                throw new IllegalStateException("BeanFactory not initialized or already closed - " +
                	"call 'refresh' before accessing beans via the ApplicationContext");
            }
            return this.beanFactory;
        }
    }
}
```

一系列的getBean()方法都是调用组合的beanFactory获得的。

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
		implements ConfigurableApplicationContext {
    // 这些getBean方法都是从BeanFactory继承下来的。
    @Override
	public Object getBean(String name) throws BeansException {
		assertBeanFactoryActive();
		return getBeanFactory().getBean(name);
	}

	@Override
	public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
		assertBeanFactoryActive();
		return getBeanFactory().getBean(name, requiredType);
	}
}
```



### 使用BeanFactory作为IOC容器

之前看到的都是采用ApplicationContext作为IOC容器，所以写一个使用BeanFactory作为IOC容器的Demo。

```java
public class BeanFactoryAsIoCContainerDemo {
    public static void main(String[] args) {
        // 创建 BeanFactory 容器
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);
        // XML 配置文件 ClassPath 路径
        String location = "META-INF/dependency-injection-context.xml";
        // 加载配置
        int beanDefinitionsCount = reader.loadBeanDefinitions(location);
        System.out.println("Bean 定义加载的数量：" + beanDefinitionsCount);
        // 依赖查找集合对象
        lookupCollectionByType(beanFactory);
    }
    private static void lookupCollectionByType(BeanFactory beanFactory) {
        if (beanFactory instanceof ListableBeanFactory) {
            ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
            Map<String, User> users = listableBeanFactory.getBeansOfType(User.class);
            System.out.println("查找到的所有的 User 集合对象：" + users);
        }
    }
}
```



### 使用注解配置bean

强大的Spring自然是提供了使用注解配置bean的功能，下面是一个Demo。

```java
@Configuration
public class AnnotationApplicationContextAsIoCContainerDemo {
    public static void main(String[] args) {
        // 创建 BeanFactory 容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        applicationContext.scan("ico.dependency.beansource");
        // 启动应用上下文
        applicationContext.refresh();
        // 依赖查找集合对象
        lookupCollectionByType(applicationContext);
        // 关闭应用上下文
        applicationContext.close();
    }

    /**
     * 通过 Java 注解的方式，定义了一个 Bean
     */
    @Bean
    public User user() {
        User user = new User();
        user.setId(1L);
        user.setName("小马哥");
        return user;
    }

    private static void lookupCollectionByType(BeanFactory beanFactory) {
        if (beanFactory instanceof ListableBeanFactory) {
            ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
            Map<String, User> users = listableBeanFactory.getBeansOfType(User.class);
            System.out.println("查找到的所有的 User 集合对象：" + users);
        }
    }
}
```



## Spring Bean

### BeanDefiniton

Spring定义bean是通过类BeanDefiniton完成的，下面演示的时Java API的Demo。

```java
public class BeanDefinitionDemo {
    public static void main(String[] args) {
        // 1、通过BeanDefinitionBuilder
        BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(User.class);
        beanDefinitionBuilder.addPropertyValue("id", 1);
        beanDefinitionBuilder.addPropertyValue("name", "isjinhao");
        AbstractBeanDefinition beanDefinition = beanDefinitionBuilder.getBeanDefinition();
        System.out.println(beanDefinition.getClass());
        System.out.println(beanDefinition);

        // 2、通过 AbstractBeanDefinition 以及派生类实现
        //    实际上 BeanDefinitionBuilder 底层也是使用的 AbstractBeanDefinition
        GenericBeanDefinition genericBeanDefinition = new GenericBeanDefinition();
        MutablePropertyValues propertyValues = new MutablePropertyValues();
//        propertyValues.add("id", 1);
//        propertyValues.add("name", "isjinhao");
        propertyValues = propertyValues.add("id", 1).add("name", "isjinhao");
        genericBeanDefinition.setPropertyValues(propertyValues);
        System.out.println(genericBeanDefinition);
    }
}
```



### 别名

```java
public class BeanAliasDemo {
    public static void main(String[] args) {
        BeanFactory beanFactory = new ClassPathXmlApplicationContext("META-INF/bean-definitions-context.xml");

        User isjinhao = beanFactory.getBean("isjinhao-user", User.class);
        User user = beanFactory.getBean("user", User.class);
        System.out.println(isjinhao);
        System.out.println(user);
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 导入第三方 Spring XML 配置文件 -->
    <!-- 跨项目时这里必须要加classpath前缀   -->
    <import resource="classpath:/META-INF/dependency-lookup-context.xml" />

    <!-- 将 Spring 容器中 "user" Bean 关联/建立别名 - "isjinhao-user" -->
    <!-- 已经有名称还定义别名的作用是： -->
    <alias name="user" alias="isjinhao-user" />

</beans>
```



### 实例化Bean

```java
public class BeanInstantiationDemo {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = 
            new ClassPathXmlApplicationContext("META-INF/bean-instantiation-context.xml");
//        User staticUser = applicationContext.getBean("user-by-static-method", User.class);
//        System.out.println("-----------------");
//        User instanceUser = applicationContext.getBean("user-by-instance-method", User.class);
//        System.out.println("-----------------");
        User factoryUser = applicationContext.getBean("user-by-factory-bean", User.class);
//        System.out.println(staticUser);
//        System.out.println(instanceUser);
        System.out.println(factoryUser);
//        System.out.println(staticUser == instanceUser);
//        System.out.println(staticUser == factoryUser);
        applicationContext.close();
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 静态方法实例化 Bean -->
    <!--  <bean id="user-by-static-method" class="ico.domain.User" factory-method="createUser"/>-->

    <!-- 实例（Bean）方法实例化 Bean -->
    <!--  <bean id="user-by-instance-method" class="ico.domain.User" factory-bean="userFactory" 
				factory-method="createUser"/>-->
    <!--  <bean id="userFactory" class="sb.bean.UserFactory"/>-->

    <!--  &lt;!&ndash; FactoryBean实例化 Bean &ndash;&gt;-->
    <bean id="user-by-factory-bean" class="sb.bean.UserFactoryBean"/>
</beans>
```

```java
public class UserFactoryBean {
    @Override
    public User getObject() throws Exception {
        User user = new User();
        user.setId(1l);
        user.setName("isjinhao");
        return user;
    }
    @Override
    public Class<?> getObjectType() {
        return User.class;
    }
}
```

```java
public class User implements InitializingBean, DisposableBean {
    private Long id;
    private String name;
    // getter and setter
    public static User createUser() {
        User user = new User();
        user.setId(1l);
        user.setName("isjinhao");
        return user;
    }
}
```

```java
public class UserFactory implements UserFactoryInterface {
    @Override
    public User createUser() {
        User user = new User();
        user.setId(1l);
        user.setName("isjinhao");
        return user;
    }
}
```

```java
public interface UserFactoryInterface {
    User createUser();
}
```

#### SPI接口实现

```java
public class SpecialBeanInstantiationDemo {

    public static void main(String[] args) {
        ApplicationContext applicationContext = 
            new ClassPathXmlApplicationContext("META-INF/special-bean-instantiation-context.xml");
        AutowireCapableBeanFactory capableBeanFactory = applicationContext.getAutowireCapableBeanFactory();
        UserFactory userFactory = capableBeanFactory.createBean(UserFactory.class);
        System.out.println(userFactory.createUser());
        ServiceLoader serviceLoader = applicationContext.getBean("userFactoryServiceLoader", ServiceLoader.class);
        displayServiceLoader(serviceLoader);
        System.out.println("---------------------------");
        demoServiceLoader();
    }

    public static void demoServiceLoader() {
        ServiceLoader<UserFactoryInterface> serviceLoader = 
            ServiceLoader.load(UserFactoryInterface.class, Thread.currentThread().getContextClassLoader());
        displayServiceLoader(serviceLoader);
    }

    public static void displayServiceLoader(ServiceLoader serviceLoader) {
        Iterator<UserFactoryInterface> iterator = serviceLoader.iterator();
        while (iterator.hasNext()) {
            UserFactoryInterface userFactoryInterface = iterator.next();
            System.out.println(userFactoryInterface.createUser());
        }
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userFactoryServiceLoader" class="org.springframework.beans.factory.serviceloader.ServiceLoaderFactoryBean">
        <property name="serviceType" value="sb.bean.UserFactory" />
    </bean>
</beans>
```

```
// META-INF/services/sb.bean.UserFactoryInterface
sb.bean.UserFactory
```



### 拦截Bean的初始化和销毁

```java
public class BeanInitializationDemo {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        applicationContext.register(BeanInitializationDemo.class);
        applicationContext.refresh();
        System.out.println("Spring上下文已经启动");
        
        User user = applicationContext.getBean(User.class);
        System.out.println(user);

        System.out.println("Spring上下文准备关闭");
        applicationContext.close();
        System.out.println("Spring上下文已经关闭");
    }
    
    @Bean(initMethod = "initUser", destroyMethod = "destroyUser")
    public User user() {
        User user = new User();
        user.setName("isjinhao");
        user.setId(100l);
        return new User();
    }
}
```

```java
public class User implements InitializingBean, DisposableBean {
    private Long id;
    private String name;
   	// getter and setter
    public static User createUser() {
        User user = new User();
        user.setId(1l);
        user.setName("isjinhao");
        return user;
    }
    @PostConstruct
    public void init() {
        System.out.println("@PostConstruct : User 初始化中 ... ");
    }
    @PreDestroy
    public void preDestroy() {
        System.out.println("@PreDestroy : User 销毁中...");
    }
    public void initUser() {
        System.out.println("@Bean(initMethod = \"initUser\") : User 初始化中 ... ");
    }
    public void destroyUser() {
        System.out.println("@Bean(destroyMethod = \"destroyUser\") : User 销毁中...");
    }
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("InitializingBean接口 : User 初始化中 ... ");
    }
    @Override
    public void destroy() throws Exception {
        System.out.println("DisposableBean接口 : User 销毁中...");
    }
    @Override
    protected void finalize() throws Throwable {
        System.out.println("User 对象正在被回收" );
    }
}
```



### 注册

#### BeanFactory

BeanFactory的实现类DefaultListableBeanFactory实现了BeanDefinitionRegistry。

```java
public class BeanFactoryRegisterBeanDefinitionDemo {
    public static void main(String[] args) {
        // 创建 BeanFactory 容器
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        registerBeanDefinition(beanFactory, "isjinhao");
        registerBeanDefinition(beanFactory);

        Map<String, User> beansOfType = beanFactory.getBeansOfType(User.class);
        System.out.println(beansOfType);
    }

    // 将 User 注册到指定的 BeanRegistry 中
    public static void registerBeanDefinition(BeanDefinitionRegistry registry, String beanName) {
        BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(User.class);
        beanDefinitionBuilder.addPropertyValue("id", 1);
        beanDefinitionBuilder.addPropertyValue("name", "isjinhao");
        AbstractBeanDefinition beanDefinition = beanDefinitionBuilder.getBeanDefinition();
        if (StringUtils.hasText(beanName)) {
            registry.registerBeanDefinition(beanName, beanDefinition);
        } else {
            // 默认的bean名称
            BeanDefinitionReaderUtils.registerWithGeneratedName(beanDefinition, registry);
        }
    }

    public static void registerBeanDefinition(BeanDefinitionRegistry registry) {
        registerBeanDefinition(registry, null);
    }
}
```

#### ApplicationContext

ApplicationContext的实现类GenericApplicationContext实现了BeanDefinitionRegistry。

```java
public class ApplicationContextBeanDefinitionDemo {
    public static void main(String[] args) {
        // 创建 ApplicationContext 容器，GenericApplicationContext实现了BeanDefinitionRegistry
        GenericApplicationContext applicationContext = new GenericApplicationContext();

        registerBeanDefinition(applicationContext, "isjinhao");
        registerBeanDefinition(applicationContext);

        // 启动应用上下文
        applicationContext.refresh();
        // 依赖查找集合对象
        Map<String, User> beansOfType = applicationContext.getBeansOfType(User.class);
        System.out.println(beansOfType);
        // 关闭应用上下文
        applicationContext.close();
    }

    // 将 User 注册到指定的 BeanRegistry 中
    public static void registerBeanDefinition(BeanDefinitionRegistry registry, String beanName) {
        BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(User.class);
        beanDefinitionBuilder.addPropertyValue("id", 1);
        beanDefinitionBuilder.addPropertyValue("name", "isjinhao");
        AbstractBeanDefinition beanDefinition = beanDefinitionBuilder.getBeanDefinition();
        if(StringUtils.hasText(beanName)) {
            registry.registerBeanDefinition(beanName, beanDefinition);
        } else {
            // 默认的bean名称
            BeanDefinitionReaderUtils.registerWithGeneratedName(beanDefinition, registry);
        }
    }

    public static void registerBeanDefinition(BeanDefinitionRegistry registry) {
        registerBeanDefinition(registry, null);
    }
}
```

#### Spring对注解的支持

AnnotationApplicationContext继承了GenericApplicationContext，实现了对注解的支持。

```java
// 导入一个配置类，作用类似于xml配置中的 <import> 标签。
@Import(AnnotationApplicationContextEnhanceDemo.Config.class)
public class AnnotationApplicationContextEnhanceDemo {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        // 将当前类作为配置类
        applicationContext.register(AnnotationApplicationContextEnhanceDemo.class);
//        applicationContext.scan("sb.bean.register");

        // 启动应用上下文
        applicationContext.refresh();
        // 依赖查找集合对象
        Map<String, User> beansOfType = applicationContext.getBeansOfType(User.class);
        System.out.println(beansOfType);
        // 关闭应用上下文
        applicationContext.close();
    }

    public static class Config {
        @Bean(name = {"user", "isjinhao"})
        public User user() {
            User user = new User();
            user.setId(1L);
            user.setName("小马哥");
            return user;
        }
    }
}
```

#### Bean的回收

ApplicationContext关闭之后，Bean是会被回收的。

```java
public class BeanGarbageCollectionDemo {
    public static void main(String[] args) throws InterruptedException {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        applicationContext.register(BeanInitializationDemo.class);
        applicationContext.refresh();
        User user = applicationContext.getBean(User.class);

        System.out.println("Spring上下文准备关闭");
        applicationContext.close();
        System.out.println("Spring上下文已经关闭");

        Thread.sleep(10000L);
        System.gc();
        Thread.sleep(10000L);
    }
}
```

