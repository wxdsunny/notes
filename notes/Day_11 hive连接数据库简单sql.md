---
title: Day_11 hive连接数据库简单sql
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


数据仓库:

数据库:

hive:
- 建立在hadoop上的数据仓库基础构架
- 脱敏:


hive的总体架构:
- hive的客户端
	- CLI:
	- JDBC/ODBC:连接数据库
	- WebGUI:web页面
- Driver:(hive的核心)
	- 解析(compare)SQL
	- 优化(optimizer)
	- 执行(executor)mapreduce
- 数据存储在hdfs上
- 数据操作处理在mapreducer上


hue是cdh(Hadoop的商业实现版) 大数据开发工具
hue是一种web工具

hive和普通关系数据库的异同
在hive上处理sql延迟高

# 内部表与外部表的区别与联系

> 内部表或外部表如果使用load data操作，hive都会把数据剪贴到hive的目录结构里面
外部表可以通过添加location把hive表和外部目录进行映射

1. 在导入数据到外部表，数据并没有移动到自己的数据仓库目录下(如果指定了location的话)，也就是说外部表中的数据并不是由它自己来管理的！而内部表则不一样
2. 在删除内部表的时候，Hive将会把属于表的元数据和数据全部删掉；而删除外部表的时候，Hive仅仅删除外部表的元数据，数据是不会删除的！
3. 在创建内部表或外部表时加上location 的效果是一样的，只不过表目录的位置不同而已，加上partition用法也一样，只不过表目录下会有分区目录而已，load data local inpath直接把本地文件系统的数据上传到hdfs上，有location上传到location指定的位置上，没有的话上传到hive默认配置的数据仓库中。
4. 外部表相对来说更加安全些，数据组织也更加灵活，方便共享源数据。



# DDL数据定义语言

DDL(Data Definition Language)
create 数据库对象的创建
alter 修改数据库对象
drop 删除数据库对象
truncate 清空表数据，表级别的操作，删除后数据不可恢复
truncate和delete之间的区别
truncate是表级别的操作，delete是行级别的操作；truncate删除数据不能通过日志进行恢复，delete删除之后可以通过日志进行恢复

# DML数据操纵语言
DML(Data Manipulation Language)
insert 插入操作
update 更新操作
delete 删除操作

# DCL 数据控制语言(基本上不使用)
Data Control Language
用语执行权限的授予和收回操作
GRANT:授予，给用户授权
Revoke:收回用户已有的权限
Create user:创建用户
create user usernamexxx identity by '123456'

# 对于表之间的操作

## 根据已有表创建表

``` sql
create table dw_employee as 
select cast(emp_id as int)
	,emp_name
	,status
	,cast(salary as double)
	,cast(status_salary as double)
	,cast(in_word_date as date)
	,cast(leader_id as int)
	,cast(dep_id as int)
from employee
select * from dw_employee
```
## 表克隆
> 表克隆只克隆表的样式，不克隆数据

``` sql
-- 表克隆
create table employee_clone like employee
select * from employee_clone
```
## 查看表结构

``` sql
-- 查看表结构
describe employee_clone
describe formatted employee_clone
describe extended employee_clone
```
# SQL练习
## 计算员工的月薪、季度薪、年薪

``` sql
-- 计算员工的月薪、季度薪、年薪
select emp_id
	,emp_name
	,salary
	,salary*3 as month_salary
	,salary*12 as year_salary
from dw_employee
```
## 计算员工的月收入（月薪加奖金）

``` sql
-- 计算员工的月收入（月薪加奖金）
select emp_id
    ,emp_name
    ,salary + status_salary
from dw_employee

-- 判断status_salary是否为空
select emp_id
	,emp_name
	,salary + case when status_salary is null then 0 else status_salary end
from dw_employee
--简化操作，nvl判断是否为空
select emp_id
	,emp_name
	,salary + nvl(status_salary,0)
from dw_employee

```

## 插入一条数据id为1111，姓名为aaa的数据，其余字段空

``` sql
-- hive 几乎不会使用insert values 的写法
insert into employee_clone (emp_id,emp_name) values('ss','fff')
select * from employee_clone
-- hive 支持delete和update，但是数据分析几乎不使用这两种方式来更新表
-- hive对这两个语法的支持需要特殊配置
delete from employee_clone
update employee set emp_id='sss'
```

## 查询员工表如果没有职位，显示‘普通员工’，有职位显示职位、没有入职日期显示入职日期为10-may-15,有入职日期则显示入职日期

``` sql
-- 查询员工表如果没有职位，显示‘普通员工’，有职位显示职位、没有入职日期显示入职日期为10-may-15,有入职日期则显示入职日期
select emp_id
	,emp_name
	,nvl(status,'普通员工')
	,nvl(in_word_date,'2015-5-10')
from dw_employee
```
## 复制员工表，表名为emp_copy

``` sql
-- 复制员工表，表名为emp_copy
create table emp_copy as
select * from dw_employee
```
## 机构中有多少种职位（distinct）

``` sql
-- 机构中有多少种职位（distinct）
select distinct status
from dw_employee
-- 通过group by进行排重
select status 
from dw_employee 
group by status
```

## 从employee查询出每个部门中不重复的职位（distinct）

``` sql
-- 从employee查询出每个部门中不重复的职位（distinct）
select distinct dep_id
	,status
from dw_employee
```
## 薪水高于6000的员工

``` sql
-- 薪水高于6000的员工
select *
from dw_employee
where salary>6000
```
## 职位是anaylst 的员工

``` sql
-- 职位是anaylst 的员工
select *
from dw_employee
where status = 'Anaylst'
```
## 以小写格式展示职位信息(lower())

``` sql
-- 以小写格式展示职位信息（lower()）
select emp_id 
	,emp_name
	,lower(status)
	,upper(status)
from dw_employee
```

## 忽略大小写匹配职位等于‘ANAYLST’的记录

``` sql
select * 
from dw_employee
where upper(status)='ANAYLST'
```

## 查询出薪水在5000到8000之间的员工（between and）

``` sql
select * 
from dw_employee
where salary between 5000 and 8000
```
## 查询出2016年入职的员工

``` sql
-- 查询出2016年入职的员工
select * 
from dw_employee
where year(in_word_date)=2014
```

## 薪水不在5000到8000的员工

``` sql
-- 薪水不在5000到8000的员工
select * 
from dw_employee
where salary not between 5000 and 8000
```
## 查询出职位是Manager或者analyst的员工

``` sql
-- 查询出职位是Manager或者analyst的员工
select * 
from dw_employee
where status = 'Manager' or status = 'Anaylst'

select * 
from dw_employee
where status in('Manager','Anaylst')
```

## 查询出薪水大于7000并且职位是Coder或者Anaylst的员工信息

``` sql
 -- 查询出薪水大于7000并且职位是Coder或者Anaylst的员工信息
select *
from dw_employee
where salary > 7000
	and ( status = 'Manager'
	or status = 'Anaylst')
```
## 查询出没有岗位工资的员工

``` sql
-- 查询出没有岗位工资的员工
select *
from dw_employee
where status_salary is null
查询有岗位工资的员工
select *
from dw_employee
where status_salary is not null
```

## 模糊查询like % _

``` sql
select * 
from dw_employee
where emp_name like'%妖%'
```
## 将查询的结果插入到表中

``` sql
-- insert select
create table dw_employee_leader like dw_employee;
show tables;

insert into dw_employee_leader
select * 
from dw_employee
where leader_id is null

select * from dw_employee_leader
```
## 先把表中的数据清空，然后将数据插入到表中

``` sql
insert overwrite table dw_employee_leader 
select * 
from dw_employee
where leader_id is null

```
# 排序
> 降序desc，升序asc
> Hive的 order by全排序是通过只设置一个reduce节点的方式来实现的

``` sql
select *
from dw_employee
order by salary desc
```
二次排序

``` sql
-- 二次排序
select * 
from dw_employee
order by salary desc,status_salary desc
```

# 聚合函数
聚合----多行数据用一个函数制定的规则来进行运算
Sum：求和
Count：计数，count字段值为空的不计算,count(1)或count(*)比较推荐使用
Avg：平均值
Max：最大值
Min：最小值
分组----为聚合创造多行数据来源的条件
Group by
分组和聚合一般组合起来使用

``` sql
-- Hive的 order by全排序是通过只设置一个reduce节点的方式来实现的
select *
from dw_employee
order by salary desc

-- 二次排序
select * 
from dw_employee
order by salary desc,status_salary desc

-- 查询有多少个员工
select count(status_salary)
from dw_employee

-- 设置reduce的个数
set mapreduce.job.reduces = 2
-- 查询参数
set mapreduce.job.reduces

-- count 字段值为空的不计算
select dep_id
	,count(*)
from dw_employee
group by dep_id
```

## 查询有多少个员工姓张

``` sql
select count(*)
from dw_employee
where emp_name like '%张%'
```
## 计算员工的总薪水是多少

``` sql
select sum(salary)
from dw_employee
```
## 计算员工的人数总和、薪水综合、平均薪水，最高薪水、最低薪水

``` sql
-- 计算员工的人数总和、薪水综合、平均薪水，最高薪水、最低薪水
select count(*)
	,sum(salary)
	,avg(salary)
	,max(salary)
	,min(salary)
from dw_employee
```
## 求出每个部门的最高薪水、最低薪水、平均薪水、总薪水、总人数

``` sql
-- 求出每个部门的最高薪水、最低薪水、平均薪水、总薪水、总人数
select dep_id 
	,sum(salary)
	,avg(salary)
	,max(salary)
	,min(salary)
	,count(1)
from dw_employee
group by dep_id
```

## 求出总人数超过2人的部门的最高薪水、最低薪水、平均薪水、总薪水、总人数

``` sql
-- 求出总人数超过2人的部门的最高薪水、最低薪水、平均薪水、总薪水、总人数
select dep_id 
	,sum(salary)
	,avg(salary)
	,max(salary)
	,min(salary)
	,count(1)
from dw_employee
group by dep_id
having count(1)>2
```
## 查询出最高薪水的人的信息

``` sql
-- 查询出最高薪水的人的信息
select * 
from dw_employee
where salary in (select max(salary) from dw_employee)

--In在后面添加子查询结果作为判断条件时，如果子查询中有索引的话，in是用不到索引的，因此in对于子查询效率低
select *
from dw_employee a
where exists(
	select maxsalary from (select max(salary) maxsalary from dw_employee) b
	where b.maxsalary = a.salary
)
```
## 最低薪水的人

``` sql
-- 最低薪水的人
select * 
from dw_employee
where salary in (select min(salary) from dw_employee)
```
## 工资高于平均薪水的人的信息

``` sql
-- 工资高于平均薪水的人的信息
-- hive 目前版本不支持非等值的链接查询
-- 不能使用
select *
from dw_employee
where salary > (
	select avg(salary) from dw_employee
)
-- 不能使用
select * 
from dw_employee a
where exists(
	select avgsalary from (select avg(salary)avgsalary from dw_employee) b
	where a.salary > b.avgsalary
)
-- 设置不检验笛卡尔乘积
set hive.strict.checks.cartesian.product=false

select a.*
	,b.*
from dw_employee a,(select avg(salary) avgsalary from dw_employee) b 
where a.salary > b.avgsalary

```
## 研发部有哪些职位

``` sql
-- 研发部有哪些职位
select * from dep where dep_name = '研发部'

select distinct status
from dw_employee a
where exists(
	select dep_id from dep where dep_name='研发部'
	and a.dep_id = dep_id
)
```
## 谁是妖姬的同部门的同事

``` sql
select * 
from dw_employee a
where dep_id in(select dep_id from dw_employee where emp_name='妖姬')
and a.emp_name<>'妖姬'
```
## 每个部门最高薪水的人是谁

``` sql
-- 每个部门最高薪水的人是谁
select dep_id
	,max(salary)
from dw_employee
group by dep_id


select * 
from dw_employee a
where exists(
	select b.dep_id
		,b.max_salary
	from(
		select dep_id
			,max(salary) max_salary
		from dw_employee
		group by dep_id
	) b
	where a.dep_id = b.dep_id and a.salary = b.max_salary
)
```

## 那个部门的人数比销售部的人数多

``` sql
-- 那个部门的人数比销售部的人数多,有问题
select a.dep_id
	,a.p_num
	,c.sp_num
from (select dep_id
		,count(1) p_num
	from dw_employee
	group by dep_id
	) a
	,(
	select dep_id
		,count(1) sp_num
	from dw_employee b
	where b.dep_id in(
		select dep_id from dep where dep_name='销售部'
	)
	group by dep_id
     ) c
where a.p_num > c.sp_num
```
## 哪些人是其他人的上级

``` sql
-- 哪些人是其他人的上级
select *
from dw_employee a
where exists(
	select emp_id 
	from dw_employee
	where leader_id = a.emp_id
)
```
## 哪些人不是其他人的上级

``` sql
--哪些人不是其他人的上级
select *
from dw_employee a
where not exists(
	select 1 
	from dw_employee
	where leader_id = a.emp_id
)
```
## 薪水大于8000或者小于2000或者等于5000的员工

``` sql
-- 薪水大于8000或者小于2000或者等于5000的员工
select * from dw_employee where salary > 8000
union all
select* from dw_employee where salary < 2000
union all 
select * from dw_employee where salary = 5000
```
## 提取平均薪水大于5000或者地址在北京的部门的dep_id

``` sql
-- 提取平均薪水大于5000或者地址在北京的部门的dep_i  union
select dep_id
from employee
group by dep_id
having avg(salary) > 5000
union 
select dep_id
from dep
where dep_address like '%北京%'

```
> Union是纵向扩展，join上横向扩展
没有关联条件的join就是笛卡尔乘积
合集
	union不重复
	union all 重复
交集
	instersect
差集
	minus

## 列出员工的姓名和所在的部门的名称和地址
> 可以进行转换，为了提高效率推荐转换，系统默认帮我们转换

``` sql
-- 列出员工的姓名和所在的部门的名称和地址
select e.emp_name
	,d.dep_name,
	d.dep_address
from dw_employee e 
join dep d 
on e.dep_id = cast(d.dep_id as int)
```

## 列出所有员工的姓名和他上司的姓名

``` sql
-- 列出所有员工的姓名和他上司的姓名
select a.emp_name,b.emp_name
from dw_employee a
join dw_employee b
on a.emp_id = b.leader_id
```
> 内连接是两张表根据关联条件相互过滤，能够关联上的数据才会出现在结果集中

## 列出员工的姓名和他所在的部门，把没有部门的员工也列举出来

``` sql
-- 列出员工的姓名和他所在的部门，把没有部门的员工也列举出来
select e.emp_name
	,d.*
from dw_employee e
left join dep d
on e.dep_id=cast(d.dep_id as int)
```

## 全外连接
	
``` sql
-- full outer join 两张表相互补充
-- 列举员工以及部门信息，没有部门的员工也列举出来，没有员工的部门也列举出来
select a.emp_name
	,b.*
from dw_employee a
full outer join dep b
on a.dep_id = cast(b.dep_id as int)
```




