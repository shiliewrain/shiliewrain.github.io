---
layout : post
title : "Mybatis学习笔记（二）--基础CURD"
categories : Mybatis
tags : mybatis CURD
---

* content
{:toc}


　　今天接着加班的时间把Mybatis的基础CURD操作实践了一遍，也就是基础增删改查操作，内容比较简单，记录学习过程，打好基础。





## Mybatis CURD

　　就我目前了解来说，在Mybatis中，对于数据库的CURD操作主要涉及两个文件，一个是定义CURD方法的接口，另一个就是实现CURD的sql语句的xml文件。在xml文件mapper标签的namespace中指定对应的接口就行，包名和接口名都必须一致。

　　然后就说明一下resultMap，与resultType对比区别一下。返回的数据类型由xml和dao文件里的方法共同决定，mybatis会将返回的数据放在一个map里，然后去映射resultType或者resultMap。返回值如果是多个对象，而方法只返回一个非list值，则会报错。resultMap更多用于联合查询，仅仅而已。两者不可同时出现。关于更详细的对比，可以参考[Mybatis中的resultType和resultMap](http://blog.csdn.net/woshixuye/article/details/27521071)。

　　在resultMap中可以定义type和id，type指定集合元素中的类型，id留给外部引用。然后就在resultMap标签中定义相应的属性。示例如下：

```xml
	<resultMap id="UserList" type="User">
      	<id column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="password" property="password"/>
	</resultMap>
```

　　此resultMap会在后面的查询中用到。

### 查询

　　在Mybatis中实现一个查询功能，需要在dao接口中定义一个查询方法，然后在对应的xml文件中实现sql，示例如下:

```xml
	<!-- 查询单个User dao层中：User selectUserByID(Integer id);-->
	<select id="selectUserByID" parameterType="int" resultType="User">
	    select * from user where id = #{id}
	</select>
	<!-- 查询全部User dao层中：List<User> queryUserList();-->
	<select id="queryUserList" resultMap="UserList">
	    SELECT * FROM USER
	</select>
```

　　上面两个查询都可以使用```resultType="User"```和```resultMap="UserList"```，因为它们只是数据与对象属性的映射，其实resultType是将resultMap中的元素直接映射到了User对象上，方法的返回值才决定是对象还是对象list。resultMap主要用于复杂的联合查询，举例：存在一个Task类，描述任务内容和执行任务的人，那么查询任务要求查询出对应的User信息。只是举例说明resultMap大概用于哪些方面多一些，没有绝对。

```xml
<resultMap type="Task" id="TaskMap">
    <association property="user" select="selectUserByID" column="user" javaType="User"/>
</resultMap>
```

### 插入

　　先在UserDao中定义一个插入方法insertUser，然后在User.xml中实现相应的sql。useGeneratedKeys表示是否使用JDBC的getGenereatedKeys方法获取主键并赋值到keyProperty设置的领域模型属性中。Mybatis将采用反射的形式读取User中name和password的值，如下：

```xml
	<!-- 插入一条User dao层中：void insertUser(User user); -->
	<insert id="insertUser" parameterType="User" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO USER(name ,password) VALUE (#{name}, #{password})
    </insert>
```

### 更新

　　先在UserDao中定义一个更新方法updateUser，然后在User.xml中实现相应的sql。如下：

```xml
	<!-- 插入一条User dao层中：void updateUser(User user); -->
    <update id="updateUser" parameterType="User">
        UPDATE USER SET name=#{name}, password=#{password} WHERE id=#{id}
    </update>
```

### 删除

　　先在UserDao中定义一个删除方法deleteUser，然后在User.xml中实现相应的sql。如下：

```xml
	<!-- 删除一条User dao层中：void deleteUser(User user); -->
    <delete id="deleteUser" parameterType="User">
        DELETE FROM USER WHERE id=#{id}
    </delete>
```

## 测试

　　测试代码如下：

```java
public static void main(String [] args){
        SqlSession session = sqlSessionFactory.openSession();
        try {
            UserDao userDao = session.getMapper(UserDao.class);
//            User user1 = new User();
//            user1.setName("111");
//            user1.setPassword("111");
//            userDao.insertUser(user1);

            /*User user1 = userDao.selectUserByID(5);
            user1.setName("555");
            user1.setPassword("555");
            userDao.updateUser(user1);

            User user2 = userDao.selectUserByID(3);
            userDao.deleteUser(user2);*/

            List<User> list = userDao.queryUserList();
            //因为mybatis-config.xml中声明了事务，所以一定要在执行完增删改操作之后提交事务，否则数据不会保存到表中。
            session.commit();
            for (User user : list){
                System.out.println(user.getId());
                System.out.println(user.getName());
                System.out.println(user.getPassword());
            }
//            System.out.println(user.getName());
//            System.out.println(user.getPassword());
        }finally {
            session.close();
        }
    }
```