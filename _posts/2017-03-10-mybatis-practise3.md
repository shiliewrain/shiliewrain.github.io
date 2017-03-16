---
layout : post
title : "Mybatis学习笔记（三）--关联查询"
categories : Mybatis
tags : Mybatis 关联查询
---

* content
{:toc}

　　时间不多，就实践了一下Mybatis的关联查询，感觉确实很好用，特此记下学习记录。




## Mybatis关联查询

　　在前面学习了Mybatis的基础CURD操作，在实际工作中肯定不够用，多表联合查询肯定是经常出现的，因此关联查询的用处很大。

### 虚拟场景

　　在之前的实践的基础上，建立一个很简单的虚拟场景。数据库中已存在user表，然后新建一个task表，记录user表中用户的任务，字段包括：id（主键）、user_id（用户id）、name（任务名称）、content（任务内容）。一个用户可能拥有0到多个任务，我们需要查询出某一个用户的名字和其所有任务。如查找用户id为1的数据，使用sql语句写就如下：

```sql
SELECT user.id, user.name, task.id,task.name,task.content FROM user, task WHERE user.id = task.user_id AND user.id=1;
```

### Mybatis实现

　　首先需要建立一个存放以上数据的类，这里为Task.java，代码如下：

```java

package com.shiliew.model;

public class Task {

    private Integer id;

    private User user;

    private String name;

    private String content;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }
}
```

　　这个实体类并不是对应着数据库中的task表，属性里包含User，而不是userId。这是因为我们的需求是要用户的名字和他的所有任务，所以才这样建立实体。然后，我们需要在UserDao中定义一个查询的方法——`List<Task> getUserTaskList(Integer id);`，方法返回Task的集合，接收用户的id为入参。最后，需要在User.xml中实现该方法的sql。

### resultMap的实现

　　联合查询中的resultMap相对于之前简单的查找User要复杂一些，复杂的点就在于属性User的定义。这时，就需要使用<association>标签来实现User的定义，<association>中的property指明Task.java实体类中的user属性，javaType则表示user的属性，在<association>标签中继续定义user的属性，将id和name写入，则满足了我们的需求。代码如下：

```xml

	<resultMap id="UserTaskList" type="Task">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="content" property="content"/>

        <association property="user" javaType="User">
            <id column="id" property="id"/>
            <result column="name" property="name"/>
        </association>
    </resultMap>

    <select id="getUserTaskList" parameterType="int" resultMap="UserTaskList">
        SELECT user.id, user.name, task.id,task.name,task.content FROM user, task
        WHERE user.id = task.user_id AND user.id=#{id}
    </select>
```

　　对于<association>的定义还有另外一种写法，就是将标签里的属性抽出来，作为一个resultMap定义。这样做的好处是方便复用。示例如下：

```xml

	<resultMap id="UserList" type="User">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="password" property="password"/>
    </resultMap>

    <resultMap id="UserTaskList" type="Task">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="content" property="content"/>

        <association property="user" javaType="User" resultMap="UserList"/>
    </resultMap>
```

　　在本例中，我直接把之前的resultMap拿来复用，十分方便。最后，将getUserTaskList方法实现，代码如下：

```xml

	<select id="getUserTaskList" parameterType="int" resultMap="UserTaskList">
        SELECT user.id, user.name, task.id,task.name,task.content FROM user, task
        WHERE user.id = task.user_id AND user.id=#{id}
    </select>
```

## 测试

　　测试代码如下：

```java

package com.shiliew.test;

import com.shiliew.dao.UserDao;
import com.shiliew.model.Task;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.Reader;
import java.util.List;

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
            UserDao userDao = session.getMapper(UserDao.class);
            List<Task> list = userDao.getUserTaskList(1);
            for (Task task : list){
                System.out.println(task.getUser().getName()+"的任务名称为"+task.getName()+",任务内容为"+task.getContent());
            }
        }finally {
            session.close();
        }
    }
}
```
