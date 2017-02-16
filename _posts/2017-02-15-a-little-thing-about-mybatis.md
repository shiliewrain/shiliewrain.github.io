---
layout: post
title: "关于Mybatis的一点笔记"
categories: 持久层框架
tags: 后台 框架 持久层
---
* content
{:toc}

## 背景

　　今天上班由“CJ”转“70”的一个方法看到了缓存，具体的上下文就不说那么多了。公司的项目中，我暂时了解到的缓存有两种方式，一个是保存在本地内存，另外一个是保存在大缓存平台。反正我现在都不太了解，于是就提出了自己的第一个需要了解的地方——缓存。
说起缓存，研究生期间也用hibernate搭建过SSH框架，但是当时只是为了会用，也没太管一级缓存和二级缓存，反正按照网上的配置搬，框架能跑起来，并且相关的CURD操作能成功执行就够了。现在只能说出来混，迟早是要还的。言归正传，今天我学习的东西是以缓存为出发点的，由缓存想到了hibernate、持久化、持久化框架、Mybatis、Mybatis一级缓存。这是今天主要的学习路线。只是由hibernate想到了持久化，所以没有去查找hibernate的相关资料，就暂不做记录了。






## 持久化

　　以前一直不太理解持久化，以为它表达的意思是持久化框架将某一个对象对应的数据库里的数据保存在持久化框架中，甚至以为是存在一级或者二级缓存中。今天请教了呈呈大神之后，才豁然开朗。持久化是指将数据保存到可永久保存的存储设备中，如将内存中的对象存储到关系型数据库中，或者XML文件中，或者其它什么。相比它的实际含义，真对不起它高大上的名称。类比来说，持久化就是将你头脑中的想法写在纸上，于是想法持久化了。具体来说，持久化就是将一个对象由内存插入到数据库的表中，数据表里有了这个对象的所有属性值，则这个对象就被持久化了。（被这三个字蒙骗这么久，我真疑惑自己是不是入错行了...）

## 持久化框架

　　能力有限，只能说说我仅仅认识的Java持久化框架。应该有不少，但我知道的就hibernate和Mybatis，有使用过hibernate，但原理不清楚，Mybatis没自己弄过。但今天粗略看了一下，感觉这两个持久化框架大体上是相似的。因为了解还不够，所以就只记录这一点点：持久层框架就是为了操作数据库，所以都是封装JDBC，提供CURD操作。

### JDBC连接数据库

>　　JDBC（Java Data Base Connectivity,java数据库连接）是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。

　　JDBC连接数据库步骤，以MySql为例：
  
　　1.　加载数据库驱动：`Class.forName("com.mysql.jdbc.Driver");`

　　2.　建立数据库连接：`Connection con=DriverManager.getConnection(url,uname,pword);`

　　3.　创建Statement：`Statement st=con.createStatement();`

　　4.　执行sql语句并返回结果集：`ResultSet rs=st.execute(sql);`
  
　　大致上应该就这些步骤，先大概记录一下。

## Mybatis

　　网上有特别多的简介，我就懒得搬了。刚发现一个讲解Mybatis的专栏--[传送门](http://blog.csdn.net/column/details/mybatis-principle.html)
　　
　　在某篇文章上截了个图[传送门](http://blog.csdn.net/hupanfeng/article/details/9068003/)，介绍了Mybatis的整体流程图，如下：

![Mybatis process](http://img.my.csdn.net/uploads/201306/09/1370783456_4126.JPG)

　　大概说来，就是SqlSessionFactoryBuilder创建Configuration对象，然后创建SqlSession工厂，新建一个sqlSession对象，里面会包含一个Executor对象，该对象调用StatementHandler就可以调用JDBC的接口操作数据库了。

### Mybatis一级缓存

　　关于Mybatis一级缓存，我读的是这篇博客[传送门](http://blog.csdn.net/luanlouis/article/details/41280959)

　　看过之后确实对一些概念有了些认识，可能理解不够深刻。把自己认为的重点记下来。
　
* Mybatis和数据库开启一次对话就会创建一个SqlSession对象，该对象中包含一个Executor执行器，该执行器中存在并维护一个Cache对象。
* Mybatis一级缓存只会涉及perpetualCache这一个Cache接口的实现类，其实现原理就是通过HashMap保存缓存信息。
* 生命周期：（1）会话开始时，会有新的SqlSession、Executor和PerpetualCache，会话结束则全部释放；（2）SqlSession调用close()方法，会释放PerpetualCache对象；（3）update操作会清空PerpetualCache。
* 工作流程：查询操作会访问PerpetualCache，当返回为空时，才会访问数据库，并在缓存之后返回结果。
* CacheKey：statementId + rowBounds + sql + JDBC所需要的参数

