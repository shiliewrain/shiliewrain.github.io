---
layout : post
title : "Mysql复习与学习（三）--having"
category : 数据库
tags : Mysql select having
---
* content
{:toc}

　　having子句一般是和group by组合使用的，在后面接一个聚合函数，对group by的分组结果进行筛选。当然，having也可以单独使用，但不推荐。





#### having子句

　　在select语句中，可以接where、group by、having和order by这四个子句，而这四个子句的书写顺序也是如此排列的。having一般接在group by后面，单独使用的意义不大。

#### 示例

　　存在一张记录水果名称、产地以及价格的表fruit，如下：

![fruit表内容](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-groupby-table.png?raw=true)

　　having只要是对group by的分组结果进行筛选，例如，筛选出水果种类不止一种的产地。

```sql
SELECT COUNT(name) AS 水果种类, product_place as 产地 from fruit GROUP BY product_place HAVING COUNT(name) > 1;
```

![查询结果1](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-having-result1.png?raw=true)

　　having可以单独使用，但不推荐。使用聚合函数，而只有一组查询结果，显然没有太多的意义。having也可以像where那样使用，结果没区别。

```sql
SELECT name AS 水果种类, product_place as 产地 from fruit  HAVING product_place != 'America';
```

![查询结果2](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-having-result2.png?raw=true)

　　where是用于筛选表或者视图，而having主要用于分组。

#### 总结

　　having主要通过聚合函数对分组后记录进行筛选，因此总是跟在group by后面，分组之前通常由where进行筛选。

#### 参考

[sql语句中where和having的区别](http://www.jb51.net/article/39034.htm)

[SQL HAVING用法详解](http://blog.csdn.net/qq_26562641/article/details/53301063)

[SQL语句中的Having子句与where子句](http://www.cnblogs.com/yuyutianxia/p/3832269.html)