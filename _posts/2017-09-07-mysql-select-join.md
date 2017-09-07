---
layout : post
title : "Mysql复习与学习（五）--连接join"
category : 数据库
tags : Mysql select join
---
* content
{:toc}

　　进行多表查询的时候会需要使用到join，常用的为inner join、right join和left join。我没有去画图解释，也懒得盗图，那干脆就在这里贴一篇参考吧。

　　[Mysql Join语法解析与性能分析](http://www.cnblogs.com/BeginMan/p/3754322.html)




### 连接

　　在需要同时查询两个或者更多表的数据的时候，就需要使用join去连接不同的表。而连接分为内连接（inner join）和外连接（outer join），外连接又分为右外连接（right [outer] join）和左外连接（left [outer] join），通过关键字on来建立连接条件。具体的解释看看前面的参考，应该就能理解得差不多了，那下面就只举一些例子吧。存在一张水果表和一张水果种类表，表内容如下：

![水果表](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-join-table1.png?raw=true)
![水果种类表](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-join-table2.png?raw=true)

#### 内连接

　　内连接，关键字为inner join，查询出同时符合条件的连接两表的数据。

```sql
-- 查询两表中水果名相同的数据
SELECT * FROM fruit INNER JOIN fruit_cate ON fruit.name = fruit_cate.name;
```

![查询结果1](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-join-result1.png?raw=true)

　　在inner join中，可以不加连接条件，查找出来的则是两个表的笛卡尔积，与cross join相同，这里不举示例。

#### 外连接

　　外连接分为左外连接和右外连接。左外连接查询出左表所有满足where后查询条件的数据，然后右表进行匹配，匹配不到则值为null，右外连接则反之。

select * from table1(左表) left/right join table2(右表) on table1.col = table2.col where ...

##### 左外连接

　　查询出左表所有符合条件的字段，右表进行匹配。示例如下：

```sql
SELECT * FROM fruit LEFT JOIN fruit_cate ON fruit.name = fruit_cate.name;

SELECT * FROM fruit LEFT JOIN fruit_cate ON fruit.name = fruit_cate.name WHERE price > 5;
```

![查询结果2](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-join-result2.png?raw=true)
![查询结果3](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-join-result3.png?raw=true)

　　由上面的结果可看出，左外连接会先根据查询条件查询出左表fruit的数据，然后根据连接条件，右表fruit_cate进行匹配，右表匹配不上则值为null。

##### 右外连接

　　右外连接和左外连接正好相反，先查出右表的数据，然后左表进行匹配，匹配不上则值为null。示例如下：

```sql
SELECT * FROM fruit RIGHT JOIN fruit_cate ON fruit.name = fruit_cate.name;
```

![查询结果4](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-join-result4.png?raw=true)

### 总结

　　内连接取交集，外连接分左右，左外取左表，右外取右表。