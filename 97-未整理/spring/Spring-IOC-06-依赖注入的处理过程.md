## 依赖的处理过程

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

对于注入的一个普通Bean，这里也是使用doResolveDependency方法处理。实际上真正的处理依赖的方法只有一个，doResolveDependency，其他的都只是对它的包装，就是所以后面就会介绍这个方法的执行。



### DependencyDescriptor

```java
public class InjectionPoint {
    // 被注入的是方法，此字段不为空
    @Nullable
    protected MethodParameter methodParameter;

    // 被注入的是属性，此字段不为空
    @Nullable
    protected Field field;

    // 被注入的是属性，此字段不为空
    @Nullable
    private volatile Annotation[] fieldAnnotations;
}
```

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

    // 如果一个注入点被标记为required为ture，当无法注入时
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

DependencyDescriptor是用来描述依赖的，我们已经看见这个类很久了，它有一个父类InjectPoint，InjectPoint只有一个直接子类，就是DependencyDescriptor。





### doResolveDependency

到现在为止，依赖注入的ObjectProvider情景、Optional情景和Provider情景。现在还有四种类型：

1. Bean类型
2. Collection类型
3. Map类型
4. 限定注入

除了这四种类型，还有一种外部依赖@Value，这个也是在这个方法里面处理。这个我们在外部化配置章节里面分析。

这四种类型的注入功能都是由方法doResolveDependency完成的。

#### 处理multipleBeans

处理multipleBeans会处理上面四种类型的后三种类型。

```java
public Object doResolveDependency(DependencyDescriptor descriptor, @Nullable String beanName,
    	@Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter)
    		throws BeansException {

    InjectionPoint previousInjectionPoint = ConstructorResolver.setCurrentInjectionPoint(descriptor);
    try {
		// shortcut ...

		// 处理@Value

        Object multipleBeans = 
            resolveMultipleBeans(descriptor, beanName, autowiredBeanNames, typeConverter);
        if (multipleBeans != null) {
            return multipleBeans;
        }

        // 处理其他情况
    } finally {
        ConstructorResolver.setCurrentInjectionPoint(previousInjectionPoint);
    }
}
```

```java
private Object resolveMultipleBeans(DependencyDescriptor descriptor, @Nullable String beanName,
    	@Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) {

    final Class<?> type = descriptor.getDependencyType();

    if (descriptor instanceof StreamDependencyDescriptor) {
        Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
        if (autowiredBeanNames != null) {
            autowiredBeanNames.addAll(matchingBeans.keySet());
        }
        Stream<Object> stream = matchingBeans.keySet().stream()
            .map(name -> descriptor.resolveCandidate(name, type, this))
            .filter(bean -> !(bean instanceof NullBean));
        // 这里的isOrdered是通过StreamDependencyDescriptor的构造方法设置的
        // 此构造方法只有在 ObjectProvider 情景才会被使用
        if (((StreamDependencyDescriptor) descriptor).isOrdered()) {
            stream = stream.sorted(adaptOrderComparator(matchingBeans));
        }
        return stream;
    }
    else if (type.isArray()) {
        Class<?> componentType = type.getComponentType();
        ResolvableType resolvableType = descriptor.getResolvableType();
        Class<?> resolvedArrayType = resolvableType.resolve(type);
        if (resolvedArrayType != type) {
            componentType = resolvableType.getComponentType().resolve();
        }
        if (componentType == null) {
            return null;
        }
        Map<String, Object> matchingBeans = findAutowireCandidates(beanName, componentType,
        													new MultiElementDescriptor(descriptor));
        if (matchingBeans.isEmpty()) {
            return null;
        }
        if (autowiredBeanNames != null) {
            autowiredBeanNames.addAll(matchingBeans.keySet());
        }
        TypeConverter converter = (typeConverter != null ? typeConverter : getTypeConverter());
        Object result = converter.convertIfNecessary(matchingBeans.values(), resolvedArrayType);
        if (result instanceof Object[]) {
            Comparator<Object> comparator = adaptDependencyComparator(matchingBeans);
            if (comparator != null) {
                Arrays.sort((Object[]) result, comparator);
            }
        }
        return result;
    }
    else if (Collection.class.isAssignableFrom(type) && type.isInterface()) {
        Class<?> elementType = descriptor.getResolvableType().asCollection().resolveGeneric();
        if (elementType == null) {
            return null;
        }
        Map<String, Object> matchingBeans = findAutowireCandidates(beanName, elementType,
        													new MultiElementDescriptor(descriptor));
        if (matchingBeans.isEmpty()) {
            return null;
        }
        if (autowiredBeanNames != null) {
            autowiredBeanNames.addAll(matchingBeans.keySet());
        }
        TypeConverter converter = (typeConverter != null ? typeConverter : getTypeConverter());
        Object result = converter.convertIfNecessary(matchingBeans.values(), type);
        if (result instanceof List) {
            Comparator<Object> comparator = adaptDependencyComparator(matchingBeans);
            if (comparator != null) {
                ((List<?>) result).sort(comparator);
            }
        }
        return result;
    }
    else if (Map.class == type) {
        ResolvableType mapType = descriptor.getResolvableType().asMap();
        Class<?> keyType = mapType.resolveGeneric(0);
        if (String.class != keyType) {
            return null;
        }
        Class<?> valueType = mapType.resolveGeneric(1);
        if (valueType == null) {
            return null;
        }
        Map<String, Object> matchingBeans = findAutowireCandidates(beanName, valueType,
        													new MultiElementDescriptor(descriptor));
        if (matchingBeans.isEmpty()) {
            return null;
        }
        if (autowiredBeanNames != null) {
            autowiredBeanNames.addAll(matchingBeans.keySet());
        }
        return matchingBeans;
    }
    else {
        return null;
    }
}
```

resolveMultipleBeans方法分了四种情况来处理：

1. Stream
2. 数组
3. Collection
4. Map

虽然是四种情况，但是本质都是调用了findAutowireCandidates方法来寻找依赖注入的Beans。

```java
protected Map<String, Object> findAutowireCandidates(
    @Nullable String beanName, Class<?> requiredType, DependencyDescriptor descriptor) {

    String[] candidateNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
        this, requiredType, true, descriptor.isEager());
    Map<String, Object> result = new LinkedHashMap<>(candidateNames.length);
    
    // resolvableDependencies的结构是：<依赖的类型, 注入的对象>
    for (Map.Entry<Class<?>, Object> classObjectEntry : this.resolvableDependencies.entrySet()) {
        Class<?> autowiringType = classObjectEntry.getKey();
        if (autowiringType.isAssignableFrom(requiredType)) {
            Object autowiringValue = classObjectEntry.getValue();
            autowiringValue = AutowireUtils.resolveAutowiringValue(autowiringValue, requiredType);
            if (requiredType.isInstance(autowiringValue)) {
                // 这里添加的是<名称, 注入的对象>
                result.put(ObjectUtils.identityToString(autowiringValue), autowiringValue);
                break;
            }
        }
    }
    
    // 在非自我引用的情况下，会判断是否可以加入
    // 如果 A里面定义了Bean b1和b2，同时需要将b1和b2注入到b这个集合里面，就属于自我引用
    // isAutowireCandidate会处理@Autowired注解和@Qualifier注解，这个后面单独解释
    for (String candidate : candidateNames) {
        if (!isSelfReference(beanName, candidate) && isAutowireCandidate(candidate, descriptor)) {
            addCandidateEntry(result, candidate, descriptor, requiredType);
        }
    }
    if (result.isEmpty()) {
        // 只有数组、Collection和Map情景会返回true，Stream和普通Bean返回false
        boolean multiple = indicatesMultipleBeans(requiredType);
        // Consider fallback matches if the first pass failed to find anything...
        // 我对FallBack的理解就是在遇到泛型的原始类型时，将所有的可以设置的类都放在里面
        DependencyDescriptor fallbackDescriptor = descriptor.forFallbackMatch();
        for (String candidate : candidateNames) {
            if (!isSelfReference(beanName, candidate) && 
                	isAutowireCandidate(candidate, fallbackDescriptor) &&
                (!multiple || getAutowireCandidateResolver().hasQualifier(descriptor))) {
                addCandidateEntry(result, candidate, descriptor, requiredType);
            }
        }
        if (result.isEmpty() && !multiple) {
            // Consider self references as a final pass...
            // but in the case of a dependency collection, not the very same bean itself.
            // 在普通Bean且不注入自己的时候给自我引用一个机会，但是待注入的Bean不能是数组、Collection和Map
            for (String candidate : candidateNames) {
                if (isSelfReference(beanName, candidate) &&
                    (!(descriptor instanceof MultiElementDescriptor) || !beanName.equals(candidate)) &&
                    isAutowireCandidate(candidate, fallbackDescriptor)) {
                    addCandidateEntry(result, candidate, descriptor, requiredType);
                }
            }
        }
    }
    return result;
}
```

addCandidateEntry的作用是将Bean设置在集合中。普通的Bean就直接返回 `Map<id, beanInstance>`，对于泛型的原始类型会返回 `Map<id, type>`。

```java
private void addCandidateEntry(Map<String, Object> candidates, String candidateName,
                               DependencyDescriptor descriptor, Class<?> requiredType) {

    // 处理 数组、Collection和Map情景
    if (descriptor instanceof MultiElementDescriptor) {
        Object beanInstance = descriptor.resolveCandidate(candidateName, requiredType, this);
        if (!(beanInstance instanceof NullBean)) {
            candidates.put(candidateName, beanInstance);
        }
    }
    // 处理Stream情景
    else if (containsSingleton(candidateName) || (descriptor instanceof StreamDependencyDescriptor &&
    				((StreamDependencyDescriptor) descriptor).isOrdered())) {
        Object beanInstance = descriptor.resolveCandidate(candidateName, requiredType, this);
        candidates.put(candidateName, (beanInstance instanceof NullBean ? null : beanInstance));
    }
    // 个人理解，在处理泛型的原始类型时会走到这里，比如处理属性：
    // 	private List strs;
    // 可以注入List<String>类型的数据
    else {
        candidates.put(candidateName, getType(candidateName));
    }
}
```

#### 处理普通的Bean

```java
public Object doResolveDependency(DependencyDescriptor descriptor, @Nullable String beanName,
    	@Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter)
    		throws BeansException {

    InjectionPoint previousInjectionPoint = ConstructorResolver.setCurrentInjectionPoint(descriptor);
    try {
		// shortcut ...

		// 处理@Value

        // 处理multipleBeans

        // 对于普通的Bean，同样通过findAutowireCandidates寻找Bean
        Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
        // 如果找不到，抛异常
        if (matchingBeans.isEmpty()) {
            if (isRequired(descriptor)) {
                raiseNoMatchingBeanFound(type, descriptor.getResolvableType(), descriptor);
            }
            return null;
        }

        String autowiredBeanName;
        Object instanceCandidate;

        if (matchingBeans.size() > 1) {
            // 对于候选者有多个的情景，寻找真正的Bean，即：@Primary和优先级最高的
            autowiredBeanName = determineAutowireCandidate(matchingBeans, descriptor);
            if (autowiredBeanName == null) {
                if (isRequired(descriptor) || !indicatesMultipleBeans(type)) {
                    // 默认情况会抛出异常，子类可以覆盖此方法。getIfUnique
                    // 在ObjectProvider中调用getIfUnique方法，会覆盖此方法，让其不抛异常，返回null
                    return descriptor.resolveNotUnique(descriptor.getResolvableType(), matchingBeans);
                } else {
                    // In case of an optional Collection/Map, silently ignore a non-unique case:
                    // possibly it was meant to be an empty collection of multiple regular beans
                    // (before 4.3 in particular when we didn't even look for collection beans).
                    return null;
                }
            }
            instanceCandidate = matchingBeans.get(autowiredBeanName);
        } else {
            // We have exactly one match.
            Map.Entry<String, Object> entry = matchingBeans.entrySet().iterator().next();
            autowiredBeanName = entry.getKey();
            instanceCandidate = entry.getValue();
        }

        if (autowiredBeanNames != null) {
            autowiredBeanNames.add(autowiredBeanName);
        }
        // 对于泛型的原始类型情景，还需要获取真正的beanInstance
        if (instanceCandidate instanceof Class) {
            instanceCandidate = descriptor.resolveCandidate(autowiredBeanName, type, this);
        }
        Object result = instanceCandidate;
        if (result instanceof NullBean) {
            if (isRequired(descriptor)) {
                raiseNoMatchingBeanFound(type, descriptor.getResolvableType(), descriptor);
            }
            result = null;
        }
        if (!ClassUtils.isAssignableValue(type, result)) {
            throw new BeanNotOfRequiredTypeException
                (autowiredBeanName, type, instanceCandidate.getClass());
        }
        return result;
    } finally {
        ConstructorResolver.setCurrentInjectionPoint(previousInjectionPoint);
    }
}
```

#### 寻找候选者

```java
// DefaultListableBeanFactory.java
public boolean isAutowireCandidate(String beanName, DependencyDescriptor descriptor)
    throws NoSuchBeanDefinitionException {
    return isAutowireCandidate(beanName, descriptor, getAutowireCandidateResolver());
}
protected boolean isAutowireCandidate(String beanName, DependencyDescriptor descriptor, 
    	AutowireCandidateResolver resolver) throws NoSuchBeanDefinitionException {

    String beanDefinitionName = BeanFactoryUtils.transformedBeanName(beanName);
    // 当Bean是BeanDefinititon走这个路
    if (containsBeanDefinition(beanDefinitionName)) {
        return isAutowireCandidate(beanName, 
               		getMergedLocalBeanDefinition(beanDefinitionName), descriptor, resolver);
    }
    else if (containsSingleton(beanName)) {
        return isAutowireCandidate(beanName, 
                                   new RootBeanDefinition(getType(beanName)), descriptor, resolver);
    }

    BeanFactory parent = getParentBeanFactory();
    if (parent instanceof DefaultListableBeanFactory) {
        // No bean definition found in this factory -> delegate to parent.
        return ((DefaultListableBeanFactory) parent).
            isAutowireCandidate(beanName, descriptor, resolver);
    }
    else if (parent instanceof ConfigurableListableBeanFactory) {
        // If no DefaultListableBeanFactory, can't pass the resolver along.
        return ((ConfigurableListableBeanFactory) parent).isAutowireCandidate(beanName, descriptor);
    }
    else {
        return true;
    }
}
protected boolean isAutowireCandidate(String beanName, RootBeanDefinition mbd,
    	DependencyDescriptor descriptor, AutowireCandidateResolver resolver) {

    String beanDefinitionName = BeanFactoryUtils.transformedBeanName(beanName);
    resolveBeanClass(mbd, beanDefinitionName);
    if (mbd.isFactoryMethodUnique && mbd.factoryMethodToIntrospect == null) {
        new ConstructorResolver(this).resolveFactoryMethodIfPossible(mbd);
    }
    return resolver.isAutowireCandidate(
        new BeanDefinitionHolder(mbd, beanName, getAliases(beanDefinitionName)), descriptor);
}
```

我们获取的是ContextAnnotationAutowireCandidateResolver，isAutowireCandidate的执行流程如下：

```java
// SimpleAutowireCandidateResolver.java
public boolean isAutowireCandidate(BeanDefinitionHolder bdHolder, DependencyDescriptor descriptor) {
    // 默认返回true
    return bdHolder.getBeanDefinition().isAutowireCandidate();
}
```

```java
// GenericTypeAwareAutowireCandidateResolver.java
public boolean isAutowireCandidate(BeanDefinitionHolder bdHolder, DependencyDescriptor descriptor) {
    if (!super.isAutowireCandidate(bdHolder, descriptor)) {
        // If explicitly false, do not proceed with any other checks...
        return false;
    }
    return checkGenericTypeMatch(bdHolder, descriptor);
}

protected boolean checkGenericTypeMatch
    	(BeanDefinitionHolder bdHolder, DependencyDescriptor descriptor) {
    ResolvableType dependencyType = descriptor.getResolvableType();
    
    // 分为两种情况考虑：类和泛型。泛型的原始类型属于类类型，带指定泛型的不是类类型
    
    if (dependencyType.getType() instanceof Class) {
        // No generic type -> we know it's a Class type-match, so no need to check again.
        return true;
    }

    // targetType是待注入的Bean的type
    ResolvableType targetType = null;
    boolean cacheType = false;
    RootBeanDefinition rbd = null;
    if (bdHolder.getBeanDefinition() instanceof RootBeanDefinition) {
        rbd = (RootBeanDefinition) bdHolder.getBeanDefinition();
    }
    if (rbd != null) {
        targetType = rbd.targetType;
        if (targetType == null) {
            cacheType = true;
            // First, check factory method return type, if applicable
            targetType = getReturnTypeForFactoryMethod(rbd, descriptor);
            if (targetType == null) {
                RootBeanDefinition dbd = getResolvedDecoratedDefinition(rbd);
                if (dbd != null) {
                    targetType = dbd.targetType;
                    if (targetType == null) {
                        targetType = getReturnTypeForFactoryMethod(dbd, descriptor);
                    }
                }
            }
        }
    }

    if (targetType == null) {
        // Regular case: straight bean instance, with BeanFactory available.
        if (this.beanFactory != null) {
            Class<?> beanType = this.beanFactory.getType(bdHolder.getBeanName());
            if (beanType != null) {
                targetType = ResolvableType.forClass(ClassUtils.getUserClass(beanType));
            }
        }
        // Fallback: no BeanFactory set, or no type resolvable through it
        // -> best-effort match against the target class if applicable.
        if (targetType == null && rbd != null && rbd.hasBeanClass() 
            	&& rbd.getFactoryMethodName() == null) {
            Class<?> beanClass = rbd.getBeanClass();
            if (!FactoryBean.class.isAssignableFrom(beanClass)) {
                targetType = ResolvableType.forClass(ClassUtils.getUserClass(beanClass));
            }
        }
    }

    if (targetType == null) {
        return true;
    }
    if (cacheType) {
        rbd.targetType = targetType;
    }
    if (descriptor.fallbackMatchAllowed() &&
        (targetType.hasUnresolvableGenerics() || targetType.resolve() == Properties.class)) {
        // Fallback matches allow unresolvable generics, e.g. plain HashMap to Map<String,String>;
        // and pragmatically also java.util.Properties to any Map (since despite formally being a
        // Map<Object,Object>, java.util.Properties is usually perceived as a Map<String,String>).
        return true;
    }
    // Full check for complex generic type match...
    return dependencyType.isAssignableFrom(targetType);
}
```

GenericTypeAwareAutowireCandidateResolver的isAutowireCandidate是判断泛型的类型是否合法。

```java
// QualifierAnnotationAutowireCandidateResolve.java
public boolean isAutowireCandidate(BeanDefinitionHolder bdHolder, DependencyDescriptor descriptor) {
    boolean match = super.isAutowireCandidate(bdHolder, descriptor);
    if (match) {
        match = checkQualifiers(bdHolder, descriptor.getAnnotations());
        if (match) {
            MethodParameter methodParam = descriptor.getMethodParameter();
            if (methodParam != null) {
                Method method = methodParam.getMethod();
                if (method == null || void.class == method.getReturnType()) {
                    match = checkQualifiers(bdHolder, methodParam.getMethodAnnotations());
                }
            }
        }
    }
    return match;
}

// 检查BeanDefinition是否存在和注解数组中
protected boolean checkQualifiers(BeanDefinitionHolder bdHolder, Annotation[] annotationsToSearch) {
    if (ObjectUtils.isEmpty(annotationsToSearch)) {
        return true;
    }
    SimpleTypeConverter typeConverter = new SimpleTypeConverter();
    for (Annotation annotation : annotationsToSearch) {
        Class<? extends Annotation> type = annotation.annotationType();
        boolean checkMeta = true;
        boolean fallbackToMeta = false;
        // 查看注解是否是限定类型中的类型
        // 如果有，则判断BeanDefinition是否会能匹配上限定的类型
        // 如果没有，则检查当前注解的头上的元注解是否能满足限定的类型
        if (isQualifier(type)) {
            if (!checkQualifier(bdHolder, annotation, typeConverter)) {
                fallbackToMeta = true;
            } else {
                checkMeta = false;
            }
        }
        if (checkMeta) {
            boolean foundMeta = false;
            for (Annotation metaAnn : type.getAnnotations()) {
                Class<? extends Annotation> metaType = metaAnn.annotationType();
                if (isQualifier(metaType)) {
                    foundMeta = true;
                    // Only accept fallback match if @Qualifier annotation has a value...
                    // Otherwise it is just a marker for a custom qualifier annotation.
                    if ((fallbackToMeta && StringUtils.isEmpty(AnnotationUtils.getValue(metaAnn))) ||
                        !checkQualifier(bdHolder, metaAnn, typeConverter)) {
                        return false;
                    }
                }
            }
            if (fallbackToMeta && !foundMeta) {
                return false;
            }
        }
    }
    return true;
}

protected boolean isQualifier(Class<? extends Annotation> annotationType) {
    for (Class<? extends Annotation> qualifierType : this.qualifierTypes) {
        if (annotationType.equals(qualifierType) || annotationType.isAnnotationPresent(qualifierType)) {
            return true;
        }
    }
    return false;
}
```

检查限定注解和BeanDefinition是否一致的方法是checkQualifier。

```java
protected boolean checkQualifier(
    BeanDefinitionHolder bdHolder, Annotation annotation, TypeConverter typeConverter) {
	// bdHolder是待注入的Bean
    
    Class<? extends Annotation> type = annotation.annotationType();
    RootBeanDefinition bd = (RootBeanDefinition) bdHolder.getBeanDefinition();

    AutowireCandidateQualifier qualifier = bd.getQualifier(type.getName());
    if (qualifier == null) {
        qualifier = bd.getQualifier(ClassUtils.getShortName(type));
    }
    if (qualifier == null) {
        // First, check annotation on qualified element, if any
        Annotation targetAnnotation = getQualifiedElementAnnotation(bd, type);
        // Then, check annotation on factory method, if applicable
        if (targetAnnotation == null) {
            // 如果字段上是@Qualifier，Bean定义是@UserGroup1，返回空
            // 如果字段定义是@UserGroup1，字段上是@Qualifier，返回@UserGroup1
            targetAnnotation = getFactoryMethodAnnotation(bd, type);
        }
        if (targetAnnotation == null) {
            RootBeanDefinition dbd = getResolvedDecoratedDefinition(bd);
            if (dbd != null) {
                targetAnnotation = getFactoryMethodAnnotation(dbd, type);
            }
        }
        
        if (targetAnnotation == null) {
            // Look for matching annotation on the target class
            if (getBeanFactory() != null) {
                try {
                    Class<?> beanType = getBeanFactory().getType(bdHolder.getBeanName());
                    if (beanType != null) {
                        targetAnnotation = 
                            AnnotationUtils.getAnnotation(ClassUtils.getUserClass(beanType), type);
                    }
                }
                catch (NoSuchBeanDefinitionException ex) {
                    // Not the usual case - simply forget about the type check...
                }
            }
            if (targetAnnotation == null && bd.hasBeanClass()) {
                targetAnnotation = 
                    AnnotationUtils.getAnnotation(ClassUtils.getUserClass(bd.getBeanClass()), type);
            }
        }
        
        // 传入的参数annotation是字段上面的注解，如果能进入此方法，表明是@Qualifier注解
        // targetAnnotation是定义Bean的时候在Bean头上加的注解：targetAnnotation的寻找有四种方式：
        // 1、BeanDefinition在定义的时候可以调用setQualifiedElement方法设置Qualifier
        // 2、在创建Bean的FactoryMethod上加@Qualifier
        // 3、如果BeanDefinition有父子关系，子可以使用父的@Qualifier
        // 4、在Bean的类上可以使用@Qualifier注解
        // 如果Bean定义时限定注解和字段上面的限定注解一致时，就会返回真
        if (targetAnnotation != null && targetAnnotation.equals(annotation)) {
            return true;
        }
    }

    Map<String, Object> attributes = AnnotationUtils.getAnnotationAttributes(annotation);
    if (attributes.isEmpty() && qualifier == null) {
        // If no attributes, the qualifier must be present
        return false;
    }
    for (Map.Entry<String, Object> entry : attributes.entrySet()) {
        String attributeName = entry.getKey();
        Object expectedValue = entry.getValue();
        Object actualValue = null;
        // Check qualifier first
        // 针对每一个类型可以设置一个qualifier，这是第一优先级的
        if (qualifier != null) {
            actualValue = qualifier.getAttribute(attributeName);
        }
        // BeanDefinition可以存在一些附加属性，这些属性也可以被用于匹配。
        if (actualValue == null) {
            // Fall back on bean definition attribute
            actualValue = bd.getAttribute(attributeName);
        }
        // 如果注解里的属性名是value且字段限定的要求是String类型，查看beanDifinition和请求的名称是否一致。
        // 此时又进入了bean名称的匹配逻辑中
        if (actualValue == null && attributeName.equals(AutowireCandidateQualifier.VALUE_KEY) &&
            expectedValue instanceof String && bdHolder.matchesName((String) expectedValue)) {
            // Fall back on bean name (or alias) match
            continue;
        }
        if (actualValue == null && qualifier != null) {
            // Fall back on default, but only if the qualifier is present
            actualValue = AnnotationUtils.getDefaultValue(annotation, attributeName);
        }
        if (actualValue != null) {
            actualValue = typeConverter.convertIfNecessary(actualValue, expectedValue.getClass());
        }
        if (!expectedValue.equals(actualValue)) {
            return false;
        }
    }
    return true;
}
```



### 测试

#### 懒注入的测试

```java
public class InjectionMethodDebugDemo {
    @Lazy
    @Autowired
    private void init(User user1, User user2) {
        System.out.println(user1.getClass());
        System.out.println(user2.getClass());
    }

    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext =
                new AnnotationConfigApplicationContext();

        applicationContext.register(InjectionMethodDebugDemo.class);
        applicationContext.refresh();
        applicationContext.close();
    }

    @Bean
    public User user() {
        return new User(888L, "jinhao", City.SHENZHEN);
    }
}
// class fsc.domain.User$$EnhancerBySpringCGLIB$$ba1c3628
// class fsc.domain.User$$EnhancerBySpringCGLIB$$ba1c3628
```

#### 注入属性

```java
@Import(InjectionMethodDebugDemo.class)
public class InjectionFiledDebugDemo {
    @Autowired
    private User user;

    @Autowired
    @Qualifier("userHolder1")
    private UserHolder userHolder;

    @Autowired
    private List<UserHolder> userHolderList;

    @Autowired
    private Map<String, UserHolder> userHolderMap;

    @Autowired
    private ObjectProvider<UserHolder> userHolderObjectProvider;

    @Autowired
    private Optional<User> userOptional;
    
    public static void main(String[] args) {

        AnnotationConfigApplicationContext applicationContext =
                new AnnotationConfigApplicationContext();

        applicationContext.register(InjectionFiledDebugDemo.class);
        applicationContext.refresh();
        InjectionFiledDebugDemo demo = applicationContext.getBean(InjectionFiledDebugDemo.class);

        System.out.println(demo.user);
        System.out.println(demo.userHolder);
        System.out.println(demo.userHolderList);
        System.out.println(demo.userHolderMap);
        demo.userHolderObjectProvider.stream().forEach(System.out::println);
        System.out.println(demo.userOptional);
        applicationContext.close();
    }

    @Bean
    public UserHolder userHolder1(User user) {
        UserHolder userHolder = new UserHolder(user);
        userHolder.setId("userHolder1");
        return userHolder;
    }

    @Bean
    public UserHolder userHolder2(User user) {
        UserHolder userHolder = new UserHolder(user);
        userHolder.setId("userHolder2");
        return userHolder;
    }
}
```













