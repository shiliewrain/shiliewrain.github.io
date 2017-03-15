---
layout : post
title : "Mybatis学习笔记（四）--Mybatis与Spring整合"
categories : mybatis
tags : mybatis spring
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


