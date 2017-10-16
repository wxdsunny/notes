---
title: Day_06  Combinner和倒序索引
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---



## static 修饰的内部类与其他内部类
- static class 内部类与普通内部类类区别:
	- 添加static的话属于静态内部类,和普通类没有多大区别,在使用上可以直接new来实例化对象
	- 如果不加static的话,内部类的实例化之前要先实例化外部类

## Map或reduce的成员变量

> 在成员变量位置定义变量
>  mr运行过程中每个map节点只实例化一个map对象 
>  map方法调用时每一行数据记录都会调用一次map方法

## Combinner的过程和作用

> mr运行过程中,每个map节点值实例化一个map对象,map方法调用是每一行数据记录都会调用map方法

Combinner过程:

![][1]

- 减少shuffle的数据量:
	- 每一个map 的输出kv都要经过整个shuffle的过程
	- shuffle是最消耗时间的过程,减少shuffle的数据量,能提高mr的运行速度
	- 可以先在map节点进行一次聚合,聚合后的结果再发送到reducer上,
		- 1减少shuffle的数据量
		- 2把reducer上的计算的压力在每一个reducer上进行分担
    - 通过在mr中添加Combinner的方式给mr配置map端聚合
		- Combinner类型是Reducer
		- Combinner的reducer方法定义就是map聚合的方法
		- Combinner这个Reduce是在map节点发送到Reducer之前来完成的,只计算map端上面的数据
		- 真正的Reducer是在独立的Reducer节点上计算的,从每一个map抓取数据过来一同计算
    - 设置Combinner
		- job.setCombinerClass(SomeCombinerClass.class)
    - 在mr中可以使用Combinner就最好使用
    - 在溢写之后执行
    - 不适合使用Combinner场景:
		- 1.计算平均值,期望,方差等计算过程不适合使用
		- 2.Combinner的输入kv类型和输出kv类型必须保持一致,在计算不支持这种方式的情况下不适用
    - 当Reducer的输入类型和输出类型一致,并且算法相同时,一般会把Reducer直接当作Combinner,设置IntWritable为1,这样就会便于计算统计


## 倒排索引(目的:搜索和检查)

### 倒排索引原理
1.先把每一行的每个字段进行解析,然后对分出来的词作为索引
2.给索引设置id,相同的词会在后面追加id
3.根据索引和id就可以快速查找数据

### 多分片问题


## wordcount的topN问题
- 按照顺序计算出词频前三的数据
	- 1.第一个mr计算词频,第二个mr只取前三输出,用两个mr
	- 2.统计词频,根据词频取TopN,只用一个mr
		- map端加载数据,分析数据,发送给Reducer
		- Reducer接收map发送的数据,按照key进行聚合
		- 在Reducer上开辟一块内存空间,用来存储map解析放入词频,把单词作为key,单词的出现次数作为value
		-  要保证只有一个Reducer节点,则上述方法会造成内存溢出问题
     - 3 解决内存问题
		 - map 中只放三个,可以用treemap
		 - 开辟一块内存空间,只保存三个数据
		 - 在每个map上分别取TopN(用Combiner来实现)






  [1]: https://www.github.com/wxdsunny/images/raw/master/1508118485887.jpg