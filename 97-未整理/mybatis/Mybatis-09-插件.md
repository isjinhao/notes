## 插件

Mybatis支持拦截四个类：

- 执行器Executor（update、query、commit、rollback等方法）；
- 参数处理器ParameterHandler（getParameterObject、setParameters方法）；
- 结果集处理器ResultSetHandler（handleResultSets、handleOutputParameters等方法）；
- SQL语法构建器StatementHandler（prepare、parameterize、batch、update、query等方法）；

代码如下：

```java
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
        executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
        executor = new ReuseExecutor(this, transaction);
    } else {
        executor = new SimpleExecutor(this, transaction);
    }
    if (cacheEnabled) {
        executor = new CachingExecutor(executor);
    }
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
}
```

```java
public ParameterHandler newParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql) {
    ParameterHandler parameterHandler = 
        mappedStatement.getLang().createParameterHandler(mappedStatement, parameterObject, boundSql);
    parameterHandler = (ParameterHandler) interceptorChain.pluginAll(parameterHandler);
    return parameterHandler;
}
```

```java
public ResultSetHandler newResultSetHandler(Executor executor, MappedStatement mappedStatement, RowBounds rowBounds, 
    	ParameterHandler parameterHandler, ResultHandler resultHandler, BoundSql boundSql) {
    ResultSetHandler resultSetHandler = 
        new DefaultResultSetHandler(executor, mappedStatement, parameterHandler, resultHandler, boundSql, rowBounds);
    resultSetHandler = (ResultSetHandler) interceptorChain.pluginAll(resultSetHandler);
    return resultSetHandler;
}
```

```java
public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, 
    	RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    StatementHandler statementHandler = 
        new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
    return statementHandler;
}
```

### Demo分析

写一个简单的案例来分析一下作用。

```java
@Intercepts({@Signature(type = Executor.class, method = "query", 
                        args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class})})
public class ExecutorPlugin implements Interceptor {
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        System.out.println(invocation.getTarget().getClass());
        System.out.println(invocation.getMethod());
        System.out.println(Arrays.deepToString(invocation.getArgs()));
        return invocation.proceed();
    }
}
```

**配置插件**

```xml
<plugins>
    <plugin interceptor="demo.plugin.ExecutorPlugin"></plugin>
</plugins>
```

**生成动态代理**

Configuration类里面有一个InterceptorChain的对象。生成动态代理就是调用它的pluginAll方法。

```java
public class InterceptorChain {
    private final List<Interceptor> interceptors = new ArrayList<>();
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

所有的拦截器都得继承接口Interceptor。

```java
public interface Interceptor {
    Object intercept(Invocation invocation) throws Throwable;
    default Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }
    default void setProperties(Properties properties) {
        // NOP
    }
}
```

Plugin实现了JDK的InvocationHandler，所以可以作为参数传入到Proxy.newProxyInstance()里用于执行方法。

```java
public class Plugin implements InvocationHandler {

    private final Object target;
    private final Interceptor interceptor;
    private final Map<Class<?>, Set<Method>> signatureMap;

    private Plugin(Object target, Interceptor interceptor, Map<Class<?>, Set<Method>> signatureMap) {
        this.target = target;
        this.interceptor = interceptor;
        this.signatureMap = signatureMap;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
            Set<Method> methods = signatureMap.get(method.getDeclaringClass());
            if (methods != null && methods.contains(method)) {
                return interceptor.intercept(new Invocation(target, method, args));
            }
            return method.invoke(target, args);
        } catch (Exception e) {
            throw ExceptionUtil.unwrapThrowable(e);
        }
    }
    
    public static Object wrap(Object target, Interceptor interceptor) {
        // Intercepts 注解里支持多个 Signature。
        // 每个 Signature 可以配置一个被拦截的方法，最终生成一个 Map<Class, Set<Method>>
        Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
        Class<?> type = target.getClass();
        // 获得接口，接口是当前对象的接口同时需要存在于signatureMap的keySet中
        Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
        if (interfaces.length > 0) {
            return Proxy.newProxyInstance(
                type.getClassLoader(),
                interfaces,
                new Plugin(target, interceptor, signatureMap));
        }
        return target;
    }

    private static Map<Class<?>, Set<Method>> getSignatureMap(Interceptor interceptor) {
        Intercepts interceptsAnnotation = interceptor.getClass().getAnnotation(Intercepts.class);
        // issue #251
        if (interceptsAnnotation == null) {
            throw new PluginException("No @Intercepts annotation was found in interceptor " + 
                                      interceptor.getClass().getName());
        }
        Signature[] sigs = interceptsAnnotation.value();
        Map<Class<?>, Set<Method>> signatureMap = new HashMap<>();
        for (Signature sig : sigs) {
            Set<Method> methods = signatureMap.computeIfAbsent(sig.type(), k -> new HashSet<>());
            try {
                Method method = sig.type().getMethod(sig.method(), sig.args());
                methods.add(method);
            } catch (NoSuchMethodException e) {
                throw new PluginException("Could not find method on " + 
                                          sig.type() + " named " + sig.method() + ". Cause: " + e, e);
            }
        }
        return signatureMap;
    }

    private static Class<?>[] getAllInterfaces(Class<?> type, Map<Class<?>, Set<Method>> signatureMap) {
        Set<Class<?>> interfaces = new HashSet<>();
        while (type != null) {
            for (Class<?> c : type.getInterfaces()) {
                if (signatureMap.containsKey(c)) {
                    interfaces.add(c);
                }
            }
            type = type.getSuperclass();
        }
        return interfaces.toArray(new Class<?>[interfaces.size()]);
    }
}
```



## PageHelper

```java
PageHelper.startPage(1, 1);
List<FullStudent> fullStudents = studentMapper.selectFullStudent(null);
```



```java
protected static boolean DEFAULT_COUNT = true;
public static <E> Page<E> startPage(int pageNum, int pageSize) {
    return startPage(pageNum, pageSize, DEFAULT_COUNT);
}
public static <E> Page<E> startPage(int pageNum, int pageSize, boolean count) {
    return startPage(pageNum, pageSize, count, null, null);
}
public static <E> Page<E> startPage(int pageNum, int pageSize, boolean count, Boolean reasonable, Boolean pageSizeZero) {
    Page<E> page = new Page<E>(pageNum, pageSize, count);
    page.setReasonable(reasonable);
    page.setPageSizeZero(pageSizeZero);
    Page<E> oldPage = getLocalPage();
    if (oldPage != null && oldPage.isOrderByOnly()) {
        page.setOrderBy(oldPage.getOrderBy());
    }
    setLocalPage(page);
    return page;
}
```



## 拦截selectKey

一个拦截标签selectKey执行，将Oracle序列转为Redis逐渐的方式。

```java
@Intercepts({@Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class})})
public class SelectKeyPlugin implements Interceptor {

    private String prefix;

    private static final String sequenceQueryReg = "^SELECT(\\s+)(\\w+).NEXTVAL(\\s+)FROM(\\s+)DUAL$";

    @Override
    public Object intercept(Invocation invocation) throws Throwable {

        Object[] args = invocation.getArgs();
        MappedStatement mappedStatement = (MappedStatement) args[0];
        RowBounds rowBounds = (RowBounds) args[2];
        ResultHandler resultHandler = (ResultHandler) args[3];

        // 1、SelectKeyGenerator.processGeneratedKeys 在执行 Executor.query 方法时传入的后两个参数是 RowBounds.DEFAULT 和 Executor.NO_RESULT_HANDLER；
        // 2、MapperAnnotationBuilder.handleSelectKeyAnnotation 和 XMLStatementBuilder.parseSelectKeyNodes 方法在处理 selectKey 注解和 selectKey 标签
        //    时都会把 StatementHandler 后面加上后缀 SelectKeyGenerator.SELECT_KEY_SUFFIX；
        // 所以这三个条件作为一个过滤条件，如果不满足这三个条件，则不认为是 selectKey 的执行语句。
        if (rowBounds != RowBounds.DEFAULT || resultHandler != Executor.NO_RESULT_HANDLER || !mappedStatement.getId().endsWith(SelectKeyGenerator.SELECT_KEY_SUFFIX)) {
            return invocation.proceed();
        }

        Object parameter = args[1];
        // 获取待执行的获取序列ID的sql语句
        BoundSql boundSql = mappedStatement.getBoundSql(parameter);
        String sql = boundSql.getSql();

        // 老库的表依然走老库的序列
        if (sql.contains("@com.sf.sfa.common.constant.TableNameConst@OLD_FOC_DUAL")) {
            invocation.proceed();
        }

        System.out.println("SelectKeyPlugin 拦截到方法 " + mappedStatement.getId().replaceAll(SelectKeyGenerator.SELECT_KEY_SUFFIX, "") + " 的selectKey执行！");

        // 提取序列名
        String sequenceName = extractSequenceName(sql);
        if (sequenceName == null) {
            return invocation.proceed();
        }

        // SelectKeyGenerator.processGeneratedKeys 方法中调用 Executor.query 方法时用 List<Object> 接收
        List<Object> objects = new ArrayList<>();
        //        objects.add(BizCodeGenerator.generateSequence(sequenceName));
        return objects;
    }

    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {
        this.prefix = properties.getProperty("prefix");
    }

    private static String extractSequenceName(String sql) {
        sql = sql.toUpperCase();
        Pattern compile = Pattern.compile(sequenceQueryReg);
        Matcher matcher = compile.matcher(sql);
        List<String> sequenceNameList = new ArrayList<>();
        // 遍历SQL，有且仅有一个序列名的时候，返回序列名，其他情况返回空
        while (matcher.find()) {
            sequenceNameList.add(matcher.group(2));
        }
        if (sequenceNameList != null && sequenceNameList.size() == 1) {
            return sequenceNameList.get(0);
        }
        return null;
    }
}
```







