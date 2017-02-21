---
layout : post
title : "HTTP、TCP与UDP的区别"
categories : 网络基础
tags : 网络协议 TCP/IP
---
* content
{:toc}

　　今天看了一篇打开了很久的博文，讲的内容就是HTTP、TCP与UDP的区别，比较基础，正好可以记录一下。将其中的内容总结一下，自己以前也没有好好地学习网络基础，由浅入深，重新学习吧。






## TCP/IP

　　TCP/IP是一个协议栈，参考模型一般为四层结构：应用层、传输层、网络层和网络接口层。也有五层结构的，将网络接口层分为链路层和物理层。OSI参考模型为七层结构：应用层、表示层、会话层、传输层、网络层、链路层和物理层。参考图如下：

![TCP/IP模型对比](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1488285794&di=9dca6451181f13be5e24766d89131314&imgtype=jpg&er=1&src=http%3A%2F%2Fimg694.ph.126.net%2F-cQJOt4nkmF-U2LrVzv74g%3D%3D%2F1151232654747720867.jpg)

　　在TCP/IP协议栈中，应用层常见的协议有：FTP、HTTP；传输层协议有：TCP和UDP；网络层协议有：IP、ARP等。参考图如下：
![协议栈](http://blog.chinaunix.net/attachment/201304/27/26833883_1367053079KNJe.png)

## TCP

　　TCP是基于连接的协议，在发送消息之前，会先经过三次握手建立连接，只要客户端或者服务器没有主动断开，连接就一直会存在，称为长连接。TCP协议传输数据无大小限制，保证数据可靠性和顺序。

## UDP

　　UDP是基于无连接的协议，不建立连接，直接将数据发送出去。因此UDP不能保证数据的可靠性和顺序，适用于少量数据的发送，一般数据包限定在64KB之内。

## HTTP

　　HTTP是基于TCP的协议，发送请求时建立TCP连接，请求结束后，断开连接，被称为短连接。HTTP是无状态的，这表示客户端和服务器互相都不知道对方的状态，比如，在无cookie和Session的情况下，服务器不知道客户端是否完成了登陆。


## 参考
[传送门一](http://blog.chinaunix.net/uid-26833883-id-3627644.html)
[传送门二](http://www.cnblogs.com/BlueTzar/articles/811160.html)
[传送门三](http://www.jianshu.com/p/42260a2575f8)
[传送门四](http://www.cnblogs.com/xhwy/archive/2012/03/03/2378293.html)