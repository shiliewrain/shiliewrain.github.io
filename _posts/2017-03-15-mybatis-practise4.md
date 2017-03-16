---
layout : post
title : "Mybatis学习笔记（四）--Mybatis与Spring整合"
categories : Mybatis
tags : Mybatis Spring
---
* content
{:toc}

　　按照网上的教程进行的整合，主要是将数据源datasource和sqlsession交给Spring去管理，数据源体现在将datasource的配置从原来的mybatis-config.xml文件中移到了spring的配置文件application.xml中，sqlsession体现在将写在测试方法中的session移到了spring的配置文件application.xml中。







### 包的引入

　　要引入Spring，则要引入相应的包，主要包括Spring核心包、Spring上下文、Spring-JDBC。然后，整合Mybatis和Spring，需要引入Mybatis-Spring的功能包。最后，需要引入连接池，这里引入commons-dbcp。示例如下：

```xml

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>mybatis-shiliew</groupId>
  <artifactId>test</artifactId>
  <packaging>war</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>test Maven Webapp</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.2</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.0.5</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.springframework/spring-core -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>3.2.18.RELEASE</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>3.2.18.RELEASE</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>3.2.18.RELEASE</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.1</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/commons-dbcp/commons-dbcp -->
    <dependency>
      <groupId>commons-dbcp</groupId>
      <artifactId>commons-dbcp</artifactId>
      <version>1.4</version>
    </dependency>

  </dependencies>
  
  <build>
    <finalName>test</finalName>
  </build>
</project>

```

### Spring配置文件

　　在项目中新建一个application.xml文件，这个即为Spring的配置文件，在其中要定义数据源和SqlSession工厂。配置如下：

```xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

       <!-- 建立数据源 -->
       <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
              <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
              <property name="url" value="jdbc:mysql://localhost:3306/shiliew"/>
              <property name="username" value="root"/>
              <property name="password" value="root"/>
       </bean>

       <!-- 建立sqlsession工厂 -->
       <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
              <property name="dataSource" ref="dataSource"/>
              <property name="configLocation" value="mybatis-config.xml"/>
       </bean>

       <bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
              <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
              <property name="mapperInterface" value="com.shiliew.dao.UserDao"/>
       </bean>

</beans>

```

　　配置数据源是为了将数据库连接由Java硬编码转为Spring管理，也能方便地进行事务管理。Spring配置数据源有四种方式，暂时参考这篇文章[Spring中配置数据源的4种形式](http://blog.csdn.net/orclight/article/details/8616103)，因为我参照的教程使用的是dbcp，所以我采用的为dbcp。建立数据源之后，就可以访问数据库了。每一次访问数据库，都会打开一个会话，即session。在之前的测试方法中，我使用了openSession()方法去打开一次会话，访问结束之后使用close()方法将会话关闭。这样的操作很麻烦，还好Spring也将Session进行了管理。因此，我们在application.xml中还要配置sqlSession工厂，这个工厂主要是为我们自动建立一次会话，省去了我们自己编码的麻烦。最后就是配置Mapper工厂，这个工厂是根据Mapper接口提供我们想要的Mapper对象，它封装了之前测试类中用到的session.getMapper()方法。

### 测试

　　写到这里，就可以测试了，测试代码如下：

```java

package com.shiliew.test;

import com.shiliew.dao.UserDao;
import com.shiliew.model.Task;
import com.shiliew.model.User;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.util.List;

public class MybatisSpringTest {

    private static ApplicationContext ctx = new ClassPathXmlApplicationContext("application.xml");

    public static void main(String [] args){
        UserDao userDao = (UserDao) ctx.getBean("userMapper");
        User user = userDao.selectUserByID(1);
        System.out.println(user.getName());
        List<Task> taskList = userDao.getUserTaskList(1);
        for (Task task : taskList){
            System.out.print(task.getId() + " ");
            System.out.print(task.getTaskName() + " ");
            System.out.print(task.getContent() + " ");
            System.out.println(task.getUser().getName());
        }
    }
}

```

　　在测试过程中，我遇到了一个问题，就是我在User实体和Task实体我都用到了name这个变量名，在获取task的name时，显示的是User的name。最后没办法，只有将task的name修改为taskName。这个问题发生的原因我估计是Mybatis在映射时分不清造成的，等之后对Mybatis的实体映射有足够了解之后再来解释。