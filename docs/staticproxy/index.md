# Static Proxies
Compared to the DynamicProxies we are now generating static proxies. This could be done with 
***AnnotationProcessing*** during the compile process or at runtime.

## Generated
Using ***AnnotationProcessing*** to generate the Proxies will give you the possibility to get 
proxies without the overhead of Reflection. On the other side you will generate maybe a lot of code, that must be compiled.

## Runtime Generated
Compared to the way of using ***AnnotationProcessing*** this one will create and compile at runtime the static proxies. This
is possible if you have access to the Compiler (tools.jar) during runtime and will mostly lead you to use Unsafe to put your new classes to the SystemClassLoader.

## Static Generated VirtualProxy
With the Annotation ***@StaticVirtualProxy*** you can generate Static - VirtualProxies during the clean compile process. This Annotation have a parameter called ***strategy***. The Strategy is used to define the right time to call the ***InstanceFactory***
See [DynamicProxies - CreationStrategies](/dynamicproxy/#creationstrategies)


## Static Generated MetricsProxy
With the static ***MetricsProxy*** we will get generated MetricsProxies that are using Dropwizard-Metrics to measure the 
usage of the methods. The generated static MetricsProxies are generating Methods for all declared Methods in the inheritance including ***hashCode()*** and ***equals()***. You can use the Annotation  ***@StaticMetricsProxy*** in combination with interfaces and classes.

There is one difference between the generated classes based on interfaces and classes. If you annotate an interface you will only get Metrics for the declared Methods, not for ***hashCode*** and ***equals*** if it is not explicitly declared. But for annotated classes you will get the Metrics for methods from Object, too.

```java
@StaticMetricsProxy
public interface Service {
  String doWork(String txt);

  String doMoreWorkA(String txt);

  String doMoreWorkB(String txt);

  String doMoreWorkC(String txt);

  String doMoreWorkD(String txt);
}
```

You can use the MetricsProxy easily by creating an instance and setting the Delegator. After this
every Methodcall will be counted by DropwizardMetrics. For every Method, you will get a separate Histogram, named with the full Classname and Methodname. In my case you will get a Histogram with the name
***org.rapidpm.demo.proxybuilder.staticproxy.v002.Service.doMoreWorkC***


```java
    final ServiceStaticMetricsProxy proxy = new ServiceStaticMetricsProxy();
    proxy.withDelegator(new ServiceImpl());

    final Service service = proxy;

    RapidPMMetricsRegistry.getInstance().startConsoleReporter();
    try (final IntStream intStream = IntStream.range(0, 10_000_000)) {
      intStream
          .onClose(() -> out.println("Stream will be closed now..."))

          .forEach(i -> service.doMoreWorkC("aaahhhhhh " + i));
    }

    RapidPMMetricsRegistry
        .getInstance()
        .getMetrics()
        .getHistograms()
        .forEach((s, histogram) -> {
          out.println("s = " + s);
          out.println("histogram - get999thPercentile= " + histogram.getSnapshot().get999thPercentile());
        });
```



## Static Runtime VirtualProxy
partly implemented until now.. stay tuned

## Static Runtime MetricsProxy
partly implemented until now.. stay tuned
