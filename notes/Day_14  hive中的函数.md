---
title: Day_14  hive中的函数
tags: 数学函数,日期函数,自定义函数
grammar_cjkRuby: true
---


## hive核心流程

> hive解释sql编译成一系列的mr然后按照顺序由executor发送到yarn 框架下去执行
> 解释完sql会形成逻辑计划:有向无环图
> 多个mr之间是有执行顺序的
> stage是根据shuffle来划分的

## hive函数:
### 查看函数

``` sql
1.查看有哪些函数
 show functions
 2.查看某个函数
 describe function last_day
```

### 数学运算函数

关于hive中的函数,可以参考[https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF][1]
1.逻辑函数

![][2]

常用的函数:
- 字符串拼接
``` sql
-- ||为字符串拼接,在mysql中用不了或者concat(emp_name,':',salary)
select '姓名'||emp_name
       ,'薪水'||salary
from dw_employee
-- is null判断为空不为空
select boolean1
from test_serializer
where boolean1 is null
```

- hive支持正则表达式

``` sql
describe function extended regexp
select * from dw_employee
where status regexp '(.{4})'
```

![][3]


2.复杂类型
- 复杂类型构造方法

``` sql
-- 复杂类型构造方法
select map(emp_name,status)
from dw_employee
```

![][4]

3.常用的数学运算函数

- 四舍五入，参数1：小数；参数2：小数点后取的位数 `select round(1.222223,2);`
- 向下取整 `select floor(2.344)`
- 向上取整 `select ceil(2.344)`
- 获取随机值 `select rand();`
- 开平方 `select sqrt(4);`

### 日期类型函数
- 获取当前时间戳

``` sql
select unix_timestamp();
select unix_timestamp('2017-01-02 00:00:00');
```
- 把字符串时间转换成时间戳,指定格式 `select unix_timestamp('2017-01-02 00:00:00','yyyy-MM-dd HH:mm:ss')`
- 时间戳转换成时间 `select from_unixtime(unix_timestamp(),'yyyyMMdd');`
- 把字符串转换成日期格式 yyyy-MM-dd HH:mm:ss `select to_date('2017-03-20 11:30:01');`
- 抽取日期的天数 `select extract(day from '2017-8-10')`
- 计算两个日期相隔天数 `select abs(datediff('2017-08-03','2017-09-02'));`
- 在一个日期上加天数 求新日期 `select date_add('2017-10-1',10);`
- 在一个日期上减天数 求新日期 `select date_add('2017-10-11',-10);select data_sub('2017-10-11',10);`
- 获取当前时间 `select current_date(); select current_timestamp();`
- 时间格式化

``` sql
select date_format(current_date(),'yyyy--MM--dd');
select date_format(current_timestamp(),'yyyy--MM--dd');
select date-format('2017-10-11','yyyy--MM--dd');
```

### 条件函数

![][5]

- if--else

``` sql
select if(salary > 5000,'中等收入','低收入')
from dw_employee
```
- isnull

``` sql
select emp_id
	,emp_name
	,isnull(status_salary)
from dw_employee
```
- isnotnull

``` sql
select emp_id
	,emp_name
	,isnotnull(status_salary)
from dw_employee
```

- COALESCE 求非空

``` sql
-- 取出每个人的上级id，如果没有上级部门id返回-1，如果有返回部门id
select emp_id
	,emp_name
	,COALESCE(leader_id,dep_id,-1)
from dw_employee
```
### String函数

- 获取字符串长度`select character_length('2017')` 或者 `select length('2222');`
- 字符串转换成二进制 `select binary('你好');`
- 二进制转换成string,默认编码格式utf-8 `select decode(binary('你好'),'UTF-8'); select decode(binary('你好'),'GBK');`
- format_number对数字进行格式化，参数2是小数点后保留几位，返回值是字符串 `select format_number(1212.123,2);`
- lcase等同于lower `select lcase('asRfd');`
- trim ltrim rtrim 去除首尾空白字符

``` sql
select trim(' aaa ');
select ltrim(' aaa ');
select rtrim(' aaa ');
```

- 正则表达式抽取 `select regexp_extract('张三:年龄19岁,email:xxx@163.com','.*(\\d+).*(\\w+@163.com)',1)`
- 正则表达式 `select regexp_replace('13522220064','^\\d{7}',"*******");`
- 替换电话号码 `select concat(substr('13134720265',1,3),'xxxx',substr('13134720265',8,4));`
- 字符串反转 `select reverse('abc');`

## 日志

### 日志信息结构分析

``` xml
79.133.215.123 - - [14/Jun/2014:10:30:13 -0400] "GET /home HTTP/1.1" 200 1671 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
```

	1. 用户访问的ip地址
	2. 用户标识
	3. 用户名
	4. 访问时间
	5. 请求信息(请求方法，请求url，协议类型)
	6. 请求相应状态
	7. 请求相应流量大小(byte)
	8. 关联页面
	9. 客户端的浏览器类型

> 对数据进行分析之后，发现使用基本数据类型很难将数据给分隔开来，因此，我们使用特殊的数据类型进行数据存储

### 创建日志对应的表

``` sql
CREATE TABLE apachelog (
  host STRING,
  identity STRING,
  username STRING,
  time STRING,
  request STRING,
  status STRING,
  size STRING,
  referer STRING,
  agent STRING)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
WITH SERDEPROPERTIES (
  "input.regex" = "([^ ]*) ([^ ]*) ([^ ]*) (-|\\[[^\\]]*\\]) ([^ \"]*|\"[^\"]*\") (-|[0-9]*) (-|[0-9]*)(?: ([^ \"]*|\"[^\"]*\") ([^ \"]*|\"[^\"]*\"))?"
  ,"output.format.string"="%1$s %2$s %3$s %4$s %5$s %6$s %7$s %8$s %9$s"
)
STORED AS TEXTFILE;
```

### 加载数据

``` sql
load data inpath '/log' overwrite into table apachelog
select * from apachelog
describe formatted apachelog
```

### 计算当日网站的pv uv

``` sql
-- 计算当日网站的pv uv
-- pv 用户的每个请求就是一个pv；uv(一个ip就是uv的数量)
select count(*) pv
	,count(distinct host) uv
from apachelog
```

### 统计出网站用户访问使用windows系统和使用mac系统的占比和数量

> 这个题目相对比较复杂，我们先将windows，mac，orther用户打上标签，然后，将标记过的数据放到一张临时表中，我们对这个临时表进行统计计算

- 匹配不同的用户

``` sql
select * from apachelog where agent rlike 'Windows NT'
select * from apachelog where agent rlike 'Mac OS'
```

- 创建临时表

``` sql
create table tmp_user_sys
stored as orc
as 
select host
	,sys_type
from (
select host
	,case 
		when agent rlike 'Windows NT' then 'windows'
		when agent rlike 'Mac OS' then 'mac'
		else 'other'
	end sys_type
from apachelog
) a
group by host
	,sys_type
```

- 通过创建的临时表，我们看到同一用户有两台电脑，这里我们把既有mac又有windows的用户看做是两个不同的用户

``` sql
select count(1) p_num
	,sum(case when sys_type='mac' then 1 else 0 end) mac_num
	,sum(case when sys_type='windows' then 1 else 0 end) windows_num
	,sum(case when sys_type='other' then 1 else 0 end) other_num
	,sum(case when sys_type='mac' then 1 else 0 end)/count(1) mac_rate
	,sum(case when sys_type='windows' then 1 else 0 end)/count(1) windows_rate
	,sum(case when sys_type='other' then 1 else 0 end)/count(1) other_rate
from tmp_user_sys
```

## 在hive中自定义函数
> 自定义函数，我们以时间格式转换为例

1. 编写时间转换格式代码

``` java
package com.zhiyou100.hive;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Locale;

import org.apache.hadoop.hive.ql.exec.UDF;

//该udf的功能: [14/Jun/2014:10:30:14 -0400] ---> 2014-07-14 10:30:14
//自定义udf需要继承UDF类,并实现一个evaluate方法
public class LogDateConvert extends UDF {
	
	private static final SimpleDateFormat format=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
	
	//输入数据的日期格式是带有local和时区的,local是英文,可以使用英文格式来进行匹配转换
	private static final SimpleDateFormat srcformat=new SimpleDateFormat("[dd/MMM/yyyy:HH:mm:ss Z]",Locale.ENGLISH);
	
	
    //返回值类型上,可以是JAVA基础类,writable子类,String
    //该方法的参数就是sql调用该方法需要传入的参数
    //该方法的返回参数就是sql调用该方法的计算结果
	public String evaluate(String datestr){
		try {
			Date oldDate = srcformat.parse(datestr);
			return format.format(oldDate);
		} catch (Exception e) {
			e.printStackTrace();
			return null;
		}
	}
}

```

2. 将编写的代码，打成jar包，上传到linux环境下

``` sql
add jar /root/udf.jar;
list jars;
```

3. 将jar包加载到hive上，创建function `create function logdate_convert as 'top.xiesen.udf.LogDateConvert';`
4. 使用自定义函数

``` sql
-- 使用函数
create table hour_pvuv
stored as orc
as
select b.hour 
	,count(1) pv
	,count(distinct b.host) uv
from (select hour(logdate_convert(time)) hour
	,a.*
from apachelog a) b
group by b.hour
```

## 异常处理
1. 在创建hive function时出现 ==ClassNotFoundException==异常信息
解决方案：将jar包上传到hdfs上，再执行添加操作

2. 在查询结果信息时，出现查询结果不一致的现象
解决方案：出现这种现象的原因是虚拟机上linux的时间是英国的时区，老师的系统将时区修改为东八区了，我们可以将linux系统上的时区修改一下，也可以将我们的代码修改了。出现这个问题，给我们的程序没有关系，也可以选择沉默。



  [1]: https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF
  [2]: https://www.github.com/wxdsunny/images/raw/master/1509019461145.jpg
  [3]: https://www.github.com/wxdsunny/images/raw/master/1509020717278.jpg
  [4]: https://www.github.com/wxdsunny/images/raw/master/1509020803105.jpg
  [5]: https://www.github.com/wxdsunny/images/raw/master/1509021587671.jpg