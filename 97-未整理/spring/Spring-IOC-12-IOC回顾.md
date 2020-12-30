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

prepareRefresh的功能就是解决处理早期事件。

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
    // 默认空实现
    initPropertySources();

    // Validate that all properties marked as required are resolvable:
    // see ConfigurablePropertyResolver#setRequiredProperties
    // 遗留问题，MVC模块分析
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

#### obtainFreshBeanFactory

```java
// AbstractApplicationContext.java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    refreshBeanFactory();
    return getBeanFactory();
}
```

```java
// AbstractRefreshableApplicationContext.java
protected final void refreshBeanFactory() throws BeansException {
    if (hasBeanFactory()) {
        destroyBeans();
        closeBeanFactory();
    }
    try {
        // 默认就是返回一个 DefaultListableBeanFactory
        DefaultListableBeanFactory beanFactory = createBeanFactory();
        // 每个ApplciationContext和BeanFactory都有一个Id，这里将ApplciationContext的Id设置到BeanFactory里面
        beanFactory.setSerializationId(getId());
        // 配置BeanFactory的两个属性：是否允许循环引用?  是否允许BeanDefinition覆盖。
        // BeanFactory里这两个属性默认都是true
        customizeBeanFactory(beanFactory);
        // 加载BeanDefinition
        loadBeanDefinitions(beanFactory);
        synchronized (this.beanFactoryMonitor) {
            this.beanFactory = beanFactory;
        }
    }
    catch (IOException ex) {
        throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
    }
}
```

```java
// AbstractXmlApplicationContext.java
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
    // Create a new XmlBeanDefinitionReader for the given BeanFactory.
    XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

    // Configure the bean definition reader with this context's
    // resource loading environment.
    // 由于我们传入了xml文件的路径，所以Environment在构造函数启动的时候就已经被构建了
    beanDefinitionReader.setEnvironment(this.getEnvironment());
    // ApplciationContext的继承了ResourceLoader，默认的实现是DefaultResourceLoader，采用的是类路径
    beanDefinitionReader.setResourceLoader(this);
    // EntityResolver是用来验证xml文件的，这是JDK提供的接口
    // ResourceEntityResolver接收的参数是RedourceLoader，所以就将this传过去了
    beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

    // Allow a subclass to provide custom initialization of the reader,
    // then proceed with actually loading the bean definitions.
    // init操作只有一个功能，是否开启XML文件的验证。默认是开启的。
    initBeanDefinitionReader(beanDefinitionReader);
    loadBeanDefinitions(beanDefinitionReader);
}
```

##### loadBeanDefinitions

loadBeanDefinitions的目的是将Bean加载到BeanFactory中去。加载时通过命名空间区分加载的具体实现。

```java
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
    if (delegate.isDefaultNamespace(root)) {
        NodeList nl = root.getChildNodes();
        for (int i = 0; i < nl.getLength(); i++) {
            Node node = nl.item(i);
            if (node instanceof Element) {
                Element ele = (Element) node;
                if (delegate.isDefaultNamespace(ele)) {
                    parseDefaultElement(ele, delegate);
                } else {
                    delegate.parseCustomElement(ele);
                }
            }
        }
    } else {
        delegate.parseCustomElement(root);
    }
}
```

对于beans命名空间下面的beans、alias、import、bean标签，采用parseDefaultElement方法进行解析。

```java
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
    if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
        importBeanDefinitionResource(ele);
    } else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
        processAliasRegistration(ele);
    } else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
        processBeanDefinition(ele, delegate);
    } else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
        // recurse
        doRegisterBeanDefinitions(ele);
    }
}
```

对于其他的采用parseCustomElement方法进行解析。

```java
public BeanDefinition parseCustomElement(Element ele, @Nullable BeanDefinition containingBd) {
    String namespaceUri = getNamespaceURI(ele);
    if (namespaceUri == null) {
        return null;
    }
	// NamespaceHandlerResolver是解析命名空间获得NamespaceHandler的。只有一个实现类DefaultNamespaceHandlerResolver
    // NamespaceHandlerResolver将namespaceUri和NamespaceHandler的对应关系放在META-INF/spring.handlers中。
    NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
    if (handler == null) {
        error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
        return null;
    }
    return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
}
```

##### NamespaceHandler#parse

NamespaceHandler的目的就是设置对应的Bean到Application中去。这些Bean可能是普通的Bean，也可能是BeanPostProcessor，也可能是BeanFactoryPostProcessor。context包下最常用的命名空间就是ContextNamespaceHandler。

```java
public class ContextNamespaceHandler extends NamespaceHandlerSupport {
	@Override
	public void init() {
		registerBeanDefinitionParser("property-placeholder", new PropertyPlaceholderBeanDefinitionParser());
		registerBeanDefinitionParser("property-override", new PropertyOverrideBeanDefinitionParser());
		registerBeanDefinitionParser("annotation-config", new AnnotationConfigBeanDefinitionParser());
		registerBeanDefinitionParser("component-scan", new ComponentScanBeanDefinitionParser());
		registerBeanDefinitionParser("load-time-weaver", new LoadTimeWeaverBeanDefinitionParser());
		registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
		registerBeanDefinitionParser("mbean-export", new MBeanExportBeanDefinitionParser());
		registerBeanDefinitionParser("mbean-server", new MBeanServerBeanDefinitionParser());
	}
}
```

回顾三个使用过的标签。PropertyPlaceholderBeanDefinitionParser、AnnotationConfigBeanDefinitionParser和ComponentScanBeanDefinitionParser。

**PropertyPlaceholderBeanDefinitionParser**

PropertyPlaceholderBeanDefinitionParser的目的就是将PropertySourcesPlaceholderConfigurer这个Bean设置到Application中去。PropertySourcesPlaceholderConfigurer是BeanFactoryPostProcessor。

```java
// PropertyPlaceholderBeanDefinitionParser是继承AbstractBeanDefinitionParser
public final BeanDefinition parse(Element element, ParserContext parserContext) {
    // 子类继承去从xml中解析出来BeanDefinition
    AbstractBeanDefinition definition = parseInternal(element, parserContext);
    if (definition != null && !parserContext.isNested()) {
        try {
            String id = resolveId(element, definition, parserContext);
            String[] aliases = null;
            // 可以看出来name属性会被解释为别名
            if (shouldParseNameAsAliases()) {
                String name = element.getAttribute(NAME_ATTRIBUTE);
                if (StringUtils.hasLength(name)) {
                    aliases = StringUtils.trimArrayElements(StringUtils.commaDelimitedListToStringArray(name));
                }
            }
            BeanDefinitionHolder holder = new BeanDefinitionHolder(definition, id, aliases);
            registerBeanDefinition(holder, parserContext.getRegistry());
            if (shouldFireEvents()) {
                // 这里是说在Bean被创建之后，在注册到BeanFactory之前。可以对其进行操作，默认的操作为空实现
                BeanComponentDefinition componentDefinition = new BeanComponentDefinition(holder);
                // 空操作，子类可以继承来进行一定的操作
                postProcessComponentDefinition(componentDefinition);
                // 可以发射一个事件，这个事件会被ReaderEventListener接收到
                // 默认的ReaderEventListener是空实现，EmptyReaderEventListener。
                // 我们可以调用XmlBeanDefinitionReader#setEventListener设置一个自己的ReaderEventListener
                parserContext.registerComponent(componentDefinition);
            }
        }
    }
    return definition;
}
```

```java
// AbstractSingleBeanDefinitionParser.java
protected final AbstractBeanDefinition parseInternal(Element element, ParserContext parserContext) {
    BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition();
    String parentName = getParentName(element);
    if (parentName != null) {
        builder.getRawBeanDefinition().setParentName(parentName);
    }
    // 从这里获得BeanDefinition的类型
    Class<?> beanClass = getBeanClass(element);
    if (beanClass != null) {
        builder.getRawBeanDefinition().setBeanClass(beanClass);
    } else {
        String beanClassName = getBeanClassName(element);
        if (beanClassName != null) {
            builder.getRawBeanDefinition().setBeanClassName(beanClassName);
        }
    }
    builder.getRawBeanDefinition().setSource(parserContext.extractSource(element));
    BeanDefinition containingBd = parserContext.getContainingBeanDefinition();
    if (containingBd != null) {
        // Inner bean definition must receive same scope as containing bean.
        builder.setScope(containingBd.getScope());
    }
    if (parserContext.isDefaultLazyInit()) {
        // Default-lazy-init applies to custom bean definitions as well.
        builder.setLazyInit(true);
    }
    // 做真正的解析操作
    doParse(element, parserContext, builder);
    return builder.getBeanDefinition();
}
```

```java
// AbstractPropertyLoadingBeanDefinitionParser.java
// AbstractPropertyLoadingBeanDefinitionParser 继承 AbstractSingleBeanDefinitionParser
// AbstractPropertyLoadingBeanDefinitionParser处理加载文件的常用属性，location、properties-ref、file-encoding等
@Override
protected void doParse(Element element, ParserContext parserContext, BeanDefinitionBuilder builder) {
    String location = element.getAttribute("location");
    if (StringUtils.hasLength(location)) {
        location = parserContext.getReaderContext().getEnvironment().resolvePlaceholders(location);
        String[] locations = StringUtils.commaDelimitedListToStringArray(location);
        builder.addPropertyValue("locations", locations);
    }

    String propertiesRef = element.getAttribute("properties-ref");
    if (StringUtils.hasLength(propertiesRef)) {
        builder.addPropertyReference("properties", propertiesRef);
    }

    String fileEncoding = element.getAttribute("file-encoding");
    if (StringUtils.hasLength(fileEncoding)) {
        builder.addPropertyValue("fileEncoding", fileEncoding);
    }

    String order = element.getAttribute("order");
    if (StringUtils.hasLength(order)) {
        builder.addPropertyValue("order", Integer.valueOf(order));
    }

    builder.addPropertyValue("ignoreResourceNotFound",
                             Boolean.valueOf(element.getAttribute("ignore-resource-not-found")));

    builder.addPropertyValue("localOverride",
                             Boolean.valueOf(element.getAttribute("local-override")));

    builder.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
}
```

```java
// PropertyPlaceholderBeanDefinitionParser.java
protected void doParse(Element element, ParserContext parserContext, BeanDefinitionBuilder builder) {
    super.doParse(element, parserContext, builder);
    builder.addPropertyValue("ignoreUnresolvablePlaceholders",
                             Boolean.valueOf(element.getAttribute("ignore-unresolvable")));

    String systemPropertiesModeName = element.getAttribute(SYSTEM_PROPERTIES_MODE_ATTRIBUTE);
    if (StringUtils.hasLength(systemPropertiesModeName) &&
        !systemPropertiesModeName.equals(SYSTEM_PROPERTIES_MODE_DEFAULT)) {
        builder.addPropertyValue("systemPropertiesModeName", "SYSTEM_PROPERTIES_MODE_" + systemPropertiesModeName);
    }

    if (element.hasAttribute("value-separator")) {
        builder.addPropertyValue("valueSeparator", element.getAttribute("value-separator"));
    }
    if (element.hasAttribute("trim-values")) {
        builder.addPropertyValue("trimValues", element.getAttribute("trim-values"));
    }
    if (element.hasAttribute("null-value")) {
        builder.addPropertyValue("nullValue", element.getAttribute("null-value"));
    }
}
```

**AnnotationConfigBeanDefinitionParser**

```java
public BeanDefinition parse(Element element, ParserContext parserContext) {
    Object source = parserContext.extractSource(element);
	
    // AnnotationConfigBeanDefinitionParser的功能就是这一句
    // Obtain bean definitions for all relevant BeanPostProcessors.
    Set<BeanDefinitionHolder> processorDefinitions =
        AnnotationConfigUtils.registerAnnotationConfigProcessors(parserContext.getRegistry(), source);

    // 对于刚才遇到的AbstractBeanDefinitionParser来说，一个标签，比如<context:property-placeholder/>，只会向Application注册一个Bean。
    // 而我们刚才说的ReaderEventListener是监听标签的解析成功事件，所以在一个标签会解析出来多个Bean的情境下，标签被封装成一个
    // CompositeComponentDefinition，而所有的BeanDefiniton都会被解析成BeanComponentDefinition，作为标签的组件。
    // 只有register标签的时候才能发布事件。 
    
    // Register component for the surrounding <context:annotation-config> element.
    CompositeComponentDefinition compDefinition = new CompositeComponentDefinition(element.getTagName(), source);
    parserContext.pushContainingComponent(compDefinition);

    // Nest the concrete beans in the surrounding component.
    for (BeanDefinitionHolder processorDefinition : processorDefinitions) {
        parserContext.registerComponent(new BeanComponentDefinition(processorDefinition));
    }

    // Finally register the composite component.
    parserContext.popAndRegisterContainingComponent();

    return null;
}
```

```java
// ParserContext.java
public void registerComponent(ComponentDefinition component) {
    CompositeComponentDefinition containingComponent = getContainingComponent();
    if (containingComponent != null) {
        containingComponent.addNestedComponent(component);
    } else {
        this.readerContext.fireComponentRegistered(component);
    }
}
```

AnnotationConfigUtils#registerAnnotationConfigProcessors是注解功能的核心实现类。

```java
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
    BeanDefinitionRegistry registry, @Nullable Object source) {

    // 这里可以看出来只有DefaultListableBeanFactory才能开启注解驱动功能
    DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
    if (beanFactory != null) {
        // 开启注解排序功能。即支持@Order
        if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
            beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
        }
        // 处理@Lazy（延迟依赖注入）和@Qualifier
        // DefaultListableBeanFactory#resolveDependency
        if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
            beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
        }
    }

    Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);

    // 添加ConfigurationClassPostProcessor，是BeanFactoryPostProcessor，用来处理注解，包括：
    // @Component、@PropertySources、@ComponentScans、@ComponentScan、@ImportResource
    if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    // 添加AutowiredAnnotationBeanPostProcessor，是BeanPostProcessor，用来处理Spring提供的注解，包括：
    // @Autowired、@Value和@Inject
    if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    // 添加CommonAnnotationBeanPostProcessor，是BeanPostProcessor，用来处理JSR-250提供的注解，包括：
    // @Resource、@EJB、@WebServiceRef、@PostConstruct、@PreDestroy
    // Check for JSR-250 support, and if present add the CommonAnnotationBeanPostProcessor.
    if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    // Check for JPA support, and if present add the PersistenceAnnotationBeanPostProcessor.
    if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition();
        try {
            def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
                                                AnnotationConfigUtils.class.getClassLoader()));
        }
		// 异常处理 ... 
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    // 是SmartInitializingSingleton，在所有的Bean被处理完成后才会调用。处理注解@EventListener
    if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
    }
    
    // 将标注@EventListener的方法处理成监听器的构造工厂DefaultEventListenerFactory
    if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
    }

    return beanDefs;
}
```

**ComponentScanBeanDefinitionParser**

```java
// ComponentScanBeanDefinitionParser.java
public BeanDefinition parse(Element element, ParserContext parserContext) {
    // 处理属性 base-package
    String basePackage = element.getAttribute(BASE_PACKAGE_ATTRIBUTE);
    // 原始的包名时可以使用占位符
    basePackage = parserContext.getReaderContext().getEnvironment().resolvePlaceholders(basePackage);
    String[] basePackages = StringUtils.tokenizeToStringArray(basePackage,
                                                              ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);

    // Actually scan for bean definitions and register them.
    ClassPathBeanDefinitionScanner scanner = configureScanner(parserContext, element);
    Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);
    registerComponents(parserContext.getReaderContext(), beanDefinitions, element);

    return null;
}
```

构造扫描器

```java
// ComponentScanBeanDefinitionParser.java
protected ClassPathBeanDefinitionScanner configureScanner(ParserContext parserContext, Element element) {
    boolean useDefaultFilters = true;
    // 配置默认的过滤器
    if (element.hasAttribute(USE_DEFAULT_FILTERS_ATTRIBUTE)) {
        useDefaultFilters = Boolean.parseBoolean(element.getAttribute(USE_DEFAULT_FILTERS_ATTRIBUTE));
    }

    // Delegate bean definition registration to scanner class.
    ClassPathBeanDefinitionScanner scanner = createScanner(parserContext.getReaderContext(), useDefaultFilters);
    scanner.setBeanDefinitionDefaults(parserContext.getDelegate().getBeanDefinitionDefaults());
    scanner.setAutowireCandidatePatterns(parserContext.getDelegate().getAutowireCandidatePatterns());

    // 配置资源的pattern
    if (element.hasAttribute(RESOURCE_PATTERN_ATTRIBUTE)) {
        scanner.setResourcePattern(element.getAttribute(RESOURCE_PATTERN_ATTRIBUTE));
    }

    try {
        // 配置NameGenerator，属性名为 name-generator
        parseBeanNameGenerator(element, scanner);
    }

    try {
        // 处理Scope，默认处理器是AnnotationScopeMetadataResolver，会处理@Scope注解
        parseScope(element, scanner);
    }

    // 对类型进行过滤
    parseTypeFilters(element, scanner, parserContext);

    return scanner;
}
```

useDefaultFilters为true时，会执行下面的代码。即将@Component、@ManagedBean、@Named作为可扫描的注解。由于@Component是@Repository、@Controller、@Service的元注解，所以这三个注解也会被处理。

```java
protected void registerDefaultFilters() {
    this.includeFilters.add(new AnnotationTypeFilter(Component.class));
    ClassLoader cl = ClassPathScanningCandidateComponentProvider.class.getClassLoader();
    try {
        this.includeFilters.add(new AnnotationTypeFilter(
            ((Class<? extends Annotation>) ClassUtils.forName("javax.annotation.ManagedBean", cl)), false));
        logger.trace("JSR-250 'javax.annotation.ManagedBean' found and supported for component scanning");
    }
    catch (ClassNotFoundException ex) {
        // JSR-250 1.1 API (as included in Java EE 6) not available - simply skip.
    }
    try {
        this.includeFilters.add(new AnnotationTypeFilter(
            ((Class<? extends Annotation>) ClassUtils.forName("javax.inject.Named", cl)), false));
        logger.trace("JSR-330 'javax.inject.Named' annotation found and supported for component scanning");
    }
    catch (ClassNotFoundException ex) {
        // JSR-330 API not available - simply skip.
    }
}
```

parseTypeFilters

```java
protected void parseTypeFilters(Element element, ClassPathBeanDefinitionScanner scanner, ParserContext parserContext) {
    // Parse exclude and include filter elements.
    ClassLoader classLoader = scanner.getResourceLoader().getClassLoader();
    NodeList nodeList = element.getChildNodes();
    for (int i = 0; i < nodeList.getLength(); i++) {
        Node node = nodeList.item(i);
        if (node.getNodeType() == Node.ELEMENT_NODE) {
            String localName = parserContext.getDelegate().getLocalName(node);
            // 处理include-filter和exclude-filter
            try {
                if (INCLUDE_FILTER_ELEMENT.equals(localName)) {
                    TypeFilter typeFilter = createTypeFilter((Element) node, classLoader, parserContext);
                    scanner.addIncludeFilter(typeFilter);
                } else if (EXCLUDE_FILTER_ELEMENT.equals(localName)) {
                    TypeFilter typeFilter = createTypeFilter((Element) node, classLoader, parserContext);
                    scanner.addExcludeFilter(typeFilter);
                }
            }
            // 异常处理 ...
        }
    }
}
protected TypeFilter createTypeFilter(Element element, @Nullable ClassLoader classLoader,
                                      ParserContext parserContext) throws ClassNotFoundException {

    String filterType = element.getAttribute(FILTER_TYPE_ATTRIBUTE);
    String expression = element.getAttribute(FILTER_EXPRESSION_ATTRIBUTE);
    expression = parserContext.getReaderContext().getEnvironment().resolvePlaceholders(expression);
    if ("annotation".equals(filterType)) {
        return new AnnotationTypeFilter((Class<Annotation>) ClassUtils.forName(expression, classLoader));
    } else if ("assignable".equals(filterType)) {
        return new AssignableTypeFilter(ClassUtils.forName(expression, classLoader));
    } else if ("aspectj".equals(filterType)) {
        return new AspectJTypeFilter(expression, classLoader);
    } else if ("regex".equals(filterType)) {
        return new RegexPatternTypeFilter(Pattern.compile(expression));
    } else if ("custom".equals(filterType)) {
        Class<?> filterClass = ClassUtils.forName(expression, classLoader);
        if (!TypeFilter.class.isAssignableFrom(filterClass)) {
            throw new IllegalArgumentException(
                "Class is not assignable to [" + TypeFilter.class.getName() + "]: " + expression);
        }
        return (TypeFilter) BeanUtils.instantiateClass(filterClass);
    } else {
        throw new IllegalArgumentException("Unsupported filter type: " + filterType);
    }
}
```

扫描包

```java
// ClassPathBeanDefinitionScanner.java
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
    Assert.notEmpty(basePackages, "At least one base package must be specified");
    Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();
    for (String basePackage : basePackages) {
        Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
        for (BeanDefinition candidate : candidates) {
            ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
            candidate.setScope(scopeMetadata.getScopeName());
            String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
            if (candidate instanceof AbstractBeanDefinition) {
                postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
            }
            if (candidate instanceof AnnotatedBeanDefinition) {
                AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
            }
            if (checkCandidate(beanName, candidate)) {
                BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
                // 默认的代理措施是NO，所以这里不会使用对BeanDefinition进行代理
                definitionHolder =
                    AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
                beanDefinitions.add(definitionHolder);
                registerBeanDefinition(definitionHolder, this.registry);
            }
        }
    }
    return beanDefinitions;
}
```

寻找合适的BeanDefinition

```java
// ClassPathScanningCandidateComponentProvider.java
public Set<BeanDefinition> findCandidateComponents(String basePackage) {
    if (this.componentsIndex != null && indexSupportsIncludeFilters()) {
		// 这里是处理@Index注解的，具体的可以看 https://www.cnblogs.com/aflyun/p/11992101.html
        return addCandidateComponentsFromIndex(this.componentsIndex, basePackage);
    } else {
        // 普通情况会走这里
        return scanCandidateComponents(basePackage);
    }
}

private Set<BeanDefinition> scanCandidateComponents(String basePackage) {
    Set<BeanDefinition> candidates = new LinkedHashSet<>();
    try {
        // CLASSPATH_ALL_URL_PREFIX = "classpath*:"
        // resourcePattern = "**/*.class"
        String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
            resolveBasePackage(basePackage) + '/' + this.resourcePattern;
        Resource[] resources = getResourcePatternResolver().getResources(packageSearchPath);
        boolean traceEnabled = logger.isTraceEnabled();
        boolean debugEnabled = logger.isDebugEnabled();
        for (Resource resource : resources) {
            if (resource.isReadable()) {
                try {
                    MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);
                    // 这里用excludeFilters和includeFilters进行过滤。
                    if (isCandidateComponent(metadataReader)) {
                        ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
                        sbd.setResource(resource);
                        sbd.setSource(resource);
                        // (独立的（top-level类 or 静态内部类）) && (具体类 || (抽象类 && 存在@Lookup的方法)) 
                        if (isCandidateComponent(sbd)) {
                            candidates.add(sbd);
                        }
                    }
                }
                // 异常处理 ...
            }
        }
    }
    // 异常处理 ...
    return candidates;
}
```

postProcessBeanDefinition

```java
protected void postProcessBeanDefinition(AbstractBeanDefinition beanDefinition, String beanName) {
    beanDefinition.applyDefaults(this.beanDefinitionDefaults);
    if (this.autowireCandidatePatterns != null) {
        // 此BeanDefinition构造的Bean是否可以作为依赖注入的Bean。只会影响按类型注入。即AUTOWIRE_BY_NAME和@Autowired
        beanDefinition.setAutowireCandidate(PatternMatchUtils.simpleMatch(this.autowireCandidatePatterns, beanName));
    }
}
```

processCommonDefinitionAnnotations

```java
// AnnotationConfigUtils.java
// 处理@Lazy（延迟Bean的创建）、@DependsOn、@Role、@Description、@Primary
// 这里并不是处理@Bean，@Bean的处理在@Configuration的处理里面。
public static void processCommonDefinitionAnnotations(AnnotatedBeanDefinition abd) {
    processCommonDefinitionAnnotations(abd, abd.getMetadata());
}
static void processCommonDefinitionAnnotations(AnnotatedBeanDefinition abd, AnnotatedTypeMetadata metadata) {
    AnnotationAttributes lazy = attributesFor(metadata, Lazy.class);
    if (lazy != null) {
        abd.setLazyInit(lazy.getBoolean("value"));
    } else if (abd.getMetadata() != metadata) {
        lazy = attributesFor(abd.getMetadata(), Lazy.class);
        if (lazy != null) {
            abd.setLazyInit(lazy.getBoolean("value"));
        }
    }
    if (metadata.isAnnotated(Primary.class.getName())) {
        abd.setPrimary(true);
    }
    AnnotationAttributes dependsOn = attributesFor(metadata, DependsOn.class);
    if (dependsOn != null) {
        abd.setDependsOn(dependsOn.getStringArray("value"));
    }
    AnnotationAttributes role = attributesFor(metadata, Role.class);
    if (role != null) {
        abd.setRole(role.getNumber("value").intValue());
    }
    AnnotationAttributes description = attributesFor(metadata, Description.class);
    if (description != null) {
        abd.setDescription(description.getString("value"));
    }
}
```

配置的扫描器扫描完成之后就到了AnnotationConfigUtils.registerAnnotationConfigProcessors阶段了。也会发射ReaderEvent。

```java
protected void registerComponents(
    XmlReaderContext readerContext, Set<BeanDefinitionHolder> beanDefinitions, Element element) {

    Object source = readerContext.extractSource(element);
    CompositeComponentDefinition compositeDef = new CompositeComponentDefinition(element.getTagName(), source);

    for (BeanDefinitionHolder beanDefHolder : beanDefinitions) {
        compositeDef.addNestedComponent(new BeanComponentDefinition(beanDefHolder));
    }

    // Register annotation config processors, if necessary.
    boolean annotationConfig = true;
    if (element.hasAttribute(ANNOTATION_CONFIG_ATTRIBUTE)) {
        annotationConfig = Boolean.parseBoolean(element.getAttribute(ANNOTATION_CONFIG_ATTRIBUTE));
    }
    if (annotationConfig) {
        Set<BeanDefinitionHolder> processorDefinitions =
            AnnotationConfigUtils.registerAnnotationConfigProcessors(readerContext.getRegistry(), source);
        for (BeanDefinitionHolder processorDefinition : processorDefinitions) {
            compositeDef.addNestedComponent(new BeanComponentDefinition(processorDefinition));
        }
    }

    readerContext.fireComponentRegistered(compositeDef);
}
```

加载BeanDefinition过程完毕之后，obtainFreshBeanFactory也就结束了。

#### prepareBeanFactory

```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // Tell the internal bean factory to use the context's class loader etc.
    beanFactory.setBeanClassLoader(getClassLoader());
    // StandardBeanExpressionResolver默认使用SpelExpressionParser对字符串进行解析。
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

##### ApplicationContextAwareProcessor

```java
// ApplicationContextAwareProcessor.java
// ApplicationContextAwareProcessor是BeanPostProcessor
private void invokeAwareInterfaces(Object bean) {
    if (bean instanceof EnvironmentAware) {
        ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
    }
    if (bean instanceof EmbeddedValueResolverAware) {
        ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
    }
    if (bean instanceof ResourceLoaderAware) {
        ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
    }
    if (bean instanceof ApplicationEventPublisherAware) {
        ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
    }
    if (bean instanceof MessageSourceAware) {
        ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
    }
    if (bean instanceof ApplicationContextAware) {
        ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
    }
}
```

从代码中可以看出来，ResourceLoaderAware、ApplicationEventPublisherAware、MessageSourceAware、ApplicationContextAware注入的都是ApplicationContext。

**EmbeddedValueResolverAware**

```java
// ApplicationContextAwareProcessor.java
private final StringValueResolver embeddedValueResolver;

public ApplicationContextAwareProcessor(ConfigurableApplicationContext applicationContext) {
    this.applicationContext = applicationContext;
    this.embeddedValueResolver = new EmbeddedValueResolver(applicationContext.getBeanFactory());
}
```

EmbeddedValueResolver会调用BeanFactory#resolveEmbeddedValue。StringValueResolver是一个函数式接口，只有resolveStringValue一个方法。

```java
public class EmbeddedValueResolver implements StringValueResolver {
    private final BeanExpressionContext exprContext;

    @Nullable
    private final BeanExpressionResolver exprResolver;

    public EmbeddedValueResolver(ConfigurableBeanFactory beanFactory) {
        this.exprContext = new BeanExpressionContext(beanFactory, null);
        this.exprResolver = beanFactory.getBeanExpressionResolver();
    }

    @Override
    @Nullable
    public String resolveStringValue(String strVal) {
        String value = this.exprContext.getBeanFactory().resolveEmbeddedValue(strVal);
        if (this.exprResolver != null && value != null) {
            // 这里的exprResolver就是StandardBeanExpressionResolver
            Object evaluated = this.exprResolver.evaluate(value, this.exprContext);
            value = (evaluated != null ? evaluated.toString() : null);
        }
        return value;
    }
}
```

**StandardBeanExpressionResolver**

```java
public Object evaluate(@Nullable String value, BeanExpressionContext evalContext) throws BeansException {
    if (!StringUtils.hasLength(value)) {
        return value;
    }
    try {
        Expression expr = this.expressionCache.get(value);
        if (expr == null) {
            expr = this.expressionParser.parseExpression(value, this.beanExpressionParserContext);
            this.expressionCache.put(value, expr);
        }
        StandardEvaluationContext sec = this.evaluationCache.get(evalContext);
        if (sec == null) {
            sec = new StandardEvaluationContext(evalContext);
            sec.addPropertyAccessor(new BeanExpressionContextAccessor());
            sec.addPropertyAccessor(new BeanFactoryAccessor());
            sec.addPropertyAccessor(new MapAccessor());
            sec.addPropertyAccessor(new EnvironmentAccessor());
            sec.setBeanResolver(new BeanFactoryResolver(evalContext.getBeanFactory()));
            sec.setTypeLocator(new StandardTypeLocator(evalContext.getBeanFactory().getBeanClassLoader()));
            ConversionService conversionService = evalContext.getBeanFactory().getConversionService();
            if (conversionService != null) {
                sec.setTypeConverter(new StandardTypeConverter(conversionService));
            }
            customizeEvaluationContext(sec);
            this.evaluationCache.put(evalContext, sec);
        }
        return expr.getValue(sec);
    }
    catch (Throwable ex) {
        throw new BeanExpressionException("Expression parsing failed", ex);
    }
}
```





## 依赖来源

###  Dependency Lookup









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



