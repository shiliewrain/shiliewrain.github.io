---
layout : post
title : "mysql实现全连接以及初步优化"
category : mysql
tags : 全连接 视图 子查询
---

* content
{:toc}


　　在项目中碰到需要实现mysql多表全连接的需求，最初使用left join加union来实现，后面考虑到性能问题，将全连接改为了多步去实现，也因此碰到了许多的问题，特此记录。





### 需求背景

　　在mysql中存在两张表aTable和bTable，其基础数据字段类似，因业务需求需要进行全连接，但是因为mysql并不支持full join，因此采用了曲线救国的方式。实际项目中，是一张表和一个视图需要进行全连接，因为区别不太大，并且为了方便理解，下面就按两表进行全连接说明了。


### mysql实现全连接

　　mysql本身不支持全连接，但考虑到全连接的数学模型，可以利用left join和union来实现全连接，示意sql语句如下：

```sql
SELECT 
	a.xxx,
	a.yyy,
	b.zzz
from aTable a LEFT JOIN bTable b 
ON a.xxx = b.xxx
UNION
(SELECT
	b.xxx,
	b.yyy,
	a.zzz
from bTable b LEFT JOIN aTable a
ON a.xxx = b.xxx)
```

### 优化

　　上面这样写能够查询出正确的结果，但是考虑到后期数据量会逐渐增大，该sql的执行效率会越来越低，因此考虑采用别的方式来实现。能用直连就不要使用外连，因此将全连拆为三种情况：

1. 存在于aTable，不存在于bTable中的数据。```sql select a.xxx, a.yyy, 0 as zzz from aTable a where not exists (select 1 from bTable b where a.xxx=b.xxx)```

2. 存在于bTable，不存在于aTable中的数据。```sql select b.xxx, b.yyy, 0 as zzz from bTable b where not exists (select 1 from aTable a where a.xxx=b.xxx)```

3. 同时存在于aTable和bTable中的数据。```sql select a.xxx, a.yyy, b.zzz from aTable a, bTable b where a.xxx=b.xxx```

　　以上三句sql就将全连接的各种情况都查出来了，然后使用union将结果集合并。准备将这三句sql合并生成一张视图，示意sql如下：

```sql
create or replace view abView as (
select xxx, yyy, zzz from (
select a.xxx, a.yyy, 0 as zzz from aTable a where not exists (select 1 from bTable b where a.xxx=b.xxx)
union
select b.xxx, b.yyy, 0 as zzz from bTable b where not exists (select 1 from aTable a where a.xxx=b.xxx)
union
select a.xxx, a.yyy, b.zzz from aTable a, bTable b where a.xxx=b.xxx
) c
)
```

### mysql创建视图不支持子查询

　　mysql执行上面的语句会报错，错误信息```[Err] 1349 - View's SELECT contains a subquery in the FROM clause```，这是说mysql创建视图不支持子查询，即不能在from后面写select子查询。网上查了很多资料，大部分的解决方案都是将子查询拆分建为新的视图。示意sql如下：

```sql
create or replace view abView as (
select xxx, yyy, zzz from (
select xxx, yyy, zzz from aView
union
select xxx, yyy, zzz from bView
union
select xxx, yyy, zzz from cView
) c
)
```

　　可以看到上面并没有改变子查询依旧存在的情况，考虑到上面子查询的sql是可以执行的，于是将外层的select去掉了。示意代码如下：

```sql
create or replace view abView as (
select xxx, yyy, zzz from aView
union
select xxx, yyy, zzz from bView
union
select xxx, yyy, zzz from cView
)
```

　　但是语句执行不了，继续查资料，发现可以将括号去除即可执行，实践之后发现确实可行。然后想到可以将现在的视图还原到最初的sql语句，示意sql如下：

```sql
create or replace view abView as 
select a.xxx, a.yyy, 0 as zzz from aTable a where not exists (select 1 from bTable b where a.xxx=b.xxx)
union
select b.xxx, b.yyy, 0 as zzz from bTable b where not exists (select 1 from aTable a where a.xxx=b.xxx)
union
select a.xxx, a.yyy, b.zzz from aTable a, bTable b where a.xxx=b.xxx
```

　　执行，成功！可以省去建立三张子视图的步骤。

### 总结

　　可能上面的sql存在错误，而且也应该存在更大的性能优化空间，但一个好的习惯是先写出能满足基本功能的sql语句，后面再考虑sql的优化。实际过程中，还遇到了多表连接关于on和where的一些问题，导致了部分关联查询的结果不符合预期，这些问题后面再讨论。

### 参考

[MySQL的进阶实战篇](http://blog.csdn.net/chaoluo001/article/details/70670227)

[Mysql中的视图](http://www.cnblogs.com/chenpi/p/5133648.html)

[left join后面加上where条件浅析](http://www.cnblogs.com/huahua035/p/5718469.html)

[Mysql Join语法以及性能优化](http://www.cnblogs.com/blueoverflow/p/4714470.html)

[mysql 视图不支持子查询的解决办法](http://dove19900520.iteye.com/blog/2309865)

[mysql创建视图中使用union报错1064](http://blog.csdn.net/u011974797/article/details/49618075)
