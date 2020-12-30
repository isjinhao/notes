## XML外部化配置

### 配置外部化属性

```java
public class XmlExternalizedConfigurationDemo {
    @Value("${user.id}")
    private Long userId;

    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext =
                new ClassPathXmlApplicationContext("META-INF/xml-ec-context.xml");

        applicationContext.refresh();
        User user = applicationContext.getBean("user", User.class);
        XmlExternalizedConfigurationDemo demo = 
            applicationContext.getBean("xmlExternalizedConfigurationDemo", 
                                       XmlExternalizedConfigurationDemo.class);
        System.out.println(demo.userId);
        System.out.println(user);
        applicationContext.close();

    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

  <!-- 启动注解后才能解析@Value注解 -->
  <context:annotation-config />

  <!-- 配置user.properties -->
  <context:property-placeholder location="classpath*:META-INF/user.properties" file-encoding="UTF-8" />

  <bean id="user" class="fsc.domain.User">
    <property name="id" value="${user.id}"/>
    <property name="name" value="${user.localName}"/>
    <property name="city" value="${user.city}"/>
    <property name="workCities" value="${user.workCities}"/>
    <property name="lifeCities">
      <list>
        <value>BEIJING</value>
        <value>SHANGHAI</value>
      </list>
    </property>
    <property name="configFileLocation" value="${user.configFileLocation}"/>
  </bean>

  <bean class="ex.xml.XmlExternalizedConfigurationDemo" id="xmlExternalizedConfigurationDemo" />

</beans>
```

#### context:property-placeholder

和annotation-config驱动一样，解析context:property-placeholder的入口也是NamespaceHandlerSupport的parsef方法：

```java
public BeanDefinition parse(Element element, ParserContext parserContext) {
    BeanDefinitionParser parser = findParserForElement(element, parserContext);
    return (parser != null ? parser.parse(element, parserContext) : null);
}
```

此时的Parser是PropertyPlaceholderBeanDefinitionParser。

```java
// AbstractBeanDefinitionParser#parse
public final BeanDefinition parse(Element element, ParserContext parserContext) {
    AbstractBeanDefinition definition = parseInternal(element, parserContext);
    if (definition != null && !parserContext.isNested()) {
        try {
            String id = resolveId(element, definition, parserContext);
            if (!StringUtils.hasText(id)) {
                parserContext.getReaderContext().error(
                    "Id is required for element '" + parserContext.getDelegate().getLocalName(element)
                    + "' when used as a top-level tag", element);
            }
            String[] aliases = null;
            if (shouldParseNameAsAliases()) {
                String name = element.getAttribute(NAME_ATTRIBUTE);
                if (StringUtils.hasLength(name)) {
                    aliases = StringUtils
                        .trimArrayElements(StringUtils.commaDelimitedListToStringArray(name));
                }
            }
            BeanDefinitionHolder holder = new BeanDefinitionHolder(definition, id, aliases);
            registerBeanDefinition(holder, parserContext.getRegistry());
            if (shouldFireEvents()) {
                BeanComponentDefinition componentDefinition = new BeanComponentDefinition(holder);
                postProcessComponentDefinition(componentDefinition);
                parserContext.registerComponent(componentDefinition);
            }
        }
        catch (BeanDefinitionStoreException ex) {
            String msg = ex.getMessage();
            parserContext.getReaderContext().error((msg != null ? msg : ex.toString()), element);
            return null;
        }
    }
    return definition;
}
```

parse方法的功能是将解析出来的BeanDefinition注册到BeanFactory中。

```java
// AbstractSingleBeanDefinitionParser.java
protected final AbstractBeanDefinition parseInternal(Element element, ParserContext parserContext) {
    BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition();
    String parentName = getParentName(element);
    if (parentName != null) {
        builder.getRawBeanDefinition().setParentName(parentName);
    }
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
    doParse(element, parserContext, builder);
    return builder.getBeanDefinition();
}
```

PropertyPlaceholderBeanDefinitionParser继承的是AbstractSingleBeanDefinitionParser，AbstractSingleBeanDefinitionParser的作用是解析一些通用属性，这里面有两个方法被交给了子类实现getBeanClass和doParse。

```java
// PropertyPlaceholderBeanDefinitionParser.java
private static final String SYSTEM_PROPERTIES_MODE_ATTRIBUTE = "system-properties-mode";
private static final String SYSTEM_PROPERTIES_MODE_DEFAULT = "ENVIRONMENT";
protected Class<?> getBeanClass(Element element) {
    // As of Spring 3.1, the default value of system-properties-mode has changed from
    // 'FALLBACK' to 'ENVIRONMENT'. This latter value indicates that resolution of
    // placeholders against system properties is a function of the Environment and
    // its current set of PropertySources.
    if (SYSTEM_PROPERTIES_MODE_DEFAULT.equals(element.getAttribute(SYSTEM_PROPERTIES_MODE_ATTRIBUTE))) {
        return PropertySourcesPlaceholderConfigurer.class;
    }

    // The user has explicitly specified a value for system-properties-mode: revert to
    // PropertyPlaceholderConfigurer to ensure backward compatibility with 3.0 and earlier.
    // This is deprecated; to be removed along with PropertyPlaceholderConfigurer itself.
    return org.springframework.beans.factory.config.PropertyPlaceholderConfigurer.class;
}
```

getBeanClass返回的是处理这个标签的类。3.1及以后默认使用的是PropertySourcesPlaceholderConfigurer，之前使用的是PropertyPlaceholderConfigurer。

```java
protected void doParse(Element element, ParserContext parserContext, BeanDefinitionBuilder builder) {
    super.doParse(element, parserContext, builder);

    builder.addPropertyValue("ignoreUnresolvablePlaceholders",
                             Boolean.valueOf(element.getAttribute("ignore-unresolvable")));

    String systemPropertiesModeName = element.getAttribute(SYSTEM_PROPERTIES_MODE_ATTRIBUTE);
    if (StringUtils.hasLength(systemPropertiesModeName) &&
        !systemPropertiesModeName.equals(SYSTEM_PROPERTIES_MODE_DEFAULT)) {
        builder.addPropertyValue("systemPropertiesModeName", 
                                 "SYSTEM_PROPERTIES_MODE_" + systemPropertiesModeName);
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

PropertySourcesPlaceholderConfigurer并不是直接继承AbstractSingleBeanDefinitionParser的，而是继承的AbstractPropertyLoadingBeanDefinitionParser，在AbstractPropertyLoadingBeanDefinitionParser也是获取了一些通用属性，比如我们的location。

```java
// AbstractPropertyLoadingBeanDefinitionParser.java
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



### PropertySourcesPlaceholderConfigurer

PropertySourcesPlaceholderConfigurer间接继承了BeanFactoryPostProcessor。所以在BeanFactory prepare之后，它的postProcessBeanFactory方法会被执行。

```java
public static final String ENVIRONMENT_PROPERTIES_PROPERTY_SOURCE_NAME = "environmentProperties";
public static final String LOCAL_PROPERTIES_PROPERTY_SOURCE_NAME = "localProperties";
public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
    if (this.propertySources == null) {
        this.propertySources = new MutablePropertySources();
        if (this.environment != null) {
            this.propertySources.addLast(
                new PropertySource<Environment>(ENVIRONMENT_PROPERTIES_PROPERTY_SOURCE_NAME, 
                                                this.environment) {
                    @Override
                    @Nullable
                    public String getProperty(String key) {
                        return this.source.getProperty(key);
                    }
                }
            );
        }
        try {
            PropertySource<?> localPropertySource =
                new PropertiesPropertySource(LOCAL_PROPERTIES_PROPERTY_SOURCE_NAME, mergeProperties());
            if (this.localOverride) {
                this.propertySources.addFirst(localPropertySource);
            } else {
                this.propertySources.addLast(localPropertySource);
            }
        }
        // 异常处理 ...
    }

    processProperties(beanFactory, new PropertySourcesPropertyResolver(this.propertySources));
    this.appliedPropertySources = this.propertySources;
}
```

在这个方法里，可以看见，系统设置了两个数据源。一个是environmentProperties，一个是localProperties。

#### PropertySource

```java
public abstract class PropertySource<T> {

	protected final Log logger = LogFactory.getLog(getClass());
	protected final String name;
	protected final T source;

	/**
	 * Create a new {@code PropertySource} with the given name and source object.
	 */
	public PropertySource(String name, T source) {
		Assert.hasText(name, "Property source name must contain at least one character");
		Assert.notNull(source, "Property source must not be null");
		this.name = name;
		this.source = source;
	}

	/**
	 * Return the underlying source object for this {@code PropertySource}.
	 */
	public T getSource() {
		return this.source;
	}

	/**
	 * Return the value associated with the given name,
	 * or {@code null} if not found.
	 * @param name the property to find
	 * @see PropertyResolver#getRequiredProperty(String)
	 */
	@Nullable
	public abstract Object getProperty(String name);
}
```

Spring对PropertySource的注释上写着。

> Abstract base class representing a source of name/value property pairs. The underlying source object may be of any type T that encapsulates properties. Examples include Properties objects, Map objects, ServletContext and ServletConfig objects (for access to init parameters).

所以我们可以知道，此类的目的就是表示封装了键值对的数据。

#### Environment封装的数据

PropertySourcesPlaceholderConfigurer实现了EnvironmentAware，同时在refresh的时候会invokeBeanFactoryPostProcessors，此时，所有的BeanFactoryProcessor会被初始化，初始化会触发AbstractAutowireCapableBeanFactory#initializeBean执行，此时会调用ApplicationContextAwareProcessor的invokeAwareInterfaces方法给environment赋值。

```java
public class PropertySourcesPlaceholderConfigurer extends PlaceholderConfigurerSupport 
    implements EnvironmentAware {
    private Environment environment;
    @Override
	public void setEnvironment(Environment environment) {
		this.environment = environment;
	}
}
```

那ApplicationContext的Environment又是什么时候初始化的呢？实际上在AbstractRefreshableConfigApplicationContext的setConfigLocations方法里解析spring-context.xml文件时就会创建。

```java
public void setConfigLocations(@Nullable String... locations) {
    if (locations != null) {
        Assert.noNullElements(locations, "Config locations must not be null");
        this.configLocations = new String[locations.length];
        for (int i = 0; i < locations.length; i++) {
            this.configLocations[i] = resolvePath(locations[i]).trim();
        }
    } else {
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
        // 这里可以看见，此时构建的Environment就是ApplicationContext的Environment
        this.environment = createEnvironment();
    }
    return this.environment;
}
protected ConfigurableEnvironment createEnvironment() {
    return new StandardEnvironment();
}
```

StandardEnvironment在构建的的时候会获取系统配置。也就是getSystemProperties()和getSystemEnvironment()方法。

```java
public class StandardEnvironment extends AbstractEnvironment {

	public static final String SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME = "systemEnvironment";

	public static final String SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME = "systemProperties";

	@Override
	protected void customizePropertySources(MutablePropertySources propertySources) {
		propertySources.addLast(
				new PropertiesPropertySource(SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME, 
                                             getSystemProperties()));
		propertySources.addLast(
				new SystemEnvironmentPropertySource(SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME, 
                                                    getSystemEnvironment()));
	}
}
```

```java
// AbstractEnvironment.java
public Map<String, Object> getSystemProperties() {
    try {
        return (Map) System.getProperties();
    }
    // 异常处理...
}
public Map<String, Object> getSystemEnvironment() {
    // 可以配置是否让Spring读取系统环境
    if (suppressGetenvAccess()) {
        return Collections.emptyMap();
    }
    try {
        return (Map) System.getenv();
    }
    // 异常处理 ...
}
```

#### 对象封装的用户配置的数据

PropertySourcesPlaceholderConfigurer继承了PlaceholderConfigurerSupport，加载用户配置文件的就是这个类。

```java
@Nullable
private Resource[] locations;
protected Properties mergeProperties() throws IOException {
    Properties result = new Properties();

    if (this.localOverride) {
        // Load properties from file upfront, to let local properties override.
        loadProperties(result);
    }

    if (this.localProperties != null) {
        for (Properties localProp : this.localProperties) {
            CollectionUtils.mergePropertiesIntoMap(localProp, result);
        }
    }

    if (!this.localOverride) {
        // Load properties from file afterwards, to let those properties override.
        loadProperties(result);
    }

    return result;
}
protected void loadProperties(Properties props) throws IOException {
    if (this.locations != null) {
        for (Resource location : this.locations) {
            if (logger.isTraceEnabled()) {
                logger.trace("Loading properties file from " + location);
            }
            try {
                PropertiesLoaderUtils.fillProperties(
                    props, new EncodedResource(location, this.fileEncoding), this.propertiesPersister);
            }
            // 异常处理 ...
        }
    }
}
```

填充属性的方法是PropertiesLoaderUtils的fillProperties。

```java
// PropertiesLoaderUtils.java
private static final String XML_FILE_EXTENSION = ".xml";
static void fillProperties(Properties props, EncodedResource resource, PropertiesPersister persister)
    	throws IOException {

    InputStream stream = null;
    Reader reader = null;
    try {
        String filename = resource.getResource().getFilename();
        if (filename != null && filename.endsWith(XML_FILE_EXTENSION)) {
            stream = resource.getInputStream();
            persister.loadFromXml(props, stream);
        }
        else if (resource.requiresReader()) {
            reader = resource.getReader();
            persister.load(props, reader);
        }
        else {
            stream = resource.getInputStream();
            persister.load(props, stream);
        }
    }
    finally {
        // 关闭流
    }
}
```

我们现在的Persister是DefaultPropertiesPersister。

```java
public class DefaultPropertiesPersister implements PropertiesPersister {
	@Override
	public void load(Properties props, InputStream is) throws IOException {
		props.load(is);
	}

	@Override
	public void load(Properties props, Reader reader) throws IOException {
		props.load(reader);
	}

	@Override
	public void loadFromXml(Properties props, InputStream is) throws IOException {
		props.loadFromXML(is);
	}
}
```

DefaultPropertiesPersister调用JDK里Properties类的方法加载Properties文件。

```java
// PropertySourcesPlaceholderConfigurer.java
protected void processProperties(ConfigurableListableBeanFactory beanFactoryToProcess,
		final ConfigurablePropertyResolver propertyResolver) throws BeansException {

    propertyResolver.setPlaceholderPrefix(this.placeholderPrefix);
    propertyResolver.setPlaceholderSuffix(this.placeholderSuffix);
    propertyResolver.setValueSeparator(this.valueSeparator);

    // 真正用于处理外部配置的是这个lambda表达式
    StringValueResolver valueResolver = strVal -> {
        String resolved = (this.ignoreUnresolvablePlaceholders ?
                           propertyResolver.resolvePlaceholders(strVal) :
                           propertyResolver.resolveRequiredPlaceholders(strVal));
        if (this.trimValues) {
            resolved = resolved.trim();
        }
        return (resolved.equals(this.nullValue) ? null : resolved);
    };

    // beanFactoryToProcess是BeanFactoryPostProcessor方法传过来的
    doProcessProperties(beanFactoryToProcess, valueResolver);
}
```

```java
// PlaceholderConfigurerSupport.java
protected void doProcessProperties(ConfigurableListableBeanFactory beanFactoryToProcess,
                                   StringValueResolver valueResolver) {

    BeanDefinitionVisitor visitor = new BeanDefinitionVisitor(valueResolver);

    String[] beanNames = beanFactoryToProcess.getBeanDefinitionNames();
    for (String curName : beanNames) {
        // Check that we're not parsing our own bean definition,
        // to avoid failing on unresolvable placeholders in properties file locations.
        if (!(curName.equals(this.beanName) && beanFactoryToProcess.equals(this.beanFactory))) {
            BeanDefinition bd = beanFactoryToProcess.getBeanDefinition(curName);
            try {
                visitor.visitBeanDefinition(bd);
            }
            // 抛出异常 ...
        }
    }

    // New in Spring 2.5: resolve placeholders in alias target names and aliases as well.
    beanFactoryToProcess.resolveAliases(valueResolver);

    // New in Spring 3.0: resolve placeholders in embedded values such as annotation attributes.
    // 这里添加的EmbeddedValueResolver就是后面用于处理配置信息的对象
    beanFactoryToProcess.addEmbeddedValueResolver(valueResolver);
}
```

### @Value

之前提到doResolveDependency方法会处理@Value这个注解。@Value便是引入外部化配置的经典方式。

```java
public Object doResolveDependency(DependencyDescriptor descriptor, @Nullable String beanName,
    	@Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter)
    		throws BeansException {

    InjectionPoint previousInjectionPoint = ConstructorResolver.setCurrentInjectionPoint(descriptor);
    try {
		// shortcut ...

        Class<?> type = descriptor.getDependencyType();
        Object value = getAutowireCandidateResolver().getSuggestedValue(descriptor);
        if (value != null) {
            if (value instanceof String) {
                String strVal = resolveEmbeddedValue((String) value);
                BeanDefinition bd = (beanName != null && containsBean(beanName) ?
                                     getMergedBeanDefinition(beanName) : null);
                value = evaluateBeanDefinitionString(strVal, bd);
            }
            TypeConverter converter = (typeConverter != null ? typeConverter : getTypeConverter());
            try {
                return converter.convertIfNecessary(value, type, descriptor.getTypeDescriptor());
            }
            catch (UnsupportedOperationException ex) {
                // A custom TypeConverter which does not support TypeDescriptor resolution...
                return (descriptor.getField() != null ?
                        converter.convertIfNecessary(value, type, descriptor.getField()) :
                        converter.convertIfNecessary(value, type, descriptor.getMethodParameter()));
            }
        }

        // 处理其他情况 ...
    } finally {
        ConstructorResolver.setCurrentInjectionPoint(previousInjectionPoint);
    }
}
```

#### getSuggestedValue

getSuggestedValue方法的实现在类QualifierAnnotationAutowireCandidateResolver类中。

```java
// QualifierAnnotationAutowireCandidateResolver.java
public Object getSuggestedValue(DependencyDescriptor descriptor) {
    Object value = findValue(descriptor.getAnnotations());
    if (value == null) {
        MethodParameter methodParam = descriptor.getMethodParameter();
        if (methodParam != null) {
            value = findValue(methodParam.getMethodAnnotations());
        }
    }
    return value;
}

private Class<? extends Annotation> valueAnnotationType = Value.class;
protected Object findValue(Annotation[] annotationsToSearch) {
    if (annotationsToSearch.length > 0) {   // qualifier annotations have to be local
        // 这个会获取到@Value注解的属性值，用键值对保存
        AnnotationAttributes attr = AnnotatedElementUtils.getMergedAnnotationAttributes(
            AnnotatedElementUtils.forAnnotations(annotationsToSearch), this.valueAnnotationType);
        if (attr != null) {
            return extractValue(attr);
        }
    }
    return null;
}

// 获得注解里value属性的值
protected Object extractValue(AnnotationAttributes attr) {
    Object value = attr.get(AnnotationUtils.VALUE);
    if (value == null) {
        throw new IllegalStateException("Value annotation must have a value attribute");
    }
    return value;
}
```

可以看出，上面的代码就是将@Value里value属性的值取出来。取出来的是占位符，如${user.id}这种。

#### StringValueResolver解析

取出来的值，通过StringValueResolver解析。

```java
// AbstractBeanFactory.java
private final List<StringValueResolver> embeddedValueResolvers = new CopyOnWriteArrayList<>();
public String resolveEmbeddedValue(@Nullable String value) {
    if (value == null) {
        return null;
    }
    String result = value;
    // 这里的embeddedValueResolvers就是之前的那个lambda表达式
    for (StringValueResolver resolver : this.embeddedValueResolvers) {
        result = resolver.resolveStringValue(result);
        if (result == null) {
            return null;
        }
    }
    return result;
}
```

如果没有启动property-placeholder标签，这个属性会在refresh方法的finishBeanFactoryInitialization方法中被设置。

```java
// AbstractApplicationContext.java 	
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // Register a default embedded value resolver if no bean post-processor
    // (such as a PropertyPlaceholderConfigurer bean) registered any before:
    // at this point, primarily for resolution in annotation attribute values.
    if (!beanFactory.hasEmbeddedValueResolver()) {
        beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
    }
}
```

在finishBeanFactoryInitialization方法中添加的在这个行为，就是对于任意一个字符串，都调用Environment的resolvePlaceholders方法进行处理。Environment的默认实现类是StandardEnvironment。

```java
// StandardEnvironment.java
private final MutablePropertySources propertySources = new MutablePropertySources();
private final ConfigurablePropertyResolver propertyResolver = 
    						new PropertySourcesPropertyResolver(this.propertySources);

public String resolvePlaceholders(String text) {
    return this.propertyResolver.resolvePlaceholders(text);
}
```

从代码中可以看到，此时只会存在系统属性。但此两种情景都是使用PropertySourcesPropertyResolver解析的占位符。

#### PropertySourcesPropertyResolver

这个Lambda是解析占位符的核心。

```java
public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) 
    	throws BeansException {
	// ...
    processProperties(beanFactory, new PropertySourcesPropertyResolver(this.propertySources));
    this.appliedPropertySources = this.propertySources;
}
protected void processProperties(ConfigurableListableBeanFactory beanFactoryToProcess,
		final ConfigurablePropertyResolver propertyResolver) throws BeansException {

    // ...
    StringValueResolver valueResolver = strVal -> {
        String resolved = (this.ignoreUnresolvablePlaceholders ?
                           propertyResolver.resolvePlaceholders(strVal) :
                           propertyResolver.resolveRequiredPlaceholders(strVal));
        if (this.trimValues) {
            resolved = resolved.trim();
        }
        return (resolved.equals(this.nullValue) ? null : resolved);
    };

    doProcessProperties(beanFactoryToProcess, valueResolver);
}
```

Lambda里面的propertyResolver就是包含配置文件的解析器。

```java
// AbstractPropertyResolver.java
// PropertySourcesPropertyResolver 继承 AbstractPropertyResolver
public String resolvePlaceholders(String text) {
    if (this.nonStrictHelper == null) {
        this.nonStrictHelper = createPlaceholderHelper(true);
    }
    return doResolvePlaceholders(text, this.nonStrictHelper);
}
private PropertyPlaceholderHelper createPlaceholderHelper(boolean ignoreUnresolvablePlaceholders) {
    return new PropertyPlaceholderHelper(this.placeholderPrefix, this.placeholderSuffix,
                                         this.valueSeparator, ignoreUnresolvablePlaceholders);
}

private String doResolvePlaceholders(String text, PropertyPlaceholderHelper helper) {
    // this::getPropertyAsRawString 这个lambda是获取配置的核心
    return helper.replacePlaceholders(text, this::getPropertyAsRawString);
}

// this::getPropertyAsRawString 的实现
protected String getPropertyAsRawString(String key) {
    return getProperty(key, String.class, false);
}

@Nullable
protected <T> T getProperty(String key, Class<T> targetValueType, boolean resolveNestedPlaceholders) {
    if (this.propertySources != null) {
        for (PropertySource<?> propertySource : this.propertySources) {
            if (logger.isTraceEnabled()) {
                logger.trace("Searching for key '" + key + "' in PropertySource '" +
                             propertySource.getName() + "'");
            }
            // 本质就是便利所有的PropertySource
            // PropertySource是有序的，所以会先查找系统变量
            Object value = propertySource.getProperty(key);
            if (value != null) {
                if (resolveNestedPlaceholders && value instanceof String) {
                    value = resolveNestedPlaceholders((String) value);
                }
                logKeyFound(key, propertySource, value);
                return convertValueIfNecessary(value, targetValueType);
            }
        }
    }
    if (logger.isTraceEnabled()) {
        logger.trace("Could not find key '" + key + "' in any property source");
    }
    return null;
}
```

构建PropertyPlaceholderHelper。


```java
// PropertyPlaceholderHelper.java
private static final Map<String, String> wellKnownSimplePrefixes = new HashMap<>(4);
static {
    wellKnownSimplePrefixes.put("}", "{");
    wellKnownSimplePrefixes.put("]", "[");
    wellKnownSimplePrefixes.put(")", "(");
}

public PropertyPlaceholderHelper(String placeholderPrefix, String placeholderSuffix,
    	@Nullable String valueSeparator, boolean ignoreUnresolvablePlaceholders) {

    Assert.notNull(placeholderPrefix, "'placeholderPrefix' must not be null");
    Assert.notNull(placeholderSuffix, "'placeholderSuffix' must not be null");
    this.placeholderPrefix = placeholderPrefix;
    this.placeholderSuffix = placeholderSuffix;
    String simplePrefixForSuffix = wellKnownSimplePrefixes.get(this.placeholderSuffix);
    if (simplePrefixForSuffix != null && this.placeholderPrefix.endsWith(simplePrefixForSuffix)) {
        this.simplePrefix = simplePrefixForSuffix;
    } else {
        this.simplePrefix = this.placeholderPrefix;
    }
    this.valueSeparator = valueSeparator;
    this.ignoreUnresolvablePlaceholders = ignoreUnresolvablePlaceholders;
}
```

PropertyPlaceholderHelper解析占位符的方法就是这个。

```java
public String replacePlaceholders(String value, PlaceholderResolver placeholderResolver) {
    Assert.notNull(value, "'value' must not be null");
    return parseStringValue(value, placeholderResolver, null);
}
// 这个lambda就是this::getPropertyAsRawString
protected String parseStringValue(
    String value, PlaceholderResolver placeholderResolver, @Nullable Set<String> visitedPlaceholders) {

    int startIndex = value.indexOf(this.placeholderPrefix);
    if (startIndex == -1) {
        return value;
    }

    StringBuilder result = new StringBuilder(value);
    while (startIndex != -1) {
        int endIndex = findPlaceholderEndIndex(result, startIndex);
        if (endIndex != -1) {
            String placeholder = result.substring(
                							startIndex + this.placeholderPrefix.length(), endIndex);
            String originalPlaceholder = placeholder;
            if (visitedPlaceholders == null) {
                visitedPlaceholders = new HashSet<>(4);
            }
            if (!visitedPlaceholders.add(originalPlaceholder)) {
                // 异常处理 ...
            }
            // Recursive invocation, parsing placeholders contained in the placeholder key.
            placeholder = parseStringValue(placeholder, placeholderResolver, visitedPlaceholders);
            // Now obtain the value for the fully resolved key...
            // 通过key寻找值。
            String propVal = placeholderResolver.resolvePlaceholder(placeholder);
            if (propVal == null && this.valueSeparator != null) {
                int separatorIndex = placeholder.indexOf(this.valueSeparator);
                if (separatorIndex != -1) {
                    String actualPlaceholder = placeholder.substring(0, separatorIndex);
                    String defaultValue = placeholder
                        					.substring(separatorIndex + this.valueSeparator.length());
                    propVal = placeholderResolver.resolvePlaceholder(actualPlaceholder);
                    if (propVal == null) {
                        propVal = defaultValue;
                    }
                }
            }
            if (propVal != null) {
                // Recursive invocation, parsing placeholders contained in the
                // previously resolved placeholder value.
                propVal = parseStringValue(propVal, placeholderResolver, visitedPlaceholders);
                result.replace(startIndex, endIndex + this.placeholderSuffix.length(), propVal);
                if (logger.isTraceEnabled()) {
                    logger.trace("Resolved placeholder '" + placeholder + "'");
                }
                startIndex = result.indexOf(this.placeholderPrefix, startIndex + propVal.length());
            }
            else if (this.ignoreUnresolvablePlaceholders) {
                // Proceed with unprocessed value.
                startIndex = 
                    result.indexOf(this.placeholderPrefix, endIndex + this.placeholderSuffix.length());
            }
            else {
                throw new IllegalArgumentException("Could not resolve placeholder '" +
                                                   placeholder + "'" + " in value \"" + value + "\"");
            }
            visitedPlaceholders.remove(originalPlaceholder);
        }
        else {
            startIndex = -1;
        }
    }
    return result.toString();
}
```



## Annotation外部化配置

```java
@PropertySource("classpath:META-INF/user.properties")
public class AnnotationExternalizedConfigurationDemo {

    @Value("${user.id}")
    private Long userId;

    @Bean(value = "user")
    public User createAnnotationUser(@Value("${user.id}") Long id, 
        	@Value("${user.localName}") String userName, @Value("${user.workCities}") City[] cities) {
        User user = new User();
        user.setName(userName);
        user.setId(id);
        user.setWorkCities(cities);
        return user;
    }

    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext =
            new AnnotationConfigApplicationContext();
        applicationContext.register(AnnotationExternalizedConfigurationDemo.class);

        applicationContext.refresh();
        User user = applicationContext.getBean("user", User.class);
        AnnotationExternalizedConfigurationDemo demo = 
            applicationContext.getBean("annotationExternalizedConfigurationDemo", 
                                       			AnnotationExternalizedConfigurationDemo.class);
        System.out.println(demo.userId);
        System.out.println(user);
        applicationContext.close();
    }
}
```

AnnotationConfigUtils#registerAnnotationConfigProcessors会注入ConfigurationClassPostProcessor这个Bean，这个Bean是BeanFactoryProcessor，最终会调用到ConfigurationClassParser的doProcessConfigurationClass。

```java
// ConfigurationClassParser.java
@Nullable
protected final SourceClass doProcessConfigurationClass(ConfigurationClass configClass,
    	SourceClass sourceClass) throws IOException {
    if (configClass.getMetadata().isAnnotated(Component.class.getName())) {
        // Recursively process any member (nested) classes first
        processMemberClasses(configClass, sourceClass);
    }

    // Process any @PropertySource annotations
    // 还有 @PropertySources 也可以被处理。
    for (AnnotationAttributes propertySource : AnnotationConfigUtils.attributesForRepeatable(
        sourceClass.getMetadata(), PropertySources.class,
        org.springframework.context.annotation.PropertySource.class)) {
        if (this.environment instanceof ConfigurableEnvironment) {
            processPropertySource(propertySource);
        }
    }

    // process @Import, 
    return null;
}
```

processPropertySource方法就是将PropertySource加入Environment的PropertySources中。

```java
private void processPropertySource(AnnotationAttributes propertySource) throws IOException {
    String name = propertySource.getString("name");
    if (!StringUtils.hasLength(name)) {
        name = null;
    }
    String encoding = propertySource.getString("encoding");
    if (!StringUtils.hasLength(encoding)) {
        encoding = null;
    }
    String[] locations = propertySource.getStringArray("value");
    Assert.isTrue(locations.length > 0, "At least one @PropertySource(value) location is required");
    boolean ignoreResourceNotFound = propertySource.getBoolean("ignoreResourceNotFound");

    Class<? extends PropertySourceFactory> factoryClass = propertySource.getClass("factory");
    PropertySourceFactory factory = (factoryClass == PropertySourceFactory.class ?
    					DEFAULT_PROPERTY_SOURCE_FACTORY : BeanUtils.instantiateClass(factoryClass));

    for (String location : locations) {
        try {
            String resolvedLocation = this.environment.resolveRequiredPlaceholders(location);
            Resource resource = this.resourceLoader.getResource(resolvedLocation);
            addPropertySource(factory
                              .createPropertySource(name, new EncodedResource(resource, encoding)));
        }
        // 处理异常 ... 
    }
}

private void addPropertySource(PropertySource<?> propertySource) {
    String name = propertySource.getName();
    MutablePropertySources propertySources = 
        ((ConfigurableEnvironment) this.environment).getPropertySources();

    if (this.propertySourceNames.contains(name)) {
        // We've already added a version, we need to extend it
        PropertySource<?> existing = propertySources.get(name);
        if (existing != null) {
            PropertySource<?> newSource = (propertySource instanceof ResourcePropertySource ?
            			((ResourcePropertySource) propertySource).withResourceName() : propertySource);
            if (existing instanceof CompositePropertySource) {
                ((CompositePropertySource) existing).addFirstPropertySource(newSource);
            }
            else {
                if (existing instanceof ResourcePropertySource) {
                    existing = ((ResourcePropertySource) existing).withResourceName();
                }
                CompositePropertySource composite = new CompositePropertySource(name);
                composite.addPropertySource(newSource);
                composite.addPropertySource(existing);
                propertySources.replace(name, composite);
            }
            return;
        }
    }

    if (this.propertySourceNames.isEmpty()) {
        // propertySources就是Environment的propertySources
        propertySources.addLast(propertySource);
    } else {
        String firstProcessed = this.propertySourceNames.get(this.propertySourceNames.size() - 1);
        propertySources.addBefore(firstProcessed, propertySource);
    }
    this.propertySourceNames.add(name);
}
```

此时没有初始化BeanFactory的embeddedValueResolvers，所以finishBeanFactoryInitialization时会加入一个Lambda来解析占位符。

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // ...
    if (!beanFactory.hasEmbeddedValueResolver()) {
        beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
    }
    // ...
}
```

















