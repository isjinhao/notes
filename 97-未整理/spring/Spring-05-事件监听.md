## JDK的事件监听接口

JDK提供了一个经典的监听者模型。Observable接口是可被监听的对象，Observer是监听的对象。

```java
public class JDKObserverDemo {
    public static void main(String[] args) {
        EventObservable observable = new EventObservable();
        // 注册观察者（监听者）
        observable.addObserver(new EventObserver());
        // 发布消息（事件）
        observable.notifyObservers("Hello,World");
    }
    static class EventObservable extends Observable {
        @Override
        public void notifyObservers(Object arg) {
            setChanged();
            super.notifyObservers(arg);
            clearChanged();
        }
    }
    static class EventObserver implements Observer {
        @Override
        public void update(Observable o, Object arg) {
            System.out.println("收到事件 ：" + arg);
        }
    }
}
```

同时JDK提供了EventListener接口和EventObject接口。前者是事件监听器的标识接口，后者封装了事件。

```java
public interface EventListener {
}
```

```java
public class EventObject implements java.io.Serializable {
    private static final long serialVersionUID = 5516075349620653480L;
    protected transient Object  source;
    public EventObject(Object source) {
        if (source == null)
            throw new IllegalArgumentException("null source");
        this.source = source;
    }
    public Object getSource() {
        return source;
    }
    public String toString() {
        return getClass().getName() + "[source=" + source + "]";
    }
}
```

两个类都很简单，不再解释了。









