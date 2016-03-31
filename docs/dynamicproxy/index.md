#DynamicProxy

Since jdk1.3 the DynamicProxy is part of the JDK. 
The official documentation/API-Doc for JDK8 you can find 
[here](http://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Proxy.html)

To create Proxies based on the DynamicProxy you have to add the following dependencies to your project.
```xml
 <dependency>
      <groupId>org.rapidpm.proxybuilder</groupId>
      <artifactId>rapidpm-proxybuilder-modules-dynamic</artifactId>
      <version>${rapidpm.version}</version>
    </dependency>
```

If you are working with this kind of Proxies, you need always an interface. If you don´t have an interface in your code and you could not create one, you have to use the ***StaticVirtualProxies***.

##VirtualProxy

#### pure VirtualProxy
```java
final DemoLogic original = new DemoLogic();
final DemoInterface demoLogic = VirtualProxyBuilder
        .createBuilder(DemoInterface.class, original)
        .build();
```
With this you can generate at runtime a ***VirtualProxy*** based on the ***DynamicProxy***. This you can use beginning from Java 1.3 if you need it (You have to backport the code by yourself, but we would like to add this as legacy module). The sourcelevel we are using in this project is Java8.

#### Security Proxy
```java
final DemoLogic original = new DemoLogic();
final DemoInterface demoLogic = VirtualProxyBuilder
        .createBuilder(DemoInterface.class, original)
        .addSecurityRule(() -> false)
        .build();
```
This is a SecureVirtualProxy. The Security-Rules are invoked before the VirtualProxy ist activated. This means, the real Subject will not be created unless all SecurityRules are OK. 
Here`s an example for multiple SecurityRules:
```java
final InnerDemoClass original = new InnerDemoClass();
    final InnerDemoInterface demoLogic = VirtualProxyBuilder
        .createBuilder(InnerDemoInterface.class, original)
        .addSecurityRule(() -> true)
        .addSecurityRule(() -> true)
        .addSecurityRule(() -> false)
        .build();
```

#### Metrics Proxy

If you need the possibility to get Metrics out of your application, you could create a MetricsProxy. In this example we are creating a pure ***MetricsProxy***. You could do the following:

```java
    final InnerDemoInterface demoLogic = VirtualProxyBuilder
        .createBuilder(InnerDemoInterface.class, new InnerDemoClass())
        .addMetrics()
        .build();
```
 You also can combine this with a ***VirtualProxy*** to get a ***VirtualMetricsProxy***

```java
    final Service service = DynamicProxyBuilder
        .createBuilder(Service.class, ServiceImpl.class, CreationStrategy.SOME_DUPLICATES)
        .addMetrics()
        .build();
```

## PreAction -- PostAction
Sometimes you want to have the possibility to do something before or even after a method invocation. For this we have the 
***PreAction*** and ***PostAction***. The Actions are executed in the order they are added to the Proxy.

```java
    final Service service = DynamicProxyBuilder
        .createBuilder(Service.class, new ServiceImpl())
        .addIPreAction((original, method, args1) -> out.println("001 = " + method.getName()))
        .addIPreAction((original, method, args1) -> out.println("002 = " + method.getName()))
        .addIPreAction((original, method, args1) -> out.println("003 = " + method.getName()))
        .build();
```

A ***VirtualProxy*** with ***PreActions*** will lead to the execution of all ***PreActions** before the real subject is created. 

```java
    final Service service = DynamicProxyBuilder
        .createBuilder(Service.class, ServiceImpl.class, CreationStrategy.SOME_DUPLICATES)
        .addIPreAction((original, method, args1) -> out.println("001 = " + method.getName()))
        .addIPreAction((original, method, args1) -> out.println("002 = " + method.getName()))
        .addIPreAction((original, method, args1) -> out.println("003 = " + method.getName()))
        .build();
```

If you combine it with a ***SecurityVirtualProxy*** with ***PreActions*** the SecurityRule will be invoked first.

```java
    final Service service = DynamicProxyBuilder
        .createBuilder(Service.class, ServiceImpl.class, CreationStrategy.SOME_DUPLICATES)
        .addIPreAction((original, method, args1) -> out.println("001 = " + method.getName()))
        .addIPreAction((original, method, args1) -> out.println("002 = " + method.getName()))
        .addIPreAction((original, method, args1) -> out.println("003 = " + method.getName()))
        .addSecurityRule(() -> {
          out.println("sec 001");
          return true;
        })
        .build();
```

## CreationStrategies
With the ***CreationStrategies*** you can choose what will be the right way of synchronization for you in case you are using ***VirtualProxies***. If needed, you can implement others by yourself. With ***CreationStrategies*** you can do different things. Not only the creation of one element is possible, you can f.e. create a pool of instances used randomly
 or you can do something like ***MethodScoped***. 

### MethodScoped
This ***CreationStrategy*** will create for every method invocation a new instance.

```java
public class ServiceStrategyFactoryMethodScoped<T> implements ServiceStrategyFactory<T> {

  @Override
  public synchronized T realSubject(ServiceFactory<T> factory) {
    return factory.createInstance();
  }
}
```


### NONE
If you choose nothing or ***CreationStrategy.NONE*** you will get the ***NotThreadSafe*** version.

```java
public class ServiceStrategyFactoryNotThreadSafe<T> implements ServiceStrategyFactory<T> {

  private T service;

  @Override
  public T realSubject(ServiceFactory<T> factory) {
    if (service == null) {
      service = factory.createInstance();
    }
    return service;
  }

}
```

### SomeDuplicates

```java
public class ServiceStrategyFactorySomeDuplicates<T> implements ServiceStrategyFactory<T> {
  private final AtomicReference<T> ref = new AtomicReference<>();

  @Override
  public T realSubject(ServiceFactory<T> factory) {

    T service = ref.get();
    if (service == null) {
      service = factory.createInstance();
      if (!ref.compareAndSet(null, service)) {
        service = ref.get();
      }
    }
    return service;
  }
}
```

### Synchronized

```java
public class ServiceStrategyFactorySynchronized<T> implements ServiceStrategyFactory<T> {

  private T service;

  @Override
  public synchronized T realSubject(ServiceFactory<T> factory) {
    if (service == null) {
      service = factory.createInstance();
    }
    return service;
  }
}
```

###NoDuplicates

```java
public class ServiceStrategyFactoryNoDuplicates<T> implements ServiceStrategyFactory<T> {

  private final Lock initializationLock = new ReentrantLock();
  private volatile T realSubject;

  @Override
  public T realSubject(ServiceFactory<T> factory) {
    T result = realSubject;
    if (result == null) {
      initializationLock.lock();
      try {
        result = realSubject;
        if (result == null) {
          result = realSubject = factory.createInstance();
        }
      } finally {
        initializationLock.unlock();
      }
    }
    return result;
  }
}
```
