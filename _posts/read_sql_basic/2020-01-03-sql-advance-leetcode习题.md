---
title: "LeetCode部分习题总结"
subtitle: "「sql」- casewhenthenelseend~round~if~复用表子查询"
layout: post
author: "Dxufe"
header-style: text
tags:
  - sql-advance
---
##### 1查询非禁止用户取消率  
``` 
Create table If Not Exists Trips (Id int, Client_Id int, Driver_Id int, City_Id int, Status ENUM('completed', 'cancelled_by_driver', 'cancelled_by_client'), Request_at varchar(50));
Create table If Not Exists Users (Users_Id int, Banned varchar(50), Role ENUM('client', 'driver', 'partner'));
Truncate table Trips;
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('1', '1', '10', '1', 'completed', '2013-10-01');
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('2', '2', '11', '1', 'cancelled_by_driver', '2013-10-01');
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('3', '3', '12', '6', 'completed', '2013-10-01');
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('4', '4', '13', '6', 'cancelled_by_client', '2013-10-01');
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('5', '1', '10', '1', 'completed', '2013-10-02');
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('6', '2', '11', '6', 'completed', '2013-10-02');
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('7', '3', '12', '6', 'completed', '2013-10-02');
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('8', '2', '12', '12', 'completed', '2013-10-03');
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('9', '3', '10', '12', 'completed', '2013-10-03');
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('10', '4', '13', '12', 'cancelled_by_driver', '2013-10-03');
Truncate table Users;
insert into Users (Users_Id, Banned, Role) values ('1', 'No', 'client');
insert into Users (Users_Id, Banned, Role) values ('2', 'Yes', 'client');
insert into Users (Users_Id, Banned, Role) values ('3', 'No', 'client');
insert into Users (Users_Id, Banned, Role) values ('4', 'No', 'client');
insert into Users (Users_Id, Banned, Role) values ('10', 'No', 'driver');
insert into Users (Users_Id, Banned, Role) values ('11', 'No', 'driver');
insert into Users (Users_Id, Banned, Role) values ('12', 'No', 'driver');
insert into Users (Users_Id, Banned, Role) values ('13', 'No', 'driver');
```    
Users 表存所有用户。每个用户有唯一键 Users_Id。Banned 表示这个用户是否被禁止，Role 则是一个表示（‘client’, ‘driver’, ‘partner’）的枚举类型。Trips 表中存所有出租车的行程信息。每段行程有唯一键 Id，Client_Id 和 Driver_Id 是 Users 表中 Users_Id 的外键。Status 是枚举类型，枚举成员为 (‘completed’, ‘cancelled_by_driver’, ‘cancelled_by_client’)。  
写一段 SQL 语句查出 2013年10月1日 至 2013年10月3日 期间非禁止用户的取消率。取消率（Cancellation Rate）保留两位小数。
```
SELECT
	NOBAN.Request_at AS 'Day',
	ROUND( COUNT( IF ( NOBAN.STATUS LIKE 'cancell%', 1, NULL ) ) / COUNT( * ), 2 ) AS 'Cancellation Rate' 
FROM
	(
SELECT
	* 
FROM
	Trips 
WHERE
	Client_Id IN ( SELECT Users_Id FROM Users WHERE Banned = 'No' ) 
	AND Driver_Id IN ( SELECT Users_Id FROM Users WHERE Banned = 'No' ) 
	AND ( Request_at BETWEEN '2013-10-01' AND '2013-10-03' ) 
	) NOBAN GROUP BY Request_at
```  
##### 2.第二高的薪水，编写一个 SQL 查询，获取 Employee 表中第二高的薪水（Salary） 。如果不存在第二高的薪水，那么查询应返回 null。  
```
Create table If Not Exists Employee (Id int, Salary int);
Truncate table Employee;
insert into Employee (Id, Salary) values ('1', '100');
insert into Employee (Id, Salary) values ('2', '200');
insert into Employee (Id, Salary) values ('3', '300');
```  
``` 
select IFNULL((select Salary from Employee order by Salary DESC LIMIT 1,1),null) AS SecondHighestSalary 
```    	
##### 3.编写一个 SQL 查询，获取 Employee 表中第 n 高的薪水（Salary）。  
```
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
DECLARE P INT;
SET P=N-1;
  RETURN (
    select IFNULL((select DISTINCT Salary from Employee order by Salary DESC LIMIT P,1),null) AS getNthHighestSalary   );
END
)
```  
##### 4.编写一个 SQL 查询来实现分数排名。如果两个分数相同，则两个分数排名（Rank）相同。请注意，平分后的下一个名次应该是下一个连续的整数值。换句话说，名次之间不应该有“间隔”。  
```  
Create table If Not Exists Scores (Id int, Score DECIMAL(3,2));
Truncate table Scores;
insert into Scores (Id, Score) values ('1', '3.5');
insert into Scores (Id, Score) values ('2', '3.65');
insert into Scores (Id, Score) values ('3', '4.0');
insert into Scores (Id, Score) values ('4', '3.85');
insert into Scores (Id, Score) values ('5', '4.0');
insert into Scores (Id, Score) values ('6', '3.65');
```  
```
SELECT
	s.Score,
	( SELECT COUNT( DISTINCT Score ) FROM Scores WHERE Score >= s.Score ) AS Rank 
FROM
	Scores s 
ORDER BY
	Score DESC
```  	
##### 5.编写一个 SQL 查询，查找所有至少连续出现三次的数字。  
```
Create table If Not Exists Logs (Id int, Num int);
Truncate table Logs;
insert into Logs (Id, Num) values ('1', '1');
insert into Logs (Id, Num) values ('2', '1');
insert into Logs (Id, Num) values ('3', '1');
insert into Logs (Id, Num) values ('4', '2');
insert into Logs (Id, Num) values ('5', '1');
insert into Logs (Id, Num) values ('6', '2');
insert into Logs (Id, Num) values ('7', '2');
```  
```  
select distinct l.Num as ConsecutiveNums from logs l join logs ll on (l.Num=ll.Num and l.Id =ll.Id-1) join logs lll on (lll.Num=ll.Num and ll.Id =lll.Id-1)
```    
##### 6. Employee 表包含所有员工，他们的经理也属于员工。每个员工都有一个 Id，此外还有一列对应员工的经理的 Id。给定 Employee 表，编写一个 SQL 查询，该查询可以获取收入超过他们经理的员工的姓名。  
```
Create table If Not Exists Employee (Id int, Name varchar(255), Salary int, ManagerId int);
Truncate table Employee;
insert into Employee (Id, Name, Salary, ManagerId) values ('1', 'Joe', '70000', '3');
insert into Employee (Id, Name, Salary, ManagerId) values ('2', 'Henry', '80000', '4');
insert into Employee (Id, Name, Salary, ManagerId) values ('3', 'Sam', '60000', 'None');
insert into Employee (Id, Name, Salary, ManagerId) values ('4', 'Max', '90000', 'None');
```  
```
SELECT e1.Name AS Employee FROM Employee e1 JOIN Employee e2 ON e1.ManagerId=e2.Id AND  e1.Salary >e2.Salary
```  
##### 7.编写一个 SQL 查询，查找 Person 表中所有重复的电子邮箱。  
```
select a.Email from Person a join Person b on a.Id<>b.Id and a.Email =b.Email
```  
##### 8.某网站包含两个表，Customers 表和 Orders 表。编写一个 SQL 查询，找出所有从不订购任何东西的客户。  
```
Create table If Not Exists Customers (Id int, Name varchar(255));
Create table If Not Exists Orders (Id int, CustomerId int);
Truncate table Customers;
insert into Customers (Id, Name) values ('1', 'Joe');
insert into Customers (Id, Name) values ('2', 'Henry');
insert into Customers (Id, Name) values ('3', 'Sam');
insert into Customers (Id, Name) values ('4', 'Max');
Truncate table Orders;
insert into Orders (Id, CustomerId) values ('1', '3');
insert into Orders (Id, CustomerId) values ('2', '1');
```   
```
select Name as Customers from Customers where Id not in (select CustomerId from Orders)
```  
##### 9.Employee 表包含所有员工信息，每个员工有其对应的 Id, salary 和 department Id。Department 表包含公司所有部门的信息。编写一个 SQL 查询，找出每个部门工资最高的员工。  
```
Create table If Not Exists Employee (Id int, Name varchar(255), Salary int, DepartmentId int);
Create table If Not Exists Department (Id int, Name varchar(255));
Truncate table Employee;
insert into Employee (Id, Name, Salary, DepartmentId) values ('1', 'Joe', '70000', '1');
insert into Employee (Id, Name, Salary, DepartmentId) values ('2', 'Jim', '90000', '1');
insert into Employee (Id, Name, Salary, DepartmentId) values ('3', 'Henry', '80000', '2');
insert into Employee (Id, Name, Salary, DepartmentId) values ('4', 'Sam', '60000', '2');
insert into Employee (Id, Name, Salary, DepartmentId) values ('5', 'Max', '90000', '1');
Truncate table Department;
insert into Department (Id, Name) values ('1', 'IT');
insert into Department (Id, Name) values ('2', 'Sales');
```
```
ELECT D.NAME AS Department,
E.NAME AS Employee,
E.Salary 
FROM
	Employee E
	JOIN ( SELECT DepartmentId, max( Salary ) AS salary FROM Employee GROUP BY DepartmentId ) MD ON E.DepartmentId = MD.DepartmentId 
	AND E.Salary = MD.salary
	JOIN Department D ON MD.DepartmentId = D.Id
```  
##### 10 部门工资前三高的员工，编写一个 SQL 查询，找出每个部门获得前三高工资的所有员工  
```
Create table If Not Exists Employee (Id int, Name varchar(255), Salary int, DepartmentId int)；
Create table If Not Exists Department (Id int, Name varchar(255))；
Truncate table Employee；
insert into Employee (Id, Name, Salary, DepartmentId) values ('1', 'Joe', '85000', '1')；
insert into Employee (Id, Name, Salary, DepartmentId) values ('2', 'Henry', '80000', '2')；
insert into Employee (Id, Name, Salary, DepartmentId) values ('3', 'Sam', '60000', '2')；
insert into Employee (Id, Name, Salary, DepartmentId) values ('4', 'Max', '90000', '1')；
insert into Employee (Id, Name, Salary, DepartmentId) values ('5', 'Janet', '69000', '1')；
insert into Employee (Id, Name, Salary, DepartmentId) values ('6', 'Randy', '85000', '1')；
insert into Employee (Id, Name, Salary, DepartmentId) values ('7', 'Will', '70000', '1')；
Truncate table Department；
insert into Department (Id, Name) values ('1', 'IT')；
insert into Department (Id, Name) values ('2', 'Sales')；
```
```
**notice：复用表做子查询 当外表给定时满足一定条件**  
SELECT
	D.NAME AS Department,
	E.NAME AS Employee,
	E.Salary 
FROM
	Employee E
	JOIN Department D ON E.DepartmentId = D.Id 
WHERE
	( SELECT COUNT( DISTINCT Salary ) FROM Employee WHERE Salary > E.Salary AND DepartmentId = E.DepartmentId ) < 3 
ORDER BY
	Department ASC,
	Salary DESC
```  
##### 11 编写一个 SQL 查询，来删除 Person 表中所有重复的电子邮箱，重复的邮箱里只保留 Id 最小 的那个.     
```
delete p from Person p join Person pp on p.Email=pp.Email and p.Id>pp.Id
```  
##### 12 给定一个 Weather 表，编写一个 SQL 查询，来查找与之前（昨天的）日期相比温度更高的所有日期的 Id。  
```
Create table If Not Exists Weather (Id int, RecordDate date, Temperature int);
Truncate table Weather;
insert into Weather (Id, RecordDate, Temperature) values ('1', '2015-01-01', '10');
insert into Weather (Id, RecordDate, Temperature) values ('2', '2015-01-02', '25');
insert into Weather (Id, RecordDate, Temperature) values ('3', '2015-01-03', '20');
insert into Weather (Id, RecordDate, Temperature) values ('4', '2015-01-04', '30');
```
```
SELECT
	a.Id 
FROM
	Weather a
	JOIN Weather b ON ( a.Temperature > b.Temperature AND DATEDIFF( a.RecordDate, b.RecordDate ) = 1 )
```  
##### 13.有一个courses 表 ，有: student (学生) 和 class (课程)。请列出所有超过或等于5名学生的课。  
```
Create table If Not Exists courses (student varchar(255), class varchar(255));
Truncate table courses;
insert into courses (student, class) values ('A', 'Math');
insert into courses (student, class) values ('B', 'English');
insert into courses (student, class) values ('C', 'Math');
insert into courses (student, class) values ('D', 'Biology');
insert into courses (student, class) values ('E', 'Math');
insert into courses (student, class) values ('F', 'Computer');
insert into courses (student, class) values ('G', 'Math');
insert into courses (student, class) values ('H', 'Math');
insert into courses (student, class) values ('I', 'Math');
```
```
SELECT
	class 
FROM
	( SELECT DISTINCT student, class FROM courses ) B 
GROUP BY
	class 
HAVING
	COUNT( student ) >=5
```  
##### 14.X 市建了一个新的体育馆，每日人流量信息被记录在这三列信息中：序号 (id)、日期 (visit_date)、 人流量 (people)。请编写一个查询语句，找出人流量的高峰期。高峰期时，至少连续三行记录中的人流量不少于100。  
```
Create table If Not Exists stadium (id int, visit_date DATE NULL, people int);
Truncate table stadium;
insert into stadium (id, visit_date, people) values ('1', '2017-01-01', '10');
insert into stadium (id, visit_date, people) values ('2', '2017-01-02', '109');
insert into stadium (id, visit_date, people) values ('3', '2017-01-03', '150');
insert into stadium (id, visit_date, people) values ('4', '2017-01-04', '99');
insert into stadium (id, visit_date, people) values ('5', '2017-01-05', '145');
insert into stadium (id, visit_date, people) values ('6', '2017-01-06', '1455');
insert into stadium (id, visit_date, people) values ('7', '2017-01-07', '199');
insert into stadium (id, visit_date, people) values ('8', '2017-01-08', '188');
```  
**注意日期可能不连续的**    
a)分情况    
```
SELECT * FROM (SELECT c.id,c.visit_date,c.people FROM(SELECT a.id,a.visit_date,a.people FROM stadium a JOIN stadium b ON (a.people>99 AND b.people>99 AND (a.id-b.id)=1))c JOIN stadium d ON (c.people>99 and d.people>99 AND ((c.id-d.id)=2))#前两天都大于100    
UNION  
SELECT c.id,c.visit_date,c.people FROM(SELECT a.id,a.visit_date,a.people FROM stadium a JOIN stadium b ON (a.people>99 AND b.people>99 AND  (a.id-b.id)=1))c JOIN stadium d ON (c.people>99 and d.people>99 AND ((d.id-c.id)=1))#前一天大于100后一天大于100    
UNION  
SELECT c.id,c.visit_date,c.people FROM(SELECT a.id,a.visit_date,a.people FROM stadium a JOIN stadium b ON (a.people>99 AND b.people>99 AND (b.id-a.id)=1))c JOIN stadium d ON (c.people>99 and d.people>99 AND ((d.id-c.id)=2))) temp ORDER BY id ASC#后两天都大于100  
```
b)三表内联，加 where，group by可以去重以及排序  
```
select s1.* 
from 
    stadium s1,
    stadium s2,
    stadium s3
where
    ((s1.id+1 = s2.id and s2.id+1 = s3.id) 
     or (s2.id+1 = s1.id and s1.id+1 = s3.id) 
     or (s2.id+1 = s3.id and s3.id+1 = s1.id))
    and s2.people>=100 and s1.people>=100 and s3.people>=100
group by id
```  
##### 15 小美是一所中学的信息科技老师，她有一张 seat 座位表，平时用来储存学生名字和与他们相对应的座位 id。其中纵列的 id 是连续递增的 小美想改变相邻俩学生的座位。 你能不能帮她写一个 SQL query 来输出小美想要的结果呢？如果学生人数是奇数，则不需要改变最后一个同学的座位。  
```
Create table If Not Exists seat(id int, student varchar(255));
Truncate table seat;
insert into seat (id, student) values ('1', 'Abbot');
insert into seat (id, student) values ('2', 'Doris');
insert into seat (id, student) values ('3', 'Emerson');
insert into seat (id, student) values ('4', 'Green');
insert into seat (id, student) values ('5', 'Jeames');
```
```
select (case when id%2=0 then id-1 when id=(select max(id) from seat) then id else id+1 end) as id,student from seat order by id 
```  
##### 16 给定一个 salary 表，有 m = 男性 和 f = 女性 的值。交换所有的 f 和 m 值（例如，将所有 f 值更改为 m，反之亦然）。要求只使用一个更新（Update）语句，并且没有中间的临时表。  
```
create table if not exists salary(id int, name varchar(100), sex char(1), salary int);
Truncate table salary;
insert into salary (id, name, sex, salary) values ('1', 'A', 'm', '2500');
insert into salary (id, name, sex, salary) values ('2', 'B', 'f', '1500');
insert into salary (id, name, sex, salary) values ('3', 'C', 'm', '5500');
insert into salary (id, name, sex, salary) values ('4', 'D', 'f', '500');
```
``` 
update salary set sex=(case when sex='m' then 'f' when sex='f' then 'm' end)
```
