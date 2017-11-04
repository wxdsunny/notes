---
title: HBase
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

- HBase适用于一次写入多次读取
	- 历史查询
	- 日志信息
	- 订单信息

- rowkey不仅仅是唯一标识

一个文件对应一个列簇(列簇越短越好)
最底层的存储形式为kv对的形式,按照rowkey的顺序保存的,索引都会进行排序

Table & Region
- 在HBase中:
	- HBase根据rowkey划分region
		- rowkey
			- 1.唯一标识
			- 2.不能过长
			- 3.不能连续
		- 根据rowkey的大小顺序进行划分
		- 第一个region没有下限,最后一个region没有上限
	- 热点访问就不会把HBase的分布式体现出来 
	- 不同的region在不同的regionserver上
		- 当一个region过大时会分裂为两个region,分到不同的regionserver上
		- 在region分裂时,表是没有办法进行对外服务的
		- 一个regionserver上可以有多个region
		- 一个region可以分布在不同的regionserver
		- 当一个regionserver故障后,HBase会通知其他的regionserver启动,映射这个故障的region

![][1]

读取记录时:
1.meta表记录数据的表都在哪些regionserver上
2.meta表是HBase里的一张表,meta表的region记录在root表中
3.root表是zookeeper上的数据

![][2]

Client访问用户数据之前需要首先访问zookeeper，然后访问-ROOT-表，接着访问.META.表，最后才能找到用户数据的位置去访问，中间需要多次网络操作，不过client端会做cache缓存。

MapReduce on HBase
从HBase上读取数据
 - 客户端发送请求
 - Hmaster返回相应的HRegionServer
 - 客户端去找响应的HRegion

保证数据的安全:
- 保存副本
- wal:先写日志
	- 

HBase系统架构
1.Client客户端
2.Zookeeper(必不可少)
3.HMaster(因为有zookeeper存在,不会出现单节点故障)
4.HRegionServer
 HRegion由多个 HStore组成
 HStore分为
 - MemStore当MemStore满了后会形成一个StoreFile
 - 当StoreFile文件数量增长到一定阈值，会触发Compact合并操作，将多个StoreFiles合并成一个StoreFile
- 当StoreFiles超过一定阈值后,会触发Split,先对外停止服务,再分成2个region,父Region会下线，新Split出的2个孩子Region会被HMaster分配到相应的HRegionServer上

![][3]

 - HBase在建表时会进行预分区
 - HLog: 多个region共用一个HLog

5.HBase存储格式
两种文件类型
- HFile

![][4]

- HLog File

![][5]

6.

解决随时读写

![][6]

- 保存数据以kv对形式
	- rowkey+columnfamily+columnqualify+timestamp---》value
- 



  [1]: https://www.github.com/wxdsunny/images/raw/master/1509329111923.jpg
  [2]: https://www.github.com/wxdsunny/images/raw/master/1509328594373.jpg
  [3]: https://www.github.com/wxdsunny/images/raw/master/1509329485357.jpg
  [4]: https://www.github.com/wxdsunny/images/raw/master/1509332757856.jpg
  [5]: https://www.github.com/wxdsunny/images/raw/master/1509332775158.jpg
  [6]: https://www.github.com/wxdsunny/images/raw/master/1509333079995.jpg