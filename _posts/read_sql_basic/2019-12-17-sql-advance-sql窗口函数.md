---
title: "sql窗口函数使用总结"
subtitle: "「sql」- 窗口函数"
layout: post
author: "Dxufe"
header-style: text
tags:
  - sql-advance
---
## 1.简介  
窗口函数也就是在满足某种条件的记录集合上执行的特殊函数。对于每条记录都要在此窗口内执行函数，有的函数随着记录不同，窗口大小都是固定的，这种属于静态窗口；有的函数则相反，不同的记录对应着不同的窗口，这种动态变化的窗口叫滑动窗口。简单的说**窗口函数就是对于查询的每一行，都使用与该行相关的行进行计算**。窗口函数和普通聚合函数很容易混淆，二者区别如下：
    聚合函数是将多条记录聚合为一条；而窗口函数是每条记录都会执行，有几条记录执行完还是几条。
    聚合函数也可以用于窗口函数中。    

## 2.序号函数：Row_Number,Rank，Dense_Rank  

名称 | 用法 
---- | ----
row_number() | 从1开始，按照顺序,row_number()的值不会存在重复,当排序的值相同时,按照表中记录的顺序进行排列;通常用于获取分组内排序第一的记录等。
rank() | 生成数据项在分组中的排名，数据项相等的排名相同，排名相等会在名次中留下空位。
dense_rank() | 生成数据项在分组中的排名，数据项相等的排名相同，排名相等会在名次中不会留下空位。
**示例**  
```
SELECT
	province,
	issuer_ful,
	financing_amount,
	row_number ( ) over ( PARTITION BY province ORDER BY financing_amount DESC ) AS rn1,
	rank ( ) over ( PARTITION BY province ORDER BY financing_amount DESC ) AS rn2,
	Dense_Rank ( ) over ( PARTITION BY province ORDER BY financing_amount DESC ) AS rn3 
FROM
	sse_kcbinfo_2019_08_16_15_17_17;
```
**结果**  
![avatar](/img/sql_number.png)  

## 3.窗口子句  
当前分区的一个子集，在分区里面再进一步细分窗口，子句用来定义子集的规则，通常用来作为滑动窗口使用。通常使用rows BETWEEN frame_start AND frame_end语法来表示行范围.
![avatar](/img/sql_window.png)  
reference:https://yq.aliyun.com/articles/719989    
```
CURRENT ROW 边界是当前行，一般和其他范围关键字一起使用
UNBOUNDED PRECEDING 边界是分区中的第一行
UNBOUNDED FOLLOWING 边界是分区中的最后一行
expr PRECEDING  当前行之前的expr(数字或表达式)行
expr FOLLOWING  当前行之后的expr(数字或表达式)行
```
**示例**  
```
SELECT
	province,
	issuer_ful,
	financing_amount,
	sum( financing_amount ) over ( PARTITION BY province ) f1,
	sum( financing_amount ) over ( PARTITION BY province ORDER BY financing_amount DESC ) f2,
	sum( financing_amount ) over ( PARTITION BY province ORDER BY financing_amount DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW ) f3,
	sum( financing_amount ) over ( PARTITION BY province ORDER BY financing_amount DESC ROWS BETWEEN 3 PRECEDING AND CURRENT ROW ) f4,
	sum( financing_amount ) over ( PARTITION BY province ORDER BY financing_amount DESC ROWS BETWEEN CURRENT ROW AND 3 FOLLOWING ) f5,
	sum( financing_amount ) over ( PARTITION BY province ORDER BY financing_amount DESC ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING ) f6,
	sum( financing_amount ) over ( PARTITION BY province ORDER BY financing_amount DESC ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING ) f7 
FROM
	sse_kcbinfo_2019_08_16_15_17_17;
```
**结果**   
![avatar](/img/sql_windowuse.png)  
分别是分组内所有行，分组内值在第一行到当前行的所有行（RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW），分组内第一行到当前行，分组内当前行+往前三行，分组内当前行+往后三行，分组内当前行+往前三行+往后三行，分组内当前行到最后一行。

## 4.分布函数：PERCENT_RANK()、CUME_DIST()  
