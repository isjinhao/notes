## 解析Mapper

```java
// XMLConfigBuilder.java
private void mapperElement(XNode parent) throws Exception {
    if (parent != null) {
        for (XNode child : parent.getChildren()) {
            if ("package".equals(child.getName())) {
                String mapperPackage = child.getStringAttribute("name");
                configuration.addMappers(mapperPackage);
            } else {
                String resource = child.getStringAttribute("resource");
                String url = child.getStringAttribute("url");
                String mapperClass = child.getStringAttribute("class");
                if (resource != null && url == null && mapperClass == null) {
                    ErrorContext.instance().resource(resource);
                    InputStream inputStream = Resources.getResourceAsStream(resource);
                    XMLMapperBuilder mapperParser = 
                        new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
                    mapperParser.parse();
                } else if (resource == null && url != null && mapperClass == null) {
                    ErrorContext.instance().resource(url);
                    InputStream inputStream = Resources.getUrlAsStream(url);
                    XMLMapperBuilder mapperParser = 
                        new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
                    mapperParser.parse();
                } else if (resource == null && url == null && mapperClass != null) {
                    Class<?> mapperInterface = Resources.classForName(mapperClass);
                    configuration.addMapper(mapperInterface);
                } else {
                    throw new BuilderException("A mapper element may only specify a url, " 
                                               + "resource or class, but not more than one.");
                }
            }
        }
    }
}
```

在Mapper文件中，最重要的就是resultMap和select|update|insert|delete标签，所以后面的内容主要分析这两部分是如何解析的。由于我们最常用的方式是配置包，所以就从这里开始进行debug。

```java
// Configuration.java
public void addMappers(String packageName) {
    mapperRegistry.addMappers(packageName);
}
```

```java
// MapperRegistry.java
public void addMappers(String packageName) {
    addMappers(packageName, Object.class);
}
// 搜索所有的类，可能某些类并不是我们的Mapper类
public void addMappers(String packageName, Class<?> superType) {
    ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<>();
    resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
    Set<Class<? extends Class<?>>> mapperSet = resolverUtil.getClasses();
    for (Class<?> mapperClass : mapperSet) {
        addMapper(mapperClass);
    }
}
private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<>();
public <T> void addMapper(Class<T> type) {
    // Mapper只能是接口
    if (type.isInterface()) {
        // 每个Mapper只能被绑定一次
        if (hasMapper(type)) {
            throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
        }
        boolean loadCompleted = false;
        try {
            knownMappers.put(type, new MapperProxyFactory<>(type));
            MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
            // 解析流程
            parser.parse();
            loadCompleted = true;
        } finally {
            if (!loadCompleted) {
                knownMappers.remove(type);
            }
        }
    }
}
```

```java
// MapperAnnotationBuilder.java
public void parse() {
    String resource = type.toString();
    if (!configuration.isResourceLoaded(resource)) {
        // 这里会加载对应的XML文件
        loadXmlResource();
        // 加载过的文件放在Configuration对象中
        configuration.addLoadedResource(resource);
        assistant.setCurrentNamespace(type.getName());
        parseCache();
        parseCacheRef();
        for (Method method : type.getMethods()) {
            if (!canHaveStatement(method)) {
                continue;
            }
            if (getAnnotationWrapper(method, false, Select.class, SelectProvider.class).isPresent()
                && method.getAnnotation(ResultMap.class) == null) {
                parseResultMap(method);
            }
            try {
                parseStatement(method);
            } catch (IncompleteElementException e) {
                configuration.addIncompleteMethod(new MethodResolver(this, method));
            }
        }
    }
    parsePendingMethods();
}
private void loadXmlResource() {
    // Spring may not know the real resource name so we check a flag
    // to prevent loading again a resource twice
    // this flag is set at XMLMapperBuilder#bindMapperForNamespace
    // ？
    if (!configuration.isResourceLoaded("namespace:" + type.getName())) {
        String xmlResource = type.getName().replace('.', '/') + ".xml";
        // #1347
        InputStream inputStream = type.getResourceAsStream("/" + xmlResource);
        if (inputStream == null) {
            // Search XML mapper that is not in the module but in the classpath.
            try {
                inputStream = Resources.getResourceAsStream(type.getClassLoader(), xmlResource);
            } catch (IOException e2) {
                // ignore, resource is not required
            }
        }
        if (inputStream != null) {
            XMLMapperBuilder xmlParser = new XMLMapperBuilder(inputStream, 
            					assistant.getConfiguration(), xmlResource, configuration.getSqlFragments(), type.getName());
            // 解析mybatis-config.xml文件使用的是XMLConfigBuilder类，解析Mapper文件使用的是XMLMapperBuilder类。
            xmlParser.parse();
        }
    }
}
```

解析xml文件

```java
// XMLMapperBuilder.java
public void parse() {
    if (!configuration.isResourceLoaded(resource)) {
        configurationElement(parser.evalNode("/mapper"));
        configuration.addLoadedResource(resource);
        bindMapperForNamespace();
    }

    parsePendingResultMaps();
    parsePendingCacheRefs();
    parsePendingStatements();
}
private void configurationElement(XNode context) {
    try {
        String namespace = context.getStringAttribute("namespace");
        if (namespace == null || namespace.isEmpty()) {
            throw new BuilderException("Mapper's namespace cannot be empty");
        }
        builderAssistant.setCurrentNamespace(namespace);
        cacheRefElement(context.evalNode("cache-ref"));
        cacheElement(context.evalNode("cache"));
        // 这个标签已经被废弃了，所以不再考虑这个标签
        parameterMapElement(context.evalNodes("/mapper/parameterMap"));
        resultMapElements(context.evalNodes("/mapper/resultMap"));
        sqlElement(context.evalNodes("/mapper/sql"));
        buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
    } catch (Exception e) {
        throw new BuilderException("Error parsing Mapper XML. The XML location is '" + resource + "'. Cause: " + e, e);
    }
}
```



### cacheRefElement

```java
// XMLMapperBuilder.java
protected final Configuration configuration;
private void cacheRefElement(XNode context) {
    if (context != null) {
        // 在configuration里添加记录
        configuration.addCacheRef(builderAssistant.getCurrentNamespace(), context.getStringAttribute("namespace"));
        CacheRefResolver cacheRefResolver = new CacheRefResolver(builderAssistant, context.getStringAttribute("namespace"));
        try {
            // 解析当前的CacheRefResolver
            cacheRefResolver.resolveCacheRef();
        } catch (IncompleteElementException e) {
            configuration.addIncompleteCacheRef(cacheRefResolver);
        }
    }
}
```

```java
// Configuration.java
protected final Map<String, String> cacheRefMap = new HashMap<>();
public void addCacheRef(String namespace, String referencedNamespace) {
    // <当前的 namespace, 引用的namespace>
    cacheRefMap.put(namespace, referencedNamespace);
}
```

```java
// CacheRefResolver.java
private final MapperBuilderAssistant assistant;
public CacheRefResolver(MapperBuilderAssistant assistant, String cacheRefNamespace) {
    this.assistant = assistant;
    this.cacheRefNamespace = cacheRefNamespace;
}
public Cache resolveCacheRef() {
    return assistant.useCacheRef(cacheRefNamespace);
}
```

```java
// MapperBuilderAssistant.java
private Cache currentCache;
private boolean unresolvedCacheRef;
public Cache useCacheRef(String namespace) {
    if (namespace == null) {
        throw new BuilderException("cache-ref element requires a namespace attribute.");
    }
    try {
        unresolvedCacheRef = true;
        // 如果未加载到这个namespace的cache，先不处理
        Cache cache = configuration.getCache(namespace);
        if (cache == null) {
            throw new IncompleteElementException("No cache for namespace '" + namespace + "' could be found.");
        }
        // 这个currentCache就是后面用于构建MappedStatement的cache
        currentCache = cache;
        unresolvedCacheRef = false;
        return cache;
    } catch (IllegalArgumentException e) {
        throw new IncompleteElementException("No cache for namespace '" + namespace + "' could be found.", e);
    }
}
```

```java
// MapperBuilderAssistant.java
public MappedStatement addMappedStatement(String id, SqlSource sqlSource, StatementType statementType,
    SqlCommandType sqlCommandType, Integer fetchSize, Integer timeout, String parameterMap, Class<?> parameterType,
    String resultMap, Class<?> resultType, ResultSetType resultSetType, boolean flushCache, boolean useCache,
    boolean resultOrdered, KeyGenerator keyGenerator, String keyProperty, String keyColumn, String databaseId,
    LanguageDriver lang, String resultSets) {
    if (unresolvedCacheRef) {
        throw new IncompleteElementException("Cache-ref not yet resolved");
    }
    
    id = applyCurrentNamespace(id, false);
    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
    MappedStatement.Builder statementBuilder = new MappedStatement.Builder(configuration, id, sqlSource, sqlCommandType)
        .resource(resource).fetchSize(fetchSize).timeout(timeout).statementType(statementType).keyGenerator(keyGenerator)
        .keyProperty(keyProperty).keyColumn(keyColumn).databaseId(databaseId).lang(lang).resultOrdered(resultOrdered)
        .resultSets(resultSets).resultMaps(getStatementResultMaps(resultMap, resultType, id)).resultSetType(resultSetType)
        .flushCacheRequired(valueOrDefault(flushCache, !isSelect)).useCache(valueOrDefault(useCache, isSelect))
        .cache(currentCache);
    // 上面就是在使用currentCache构建MappedStatement
    ParameterMap statementParameterMap = getStatementParameterMap(parameterMap, parameterType, id);
    if (statementParameterMap != null) {
        statementBuilder.parameterMap(statementParameterMap);
    }
    MappedStatement statement = statementBuilder.build();
    configuration.addMappedStatement(statement);
    return statement;
}
```



### cacheElement

```java
private void cacheElement(XNode context) {
    if (context != null) {
        String type = context.getStringAttribute("type", "PERPETUAL");
        Class<? extends Cache> typeClass = typeAliasRegistry.resolveAlias(type);
        String eviction = context.getStringAttribute("eviction", "LRU");
        Class<? extends Cache> evictionClass = typeAliasRegistry.resolveAlias(eviction);
        Long flushInterval = context.getLongAttribute("flushInterval");
        Integer size = context.getIntAttribute("size");
        boolean readWrite = !context.getBooleanAttribute("readOnly", false);
        boolean blocking = context.getBooleanAttribute("blocking", false);
        Properties props = context.getChildrenAsProperties();
        builderAssistant.useNewCache(typeClass, evictionClass, flushInterval, size, readWrite, blocking, props);
    }
}
```

```java
// MapperBuilderAssistant.java
public Cache useNewCache(Class<? extends Cache> typeClass, Class<? extends Cache> evictionClass, Long flushInterval,
		Integer size, boolean readWrite, boolean blocking, Properties props) {
    Cache cache = new CacheBuilder(currentNamespace)
        .implementation(valueOrDefault(typeClass, PerpetualCache.class))
        .addDecorator(valueOrDefault(evictionClass, LruCache.class)).clearInterval(flushInterval)
        .size(size).readWrite(readWrite).blocking(blocking).properties(props).build();
    // 
    configuration.addCache(cache);
    currentCache = cache;
    return cache;
}
```

```java
// Configuration.java
protected final Map<String, Cache> caches = new StrictMap<>("Caches collection");
public void addCache(Cache cache) {
    caches.put(cache.getId(), cache);
}
```

#### StrictMap

```java
protected static class StrictMap<V> extends HashMap<String, V> {
    private static final long serialVersionUID = -4950446264854982944L;
    private final String name;
    private BiFunction<V, V, String> conflictMessageProducer;

    public StrictMap(String name, int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        this.name = name;
    }
    public StrictMap(String name, int initialCapacity) {
        super(initialCapacity);
        this.name = name;
    }
    public StrictMap(String name) {
        super();
        this.name = name;
    }
    public StrictMap(String name, Map<String, ? extends V> m) {
        super(m);
        this.name = name;
    }

    /**
     * Assign a function for producing a conflict error message when contains value with the same key.
     * <p>
     * function arguments are 1st is saved value and 2nd is target value.
     * @param conflictMessageProducer A function for producing a conflict error message
     * @return a conflict error message
     * @since 3.5.0
     */
    public StrictMap<V> conflictMessageProducer(BiFunction<V, V, String> conflictMessageProducer) {
        this.conflictMessageProducer = conflictMessageProducer;
        return this;
    }

    @Override
    @SuppressWarnings("unchecked")
    public V put(String key, V value) {
        // 重复会报错
        if (containsKey(key)) {
            throw new IllegalArgumentException(name + " already contains value for " + key + 
            	(conflictMessageProducer == null ? "" : conflictMessageProducer.apply(super.get(key), value)));
        }
        if (key.contains(".")) {
            final String shortKey = getShortName(key);
            if (super.get(shortKey) == null) {
                // <短名，值>
                super.put(shortKey, value);
            } else {
                super.put(shortKey, (V) new Ambiguity(shortKey));
            }
        }
        // <全名，值>
        return super.put(key, value);
    }

    @Override
    public V get(Object key) {
        V value = super.get(key);
        if (value == null) {
            throw new IllegalArgumentException(name + " does not contain value for " + key);
        }
        if (value instanceof Ambiguity) {
            throw new IllegalArgumentException(((Ambiguity) value).getSubject() + " is ambiguous in " + name
            	+ " (try using the full name including the namespace, or rename one of the entries)");
        }
        return value;
    }

    protected static class Ambiguity {
        private final String subject;
        public Ambiguity(String subject) {
            this.subject = subject;
        }
        public String getSubject() {
            return subject;
        }
    }

    private String getShortName(String key) {
        final String[] keyParts = key.split("\\.");
        return keyParts[keyParts.length - 1];
    }
}
```



### sqlElement

```java
private void sqlElement(List<XNode> list) {
    if (configuration.getDatabaseId() != null) {
        sqlElement(list, configuration.getDatabaseId());
    }
    sqlElement(list, null);
}
private void sqlElement(List<XNode> list, String requiredDatabaseId) {
    for (XNode context : list) {
        String databaseId = context.getStringAttribute("databaseId");
        String id = context.getStringAttribute("id");
        id = builderAssistant.applyCurrentNamespace(id, false);
        if (databaseIdMatchesCurrent(id, databaseId, requiredDatabaseId)) {
            // 这个地方会放入一个全名和一个短名，比如testA命名空间下面有一个叫aaa的sql这里会放入
            // <testA.aaa, context> 和 <aaa, context>
            sqlFragments.put(id, context);
        }
    }
}
private boolean databaseIdMatchesCurrent(String id, String databaseId, String requiredDatabaseId) {
    if (requiredDatabaseId != null) {
        return requiredDatabaseId.equals(databaseId);
    }
    if (databaseId != null) {
        return false;
    }
    if (!this.sqlFragments.containsKey(id)) {
        return true;
    }
    // skip this fragment if there is a previous one with a not null databaseId
    XNode context = this.sqlFragments.get(id);
    return context.getStringAttribute("databaseId") == null;
}
```



### resultMapElements

下面以一个案例debug对这个标签的解释。这个案例包含了一对一查询和一对多查询。

```java
public class Student {
    private String sno;
    private String sname;
    private String ssex;
    private Date sbirthday;
    private String clazz;
}

public class Score {
    private String sno;
    private String cno;
    private String degree;
}

public class House {
    private String houseId;
    private String houseHolder;
    private String houseMember;
}
```

HouseMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="demo.HouseMapper"  >
  <resultMap id="basicResult" type="pojo.House">
    <result property="houseMember" column="SNO" jdbcType="VARCHAR" />
  </resultMap>
</mapper>
```

ScoreMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="demo.ScoreMapper"  >
  <resultMap id="basicResult" type="pojo.Score" />
</mapper>
```

StudentMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="demo.StudentMapper" >
  <cache type="PERPETUAL" eviction="LRU" flushInterval="120000" readOnly="true" size="1024"></cache>
  <resultMap id="fullResult" type="pojo.FullStudent" extends="demo.StudentMapper.basicResult">
    <association resultMap="demo.HouseMapper.basicResult" property="house" />
    <collection property="scoreList" ofType="pojo.Score" resultMap="demo.ScoreMapper.basicResult" />
  </resultMap>
  <select id="selectFullStudent" resultMap="fullResult" databaseId="mysql">
    select
      STUDENT.SNO SNO,
      STUDENT.SNAME SNAME,
      STUDENT.SSEX SSEX,
      STUDENT.SBIRTHDAY SBIRTHDAY,
      STUDENT.CLASS CLASS,
      SCORE.CNO CNO,
      SCORE.DEGREE DEGREE,
      HOUSE.HOUSE_ID HOUSE_ID,
      HOUSE.HOUSE_HOLDER HOUSE_HOLDER,
      HOUSE.HOUSE_MEMBER HOUSE_MEMBER
    from STUDENT, SCORE, HOUSE where STUDENT.SNO = SCORE.SNO and HOUSE.HOUSE_MEMBER = STUDENT.SNO;
  </select>
</mapper>
```





```java
private void resultMapElements(List<XNode> list) {
    for (XNode resultMapNode : list) {
        try {
            resultMapElement(resultMapNode);
        } catch (IncompleteElementException e) {
            // ignore, it will be retried
        }
    }
}S
private ResultMap resultMapElement(XNode resultMapNode) {
    return resultMapElement(resultMapNode, Collections.emptyList(), null);
}
private ResultMap resultMapElement
    	(XNode resultMapNode, List<ResultMapping> additionalResultMappings, Class<?> enclosingType) {
    ErrorContext.instance().activity("processing " + resultMapNode.getValueBasedIdentifier());
    String type = resultMapNode.getStringAttribute("type",
                  	  resultMapNode.getStringAttribute("ofType", 
                  		  resultMapNode.getStringAttribute("resultType",
                  			  resultMapNode.getStringAttribute("javaType"))));
    Class<?> typeClass = resolveClass(type);
    if (typeClass == null) {
        typeClass = inheritEnclosingType(resultMapNode, enclosingType);
    }
    Discriminator discriminator = null;
    List<ResultMapping> resultMappings = new ArrayList<>(additionalResultMappings);
    List<XNode> resultChildren = resultMapNode.getChildren();
    for (XNode resultChild : resultChildren) {
        if ("constructor".equals(resultChild.getName())) {
            processConstructorElement(resultChild, typeClass, resultMappings);
        } else if ("discriminator".equals(resultChild.getName())) {
            discriminator = processDiscriminatorElement(resultChild, typeClass, resultMappings);
        } else {
            List<ResultFlag> flags = new ArrayList<>();
            if ("id".equals(resultChild.getName())) {
                flags.add(ResultFlag.ID);
            }
            resultMappings.add(buildResultMappingFromContext(resultChild, typeClass, flags));
        }
    }
    String id = resultMapNode.getStringAttribute("id", resultMapNode.getValueBasedIdentifier());
    String extend = resultMapNode.getStringAttribute("extends");
    Boolean autoMapping = resultMapNode.getBooleanAttribute("autoMapping");
    ResultMapResolver resultMapResolver = 
    	new ResultMapResolver(builderAssistant, id, typeClass, extend, discriminator, resultMappings, autoMapping);
    try {
        return resultMapResolver.resolve();
    } catch (IncompleteElementException e) {
        configuration.addIncompleteResultMap(resultMapResolver);
        throw e;
    }
}
```





























