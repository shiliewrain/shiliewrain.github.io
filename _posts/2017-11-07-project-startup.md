---
layout : post
title : "Web项目启动过程"
category : Web
tags : servlet listener filter
---
* content
{:toc}

　　一个Web项目随着Tomcat服务器启动，Tomcat会读取项目中的web.xml文件，文件中有context-param、listener、filter和servlet这几个重要的标签，而web项目的启动也和这四个标签有很大的关系。




### 前言

　　这里以spring-mvc、spring和mybatis框架为基础的web项目启动为例，简单地介绍下四个标签的作用，以及web启动时这四个标签加载的顺序。更详细的内容可以查参考或者查源码。

### web.xml的四种重要标签

　　tomcat启动时，会读取项目中的web.xml文件。其中较为重要的标签包括：context-param、listener、filter和servlet。

#### context-param

```xml
<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>
			classpath:applicationContext.xml
		</param-value>
	</context-param>
```

　　该标签包含web应用servlet上下文初始化参数的声明，在这里声明的参数是全局的，存在于ServletContext容器中。listener、filter等在初始化时可以使用这些上下文信息。在servlet中可以通过getServletContext().getInitParameter("context/param")方法获取参数的值，用其指定spring配置文件应该是该标签最常用的用途。

#### listener

```xml
<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
```

　　该标签用来注册一个监听器类。事件监听程序包括：应用的启动和关闭；session的创建与销毁，属性的新增、移除和更改；对象被绑定到session中或从session中删除。

#### filter

```xml
<filter>
		<filter-name>Set Character Encoding</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>utf-8</param-value>
		</init-param>
	</filter>	
	<filter-mapping>
	    <filter-name>Set Character Encoding</filter-name>
	    <url-pattern>/*</url-pattern>
  	</filter-mapping>
```

　　该标签用来过滤请求。在request请求到达servlet之前进行预处理，在response响应离开servlet之后进行后处理。常用的用途如上面代码中的设置请求和响应的编码方式。

#### servlet

```xml
<servlet>
		<servlet-name>dispatcher</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>dispatcher</servlet-name>
		<url-pattern>*.do</url-pattern>
	</servlet-mapping>
```

　　该标签用于处理用户请求。可以看到servlet和filter标签的使用比较像，filter就像servlet前面的安检。

### web.xml中标签加载顺序

　　web项目启动时，以上四种标签加载的顺序是：context-param，listener，filter，servlet。除了filter-mapping不能写在filter前面和servlet-mapping不能写在servlet前面，其余的没有书写顺序要求。启动步骤如下：

1. web项目启动，服务器容器读取context-param和listener；

2. 服务器容器创建ServletContext；

3. 将context-param转化为键值对放入ServletContext；

4. 服务器容器创建listener的实例；

5. 服务器容器创建filter实例；

6. 按照load-on-startup去实例servlet。

### 总结

　　内容写的比较简单，很多细节都没有去讲，因为我也不清楚，而且要全部描述清楚，恐怕篇幅会很恐怖。这里记住context-param、listener、filter和servlet的加载顺序，和标签的基本用途就可以了。

### 参考

[servlet/filter/listener/interceptor区别与联系](http://www.cnblogs.com/doit8791/p/4209442.html)

[java web项目的启动及初始化](http://www.cnblogs.com/lookdown/p/5170866.html)

[Java Web项目启动执行顺序](http://blog.csdn.net/qq_20805103/article/details/77851996)

[servlet什么时候被实例化？【转】](http://www.cnblogs.com/disneyland/p/4692339.html)