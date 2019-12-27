---
title: "sql基础学习6"
subtitle: "「sql」- basic"
layout: post
author: "Dxufe"
header-style: text
tags:
  - sql-basic
---
#### 1. sql 定义变量 
```
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
**DECLARE P INT;
SET P=N-1;**
  RETURN (
      # Write your MySQL query statement below.
SELECT IFNULL((SELECT DISTINCT Salary FROM Employee ORDER BY Salary DESC LIMIT P,1),NULL) AS getNthHighestSalary
  );
END
```
#### 2.:=和=的区别(slect :=赋值的作用)
**=**
只有在set和update时才是和:=一样，赋值的作用，其它都是等于的作用。鉴于此，用变量实现行号时，必须用:=
**:=**
不只在set和update时时赋值的作用，在select也是赋值的作用。
如果明白了=和:=的区别，那么也就理解了下边的现象。
@num:=@num+1,:=是赋值的作用，所以，先执行@num+1,然后再赋值给@num，所以能正确实现行号的作用。
```
# Write your MySQL query statement below
SELECT
	P.Score,
	@RANKK := @RANKK + 1 AS Rank 
FROM
	Scores AS P,
	(SELECT @RANKK := 0) AS Q 
ORDER BY Score DESC
```
**注意from 里面有多个的时候要指定别名**
