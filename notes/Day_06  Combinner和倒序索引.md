---
title: Day_06  Combinner和倒序索引
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---



## static 修饰的内部类与其他内部类

> - static class 内部类与普通内部类类区别:
> 	- 添加static的话属于静态内部类,和普通类没有多大区别,在使用上可以直接new来实例化对象
> 	- 如果不加static的话,内部类的实例化之前要先实例化外部类

## Map或reduce的成员变量

> 在成员变量位置定义变量
>  mr运行过程中每个map节点只实例化一个map对象 
>  map方法调用时每一行数据记录都会调用一次map方法

## Combinner的过程和作用

> mr运行过程中,每个map节点值实例化一个map对象,map方法调用是每一行数据记录都会调用map方法

Combinner过程:

![][1]

> - 减少shuffle的数据量:
> 	- 每一个map 的输出kv都要经过整个shuffle的过程
> 	- shuffle是最消耗时间的过程,减少shuffle的数据量,能提高mr的运行速度
> 	- 可以先在map节点进行一次聚合,聚合后的结果再发送到reducer上,
> 		- 1减少shuffle的数据量
> 		- 2把reducer上的计算的压力在每一个reducer上进行分担
>     - 通过在mr中添加Combinner的方式给mr配置map端聚合
> 		- Combinner类型是Reducer
> 		- Combinner的reducer方法定义就是map聚合的方法
> 		- Combinner这个Reduce是在map节点发送到Reducer之前来完成的,只计算map端上面的数据
> 		- 真正的Reducer是在独立的Reducer节点上计算的,从每一个map抓取数据过来一同计算
>     - 设置Combinner
> 		- job.setCombinerClass(SomeCombinerClass.class)
>     - 在mr中可以使用Combinner就最好使用
>     - 在溢写之后执行
>     - 不适合使用Combinner场景:
> 		- 1.计算平均值,期望,方差等计算过程不适合使用
> 		- 2.Combinner的输入kv类型和输出kv类型必须保持一致,在计算不支持这种方式的情况下不适用
>     - 当Reducer的输入类型和输出类型一致,并且算法相同时,一般会把Reducer直接当作Combinner,设置IntWritable为1,这样就会便于计算统计


## 倒排索引(目的:搜索和检查)

### 倒排索引原理

> 1.分词解析
> - 先把每一行的每个字段进行解析分词
> - 把分出来的每个词作为索引
> - 把相同的词合并索引
> - 倒排索引合并相同的key(词),排序,索引不合并,排序
> 
> 2.给索引设置id,相同的词会在后面追加id
> 3..根据索引和id就可以快速查找数据

### 多分片问题

- 概念 
 当一个文件有多个block时,一个文件会被分割成多个split,,然后每一个split中都存在一个MapReduce任务,Combiner任务,当不同的map
 中相同的key到达Reducer中,会发生value值重复且没有做合并的情况
- 解决
	- 设置split不分片(目的:解决一个文件可能会有多个分片导致结果错误
	- 使用范围:文件不宜过大
	- 自定义InputFormat继承oTextInputFormat(mr引擎默认使用TextInputformat)
	- 重写isSplitable方法(return false,是关闭分片)
		- 代码:


``` stylus
public static class ReversedIndexFormat extends TextInputFormat {
		@Override
		protected boolean isSplitable(JobContext context, Path file) {
			return false;
		}
	}
```
 
   - - 在job中设置自定义Format

``` stylus
job.setInputFormatClass(ReversedIndexFormat.class);
```


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