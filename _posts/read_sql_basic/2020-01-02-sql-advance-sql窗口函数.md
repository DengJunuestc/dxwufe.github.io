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
*row_number()* : 从1开始，按照顺序,row_number()的值不会存在重复,当排序的值相同时,按照表中记录的顺序进行排列;通常用于获取分组内排序第一的记录等。  
*rank()* : 生成数据项在分组中的排名，数据项相等的排名相同，排名相等会在名次中留下空位。  
*dense_rank()* : 生成数据项在分组中的排名，数据项相等的排名相同，排名相等会在名次中不会留下空位。

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

## 3.窗口子句-与聚合函数一起用  
当前分区的一个子集，在分区里面再进一步细分窗口，子句用来定义子集的规则，通常用来作为滑动窗口使用。通常使用rows BETWEEN frame_start AND frame_end语法来表示行范围.（reference:https://yq.aliyun.com/articles/719989）      
```
CURRENT ROW 边界是当前行，一般和其他范围关键字一起使用
UNBOUNDED PRECEDING 边界是分区中的第一行
UNBOUNDED FOLLOWING 边界是分区中的最后一行
expr PRECEDING  当前行之前的expr(数字或表达式)行
expr FOLLOWING  当前行之后的expr(数字或表达式)行
```  
![avatar](/img/sql_window.png)  
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
![avatar](/img/sql_windowuse.png)  
分别是分组内所有行，分组内值在第一行到当前行的所有行（RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW），分组内第一行到当前行，分组内当前行+往前三行，分组内当前行+往后三行，分组内当前行+往前三行+往后三行，分组内当前行到最后一行。

## 4.分布函数：PERCENT_RANK()、CUME_DIST()    
*CUME_DIST()* :当前值之前（包括等于当前值）的行数/分组内总行数。  
*PERCENT_RANK()* :分组内当前行的RANK值-1/分组内总行数-1  
*注意不支持window子句，要指定order by*  
**示例**  
```
SELECT
	province,
	issuer_ful,
	financing_amount,
	cume_dist() over ( PARTITION BY province ORDER BY financing_amount DESC ) f1,
	percent_rank() over ( PARTITION BY province ORDER BY financing_amount DESC ) f2
FROM
	sse_kcbinfo_2019_08_16_15_17_17;
```  
![avatar](/img/sql_fenbu.png)  
分别表示分组内大于等于当前行的值的行数/分组内的总行数；大于等于当前行的值的行数-1/分组内的总行数-1

## 5.前后函数：LAG()、LEAD()，返回上下行指定字段的数据
*LAG(col,n,DEFAULT)* ：获取当前数据行按照某种排序规则的上n行数据的某个字段，第一个参数为列名，第二个参数为往上第n行（可选，默认为1），第三个参数为默认值（当往上第n行为NULL时候，取默认值，如不指定，则为NULL）。  
*LEAD(col,n,DEFAULT)* ：获取当前数据行按照某种排序规则的下n行数据的某个字段，第一个参数为列名，第二个参数为往下第n行（可选，默认为1），第三个参数为默认值（当往下第n行为NULL时候，取默认值，如不指定，则为NULL）。  
**示例**  
```
SELECT
	province,
	issuer_ful,
	financing_amount,
	lead ( issuer_ful, 2, '没有公司' ) over ( PARTITION BY province ORDER BY financing_amount DESC ) "后面的公司",
	lag ( issuer_ful, 2, '没有公司' ) over ( PARTITION BY province ORDER BY financing_amount DESC ) "前面的公司" 
FROM
	sse_kcbinfo_2019_08_16_15_17_17;
```  
![avatar](/img/sql_qianhou.png)  
分别表示按province分组后，按financing_amount降序排列，获取每个公司其分组内融资额比自己小两个位置的公司，以及多两个位置的公司。

## 6.头尾函数：FIRST_VALUE()、LAST_VALUE()，NTH_VALUE()
*FIRST_VALUE()*:取分组内排序后，截止到当前行，第一个行的列值  
*LAST_VALUE()*:取分组内排序后，截止到与当前行的排序字段的值相等的最后一个行的列值  
*NTH_VALUE(column,n)*:去分组内排序后，截止到当前行的第n行指定字段的值 
**示例**  
```
SELECT
	province,
	`update`,
	financing_amount,
	first_value ( financing_amount ) over ( PARTITION BY province ORDER BY `update` DESC ) v1,
	last_value ( financing_amount ) over ( PARTITION BY province ORDER BY `update` DESC ,financing_amount desc) v2,
	first_value ( financing_amount ) over ( PARTITION BY province ORDER BY `update`  ) v3,
	last_value ( financing_amount ) over ( PARTITION BY province ORDER BY `update`  ) v4,	
	NTH_VALUE(financing_amount,3) over ( PARTITION BY province ORDER BY `update` desc ) v5
FROM
	sse_kcbinfo_2019_08_16_15_17_17 order by province,`update` desc;
```  
![avatar](/img/sql_shouwei.png)

## 7.其他函数：NTILE()
*NTILE(n)* :用于将分组数据按照指定排序字段的顺序切分成n片，返回当前切片值，如果切片不均匀，默认增加第一个切片的分布。NTILE不支持ROWS BETWEEN 。
**示例** 
```
SELECT
	province,
	`update`,
	financing_amount,
	ntile(3) over ( PARTITION BY province ORDER BY `update` DESC ) nt1
FROM
	sse_kcbinfo_2019_08_16_15_17_17;
```  
![avatar](/img/sql_fenzu.png)



