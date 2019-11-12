## Interview questions
What is Dubbo's spi thought?

## Psychnological analysis of interviewers
Continue to ask questions, some basic things in front of the question, make sure you should be ok, understand some basic things of dubbo, then ask a little more difficult problem, is spi, ask you first spi? Then ask how is the spi of dubbo implemented?

In fact, it is to see how you master the dubbo.

## Analysis of interview questions
### What is spi?
Spi, in a nutshell, is `service provider interface`. What does it mean to say it, for example, if you have an interface, now this interface has 3 implementation classes, so which implementation class is selected for this interface when the systen is running? This requires spi, need to **according to the specified configuration** or **default configuration**, go to **find the corresponding implementation class** loaded, and then use this instance object of the implementation class.

Give a chestnut.

You have an interface A. A1/A2/A3 are different implementations of interface A, respectively. By configuring `interface A=to implement A2`, then when the system is actually running, it will load your configuration and instantiate an object with A2 to provide the service.

Where is the spi mechanism generally used? **Scenario extension scenario**, for example, you have developed an open source framework for others. If you want someone to write a plugin and plug it into your open source framework to extends a feature, this time spi thought just used it.

### The embodiment of Java spi though
The classic idea of spi is reflected in everyone's use, such as jdbc.

Java defines a set of jdbc interfaces, but Java  does not provide an implementation class for jdbc.

But what implementation classes do you want to use the jdbc interface when the project is running? In general, we want to **according to the database you use**, such as mysql, you will introduce `mysql-jdbc-connector.jar`, oracle, you will introduce `oracle-jdbc-connector.jar`.

在系统跑的时候，碰到你使用 jdbc 的接口，他会在底层使用你引入的那个 jar 中提供的实现类。

### Dubbo's spi thought
Dubbo also uses the spi idea, but does not use jdk's spi mechanism, which is a set of spi mechanism implemented by itself.
```java
Protocol protocol = ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension();
```

The Protocol interface, when the system is running, dubbo will determine which implementation class of the Protocol interface should be used to instantiate the object for use.

It will find a Protocol you configured, load the Protocol implementation class you configured into jvm, and then instantiate the object, you can use your Protocol implementation class.


The above line of code is used extensively in dubbo, that is, for many components, one interface and multiple implementations are reserved, and then the corresponding implementation class is dynamically found according to the configuration when the system is running. If you don't configure it, then the default implementation is fine, no problem.
```java
@SPI("dubbo")  
public interface Protocol {  
      
    int getDefaultPort();  
  
    @Adaptive  
    <T> Exporter<T> export(Invoker<T> invoker) throws RpcException;  
  
    @Adaptive  
    <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException;  

    void destroy();  
  
}  
```

In dubbo's own jar, in the `/META_INF/dubbo/internal/com.alibaba.dubbo.rpc.Protocol` file.
```xml
dubbo=com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol
http=com.alibaba.dubbo.rpc.protocol.http.HttpProtocol
hessian=com.alibaba.dubbo.rpc.protocol.hessian.HessianProtocol
```

So, this shows how the dubbo spi mechanism defaults to play,  in fact, is the Protocal interface, `@SPI("dubbo")`, that is, the implementation class is provided through the SPI mechanism, and the implementation class is through dubbo. As the default key to find in the configuration file, the configuration file name is the same as the interface fully qualified name. The default implementation class is `com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol` by using dubbo as the key.

If you want to dynamically replace the default implementation class, you need to use the `@Adaptive` interface. In the Protocol interface, there are two methods with the `@Adaptive` annotation, which means that the two interfaces will be implemented by the proxy.

What do you mean?

For example, the Protocol interface has two `@Adaptive` annotation annotation methods. when running ,it will generate a proxy class for the Protocol. The two methods of the proxy class will have proxy code. The proxy code will be dynamically based on the url when running. In the protocol to get that key, the default is dubbo, you can also specify it youself, if you specify another key, then you will get an instance of another implementation class.

### How to extends the components in dubbo yourself?
Let's talk about how to extend the components in dubbo yourself.

Write a project yourself, if it can be labeled as a jar package, inside the `src/main/resources` directory, make a `METE-INF/services`, which puts a file called: `com.alibaba.dubbo.rpc.Protocol`, make a `my=com.bingo.MyProtocol` in the file. Get the jar yourself into the nexus private service.

Then do a `dubbo provider` project yourself, rely on the jar you made yourself in this project, and then give a configuration in the spring configuration file:

```xml
<dubbo:protocol name=”my” port=”20000” />
```
When the provider starts, it will be loaded into the `my=com.bingo.MyProtocol` line in our jar package. The you will use your defined MyProtocol according to your configuration. This is a simple expanation, you pass in the above way, you can replace a lot of dubbo internal components, just throw your own jar package, and then configure it.

![dubbo-spi](/images/dubbo-spi.png)

Dubbo provides a lot of extension points similar to the above, that is, if you want to expand a thing, just write a jar yourself, let your consumer or provider project, rely on your jar, specify the directory in your jar. Configure the file corresponding to the interface name, which is implemented by `key=implementation clas`.

Then for the corresponding component, like `<dubbo:protocol>` to implement an interface with the implementation class of your key, you can extend the various functions of dubbo and provide your own implementation.