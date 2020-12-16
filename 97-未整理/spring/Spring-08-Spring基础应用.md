## 外部化配置

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

### @Value

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

可以看出，上面的代码就是将@Value里value属性的值取出来。

取出来的值，通过StringValueResolver解析。

```java
// AbstractBeanFactory.java
private final List<StringValueResolver> embeddedValueResolvers = new CopyOnWriteArrayList<>();
public String resolveEmbeddedValue(@Nullable String value) {
    if (value == null) {
        return null;
    }
    String result = value;
    for (StringValueResolver resolver : this.embeddedValueResolvers) {
        result = resolver.resolveStringValue(result);
        if (result == null) {
            return null;
        }
    }
    return result;
}
```

这个属性是在BeanFactory启动的时候被设置的。具体是在refresh方法的finishBeanFactoryInitialization方法中。

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

```java
// ConfigurableBeanFactory.java 定义
// AbstractBeanFactory.java 实现
public void addEmbeddedValueResolver(StringValueResolver valueResolver) {
    Assert.notNull(valueResolver, "StringValueResolver must not be null");
    this.embeddedValueResolvers.add(valueResolver);
}
```

还有一个地方会调用addEmbeddedValueResolver方法，就是在解析context:property-placeholder标签时，这个后面再说。

在finishBeanFactoryInitialization方法中添加的在这个行为，就是对于任意一个字符串，都调用Environment的resolvePlaceholders方法进行处理。Environment的默认实现类是StandardEnvironment。

```java
// StandardEnvironment.java
public String resolvePlaceholders(String text) {
    return this.propertyResolver.resolvePlaceholders(text);
}
```

StandardEnvironment里的实现是使用PropertySourcesPropertyResolver类进行的处理。

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
    return helper.replacePlaceholders(text, this::getPropertyAsRawString);
}
```

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

PropertyPlaceholderHelper

```java
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
            String placeholder = result.substring(startIndex + this.placeholderPrefix.length(), endIndex);
            String originalPlaceholder = placeholder;
            if (visitedPlaceholders == null) {
                visitedPlaceholders = new HashSet<>(4);
            }
            if (!visitedPlaceholders.add(originalPlaceholder)) {
                throw new IllegalArgumentException(
                    "Circular placeholder reference '" + originalPlaceholder + "' in property definitions");
            }
            // Recursive invocation, parsing placeholders contained in the placeholder key.
            placeholder = parseStringValue(placeholder, placeholderResolver, visitedPlaceholders);
            // Now obtain the value for the fully resolved key...
            String propVal = placeholderResolver.resolvePlaceholder(placeholder);
            if (propVal == null && this.valueSeparator != null) {
                int separatorIndex = placeholder.indexOf(this.valueSeparator);
                if (separatorIndex != -1) {
                    String actualPlaceholder = placeholder.substring(0, separatorIndex);
                    String defaultValue = placeholder.substring(separatorIndex + this.valueSeparator.length());
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







