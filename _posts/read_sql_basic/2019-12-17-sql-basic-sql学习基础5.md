---
title: "sql基础学习5"
subtitle: "「sql」- basic"
layout: post
author: "Dxufe"
header-style: text
tags:
  - sql-basic
---

*Aggregate 函数
    Aggregate 函数的操作面向一系列的值，并返回一个单一的值。
*Scalar 函数

##### 1.sql-avg函数
*AVG 函数返回数值列的平均值。NULL 值不包括在计算中。
    SELECT Customer FROM Orders WHERE OrderPrice>(SELECT AVG(OrderPrice) FROM Orders)
    别写成~~SELECT Customer FROM Orders WHERE OrderPrice>AVG(OrderPrice)~~
##### 2.sql-count函数
*COUNT(column_name) 函数返回指定列的值的数目（NULL 不计入）:SELECT COUNT(column_name) FROM table_name

*COUNT(星) 函数返回表中的记录数：SELECT COUNT(*) FROM table_name

*COUNT(DISTINCT column_name) 函数返回指定列的不同值的数目：SELECT COUNT(DISTINCT column_name) FROM table_name

*SELECT COUNT(Customer) **AS CustomerNilsen** FROM Orders WHERE Customer='Carter'

#### 3.sql-first-last函数
*FIRST() 函数返回指定的字段中第一个记录的值。
*SELECT FIRST(column_name) FROM table_name
#
*LAST() 函数返回指定的字段中最后一个记录的值。
*SELECT LAST(column_name) FROM table_name

#### 4.sql-MAX-MIN函数

*MAX 函数返回一列中的最大值。NULL 值不包括在计算中。
*SELECT MAX(column_name) FROM table_name
*注释：MIN 和 MAX 也可用于文本列，以获得按字母顺序排列的最高或最低值。
*MIN 函数返回一列中的最小值。NULL 值不包括在计算中。
*SELECT MIN(column_name) FROM table_name**注释：MIN 和 MAX 也可用于文本列，以获得按字母顺序排列的最高或最低值。
#### 5.sql-SUM函数
*SUM 函数返回数值列的总数（总额）
*SELECT SUM(column_name) FROM table_name

##### 6.GROUP-BY
   **GROUP BY 语句用于结合合计函数(aggregate)，根据一个或多个列对结果集进行分组。**
![82c7c59799e5bab809e14b3ef6bf4bdd.png](en-resource://database/1108:1)
希望查找每个客户的总金额（总订单）：
`
SELECT Customer,SUM(OrderPrice) FROM Orders
GROUP BY Customer`
![07493c0b487a06fd8787cc24820c6582.png](en-resource://database/1110:1)

*如果不要group by
~~`
SELECT Customer,SUM(OrderPrice) FROM Orders`~~
![51ec745bc38fe5915a6a65a27671d79a.png](en-resource://database/1112:1)

**group by后面跟多个**
SELECT Customer,OrderDate,SUM(OrderPrice) FROM Orders
GROUP BY Customer,OrderDate

#### 7.HAVING
*在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与合计函数一起使用。
*现在，我们希望查找订单总金额少于 2000 的客户。
`
SELECT Customer,SUM(OrderPrice) FROM Orders
GROUP BY Customer
HAVING SUM(OrderPrice)<2000`

*现在我们希望查找客户 "Bush" 或 "Adams" 拥有超过 1500 的订单总金额。
`
SELECT Customer,SUM(OrderPrice) FROM Order
WHERE Customer='Bush' OR Customer='Adams'
GROUP BY Customer
HAVING SUM(OrderPrice)>1500`
#### 8 UCASE LCASE

*UCASE 函数把字段的值转换为大写。

`SELECT UCASE(column_name) FROM table_name
`
*LCASE 函数把字段的值转换为小写。
a
`SELECT LCASE(column_name) FROM table_name
`
#### 9 MID
*MID 函数用于从文本字段中提取字符。
`SELECT MID(column_name,start,length) FROM table_name`
![023b56d11f73a1bbf2febcb481f2db38.png](en-resource://database/1114:1)
#### 10 LEN

*LEN 函数返回文本字段中值的长度。

`SELECT LEN(column_name) FROM table_name
`![6db83da2e813c121d1e3401c0ea72354.png](en-resource://database/1116:1)

#### 11 ROUND
*ROUND 函数用于把数值字段舍入为指定的小数位数。
`    SELECT ROUND(column_name,decimals) FROM table_name
`![0ba4d292e18581c5719a43a91c196fda.png](en-resource://database/1118:1)
#### 12 FORMAT

*FORMAT 函数用于对字段的显示进行格式化。
  
`  SELECT FORMAT(column_name,format) FROM table_name
`![afb558d135565f0ca7903df885c476fb.png](en-resource://database/1120:1)


`*SELECT ProductName, UnitPrice,       FORMAT(Now(),'YYYY-MM-DD') as PerDate FROM Products`
![8b5b8608dbd5827f33e372fa73c03e13.png](en-resource://database/1122:1)

**SQL 是一种与数据库程序协同工作的标准语言，这些数据库程序包括 MS Access、DB2、Informix、MS SQL Server、Oracle、MySQL、Sybase 等等。**

#### 13 LIMIT
select * from tableName limit i,n 
*tableName：表名 
* i：为查询结果的索引值(默认从0开始)，当i=0时可省略i 
*n：为查询结果返回的数量 #
*i与n之间使用英文逗号","隔开
*表示从i 开始的n个



