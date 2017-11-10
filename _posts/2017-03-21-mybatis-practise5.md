---
layout : post
title : "Mybatis学习笔记（五）--与SpringMVC整合"
categories : Mybatis
tags : Mybatis SpringMVC
---

* content
{:toc}

　　使用IntelliJ IDEA还是有点水土不服，在写配置文件和访问服务器的路径上面吃了些亏，花了两天才将项目跑起来。搭建一个框架，需要注意的地方实在是太多了，一点不明白的地方，出了问题就不知道怎么解决，暂时我就先记下与SpringMVC整合的过程和碰到的一些问题。





## 整合过程

　　与SpringMVC整合的过程会增加一些重要的配置，然后就是测试需要用到的Controller类和JSP页面。涉及到的东西有拦截器、注解、JSTL标签等。

### 配置

　　首先去配置pom.xml，引入spring-context和spring-webmvc，代码如下：

```xml

<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>3.2.18.RELEASE</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>3.2.18.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>jstl</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>

  </dependencies>

  <build>
    <finalName>test</finalName>
    <resources>
      <resource>
        <directory>src/main/java</directory>
        <includes>
          <include>**/*.xml</include>
        </includes>
      </resource>
    </resources>
  </build>

```

　　我将此次整合SpringMVC中在pom.xml文件中增加的内容全部写了出来，后面就不需要再贴了。引入的jstl是为了支持JSP使用jstl标签库，而build标签中的&lt;resources&gt;是为了解决与dao对应的mapper.xml文件不写在resources文件夹而不编译的问题。

　　然后就去web.xml配置拦截器，整体的内容如下：

```xml

<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:application.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>

```

　　在web.xml文件中，首先定义系统级别的上下文，即在<context-param>中指定application.xml的位置。然后注册ContextLoaderListenerd，ContextLoaderListener通过读取contextConfigLocation参数来读取配置参数，一般来说它配置的是Spring项目的中间层，服务层组件的注入，装配，AOP。最后定义请求拦截器并配置需要拦截的请求，“/”表示拦截所有请求。

　　最后，配置拦截器的配置文件。需要新建一个xml文件，文件名默认为拦截器名+servlet。代码如下：

```xml

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.0.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd">

       <context:component-scan base-package="com.shiliew.controller"/>
       <mvc:annotation-driven />

       <mvc:resources mapping="/static/**" location="/WEB-INF/static"/>
       <mvc:default-servlet-handler/>

       <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
              <property name="prefix">
                     <value>/WEB-INF/pages/</value>
              </property>
              <property name="suffix">
                     <value>.jsp</value>
              </property>
       </bean>
</beans>

```

　　开头的xsd很重要，暂时没找到方法去引入正确的xsd，只能先抄了。我在项目中用的是@Controller标识Controller类，于是需要配置需要扫描的类的路径，声明为注解驱动。下面的<bean>是为了配置正确的jsp页面的，前缀加后缀，返回一个正确的jsp页面的路径。

### Controller类和JSP页面

　　Controller类和JSP页面和我在公司写的没多大区别，直接贴码：

```java

package com.shiliew.controller;

import com.shiliew.dao.UserDao;
import com.shiliew.model.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
import java.util.List;

@Controller
@RequestMapping("user")
public class UserController {

    @Autowired
    private UserDao userDao;

    @RequestMapping("queryUser")
    public ModelAndView queryUser(){
        ModelAndView mv = new ModelAndView("test");
        List<User> list = userDao.queryUserList();
        mv.addObject("list", list);
        return mv;
    }
}

```

```jsp

<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jstl/core"%>
<html>
<head>
    <title></title>
</head>
<body>
<c:out value="${list}"/>
<c:forEach items="${list}" var="item">
  <c:out value="${item.id}"/>
  <c:out value="${item.name}"/>
  <c:out value="${item.password}"/>
</c:forEach>
</body>
</html>

```

### 问题

　　花了两天时间才将程序跑起来，遇到的问题还是有一些。记录一下：
1. 首先是访问http://localhost:8080/mybatisTest/user/queryUser报404，mybatisTest是我的项目名，昨晚回去之前碰到的问题，尽早来到公司想了一下，猜想可能是tomcat的问题，于是就把mybatisTest去掉了，然后就可以访问到项目了。于是去查看tomcat的配置，看到Deployment页有个Application context为“/”，然后就懂了。

2. 第二个问题就是Invalid bound statement (not found): com.shiliew.dao.UserDao.queryUserList。意思大约就是找不到该方法的实现，调用报错。查阅相关资料后，问题定位在User.xml文件没有编译，在idea中很容易去查看target/classes/com/shiliew，发现果然没有mapper文件夹。再百度之后，问题就变成了“IDEA 中src下xml等资源文件无法读取的问题”。参考[解决IDEA 中src下xml等资源文件无法读取的问题](http://blog.csdn.net/shifangwannian/article/details/48882201)。试了试文中的第四种方法，结果把项目的目录搞得乱七八糟，不知道怎么还原。无奈查看了公司项目有没有相应的处理方式，发现公司在pom.xml文件中做了声明，即上面提到的<resources>，我也应用之，问题解决。其他的方式暂时就先不考虑了。

3. 最后一个问题是关于JSTL的，在没有引入jstl之前，jsp页面报错The absolute uri: http://java.sun.com/jsp/jstl/core cannot be resolved ...，然后将引入了JSTL。在JSP页面中声明<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>，结果将${list}当成字符串输出了，再百度，了解到版本不对，需要引入<%@ taglib prefix="c" uri="http://java.sun.com/jstl/core"%>，然后得到正确的结果。

## 结语

　　暂时就记这么多吧，记得比较粗略，先下班...

