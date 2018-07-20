---
layout : post
title : "zipkin与dubbo整合"
category : zipkin
tags : zipkin dubbo 调用链
---
* content
{:toc}

　　利用zipkin可以对dubbo进行调用链监控，可以查到调用链中的dubbo服务的性能，并且dubbo提供了SPI的接口，能很容易完成并自定义相应的filter去监控dubbo服务。




### 整合

　　简单的描述一下，同ServletFilter一样，在dubbo中利用Filter过滤请求，传递TraceId等参数，生成相应的span传递给zipkin服务器。

　　引入brave-instrumentation-dubbo-rpc包，这是一个SPI的Filter包。

```xml
<dependency>
    <groupId>io.zipkin.brave</groupId>
    <artifactId>brave-instrumentation-dubbo-rpc</artifactId>
    <version>5.1.2</version>
</dependency>
```

　　里面有一个TracingFilter的类，在其invoke方法中实现了对span的一些操作。

```java
@Override
  public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
    if (tracer == null) return invoker.invoke(invocation);

    RpcContext rpcContext = RpcContext.getContext();
    Kind kind = rpcContext.isProviderSide() ? Kind.SERVER : Kind.CLIENT;
    final Span span;
    if (kind.equals(Kind.CLIENT)) {
      span = tracer.nextSpan();
      injector.inject(span.context(), invocation.getAttachments());
    } else {
      TraceContextOrSamplingFlags extracted = extractor.extract(invocation.getAttachments());
      span = extracted.context() != null
          ? tracer.joinSpan(extracted.context())
          : tracer.nextSpan(extracted);
    }

    if (!span.isNoop()) {
      span.kind(kind).start();
      String service = invoker.getInterface().getSimpleName();
      String method = RpcUtils.getMethodName(invocation);

      span.kind(kind);
      span.name(service + "/" + method);

      InetSocketAddress remoteAddress = rpcContext.getRemoteAddress();
      Endpoint.Builder remoteEndpoint = Endpoint.newBuilder().port(remoteAddress.getPort());
      if (!remoteEndpoint.parseIp(remoteAddress.getAddress())) {
        remoteEndpoint.parseIp(remoteAddress.getHostName());
      }
      span.remoteEndpoint(remoteEndpoint.build());
    }

    boolean isOneway = false, deferFinish = false;
    try (Tracer.SpanInScope scope = tracer.withSpanInScope(span)) {
      Result result = invoker.invoke(invocation);
      if (result.hasException()) {
        onError(result.getException(), span);
      }
      isOneway = RpcUtils.isOneway(invoker.getUrl(), invocation);
      Future<Object> future = rpcContext.getFuture(); // the case on async client invocation
      if (future instanceof FutureAdapter) {
        deferFinish = true;
        ((FutureAdapter) future).getFuture().setCallback(new FinishSpanCallback(span));
      }
      return result;
    } catch (Error | RuntimeException e) {
      onError(e, span);
      throw e;
    } finally {
      if (isOneway) {
        span.flush();
      } else if (!deferFinish) {
        span.finish();
      }
    }
  }
```

　　该Filter是以SPI的方式引入dubbo的，在默认文件/META-INF/dubbo/com.alibaba.dubbo.rpc.Filter中，有这样的声明：
```
tracing=brave.dubbo.rpc.TracingFilter
```

　　然后在dubbo的配置中引入该Filter即可。

```xml
    <!-- 服务提供方filter -->
    <dubbo:provider filter="tracing"/>
    <!-- 服务消费方filter -->
    <dubbo:consumer filter="tracing"/>
```

　　然后还需要为该Filter注入一个Tracing实例，并且该实例名必须为tracing。

```java
@Configuration
public class ZipkinConfiguration {

    @Autowired
    private ZipkinProperties properties;

    @Bean
    public Tracing tracing(){

        Sender sender = OkHttpSender.create(properties.getUrl());

        AsyncReporter reporter = AsyncReporter.builder(sender)
                .closeTimeout(properties.getConnectTimeout(), TimeUnit.MILLISECONDS)
                .messageTimeout(properties.getReadTimeout(), TimeUnit.MILLISECONDS)
                .build();

        Tracing tracing = Tracing.newBuilder()
                .localServiceName(properties.getServiceName())
                .propagationFactory(ExtraFieldPropagation.newFactory(B3Propagation.FACTORY, "shiliew"))
                .sampler(Sampler.ALWAYS_SAMPLE)
                .spanReporter(reporter)
                .build();
        return tracing;
    }
}
```

　　Tracing类的主要作用是针对span的操作做一些配置，并设置上传zipkin服务器。

### 效果

　　在这里不演示如何配置dubbo服务，之前的博文有相关介绍。效果图如下：

![zipkin-dubbo1](https://raw.githubusercontent.com/shiliewrain/shiliewrain.github.io/master/img/zipkin-dubbo1.png)

### 问题

　　在整合zipkin和dubbo的过程中，碰到了一些问题，在这里记录一下。

#### 启动了两个服务提供方，使用了相同的端口号，报错

　　项目中有一个服务提供方moduleA，一个服务提供方和消费方moduleB。先启动了moduleA，然后启动moduleB，启动报错，显示端口已被占用。两个module的配置端口一样：

```xml
<!-- 协议和端口号 -->
<dubbo:protocol name="dubbo" port="20880"/>
```

　　每个dubbo服务都有自己的端口，修改即可。

#### zipkin只记录了server span，无client span

　　在调用了dubbo服务之后，发现zipkin服务器上没有收到client的span，猜想问题出现在filter上面。断点调试，发现Consumer调用服务时，filter并没有执行，而Provider接收请求时有执行filter。经过排查，发现问题出现在dubbo配置上，最开始没有配置consumer方的过滤器。

```xml
    <!-- 服务提供方filter -->
    <dubbo:provider filter="tracing"/>
    <!-- 服务消费方filter -->
    <dubbo:consumer filter="tracing"/>
```

　　添加上去即可。

### 参考

[阿里dubbo服务注册原理解析](https://www.cnblogs.com/linlinismine/p/7814521.html)

[dubbo+zipkin调用链监控](https://www.cnblogs.com/ASPNET2008/p/6709900.html)

[基于zipkin和brave完成对dubbo的链路追踪](https://blog.csdn.net/will0532/article/details/78552751)