## ApplicationContext Bean

### ApplicationContext启动

#### registerBeanPostProcessors

```java
public static void registerBeanPostProcessors(
    ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {

    String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);

    // Register BeanPostProcessorChecker that logs an info message when
    // a bean is created during BeanPostProcessor instantiation, i.e. when
    // a bean is not eligible for getting processed by all BeanPostProcessors.
    int beanProcessorTargetCount = 
        beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
    beanFactory.addBeanPostProcessor(
        new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));

    // Separate between BeanPostProcessors that implement PriorityOrdered,
    // Ordered, and the rest.
    List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
    List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();
    List<String> orderedPostProcessorNames = new ArrayList<>();
    List<String> nonOrderedPostProcessorNames = new ArrayList<>();
    for (String ppName : postProcessorNames) {
        if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
            BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
            priorityOrderedPostProcessors.add(pp);
            if (pp instanceof MergedBeanDefinitionPostProcessor) {
                internalPostProcessors.add(pp);
            }
        }
        else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
            orderedPostProcessorNames.add(ppName);
        }
        else {
            nonOrderedPostProcessorNames.add(ppName);
        }
    }

    // First, register the BeanPostProcessors that implement PriorityOrdered.
    sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);

    // Next, register the BeanPostProcessors that implement Ordered.
    List<BeanPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
    for (String ppName : orderedPostProcessorNames) {
        BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
        orderedPostProcessors.add(pp);
        if (pp instanceof MergedBeanDefinitionPostProcessor) {
            internalPostProcessors.add(pp);
        }
    }
    sortPostProcessors(orderedPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, orderedPostProcessors);

    // Now, register all regular BeanPostProcessors.
    List<BeanPostProcessor> nonOrderedPostProcessors = 
        new ArrayList<>(nonOrderedPostProcessorNames.size());
    for (String ppName : nonOrderedPostProcessorNames) {
        BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
        nonOrderedPostProcessors.add(pp);
        if (pp instanceof MergedBeanDefinitionPostProcessor) {
            internalPostProcessors.add(pp);
        }
    }
    registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);

    // Finally, re-register all internal BeanPostProcessors.
    sortPostProcessors(internalPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, internalPostProcessors);

    // Re-register post-processor for detecting inner beans as ApplicationListeners,
    // moving it to the end of the processor chain (for picking up proxies etc).
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
}
```

在注册用户自定义的BeanPostProcessor之前已经注册过两个内置的BeanPostProcessor：ApplicationContextAwareProcessor和ApplicationListenerDetector。

BeanPostProcessor也是有顺序的：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd" default-autowire="byType">
    <bean class="fsc.domain.User" id="test-user"></bean>

    <bean class="ab.order.BeanProcessorOrderedDemo.MyBeanPostProcessor1"></bean>
    <bean class="ab.order.BeanProcessorOrderedDemo.MyBeanPostProcessor2"></bean>
    <bean class="ab.order.BeanProcessorOrderedDemo.MyBeanPostProcessor3"></bean>
    <bean class="ab.order.BeanProcessorOrderedDemo.MyBeanPostProcessor4"></bean>
    <bean class="ab.order.BeanProcessorOrderedDemo.MyBeanPostProcessor5"></bean>
</beans>
```

```java
public class BeanProcessorOrderedDemo {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext =
                new ClassPathXmlApplicationContext(
            		new String[]{"META-INF/bean-bean-order-context.xml"}, false, null);
        applicationContext.refresh();
        System.out.println("refresh finish!");
        System.out.println(applicationContext.getBean("test-user"));
        applicationContext.close();
    }

    static class MyBeanPostProcessor1 implements BeanPostProcessor, Ordered {
        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) 
            	throws BeansException {
            System.out.println("order 0  " + beanName);
            return bean;
        }
        @Override
        public int getOrder() {
            return 0;
        }
    }

    static class MyBeanPostProcessor2 implements BeanPostProcessor, Ordered {
        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) 
            	throws BeansException {
            System.out.println("order 1  " + beanName);
            return bean;
        }
        @Override
        public int getOrder() {
            return 1;
        }
    }

    static class MyBeanPostProcessor3 implements BeanPostProcessor, PriorityOrdered {
        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) 
            	throws BeansException {
            System.out.println("priority 0  " + beanName);
            return bean;
        }
        @Override
        public int getOrder() {
            return 0;
        }
    }

    static class MyBeanPostProcessor4 implements BeanPostProcessor, PriorityOrdered {
        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) 
            	throws BeansException {
            System.out.println("priority 1  " + beanName);
            return bean;
        }
        @Override
        public int getOrder() {
            return 1;
        }
    }

    static class MyBeanPostProcessor5 implements BeanPostProcessor {
        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) 
            	throws BeansException {
            System.out.println("no order  " + beanName);
            return bean;
        }
    }
}
```

它的输出结果如下：

```
priority 0  ab.order.BeanProcessorOrderedDemo.MyBeanPostProcessor1#0
priority 1  ab.order.BeanProcessorOrderedDemo.MyBeanPostProcessor1#0
priority 0  ab.order.BeanProcessorOrderedDemo.MyBeanPostProcessor2#0
priority 1  ab.order.BeanProcessorOrderedDemo.MyBeanPostProcessor2#0
priority 0  ab.order.BeanProcessorOrderedDemo.MyBeanPostProcessor5#0
priority 1  ab.order.BeanProcessorOrderedDemo.MyBeanPostProcessor5#0
order 0  ab.order.BeanProcessorOrderedDemo.MyBeanPostProcessor5#0
order 1  ab.order.BeanProcessorOrderedDemo.MyBeanPostProcessor5#0
priority 0  test-user
priority 1  test-user
order 0  test-user
order 1  test-user
no order  test-user
refresh finish!
User(id=null, name=null, city=null, workCities=null, lifeCities=null, configFileLocation=null)
```

从代码中可以看出，同种级别（PriorityOrdered、Ordered和NoOrder）的BeanPostProcessor是同一批注册的。所以当两个PriorityOrdered注册的时候，是没有输出的，两个Ordered注册的时候，每个输出priority 0和priority 1，无序的BeanProcessor注册时输出priority 0、priority 1、order 0和order 1。当test-user注册时输出priority 0、priority 1、order 0、order 1和no order。

#### finishBeanFactoryInitialization

initMessageSource、initApplicationEventMulticaster和registerListeners的功能暂时不分析，onRefresh默认实现为空也暂时不考虑。

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // Initialize conversion service for this context.
    if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
        beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
        beanFactory.setConversionService(
            beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
    }

    // Register a default embedded value resolver if no bean post-processor
    // (such as a PropertyPlaceholderConfigurer bean) registered any before:
    // at this point, primarily for resolution in annotation attribute values.
    if (!beanFactory.hasEmbeddedValueResolver()) {
        beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
    }

    // Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
    String[] weaverAwareNames = 
        beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
    for (String weaverAwareName : weaverAwareNames) {
        getBean(weaverAwareName);
    }

    // Stop using the temporary ClassLoader for type matching.
    beanFactory.setTempClassLoader(null);

    // Allow for caching all bean definition metadata, not expecting further changes.
    // 在此之后便不再允许通过BeanDefinition的方式注册Bean了，但是仍然可以注册singleton。
    beanFactory.freezeConfiguration();

    // Instantiate all remaining (non-lazy-init) singletons.
    beanFactory.preInstantiateSingletons();
}
```

finishBeanFactoryInitialization的功能就是初始化非懒加载的Bean，但是在初始化之前又做了一些初始化BeanFactory的工作，这些工作我暂时还不明白，所以先跳过。preInstantiateSingletons的功能就是初始化非懒加载的Bean。

preInstantiateSingletons的实现是在DefaultListableBeanFactor

```java
public void preInstantiateSingletons() throws BeansException {
    if (logger.isTraceEnabled()) {
        logger.trace("Pre-instantiating singletons in " + this);
    }

    // Iterate over a copy to allow for init methods which in turn register new bean definitions.
    // While this may not be part of the regular factory bootstrap, it does otherwise work fine.
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

    // Trigger initialization of all non-lazy singleton beans...
    for (String beanName : beanNames) {
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            if (isFactoryBean(beanName)) {
                Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
                if (bean instanceof FactoryBean) {
                    final FactoryBean<?> factory = (FactoryBean<?>) bean;
                    boolean isEagerInit;
                    if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                        isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
                        	((SmartFactoryBean<?>) factory)::isEagerInit, getAccessControlContext());
                    } else {
                        isEagerInit = (factory instanceof SmartFactoryBean &&
                                       ((SmartFactoryBean<?>) factory).isEagerInit());
                    }
                    if (isEagerInit) {
                        getBean(beanName);
                    }
                }
            } else {
                getBean(beanName);
            }
        }
    }
    // Trigger post-initialization callback for all applicable beans...
    for (String beanName : beanNames) {
        Object singletonInstance = getSingleton(beanName);
        if (singletonInstance instanceof SmartInitializingSingleton) {
            final SmartInitializingSingleton smartSingleton = 
                (SmartInitializingSingleton) singletonInstance;
            if (System.getSecurityManager() != null) {
                AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
                    smartSingleton.afterSingletonsInstantiated();
                    return null;
                }, getAccessControlContext());
            } else {
                smartSingleton.afterSingletonsInstantiated();
            }
        }
    }
}
```

**SmartFactoryBean**

如果注册了一个FactoryBean，在获取FactoryBean本身的时候，是不会实例化其内部的真实Bean的。但是SmartFactoryBean可以让其内部的Bean被实例化。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd" default-autowire="byType">
    <bean class="ab.smartfactorybean.SmartFactoryBeanDemo" id = "smartfactorybean"></bean>
</beans>
```

```java
public class SmartFactoryBeanDemo implements SmartFactoryBean<String> {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext =
            new ClassPathXmlApplicationContext(
            	new String[]{"META-INF/bean-smartfactorybean-context.xml"}, false, null);
        applicationContext.refresh();
        System.out.println(applicationContext.getBean("&smartfactorybean"));
        applicationContext.close();
    }

    @Override
    public String getObject() throws Exception {
        System.out.println("SmartFactoryBean 内部真实的Bean被初始化");
        return "SmartFactoryBean aha";
    }
    @Override
    public Class<?> getObjectType() {
        return String.class;
    }
    @Override
    public boolean isEagerInit() {
        return true;
    }
}
```

**SmartInitializingSingleton**

在所有非懒加载的Bean都实例化完成之后，如果单例Bean实现了SmartInitializingSingleton，会调用afterSingletonsInstantiated方法。

````java
@Setter
@Getter
@ToString
public class ACDomainWithSmartInitializing implements SmartInitializingSingleton {
    private BeanFactory beanFactory;
    private ApplicationContext applicationContext;

    @Override
    public void afterSingletonsInstantiated() {
        System.out.println
            ("所有Bean都实例化完成了，现在执行ACDomainWithSmartInitializing的afterSingletonsInstantiated");
    }
}
````

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="acDomain" class="ab.holder.ACDomain" autowire="byType" />
    <bean id="aCDomainWithSmartInitializing" 
          class="ab.holder.ACDomainWithSmartInitializing" autowire="byType" />

    <bean class="ab.smartinitializingsingleton.SmartInitializingFactoryBeanDemo.MyBeanPostProcessor" />
</beans>
```

```java
public class SmartInitializingFactoryBeanDemo {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext =
                new ClassPathXmlApplicationContext(
            		new String[]{"META-INF/bean-smartinitializing-context.xml"}, false, null);
        applicationContext.refresh();
        applicationContext.close();
    }

    static class MyBeanPostProcessor implements BeanPostProcessor {
        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) 
            	throws BeansException {
            System.out.println(beanName + " 实例化完成！");
            return null;
        }
    }
}
```

#### finishRefresh

```java
protected void finishRefresh() {
    // Clear context-level resource caches (such as ASM metadata from scanning).
    clearResourceCaches();

    // Initialize lifecycle processor for this context.
    initLifecycleProcessor();

    // Propagate refresh to lifecycle processor first.
    getLifecycleProcessor().onRefresh();

    // Publish the final event.
    publishEvent(new ContextRefreshedEvent(this));

    // Participate in LiveBeansView MBean, if active.
    LiveBeansView.registerApplicationContext(this);
}
```

finishRefresh的目的就是激活生命周期，并且发布事件。



## 依赖注入的使用

ApplicationContext提供了依赖注入功能，这也是我们最常使用的Spring功能。下面将依赖注入分类演示。

### Setter方法注入

#### xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="user" class="fsc.domain.User">
        <property name="city" value="SHENZHEN" />
        <property name="name" value="isjinhao" />
        <property name="id" value="100" />
    </bean>

    <bean id="xmlUserHolder" class="ab.holder.UserHolder">
        <!-- 这个ref就表示依赖注入，但它不是自动注入 -->
        <property name="user" ref="user" />
        <property name="id"  value="xml dependency setter injection" />
    </bean>
</beans>
```

```java
public class XmlDependencySetterInjectionDemo {
    public static void main(String[] args) {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
        String xmlResourcePath = "META-INF/dependency-setter-injection.xml";
        // 加载 XML 资源，解析并且生成 BeanDefinition
        beanDefinitionReader.loadBeanDefinitions(xmlResourcePath);

        // 依赖查找并且创建 Bean
        UserHolder userHolder = beanFactory.getBean(UserHolder.class);
        System.out.println(userHolder);
    }
}
```

#### annotation

```java
public class AnnotationDependencySetterInjectionDemo {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = 
            new AnnotationConfigApplicationContext();
        applicationContext.register(AnnotationDependencySetterInjectionDemo.class);

        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(applicationContext);
        String xmlResourcePath = "META-INF/dependency-setter-injection.xml";
        beanDefinitionReader.loadBeanDefinitions(xmlResourcePath);
        
        applicationContext.refresh();
        
        UserHolder userHolder = applicationContext.getBean("annotationUserHolder", UserHolder.class);
        System.out.println(userHolder);
        
        applicationContext.close();
    }

    // 依赖注入
    @Bean
    public UserHolder annotationUserHolder(User user) {
        UserHolder userHolder = new UserHolder();
        userHolder.setUser(user);
        userHolder.setId("annotation dependency setter injection");
        return userHolder;
    }
}
```

#### API

```java
public class ApiDependencySetterInjectionDemo {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = 
            new AnnotationConfigApplicationContext();

        BeanDefinition userHolderBeanDefinition = createUserHolderBeanDefinition();
        applicationContext.registerBeanDefinition("apiUserHolder", userHolderBeanDefinition);

        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(applicationContext);
        String xmlResourcePath = "META-INF/dependency-setter-injection.xml";
        beanDefinitionReader.loadBeanDefinitions(xmlResourcePath);

        applicationContext.refresh();

        UserHolder userHolder = applicationContext.getBean("apiUserHolder", UserHolder.class);
        System.out.println(userHolder);

        // 显示地关闭 Spring 应用上下文
        applicationContext.close();
    }

    private static BeanDefinition createUserHolderBeanDefinition() {
        BeanDefinitionBuilder definitionBuilder = 
            BeanDefinitionBuilder.genericBeanDefinition(UserHolder.class);
        definitionBuilder.addPropertyReference("user", "user");
        definitionBuilder.addPropertyValue("id", "api dependency setter injection");
        return definitionBuilder.getBeanDefinition();
    }
}
```

#### autowire

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <import resource="dependency-setter-injection.xml" />

    <bean id="autowireUserHolder" class="ab.holder.UserHolder" autowire="byType" >
        <property name="id" value="autowire dependency setter injection" />
    </bean>
</beans>
```

```java
public class AutowireDependencySetterInjectionDemo {
    public static void main(String[] args) {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        
        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
        String xmlResourcePath = "META-INF/autowire-dependency-setter-injection.xml";
        beanDefinitionReader.loadBeanDefinitions(xmlResourcePath);
        
        UserHolder userHolder = beanFactory.getBean("autowireUserHolder", UserHolder.class);
        System.out.println(userHolder);
    }
}
```

### Constructor

#### xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    <import resource="dependency-setter-injection.xml" />

    <bean id="constructorXmlUserHolder" class="ab.holder.UserHolder">
        <constructor-arg name="user" ref="user" />
        <property name="id" value="constructor xml dependency injection" />
    </bean>
</beans>
```

```java
public class XmlDependencyConstructorInjectionDemo {
    public static void main(String[] args) {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
        String xmlResourcePath = "META-INF/dependency-constructor-injection.xml";
        beanDefinitionReader.loadBeanDefinitions(xmlResourcePath);
        
        UserHolder userHolder = beanFactory.getBean("constructorXmlUserHolder", UserHolder.class);
        System.out.println(userHolder);
    }
}
```

#### annotation

```java
public class AnnotationDependencyConstructorInjectionDemo {
    private UserHolder userHolder;
    @Autowired
    public AnnotationDependencyConstructorInjectionDemo(User user) {
        UserHolder userHolder = new UserHolder(user);
        userHolder.setId("annotation dependency constructor injection");
        this.userHolder = userHolder;
    }
    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = 
            new AnnotationConfigApplicationContext();
        applicationContext.register(AnnotationDependencyConstructorInjectionDemo.class);

        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(applicationContext);
        String xmlResourcePath = "META-INF/dependency-setter-injection.xml";
        beanDefinitionReader.loadBeanDefinitions(xmlResourcePath);
        
        applicationContext.refresh();
        AnnotationDependencyConstructorInjectionDemo demo = 
            applicationContext.getBean(AnnotationDependencyConstructorInjectionDemo.class);
        System.out.println(demo.userHolder);
        applicationContext.close();
    }
}
```

#### API

```java
public class ApiDependencyConstructorInjectionDemo {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = 
            new AnnotationConfigApplicationContext();
        BeanDefinition userHolderBeanDefinition = createUserHolderBeanDefinition();
        applicationContext.registerBeanDefinition("constructorApiUserHolder", userHolderBeanDefinition);
        
        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(applicationContext);
        String xmlResourcePath = "META-INF/dependency-setter-injection.xml";
        beanDefinitionReader.loadBeanDefinitions(xmlResourcePath);

        applicationContext.refresh();

        UserHolder userHolder = 
            applicationContext.getBean("constructorApiUserHolder", UserHolder.class);
        System.out.println(userHolder);

        applicationContext.close();
    }

    private static BeanDefinition createUserHolderBeanDefinition() {
        BeanDefinitionBuilder definitionBuilder = 
            BeanDefinitionBuilder.genericBeanDefinition(UserHolder.class);
        definitionBuilder.addConstructorArgReference("user");
        definitionBuilder.addPropertyValue("id", "constructor api dependency injection");
        return definitionBuilder.getBeanDefinition();
    }
}
```

#### autowire

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <import resource="dependency-setter-injection.xml"/>
    <bean id="autowireConstructorUserHolder" class="ab.holder.UserHolder" autowire="constructor" >
        <property name="id" value="autowire constructor dependency injection" />
    </bean>
</beans>
```

```java
public class AutowireConstructorDependencyConstructorInjectionDemo {
    public static void main(String[] args) {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        
        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
        String xmlResourcePath = "META-INF/autowiring-dependency-constructor-injection.xml";
        beanDefinitionReader.loadBeanDefinitions(xmlResourcePath);
        
        UserHolder userHolder = beanFactory.getBean("autowireConstructorUserHolder", UserHolder.class);
        System.out.println(userHolder);
    }
}
```



### 方法注入

```java
public class AnnotationDependencyMethodInjectionDemo {
    private UserHolder userHolder;
    private UserHolder userHolder2;

    /**
     * 方法注入和方法名无关，只和方法参数的类型有关
     */
    @Autowired
    public void init1(UserHolder userHolder) {
        this.userHolder = userHolder;
    }
    @Resource
    public void init2(UserHolder userHolder2) {
        this.userHolder2 = userHolder2;
    }

    @Bean
    public UserHolder userHolder(User user) {
        return new UserHolder(user);
    }

    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = 
            new AnnotationConfigApplicationContext();
        applicationContext.register(AnnotationDependencyMethodInjectionDemo.class);

        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(applicationContext);
        String xmlResourcePath = "META-INF/dependency-setter-injection.xml";
        beanDefinitionReader.loadBeanDefinitions(xmlResourcePath);

        applicationContext.refresh();
        
        AnnotationDependencyMethodInjectionDemo demo = 
            applicationContext.getBean(AnnotationDependencyMethodInjectionDemo.class);

        UserHolder userHolder = demo.userHolder;
        System.out.println(userHolder);
        System.out.println(demo.userHolder2);

        System.out.println(userHolder == demo.userHolder2);
        applicationContext.close();
    }
}
```

### 字段注入

```java
public class AnnotationDependencyFieldInjectionDemo {
    // @Autowired 会忽略掉静态字段
    @Autowired
    @Qualifier("userHolder")
    private UserHolder userHolder;
    @Resource(name = "userHolder")
    private UserHolder userHolder2;

    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = 
            new AnnotationConfigApplicationContext();
        applicationContext.register(AnnotationDependencyFieldInjectionDemo.class);

        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(applicationContext);
        String xmlResourcePath = "META-INF/dependency-setter-injection.xml";
        beanDefinitionReader.loadBeanDefinitions(xmlResourcePath);

        applicationContext.refresh();
        AnnotationDependencyFieldInjectionDemo demo = 
            applicationContext.getBean(AnnotationDependencyFieldInjectionDemo.class);

        System.out.println(applicationContext.getBeansOfType(UserHolder.class));

        UserHolder userHolder = demo.userHolder;
        System.out.println(userHolder);
        System.out.println(demo.userHolder2);
        System.out.println(userHolder == demo.userHolder2);
        
        applicationContext.close();
    }

    @Bean
    public UserHolder userHolder(User user) {
        return new UserHolder(user);
    }
}
```

### Aware

```java
public class AwareInterfaceDependencyInjectionDemo 
    	implements BeanFactoryAware, ApplicationContextAware {

    private static BeanFactory beanFactory;
    private static ApplicationContext applicationContext;

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        context.register(AwareInterfaceDependencyInjectionDemo.class);

        context.refresh();

        System.out.println(beanFactory == context.getBeanFactory());
        System.out.println(applicationContext == context);

        context.close();
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        AwareInterfaceDependencyInjectionDemo.beanFactory = beanFactory;
    }
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        AwareInterfaceDependencyInjectionDemo.applicationContext = applicationContext;
    }
}
```

### 延迟注入

```java
// 非延迟Bean实现了延迟注入
public class LazyAnnotationDependencyInjectionDemo {
    @Autowired
    private User user; // 实时注入
    @Autowired
    private ObjectProvider<User> userObjectProvider; // 延迟注入
    @Autowired
    private ObjectFactory<Set<User>> usersObjectFactory;

    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = 
            new AnnotationConfigApplicationContext();
        applicationContext.register(LazyAnnotationDependencyInjectionDemo.class);

        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(applicationContext);
        String xmlResourcePath = "META-INF/dependency-setter-injection.xml";
        beanDefinitionReader.loadBeanDefinitions(xmlResourcePath);

        applicationContext.refresh();

        // 依赖查找 QualifierAnnotationDependencyInjectionDemo Bean
        LazyAnnotationDependencyInjectionDemo demo = 
            applicationContext.getBean(LazyAnnotationDependencyInjectionDemo.class);

        System.out.println("demo.user = " + demo.user);
        System.out.println("demo.userObjectProvider = " + demo.userObjectProvider.getObject());
        System.out.println("demo.usersObjectFactory = " + demo.usersObjectFactory.getObject());

        demo.userObjectProvider.forEach(System.out::println);

        applicationContext.close();
    }
}
```

### 限定注入

```java
public class QualifierAnnotationDependencyInjectionDemo {

    @Autowired
    private User user;

    @Autowired
    @Qualifier("user1")
    private User namedUser;

    @Autowired
    private Collection<User> allUsers;

    @Autowired
    @Qualifier
    private Collection<User> qualifiedUsers;

    @Bean
    @Qualifier
    public User user1() {
        return createUser(1L);
    }
    @Bean
    @Qualifier
    public User user2() {
        return createUser(2L);
    }

    @Bean
    public User use3() {
        return createUser(3L);
    }

    private static User createUser(Long id) {
        User user = new User();
        user.setId(id);
        return user;
    }

    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = 
            new AnnotationConfigApplicationContext();
        applicationContext.register(QualifierAnnotationDependencyInjectionDemo.class);

        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(applicationContext);
        String xmlResourcePath = "META-INF/dependency-setter-injection.xml";
        beanDefinitionReader.loadBeanDefinitions(xmlResourcePath);

        applicationContext.refresh();

        QualifierAnnotationDependencyInjectionDemo demo = 
            applicationContext.getBean(QualifierAnnotationDependencyInjectionDemo.class);

        System.out.println(applicationContext.getBeansOfType(User.class));
        System.out.println("demo.user = " + demo.user);
        System.out.println("demo.namedUser = " + demo.namedUser);
        System.out.println("demo.allUsers = " + demo.allUsers);
        System.out.println("demo.qualifiedUsers = " + demo.qualifiedUsers);

        applicationContext.close();
    }
}
```

### 自定义注解

#### 分组

```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Qualifier
public @interface UserGroup1 {
}
```

```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Qualifier
public @interface UserGroup2 {
}
```

```java
public class GroupTestBeanFactory {
    @Bean
    public User user1() {
        return createUser(1L);
    }

    @Bean
    @Qualifier
    public User user2() {
        return createUser(2L);
    }

    @Bean
    @UserGroup1
    public static User user3() {
        return createUser(3L);
    }

    @Bean
    @UserGroup1
    public static User user4() {
        return createUser(4L);
    }

    @Bean
    @UserGroup2
    public static User user5() {
        return createUser(5L);
    }

    @Bean
    @UserGroup2
    public static User user6() {
        return createUser(6L);
    }

    private static User createUser(Long id) {
        User user = new User();
        user.setId(id);
        return user;
    }
}
```

```java
public class CustomizeQualifierAnnotationDependencyInjectionDemo {

    @Autowired
    private Collection<User> allUsers;

    @Autowired
    @Qualifier
    private Collection<User> qualifiedUsers;

    @Autowired
    @UserGroup1
    private Collection<User> group1Users;

    @Autowired
    @UserGroup2
    private Collection<User> group2Users;

    public static void main(String[] args) {

        // 创建 BeanFactory 容器
        AnnotationConfigApplicationContext applicationContext = 
            new AnnotationConfigApplicationContext();
        // 注册 Configuration Class（配置类） -> Spring Bean
        applicationContext.register(GroupTestBeanFactory.class);
        applicationContext.register(CustomizeQualifierAnnotationDependencyInjectionDemo.class);

        // 启动 Spring 应用上下文
        applicationContext.refresh();

        CustomizeQualifierAnnotationDependencyInjectionDemo demo = 
            applicationContext.getBean(CustomizeQualifierAnnotationDependencyInjectionDemo.class);

        System.out.println("demo.allUsers = ");
        demo.allUsers.stream().forEach(System.out::println);
        System.out.println("-----------------------------");
        System.out.println("demo.qualifiedUsers = ");
        demo.qualifiedUsers.stream().forEach(System.out::println);
        System.out.println("-----------------------------");

        System.out.println("demo.group1Users = ");
        demo.group1Users.stream().forEach(System.out::println);
        System.out.println("-----------------------------");
        System.out.println("demo.group2Users = ");
        demo.group2Users.stream().forEach(System.out::println);
        System.out.println("-----------------------------");

        // 显示地关闭 Spring 应用上下文
        applicationContext.close();
    }

}
```

#### 继承@Autowired

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Autowired
public @interface MyAutowired {
    boolean required() default true;
}
```

```java
public class ExtendAutowiredAnnotationDemo {
    @MyAutowired
    private Optional<User> userOptional; // superUser
    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = 
            new AnnotationConfigApplicationContext();
        applicationContext.register(ExtendAutowiredAnnotationDemo.class);

        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(applicationContext);
        String xmlResourcePath = "META-INF/dependency-setter-injection.xml";
        beanDefinitionReader.loadBeanDefinitions(xmlResourcePath);

        applicationContext.refresh();
        ExtendAutowiredAnnotationDemo demo = 
            applicationContext.getBean(ExtendAutowiredAnnotationDemo.class);
        System.out.println("demo.userOptional = " + demo.userOptional);

        applicationContext.close();
    }
}
```

#### 自定义注解

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface InjectedUser {
}
```

```java
public class ApiCustomizeAnnotationDemo {

    @InjectedUser
    private User myInjectedUser;

    @Bean(name = AnnotationConfigUtils.AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)
    public static AutowiredAnnotationBeanPostProcessor beanPostProcessor() {
        AutowiredAnnotationBeanPostProcessor beanPostProcessor = 
            new AutowiredAnnotationBeanPostProcessor();
        // @Autowired + @Inject +  新注解 @InjectedUser
        Set<Class<? extends Annotation>> autowiredAnnotationTypes =
                new LinkedHashSet<>(Arrays.asList(Autowired.class, Inject.class, InjectedUser.class));
        beanPostProcessor.setAutowiredAnnotationTypes(autowiredAnnotationTypes);
        return beanPostProcessor;
    }

//    @Bean
//    @Order(Ordered.LOWEST_PRECEDENCE - 3)
//    public static AutowiredAnnotationBeanPostProcessor beanPostProcessor() {
//        AutowiredAnnotationBeanPostProcessor beanPostProcessor = 
//                new AutowiredAnnotationBeanPostProcessor();
//        beanPostProcessor.setAutowiredAnnotationType(InjectedUser.class);
//        return beanPostProcessor;
//    }

    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = 
            new AnnotationConfigApplicationContext();
        applicationContext.register(ApiCustomizeAnnotationDemo.class);

        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(applicationContext);
        String xmlResourcePath = "META-INF/dependency-setter-injection.xml";
        beanDefinitionReader.loadBeanDefinitions(xmlResourcePath);

        applicationContext.refresh();

        ApiCustomizeAnnotationDemo demo = applicationContext.getBean(ApiCustomizeAnnotationDemo.class);
        System.out.println("demo.myInjectedUser = " + demo.myInjectedUser);

        applicationContext.close();
    }
}
```







