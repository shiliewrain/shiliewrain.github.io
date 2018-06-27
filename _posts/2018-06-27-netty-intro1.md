---
layout : post
title : "Netty基础学习"
category : Netty
tags : socket 
---
* content
{:toc}

　　之前没有接触过关于网络编程的知识，学习Netty也是一脸懵，从最简单的应用开始整理记录，后面逐步加深理解。
　　
　　
　　
　　

### Netty介绍

> Netty是由[JBOSS](https://baike.baidu.com/item/JBOSS)提供的一个[java开源](https://baike.baidu.com/item/java%E5%BC%80%E6%BA%90)框架。Netty提供异步的、[事件驱动](https://baike.baidu.com/item/%E4%BA%8B%E4%BB%B6%E9%A9%B1%E5%8A%A8)的网络应用程序框架和工具，用以快速开发高性能、高可靠性的[网络服务器](https://baike.baidu.com/item/%E7%BD%91%E7%BB%9C%E6%9C%8D%E5%8A%A1%E5%99%A8)和客户端程序。 



　　简单的理解话，Netty是一个能帮你简化网络编程的框架。



### Socket编程

> 网络上的两个程序通过一个双向的通信连接实现数据的交换，这个连接的一端称为一个socket。
> 建立网络通信连接至少要一对端口号(socket)。socket本质是编程接口(API)，对TCP/IP的封装，TCP/IP也要提供可供程序员做网络开发所用的接口，这就是Socket编程接口；HTTP是轿车，提供了封装或者显示数据的具体形式；Socket是发动机，提供了网络通信的能力。

　　除了本科时利用Socket完成过一个聊天室，就再也没写过任何关于Socket的代码了。再写一遍最简单的，Socket服务器实现如下：

```java
public class SocketServer {

    public static void main(String[] args) throws IOException {
        //服务器监听端口
        ServerSocket serverSocket = new ServerSocket(1000);
        //打开一个连接，等待请求
        Socket socket = serverSocket.accept();
        //获取请求内容
        InputStream inputStream = socket.getInputStream();
        byte[] bytes = new byte[1024];
        int len;
        StringBuilder stringBuilder = new StringBuilder();
        while ((len = inputStream.read(bytes)) != -1){
            stringBuilder.append(new String(bytes, 0, len, "utf-8"));
        }
        System.out.println(stringBuilder.toString());
        inputStream.close();
        socket.close();
        serverSocket.close();
    }
}
```

　　Socket客户端实现如下：
　　
```java
public class SocketClient {

    public static void main(String[] args) throws IOException {
        //建立到服务器的socket连接
        Socket socket = new Socket("127.0.0.1", 1000);
        //获取socket输出流
        OutputStream outputStream = socket.getOutputStream();
        String message = "hello socket";
        outputStream.write(message.getBytes());
        outputStream.close();
        socket.close();
    }
}
```

　　以上就是最简单的socket编程实现。为了实现更丰富的功能，Socket编程也会更加的复杂。我目前不会，但是Netty会。

### Netty特点

　　Netty相比Tomcat，提供了更多的协议，甚至可以自定义协议。
　　
　　Netty的另外三个被提到的优点：
　　
　　1. 并发高：采用NIO模型，并发性能高
　　
　　2. 传输快：采用零拷贝
　　
　　3. 封装好：提供丰富的API

### Netty简单示例

　　Netty服务端实现如下：

```java
public class NettyServerTest {
    
    private int port;
    
    public NettyServerTest(int port){
        this.port = port;
    }

    public void start(){
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new NettyServerTestHandler());
                        }
                    })
                    .option(ChannelOption.SO_BACKLOG, 128)
                    .option(ChannelOption.SO_SNDBUF, 32*1024)
                    .option(ChannelOption.SO_RCVBUF, 32*1024)
                    .childOption(ChannelOption.SO_KEEPALIVE, true);
            ChannelFuture f = bootstrap.bind(port).sync();
            f.channel().closeFuture().sync();
        } catch (Exception e){
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        new NettyServerTest(1234).start();
    }
}
```

　　上面的代码中，NioEventLoopGroup是一个处理I / O操作的多线程事件循环。上面声明了两个，是因为这是Netty的默认线程模型。[参考这篇文章](http://www.importnew.com/15656.html)bossGroup接收到连接后，就将连接注册到workGroup，由workGroup负责处理。ServerBootstrap是一个工具类，用于设置服务器。然后是按照NioServerSocketChannel来实例化新的Channel去接收连接。再后面就是利用ChannelInitializer去自定义初始化的channel，里面实现了一个NettyServerTestHandler类，代码如下：
　　
```java
public class NettyServerTestHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext context, Object msg){
        try {
            ByteBuf buf = (ByteBuf) msg;
            while (buf.isReadable()) {
                System.out.print((char) buf.readByte());
                System.out.flush();
            }
        } finally {
            ReferenceCountUtil.release(msg);
        }
    }
}
```

　　再来看看客户端的代码：
　　
```java
public class ClientTest {

    public static void main(String[] args) throws InterruptedException {
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        Bootstrap bootstrap = new Bootstrap();
        bootstrap.group(workerGroup)
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                        socketChannel.pipeline().addLast(new Clienthandler());
                    }
                });
        ChannelFuture future = bootstrap.connect("127.0.0.1", 1234).sync();
        future.channel().writeAndFlush(Unpooled.copiedBuffer("Shiliew".getBytes()));
        future.channel().closeFuture().sync();
        workerGroup.shutdownGracefully();
    }
}
```

　　客户端的实现和服务端非常类似，如果有消息被写回到客户端，那么也会执行Clienthandler。

　　以上的代码中有一些概念需要介绍一下：

　　channel：表示一个连接，可以理解为每一个连接就是一个channel

　　channelHandler：处理连接请求

　　ChannelHandlerContext：用于传输业务数据

　　ChannelPipeline：像管道一样，保存需要的channelHandler和ChannelHandlerContext
　　
### 参考

[Netty入门教程——认识Netty](https://www.jianshu.com/p/b9f3f6a16911)

[Netty从零开始（一）](https://blog.csdn.net/qq_23660243/article/details/69258687)

[Netty实现原理浅析](http://www.importnew.com/15656.html)

[【总结】Netty（RPC高性能之道）原理剖析](https://blog.csdn.net/zhiguozhu/article/details/50517551)