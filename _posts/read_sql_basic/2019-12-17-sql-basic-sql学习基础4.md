---
title: "sql基础学习4"
subtitle: "「sql」- basic"
layout: post
author: "Dxufe"
header-style: text
tags:
  - sql-basic
---
##### 1.date 函数  
*mysql*  
![avatar](/img/mysql_datefunction.png)  
```
SELECT
	NOW( ),CURDATE( ),CURTIME( ),
	date( now( ) ),
	extract( YEAR FROM now( ) ),extract( MONTH FROM now( ) ),extract( DAY FROM now( ) ),
	extract( HOUR FROM now( ) ),extract( MINUTE FROM now( ) ),
	date_add(now(),interval 2 year),date_add(now(),interval 2 month),
	date_add(now(),interval 2 day),date_sub(now(),interval 2 hour),
	date_sub(now(),interval 2 minute),date_sub(now(),interval 2 second),
	datediff(now(),date_add(now(),interval 2 day)),#只有值的日期部分参与计算,且是第一个日期减去第二个日期
date_format(now(),'%Y-%m-%d %u-%W %H:%i:%S') #年月日 第几周星期几 时分秒 
```  
![avatar](/img/mysql_dateresult.png)  
*sql-server*
![avatar](/img/sql-server-date.png)  
##### 2.NULL
   如果表中的某个列是可选的，那么我们可以在不向该列添加值的情况下插入新记录或更新已有的记录。这意味着该字段将<u>以 NULL 值保存</u>。
 **  无法使用比较运算符来测试 NULL 值，比如 =, <, 或者 <>。** 始终使用 IS NULL 来查找 NULL 值。IS NOT NULL查找非NULL值 **
    当希望null为0时，SELECT ProductName,UnitPrice*(UnitsInStock+<u>IFNULL</u>(UnitsOnOrder,0)) FROM Products
##### 3.mysql数据类型  
**text**  
![avatar](/img/mysql_text.png)  
**number**  
![avatar](/img/mysql_number.png)  
 **date**  
![avatar](/img/mysql_dateformat.png )
 **datetime 和timestamp的区别**  
 即便 DATETIME 和 TIMESTAMP 返回相同的格式，它们的工作方式很不同。在 INSERT 或 UPDATE 查询中，TIMESTAMP 自动把自身设置为当前的日期和时间。TIMESTAMP 也接受不同的格式，比如 YYYYMMDDHHMMSS、YYMMDDHHMMSS、YYYYMMDD 或 YYMMDD。  
 ##### 3.sql server数据类型  
**char**   
![avatar](/img/sql_charformat.png)  
*number**   
![avatar](/img/sql_numberformat.png)  
**date**   
![avatar](/img/sql_dateformat.png)  
**数据库管理系统**  
![avatar](/img/sql_RDBMS.png)


**reference**  
https://www.w3school.com.cn/sql/sql_server.asp  
https://www.w3school.com.cn/sql/sql_datatypes.asp  
https://www.w3school.com.cn/sql/sql_dates.asp
