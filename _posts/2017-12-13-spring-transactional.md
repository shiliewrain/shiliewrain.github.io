---
layout : post
title : "Spring事务管理"
category : Spring
tags : 事务 Spring 隔离级别
---
* content
{:toc}

　　Spring是基于数据库的事务提供事务功能的，若数据库不支持事务，则Spring无法提供事务管理。因为Spring的事务管理知识都比较理论，因此本文只是基于其他文章的总结。





### 事务的原理

　　在数据库中，开启一个事务使用的是```BEGIN TRANSACTION```，提交事务为```COMMIT TRANSACTION```。Spring是基于数据库的事务功能提供事务服务的，大致的步骤为：

1. 获取连接 Connection con = DriverManager.getConnection();

2. 开启事务 con.setAutoCommit(true/false);

3. 执行sql

4. 提交/回滚事务 con.commit() / con.rollback();

5. 关闭连接 con.close();

　　Spring为我们提供的事务就是2，4两步。当我们使用声明式事务，Spring会在启动时为每个被声明了事务的bean生成一个代理类，该代理类就提供了事务的开启和关闭。而真正的事务提交和回滚是由数据库的binlog和redolog实现的。

### Spring的事务传播属性

　　Spring的事务传播属性，在我看来是在存在事务嵌套的情况下，Spring如何去处理事务。Spring主要有七种传播属性，但只需要理解其中的四种。以A类中的a()会调用B类中的b()方法为例来介绍，b方法为下列传播属性。

* PROPAGATION_REQUIRED 支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择，也是 Spring 默认的事务的传播。若a起了事务，b就使用当前事务，否则新建事务。

* PROPAGATION_REQUIRES_NEW 新建事务，如果当前存在事务，把当前事务挂起。新建的事务将和被挂起的事务没有任何关系，是两个独立的事务，外层事务失败回滚之后，不能回滚内层事务执行的结果，内层事务失败抛出异常，外层事务捕获，也可以不处理回滚操作。若a起了事务，则挂起，b新建事务。a回滚不影响b，b回滚，a可捕获异常。

* PROPAGATION_SUPPORTS 支持当前事务，如果当前没有事务，就以非事务方式执行。若a起了事务，b就使用当前事务，否则无事务。

* PROPAGATION_MANDATORY 支持当前事务，如果当前没有事务，就抛出异常。若a起了事务，b就使用当前事务，否则报错。

* PROPAGATION_NOT_SUPPORTED 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。若a起了事务，则挂起该事务，b执行。

* PROPAGATION_NEVER 以非事务方式执行，如果当前存在事务，则抛出异常。若a起了事务，则报错。

* PROPAGATION_NESTED 如果一个活动的事务存在，则运行在一个嵌套的事务中。如果没有活动事务，则按REQUIRED属性执行。它使用了一个单独的事务，这个事务拥有多个可以回滚的保存点。内部事务的回滚不会对外部事务造成影响。它只对DataSourceTransactionManager事务管理器起效。无论a是否起了事务，b都新建一个事务。

　　PROPAGATION_REQUIRES_NEW和PROPAGATION_NESTED的区别在于：

　　PROPAGATION_REQUIRES_NEW起的是一个独立的事务，该事务不依赖于外部环境，拥有自己的隔离范围和锁，当此事务执行时，外部事务会被挂起，知道该事务提交后，外部事务才继续执行。

　　PROPAGATION_NESTED起的是一个嵌套事务，若外部存在事务，则该嵌套事务开始执行时会获得一个savepoint点。如果嵌套事务执行失败，则回滚到此savepoint。只有外部事务提交时，该嵌套事务才会提交。若外部事务执行失败，会带着嵌套事务一起回滚。

### 隔离级别

#### 数据库隔离级别

　　数据库的隔离级别由低到高分为Read-Uncommitted、Read-Committed、Repeatable-Read和Serializable，主要是对脏读、不可重复读和幻读的隔离。

* 脏读 一事务对数据进行了增删改，但未提交，另一事务可以读取到未提交的数据。如果第一个事务这时候回滚了，那么第二个事务就读到了脏数据。

* 不可重复读 一个事务中发生了两次读操作，第一次读操作和第二次操作之间，另外一个事务对数据进行了修改，这时候两次读取的数据是不一致的。

* 幻读 第一个事务对一定范围的数据进行批量修改，第二个事务在这个范围增加一条数据，这时候第一个事务就会丢失对新增数据的修改。

#### Spring隔离级别

　　Spring的隔离级别有5种，但只是针对PlatfromTransactionManager单独提出了一种ISOLATION_DEFAULT，它使用数据库默认的隔离级别。而其他的隔离级别与上面对应，分别为：ISOLATION_READ_UNCOMMITTED、ISOLATION_READ_COMMITTED、ISOLATION_REPEATABLE_READ和ISOLATION_SERIALIZABLE。

### 总结

　　Spring是基于数据库提供事务支持的，在使用声明式事务的方式中，Spring通过代理类实现事务。Spring的默认传播属性为PROPAGATION_REQUIRED，事务的传播属性主要规范在嵌套事务存在的情况下，Spring该如何执行事务。Spring提供了5种隔离级别，隔离级别越高，CRUD的效率越差。

### 参考

[深入理解 Spring 事务原理](http://www.codeceo.com/article/spring-transactions.html)

[PROPAGATION_REQUIRES_NEW 和 PROPAGATION_NESTED区别](http://blog.csdn.net/u011285162/article/details/19247711)