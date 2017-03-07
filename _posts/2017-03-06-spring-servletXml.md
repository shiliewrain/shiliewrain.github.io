---
layout : post
title : "关于spring-servlet.xml配置文件的一点记录"
categories : spring
tags : springMVC spring配置
---

* content
{:toc}

　　重新学习的路上层层艰难，以前挖的坑确实很大，现在几乎等于从零起步。今天学习搭建springMVC+maven+mybatis框架，IDE用的IntelliJIDEA。过程估计会很长，一点一点记录。今天就弄懂了spring-servlet.xml配置文件中的一些小地方，记下以便后期查看。






## application.xml与servlet.xml

　　从惯例上来说，application.xml配置文件一般为系统级别，为系统应用上下文，主要用于系统的整体配置，如数据源、事物和日志。而servlet.xml配置文件为controller级别的，为servlet应用上下文，主要用于servlet分发请求的。在web.xml文件中，需要在<context-param>标签中定义application.xml，而在<servlet>中定义servlet.xml。

## classpath与classpath*

　　在定义application.xml与servlet.xml的位置的时候，经常会出现classpath或者classpath\*，尤其是定义application.xml的位置。classpath所指的默认位置为/WEB-INF/classes/，会从其中找出所有的指定配置文件。而classpath*则会找出/WEB-INF/classes/及其子文件夹下的所有指定配置文件，简单来说，就是classpath\*会查找得更深一些，当然初始化过程中花费的时间也会更多一下，因此如果清楚配置文件的位置，尽量少用classpath\*。另外一种情况，在声明servlet的时候不去指定配置文件的位置，则系统会根据<servlet-name>中的值去查找/WEB-INF/中的<servlet-name>-servlet.xml文件，如果没有，则会报错。