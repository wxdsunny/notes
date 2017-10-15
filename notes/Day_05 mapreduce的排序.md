---
title: Day_05 mapreduce的排序
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

## 数据的排序

> reducer的排序为单节点排序 
> 使用新的分区策略解决多个reduce的排序

### 全排序

- 过程:
	- 先进行排序,排序完取个中间值,作为分区的条件,就会保证是有序的

![][1]
- 定义抽样:
	- 对大数据进行数据的统计分布分区时,如果进行全部直接排序的话,需要运行所有的文件就会浪费时间,则出现了抽样
	- 抽样从数据中取出样本,然后对样本进行排序,找到中值,作为数据的中值,进行分区
- 分区
	- 在进行计算的时候要将map输出的数据分配到不同的reduce中,而mapper分配划分数据过程就是Partition,负责分区数据的类焦作Partitioner
	- 设置全排序的分区和分区文件的保存路径,则Partitioner程序就会读取分区文件来完成按顺序进行分区

- 加载分区文件到分布式缓存中,则缓存文件会自动把分区文件加载分配到每一个map节点的内存中
- 设置reducer节点的个数,默认为1个
- 要是想倒序排序,方法之一是指定job的自己定义的继承了compare的类
- 设置读取文件输出文件:
	- map端的输入会把文本文件读取成kv对,按照分割符把一行分为两部分,前面为key后面为value
	- 如果分隔符不存在则整行都是key,value为空
	- 默认分隔符为/t,手动指定分隔符参数:mapreduce.input.keyvaluelinerecordreader.key.value.separator
	- 如果用KeyValueTextInputFormat,则参数类型为Text,Text,以文本字符串来存储,则不能满足用数字存储
	- SequenceFile在保存数据的同时会保留数据的类型,本身就是以kv方式保存数据的,则解决了KeyValueTextInputFormat的问题


- 将随机抽样写入分区文件,在job启动之前启动抽样程序,并将抽样程序得到的中值写入分区文件中
抽样,求中值,根据中值定义partition




  [1]: https://www.github.com/wxdsunny/images/raw/master/1507942865699.jpg