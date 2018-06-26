---
layout : post
title : RocketMQ基础学习
category : Java
tags : RocketMQ 消息中间件
---

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

　　另一种是pull模式，代码如下：

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



### 参考



[RocketMQ 实战之快速入门](https://www.jianshu.com/p/824066d70da8)

[十分钟入门RocketMQ ](http://jm.taobao.org/2017/01/12/rocketmq-quick-start-in-10-minutes/)