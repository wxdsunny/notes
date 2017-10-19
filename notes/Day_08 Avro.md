---
title: Day_08 Avro
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


## Avro
### Avro
- 序列化和反序列化工具(核心功能)
	- 序列化过程中提供复杂的数据结构
		- 复杂数据结构:多个字段,多种数据类型
		- 类似数据库中的表
		- 一种模式
- rpc库
- 数据的交互
- 版本的控制
- 压缩二进制数据格式(可分片)
- 可以在mr中处理

Avro实现:
- 定义模式
	- 使用joson形式定义schema
- 作为大数据平台上的文件格式
	- MapReduce读Avro文件
	- MapReduce写Avro文件
- 合并小文件
	- 作为一个文件容器容纳多个小文件
	- 本身在hdfs上是一个大文件 

大数据:
- 数据采集
	- 爬虫
	- 数据产生方推送
- 数据分析
	- 数据源是文本的可能性比较大
	- 中间过程一般不使用文本格式
- 数据展现
	- 发送到rdbms



sequenceFlie和Avro比较
- Avro能把每一个字段的类型保存下来
- sequenceFlie只保存key,value的格式