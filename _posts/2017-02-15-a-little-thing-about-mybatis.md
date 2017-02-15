---
layout:post
title:关于Mybatis的一点笔记
categories:持久层框架
tags:
- 后台
- 框架
- 持久层
description:记录刚开始了解到的一点点Mybatis的内容
---
* content
{:toc}
## 背景

	今天上班由“CJ”转“70”的一个方法看到了缓存，具体的上下文就不说那么多了。公司的项目中，我暂时了解到的缓存有两种方式，一个是保存在本地内存，另外一个是保存在大缓存平台。反正我现在都不太了解，于是就提出了自己的第一个需要了解的地方————缓存。
	说起缓存，研究生期间也用hibernate搭建过SSH框架，但是当时只是为了会用，也没太管一级缓存和二级缓存，反正按照网上的配置搬，框架能跑起来，并且相关的CURD操作能成功执行就够了。现在只能说出来混，迟早是要还的。言归正传，今天我学习的东西是以缓存为出发点的，由缓存想到了hibernate、持久化、持久化框架、Mybatis、Mybatis一级缓存。这是今天主要的学习路线。只是由hibernate想到了持久化，所以没有去查找hibernate的相关资料，就暂不做记录了。

## 持久化

	以前一直不太理解持久化，以为它表达的意思是持久化框架将某一个对象对应的数据库里的数据保存在持久化框架中，甚至以为是存在一级或者二级缓存中。今天请教了呈呈大神之后，才豁然开朗。持久化是指将数据保存到可永久保存的存储设备中，如将内存中的对象存储到关系型数据库中，或者XML文件中，或者其它什么。相比它的实际含义，真对不起它高大上的名称。类比来说，持久化就是将你头脑中的想法写在纸上，于是想法持久化了。具体来说，持久化就是将一个对象由内存插入到数据库的表中，数据表里有了这个对象的所有属性值，则这个对象就被持久化了。（被这三个字蒙骗这么久，我真疑惑自己是不是入错行了...）

## 持久化框架

	能力有限，只能说说我仅仅认识的Java持久化框架。应该有不少，但我知道的就hibernate和Mybatis，有使用过hibernate，但原理不清楚，Mybatis没自己弄过。但今天粗略看了一下，感觉这两个持久化框架大体上是相似的。因为了解还不够，所以就只记录这一点点：持久层框架就是为了操作数据库，所以都是封装JDBC，提供CURD操作。

### JDBC连接数据库

	>JDBC（Java Data Base Connectivity,java数据库连接）是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。
	JDBC连接数据库步骤，以MySql为例：
	1.加载数据库驱动：`Class.forName("com.mysql.jdbc.Driver");`
	2.建立数据库连接：`Connection con = DriverManager.getConnection("jdbc:mysql://localhost:3306/database",username,password);`
	3.创建Statement：`Statement st = con.createStatement();`
	4.执行sql语句并返回结果集：`ResultSet rs = st.execute(sql);`
	大致上应该就这些步骤，先大概记录一下。

## Mybatis

	网上有特别多的简介，我就懒得搬了。刚发现一个讲解Mybatis的专栏，[传送门](http://blog.csdn.net/column/details/mybatis-principle.html)
