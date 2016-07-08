# Object Adapter
In this section we want to describe the ObjectAdapter Pattern that is realized in this module.
We can use a dynamic and a static version of the ***ObjectAdapter*** Pattern. The dynamic version is based on the DynamicProxy from jdk1.3 and the static version is purely generated via Annotation Processing. 

## DynamicObjectAdapter
If you want to use the ***DynamicObjectAdapter*** you need at leaset one Interface you can cast to. To create the ***DynamicObjectAdapter*** you can use the Annotation ***@DynamicObjectAdapterBuilder***. This will create via Annotation Processing the ***DynamicObjectAdapter*** and the corresponding ***Builder*** that you can use for convenience and type safety. Let´s assume you have the following interface ***Service***:


```java
@DynamicObjectAdapterBuilder
public interface Service {
  String doWork(String txt);
  String doMoreWorkA(String txt);
  String doMoreWorkB(String txt);
  String doMoreWorkC(String txt);
  String doMoreWorkD(String txt);
}
``` 

With the Annotation we are generating the corresponding parts, used for the typesafe generated DynamicObjectAdapterBuilder.
You will get a FunctionalInterface for every Method declared in your interface, and an typed InvocationHandler and the Builder itself. 

To use the ***DynamicObjectAdapter*** you can do the following:

```java
    final Service service = ServiceAdapterBuilder
        .newBuilder()
        .setOriginal(new ServiceImpl())
        .withDoWork(new ServiceMethodDoWork() {
          @Override
          public String doWork(final String txt) {
            return "mocked";
          }
        })
        .withDoMoreWorkC(new ServiceMethodDoMoreWorkC() {
          @Override
          public String doMoreWorkC(final String txt) {
            return "mocked again";
          }
        })
        .buildForTarget(Service.class);

```

Also you can use the Lambda-Expressions for this. So your code will be shorter.
```java
    final Service serviceJDK8 = ServiceAdapterBuilder
        .newBuilder()
        .setOriginal(new ServiceImpl())
        .withDoWork(txt -> "mocked")
        .withDoMoreWorkC(txt -> "mocked again")
        .buildForTarget(Service.class);
``` 

You can use this for mocking too, if you want. For this you could adapt the methods you need and set the original to null.

```java
    final Service serviceJDK8Mock = ServiceAdapterBuilder
        .newBuilder()
        .setOriginal(null)
        .withDoWork(txt -> "mocked")
        .withDoMoreWorkC(txt -> "mocked again")
        .buildForTarget(Service.class);
```

If you can not use AnnotationProcessing, you are able to use the DynamicObjectAdapter itself.  

```java
    final ExtendedInvocationHandler<Service> extendedInvocationHandler
        = new ExtendedInvocationHandler<Service>() { };

    extendedInvocationHandler.addAdapter(new Object() {
      public String doMoreWorkB(String txt) {
        return "mocked";
      }
    });

    final AdapterBuilder<Service> adapterBuilder = new AdapterBuilder<Service>() {
      @Override
      protected ExtendedInvocationHandler<Service> getInvocationHandler() {
        return extendedInvocationHandler;
      }
    };

    final Service service = adapterBuilder.buildForTarget(Service.class);
```

Or if you want to write it even more compact..

```java
final Service service = new AdapterBuilder<Service>() {
      protected ExtendedInvocationHandler<Service> getInvocationHandler() {
        return new ExtendedInvocationHandler<Service>() {
          {
            addAdapter(new Object() {
              public String doMoreWorkB(String txt) {
                return "mocked";
              }
            });
          }
        };
      }
    }
    .buildForTarget(Service.class);
```

## StaticObjectAdapter
If you want, you can use the static ObjectAdapter nearly in the same way. But here you will get no Builder. You only will get the functional interfaces and the Adapter itself. Add the Annotation ***@StaticObjectAdapter*** to an Interface or Class.

```java
@StaticObjectAdapter
public interface Service {
  String doWork(String txt);
  String doMoreWorkA(String txt);
  String doMoreWorkB(String txt);
  String doMoreWorkC(String txt);
  String doMoreWorkD(String txt);
}
```

After a ***mvn clean compile*** you will get the Functionalinterfaces and the StaticObjectAdapter itself.


```java
final Service service = new ServiceStaticObjectAdapter()
        .withService(new ServiceImpl())
        .withServiceMethodDoMoreWorkC(new ServiceMethodDoMoreWorkC() {
          @Override
          public String doMoreWorkC(final String txt) {
            return "mocked";
          }
        })
        .withServiceMethodDoMoreWorkD(new ServiceMethodDoMoreWorkD() {
          @Override
          public String doMoreWorkD(final String txt) {
            return "mocked";
          }
        });

```

And here again, you can write it with short Lambda-syntax:

```java
final Service service = new ServiceStaticObjectAdapter()
        .withService(new ServiceImpl())
        .withServiceMethodDoMoreWorkC(txt -> "mocked")
        .withServiceMethodDoMoreWorkD(txt -> "mocked");
```
