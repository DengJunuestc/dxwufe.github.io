---
title: "mysql行列转换"
subtitle: "「mysql」- 行列转换"
layout: post
author: "WynnZuo"
header-style: text
tags:
  - mysql
---


### 适用场景
在一行数据里，同一个列中存储了多个属性值，需要将此行的多属性列依据分隔符分隔，
得到多行的结果，每行的被分隔列只存储单一属性。
#### 建表语句
```
DROP TABLE IF EXISTS tbl_name;
	
CREATE TABLE tbl_name 
( 
 id INT ( 11 ) NOT NULL auto_increment, 
 userName VARCHAR ( 100 ) NOT NULL, 
 PRIMARY KEY ( id ) 
) 
ENGINE = INNODB AUTO_INCREMENT = 2 DEFAULT CHARSET = utf8;

INSERT INTO tbl_name VALUES ( 1, 'a,aa,aaa' ),( 2, 'b,bb' ),( 3, 'c,cc' );
```
![img](/img/in-post/post-wynn/line-to-list1.png)
#### 查询语句
```
SELECT
	a.id,
	SUBSTRING_INDEX( SUBSTRING_INDEX( a.userName, ',', b.help_topic_id + 1 ), ',',- 1 ) AS NAME 
FROM
	tbl_name a
	LEFT JOIN mysql.help_topic b ON b.help_topic_id < ( LENGTH( a.userName ) - LENGTH( REPLACE ( a.userName, ',', '' ) ) + 1 ) 
ORDER BY
	a.id;
```
**查询结果**  
![img](/img/in-post/post-wynn/line-to-list2.png)
#### 分析
```
SELECT
	a.id,
	b.help_topic_id + 1 num,
	SUBSTRING_INDEX( a.userName, ',', b.help_topic_id + 1 ) AS NAME_tmp,
	SUBSTRING_INDEX( SUBSTRING_INDEX( a.userName, ',', b.help_topic_id + 1 ), ',',- 1 ) AS NAME 
FROM
	tbl_name a
	LEFT JOIN mysql.help_topic b ON b.help_topic_id < ( LENGTH( a.userName ) - LENGTH( REPLACE ( a.userName, ',', '' ) ) + 1 ) 
ORDER BY
	a.id;
```
**分析结果**  
![img](/img/in-post/post-wynn/line-to-list3.png)  
**拆分数量**
```
LENGTH( a.userName ) - LENGTH( REPLACE ( a.userName, ',', '' ) ) + 1
```
#### 其他
需要一个拥有连续数列的独立表，并且连续数列的最大值一定要大于符合分割的值的个数。当然，mysql内部也有现成的连续数列表可用。
如mysql.help_topic 其中 help_topic_id 共有504个数值，一般能满足于大部分需求了。

