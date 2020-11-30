## Spring IOC入门

Spring IOC的目的就是管理Bean，所以本节的目的就是学习如何运用Spring完成依赖查找和依赖注入。



### Dependency Lookup

spring配置文件：dependency-lookup-context.xml

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
        AnnotationConfigApplicationContext applicationContext = 
            						new AnnotationConfigApplicationContext();
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
        BeanFactory beanFactory = 
            new ClassPathXmlApplicationContext("META-INF/bean-definitions-context.xml");

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

### Bean的回收

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









