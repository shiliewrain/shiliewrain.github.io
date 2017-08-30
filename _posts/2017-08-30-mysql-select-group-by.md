---
layout : post
title : "Mysql复习与学习（二）--group by"
category : 数据库
tags : Mysql select groupby
---
* content
{:toc}

　　group by是在select语句中经常被用到的关键字，将查询结果按某种条件进行分组。要注意的一点是select返回的字段必须被group by或者聚合函数包含的字段。




#### group by子句

　　在select语句中，可以接where、group by、having和order by这四个子句，而这四个子句的书写顺序也是如此排列的，在存在where的情况下，group by要写在where后面。在group by后面可以书写多个分组字段，用“,”分隔即可。

#### 示例

　　存在一张记录水果名称、产地以及价格的表fruit，如下：

![fruit表内容](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-groupby-table.png?raw=true)

　　这里可以理解一下为何select查询的字段必须被group by或者聚合函数包含了。一种水果可能有多个产地，一个产地也会出多种水果。按产地或者水果进行分组的时候，都会出现一对多的情况。例如按产地分组查找水果，“China”会出现三条记录：“Apple”、“Orange”和“Banana”。一条“China”记录如何存放三种水果记录呢，这时不同的数据库，查询可能会报错，也可能就显示其中一条记录。聚合函数就是将多条记录聚合为一条，因此可以和分组的记录进行一一对应。

　　按产地查询水果种类数量，sql如下：

```sql
SELECT COUNT(name) AS 水果种类, product_place as 产地 from fruit GROUP BY product_place;
```

![查询结果1](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-groupby-result1.png?raw=true)

　　如果不用聚合函数包含name，sql如下：

```sql
SELECT name AS 水果种类, product_place as 产地 from fruit GROUP BY product_place;
```

![查询结果2](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-groupby-result2.png?raw=true)

　　在Mysql中，可以看到显示的都是按product_place分组的第一条记录。在group by后面再加上name之后，就是按产地和水果种类进行分组。

```sql
-- 主分组为产地
SELECT name AS 水果种类, product_place as 产地 from fruit GROUP BY product_place, name;

-- 主分组为水果名称
SELECT name AS 水果种类, product_place as 产地 from fruit GROUP BY name, product_place;
```

![查询结果3](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-groupby-result3.png?raw=true)
![查询结果4](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-groupby-result4.png?raw=true)

　　由上面两句sql及查询结果可以看出，group by子句中，写在前面的字段为主要分组条件，结果集中的记录按照其分组显示。

#### 总结

　　在where存在的情况下，group by跟在where后面。select查询字段的字段必须被group by或者聚合函数包含。group by后面可以接多个分组字段，按书写顺序分主次。

#### 参考

[SQL语句：Group By总结](http://www.cnblogs.com/glaivelee/archive/2010/11/19/1881381.html)

[mysql group by 用法解析(详细)](http://blog.csdn.net/xxpyeippx/article/details/8059910)