---
title: rowkey和二级索引
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


一张表列簇不宜过多
1.表的划分根据rowkey
2.根据rowkey进行存储
3.rowkey为唯一标识
4.唯一索引,不支持其他索引
5.rowkey最大放64kb,在使用时不超过100字节,定长

rowkey长度原则:
- 不宜过长,在100字节之内
- 定长
- 长度最好是8的整数倍

rowkey散列原则:
- 原则
- 方法
	- 随机数(不太好,容易重复)
	- UUID
	- MD5,Hash等加密算法
	- 业务有序数据进行反向

rowkey作为索引原则:
- rowkey是HBase里面唯一的索引,对于某些查询频繁的限定条件数据需要把其内容放在rowkey里面


hbase的二级索引:
目的:提高查询速度
二级索引是hbase里面的一张表
