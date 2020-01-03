---
title: "牛客网部分sql习题总结"
subtitle: "「sql」- engine-charlength~等"
layout: post
author: "Dxufe"
header-style: text
tags:
  - sql-advance
---
1. 查找入职员工时间排名倒数第三的员工所有信息  
```
SELECT *
FROM
	employees 
WHERE
	hire_date = ( SELECT DISTINCT hire_date FROM employees ORDER BY hire_date DESC LIMIT 2, 1 )
%如果把这里的 = 换成limit会报错。  
```
2. 查找当前薪水详情以及部门编号dept_no，join on后面的条件应该是左表与右表关系的限制，其他条件应该在where后面  
```
select s.emp_no,s.salary,s.from_date,s.to_date,d.dept_no 
from salaries s 
join dept_manager d 
on d.emp_no=s.emp_no where s.to_date='9999-01-01' and d.to_date='9999-01-01'
```
3. 查找所有员工入职时的薪水，注意order by后面能指定别名的还是指定别名加点。  
```  
select e.emp_no,s.salary from employees e 
join salaries s on e.emp_no=s.emp_no and e.hire_date=s.from_date  order by e.emp_no desc
```
4. 不用join的效率应该会更高，直接用并列查询。  
```
SELECT e.emp_no, s.salary FROM employees AS e, salaries AS s
WHERE e.emp_no = s.emp_no AND e.hire_date = s.from_date
ORDER BY e.emp_no DESC
```
5. 查找已分配部门的员工的last name 和first name。  
```
select e.last_name,e.first_name,d.dept_no from employees e  join dept_emp d on e.emp_no=d.emp_no
```
6. 查找所有员工的last_name和first_name以及对应部门编号dept_no，也包括展示没有分配具体部门的员工。  
```
select e.last_name,e.first_name,d.dept_no from employees e  left join dept_emp d on e.emp_no=d.emp_no
```
7. 查找所有员工入职时候的薪水情况，给出emp_no以及salary， 并按照emp_no进行逆序。  
```
select e.emp_no,s.salary from employees e join salaries s on e.emp_no=s.emp_no and e.hire_date=s.from_date order by e.emp_no desc 
```
8. 查找薪水涨幅超过15次的员工号emp_no以及其对应的涨幅次数t。  
```
select emp_no,count(distinct from_date) as t from salaries group by emp_no having count(distinct from_date)>15
```
9. 找出所有员工当前(to_date='9999-01-01')具体的薪水salary情况，对于相同的薪水只显示一次,并按照逆序显示。  
```
select distinct salary from salaries  where to_date='9999-01-01' order by salary DESC
```
10. 获取所有部门当前manager的当前薪水情况，给出dept_no, emp_no以及salary，当前表示to_date='9999-01-01'。    
```
select d.dept_no,d.emp_no,s.salary from dept_manager d join salaries s on d.emp_no=s.emp_no where d.to_date='9999-01-01' and s.to_date='9999-01-01' 
```
11. 获取所有非manager的员工emp_no  
```
select emp_no from employees where emp_no not in (select emp_no from dept_manager)
```
12. 获取所有员工当前的manager，如果当前的manager是自己的话结果不显示，当前表示to_date='9999-01-01'。结果第一列给出当前员工的emp_no,第二列给出其manager对应的manager_no。  
```
select e.emp_no,m.emp_no as manager_no from dept_emp e join dept_manager m on (e.dept_no=m.dept_no and e.emp_no <> m.emp_no) where e.to_date='9999-01-01' and m.to_date='9999-01-01'
```
13. 获取所有部门中当前员工薪水最高的相关信息，给出dept_no, emp_no以及其对应的salary,两种写法第一种写法效率更高，先join 后再select  group by。  
``` 
select dept_no,emp_no,max(salary) as salary from (select d.dept_no,d.emp_no,s.salary from dept_emp d join salaries s on d.emp_no=s.emp_no where d.to_date='9999-01-01' and s.to_date='9999-01-01') ee group by dept_no  
select d.dept_no,d.emp_no,max(s.salary) as salary from dept_emp d join salaries s on d.emp_no=s.emp_no where d.to_date='9999-01-01' and s.to_date='9999-01-01'  group by d.dept_no
```
14. 从titles表获取按照title进行分组，每组个数大于等于2，给出title以及对应的数目t。  
```
 select title,count(title) as t from titles group by title having count(title)>=2
```
15. 查找employees表所有emp_no为奇数，且last_name不为Mary的员工信息，并按照hire_date逆序排列。  
```
select * from employees where mod(emp_no,2)=1 and last_name<>'Mary' ORDER BY hire_date DESC
select * from employees where mod(emp_no,2)=1 and last_name NOT IN ('Mary') ORDER BY hire_date DESC
select * from employees where emp_no % 2 = 1 and last_name != 'Mary' order by hire_date desc
```
16. 统计出当前各个title类型对应的员工当前（to_date='9999-01-01'）薪水对应的平均工资。结果给出title以及平均工资avg。  
%两种写法都可以，第二种直接的要快些。
```
select title,avg(salary) as salary from (select t.emp_no,t.title,s.salary from titles t join salaries s on t.emp_no=s.emp_no where t.to_date='9999-01-01' and s.to_date='9999-01-01')st group by title  
select t.title,avg(s.salary) as salary from titles t join salaries s on t.emp_no=s.emp_no where t.to_date='9999-01-01' and s.to_date='9999-01-01' group by title
```
17. 获取当前（to_date='9999-01-01'）薪水第二多的员工的emp_no以及其对应的薪水salary,这要考虑到第二多的员工不只一个。  
```
select emp_no,salary from salaries where salary =(select distinct salary from salaries where to_date='9999-01-01' ORDER BY salary DESC limit 1,1)
```
18. 查找当前薪水(to_date='9999-01-01')排名第二多的员工编号emp_no、薪水salary、last_name以及first_name，不准使用order by ,排名第二也就是唯一大于它的只有1个。  
```
select s.emp_no,s.salary,e.last_name,e.first_name from salaries s join employees e on s.emp_no=e.emp_no where s.to_date='9999-01-01' and (select count(distinct salary) from salaries where salary>s.salary)=1
```
19. 查找所有员工的last_name和first_name以及对应的dept_name，也包括暂时没有分配部门的员工。  
```
#两个left join   
select e.last_name,e.first_name,d.dept_name from employees e left join dept_emp de on e.emp_no=de.emp_no left join departments d on de.dept_no=d.dept_no
```
20. 查找员工编号emp_no为10001其自入职以来的薪水salary涨幅值growth。  
```
select(
    (select salary from salaries where emp_no= 10001 order by to_date DESC limit 1)
    -
    (select salary from salaries where emp_no= 10001 order by to_date ASC LIMIT 1)) 
as growth
```
21. 查找所有员工自入职以来的薪水涨幅情况。  
```
*1.分别join得到初始工资表（employees join salaries emp_no相等并且日期条件）和当前工资表，再join这两个表以emp_no关联*
SELECT ls.emp_no,(ls.salary1-fs.salary2) as growth FROM (SELECT s.emp_no,s.salary as salary1
from salaries s join employees e on s.emp_no=e.emp_no where s.to_date='9999-01-01' ) ls join
(SELECT s.emp_no,s.salary as salary2 from salaries s join employees e on s.emp_no=e.emp_no and s.from_date=e.hire_date) fs 
on ls.emp_no=fs.emp_no 
order by growth asc  
*讲道理只用salary表应该也可以*  
SELECT ls.emp_no,(ls.salary1-fs.salary2) as growth FROM 
(SELECT s.emp_no,s.salary as salary1
from salaries s join 
(SELECT emp_no,MAX(to_date) as date1 from salaries group by emp_no) ss 
on s.to_date =ss.date1 and s.emp_no=ss.emp_no ) ls join
(SELECT s.emp_no,s.salary as salary2
from salaries s join 
(SELECT emp_no,MIN(to_date) as date2 from salaries group by emp_no) sss
on s.to_date =sss.date2 and s.emp_no=sss.emp_no) fs on ls.emp_no=fs.emp_no 
```
22. 统计各个部门对应员工涨幅的次数总和，给出部门编码dept_no、部门名称dept_name以及次数sum。  
*思路：先得到每个员工的涨幅次数，也即时唯一的fromdate次数然后两个join分组求和*
```
select de.dept_no,d.dept_name,sum(et.t) as sum from (select emp_no,count(from_date) as t from salaries group by emp_no) et join dept_emp de on de.emp_no=et.emp_no join departments d on d.dept_no=de.dept_no group by de.dept_no
```
23. 对所有员工的当前(to_date='9999-01-01')薪水按照salary进行按照1-N的排名，相同salary并列且按照emp_no升序排列  
*思路：排名即为唯一的大于等于其薪水的个数，为同一张表指定别名做比较出排名。*
```
select s.emp_no,s.salary,(select count(distinct salary) from salaries where salary>=s.salary and to_date='9999-01-01') as rank from salaries s where s.to_date='9999-01-01' order by rank asc,emp_no asc 
```
24. 获取所有非manager员工当前的薪水情况，给出dept_no、emp_no以及salary ，当前表示to_date='9999-01-01'  
*用上employees表和不用都行*
```
select d.dept_no, e.emp_no,s.salary from employees e join dept_emp d on e.emp_no=d.emp_no join salaries s on e.emp_no=s.emp_no where e.emp_no not in (select emp_no from dept_manager) and d.to_date='9999-01-01' and s.to_date='9999-01-01'
select d.dept_no,d.emp_no,s.salary from dept_emp d  join salaries s on d.emp_no=s.emp_no where d.emp_no not in (select emp_no from dept_manager where to_date='9999-01-01') and d.to_date='9999-01-01' and s.to_date='9999-01-01'
```
25. 获取员工其当前的薪水比其manager当前薪水还高的相关信息，当前表示to_date='9999-01-01',
结果第一列给出员工的emp_no，
第二列给出其manager的manager_no，
第三列给出该员工当前的薪水emp_salary,
第四列给该员工对应的manager当前的薪水manager_salary  
*找出员工当前薪水，经理当期薪水，限制条件，同一部门下员工工资大于经理工资进行筛选*  
```
select des.emp_no,dms.manager_no,des.emp_salary,dms.manager_salary from 
(select de.emp_no,de.dept_no,se.salary as emp_salary from dept_emp de join salaries se on de.emp_no=se.emp_no where se.to_date='9999-01-01') des
join 
(select dm.emp_no as manager_no,dm.dept_no,sm.salary as manager_salary from dept_manager dm join salaries sm on dm.emp_no=sm.emp_no where sm.to_date='9999-01-01') dms 
on des.dept_no=dms.dept_no
where des.emp_salary>dms.manager_salary
```
26. 汇总各个部门当前员工的title类型的分配数目，结果给出部门编号dept_no、dept_name、其当前员工所有的title以及该类型title对应的数目count  
```
SELECT
	de.dept_no,
	d.dept_name,
	t.title,
	count( t.title ) AS count 
FROM
	dept_emp de
	JOIN departments d ON de.dept_no = d.dept_no
	JOIN titles t ON de.emp_no = t.emp_no where de.to_date='9999-01-01' and t.to_date='9999-01-01'
GROUP BY
	de.dept_no,t.title
```
27. 给出每个员工每年薪水涨幅超过5000的员工编号emp_no、薪水变更开始日期from_date以及薪水涨幅值salary_growth，并按照salary_growth逆序排列。  
*提示：在sqlite中获取datetime时间对应的年份函数为strftime('%Y', to_date),mysql用extract(year from coulumn),sql-server用datepart(yyyy,column)*  
*注意：不应该只比较to_date的年份，也应该比较from_date的年份 ，防止数据丢失*  
```
SELECT
	s.emp_no,
	s.from_date,
	( s.salary - ss.salary ) AS salary_growth 
FROM
	salaries s
	JOIN salaries ss ON s.emp_no = ss.emp_no AND ((strftime("%Y",s.to_date)-strftime("%Y",ss.to_date)=1) OR (strftime("%Y",s.from_date)-strftime("%Y",ss.from_date) =1))
	WHERE
	(s.salary - ss.salary) > 5000 
ORDER BY
	salary_growth DESC  
*%！！不用join select from 并列查询多加几个where条件*  
SELECT
	s.emp_no,
	s.from_date,
	(s.salary - ss.salary ) AS salary_growth 
FROM
	salaries s,salaries ss
where s.emp_no = ss.emp_no 
AND (s.salary - ss.salary )  > 5000 
AND ((strftime("%Y",s.to_date)-strftime("%Y",ss.to_date)=1) OR (strftime("%Y",s.from_date)-strftime("%Y",ss.from_date) =1))
ORDER BY
	salary_growth DESC
```
28. 将employees表的所有员工的last_name和first_name拼接起来作为Name，中间以一个空格区分 concat,MySQL、SQL Server、Oracle等数据库支持CONCAT方法  
```
select concat(last_name,' ',first_name) as Name from employees;
#select last_name||' '||first_name as Name from employees %sqlite
```
29. 针对salaries表emp_no字段创建索引idx_emp_no，查询emp_no为10005, 使用强制索引。  
```create index idx_emp_no on salaries(emp_no);  ```  
% MySQL中，使用 FORCE INDEX 语句进行强制索引查询，可参考：  
```SELECT * FROM salaries FORCE INDEX(idx_emp_no) WHERE emp_no = 10005```  

30. 构造一个触发器audit_log，在向employees_test表中插入一条数据的时候，触发插入相关的数据到audit中。
构造触发器时注意以下几点：
* create trigger ：创建触发器
* 触发器要说明是在after 还是before事务发生时触发
* 要指明是insert 、delete、update操作
* on 表名 
* FOR EACH ROW表任何一条记录上的操作满足触发事件都会触发该触发器,sqllite不用
* begin和end之间写触发的动作，语句用分号隔开
* new 关键字表示更新后的表的字段 ，old表示更新前的表的字段 ,也可以插入时间如now()
* 注意：  
一般情况下，mysql默认是以 ; 作为结束执行语句，与触发器中需要的分行起冲突为解决此问题可用DELIMITER，如：DELIMITER ||，可以将结束符号变成||当触发器创建完成后，可以用DELIMITER ;来将结束符号变成;  
```CREATE TRIGGER 触发器名 BEFORE|AFTER 触发事件 ON 表名 FOR EACH ROW```  
```BEGIN```  
```执行语句列表```   
```END```  
```
DELIMITER ||
CREATE TRIGGER audit_log AFTER INSERT ON employees_test FOR EACH ROW
BEGIN
	INSERT INTO audit VALUES	( NEW.ID, NEW.NAME );
END || 
DELIMITER;
```
31. 删除titles_test中emp_no重复的记录，只保留最小的id对应的记录。使用mysql进行delete from操作时，若子查询的 FROM 子句和更新/删除对象使用同一张表，会出现错误,  
*%在sqllite中可以执行*  
```delete from titles_test where id not in (select min(id) from titles_test group by emp_no) ```  
*%在mysql中这样会报错。*  
```delete from titles_test where id not in (select min(id) from titles_test group by emp_no ) ``` in子查询这个样的语句在mysql5.6之前一直是禁止使用的 效率极差  
%mysql delete from 使用改成表连接的方式  
```delete a from table1 a inner join table2 b on a.id=b.id，in子查询```  
```
delete t from titles_test t join (select min(id) as id,emp_no from titles_test group by emp_no) tt on t.emp_no=tt.emp_no and t.id<>tt.id
```
32. 将titles_test中所有to_date为9999-01-01的全部更新为NULL,且 from_date更新为2001-01-01。  
```notcie:update语句中若干列之间要用逗号隔开，不要用and```
```UPDATE titles_test SET to_date=NULL,from_date='2001-01-01' WHERE to_date='0000-00-00'```

33. 将id=5以及emp_no=10001的行数据替换成id=5以及emp_no=10005,其他数据保持不变，使用replace实现。  
```
选择替换:select replace(emp_no,'10001','10005') as emp_no from titles_test where id=5 and emp_no=10001  
更新替换:update titles_test set  emp_no=replace(emp_no,'10001','10005') where id=5 and emp_no='10001'
```
34. 将titles_test表名修改为titles_2017。alter table tablename rename  
```
MYSQL:alter table titles_test rename  (to) titles_2017 
SQLITE:alter table titles_2017 rename to  titles_test
```
35. 在audit表上创建外键约束，其emp_no对应employees_test表的主键id。  
*notice:要设置外键的字段不能为主键；该建所参考的字段必须为主键；两个字段必须具有相同的数据类型和约束*  
```
%foreign key() references biaoming(lieming)
alter table audit add foreign key(emp_no) references empployees_test(id)
```
36. 存在视图 create view emp_v as select * from employees where emp_no >10005;  
*如何获取emp_v和employees有相同的数据？ notice:把视图也当成一张表就行了 *
```select e.* from employees e join emp_v ev on e.emp_no=ev.emp_no ```

37. 将所有获取奖金的员工当前的薪水增加10%。  
```
create table emp_bonus(
emp_no int not null,
recevied datetime not null,
btype smallint not null);
update salaries set salary=salary*1.1 where emp_no in (select emp_no from emp_bonus) 
```
38. 针对库中的所有表生成select count(*)对应的SQL语句,SQLITE里面如下写    
SELECT "select count(*) from " || name || ";" AS cnts FROM sqlite_master WHERE type = 'table'

39. 将employees表中的所有员工的last_name和first_name通过'连接起来。'  
```SELECT concat(last_name,"'",first_name) as name from employees``` 

40. 查找字符串'10,A,B' 中逗号','出现的次数cnt。  
* char_length() 返回字符个数，一个中文或英文字符返回都是1
* length() 字符在当前编码下存储，所占的字节数
* bit_length() 字符在当前编码下存储，所占的bit，也就是length*8
* 比较常用的就是char_length()
* notice:统计全部长度，减去把目标字段替换为空后的长度再除以单个目标字段的长度即为出现的次数
```select (length('10,A,B')-length(replace('10,A,B',',','')))/length(',') AS cnt```  

41. 获取Employees中的first_name，查询按照first_name最后两个字母，按照升序进行排列  
*substr(X,Y,Z) 或 substr(X,Y) 函数的使用, substr/substring    
```
select first_name from employees order by substr(first_name,length(first_name)-1,2) asc
select first_name from employees order by right(first_name,2) asc
```
42. 按照dept_no进行汇总，属于同一个部门的emp_no按照逗号进行连接，结果给出dept_no以及连接出的结果employees  
* notice：mysql中的函数，字符串拼接的话，可以用concat()，但是此函数是针对一条记录的。mysql中
* group_concat适用多条记录的某一字段。
* group_concat(X,Y)，其中X是要连接的字段，Y是连接时用的符号，可省略，默认为逗号。
* group_concat只有与group by语句同时使用才能产生效果，此函数必须与 GROUP BY 配合使用
* 需要将拼接的结果去重的话，可与DISTINCT结合使用即可。
```
SELECT
	dept_no,
	group_concat( emp_no ) AS employees 
FROM
	dept_emp 
GROUP BY
	dept_no
```
43. 查找排除当前最大、最小salary之后的员工的平均工资avg_salary。  
```
select avg(s.salary) as avg_salary from salaries s  join (select min(salary) as mins,max(salary) as maxs from salaries where to_date='9999-01-01') mm on s.salary<>mm.mins and s.salary<>mm.maxs where to_date='9999-01-01'
```
44. 分页查询employees表，每5行一页，返回第2页的数据  
limit仅适用于小数据范围内的分页查询:  
```select * from employees limit (pgaeno-1).pagesize,pagesize```
通过主键id过滤的方法:  
```select * from employees where emp_no >(select emp_no from employees limit (pgaeno-1)*pagesize-1,1) limit pagesize```

45. 获取所有员工的emp_no、部门编号dept_no以及对应的bonus类型btype和received ，没有分配具体的员工不显示  
```
select e.emp_no,d.dept_no,eb.btype,eb.recevied from employees e join dept_emp d on e.emp_no=d.emp_no left join emp_bonus eb on d.emp_no=eb.emp_no
```
46. 使用含有关键字exists查找未分配具体部门的员工的所有信息。  
* 语法： EXISTS subquery
* 参数： subquery 是一个受限的 SELECT 语句 (不允许有 COMPUTE 子句和 INTO 关键字)。使用exists内查询和外查询要关联起来。
* 结果类型： Boolean 如果子查询包含行，则返回 TRUE ，否则返回 FLASE 。  
```select e.* from employees e where not exists(select * from dept_emp where emp_no=e.emp_no)```  
* 和下面这个in 的子查询效果一样，但是上面的效率更高。  
```select * from employees where emp_no not in (select emp_no from dept_emp )```

47. 获取employees中的行数据，且这些行也存在于emp_v中。注意不能使用intersect关键字。  
```
select e.* from employees e join emp_v ev on e.emp_no=ev.emp_no 
```
48. 给出emp_no、first_name、last_name、奖金类型btype、对应的当前薪水情况salary以及奖金金额bonus。btype为1其奖金为薪水salary的10%，btype为2其奖金为薪水的20%，其他类型均为薪水的30%。 当前薪水表示to_date='9999-01-01'  
*notice:case when  then  when then else  end*  
```
select eb.emp_no,ee.first_name,ee.last_name,eb.btype,s.salary,(case when eb.btype=1 then s.salary*0.1 when eb.btype=2 then s.salary*0.2 else s.salary*0.3 end) as bonus from emp_bonus eb join employees ee on eb.emp_no=ee.emp_no join salaries s on ee.emp_no=s.emp_no where s.to_date='9999-01-01'
```
49. 求salary的累计和running_total，其中running_total为前两个员工的salary累计和，其他以此类推,复用 salaries 表进行子查询，最后以 s1.emp_no 排序输出求和结果。输出的第三个字段，是由一个 SELECT 子查询构成。将子查询内复用的 salaries 表记为 s2，主查询的 salaries 表记为 s1，当主查询的 s1.emp_no 确定时，对子查询中不大于 s1.emp_no 的 s2.emp_no 所对应的薪水求和  
```
select s.emp_no,s.salary,(select sum(salary) from salaries  where emp_no<=s.emp_no and to_date='9999-01-01') as running_total from salaries s where s.to_date='9999-01-01'order by emp_no 
```
*用窗口函数一样效果*  
```select emp_no,salary,sum(salary) over(order by emp_no) from salaries;```  

50. 对于employees表中，给出奇数行的first_name  
*notice:表可以跟自己关联比较，分数排名的操作类似,复用表进行子查询，where里面限定子查询的操作条件，当主查询的字段确定时*  
```
SELECT
	first_name 
FROM
	( SELECT e1.first_name, ( SELECT count( * ) FROM employees WHERE first_name <= e1.first_name ) AS rank FROM employees e1 ) t1 
WHERE
	rank % 2 = 1
```
51. update 与join 一起用，update tablea join tableb on tablea.=tableb. set tablea.=  
```
UPDATE company_detail_info
JOIN file_related ON s_info_compname = file_related.prospectusName 
SET s_info_mkt = file_related.tobeMkt
```
52. 查找描述信息中包括robot的电影对应的分类名称以及电影数目，而且还需要该分类对应电影数量>=2部 三表内联  
```
SELECT
	c.name,count(f.film_id) as amount
FROM
	film f
	JOIN film_category fc ON f.film_id = fc.film_id
	JOIN category c ON fc.category_id = c.category_id where f.description LIKE '%robot%' 
GROUP BY c.name 
HAVING
	amount >2
```	
53. 使用join查询方式找出没有分类的电影id以及名称,注意leftjoin表示有的电影可能连电影id都没得也算在内   
```
SELECT
	f.film_id,
	f.title 
FROM
	film f
	LEFT JOIN film_category fc ON f.film_id = fc.film_id 
WHERE
	fc.category_id IS NULL
```	
54. 使用子查询的方式找出属于Action分类的所有电影对应的title,description  
```
SELECT
	title,
	description 
FROM
	film 
WHERE
	film_id IN ( SELECT film_id FROM film_category WHERE category_id = ( SELECT category_id FROM category WHERE NAME = 'Action' ) )
```	
55. explain模拟优化器执行SQL语句，在5.6以及以后的版本中，除了select，其他比如insert，update和delete均可以使用explain查看执行计划，从而知道mysql是如何处理sql语句，分析查询语句或者表结构的性能瓶颈。作用：  
* 表的读取顺序
* 数据读取操作的操作类型
* 哪些索引可以使用
* 哪些索引被实际使用
* 表之间的引用
* 每张表有多少行被优化器查询  
```explain select * from employees```  

56. 创建一个actor表，包含如下列信息  
```
create table if not exists actor(
    actor_id smallint(5) not null comment '主键id',
    first_name varchar(45) not null comment '名字',
    last_name varchar(45) not null comment '姓氏',
    last_update timestamp not null  comment '最后更新时间，默认是系统的当前时间',
		primary key (actor_id))
```
57. 对于表actor批量插入如下数据  
```INSERT INTO actor VALUES(1,'PENELOPE','GUINESS','2006-02-15 12:34:33'),(2,'NIKE','WAHLBERG','2006-02-15 12:34:33')```  
58. 对于表actor批量插入如下数据,如果数据已经存在，请忽略，不使用replace操作    
*注意：insert or ignore into(sqlite3)，mysql中为insert ignore into*  
```INSERT ignore INTO actor VALUES('3','ED','CHASE','2006-02-15 12:34:33')```

59. 创建一个actor_name表，将actor表中的所有first_name以及last_name导入该表。 actor_name表结构如下:  
create table if not exists actor_name (
    first_name varchar(45) not null,
    last_name varchar(45) not null  
);
*从其他表select再插入，就不用values*    
```insert into actor_name select last_name,first_name from actor```  
*%create table biaoming as 再加select语句也可以*  
```create table if not exists actor_name  as select first_name ,last_name from actor```

60. 对first_name创建唯一索引uniq_idx_firstname，对last_name创建普通索引idx_lastname  
```
create unique index uniq_idx_firstname on actor(first_name);
create index idx_lastname on actor(last_name);
```
61. 针对actor表创建视图actor_name_view，只包含first_name以及last_name两列，并对这两列重新命名，first_name为first_name_v，last_name修改为last_name_v：  
```
create view actor_name_view as select first_name as first_name_v,last_name as last_name_v from actor
```
62. 在actor 表last_update后面新增加一列名字为create_date, 类型为datetime, NOT NULL，默认值为'0000 00:00:00'  
```
ALTER TABLE actor
add column create_date2 datetime not null default '0000-00-00 00:00:00'
```
