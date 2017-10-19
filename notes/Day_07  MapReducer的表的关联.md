---
title: Day_07  MapReducer的表的关联
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


数据倾斜:
数据里面的某个key值的数据远远大于其他key的数据量
在job中定义的key或者value 的值与map或者reducer的不对应
Combiner的执行位置在map节点,把数据写在map本地,和Reducer不一样

## mr串联
在MapReduce里面可以有多个map,一个reducer
多个map执行不同的数据处理过程,能够提高代码的复用性
把多个map串联,前一个map的输出作为下一个map的输入
mr串联 map+ reducer map\* 
map+ 在map端,reducer和map\* 在Reducer端

![][2]

chainmapper对应map端
chainredu对应的reduce端

## MapReducer的表的关联
把两种类型的数据关联起来:
- join的连接方式
	- Map端关联
		- 效率最高
		- 一个大表和一个小表进行关联,采用
			- 在map端加载数据时,先把小文件加载到分布式缓存文件中(要点)
			- 再把小表中的kv放在内存中
			- 扫描大表,解析出kv
			- 根据kv在内存中找到相同的key,进行抓取
		- 小表:单个节点可以处理,内存完全可以容纳
	- Reducer端关联
		- 效率最低
		- 两张表都是大表的时候采用
			- 给每个读取到的kv打上标签,可以是文件名也可以是表名
				- key:两张表的关联条件
			- 根据标签把相同的所有的kv分成两个部分,这两个部分进行一个笛卡尔乘积,得到的结果就是关联后的结果
	- SemiJoin半连接
		- 比Reducer端关联高些
		- 对Reducer关联的优化
		- SemiJoin半连接的关联
			- 抽取其中一个文件中的关联字段,放入cachefile中
			- map缓存文件中加载,在解析数据时把另一张表中的key和缓存文件中数据关联不上的抛除掉
			- 给两张表kv打标识
			- Reducer根据kv标识把数据分为两组,再进行笛卡尔乘积
			- 这样过滤了两个key值不同的数据,减少了reducer的工作量

宽表

- DistributedCache:
	- 是分布式缓存的一种实现
	- 

半关联:



  [2]: https://www.github.com/wxdsunny/images/raw/master/1508210695843.jpg