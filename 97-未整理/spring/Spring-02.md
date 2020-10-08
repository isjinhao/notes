##  Dependency Lookup

## AbstractApplicationContext 内建可查找的依赖

| Bean 名称                   | Bean 实例                         | 使用场景                |
| --------------------------- | --------------------------------- | ----------------------- |
| environment                 | Environment 对象                  | 外部化配置以及 Profiles |
| systemProperties            | java.util.Properties 对象         | Java 系统属性           |
| systemEnvironment           | java.util.Map 对象                | 操作系统环境变量        |
| messageSource               | MessageSource 对象                | 国际化文案              |
| lifecycleProcessor          | LifecycleProcessor 对象           | Lifecycle Bean 处理器   |
| applicationEventMulticaster | ApplicationEventMulticaster 对 象 | Spring 事件广播器       |







## 注解驱动 Spring 应用上下文内建可查找的依赖（部分）

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