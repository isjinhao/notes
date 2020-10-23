## INSERT | UPDATE | DELETE 的执行

插入的xml代码。

```xml
<insert id="insert" useGeneratedKeys="true" keyProperty="id" keyColumn="ID" databaseId="mysql">
    <!--  select * from STUDENT  -->
    insert into STUDENT(SNO, SNAME, SSEX, SBIRTHDAY, CLASS) values (#{sno}, #{sname}, #{ssex}, #{sbirthday}, #{clazz})
</insert>
```

动态代理的拦截方法。

```java
// MapperProxy.java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
        if (Object.class.equals(method.getDeclaringClass())) {
            return method.invoke(this, args);
        } else {
            return cachedInvoker(method).invoke(proxy, method, args, sqlSession);
        }
    } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
    }
}
private final Map<Method, MapperMethodInvoker> methodCache;
private MapperMethodInvoker cachedInvoker(Method method) throws Throwable {
    try {
        // 每调用一个方法就会再代理对象中缓存方法对应的Method对象
        MapperMethodInvoker invoker = methodCache.get(method);
        if (invoker != null) {
            return invoker;
        }
        return methodCache.computeIfAbsent(method, m -> {
            if (m.isDefault()) {
                try {
                    if (privateLookupInMethod == null) {
                        return new DefaultMethodInvoker(getMethodHandleJava8(method));
                    } else {
                        return new DefaultMethodInvoker(getMethodHandleJava9(method));
                    }
                } catch (IllegalAccessException | InstantiationException | InvocationTargetException
                         | NoSuchMethodException e) {
                    throw new RuntimeException(e);
                }
            } else {
                return new PlainMethodInvoker(new MapperMethod(mapperInterface, method, sqlSession.getConfiguration()));
            }
        });
    } catch (RuntimeException re) {
        Throwable cause = re.getCause();
        throw cause == null ? re : cause;
    }
}
```

真正执行的CURD操作的 PlainMethodInvoker。

```java
// MapperMethod.java
public Object execute(SqlSession sqlSession, Object[] args) {
    Object result;
    switch (command.getType()) {	
        case INSERT: {
            Object param = method.convertArgsToSqlCommandParam(args);
            // 方法 SqlSession#insert(java.lang.String, java.lang.Object) 的第一个参数是statementd的id，第二个参数是sql的入参。
            result = rowCountResult(sqlSession.insert(command.getName(), param));
            break;
        }
        case UPDATE: {
            Object param = method.convertArgsToSqlCommandParam(args);
            result = rowCountResult(sqlSession.update(command.getName(), param));
            break;
        }
        case DELETE: {
            Object param = method.convertArgsToSqlCommandParam(args);
            result = rowCountResult(sqlSession.delete(command.getName(), param));
            break;
        }
        case SELECT:
            if (method.returnsVoid() && method.hasResultHandler()) {
                executeWithResultHandler(sqlSession, args);
                result = null;
            } else if (method.returnsMany()) {
                result = executeForMany(sqlSession, args);
            } else if (method.returnsMap()) {
                result = executeForMap(sqlSession, args);
            } else if (method.returnsCursor()) {
                result = executeForCursor(sqlSession, args);
            } else {
                Object param = method.convertArgsToSqlCommandParam(args);
                result = sqlSession.selectOne(command.getName(), param);
                if (method.returnsOptional()
                    && (result == null || !method.getReturnType().equals(result.getClass()))) {
                    result = Optional.ofNullable(result);
                }
            }
            break;
        case FLUSH:
            result = sqlSession.flushStatements();
            break;
        default:
            throw new BindingException("Unknown execution method for: " + command.getName());
    }
    if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
        throw new BindingException("Mapper method '" + command.getName()
               + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
    }
    return result;
}
```

```java
// DefaultSqlSession.java
@Override
public int insert(String statement, Object parameter) {
    return update(statement, parameter);
}

@Override
public int update(String statement, Object parameter) {
    try {
        dirty = true;
        MappedStatement ms = configuration.getMappedStatement(statement);
        return executor.update(ms, wrapCollection(parameter));
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error updating database.  Cause: " + e, e);
    } finally {
        ErrorContext.instance().reset();
    }
}

private Object wrapCollection(final Object object) {
    return ParamNameResolver.wrapToMapIfCollection(object, null);
}
```

```java
// ParamNameResolver.java
public static Object wrapToMapIfCollection(Object object, String actualParamName) {
    if (object instanceof Collection) {
        ParamMap<Object> map = new ParamMap<>();
        // mapper.xml 里的集合用list表示是从这里出来的
        map.put("collection", object);
        if (object instanceof List) {
            map.put("list", object);
        }
        Optional.ofNullable(actualParamName).ifPresent(name -> map.put(name, object));
        return map;
    } else if (object != null && object.getClass().isArray()) {
        ParamMap<Object> map = new ParamMap<>();
        // mapper.xml 里的数组用array表示是从这里出来的
        map.put("array", object);
        Optional.ofNullable(actualParamName).ifPresent(name -> map.put(name, object));
        return map;
    }
    return object;
}
```





```java
// DefaultSqlSession.java
public int update(MappedStatement ms, Object parameterObject) throws SQLException {
    flushCacheIfRequired(ms);
    return delegate.update(ms, parameterObject);
}
```









## SELECT 的执行









## 拦截器执行

