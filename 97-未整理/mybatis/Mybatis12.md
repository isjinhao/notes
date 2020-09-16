## parseConfiguration()

```java
private void parseConfiguration(XNode root) {
    try {
        // issue #117 read properties first
        propertiesElement(root.evalNode("properties"));
        Properties settings = settingsAsProperties(root.evalNode("settings"));
        loadCustomVfs(settings);
        typeAliasesElement(root.evalNode("typeAliases"));
        pluginElement(root.evalNode("plugins"));
        objectFactoryElement(root.evalNode("objectFactory"));
        objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
        reflectorFactoryElement(root.evalNode("reflectorFactory"));
        settingsElement(settings);
        // read it after objectFactory and objectWrapperFactory issue #631
        environmentsElement(root.evalNode("environments"));
        databaseIdProviderElement(root.evalNode("databaseIdProvider"));
        typeHandlerElement(root.evalNode("typeHandlers"));
        mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
        throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
}
```

`parseConfiguration()` 方法的作用就是解析 `/configuration` 标签下的十一个标签。



### properties

```java
private void propertiesElement(XNode context) throws Exception {
    if (context != null) {
        // 获得子节点的属性值，形成键值对形式，子节点就是 property 标签
        // <property name="abc" value="123"/>
        Properties defaults = context.getChildrenAsProperties();
        // 加载 properties 标签上 resource 属性对应的值
        String resource = context.getStringAttribute("resource");
        // 加载 properties 标签上 url 属性对应的值
        String url = context.getStringAttribute("url");
        // resource 和 url 只允许同时出现一个
        if (resource != null && url != null) {
            throw new BuilderException("...");
        }
        // 把 resource 或 url 指定的外部链接的配置加载进来
        if (resource != null) {
            defaults.putAll(Resources.getResourceAsProperties(resource));
        } else if (url != null) {
            defaults.putAll(Resources.getUrlAsProperties(url));
        }
        // 需要先获得默认的 Properties 是因为 SqlSessionFactory 的 build 方法有可以传入 Properties 的重载形式
        // public SqlSessionFactory build(Reader reader, Properties properties);
        Properties vars = configuration.getVariables();
        if (vars != null) {
            defaults.putAll(vars);
        }
        parser.setVariables(defaults);
        configuration.setVariables(defaults);
    }
}

public Properties getChildrenAsProperties() {
    Properties properties = new Properties();
    for (XNode child : getChildren()) {
        String name = child.getStringAttribute("name");
        String value = child.getStringAttribute("value");
        if (name != null && value != null) {
            properties.setProperty(name, value);
        }
    }
    return properties;
}
```

properties 节点的加载比较简单，我们不再继续叙述。我们需要注意的是，在加载的过程中是不进行解析的，比如在property中使用了 `${}` 表达式，加载之后也只会把表达式加载进来。如下面的配置文件和图所示。

```xml
<properties resource="db.properties">
    <property name="abc" value="${123:aha}"/>
</properties>
```

<div align="center"><img style="width:50%; " src="image/2020-09-07_163657.png" /></div>



### settings

```java
private final ReflectorFactory localReflectorFactory = new DefaultReflectorFactory();

private Properties settingsAsProperties(XNode context) {
    if (context == null) {
        return new Properties();
    }
    Properties props = context.getChildrenAsProperties();
    // Check that all settings are known to the configuration class
    MetaClass metaConfig = MetaClass.forClass(Configuration.class, localReflectorFactory);
    for (Object key : props.keySet()) {
        if (!metaConfig.hasSetter(String.valueOf(key))) {
            throw new BuilderException("The setting " + key + " is not known.  Make sure you spelled it correctly (case sensitive).");
        }
    }
    return props;
}

private void settingsElement(Properties props) throws Exception {
    configuration.setAutoMappingBehavior(AutoMappingBehavior.valueOf(props.getProperty("autoMappingBehavior", "PARTIAL")));
    configuration.setAutoMappingUnknownColumnBehavior(AutoMappingUnknownColumnBehavior.valueOf(props.getProperty("autoMappingUnknownColumnBehavior", "NONE")));
    configuration.setCacheEnabled(booleanValueOf(props.getProperty("cacheEnabled"), true));
    configuration.setProxyFactory((ProxyFactory) createInstance(props.getProperty("proxyFactory")));
    configuration.setLazyLoadingEnabled(booleanValueOf(props.getProperty("lazyLoadingEnabled"), false));
    configuration.setAggressiveLazyLoading(booleanValueOf(props.getProperty("aggressiveLazyLoading"), false));
    configuration.setMultipleResultSetsEnabled(booleanValueOf(props.getProperty("multipleResultSetsEnabled"), true));
    configuration.setUseColumnLabel(booleanValueOf(props.getProperty("useColumnLabel"), true));
    configuration.setUseGeneratedKeys(booleanValueOf(props.getProperty("useGeneratedKeys"), false));
    configuration.setDefaultExecutorType(ExecutorType.valueOf(props.getProperty("defaultExecutorType", "SIMPLE")));
    configuration.setDefaultStatementTimeout(integerValueOf(props.getProperty("defaultStatementTimeout"), null));
    configuration.setDefaultFetchSize(integerValueOf(props.getProperty("defaultFetchSize"), null));
    configuration.setMapUnderscoreToCamelCase(booleanValueOf(props.getProperty("mapUnderscoreToCamelCase"), false));
    configuration.setSafeRowBoundsEnabled(booleanValueOf(props.getProperty("safeRowBoundsEnabled"), false));
    configuration.setLocalCacheScope(LocalCacheScope.valueOf(props.getProperty("localCacheScope", "SESSION")));
    configuration.setJdbcTypeForNull(JdbcType.valueOf(props.getProperty("jdbcTypeForNull", "OTHER")));
    configuration.setLazyLoadTriggerMethods(stringSetValueOf(props.getProperty("lazyLoadTriggerMethods"), "equals,clone,hashCode,toString"));
    configuration.setSafeResultHandlerEnabled(booleanValueOf(props.getProperty("safeResultHandlerEnabled"), true));
    configuration.setDefaultScriptingLanguage(resolveClass(props.getProperty("defaultScriptingLanguage")));
    @SuppressWarnings("unchecked")
    Class<? extends TypeHandler> typeHandler = (Class<? extends TypeHandler>)resolveClass(props.getProperty("defaultEnumTypeHandler"));
    configuration.setDefaultEnumTypeHandler(typeHandler);
    configuration.setCallSettersOnNulls(booleanValueOf(props.getProperty("callSettersOnNulls"), false));
    configuration.setUseActualParamName(booleanValueOf(props.getProperty("useActualParamName"), true));
    configuration.setReturnInstanceForEmptyRow(booleanValueOf(props.getProperty("returnInstanceForEmptyRow"), false));
    configuration.setLogPrefix(props.getProperty("logPrefix"));
    @SuppressWarnings("unchecked")
    Class<? extends Log> logImpl = (Class<? extends Log>)resolveClass(props.getProperty("logImpl"));
    configuration.setLogImpl(logImpl);
    configuration.setConfigurationFactory(resolveClass(props.getProperty("configurationFactory")));
}
```

settings是整个配置中最繁杂的配置，我们只介绍其中常用的一些属性。实际上我也只明白常用的属性。在解释这个标签之前，还是先解释一些基础类。

#### 属性工具类

属性工具类一共有三个，`PropertyNamer`、`PropertyCopier` 和 `PropertyTokenizer`。

```java
public final class PropertyNamer {
    private PropertyNamer() {
    }
    
    // 由方法名得到属性名
    public static String methodToProperty(String name) {
        if (name.startsWith("is")) {
            name = name.substring(2);
        } else if (name.startsWith("get") || name.startsWith("set")) {
            name = name.substring(3);
        } else {
            throw new ReflectionException("Error parsing property name '" + 
                                          name + "'.  Didn't start with 'is', 'get' or 'set'.");
        }
        if (name.length() == 1 || (name.length() > 1 && !Character.isUpperCase(name.charAt(1)))) {
            name = name.substring(0, 1).toLowerCase(Locale.ENGLISH) + name.substring(1);
        }
        return name;
    }
    public static boolean isProperty(String name) {
        return name.startsWith("get") || name.startsWith("set") || name.startsWith("is");
    }
    public static boolean isGetter(String name) {
        return name.startsWith("get") || name.startsWith("is");
    }
    public static boolean isSetter(String name) {
        return name.startsWith("set");
    }
}
```

```java
public final class PropertyCopier {
    private PropertyCopier() {
    }

    // 属性拷贝
    public static void copyBeanProperties(Class<?> type, Object sourceBean, Object destinationBean) {
        Class<?> parent = type;
        while (parent != null) {
            final Field[] fields = parent.getDeclaredFields();
            for(Field field : fields) {
                try {
                    field.setAccessible(true);
                    field.set(destinationBean, field.get(sourceBean));
                } catch (Exception e) {
                    // Nothing useful to do, will only fail on final fields, which will be ignored.
                }
            }
            parent = parent.getSuperclass();
        }
    }
}
```

这两个比较简单，就不在赘述。我们主要看看 `PropertyTokenizer`。

```java
public class PropertyTokenizer implements Iterator<PropertyTokenizer> {
    private String name;
    private final String indexedName;
    private String index;
    private final String children;

    public PropertyTokenizer(String fullname) {
        int delim = fullname.indexOf('.');
        if (delim > -1) {
            name = fullname.substring(0, delim);
            children = fullname.substring(delim + 1);
        } else {
            name = fullname;
            children = null;
        }
        indexedName = name;
        delim = name.indexOf('[');
        if (delim > -1) {
            index = name.substring(delim + 1, name.length() - 1);
            name = name.substring(0, delim);
        }
    }
    public String getName() {
        return name;
    }
    public String getIndex() {
        return index;
    }
    public String getIndexedName() {
        return indexedName;
    }
    public String getChildren() {
        return children;
    }
    @Override
    public boolean hasNext() {
        return children != null;
    }
    @Override
    public PropertyTokenizer next() {
        return new PropertyTokenizer(children);
    }
    @Override
    public void remove() {
        throw new UnsupportedOperationException("Remove is not supported ...");
    }
}
```

这个类是一个分词器，主要的作用是解析集合或者Map元素的String表达形式。下面是一个测试的Demo。

```java
public class TestPropertyTokenizer {
    public static void main(String[] args) {
        PropertyTokenizer propertyTokenizer = new PropertyTokenizer("orders[0].item[0].name");
        show(propertyTokenizer);
        show(propertyTokenizer.next());
        show(propertyTokenizer.next().next());
    }
    private static void show(PropertyTokenizer propertyTokenizer) {
        System.out.println(propertyTokenizer.getIndex());
        System.out.println(propertyTokenizer.getIndexedName());
        System.out.println(propertyTokenizer.getName());
        System.out.println(propertyTokenizer.getChildren());
        System.out.println("---------------------------");
    }
}
        /**
         * 0
         * orders[0]
         * orders
         * item[0].name
         * ---------------------------
         * 0
         * item[0]
         * item
         * name
         * ---------------------------
         * null
         * name
         * name
         * null
         * ---------------------------
         */
```

<div align="center"><img style="width:40%; " src="image/2020-09-07_190457.png" /></div>

#### MetaClass

```java
public class MetaClass {
    private final ReflectorFactory reflectorFactory;
    private final Reflector reflector;
}
```

MetaClass 通过应用Reflector和PropertyTokenizer实现了对复杂属性表达式解析的能力。

<div align="center"><img style="width:30%; " src="image/2020-09-07_182935.png" /></div>

我们来看一下它解析属性的实现过程，即 `findProperty()` 方法。

```java
public String findProperty(String name) {
    StringBuilder prop = buildProperty(name, new StringBuilder());
    return prop.length() > 0 ? prop.toString() : null;
}
// 递归调用。
private StringBuilder buildProperty(String name, StringBuilder builder) {
    PropertyTokenizer prop = new PropertyTokenizer(name);
    // 还有下一个的时候就递归
    if (prop.hasNext()) {
        // 如果当前属性找不着，就直接return了
        String propertyName = reflector.findPropertyName(prop.getName());
        if (propertyName != null) {
            builder.append(propertyName);
            builder.append(".");
            MetaClass metaProp = metaClassForProperty(propertyName);
            metaProp.buildProperty(prop.getChildren(), builder);
        }
    } else {
        // 最后一个的时候就不递归了
        String propertyName = reflector.findPropertyName(name);
        if (propertyName != null) {
            builder.append(propertyName);
        }
    }
    return builder;
}
public MetaClass metaClassForProperty(String name) {
    Class<?> propType = reflector.getGetterType(name);
    return MetaClass.forClass(propType, reflectorFactory);
}
public static MetaClass forClass(Class<?> type, ReflectorFactory reflectorFactory) {
    return new MetaClass(type, reflectorFactory);
}
```

从下面的Demo中可以看出来 `findProperty()` 的功能。

```java
public class TestMetaClass {
    public static void main(String[] args) {
        MetaClass metaClass = MetaClass.forClass(Customer.class, new DefaultReflectorFactory());
        System.out.println(metaClass.findProperty("order.name"));
        System.out.println(metaClass.findProperty("order.price"));
        System.out.println(metaClass.findProperty("order.age"));
        System.out.println(metaClass.getSetterType("order.name"));
        System.out.println(metaClass.getSetterType("order.price"));
    }
    class Customer {
        private Order order;
		// getter and setter
    }
    class Order {
        String name;
        Double price;
		// getter and setter
    }
}
    /**
     * order.name
     * order.price
     * order.
     * class java.lang.String
     * class java.lang.Double
     */
```

此类中唯一一个能解析集合的方法：`getGetterType()`。

```java
public Class<?> getGetterType(String name) {
    PropertyTokenizer prop = new PropertyTokenizer(name);
    if (prop.hasNext()) {
        // 由属性构建MetaClass。但是解析出来的不再是集合本身，而是集合里元素的类型
        MetaClass metaProp = metaClassForProperty(prop);
        return metaProp.getGetterType(prop.getChildren());
    }
    // issue #506. Resolve the type inside a Collection Object
    return getGetterType(prop);
}
private MetaClass metaClassForProperty(PropertyTokenizer prop) {
    Class<?> propType = getGetterType(prop);
    return MetaClass.forClass(propType, reflectorFactory);
}
private Class<?> getGetterType(PropertyTokenizer prop) {
    Class<?> type = reflector.getGetterType(prop.getName());
    // 存在index，即使index为""也算是存在
    if (prop.getIndex() != null && Collection.class.isAssignableFrom(type)) {
        // 获得集合本身的类型
        Type returnType = getGenericGetterType(prop.getName());
        if (returnType instanceof ParameterizedType) {
            // 获得集合参数的类型
            Type[] actualTypeArguments = ((ParameterizedType) returnType).getActualTypeArguments();
            if (actualTypeArguments != null && actualTypeArguments.length == 1) {
                returnType = actualTypeArguments[0];
                if (returnType instanceof Class) {
                    type = (Class<?>) returnType;
                } else if (returnType instanceof ParameterizedType) {
                    type = (Class<?>) ((ParameterizedType) returnType).getRawType();
                }
            }
        }
    }
    return type;
}
private Type getGenericGetterType(String propertyName) {
    try {
        Invoker invoker = reflector.getGetInvoker(propertyName);
        if (invoker instanceof MethodInvoker) {
            Field _method = MethodInvoker.class.getDeclaredField("method");
            _method.setAccessible(true);
            Method method = (Method) _method.get(invoker);
            return TypeParameterResolver.resolveReturnType(method, reflector.getType());
        } else if (invoker instanceof GetFieldInvoker) {
            Field _field = GetFieldInvoker.class.getDeclaredField("field");
            _field.setAccessible(true);
            Field field = (Field) _field.get(invoker);
            return TypeParameterResolver.resolveFieldType(field, reflector.getType());
        }
    } catch (NoSuchFieldException e) {
    } catch (IllegalAccessException e) {
    }
    return null;
}
```

看完了MetaClass就可以很容易明白 `settingsAsProperties()` 方法本身是加载并验证这些属性的。

```java
private Properties settingsAsProperties(XNode context) {
    if (context == null) {
        return new Properties();
    }
    Properties props = context.getChildrenAsProperties();
    // Check that all settings are known to the configuration class
    MetaClass metaConfig = MetaClass.forClass(Configuration.class, localReflectorFactory);
    for (Object key : props.keySet()) {
        if (!metaConfig.hasSetter(String.valueOf(key))) {
            throw new BuilderException("The setting " + key + 
                                       " is not known.  Make sure you spelled it correctly (case sensitive).");
        }
    }
    return props;
}
```

真正的解析settings的逻辑在settingsElement()方法里。

```java
private void settingsElement(Properties props) throws Exception {
    // 不使用自动映射，对于实际开发的帮助不大，因为一旦出现问题很难排查
    configuration.setAutoMappingBehavior(
        AutoMappingBehavior.valueOf(props.getProperty("autoMappingBehavior", "PARTIAL")));
    configuration.setAutoMappingUnknownColumnBehavior(
        AutoMappingUnknownColumnBehavior.valueOf(props.getProperty("autoMappingUnknownColumnBehavior", "NONE")));
    
    // cache 默认的开启的，关于cache会单独讲
    configuration.setCacheEnabled(booleanValueOf(props.getProperty("cacheEnabled"), true));
    
    // 动态代理的工厂类，有 CGLIB 和 JAVASSIST 两种，3.3版本以上默认是JAVASSIST。实际开发中无用
    configuration.setProxyFactory((ProxyFactory) createInstance(props.getProperty("proxyFactory")));
    
    // 延迟加载对象，实际开发中无用
    configuration.setLazyLoadingEnabled(booleanValueOf(props.getProperty("lazyLoadingEnabled"), false));
    
    // 延迟加载属性，实际开发中无用
    configuration.setAggressiveLazyLoading(booleanValueOf(props.getProperty("aggressiveLazyLoading"), false));
    
    // 允许单语句返回多结果集，需要数据库驱动的支持。我暂时还没遇到
    configuration.setMultipleResultSetsEnabled(booleanValueOf(props.getProperty("multipleResultSetsEnabled"), true));
    
    // 允许使用列标签代替列名，比如select NAME as SNAME from STUDENT; 中的SNAME是列标签，NAME是列名
    configuration.setUseColumnLabel(booleanValueOf(props.getProperty("useColumnLabel"), true));
    
    // 允许 JDBC 支持自动生成主键。如果设置为 true，将强制使用自动生成主键。所以一般不开启。
    configuration.setUseGeneratedKeys(booleanValueOf(props.getProperty("useGeneratedKeys"), false));
    
    // 设置默认的执行器，执行器有三种 SIMPLE REUSE BATCH。常用的是 SIMPLE和BATCH。后面会介绍如何使用 BATCH
    configuration.setDefaultExecutorType(ExecutorType.valueOf(props.getProperty("defaultExecutorType", "SIMPLE")));
    
    // 数据库驱动等待数据库响应的秒数
    configuration.setDefaultStatementTimeout(integerValueOf(props.getProperty("defaultStatementTimeout"), null));
    
    // 设置一次从数据库取多少条数据，默认不设置
    configuration.setDefaultFetchSize(integerValueOf(props.getProperty("defaultFetchSize"), null));
    
    // 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。开不开启取决于自己的项目规范。
    configuration.setMapUnderscoreToCamelCase(booleanValueOf(props.getProperty("mapUnderscoreToCamelCase"), false));
    configuration.setSafeRowBoundsEnabled(booleanValueOf(props.getProperty("safeRowBoundsEnabled"), false));
    configuration.setLocalCacheScope(LocalCacheScope.valueOf(props.getProperty("localCacheScope", "SESSION")));
    configuration.setJdbcTypeForNull(JdbcType.valueOf(props.getProperty("jdbcTypeForNull", "OTHER")));
    
    // 延迟加载的触发方法
    configuration.setLazyLoadTriggerMethods(
        stringSetValueOf(props.getProperty("lazyLoadTriggerMethods"), "equals,clone,hashCode,toString"));
    configuration.setSafeResultHandlerEnabled(booleanValueOf(props.getProperty("safeResultHandlerEnabled"), true));
    configuration.setDefaultScriptingLanguage(resolveClass(props.getProperty("defaultScriptingLanguage")));
    
    // 默认的枚举处理器
    @SuppressWarnings("unchecked")
    Class<? extends TypeHandler> typeHandler = 
        (Class<? extends TypeHandler>)resolveClass(props.getProperty("defaultEnumTypeHandler"));
    configuration.setDefaultEnumTypeHandler(typeHandler);
    configuration.setCallSettersOnNulls(booleanValueOf(props.getProperty("callSettersOnNulls"), false));
    configuration.setUseActualParamName(booleanValueOf(props.getProperty("useActualParamName"), true));
    configuration.setReturnInstanceForEmptyRow(booleanValueOf(props.getProperty("returnInstanceForEmptyRow"), false));
    configuration.setLogPrefix(props.getProperty("logPrefix"));
    @SuppressWarnings("unchecked")
    Class<? extends Log> logImpl = (Class<? extends Log>)resolveClass(props.getProperty("logImpl"));
    configuration.setLogImpl(logImpl);
    configuration.setConfigurationFactory(resolveClass(props.getProperty("configurationFactory")));
}
```

settings的解析非常复杂，但是绝大多数都使用默认设置就可以了，所以我们不花费过多的时间讨论。



### loadCustomVfs

虚拟文件系统的目的是屏蔽真实文件系统的差异。这里其实算不上文件系统，因为Mybatis里的VFS作用是从不同的位置加载文件，比如从jar包中，从不同的类加载器中。最经典的案例就是Mybatis默认的DefaultVFS在SpringBoot中运行时不能加载到 `<package>` 标签配置的别名。这个案例不会在解析Mybatis的系列文章中给出，因为它还涉及到很多SpringBoot的知识。我们还是继续看对VFS的加载过程。

```java
// XMLConfigBuilder.java
private void loadCustomVfs(Properties props) throws ClassNotFoundException {
    String value = props.getProperty("vfsImpl");
    if (value != null) {
        String[] clazzes = value.split(",");
        for (String clazz : clazzes) {
            if (!clazz.isEmpty()) {
                @SuppressWarnings("unchecked")
                Class<? extends VFS> vfsImpl = (Class<? extends VFS>)Resources.classForName(clazz);
                configuration.setVfsImpl(vfsImpl);
            }
        }
    }
}
```

```java
// Configuration.java
public void setVfsImpl(Class<? extends VFS> vfsImpl) {
    if (vfsImpl != null) {
        this.vfsImpl = vfsImpl;
        VFS.addImplClass(this.vfsImpl);
    }
}
```

```java
// VFS.java
public static final List<Class<? extends VFS>> USER_IMPLEMENTATIONS = 
    									new ArrayList<Class<? extends VFS>>();
public static void addImplClass(Class<? extends VFS> clazz) {
    if (clazz != null) {
        USER_IMPLEMENTATIONS.add(clazz);
    }
}
```

在VFS的配置可以指定多个VFS的实现，所有用户自定义的VFS实现都会被放在USER_IMPLEMENTATIONS这个集合里。加载过程仅仅是将类名转为Class对象保存起来。VFS实际上是一个单例，也就是说虽然我们可以传入多个实现，但是最终被使用的只有一个。

```java
// VFS.java
public static VFS getInstance() {
    return VFSHolder.INSTANCE;
}
public static final List<Class<? extends VFS>> USER_IMPLEMENTATIONS = 
    								new ArrayList<Class<? extends VFS>>();
public static final Class<?>[] IMPLEMENTATIONS = { JBoss6VFS.class, DefaultVFS.class };
private static class VFSHolder {
    static final VFS INSTANCE = createVFS();

    @SuppressWarnings("unchecked")
    static VFS createVFS() {
        // Try the user implementations first, then the built-ins
        List<Class<? extends VFS>> impls = new ArrayList<Class<? extends VFS>>();

        // 优先加载用户自定义的VFS实现。
        impls.addAll(USER_IMPLEMENTATIONS);
        // 内置的VFS实现有两个，一个是JBoos6，一个是DefaultVFS
        impls.addAll(Arrays.asList((Class<? extends VFS>[]) IMPLEMENTATIONS));

        // Try each implementation class until a valid one is found
        VFS vfs = null;
        for (int i = 0; vfs == null || !vfs.isValid(); i++) {
            Class<? extends VFS> impl = impls.get(i);
            try {
                vfs = impl.newInstance();
                if (vfs == null || !vfs.isValid()) {
                    if (log.isDebugEnabled()) {
                        log.debug("VFS implementation " + impl.getName() +
                                  " is not valid in this environment.");
                    }
                }
            } catch (InstantiationException e) {
                log.error("Failed to instantiate " + impl, e);
                return null;
            } catch (IllegalAccessException e) {
                log.error("Failed to instantiate " + impl, e);
                return null;
            }
        }
        if (log.isDebugEnabled()) {
            log.debug("Using VFS adapter " + vfs.getClass().getName());
        }
        return vfs;
    }
}
```

内置实现中优先级较高的是JBoss6，但是JBoss6需要依赖JBoss模块，所以在初始化的时候会对其进行验证，即系统中如果没有JBoss模块，JBoss6VFS就被设置为非法。所以在默认的情况下，Mybatis会使用DefaultVFS作为虚拟文件系统的实现。

```java
public class JBoss6VFS extends VFS {
    static {
        initialize();
    }
    protected static synchronized void initialize() {
        if (valid == null) {
            // Assume valid. It will get flipped later if something goes wrong.
            valid = Boolean.TRUE;

            // Look up and verify required classes
            VFS.VFS = checkNotNull(getClass("org.jboss.vfs.VFS"));
            VirtualFile.VirtualFile = checkNotNull(getClass("org.jboss.vfs.VirtualFile"));

            // Look up and verify required methods
            VFS.getChild = checkNotNull(getMethod(VFS.VFS, "getChild", URL.class));
            VirtualFile.getChildrenRecursively = checkNotNull(getMethod(VirtualFile.VirtualFile,
                                                                        "getChildrenRecursively"));
            VirtualFile.getPathNameRelativeTo = checkNotNull(getMethod(VirtualFile.VirtualFile,
                                                                       "getPathNameRelativeTo", VirtualFile.VirtualFile));

            // Verify that the API has not changed
            checkReturnType(VFS.getChild, VirtualFile.VirtualFile);
            checkReturnType(VirtualFile.getChildrenRecursively, List.class);
            checkReturnType(VirtualFile.getPathNameRelativeTo, String.class);
        }
    }
}
```

VFS本身是一个抽象类，提供了一些方法的实现，但是这些方法都是挺简单的方法，我们需要关注的是两个list方法。

<div align="center"><img width="40%" src="image/2020-09-08_185233.jpg" /></div>

```java
// VFS.java
protected abstract List<String> list(URL url, String forPath) throws IOException;

public List<String> list(String path) throws IOException {
    List<String> names = new ArrayList<String>();
    for (URL url : getResources(path)) {
        names.addAll(list(url, path));
    }
    return names;
}
protected static List<URL> getResources(String path) throws IOException {
    return Collections.list(Thread.currentThread().getContextClassLoader().getResources(path));
}
```

VFS里只会得到文件的URL对象，真正的解析被委托给了子类。`list(URL url, String forPath)` 方法在下一个标签的解析中就会被使用，所以本小节先跳过这一部分。下一小节结合具体案例再看。

### typeAliasesElement

```java
// XMLConfigBuilder.java
protected final Configuration configuration;
private void typeAliasesElement(XNode parent) {
    if (parent != null) {
        for (XNode child : parent.getChildren()) {
            if ("package".equals(child.getName())) {
                String typeAliasPackage = child.getStringAttribute("name");
                configuration.getTypeAliasRegistry().registerAliases(typeAliasPackage);
            } 
            // 单个加载的逻辑是非常简单的，我们就不在看了。
            else {
                String alias = child.getStringAttribute("alias");
                String type = child.getStringAttribute("type");
                try {
                    Class<?> clazz = Resources.classForName(type);
                    if (alias == null) {
                        typeAliasRegistry.registerAlias(clazz);
                    } else {
                        typeAliasRegistry.registerAlias(alias, clazz);
                    }
                } catch (ClassNotFoundException e) {
                    throw new BuilderException("Error registering typeAlias for '" 
                                               + alias + "'. Cause: " + e, e);
                }
            }
        }
    }
}
```

```java
// Configuration.java
public TypeAliasRegistry getTypeAliasRegistry() {
    return typeAliasRegistry;
}
```

```java
// TypeAliasRegistry.java
public void registerAliases(String packageName){
    registerAliases(packageName, Object.class);
}
// 从包名获得Class对象
public void registerAliases(String packageName, Class<?> superType){
    ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<Class<?>>();
    resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
    Set<Class<? extends Class<?>>> typeSet = resolverUtil.getClasses();
    for(Class<?> type : typeSet){
        // Ignore inner classes and interfaces (including package-info.java)
        // Skip also inner classes. See issue #6
        if (!type.isAnonymousClass() && !type.isInterface() && !type.isMemberClass()) {
            registerAlias(type);
        }
    }
}
public void registerAlias(Class<?> type) {
    String alias = type.getSimpleName();
    Alias aliasAnnotation = type.getAnnotation(Alias.class);
    if (aliasAnnotation != null) {
        alias = aliasAnnotation.value();
    } 
    registerAlias(alias, type);
}
public void registerAlias(String alias, Class<?> value) {
    if (alias == null) {
        throw new TypeException("The parameter alias cannot be null");
    }
    // issue #748
    String key = alias.toLowerCase(Locale.ENGLISH);
    if (TYPE_ALIASES.containsKey(key) && TYPE_ALIASES.get(key) != null && !TYPE_ALIASES.get(key).equals(value)) {
        throw new TypeException("The alias '" + alias + "' is already mapped to the value '" 
                                + TYPE_ALIASES.get(key).getName() + "'.");
    }
    TYPE_ALIASES.put(key, value);
}
```

TypeAliasRegistry里的这一段代码，最重要的是 `registerAliases()` 方法，解释它之前需要看两个Demo。



#### ResolverUtil

分解器的目的是为了将查找指定的类，这个指定的条件有两类：某个包下和某个注解标识。

```java
public class TestResolverUtil {
    public static void main(String[] args) {
        ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<>();
        ResolverUtil.IsA isA = new ResolverUtil.IsA(Object.class);
//        ResolverUtil.IsA annotatedWith = new ResolverUtil.AnnotatedWith(Annotated.class);
        resolverUtil.find(isA, "jishuneimu");
        Set<Class<? extends Class<?>>> classes = resolverUtil.getClasses();
        for (Class<?> c : classes) {
            System.out.println(c);
        }
    }
}
```

Test类是ResolverUtil的内部类，它的功能就是验证一个Class对象是不是符合要求。

```java
// ResolverUtil.java
public interface Test {
    /**
     * Will be called repeatedly with candidate classes. Must return True if a class
     * is to be included in the results, false otherwise.
     */
    boolean matches(Class<?> type);
}

public static class AnnotatedWith implements Test {
    private Class<? extends Annotation> annotation;
    /** Constructs an AnnotatedWith test for the specified annotation type. */
    public AnnotatedWith(Class<? extends Annotation> annotation) {
        this.annotation = annotation;
    }
    /** Returns true if the type is annotated with the class provided to the constructor. */
    @Override
    public boolean matches(Class<?> type) {
        return type != null && type.isAnnotationPresent(annotation);
    }
    @Override
    public String toString() {
        return "annotated with @" + annotation.getSimpleName();
    }
}

public static class IsA implements Test {
    private Class<?> parent;
    /** Constructs an IsA test using the supplied Class as the parent class/interface. */
    public IsA(Class<?> parentType) {
        this.parent = parentType;
    }
    /** Returns true if type is assignable to the parent type supplied in the constructor. */
    @Override
    public boolean matches(Class<?> type) {
        return type != null && parent.isAssignableFrom(type);
    }
    @Override
    public String toString() {
        return "is assignable to " + parent.getSimpleName();
    }
}
```

find() 方法就是查找方法，参数很简单，就是一个 `Test` 对象，一个包名。

```java
public ResolverUtil<T> find(Test test, String packageName) {
    // 将包名表示转为路径表示
    String path = getPackagePath(packageName);
    try {
        // 这里就是上一小节提到的虚拟文件系统
        List<String> children = VFS.getInstance().list(path);
        for (String child : children) {
            if (child.endsWith(".class")) {
                addIfMatching(test, child);
            }
        }
    } catch (IOException ioe) {
        log.error("Could not read package: " + packageName, ioe);
    }

    return this;
}
protected String getPackagePath(String packageName) {
    return packageName == null ? null : packageName.replace('.', '/');
}
protected void addIfMatching(Test test, String fqn) {
    try {
        // 把 .class 文件截取掉，再将路径名分隔符替换为.
        String externalName = fqn.substring(0, fqn.indexOf('.')).replace('/', '.');
        ClassLoader loader = getClassLoader();
        if (log.isDebugEnabled()) {
            log.debug("Checking to see if class " + externalName + " matches criteria [" + test + "]");
        }
        Class<?> type = loader.loadClass(externalName);
        if (test.matches(type)) {
            matches.add((Class<T>) type);
        }
    } catch (Throwable t) {
        log.warn("Could not examine class '" + fqn + "'" + " due to a " +
                 t.getClass().getName() + " with message: " + t.getMessage());
    }
}

// ClassLoader是可以设置的 public void setClassLoader(ClassLoader classloader)，如果没有设置
// 就使用线程上下文类加载器
public ClassLoader getClassLoader() {
    return classloader == null ? Thread.currentThread().getContextClassLoader() : classloader;
}
```

所以我们接下来的重点就是来看 `list(URL url, String path)` 方法。

```java
// VFS.java
public List<String> list(URL url, String path) throws IOException {
    InputStream is = null;
    try {
        List<String> resources = new ArrayList<String>();

        // First, try to find the URL of a JAR file containing the requested resource. If a JAR
        // file is found, then we'll list child resources by reading the JAR.
        // 这个地方是为了判断这个url是不是jar包，具体逻辑比较复杂，我们就不看了
        URL jarUrl = findJarForResource(url);
        if (jarUrl != null) {
            // 如果是jar包，就从jar包里读取
            is = jarUrl.openStream();
            if (log.isDebugEnabled()) {
                log.debug("Listing " + url);
            }
            // 这个的具体逻辑计较简单
            resources = listResources(new JarInputStream(is), path);
        }
        else {
            List<String> children = new ArrayList<String>();
            try {
                if (isJar(url)) {
                    // Some versions of JBoss VFS might give a JAR stream even if the resource
                    // referenced by the URL isn't actually a JAR
                    // 修改jboss的bug
                    is = url.openStream();
                    JarInputStream jarInput = new JarInputStream(is);
                    if (log.isDebugEnabled()) {
                        log.debug("Listing " + url);
                    }
                    for (JarEntry entry; (entry = jarInput.getNextJarEntry()) != null;) {
                        if (log.isDebugEnabled()) {
                            log.debug("Jar entry: " + entry.getName());
                        }
                        children.add(entry.getName());
                    }
                    jarInput.close();
                }
                else {
                    /*
             		 * Some servlet containers allow reading from directory resources like a
             		 * text file, listing the child resources one per line. However, there is no
             		 * way to differentiate between directory and file resources just by reading
             		 * them. To work around that, as each line is read, try to look it up via
             		 * the class loader as a child of the current resource. If any line fails
             		 * then we assume the current resource is not a directory.
             		*/
                    // 有可能这个url是一个文本文件，此时按行读取，假设每一行都是一个资源，如果有任意一行找不到
                    // 就返回空集合
                    is = url.openStream();
                    BufferedReader reader = new BufferedReader(new InputStreamReader(is));
                    List<String> lines = new ArrayList<String>();
                    for (String line; (line = reader.readLine()) != null;) {
                        if (log.isDebugEnabled()) {
                            log.debug("Reader entry: " + line);
                        }
                        lines.add(line);
                        if (getResources(path + "/" + line).isEmpty()) {
                            lines.clear();
                            break;
                        }
                    }
                    if (!lines.isEmpty()) {
                        if (log.isDebugEnabled()) {
                            log.debug("Listing " + url);
                        }
                        children.addAll(lines);
                    }
                }
            } catch (FileNotFoundException e) {
                /*
           		 * For file URLs the openStream() call might fail, depending on the servlet
           		 * container, because directories can't be opened for reading. If that happens,
           		 * then list the directory directly instead.
           		*/
                // 如果是一个文件夹，就将文件夹里的文件都加入
                if ("file".equals(url.getProtocol())) {
                    File file = new File(url.getFile());
                    if (log.isDebugEnabled()) {
                        log.debug("Listing directory " + file.getAbsolutePath());
                    }
                    if (file.isDirectory()) {
                        if (log.isDebugEnabled()) {
                            log.debug("Listing " + url);
                        }
                        children = Arrays.asList(file.list());
                    }
                }
                else {
                    // No idea where the exception came from so rethrow it
                    throw e;
                }
            }

            // The URL prefix to use when recursively listing child resources
            String prefix = url.toExternalForm();
            if (!prefix.endsWith("/")) {
                prefix = prefix + "/";
            }

            // Iterate over immediate children, adding files and recursing into directories
            // 递归调用，直至children为空集合
            for (String child : children) {
                String resourcePath = path + "/" + child;
                resources.add(resourcePath);
                URL childUrl = new URL(prefix + child);
                resources.addAll(list(childUrl, resourcePath));
            }
        }

        return resources;
    } finally {
        if (is != null) {
            try {
                is.close();
            } catch (Exception e) {
                // Ignore
            }
        }
    }
}

protected List<String> listResources(JarInputStream jar, String path) throws IOException {
    // Include the leading and trailing slash when matching names
    if (!path.startsWith("/")) {
        path = "/" + path;
    }
    if (!path.endsWith("/")) {
        path = path + "/";
    }

    // Iterate over the entries and collect those that begin with the requested path
    List<String> resources = new ArrayList<String>();
    for (JarEntry entry; (entry = jar.getNextJarEntry()) != null;) {
        if (!entry.isDirectory()) {
            // Add leading slash if it's missing
            String name = entry.getName();
            if (!name.startsWith("/")) {
                name = "/" + name;
            }

            // Check file name
            if (name.startsWith(path)) {
                if (log.isDebugEnabled()) {
                    log.debug("Found resource: " + name);
               }
                // Trim leading slash
                resources.add(name.substring(1));
            }
        }
    }
    return resources;
}
```

看完这一段代码，我们就知道下面这个注册别名全部的运行逻辑了。

```java
// TypeAliasRegistry.java
public void registerAliases(String packageName){
    registerAliases(packageName, Object.class);
}
// 从包名获得Class对象
public void registerAliases(String packageName, Class<?> superType){
    ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<Class<?>>();
    resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
    Set<Class<? extends Class<?>>> typeSet = resolverUtil.getClasses();
    for(Class<?> type : typeSet){
        // Ignore inner classes and interfaces (including package-info.java)
        // Skip also inner classes. See issue #6
        if (!type.isAnonymousClass() && !type.isInterface() && !type.isMemberClass()) {
            registerAlias(type);
        }
    }
}
public void registerAlias(Class<?> type) {
    String alias = type.getSimpleName();
    Alias aliasAnnotation = type.getAnnotation(Alias.class);
    if (aliasAnnotation != null) {
        alias = aliasAnnotation.value();
    } 
    registerAlias(alias, type);
}
public void registerAlias(String alias, Class<?> value) {
    if (alias == null) {
        throw new TypeException("The parameter alias cannot be null");
    }
    // issue #748
    String key = alias.toLowerCase(Locale.ENGLISH);
    if (TYPE_ALIASES.containsKey(key) && TYPE_ALIASES.get(key) != null && !TYPE_ALIASES.get(key).equals(value)) {
        throw new TypeException("The alias '" + alias + "' is already mapped to the value '" 
                                + TYPE_ALIASES.get(key).getName() + "'.");
    }
    TYPE_ALIASES.put(key, value);
}
```



### pluginElement

```java
// XMLConfigBuilder.java
private void pluginElement(XNode parent) throws Exception {
    if (parent != null) {
        for (XNode child : parent.getChildren()) {
            String interceptor = child.getStringAttribute("interceptor");
            Properties properties = child.getChildrenAsProperties();
            Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).newInstance();
            interceptorInstance.setProperties(properties);
            configuration.addInterceptor(interceptorInstance);
        }
    }
}
```

```java
// BaseBuilder.java
protected Class<?> resolveClass(String alias) {
    if (alias == null) {
        return null;
    }
    try {
        return resolveAlias(alias);
    } catch (Exception e) {
        throw new BuilderException("Error resolving class. Cause: " + e, e);
    }
}
protected Class<?> resolveAlias(String alias) {
    return typeAliasRegistry.resolveAlias(alias);
}
```

```java
// TypeAliasRegistry.java
public <T> Class<T> resolveAlias(String string) {
    try {
        if (string == null) {
            return null;
        }
        // issue #748
        String key = string.toLowerCase(Locale.ENGLISH);
        Class<T> value;
        if (TYPE_ALIASES.containsKey(key)) {
            value = (Class<T>) TYPE_ALIASES.get(key);
        } else {
            value = (Class<T>) Resources.classForName(string);
        }
        return value;
    } catch (ClassNotFoundException e) {
        throw new TypeException("Could not resolve type alias '" + string + "'.  Cause: " + e, e);
    }
}
```

```java
// Configuration.java
protected final InterceptorChain interceptorChain = new InterceptorChain();
public void addInterceptor(Interceptor interceptor) {
    interceptorChain.addInterceptor(interceptor);
}
```

```java
// InterceptorChain.java
public class InterceptorChain {
    // 由于这里是一个数组，所以插件的执行顺序就是配置的顺序
    private final List<Interceptor> interceptors = new ArrayList<Interceptor>();
    public Object pluginAll(Object target) {
        for (Interceptor interceptor : interceptors) {
            target = interceptor.plugin(target);
        }
        return target;
    }
    public void addInterceptor(Interceptor interceptor) {
        interceptors.add(interceptor);
    }
    public List<Interceptor> getInterceptors() {
        return Collections.unmodifiableList(interceptors);
    }
}
```

插件的加载很简单，不再多说，关于插件的执行放在后面单独说。