---
layout : post
title : RocketMQ基础学习
category : Java
tags : RocketMQ 消息中间件
---

* content
{:toc}

　　在工作中刚刚接触到RocketMQ，学习了基础内容，整理总结记录下来，加深印象。
　　





### 特点

　　RcoketMQ 是阿里出品的一款低延迟、高可靠、可伸缩、易于使用的消息中间件。其特点很多，稍微列举几个：
　
  1. 支持发布/订阅和点对点的消息模型

  2. 能严格保证消息的先后顺序

  3. 支持pull（拉）和push（推）两种消息消费模式（push实际是对pull的封装）

  4. 因为是将消息存放在文件中，因此单一队列可存放百万消息（理论上无限）

  5. 分布式

  6. 支持多种消息协议

### 架构

　　这里先引用一张阿里中间件团队博客的图，如下：
![RocketMQ物理架构](http://img3.tbcdn.cn/5476e8b07b923/TB18GKUPXXXXXXRXFXXXXXXXXXX)

　　然后来介绍几个概念：

　　Producer：消息的生产者，负责把消息发送到Borker，同一类Producer称为Producer Group

　　Consumer：消息的消费者，负责消费push或者pull过来的消息，同一类被称为Consumer Group。（Group中每一个实例平均消费Broker中某个Topic的所有队列的消息，如果是广播模式，该Group中的每一个实例消费Broker中一个Topic下的所有队列）

　　Broker：可以理解为MQ，负责消息的接收、存储

　　Name Server：保存Broker的地址信息，为Producer和Consumer提供路由信息

　　Topic：消息的逻辑分类，Broker会依据消息的Topic将其放在对应的Topic的队列中

　　Message：消息的载体

　　Tag：消息的标记，可以理解为对Topic的细化，Consumer可以依据Tag对消息进行过滤

　　对上图进行解释：

　　Broker Master和Broker Slave组成了Broker的主从关系，Master与Slave会有信息同步，多个主从的Broker组成了Broker集群。Name Server也有自己的集群，但是所有其中的所有节点无任何信息同步，Broker集群中每一个Broker实例都与Name Server中的所有节点建立长连接，定时注册Topic信息到所有的Name Server中。

　　Producer要定期从Name Server中获取指定Topic路由信息，因为Name Server中所有节点都一样，所以随机与其中一个节点建立长连接就可以了。Producer根据获取的Topic路由信息与提供指定Topic服务的Broker Master建立长连接，定时向其发送心跳，将生产的消息发送给该Broker。

　　Consumer和Producer一样会去Name Server获取Topic路由信息，但是它可以同时与Broker Master和Slave建立连接，且定时发送心跳。Consumer可以从这两者任意一个订阅消息，规则由Broker配置决定。

### 基础示例

　　这里通过Producer和Consumer两个类简单地演示一下RockerMQ的用法。要成功执行下面的示例，需要先启动Name Server和Broker服务。这里就不叙述了，网上有很多示例。引用一个在windows环境下利用ide启动这两个服务的链接：[【RocketMQ原理解析1.1】整体介绍&IDE编译并启动RocketMQ的第一个例子](https://blog.csdn.net/a2888409/article/details/53781766)

#### 生产者Producer

　　直接看下面的代码：
```java
public class RocketMQProducer {

    public static void main(String[] args) throws Exception{
        int size = 100;
        String address = "192.168.0.100:9876";
        //声明一个producer，proGro为Group名称
        DefaultMQProducer producer = new DefaultMQProducer("proGro");
        //设置NameServer地址
        producer.setNamesrvAddr(address);
        producer.start();
        for (int i = 0; i < size; i++) {
            //声明消息载体，参数依次为topic、tag、content
            Message message = new Message("Topic1", "tag", ("message:" + i).getBytes());
            //发送消息
            SendResult sendResult = producer.send(message);
            System.out.println(sendResult);
        }
        System.out.println("消息发送完成");
        producer.shutdown();
    }
}
```

　　上面是一个非常简单的消息生产者的代码示例。

#### 消费者Consumer

　　Consumer有两种模式，一种是push，代码如下：
```java
public class RocketMQConsumer {

    public static void main(String[] args) throws MQClientException {
        //声明一个消费者，conGro为Consumer Group名称
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("conGro");
        //消费者设置NameServer地址
        consumer.setNamesrvAddr("192.168.0.100:9876");
        //设置消费起始位置
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_LAST_OFFSET);
        //设置订阅的topic以及过滤使用的子表达式
        consumer.subscribe("Topic1", "*");
        //设置消息监听
        consumer.setMessageListener((MessageListenerConcurrently)(message, context) -> {
            System.out.println(Thread.currentThread().getName() + " consumer message:" + message);
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });
        consumer.start();
        System.out.println("consumer begin consume");
    }
}
```

　　push模式实际是对pull模式的封装，在Consumer中要注册一个消息监听器，当队列中有新消息时，实际还是采用pull方法把消息拉取过来，让人感觉是Broker将消息推过来的。另一种pull模式，代码如下：

```java
public class RocketMQPullConsumer {

    private static final Map<MessageQueue, Long> offsetTable = new HashMap<>();

    public static void main(String[] args) throws Exception {

        DefaultMQPullConsumer consumer = new DefaultMQPullConsumer("conGro");
        consumer.setNamesrvAddr("192.168.0.100:9876");
        consumer.start();
        Set<MessageQueue> messageQueues = consumer.fetchSubscribeMessageQueues("Topic1");
        for (MessageQueue queue : messageQueues){
            SINGLE_MQ : while (true) {
                try {
                    PullResult result = consumer.pullBlockIfNotFound(queue, null, getMessageQueueOffset(queue), 32);
                    putMessageQueueOffset(queue, result.getNextBeginOffset());
                    switch (result.getPullStatus()) {
                        case FOUND:
                            //do something
                            break;
                        case NO_MATCHED_MSG:
                            break;
                        case NO_NEW_MSG:
                            break SINGLE_MQ;
                        case OFFSET_ILLEGAL:
                            break;
                        default:
                            break;
                    }
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        }
        consumer.shutdown();
    }

    private static void putMessageQueueOffset(MessageQueue queue, long offset){
        offsetTable.put(queue, offset);
    }

    private static long getMessageQueueOffset(MessageQueue queue){
        Long offset = offsetTable.get(queue);
        if (offset != null){
            return offset;
        }
        return 0;
    }
}
```

　　
　　因为push模式最终还是用了pull模式获取消息，所以这里只分析一下pull模式获取消息的大致过程，能力有限，只能做概况，错误地方还请见谅。

　　DefaultMQPullConsumer中的start方法最终调用的是DefaultMQPullConsumerImpl中的start方法。该方法先获取一个MQClientInstance示例，然后将group和consumer注册到该实例中，注册成功后，启动该实例，代码如下：
　　
```java
public void start() throws MQClientException {
        switch(this.serviceState) {
        case CREATE_JUST:
            this.serviceState = ServiceState.START_FAILED;
            this.checkConfig();
            this.copySubscription();
            if (this.defaultMQPullConsumer.getMessageModel() == MessageModel.CLUSTERING) {
                this.defaultMQPullConsumer.changeInstanceNameToPID();
            }

            this.mQClientFactory = MQClientManager.getInstance().getAndCreateMQClientInstance(this.defaultMQPullConsumer, this.rpcHook);
            this.rebalanceImpl.setConsumerGroup(this.defaultMQPullConsumer.getConsumerGroup());
            this.rebalanceImpl.setMessageModel(this.defaultMQPullConsumer.getMessageModel());
            this.rebalanceImpl.setAllocateMessageQueueStrategy(this.defaultMQPullConsumer.getAllocateMessageQueueStrategy());
            this.rebalanceImpl.setmQClientFactory(this.mQClientFactory);
            this.pullAPIWrapper = new PullAPIWrapper(this.mQClientFactory, this.defaultMQPullConsumer.getConsumerGroup(), this.isUnitMode());
            this.pullAPIWrapper.registerFilterMessageHook(this.filterMessageHookList);
            if (this.defaultMQPullConsumer.getOffsetStore() != null) {
                this.offsetStore = this.defaultMQPullConsumer.getOffsetStore();
            } else {
                switch(this.defaultMQPullConsumer.getMessageModel()) {
                case BROADCASTING:
                    this.offsetStore = new LocalFileOffsetStore(this.mQClientFactory, this.defaultMQPullConsumer.getConsumerGroup());
                    break;
                case CLUSTERING:
                    this.offsetStore = new RemoteBrokerOffsetStore(this.mQClientFactory, this.defaultMQPullConsumer.getConsumerGroup());
                }
            }

            this.offsetStore.load();
            boolean registerOK = this.mQClientFactory.registerConsumer(this.defaultMQPullConsumer.getConsumerGroup(), this);
            if (!registerOK) {
                this.serviceState = ServiceState.CREATE_JUST;
                throw new MQClientException("The consumer group[" + this.defaultMQPullConsumer.getConsumerGroup() + "] has been created before, specify another name please." + FAQUrl.suggestTodo("http://docs.aliyun.com/cn#/pub/ons/faq/exceptions&group_duplicate"), (Throwable)null);
            } else {
                this.mQClientFactory.start();
                this.log.info("the consumer [{}] start OK", this.defaultMQPullConsumer.getConsumerGroup());
                this.serviceState = ServiceState.RUNNING;
            }
        default:
            return;
        case RUNNING:
        case SHUTDOWN_ALREADY:
        case START_FAILED:
            throw new MQClientException("The PullConsumer service state not OK, maybe started once, " + this.serviceState + FAQUrl.suggestTodo("http://docs.aliyun.com/cn#/pub/ons/faq/exceptions&service_not_ok"), (Throwable)null);
        }
    }
```

　　启动客户端完成了以下几件主要的事情：
　
```java
public void start() throws MQClientException {
        synchronized(this) {
            switch(this.serviceState) {
            case CREATE_JUST:
                this.serviceState = ServiceState.START_FAILED;
                //获取NamesrvAddr
                if (null == this.clientConfig.getNamesrvAddr()) {
                    this.clientConfig.setNamesrvAddr(this.mQClientAPIImpl.fetchNameServerAddr());
                }
                //启动Netty通信服务
                this.mQClientAPIImpl.start();
                //启动各种定时任务
                this.startScheduledTask();
                //启动消息拉取服务
                this.pullMessageService.start();
                //启动负载均衡服务
                this.rebalanceService.start();
                this.defaultMQProducer.getDefaultMQProducerImpl().start(false);
                this.log.info("the client factory [{}] start OK", this.clientId);
                this.serviceState = ServiceState.RUNNING;
            case RUNNING:
            case SHUTDOWN_ALREADY:
            default:
                return;
            case START_FAILED:
                throw new MQClientException("The Factory object[" + this.getClientId() + "] has been created before, and failed.", (Throwable)null);
            }
        }
    }
```

　　以上的几步完成了很多重要的初始化工作，包括但不仅限于获取namesrv地址、更新路由、注册消费者信息、向broker发送心跳包等。这里忽略其他的内容，主要分析获取队列消息的代码实现。

　　PullMessageService继承了ServiceThread，其run方法如下：
　　
```java
public void run() {
        this.log.info(this.getServiceName() + " service started");

        while(!this.isStoped()) {
            try {
                PullRequest pullRequest = (PullRequest)this.pullRequestQueue.take();
                if (pullRequest != null) {
                    this.pullMessage(pullRequest);
                }
            } catch (InterruptedException var2) {
                ;
            } catch (Exception var3) {
                this.log.error("Pull Message Service Run Method exception", var3);
            }
        }

        this.log.info(this.getServiceName() + " service end");
    }
```

　　pullMessage方法实现如下：
　　
```java
private void pullMessage(PullRequest pullRequest) {
        MQConsumerInner consumer = this.mQClientFactory.selectConsumer(pullRequest.getConsumerGroup());
        if (consumer != null) {
            DefaultMQPushConsumerImpl impl = (DefaultMQPushConsumerImpl)consumer;
            impl.pullMessage(pullRequest);
        } else {
            this.log.warn("No matched consumer for the PullRequest {}, drop it", pullRequest);
        }

    }
```

　　因为DefaultMQPushConsumerImpl.pullMessage(PullRequest pullRequest)这个方法长到令人发指，所以我这里就不贴它了。里面的核心是通过pullAPIWrapper.pullKernelImpl方法经过MQClientAPIImpl.pullMessage，再通过pullMessageAsync或者pullMessageSync，再通过processPullResponse调用RemotingCommand.decodeCommandCustomHeader方法完成。里面经过了很多繁琐的步骤，完成了消费队列的负载均衡、发送Pull请求等操作。
　　
### 总结

　　RocketMQ的使用并不复杂，但其中的实现还是非常复杂的，有机会再多研究透彻一点吧。

### 参考


[RocketMQ 实战之快速入门](https://www.jianshu.com/p/824066d70da8)

[十分钟入门RocketMQ ](http://jm.taobao.org/2017/01/12/rocketmq-quick-start-in-10-minutes/)

[rocketmq--push消费过程](https://www.cnblogs.com/chenjunjie12321/p/7922362.html)

[RocketMQ源码分析----消费消息](https://blog.csdn.net/u013160932/article/details/59605583)

[rocketmq--消息的产生（普通消息）](https://www.cnblogs.com/chenjunjie12321/p/7728434.html)

[RocketMQ原理](https://blog.csdn.net/wuzhengfei1112/article/details/78076718)

[【RocketMQ原理解析2.1】源码目录结构介绍&Remoting通信层](https://blog.csdn.net/a2888409/article/details/53838580)

[RocketMQ 源码分析 —— 高可用](http://www.iocoder.cn/RocketMQ/high-availability/)

[分布式开放消息系统(RocketMQ)的原理与实践](https://www.jianshu.com/p/453c6e7ff81c)