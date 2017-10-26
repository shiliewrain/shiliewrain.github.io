---
layout : post
title : "mysql外连where和on的区别"
category : 数据库
tags : mysql leftjoin where on
---
* content
{:toc}

　　本来昨天准备研究一下mysql中where和on的区别，结果硬生生改了几乎两天的bug，而今天也由一个bug正好遇到了left join外连关于on和where条件的选择的问题，感觉差不多清楚了，记录一下。




### where和on

　　在使用mysql数据库时，使用left join或者right join都必须要使用关键字on，否则就sql执行就报错。可以这样理解：on是建立外连的桥，两张表如何连接就靠on后面的条件。因为外连分主次表，数据以主表为基础，次表对应连接，如果没有on来建立连接，那么次表的数据就不知道如何对应上主表。而内连，又称为直连，不存在主次表之分，取得是两者的交集，因此不需要on。

　　对于where和on的理解，可以简单表述为：on是建立主次表联系的，on后面的条件都只为了筛选次表中数据的。where是对sql语句中from与where之间的结果集进行筛选，也是对最后的结果集进行筛选，再之后就只剩分组和排序了。需要注意的是，也只有where中的条件能对主表中的数据进行筛选。

#### 只有where中存在主表筛选条件

　　例如，```sql SELECT a.id AS AID, b.id AS BID FROM a LEFT JOIN b ON a.id = b.id WHERE a.id < 5``` 

　　上面的sql使用的是left join，以左表，即a表为主表。那么这句sql大概的执行顺序如下：

1. 根据on后面的条件，找到b表中id能对应到a表的数据，将其连接在对应的a表数据中，

2. 根据where后面的条件，对第一步连接后的结果集进行筛选。

　　因此，结果集会输出a表中所有id小于5的数据，b表对应相应的id。b表中多条对应同一a.id的输出多条结果，b表中无对应的a.id的输出一条结果，b表输出结果为null。

![示例1](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/on_where_1.png?raw=true)

#### 只有on中存在主表筛选条件

　　例如，```sql SELECT a.id AS AID, b.id AS BID FROM a LEFT JOIN b ON a.id = b.id AND a.id < 3```

　　因为on中的条件只是筛选b表用的，因此给b用来匹配的id为a.id小于3的。因为没有where对外连之后的结果集进行筛选，因此会输出a表的所有数据。

![示例2](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/on_where_2.png?raw=true)

#### 只有on中存在次表筛选条件

　　例如，```sql SELECT a.id AS AID, b.id AS BID FROM a LEFT JOIN b ON a.id = b.id AND b.id < 3```

　　on后面跟着对b表的两个条件，因此会将满足b.id小于3的数据与a外连。

![示例3](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/on_where_3.png?raw=true)

#### 只有where中存在次表的筛选条件

　　例如，```sql SELECT a.id AS AID, b.id AS BID FROM a LEFT JOIN b ON a.id = b.id WHERE b.id < 3```

　　where后面的条件是对最后的结果集进行筛选，因此会在ab外连之后，筛选出b.id小于3的数据。

![示例4](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/on_where_4.png?raw=true)

### 总结

　　在真实的项目中，sql语句不会这么的简单，但是记住on是对次表条件进行筛选，where是对最后的结果进行筛选，就能正确地写出sql，获得自己预期的结果。

### 参考

[mysql left( right ) join使用on 与where 筛选的差异](http://xianglp.iteye.com/blog/868957)

[MySQL关联left join 条件on与where不同](http://database.51cto.com/art/201005/200521.htm)
