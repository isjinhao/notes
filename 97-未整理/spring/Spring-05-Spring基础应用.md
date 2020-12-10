## 注解的引入

在使用Spring进行开发的时候，一般都是采用的xml文件作为配置文件，同时引入注解方便开发。所以下面便看看引入注解之后Spring做了什么初始化操作。



### context:annotation-config

在xml文件中配置 `<context:annotation-config />` 标签便可以引入注解。解析标签的代码之前见过，在类DefaultBeanDefinitionDocumentReader中。只不过之前解析bean标签的时候进入的是parseDefaultElement方法，现在进入的是parseCustomElement方法。是不是isDefaultNamespace的判断标准是标签是不是在beans这个schema里面的，此标签是在context这个schema里面定义的，所以不是默认的元素。

```java
// DefaultBeanDefinitionDocumentReader.java
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

然后会调用NamespaceHandlerSupport的parse方法。

```java
// NamespaceHandlerSupport.java
public BeanDefinition parse(Element element, ParserContext parserContext) {
    BeanDefinitionParser parser = findParserForElement(element, parserContext);
    return (parser != null ? parser.parse(element, parserContext) : null);
}
```

BeanDefinitionParser是一个接口，只有一个方法：parse。它的目的就是解析beans标签下面的自定义的标签。此标签对应的就是AnnotationConfigBeanDefinitionParser。

```java
private BeanDefinitionParser findParserForElement(Element element, ParserContext parserContext) {
    String localName = parserContext.getDelegate().getLocalName(element);
    BeanDefinitionParser parser = this.parsers.get(localName);
    if (parser == null) {
        parserContext.getReaderContext().fatal(
            "Cannot locate BeanDefinitionParser for element [" + localName + "]", element);
    }
    return parser;
}
```

在解析AnnotationConfigBeanDefinitionParser#parse方法中，我们可以看见AnnotationConfigUtils向当前的registry注册了一些Processor。

```java
public BeanDefinition parse(Element element, ParserContext parserContext) {
    Object source = parserContext.extractSource(element);
    // Obtain bean definitions for all relevant BeanPostProcessors.
    Set<BeanDefinitionHolder> processorDefinitions =
        AnnotationConfigUtils.registerAnnotationConfigProcessors(parserContext.getRegistry(), source);

    // Register component for the surrounding <context:annotation-config> element.
    CompositeComponentDefinition compDefinition = 
        new CompositeComponentDefinition(element.getTagName(), source);
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

然后就是注册过程。

```java
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
    BeanDefinitionRegistry registry, @Nullable Object source) {

    DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
    if (beanFactory != null) {
        if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
            beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
        }
        if (!(beanFactory.getAutowireCandidateResolver() 
              instanceof ContextAnnotationAutowireCandidateResolver)) {
            beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
        }
    }

    Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);

    if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, 
                                           CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

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
        // 异常处理...
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
    }

    if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
    }

    return beanDefs;
}
```

#### AnnotationAwareOrderComparator

AnnotationAwareOrderComparator是OrderComparator的子类。他在OrderComparator的基础上增加对@Order的支持，但是注解的优先级低于Bean直接继承Orderd接口。

```java
public class AnnotationAwareOrderComparator extends OrderComparator {

    public static final AnnotationAwareOrderComparator INSTANCE = new AnnotationAwareOrderComparator();

    @Override
    @Nullable
    protected Integer findOrder(Object obj) {
        Integer order = super.findOrder(obj);
        if (order != null) {
            return order;
        }
        return findOrderFromAnnotation(obj);
    }

    @Nullable
    private Integer findOrderFromAnnotation(Object obj) {
        AnnotatedElement element = (obj instanceof AnnotatedElement ? 
                                    (AnnotatedElement) obj : obj.getClass());
        MergedAnnotations annotations = MergedAnnotations.from(element, SearchStrategy.TYPE_HIERARCHY);
        Integer order = OrderUtils.getOrderFromAnnotations(element, annotations);
        if (order == null && obj instanceof DecoratingProxy) {
            return findOrderFromAnnotation(((DecoratingProxy) obj).getDecoratedClass());
        }
        return order;
    }
}
```

#### ContextAnnotationAutowireCandidateResolver

ContextAnnotationAutowireCandidateResolver是autowire的处理器，后面在依赖的处理过程会详细介绍它。

#### 添加BeanFactoryPostProcessor

ConfigurationClassPostProcessor是用于处理@Configuration注解的，如果一个Bean被加载这个注解，便会被认为是配置类，就会被解析。

```java
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
    int registryId = System.identityHashCode(registry);
    if (this.registriesPostProcessed.contains(registryId)) {
    	// 抛出异常...
    }
    if (this.factoriesPostProcessed.contains(registryId)) {
        // 抛出异常...
    }
    this.registriesPostProcessed.add(registryId);
    processConfigBeanDefinitions(registry);
}
```

```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
    String[] candidateNames = registry.getBeanDefinitionNames();

    // 便利出来所有加了@Configuration的Bean
    for (String beanName : candidateNames) {
        BeanDefinition beanDef = registry.getBeanDefinition(beanName);
        if (beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE) != null) {
            // debug info ...
        }
        else if (ConfigurationClassUtils
                 .checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
            configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
        }
    }

    // Return immediately if no @Configuration classes were found
    if (configCandidates.isEmpty()) {
        return;
    }

    // Sort by previously determined @Order value, if applicable
    configCandidates.sort((bd1, bd2) -> {
        int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
        int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
        return Integer.compare(i1, i2);
    });

    // Detect any custom bean name generation strategy 
    // supplied through the enclosing application context
    SingletonBeanRegistry sbr = null;
    if (registry instanceof SingletonBeanRegistry) {
        sbr = (SingletonBeanRegistry) registry;
        if (!this.localBeanNameGeneratorSet) {
            BeanNameGenerator generator = (BeanNameGenerator) sbr.getSingleton(
                AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR);
            if (generator != null) {
                this.componentScanBeanNameGenerator = generator;
                this.importBeanNameGenerator = generator;
            }
        }
    }

    if (this.environment == null) {
        this.environment = new StandardEnvironment();
    }

    // Parse each @Configuration class
    ConfigurationClassParser parser = new ConfigurationClassParser(
        this.metadataReaderFactory, this.problemReporter, this.environment,
        this.resourceLoader, this.componentScanBeanNameGenerator, registry);

    Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
    Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());
    do {
        // 解析加了@Configuration的Bean
        parser.parse(candidates);
        parser.validate();

        Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());
        configClasses.removeAll(alreadyParsed);

        // Read the model and create bean definitions based on its content
        if (this.reader == null) {
            this.reader = new ConfigurationClassBeanDefinitionReader(
                registry, this.sourceExtractor, this.resourceLoader, this.environment,
                this.importBeanNameGenerator, parser.getImportRegistry());
        }
        // 从配置类中加载信息
        this.reader.loadBeanDefinitions(configClasses);
        alreadyParsed.addAll(configClasses);

        candidates.clear();
        if (registry.getBeanDefinitionCount() > candidateNames.length) {
            String[] newCandidateNames = registry.getBeanDefinitionNames();
            Set<String> oldCandidateNames = new HashSet<>(Arrays.asList(candidateNames));
            Set<String> alreadyParsedClasses = new HashSet<>();
            for (ConfigurationClass configurationClass : alreadyParsed) {
                alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
            }
            for (String candidateName : newCandidateNames) {
                if (!oldCandidateNames.contains(candidateName)) {
                    BeanDefinition bd = registry.getBeanDefinition(candidateName);
                    if (ConfigurationClassUtils
                        	.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) &&
                        !alreadyParsedClasses.contains(bd.getBeanClassName())) {
                        candidates.add(new BeanDefinitionHolder(bd, candidateName));
                    }
                }
            }
            candidateNames = newCandidateNames;
        }
    }
    while (!candidates.isEmpty());

    // Register the ImportRegistry as a bean in order to support ImportAware @Configuration classes
    if (sbr != null && !sbr.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
        sbr.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());
    }

    if (this.metadataReaderFactory instanceof CachingMetadataReaderFactory) {
        // Clear cache in externally provided MetadataReaderFactory; this is a no-op
        // for a shared cache since it'll be cleared by the ApplicationContext.
        ((CachingMetadataReaderFactory) this.metadataReaderFactory).clearCache();
    }
}
```

##### 解析加了@Configuration的Bean

```java
public void parse(Set<BeanDefinitionHolder> configCandidates) {
    for (BeanDefinitionHolder holder : configCandidates) {
        BeanDefinition bd = holder.getBeanDefinition();
        try {
            if (bd instanceof AnnotatedBeanDefinition) {
                parse(((AnnotatedBeanDefinition) bd).getMetadata(), holder.getBeanName());
            } else if (bd instanceof 
                     AbstractBeanDefinition && ((AbstractBeanDefinition) bd).hasBeanClass()) {
                parse(((AbstractBeanDefinition) bd).getBeanClass(), holder.getBeanName());
            } else {
                parse(bd.getBeanClassName(), holder.getBeanName());
            }
        }
        // 异常处理...
    }
    this.deferredImportSelectorHandler.process();
}
```

```java
protected final void parse(Class<?> clazz, String beanName) throws IOException {
    processConfigurationClass(new ConfigurationClass(clazz, beanNaame));
}
```

```java
protected void processConfigurationClass(ConfigurationClass configClass) throws IOException {
    if (this.conditionEvaluator
        	.shouldSkip(configClass.getMetadata(), ConfigurationPhase.PARSE_CONFIGURATION)) {
        return;
    }

    ConfigurationClass existingClass = this.configurationClasses.get(configClass);
    if (existingClass != null) {
        if (configClass.isImported()) {
            if (existingClass.isImported()) {
                // 将需要Import的类合并进来
                existingClass.mergeImportedBy(configClass);
            }
            // Otherwise ignore new imported config class; existing non-imported class overrides it.
            return;
        }
        else {
            // Explicit bean definition found, probably replacing an import.
            // Let's remove the old one and go with the new one.
            this.configurationClasses.remove(configClass);
            this.knownSuperclasses.values().removeIf(configClass::equals);
        }
    }

    // Recursively process the configuration class and its superclass hierarchy.
    SourceClass sourceClass = asSourceClass(configClass);
    do {
        sourceClass = doProcessConfigurationClass(configClass, sourceClass);
    } while (sourceClass != null);
    this.configurationClasses.put(configClass, configClass);
}
```

```java
protected final SourceClass doProcessConfigurationClass
    (ConfigurationClass configClass, SourceClass sourceClass) throws IOException {

		if (configClass.getMetadata().isAnnotated(Component.class.getName())) {
			// Recursively process any member (nested) classes first
			processMemberClasses(configClass, sourceClass);
		}

    	// 处理@PropertySources
		// Process any @PropertySource annotations
		for (AnnotationAttributes propertySource : AnnotationConfigUtils.attributesForRepeatable(
				sourceClass.getMetadata(), PropertySources.class,
				org.springframework.context.annotation.PropertySource.class)) {
			if (this.environment instanceof ConfigurableEnvironment) {
				processPropertySource(propertySource);
			}
			else {
				// log info
			}
		}

		// Process any @ComponentScan annotations
    	// 使用ComponentScanAnnotationParser类解析
		Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
				sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
		if (!componentScans.isEmpty() &&
				!this.conditionEvaluator
            	.shouldSkip(sourceClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN)) {
			for (AnnotationAttributes componentScan : componentScans) {
				// The config class is annotated with @ComponentScan -> perform the scan immediately
				Set<BeanDefinitionHolder> scannedBeanDefinitions =
						this.componentScanParser
                    		.parse(componentScan, sourceClass.getMetadata().getClassName());
				// Check the set of scanned definitions for any 
                // further config classes and parse recursively if needed
				for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
					BeanDefinition bdCand = holder.getBeanDefinition().getOriginatingBeanDefinition();
					if (bdCand == null) {
						bdCand = holder.getBeanDefinition();
					}
					if (ConfigurationClassUtils
                        	.checkConfigurationClassCandidate(bdCand, this.metadataReaderFactory)) {
						parse(bdCand.getBeanClassName(), holder.getBeanName());
					}
				}
			}
		}

		// Process any @Import annotations
    	// @Import 导入类
    	// 并没有真正处理类，只是加入了 ConfigurationClass#importedResources集合中
		processImports(configClass, sourceClass, getImports(sourceClass), true);

		// Process any @ImportResource annotations。
    	// @ImportResource 导入xml配置文件
    	// 并没有真正处理xml文件，只是加入了 ConfigurationClass#importBeanDefinitionRegistrars集合中
		AnnotationAttributes importResource =
				AnnotationConfigUtils.attributesFor(sourceClass.getMetadata(), ImportResource.class);
		if (importResource != null) {
			String[] resources = importResource.getStringArray("locations");
			Class<? extends BeanDefinitionReader> readerClass = importResource.getClass("reader");
			for (String resource : resources) {
				String resolvedResource = this.environment.resolveRequiredPlaceholders(resource);
				configClass.addImportedResource(resolvedResource, readerClass);
			}
		}

		// Process individual @Bean methods
		Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
		for (MethodMetadata methodMetadata : beanMethods) {
			configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
		}

		// Process default methods on interfaces
		processInterfaces(configClass, sourceClass);

		// Process superclass, if any
		if (sourceClass.getMetadata().hasSuperClass()) {
			String superclass = sourceClass.getMetadata().getSuperClassName();
			if (superclass != null && !superclass.startsWith("java") &&
					!this.knownSuperclasses.containsKey(superclass)) {
				this.knownSuperclasses.put(superclass, configClass);
				// Superclass found, return its annotation metadata and recurse
				return sourceClass.getSuperClass();
			}
		}

		// No superclass -> processing is complete
		return null;
	}
```

##### 处理配置类加入的Bean

```java
// ConfigurationClassBeanDefinitionReader.java
public void loadBeanDefinitions(Set<ConfigurationClass> configurationModel) {
    TrackedConditionEvaluator trackedConditionEvaluator = new TrackedConditionEvaluator();
    for (ConfigurationClass configClass : configurationModel) {
        loadBeanDefinitionsForConfigurationClass(configClass, trackedConditionEvaluator);
    }
}
private void loadBeanDefinitionsForConfigurationClass(
    ConfigurationClass configClass, TrackedConditionEvaluator trackedConditionEvaluator) {

    if (trackedConditionEvaluator.shouldSkip(configClass)) {
        String beanName = configClass.getBeanName();
        if (StringUtils.hasLength(beanName) && this.registry.containsBeanDefinition(beanName)) {
            this.registry.removeBeanDefinition(beanName);
        }
        this.importRegistry.removeImportingClass(configClass.getMetadata().getClassName());
        return;
    }

    if (configClass.isImported()) {
        registerBeanDefinitionForImportedConfigurationClass(configClass);
    }
    for (BeanMethod beanMethod : configClass.getBeanMethods()) {
        loadBeanDefinitionsForBeanMethod(beanMethod);
    }

    // 处理之前@ImportResource导入的xml文件
    loadBeanDefinitionsFromImportedResources(configClass.getImportedResources());
    // 处理之家@Import导入的类
    loadBeanDefinitionsFromRegistrars(configClass.getImportBeanDefinitionRegistrars());
}
```

#### 添加BeanPostProcessor

AutowiredAnnotationBeanPostProcessor和CommonAnnotationBeanPostProcessor

这个在下面依赖处理过程分析。

#### 对@EventListener的支持

EventListenerMethodProcessor和DefaultEventListenerFactory。

这个在事件章节再说。



### context:component-scan

```java
public BeanDefinition parse(Element element, ParserContext parserContext) {
    String basePackage = element.getAttribute(BASE_PACKAGE_ATTRIBUTE);
    basePackage = parserContext.getReaderContext().getEnvironment().resolvePlaceholders(basePackage);
    String[] basePackages = StringUtils.tokenizeToStringArray(
        basePackage, ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);

    // Actually scan for bean definitions and register them.
    ClassPathBeanDefinitionScanner scanner = configureScanner(parserContext, element);
    Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);
    registerComponents(parserContext.getReaderContext(), beanDefinitions, element);

    return null;
}
```

```java
protected void registerComponents(
    XmlReaderContext readerContext, Set<BeanDefinitionHolder> beanDefinitions, Element element) {

    Object source = readerContext.extractSource(element);
    CompositeComponentDefinition compositeDef = 
        new CompositeComponentDefinition(element.getTagName(), source);

    for (BeanDefinitionHolder beanDefHolder : beanDefinitions) {
        compositeDef.addNestedComponent(new BeanComponentDefinition(beanDefHolder));
    }

    // Register annotation config processors, if necessary.
    boolean annotationConfig = true;
    if (element.hasAttribute(ANNOTATION_CONFIG_ATTRIBUTE)) {
        annotationConfig = Boolean.parseBoolean
            (element.getAttribute(ANNOTATION_CONFIG_ATTRIBUTE));
    }
    if (annotationConfig) {
        Set<BeanDefinitionHolder> processorDefinitions =
            AnnotationConfigUtils
            	.registerAnnotationConfigProcessors(readerContext.getRegistry(), source);
        for (BeanDefinitionHolder processorDefinition : processorDefinitions) {
            compositeDef.addNestedComponent(new BeanComponentDefinition(processorDefinition));
        }
    }
    readerContext.fireComponentRegistered(compositeDef);
}
```

component-scan标签是在annotation-config标签上加了一层扫描包下BeanDefinition的操作。ClassPathBeanDefinitionScanner是扫描类。在前面处理@Configuration注解的时候已经看到过这个类了。

ClassPathBeanDefinitionScanner是继承ClassPathScanningCandidateComponentProvider的。ClassPathBeanDefinitionScanner在被构造时就会注册过滤器。

```java
// ClassPathBeanDefinitionScanner.java
public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters,
    	Environment environment, @Nullable ResourceLoader resourceLoader) {

    Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
    this.registry = registry;

    if (useDefaultFilters) {
        registerDefaultFilters();
    }
    setEnvironment(environment);
    setResourceLoader(resourceLoader);
}
```

```java
// ClassPathScanningCandidateComponentProvider.java
private final List<TypeFilter> includeFilters = new LinkedList<>();
protected void registerDefaultFilters() {
    this.includeFilters.add(new AnnotationTypeFilter(Component.class));
    ClassLoader cl = ClassPathScanningCandidateComponentProvider.class.getClassLoader();
    try {
        this.includeFilters.add(new AnnotationTypeFilter(
            ((Class<? extends Annotation>) ClassUtils
             	.forName("javax.annotation.ManagedBean", cl)), false));
        // log
    }
    catch (ClassNotFoundException ex) {
        // JSR-250 1.1 API (as included in Java EE 6) not available - simply skip.
    }
    try {
        this.includeFilters.add(new AnnotationTypeFilter(
            ((Class<? extends Annotation>) ClassUtils
             	.forName("javax.inject.Named", cl)), false));
        // log
    }
    catch (ClassNotFoundException ex) {
        // JSR-330 API not available - simply skip.
    }
}
```

然后在查找的时候会进入doScan方法

```java
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
                AnnotationConfigUtils
                    .processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
            }
            if (checkCandidate(beanName, candidate)) {
                BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
                definitionHolder = AnnotationConfigUtils
                    .applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
                beanDefinitions.add(definitionHolder);
                registerBeanDefinition(definitionHolder, this.registry);
            }
        }
    }
    return beanDefinitions;
}
```

```java
public Set<BeanDefinition> findCandidateComponents(String basePackage) {
    if (this.componentsIndex != null && indexSupportsIncludeFilters()) {
        return addCandidateComponentsFromIndex(this.componentsIndex, basePackage);
    }
    else {
        return scanCandidateComponents(basePackage);
    }
}
```

```java
private Set<BeanDefinition> scanCandidateComponents(String basePackage) {
    Set<BeanDefinition> candidates = new LinkedHashSet<>();
    try {
        String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
            resolveBasePackage(basePackage) + '/' + this.resourcePattern;
        Resource[] resources = getResourcePatternResolver().getResources(packageSearchPath);
        boolean traceEnabled = logger.isTraceEnabled();
        boolean debugEnabled = logger.isDebugEnabled();
        for (Resource resource : resources) {
            if (traceEnabled) {
                logger.trace("Scanning " + resource);
            }
            if (resource.isReadable()) {
                try {
                    MetadataReader metadataReader = 
                        getMetadataReaderFactory().getMetadataReader(resource);
                    if (isCandidateComponent(metadataReader)) {
                        ScannedGenericBeanDefinition sbd = 
                            new ScannedGenericBeanDefinition(metadataReader);
                        sbd.setResource(resource);
                        sbd.setSource(resource);
                        if (isCandidateComponent(sbd)) {
                            // log
                            candidates.add(sbd);
                        }
                        else {
                            // log
                        }
                    }
                    else {
                        // log
                    }
                }
                // 异常...
            }
            else {
               // log
            }
        }
    }
    // 异常
    return candidates;
}
```

```java
protected boolean isCandidateComponent(MetadataReader metadataReader) throws IOException {
    for (TypeFilter tf : this.excludeFilters) {
        if (tf.match(metadataReader, getMetadataReaderFactory())) {
            return false;
        }
    }
    for (TypeFilter tf : this.includeFilters) {
        if (tf.match(metadataReader, getMetadataReaderFactory())) {
            return isConditionMatch(metadataReader);
        }
    }
    return false;
}
// 处理@Condition注解
private boolean isConditionMatch(MetadataReader metadataReader) {
    if (this.conditionEvaluator == null) {
        this.conditionEvaluator =
            new ConditionEvaluator(getRegistry(), this.environment, this.resourcePatternResolver);
    }
    return !this.conditionEvaluator.shouldSkip(metadataReader.getAnnotationMetadata());
}
```



### AnnotationConfigApplicationContext

AnnotatedBeanDefinitionReader在构造的时候就已经将需要的注解驱动设置进去了。

```java
// AnnotationConfigApplicationContext.java
public AnnotationConfigApplicationContext() {
    this.reader = new AnnotatedBeanDefinitionReader(this);
    this.scanner = new ClassPathBeanDefinitionScanner(this);
}
```

```java
// AnnotatedBeanDefinitionReader.java
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry) {
    this(registry, getOrCreateEnvironment(registry));
}
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
    Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
    Assert.notNull(environment, "Environment must not be null");
    this.registry = registry;
    this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
    AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}
```



## 依赖的处理过程

之前在填充属性时已经知道，处理依赖的接口定义为AutowireCapableBeanFactory#resolveDependency，实现是在类DefaultListableBeanFactory中。我们给一个很完整的案例，看看每种类型分别是怎么处理的。

- Bean类型
- Collection类型
- Map类型
- ObjectProvider
- 分组

### InjectionMetadata

填充属性是在AbstractAutowireCapableBeanFactory#populateBean方法里面进行的。之前提到过，在启用注解驱动的时候，registerAnnotationConfigProcessors方法会将AutowiredAnnotationBeanPostProcessor注册为BeanDefinition。AutowiredAnnotationBeanPostProcessor同时实现了MergedBeanDefinitionPostProcessor接口和InstantiationAwareBeanPostProcessor接口。其中填充属性时会调用InstantiationAwareBeanPostProcessor接口的postProcessProperties方法。

```java
public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
    InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
    try {
        metadata.inject(bean, beanName, pvs);
    }
    // 异常处理...
    return pvs;
}
```

```java
private InjectionMetadata findAutowiringMetadata
    	(String beanName, Class<?> clazz, @Nullable PropertyValues pvs) {
    // Fall back to class name as cache key, for backwards compatibility with custom callers.
    String cacheKey = (StringUtils.hasLength(beanName) ? beanName : clazz.getName());
    // Quick check on the concurrent map first, with minimal locking.
    InjectionMetadata metadata = this.injectionMetadataCache.get(cacheKey);
    if (InjectionMetadata.needsRefresh(metadata, clazz)) {
        synchronized (this.injectionMetadataCache) {
            metadata = this.injectionMetadataCache.get(cacheKey);
            if (InjectionMetadata.needsRefresh(metadata, clazz)) {
                if (metadata != null) {
                    metadata.clear(pvs);
                }
                metadata = buildAutowiringMetadata(clazz);
                this.injectionMetadataCache.put(cacheKey, metadata);
            }
        }
    }
    return metadata;
}
```

findAutowiringMetadata方法在AutowiredAnnotationBeanPostProcessor#postProcessMergedBeanDefinition方法中被调用过一次。

```java
public void postProcessMergedBeanDefinition
    	(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName) {
    InjectionMetadata metadata = findAutowiringMetadata(beanName, beanType, null);
    metadata.checkConfigMembers(beanDefinition);
}
```

所以实际上走到postProcessProperties方法时AutowiredAnnotationBeanPostProcessor#injectionMetadataCache里面一定有数据了。

#### 构造InjectionMetadata

构造metadata的方法是buildAutowiringMetadata。

```java
private InjectionMetadata buildAutowiringMetadata(final Class<?> clazz) {
    if (!AnnotationUtils.isCandidateClass(clazz, this.autowiredAnnotationTypes)) {
        return InjectionMetadata.EMPTY;
    }
    List<InjectionMetadata.InjectedElement> elements = new ArrayList<>();
    Class<?> targetClass = clazz;

    do {
        final List<InjectionMetadata.InjectedElement> currElements = new ArrayList<>();
        ReflectionUtils.doWithLocalFields(targetClass, field -> {
            MergedAnnotation<?> ann = findAutowiredAnnotation(field);
            if (ann != null) {
                if (Modifier.isStatic(field.getModifiers())) {
                    // log不支持静态属性
                    return;
                }
                boolean required = determineRequiredStatus(ann);
                currElements.add(new AutowiredFieldElement(field, required));
            }
        });

        ReflectionUtils.doWithLocalMethods(targetClass, method -> {
            Method bridgedMethod = BridgeMethodResolver.findBridgedMethod(method);
            if (!BridgeMethodResolver.isVisibilityBridgeMethodPair(method, bridgedMethod)) {
                return;
            }
            MergedAnnotation<?> ann = findAutowiredAnnotation(bridgedMethod);
            if (ann != null && method.equals(ClassUtils.getMostSpecificMethod(method, clazz))) {
                if (Modifier.isStatic(method.getModifiers())) {
                    // log不支持静态方法
                    return;
                }
                if (method.getParameterCount() == 0) {
                    // log方法参数不能为0
                }
                boolean required = determineRequiredStatus(ann);
                PropertyDescriptor pd = BeanUtils.findPropertyForMethod(bridgedMethod, clazz);
                currElements.add(new AutowiredMethodElement(method, required, pd));
            }
        });
        elements.addAll(0, currElements);
        targetClass = targetClass.getSuperclass();
    }
    while (targetClass != null && targetClass != Object.class);

    return InjectionMetadata.forElements(elements, clazz);
}
```

从代码中可以看出来，能被被注入的元素分为两类，属性和方法。

**属性**

```java
// AutowiredFieldElement.class extend InjectionMetadata.InjectedElement.class
public AutowiredFieldElement(Field field, boolean required) {
    super(field, null);
    this.required = required;
}
```

```java
// InjectedElement.class
protected InjectedElement(Member member, @Nullable PropertyDescriptor pd) {
    this.member = member;
    this.isField = (member instanceof Field);
    this.pd = pd;
}
```

**方法** 

```java
// AutowiredMethodElement.class extend InjectionMetadata.InjectedElement.class
public AutowiredMethodElement(Method method, boolean required, @Nullable PropertyDescriptor pd) {
    super(method, pd);
    this.required = required;
}
```

#### 属性注入

```java
public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
    InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
    try {
        metadata.inject(bean, beanName, pvs);
    }
	// 异常处理...
    return pvs;
}
```

每个Bean都只会执行一次postProcessProperties方法，所以metadata里面存储了需要注入的所有元素，这点在前面构造的时候也看见了。InjectionMetadata的inject是调用它自己的内部元素的inject。

```java
public void inject(Object target, @Nullable String beanName, @Nullable PropertyValues pvs) 
    	throws Throwable {
    Collection<InjectedElement> checkedElements = this.checkedElements;
    Collection<InjectedElement> elementsToIterate =
        (checkedElements != null ? checkedElements : this.injectedElements);
    if (!elementsToIterate.isEmpty()) {
        for (InjectedElement element : elementsToIterate) {
            // log
            element.inject(target, beanName, pvs);
        }
    }
}
```

属性的inject实现如下：

```java
protected void inject(Object bean, @Nullable String beanName, @Nullable PropertyValues pvs) 
    	throws Throwable {
    Field field = (Field) this.member;
    Object value;
    if (this.cached) {
        value = resolvedCachedArgument(beanName, this.cachedFieldValue);
    }
    else {
        DependencyDescriptor desc = new DependencyDescriptor(field, this.required);
        desc.setContainingClass(bean.getClass());
        Set<String> autowiredBeanNames = new LinkedHashSet<>(1);
        Assert.state(beanFactory != null, "No BeanFactory available");
        TypeConverter typeConverter = beanFactory.getTypeConverter();
        try {
            value = beanFactory.resolveDependency(desc, beanName, autowiredBeanNames, typeConverter);
        }
        // 异常处理...
        synchronized (this) {
            if (!this.cached) {
                if (value != null || this.required) {
                    this.cachedFieldValue = desc;
                    registerDependentBeans(beanName, autowiredBeanNames);
                    if (autowiredBeanNames.size() == 1) {
                        String autowiredBeanName = autowiredBeanNames.iterator().next();
                        if (beanFactory.containsBean(autowiredBeanName) &&
                            beanFactory.isTypeMatch(autowiredBeanName, field.getType())) {
                            this.cachedFieldValue = new ShortcutDependencyDescriptor(
                                desc, autowiredBeanName, field.getType());
                        }
                    }
                }
                else {
                    this.cachedFieldValue = null;
                }
                this.cached = true;
            }
        }
    }
    if (value != null) {
        ReflectionUtils.makeAccessible(field);
        field.set(bean, value);
    }
}
```

方法元素的inject实现如下：

```java
protected void inject(Object bean, @Nullable String beanName, @Nullable PropertyValues pvs) 
    	throws Throwable {
    if (checkPropertySkipping(pvs)) {
        return;
    }
    Method method = (Method) this.member;
    Object[] arguments;
    if (this.cached) {
        // Shortcut for avoiding synchronization...
        arguments = resolveCachedArguments(beanName);
    } else {
        int argumentCount = method.getParameterCount();
        arguments = new Object[argumentCount];
        DependencyDescriptor[] descriptors = new DependencyDescriptor[argumentCount];
        Set<String> autowiredBeans = new LinkedHashSet<>(argumentCount);
        Assert.state(beanFactory != null, "No BeanFactory available");
        TypeConverter typeConverter = beanFactory.getTypeConverter();
        for (int i = 0; i < arguments.length; i++) {
            MethodParameter methodParam = new MethodParameter(method, i);
            DependencyDescriptor currDesc = new DependencyDescriptor(methodParam, this.required);
            currDesc.setContainingClass(bean.getClass());
            descriptors[i] = currDesc;
            try {
                Object arg = beanFactory
                    .resolveDependency(currDesc, beanName, autowiredBeans, typeConverter);
                if (arg == null && !this.required) {
                    arguments = null;
                    break;
                }
                arguments[i] = arg;
            }
            // 异常处理
        }
        synchronized (this) {
            if (!this.cached) {
                if (arguments != null) {
                    DependencyDescriptor[] cachedMethodArguments = 
                        Arrays.copyOf(descriptors, arguments.length);
                    registerDependentBeans(beanName, autowiredBeans);
                    if (autowiredBeans.size() == argumentCount) {
                        Iterator<String> it = autowiredBeans.iterator();
                        Class<?>[] paramTypes = method.getParameterTypes();
                        for (int i = 0; i < paramTypes.length; i++) {
                            String autowiredBeanName = it.next();
                            if (beanFactory.containsBean(autowiredBeanName) &&
                                beanFactory.isTypeMatch(autowiredBeanName, paramTypes[i])) {
                                cachedMethodArguments[i] = new ShortcutDependencyDescriptor(
                                    descriptors[i], autowiredBeanName, paramTypes[i]);
                            }
                        }
                    }
                    this.cachedMethodArguments = cachedMethodArguments;
                } else {
                    this.cachedMethodArguments = null;
                }
                this.cached = true;
            }
        }
    }
    if (arguments != null) {
        try {
            ReflectionUtils.makeAccessible(method);
            method.invoke(bean, arguments);
        }
        // 异常处理 ...
    }
}
```

可以看到，属性元素和方法元素的处理都是调用AutowireCapableBeanFactory的resolveDependency实现的。对于方法，处理的是每一个参数。



### resolveDependency

```java
public Object resolveDependency(DependencyDescriptor descriptor, @Nullable String requestingBeanName,
    	@Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) 
    		throws BeansException {

    descriptor.initParameterNameDiscovery(getParameterNameDiscoverer());
    if (Optional.class == descriptor.getDependencyType()) {
        return createOptionalDependency(descriptor, requestingBeanName);
    }
    else if (ObjectFactory.class == descriptor.getDependencyType() ||
             ObjectProvider.class == descriptor.getDependencyType()) {
        return new DependencyObjectProvider(descriptor, requestingBeanName);
    }
    else if (javaxInjectProviderClass == descriptor.getDependencyType()) {
        return new Jsr330Factory().createDependencyProvider(descriptor, requestingBeanName);
    } else {
        Object result = getAutowireCandidateResolver().getLazyResolutionProxyIfNecessary(
            descriptor, requestingBeanName);
        if (result == null) {
            result = 
                doResolveDependency(descriptor, requestingBeanName, autowiredBeanNames, typeConverter);
        }
        return result;
    }
}
```

resolveDependency内表面上看分五种情况：

- Optional
- ObjectProvider
- jsr330的Provider
- Lazy
- 直接处理

#### Lazy

我们先看Lazy的情况。

我们之前看到过AutowireCandidateResolver这个接口的实现类：ContextAnnotationAutowireCandidateResolver。在AnnotationConfigUtils的registerAnnotationConfigProcessors里面会将这个类设置为DefaultListableBeanFactory的AutowireCandidateResolver。

```java
public Object getLazyResolutionProxyIfNecessary
    	(DependencyDescriptor descriptor, @Nullable String beanName) {
    return (isLazy(descriptor) ? buildLazyResolutionProxy(descriptor, beanName) : null);
}
```

##### 判断Lazy的标准

```java
protected boolean isLazy(DependencyDescriptor descriptor) {
    for (Annotation ann : descriptor.getAnnotations()) {
        Lazy lazy = AnnotationUtils.getAnnotation(ann, Lazy.class);
        if (lazy != null && lazy.value()) {
            return true;
        }
    }
    MethodParameter methodParam = descriptor.getMethodParameter();
    if (methodParam != null) {
        Method method = methodParam.getMethod();
        if (method == null || void.class == method.getReturnType()) {
            Lazy lazy = AnnotationUtils.getAnnotation(methodParam.getAnnotatedElement(), Lazy.class);
            if (lazy != null && lazy.value()) {
                return true;
            }
        }
    }
    return false;
}
```

DependencyDescriptor是InjectionPoint的子类。

```java
// InjectionPoint.java
public Annotation[] getAnnotations() {
    if (this.field != null) {
        Annotation[] fieldAnnotations = this.fieldAnnotations;
        if (fieldAnnotations == null) {
            fieldAnnotations = this.field.getAnnotations();
            this.fieldAnnotations = fieldAnnotations;
        }
        return fieldAnnotations;
    } else {
        return obtainMethodParameter().getParameterAnnotations();
    }
}
```

```java
// MethodParamter.java
public AnnotatedElement getAnnotatedElement() {
    return this.executable;
}
```

如果是属性，判断Lazy的标准是属性上有没有@Lazy。如果是方法，方法上加了@Lazy，所有的方法参数都是Lazy的，如果某个方法参数加上了@Lazy，此参数是Lazy的。

##### 构建Lazy

```java
protected Object buildLazyResolutionProxy
    	(final DependencyDescriptor descriptor, final @Nullable String beanName) {
    Assert.state(getBeanFactory() instanceof DefaultListableBeanFactory,
                 "BeanFactory needs to be a DefaultListableBeanFactory");
    final DefaultListableBeanFactory beanFactory = (DefaultListableBeanFactory) getBeanFactory();
    TargetSource ts = new TargetSource() {
        @Override
        public Class<?> getTargetClass() {
            return descriptor.getDependencyType();
        }
        @Override
        public boolean isStatic() {
            return false;
        }
        @Override
        public Object getTarget() {
            Object target = beanFactory.doResolveDependency(descriptor, beanName, null, null);
            if (target == null) {
                Class<?> type = getTargetClass();
                if (Map.class == type) {
                    return Collections.emptyMap();
                }
                else if (List.class == type) {
                    return Collections.emptyList();
                }
                else if (Set.class == type || Collection.class == type) {
                    return Collections.emptySet();
                }
                // 抛出异常...
            }
            return target;
        }
        @Override
        public void releaseTarget(Object target) {
        }
    };
    ProxyFactory pf = new ProxyFactory();
    pf.setTargetSource(ts);
    Class<?> dependencyType = descriptor.getDependencyType();
    if (dependencyType.isInterface()) {
        pf.addInterface(dependencyType);
    }
    return pf.getProxy(beanFactory.getBeanClassLoader());
}
```

构建Lazy对象是通过动态代理构建一个代理对象。AOP这块在后面再说。

#### 创建Optional对象

```java
private Optional<?> createOptionalDependency(
    	DependencyDescriptor descriptor, @Nullable String beanName, final Object... args) {

    DependencyDescriptor descriptorToUse = new NestedDependencyDescriptor(descriptor) {
        @Override
        public boolean isRequired() {
            return false;
        }
        @Override
        public Object resolveCandidate
            (String beanName, Class<?> requiredType, BeanFactory beanFactory) {
            return (!ObjectUtils.isEmpty(args) ? beanFactory.getBean(beanName, args) :
                    super.resolveCandidate(beanName, requiredType, beanFactory));
        }
    };
    Object result = doResolveDependency(descriptorToUse, beanName, null, null);
    return (result instanceof Optional ? (Optional<?>) result : Optional.ofNullable(result));
}
```

创建Optional对象的方式是使用doResolveDependency方法创建对象，再包装成Optional。

#### ObjectProvider

```java
// DependencyObjectProvider.class
private final DependencyDescriptor descriptor;
private final boolean optional;
private final String beanName;

public DependencyObjectProvider(DependencyDescriptor descriptor, @Nullable String beanName) {
    this.descriptor = new NestedDependencyDescriptor(descriptor);
    this.optional = (this.descriptor.getDependencyType() == Optional.class);
    this.beanName = beanName;
}

@Override
public Object getObject() throws BeansException {
    if (this.optional) {
        return createOptionalDependency(this.descriptor, this.beanName);
    }
    else {
        Object result = doResolveDependency(this.descriptor, this.beanName, null, null);
        if (result == null) {
            throw new NoSuchBeanDefinitionException(this.descriptor.getResolvableType());
        }
        return result;
    }
}
```

DependencyObjectProvider是BeanObjectProvider的实现类，而BeanObjectProvider继承了ObjectProvider和Serializable，所以DependencyObjectProvider就是ObjectProvider的实现类。

可以看见代码里面，包装的是Optional对象，会调用createOptionalDependency方法创建依赖，否则调用doResolveDependency方法创建对象。

#### Provider

```java
private class Jsr330Factory implements Serializable {
    public Object createDependencyProvider(DependencyDescriptor descriptor, @Nullable String beanName) {
        return new Jsr330Provider(descriptor, beanName);
    }

    private class Jsr330Provider extends DependencyObjectProvider implements Provider<Object> {
        public Jsr330Provider(DependencyDescriptor descriptor, @Nullable String beanName) {
            super(descriptor, beanName);
        }

        @Override
        @Nullable
        public Object get() throws BeansException {
            return getValue();
        }
    }
}
```

```java
// DependencyObjectProvider.class
private class DependencyObjectProvider implements BeanObjectProvider<Object> {
    protected Object getValue() throws BeansException {
        if (this.optional) {
            return createOptionalDependency(this.descriptor, this.beanName);
        }
        else {
            return doResolveDependency(this.descriptor, this.beanName, null, null);
        }
    }
}
```

```java
public interface Provider<T> {
    T get();
}
```

Provider是jsr330提出的一个标准，这里可以看见，Spring是实现Provider这个功能的时候也实现了ObjectProvider的方法。

#### 正常情况

对于注入的一个普通Bean，这里也是使用doResolveDependency方法处理。所以后面就会介绍这个方法的执行。



### DependencyDescriptor

DependencyDescriptor是用来描述依赖的。

```java
public class DependencyDescriptor extends InjectionPoint implements Serializable {

	private final Class<?> declaringClass;

	@Nullable
	private String methodName;

	@Nullable
	private Class<?>[] parameterTypes;

	private int parameterIndex;

	@Nullable
	private String fieldName;

	private final boolean required;

	private final boolean eager;

	private int nestingLevel = 1;

	@Nullable
	private Class<?> containingClass;

	@Nullable
	private transient volatile ResolvableType resolvableType;

	@Nullable
	private transient volatile TypeDescriptor typeDescriptor;
}
```



























