## parseConfiguration

### objectFactoryElement

```java
private void objectFactoryElement(XNode context) throws Exception {
    if (context != null) {
        String type = context.getStringAttribute("type");
        Properties properties = context.getChildrenAsProperties();
        ObjectFactory factory = (ObjectFactory) resolveClass(type).newInstance();
        factory.setProperties(properties);
        configuration.setObjectFactory(factory);
    }
}
```

#### ObjectFactory

当Mybatis想创建一个实体类对象的时候就会使用这个接口，比如 `List<Student> selectAll();`  这个方法对返回值进行封装的时候。

```java
public interface ObjectFactory {
  /**
   * Sets configuration properties.
   * @param properties configuration properties
   */
  void setProperties(Properties properties);
  /**
   * Creates a new object with default constructor. 
   * @param type Object type
   * @return
   */
  <T> T create(Class<T> type);
  /**
   * Creates a new object with the specified constructor and params.
   * @param type Object type
   * @param constructorArgTypes Constructor argument types
   * @param constructorArgs Constructor argument values
   * @return
   */
  <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs);
  /**
   * Returns true if this object can have a set of other objects.
   * It's main purpose is to support non-java.util.Collection objects like Scala collections.
   * 
   * @param type Object type
   * @return whether it is a collection or not
   * @since 3.1.0
   */
  <T> boolean isCollection(Class<T> type);
}
```

```java
public class DefaultObjectFactory implements ObjectFactory, Serializable {
  private static final long serialVersionUID = -8855120656740914948L;
  @Override
  public <T> T create(Class<T> type) {
    return create(type, null, null);
  }
  @SuppressWarnings("unchecked")
  @Override
  public <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
    Class<?> classToCreate = resolveInterface(type);
    // we know types are assignable
    return (T) instantiateClass(classToCreate, constructorArgTypes, constructorArgs);
  }
  @Override
  public void setProperties(Properties properties) {
    // no props for default
  }
  private  <T> T instantiateClass
      (Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
    try {
      Constructor<T> constructor;
      if (constructorArgTypes == null || constructorArgs == null) {
        constructor = type.getDeclaredConstructor();
        if (!constructor.isAccessible()) {
          constructor.setAccessible(true);
        }
        return constructor.newInstance();
      }
      constructor = type.getDeclaredConstructor
          (constructorArgTypes.toArray(new Class[constructorArgTypes.size()]));
      if (!constructor.isAccessible()) {
        constructor.setAccessible(true);
      }
      return constructor.newInstance(constructorArgs.toArray(new Object[constructorArgs.size()]));
    } catch (Exception e) {
      StringBuilder argTypes = new StringBuilder();
      if (constructorArgTypes != null && !constructorArgTypes.isEmpty()) {
        for (Class<?> argType : constructorArgTypes) {
          argTypes.append(argType.getSimpleName());
          argTypes.append(",");
        }
        argTypes.deleteCharAt(argTypes.length() - 1); // remove trailing ,
      }
      StringBuilder argValues = new StringBuilder();
      if (constructorArgs != null && !constructorArgs.isEmpty()) {
        for (Object argValue : constructorArgs) {
          argValues.append(String.valueOf(argValue));
          argValues.append(",");
        }
        argValues.deleteCharAt(argValues.length() - 1); // remove trailing ,
      }
      throw new ReflectionException("Error instantiating " 
       + type + " with invalid types (" + argTypes + ") or values (" + argValues + "). Cause: " + e, e);
    }
  }
  // 从这个方法可以看出来，返回的集合数据的原始类型
  protected Class<?> resolveInterface(Class<?> type) {
    Class<?> classToCreate;
    if (type == List.class || type == Collection.class || type == Iterable.class) {
      classToCreate = ArrayList.class;
    } else if (type == Map.class) {
      classToCreate = HashMap.class;
    } else if (type == SortedSet.class) { // issue #510 Collections Support
      classToCreate = TreeSet.class;
    } else if (type == Set.class) {
      classToCreate = HashSet.class;
    } else {
      classToCreate = type;
    }
    return classToCreate;
  }
  @Override
  public <T> boolean isCollection(Class<T> type) {
    return Collection.class.isAssignableFrom(type);
  }
}
```

`objectFactoryElement()` 方法的目的就是配置 `ObjectFactory`。



### objectWrapperFactoryElement

```java
private void objectWrapperFactoryElement(XNode context) throws Exception {
    if (context != null) {
        String type = context.getStringAttribute("type");
        ObjectWrapperFactory factory = (ObjectWrapperFactory) resolveClass(type).newInstance();
        configuration.setObjectWrapperFactory(factory);
    }
}
```



#### MetaObject

MetaClass 是对类的封装，而 MetaObject 是对对象的封装。

<div align="center"><img width="60%" src="image/12313132.jpg" /></div>

 ```java
public class TestMetaObject {
    public static void main(String[] args) throws Exception {
        Customer customer = new Customer();
        MetaObject metaObject = MetaObject.forObject(customer, new DefaultObjectFactory(), 
                                new DefaultObjectWrapperFactory(), new DefaultReflectorFactory());
        System.out.println(Arrays.deepToString(metaObject.getGetterNames()));
        System.out.println(metaObject.getValue("order"));
    }
    static class Customer {
        private Order order = new Order();
        private List<Address> addressList;
		// getter and setter
    }
    static class Order {
        String name;
        Double price;
		// getter and setter
    }
    static class Address {
        String name;
        String Postcode;
		// getter and setter
    }
}
 ```

MetaObject的实现中，我们可以看见，几乎所有的方法都被委托给了ObjectWrapper。

```java
public class MetaObject {
    private final Object originalObject;
    private final ObjectWrapper objectWrapper;
    private final ObjectFactory objectFactory;
    private final ObjectWrapperFactory objectWrapperFactory;
    private final ReflectorFactory reflectorFactory;
    private MetaObject(Object object, ObjectFactory objectFactory, 
                       ObjectWrapperFactory objectWrapperFactory, ReflectorFactory reflectorFactory) {
        this.originalObject = object;
        this.objectFactory = objectFactory;
        this.objectWrapperFactory = objectWrapperFactory;
        this.reflectorFactory = reflectorFactory;

        if (object instanceof ObjectWrapper) {
            this.objectWrapper = (ObjectWrapper) object;
        } else if (objectWrapperFactory.hasWrapperFor(object)) {
            this.objectWrapper = objectWrapperFactory.getWrapperFor(this, object);
        } else if (object instanceof Map) {
            this.objectWrapper = new MapWrapper(this, (Map) object);
        } else if (object instanceof Collection) {
            this.objectWrapper = new CollectionWrapper(this, (Collection) object);
        } else {
            this.objectWrapper = new BeanWrapper(this, object);
        }
    }

    public static MetaObject forObject(Object object, ObjectFactory objectFactory, 
                       ObjectWrapperFactory objectWrapperFactory, ReflectorFactory reflectorFactory) {
        if (object == null) {
            return SystemMetaObject.NULL_META_OBJECT;
        } else {
            return new MetaObject(object, objectFactory, objectWrapperFactory, reflectorFactory);
        }
    }
    
	// getter

    public String findProperty(String propName, boolean useCamelCaseMapping) {
        return objectWrapper.findProperty(propName, useCamelCaseMapping);
    }
    public String[] getGetterNames() {
        return objectWrapper.getGetterNames();
    }
    public String[] getSetterNames() {
        return objectWrapper.getSetterNames();
    }
    public Class<?> getSetterType(String name) {
        return objectWrapper.getSetterType(name);
    }
    public Class<?> getGetterType(String name) {
        return objectWrapper.getGetterType(name);
    }
    public boolean hasSetter(String name) {
        return objectWrapper.hasSetter(name);
    }
    public boolean hasGetter(String name) {
        return objectWrapper.hasGetter(name);
    }
    public Object getValue(String name) {
        PropertyTokenizer prop = new PropertyTokenizer(name);
        if (prop.hasNext()) {
            MetaObject metaValue = metaObjectForProperty(prop.getIndexedName());
            if (metaValue == SystemMetaObject.NULL_META_OBJECT) {
                return null;
            } else {
                return metaValue.getValue(prop.getChildren());
            }
        } else {
            return objectWrapper.get(prop);
        }
    }
    public void setValue(String name, Object value) {
        PropertyTokenizer prop = new PropertyTokenizer(name);
        if (prop.hasNext()) {
            MetaObject metaValue = metaObjectForProperty(prop.getIndexedName());
            if (metaValue == SystemMetaObject.NULL_META_OBJECT) {
                if (value == null && prop.getChildren() != null) {
                    // don't instantiate child path if value is null
                    return;
                } else {
                    metaValue = objectWrapper.instantiatePropertyValue(name, prop, objectFactory);
                }
            }
            metaValue.setValue(prop.getChildren(), value);
        } else {
            objectWrapper.set(prop, value);
        }
    }
    public MetaObject metaObjectForProperty(String name) {
        Object value = getValue(name);
        return MetaObject.forObject(value, objectFactory, objectWrapperFactory, reflectorFactory);
    }
    public ObjectWrapper getObjectWrapper() {
        return objectWrapper;
    }
    public boolean isCollection() {
        return objectWrapper.isCollection();
    }
    public void add(Object element) {
        objectWrapper.add(element);
    }
    public <E> void addAll(List<E> list) {
        objectWrapper.addAll(list);
    }
}
```

ObjectWrapper 是一个接口。

```java
public interface ObjectWrapper {
    Object get(PropertyTokenizer prop);
    void set(PropertyTokenizer prop, Object value);
    String findProperty(String name, boolean useCamelCaseMapping);
    String[] getGetterNames();
    String[] getSetterNames();
    Class<?> getSetterType(String name);
    Class<?> getGetterType(String name);
    boolean hasSetter(String name);
    boolean hasGetter(String name);
    MetaObject instantiatePropertyValue
        (String name, PropertyTokenizer prop, ObjectFactory objectFactory);
    boolean isCollection();
    void add(Object element);
    <E> void addAll(List<E> element);
}
```

<div align="center"><img width="45%" src="image/2020-09-13_212704.jpg" /></div>

MetaObject的作用是完成对实体类对象的包装，包装以后方便传参，以及处理结果。Mybatis给对象分为了两类，一类是集合，另一类包含Bean和Map。集合Wrapper只用在封装返回值的时候使用，所以它不用于传参。我们来写一个MapWrapper的Demo并Debug源码。

```

```



















































































