---
layout : post
title : "Mysql复习与学习（四）--order by"
category : 数据库
tags : Mysql select orderby
---
* content
{:toc}

　　order by子句用于对查询的结果集进行排序，在order by后面可以接多个需要排序的字段。




#### order by子句

　　在select语句中，可以接where、group by、having和order by这四个子句，而这四个子句的书写顺序也是如此排列的。order by后面接排序的字段，形如```sql
select * from table order by [column1] asc, [column2] desc;```。

#### 示例

　　存在一张记录水果名称、产地以及价格的表fruit，如下：

![fruit表内容](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-groupby-table.png?raw=true)

　　order by子句对查询结果集进行排序，asc表示升序，desc表示降序，不写默认为asc。

```sql
SELECT * FROM fruit ORDER BY product_place DESC;

SELECT * FROM fruit ORDER BY price;
```
![查询结果1](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-order-by-result1.png?raw=true)
![查询结果2](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-order-by-result2.png?raw=true)

　　多字段按从左到右顺序依次进行排序，左边字段的优先级高，右边字段只能在左边字段值相等的情况下局部排序。

```sql
SELECT * FROM fruit ORDER BY product_place DESC, price ASC;
```

![查询结果3](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/select-order-by-result3.png?raw=true)

　　从上图可以看出，结果集首先按照product_place进行降序排序，然后在此基础上对product_place中相同的值按照price进行升序排序。

#### 总结

　　order by子句对查询结果集进行排序，默认为asc升序，使用desc为降序。

#### 参考

[order by用法](http://blog.csdn.net/zxcvg/article/details/6670895)

[MySQL Order by 语句用法与优化详解](http://www.jb51.net/article/38953.htm)