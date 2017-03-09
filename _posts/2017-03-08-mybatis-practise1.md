---
layout : post
title : "Mybatis学习笔记（一）--环境搭建"
categories : Mybatis
tags : Mybatis 配置管理
---

* content 
{:toc}

　　工作比较忙，抽时间在IntelliJ Idea上实践了一小把Mybatis的环境搭建，东西都非常简单，但是过程并不那么顺利，特此记录下学习过程，以备后面查阅。




## Mybatis开发环境搭建

　　最近想从eclipse转到IntelliJ Idea上，因此都在idea上进行实践。虽然还有部分概念不是特别清楚明白，但是并不妨碍使用，估计后面熟悉了之后，也会对idea中的一些概念有更深刻的认识。

### 开发环境

* IntelliJ Idea 14
* mysql 5.1　　
* jdk 1.7
* maven 3.3

　　记录一下开发环境，版本确实很重要，实践过程中就碰到了`com/mysql/jdbc/Driver : Unsupported major.minor version 52.0`情况，是由于我引入mysql-connector-java.jar版本太高，jdk1.7不支持。

　　搭建Mybatis的开发环境需要用Mybatis和mysql-connector-java的jar包，可以提前下好加入到项目中，也可以直接在maven项目中的pom.xml中引入依赖。因为我新建的maven项目，所以就直接在pom.xml文件中引入了依赖，这种方式也更加的方便。在这里记录一个查询依赖的网址[maven依赖查询](http://mvnrepository.com/)


## 搭建过程

　　搭建过程只包含了项目的搭建，数据库以及表的构建省略了。这里标明为本地的mysql中的shiliew库，表为User，包含id(主键)，name和password三个字段。

### 依赖引入

　　在pom.xml文件中引入Mybatis和mysql-connector-java的jar包，如下：

```xml
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
```

　　特别注意它们的版本和jdk的版本，如果引入依赖的版本过高，可能会导致jdk不支持，从而报错。

### 类的构建

　　这里主要需要建一个实体类User.java和一个接口UserDao.java。User.java中包含数据库中表的属性，代码如下：

```java

package com.shiliew.model;

/**
 * Created by huangwenhao on 2017/3/8.
 */
public class User {

    private Integer id;

    private String name;

    private String password;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

　　接口中定义相关的增删改查操作，这里只定义了一个按id查找User的方法，代码如下：

```java
package com.shiliew.dao;

import com.shiliew.model.User;

/**
 * Created by huangwenhao on 2017/3/8.
 */
public interface UserDao {

    User selectUserByID(Integer id);
}
```

### 配置文件

　　使用Mybatis需要为每一个如同UserDao.java的接口定义一个对应的配置文件，这里为User.xml，该文件与前面定义的接口是对应的，在namespace中要指明该接口，包名必须正确，<select>标签中的id的值则为接口中的方法名，parameterType为该方法接收参数的类型，resultType则为该方法的返回值。<select>标签中的sql语句则为我们需要实现的查找功能，参数通过\#{}传递，代码如下：
```xml

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.shiliew.dao.UserDao">
<select id="selectUserByID" parameterType="int" resultType="User">
    select * from user where id = #{id}
</select>
</mapper>

```

　　最后需要定义Mybatis的配置文件，这里为mybatis-config.xml，<typeAliases>标签中声明的为类的别名，别名的作用是简化引用相应类的操作。如上文的User.xml中的resultType中的User就是使用的别名，如果没有给User声明别名，或者不使用别名，则需要写成`resultType="com.shiliew.model.User"`的形式。parameterType也是如此，如果我们返回的数据类型是User型的，则也可以使用别名或者全称。<environments>中声明了事务和数据源，内容暂时不研究。最后，我们需要在<mappers>中将所有的与Dao接口对应的xml文件引入进来，代码如下：

```xml

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<typeAliases>
    <typeAlias alias="User" type="com.shiliew.model.User"/>
</typeAliases>

<environments default="development">
    <environment id="development">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
            <property name="driver" value="com.mysql.jdbc.Driver"/>
            <property name="url" value="jdbc:mysql://localhost:3306/shiliew" />
            <property name="username" value="root"/>
            <property name="password" value="root"/>
        </dataSource>
    </environment>
</environments>

<mappers>
    <mapper resource="User.xml"/>
</mappers>
</configuration>

```

## 测试

　　将以上的内容定义完之后，就可以编写一个测试类进行测试了，如下：

```java

package com.shiliew.test;

import com.shiliew.model.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.Reader;

/**
 * Created by huangwenhao on 2017/3/8.
 */
public class MybatisTest {

    private static SqlSessionFactory sqlSessionFactory;

    private static Reader reader;

    static {
        try {
            reader = Resources.getResourceAsReader("mybatis-config.xml");
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    public static SqlSessionFactory getSqlSessionFactory(){
        return sqlSessionFactory;
    }

    public static void main(String [] args){
        SqlSession session = sqlSessionFactory.openSession();
        try {
            User user = session.selectOne("com.shiliew.dao.UserDao.selectUserByID",1);
            System.out.println(user.getName());
            System.out.println(user.getPassword());
        }finally {
            session.close();
        }
    }
}

```
