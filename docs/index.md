# Welcome to ProxyBuilder

For full documentation visit [mkdocs.org](http://mkdocs.org).

Abstracts for the Conf.

#IoT-Workshop
## Industrial Prototyping with Java 8 and TinkerForge
TinkerForge is a very simple, easy-to-handle modular electronic system. Depending on the application, a system of sensor, wireless and motor control elements can be built in a modular way and programmed with a few lines of code. No soldering iron is needed. Both the hardware and the software are open source. After a brief overview of the available sensors and actuators, we will quickly start with the actual programming. Together, we will explore different applications, architectures, and the combination of TinkerForge with Java. Using sensors, actuators and some other elements, we will take the first steps and delve into the practical world of IoT with Java 8 and TinkerForge. 

What do I need to bring?

* A laptop on which you can install software
* Java 8 (JDK) installed and running
* A JDK 8 capable IDE

Optionally, you can of course also bring your own Raspberry Pi or the like.

Goals:

* addressing sensors, storing data and display them using JavaFX.
* (opt) parallel processing of sensor data streams using Java 8 Streams
* using touch elements



```java

@Test
  public void test004() throws Exception {
    final ServiceHandle<BusinessService> serviceHandle = locator.getServiceHandle(BusinessService.class);
    final BusinessService myService = serviceHandle.getService();
    Assert.assertNotNull(myService);

    Assert.assertEquals(myService.getClass(), BusinessServiceImplA.class);
    System.out.println("myService.doWork(\"h\") = " + myService.doWork("h"));

    final Object serviceData = serviceHandle.getServiceData();
    System.out.println("serviceData = " + serviceData);


  }
```





#Talks
## Proxy Deep Dive
At this talk we will start from the basics and come shortly to Dynamic-/Static-Proxies, generated typesave DynamicObjectAdapter, ProxyBuilder and more. How we could combine this with with an other language like kotlin? We will have a deep dive to this pattern group , and I am sure you will like it ;-) 

This talk is an extension of the german book "Dynamic Proxies" written from Dr. Heinz Kabutz and me.

## From jUnit to Mutation Testing
JUnit is a well-known tool for java developers in the area of TDD. Here it has become accepted, that CodeCoverage can be measured. In this case we distinguish between coverage on the level of classes, methods and rows. The goal is to get the CodeCoverage as high as possible on the row level, but not higher than necessary.
What exactly does it mean? A CodeCoverage of appr. 75% on the row level is very good and can already provide a basis. But what does this figure say?
 
In this talk we will deal with the term Mutation Testing and show the practical ways of use. How can the coverage be defined and what can be achieved?
How can it be integrated into an existing project and what should be considered while running a test?

## DI and CDI frameworks internals
Dependency Injection is now part of nearly every Java project. But what is the difference between DI and CDI. How to decide what I could use better, what  frameworks are available and what are the differences for me as a programmer? What could be an option for the IoT-, Desktop- or Webproject?

In this talk we will get an overview over different frameworks and how they are working. We are not checking the well known big one only, but we are looking at some small sometimes specialized implementations.
