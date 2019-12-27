---
title: "sql基础学习4"
subtitle: "「sql」- 基础学习"
layout: post
author: "Dxufe"
header-style: text
tags:
  - sql
---
##### 1.date

##### 2.NULL
   如果表中的某个列是可选的，那么我们可以在不向该列添加值的情况下插入新记录或更新已有的记录。这意味着该字段将<u>以 NULL 值保存</u>。
 **  无法使用比较运算符来测试 NULL 值，比如 =, <, 或者 <>。** 始终使用 IS NULL 来查找 NULL 值。IS NOT NULL查找非NULL值 **
    当希望null为0时，SELECT ProductName,UnitPrice*(UnitsInStock+<u>IFNULL</u>(UnitsOnOrder,0)) FROM Products
##### 3.mysql数据类型
** text**

**number**
 ![9c29c3a9a11a488a4cf1a13724d88a14.png](en-resource://database/1000:1)
 **date**
 ![9dc1af9197913f3f2d4b702acc90bb7c.png](en-resource://database/1002:1)
 
 ** datetime 和timestamp的区别
 
* 即便 DATETIME 和 TIMESTAMP 返回相同的格式，它们的工作方式很不同。在 INSERT 或 UPDATE 查询中，TIMESTAMP 自动把自身设置为当前的日期和时间。TIMESTAMP 也接受不同的格式，比如 YYYYMMDDHHMMSS、YYMMDDHHMMSS、YYYYMMDD 或 YYMMDD。
* 数据库管理系统
![065886d5163a10c7784c817adfe1b451.png](en-resource://database/1004:1)
