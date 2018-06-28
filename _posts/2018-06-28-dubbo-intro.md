---
layout : post
title : "Dubbo基础学习"
category : RPC框架
tags : Dubbo RPC
---
* content
{:toc}

　　Dubbo是一个RPC框架，主要作用于服务治理。






### 介绍

　　Dubbo是一个分布式服务框架，致力于提供高性能和透明化的RPC远程服务调用方案，以及SOA服务治理方案。Dubbo是RPC框架，帮助用户便捷地调用远程服务。

#### RPC

　　RPC（Remote Procedure Call），远程过程调用，它定义了一种无需了解底层网络技术就通过网络向远端服务器请求服务的协议。说白一点，A、B两台服务器通过网络连接，A服务器上面部署的应用a想要调用部署在B服务器上的应用b提供的服务。因为不能直接调用，需要采用一致的协议通过网络来进行数据传输。而RPC框架，就帮你做了这个事，让你有在A服务器上直接调用b应用的错觉。盗个图：

![RPC调用流程](https://pic4.zhimg.com/80/45366c44f775abfd0ac3b43bccc1abc3_hd.jpg)

　　RPC的调用过程大致如下：
　　
　　1. Client执行调用，传递希望调用的方法和参数

　　2. 通过Client Stub进行封装，传递给本地系统

　　3. 本地系统通过网络发送请求

　　4. 服务器收到请求，将请求转给Server Stub

　　5. Server Stub将数据解封，传递给Server

　　6. Server执行方法调用后，将结果传递给Server Stub

　　7. Server Stub将结果封装传递给本地系统

　　8. 本地系统通过网络将数据传输到客户端

　　9. 客户端收到回复，将数据返回给Client Stub

　　10. Client Stub将结果解封后，传递给Client

　　一个RPC框架就是将Server和Client以下的部分全部封装起来，让用户感觉不到上面整个过程的存在。

#### RMI

　　RMI（Remote Method Invocation），远程方法调用，感觉它和上面的RPC很类似。其实可以把RMI理解为RPC的面向对象版本，而且RMI也是Java的一组拥护开发分布式应用程序的API。

　　RMI与RPC的一点区别就是RMI是通过客户端的stub对象作为远程接口进行远程方法调用，而RPC是通过网络服务协议向服务器发送请求，请求的内容为希望调用的方法和参数。另外RMI只用于Java，而RPC与语言无关。

### Dubbo架构

　　盗用一张Dubbo的架构图：

![dubbo架构](http://dubbo.apache.org/books/dubbo-user-book/sources/images/dubbo-architecture.jpg)

　　从上图可以简单看出，Consumer和Provider都在Register中注册自己，Provider提供自己的服务，Consumer订阅需要的服务。而Monitor是监控中心，为运维提供一系列数据展示。

### Dubbo示例

　　做了一个简单的示例，分为三个部分：通用接口、服务提供方和服务消费方。

#### 通用接口

　　通用接口为消费方和提供方提供了调用方法，消费方通过通用接口可以如调用本地方法一样便捷。代码如下：
　　
```java
public interface DemoService {

    String sayHello(String name);
}
```

　　将这个类打成jar包，然后服务提供方去实现具体的方法，服务消费方只需要调用其方法就OK。

#### 服务提供方

　　服务提供方首先要实现通用接口，代码如下：

```java
public class DemoServiceImpl implements DemoService {
    @Override
    public String sayHello(String name) {
        return "Hello " + name;
    }
}
```

　　然后启动服务，代码如下：

```java
public class DemoApplication {
    public static void main(String[] args) throws IOException {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:dubboDemo.xml");
        context.start();
        //保持服务器运行，按任意键结束
        System.in.read();
    }
}
```

　　那么dubboDemo.xml的中的内容就是注册服务的核心内容了，配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <!-- 应用名称 -->
    <dubbo:application name="demo-provider"/>
    <!-- 使用multicast广播注册中心暴露服务地址 -->
    <dubbo:registry address="multicast://224.5.6.7:1234"/>
    <!-- 协议和端口号 -->
    <dubbo:protocol name="dubbo" port="20880"/>
    <!-- 暴露的服务，引用本地的具体实现 -->
    <dubbo:service interface="com.example.demo.dubbo.DemoService" ref="demoService"/>
    <!-- 服务的具体实现 -->
    <bean id="demoService" class="com.example.demo.dubbo.producer.DemoServiceImpl"/>
</beans>
```

#### 服务消费方

　　服务消费方可以也应该与服务提供方在不同的项目中，只需要引用通用接口就可以了。代码如下：

```java
public class DemoApplication {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:dubboConsumer.xml");
        context.start();
        DemoService demoService = (DemoService) context.getBean("demoService");
        String s = demoService.sayHello("dubbo test");
        System.out.println(s);
    }
}
```

　　再来看看dubboConsumer.xml，配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <!-- 消费方应用名称 -->
    <dubbo:application name="demo-consumer"/>
    <!-- 使用multicast广播注册中心暴露发现服务地址 -->
    <dubbo:registry address="multicast://224.5.6.7:1234"/>
    <!-- 需要使用的服务 -->
    <dubbo:reference id="demoService" interface="com.example.demo.dubbo.DemoService"/>
</beans>
```

　　先启动服务提供方应用，然后启动服务消费方应用，就可以看到消费方如同调用本地方法一样调用了服务方提供的方法。

### 总结

　　以上内容只是Dubbo的一个基础入门，Dubbo主要是针对服务治理的分布式RPC框架。在后面的学习中，我也会继续总结记录。

### 参考

[Dubbo官方文档](http://dubbo.apache.org/books/dubbo-user-book/)

[Dubbo是什么？能做什么？](https://blog.csdn.net/houshaolin/article/details/76408399)

[聊聊RPC及其原理](https://www.cnblogs.com/huangqingshi/p/7803642.html)

[什么是 RPC 框架](https://blog.csdn.net/b1303110335/article/details/79557292)

[Java RMI与RPC的区别](https://www.cnblogs.com/ygj0930/p/6542811.html)

[从懵逼到恍然大悟之Java中RMI的使用](https://blog.csdn.net/lmy86263/article/details/72594760)