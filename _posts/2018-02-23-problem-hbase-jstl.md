---
layout : post
title : "Hadoop与jstl的jar包冲突"
category : JavaWeb
tags : hadoop jstl
---

* content
{:toc}

　　春节之前编写jsp页面出现的问题，从其他项目拷贝过来的jsp页面无法正常显示el表达式。而一切都和引入了Hbase的项目说起。









#### 问题描述

　　为了展示从Hbase中查找出来的数据，从其他项目拷贝了一份jsp和其样式。但是jsp页面将${serverName}这样的el表达式当作字符串直接打印出来，导致很多js和css文件获取不到，页面无法正常显示。

#### 探索解决

　　在探索的过程中，首先想到的是jstl的版本，maven中引用的为1.2，然后将jsp页面应用的标签库由
	```<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>```
	修改为
	```<%@ taglib prefix="c" uri="http://java.sun.com/jstl/core"%>```，
但是发现没有用。然后为每个el表达式套上jstl标签，比如```<c:out value="${vs.count}" />```，在web.xml中web-app2.4版本规则下，可以解析出正确的el表达式。

　　本以为问题解决了，但是jstl中的for:each却无法正确执行完，页面的数据显示不完整。在探究的过程中，乱七八糟的错误层出不穷。最后查到Hadoop和jstl存在jar包冲突的问题，日志中也出现了NoSuchMethod这样的错误信息。联想到项目引入了Hbase相关的jar包，因此利用排除法，将hadoop-common和hbase-common两个包中的相关jar包给排除了，终于，jsp页面正常显示了。

#### 解决方案

　　引入的hbase依赖存在一些与原来JavaWeb项目存在的jstl冲突的jar包，将其排除掉就可以了。

```xml
<dependency>
		  <groupId>org.apache.hadoop</groupId>
		  <artifactId>hadoop-common</artifactId>
		  <version>${hadoop.version}</version>
		  <exclusions>
			  <exclusion>
				  <artifactId>servlet-api</artifactId>
				  <groupId>javax.servlet</groupId>
			  </exclusion>
			  <exclusion>
				  <artifactId>jetty</artifactId>
				  <groupId>org.mortbay.jetty</groupId>
			  </exclusion>
			  <exclusion>
				  <artifactId>jetty-util</artifactId>
				  <groupId>org.mortbay.jetty</groupId>
			  </exclusion>
			  <exclusion>
				  <artifactId>jersey-core</artifactId>
				  <groupId>com.sun.jersey</groupId>
			  </exclusion>
			  <exclusion>
				  <artifactId>jersey-json</artifactId>
				  <groupId>com.sun.jersey</groupId>
			  </exclusion>
			  <exclusion>
				  <artifactId>jersey-server</artifactId>
				  <groupId>com.sun.jersey</groupId>
			  </exclusion>
			  <exclusion>
				  <artifactId>jasper-compiler</artifactId>
				  <groupId>tomcat</groupId>
			  </exclusion>
			  <exclusion>
				  <artifactId>jasper-runtime</artifactId>
				  <groupId>tomcat</groupId>
			  </exclusion>
			  <exclusion>
				  <artifactId>jsp-api</artifactId>
				  <groupId>javax.servlet.jsp</groupId>
			  </exclusion>
			  <exclusion>
				  <artifactId>jdk.tools</artifactId>
				  <groupId>jdk.tools</groupId>
			  </exclusion>
		  </exclusions>
	  </dependency>
```

　　示例如上，至于为何要去除以上这些包，我没深究，但大致原因在于jasper造成的NoSuchMethod错误和其他一些包造成的jstl标签无法正常工作。

#### 参考

[关于hadoop与jstl冲突的jar包问题](http://blog.csdn.net/xiao_jun_0820/article/details/54948017)

[java--遇到NoSuchMethodError通用解决思路](https://www.cnblogs.com/xiaoMzjm/p/4566672.html)