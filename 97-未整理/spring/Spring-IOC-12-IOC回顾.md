## ApplicationContext启动

我们以ClassPathXmlApplicationContext的启动为例，回顾Application的启动。



### 构造函数执行

```java
// ClassPathXmlApplicationContext.java
public ClassPathXmlApplicationContext(String... configLocations) throws BeansException {
    this(configLocations, true, null);
}
public ClassPathXmlApplicationContext(
    		String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
    throws BeansException {

    // 这个最终执行到的是在AbstractApplicationContext里面
    // 目的是给获得一个PathMatchingResourcePatternResolver。
    // public AbstractApplicationContext(@Nullable ApplicationContext parent) {
    // 	   this();
    //     setParent(parent);
    // }
    // public AbstractApplicationContext() {
    //     this.resourcePatternResolver = getResourcePatternResolver();
    // }
    // protected ResourcePatternResolver getResourcePatternResolver() {
    //     return new PathMatchingResourcePatternResolver(this);
    // }
    super(parent);
    setConfigLocations(configLocations);
    if (refresh) {
        refresh();
    }
}
```

```java
// AbstractRefreshableConfigApplicationContext.java
public void setConfigLocations(@Nullable String... locations) {
    if (locations != null) {
        Assert.noNullElements(locations, "Config locations must not be null");
        this.configLocations = new String[locations.length];
        for (int i = 0; i < locations.length; i++) {
            // 这里解析的占位符解析不会使用自定义的配置，只会使用系统配置
            this.configLocations[i] = resolvePath(locations[i]).trim();
        }
    }
    else {
        this.configLocations = null;
    }
}
protected String resolvePath(String path) {
    return getEnvironment().resolveRequiredPlaceholders(path);
}
```

```java
// AbstractApplicationContext.java
public ConfigurableEnvironment getEnvironment() {
   if (this.environment == null) {
      this.environment = createEnvironment();
   }
   return this.environment;
}
protected ConfigurableEnvironment createEnvironment() {
   return new StandardEnvironment();
}
```

StandardEnvironment会将系统环境设置到Application里面。

```java
public class StandardEnvironment extends AbstractEnvironment {
    public static final String SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME = "systemEnvironment";
    public static final String SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME = "systemProperties";

    @Override
    protected void customizePropertySources(MutablePropertySources propertySources) {
        propertySources.addLast(
            new PropertiesPropertySource
            			(SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME, getSystemProperties()));
        propertySources.addLast(
            new SystemEnvironmentPropertySource
            			(SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME, getSystemEnvironment()));
    }
}
```



### refresh

#### prepareRefresh

prepareRefresh的默认功能就是解决处理早期事件。

```java
protected void prepareRefresh() {
   // Switch to active.
   this.startupDate = System.currentTimeMillis();
   this.closed.set(false);
   this.active.set(true);

   if (logger.isDebugEnabled()) {
      if (logger.isTraceEnabled()) {
         logger.trace("Refreshing " + this);
      }
      else {
         logger.debug("Refreshing " + getDisplayName());
      }
   }

   // Initialize any placeholder property sources in the context environment.
   initPropertySources();

   // Validate that all properties marked as required are resolvable:
   // see ConfigurablePropertyResolver#setRequiredProperties
   getEnvironment().validateRequiredProperties();

   // Store pre-refresh ApplicationListeners...
   if (this.earlyApplicationListeners == null) {
      this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
   }
   else {
      // Reset local application listeners to pre-refresh state.
      this.applicationListeners.clear();
      this.applicationListeners.addAll(this.earlyApplicationListeners);
   }

   // Allow for the collection of early ApplicationEvents,
   // to be published once the multicaster is available...
   this.earlyApplicationEvents = new LinkedHashSet<>();
}
```

#### prepareBeanFactory

```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
   // Tell the internal bean factory to use the context's class loader etc.
   beanFactory.setBeanClassLoader(getClassLoader());
   beanFactory.setBeanExpressionResolver
       (new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
    
   // 遗留问题，SpringMVC模块分析
   beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

   // Configure the bean factory with context callbacks.
   beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
   beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
   beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
   beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
   beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
   beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
   beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

   // BeanFactory interface not registered as resolvable type in a plain factory.
   // MessageSource registered (and found for autowiring) as a bean.
   beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
   beanFactory.registerResolvableDependency(ResourceLoader.class, this);
   beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
   beanFactory.registerResolvableDependency(ApplicationContext.class, this);

   // Register early post-processor for detecting inner beans as ApplicationListeners.
   beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

   // 遗留问题，AOP模块分析
   // Detect a LoadTimeWeaver and prepare for weaving, if found.
   if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
      beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
      // Set a temporary ClassLoader for type matching.
      beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
   }

   // Register default environment beans.
   // 从这里可以看出类，ENVIRONMENT和ENVIRONMENT的外部化配置都会被注册为singleton
   if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
      beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
   }
   if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
      beanFactory.registerSingleton
          (SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
   }
   if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
      beanFactory.registerSingleton
          (SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
   }
}
```







## 依赖来源

##  Dependency Lookup









### AbstractApplicationContext 内建可查找的依赖

| Bean 名称                   | Bean 实例                         | 使用场景                |
| --------------------------- | --------------------------------- | ----------------------- |
| environment                 | Environment 对象                  | 外部化配置以及 Profiles |
| systemProperties            | java.util.Properties 对象         | Java 系统属性           |
| systemEnvironment           | java.util.Map 对象                | 操作系统环境变量        |
| messageSource               | MessageSource 对象                | 国际化文案              |
| lifecycleProcessor          | LifecycleProcessor 对象           | Lifecycle Bean 处理器   |
| applicationEventMulticaster | ApplicationEventMulticaster 对 象 | Spring 事件广播器       |







### 注解驱动 Spring 应用上下文内建可查找的依赖（部分）

| Bean 名称                                                    | Bean 实例                                 | 使用场景                                              |
| ------------------------------------------------------------ | ----------------------------------------- | ----------------------------------------------------- |
| org.springframework.context.annotation.internalConfigurationAnnotationProcessor | ConfigurationClassPostProcessor 对象      | 处理 Spring 配置类                                    |
| org.springframework.context.annotation.internalAutowiredAnnotationProcessor | AutowiredAnnotationBeanPostProcessor 对象 | 处理 @Autowired 以及 @Value 注解                      |
| org.springframework.context.annotation.internalCommonAnnotationProcessor | CommonAnnotationBeanPostProcessor 对象    | （条件激活）处理 JSR-250 注解，如 @PostConstruct 等   |
| org.springframework.context.event.internalEventListenerProcessor | EventListenerMethodProcessor 对象         | 处理标注 @EventListener 的Spring 事件监听方法         |
| org.springframework.context.event.internalEventListenerFactory | DefaultEventListenerFactory 对 象         | @EventListener 事件监听方法适配为 ApplicationListener |

1. ConfigurationClassPostProcessor—->BeanFactoryPostProcessor Spring容器的生命周期处理,BeanFactory后置处理器
2. AutowiredAnnotationBeanPostProcessor—->BeanPostProcessor Bean的生命周期处理,Bean的后置处理器
3. CommonAnnotationBeanPostProcessor—->BeanPostProcessor Bean的生命周期处理,Bean的后置处理器
4. EventListenerMethodProcessor—->BeanFactoryPostProcessor pring容器的生命周期处理,BeanFactory后置处理器
5. DefaultEventListenerFactory—->EventListenerFactory



Spring的ApplicationAware和实例化，初始化的时间。



FactoryBean的解析	

ObjectFactory的解析



