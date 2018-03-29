---
layout : post
title : mysql实现行转列
category : mysql
tags : mysql 行转列
---
* content
{:toc}

　　行转列在应对字典表这种key-value为行的表时非常好用，而实现行转列也比较简单，同样，列转行也是同样的实现方法。





#### 行转列应用场景

　　假设有一张成绩表score，其属性有id、studentName、className和score这几个字段。很明显，一个学生加一门科目就是唯一主键。

```sql
CREATE TABLE score(
	id int(11) not null AUTO_INCREMENT PRIMARY KEY,
	studentName varchar(40) not null DEFAULT '',
	className varchar(40) not null DEFAULT '',
	score int(11) not NULL DEFAULT 0 
);

INSERT score(studentName, className, score) 
VALUES
('张三', '语文', 80),
('李四', '语文', 90),
('王五', '语文', 84),
('张三', '数学', 90),
('李四', '数学', 90),
('王五', '数学', 69),
('王五', '英语', 83),
('李四', '英语', 73),
('张三', '英语', 93);
```

![score表](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/mysql-row-change-to-column-score-table.png?raw=true)

　　这时，如果我需要查询李四的所有科目的成绩，因为表里存了李四的三条记录，所以查询结果为三条。

```sql
SELECT * FROM score WHERE studentName = '李四';
```

![select1](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/mysql-row-change-to-column-score-select1.png?raw=true)

#### mysql行转列

　　上面的查询结果完全正确，但是某些时候，这样的结果不太好被程序处理。为了处理李四的成绩，程序需要使用循环来处理三条记录。如果这三条记录合并成一个记录，那么程序处理就方便很多了。

![select2](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/mysql-row-change-to-column-score-select2.png?raw=true)

　　如上图所示，这样，查询李四的成绩，就只有一行记录。sql如下：

```sql
SELECT
	studentName,
	MAX(CASE className WHEN '语文' THEN score END) 语文,
	MAX(CASE className WHEN '数学' THEN score END) 数学,
	MAX(CASE className WHEN '英语' THEN score END) 英语
FROM
	 score
GROUP BY
	studentName;
```

　　这是利用max函数来实现，另外也可以通过sum函数实现相同的效果。另外通过子查询或者存储过程也可以实现一样的结果，这里就不展示存储过程实现了，子查询实现行转列如下：

```sql
SELECT 
	a.studentName,
	(SELECT b.score FROM score b WHERE a.studentName = b.studentName AND b.className = '语文') 语文,
	(SELECT b.score FROM score b WHERE a.studentName = b.studentName AND b.className = '数学') 数学,
	(SELECT b.score FROM score b WHERE a.studentName = b.studentName AND b.className = '英语') 英语
FROM
	score a
GROUP BY
	a.studentName;
```

#### 为什么要使用max或者sum这种聚合函数

　　先来看看去掉max这种聚合函数后的查询效果。

```sql
SELECT
	studentName,
	CASE className WHEN '语文' THEN score END 语文,
	CASE className WHEN '数学' THEN score END 数学,
	CASE className WHEN '英语' THEN score END 英语
FROM
	 score
GROUP BY
	studentName;
```

![select3](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/mysql-row-change-to-column-score-select3.png?raw=true)

　　看了上面的查询结果，很明显不是我们想要的结果，那么为什么会出现上图的查询结果呢？我们把group by去掉看看。

```sql
SELECT
	studentName,
	CASE className WHEN '语文' THEN score END 语文,
	CASE className WHEN '数学' THEN score END 数学,
	CASE className WHEN '英语' THEN score END 英语
FROM
	 score
```

![select4](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/mysql-row-change-to-column-score-select4.png?raw=true)

　　通过上图应该会猜到原因。执行上面的sql时，查询score表中的第一条记录时，studentName为'张三'，className为'语文'，然后就被case语句转为score，即80。那么后面的两个case因为在这一行没被匹配到，因此输出为null。因此，原表中有9条记录，查询结果也就存在9条记录。加上group by之后，就会取每个studentName的第一条记录，则出现了上面只有语文有成绩，其他都为null的查询结果。

#### 参考

[mysql行转列转换](https://blog.csdn.net/sinat_27406925/article/details/77507478)

[MySQL 实现行转列SQL](https://blog.csdn.net/sxdtzhaoxinguo/article/details/55519171)

[mysql_行转列、列转行](http://x125858805.iteye.com/blog/2273503)