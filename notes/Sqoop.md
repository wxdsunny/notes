---
title: Sqoop
tags: sqoop,job,link
grammar_cjkRuby: true
---

sqoop是基于mapreduce的 
Sqoop2多了WEB
![][1]
Sqoop下的object对象:
- 基本信息
	- server
	- version
- 核心
	- connector
		- connector是Sqoop支持的存储系统连接配置类型connector的名称
		- 当前Sqoop支持的连接器和连接类型,在此查看
	- driver
		- 驱动配置信息,在此查看
	- link,job
		- 数据导入导出配置对象
		- link配置具体的存储连接,它是以connector作为类型的,比如某个jdbc数据库的连接,某个hdfs集群连接等
		- job里面配置一次导入导出过程的全部细节信息,它是以link作为输入忽然输出的,通常用from link1 tolink2表示把link1的数据导入到link2上,导出数据的限定在job里面配置
	- submission
		- 查看已提交的Sqoop导入导出任务
- 参数信息
	- option
- 相关权限信息
	- role
	- principal
	- privilege



- 小型数据库:
	- Derby
	- H2
	- Sqlite

前提条件:Hadoop启动 jobhistory启动
jobhistory启动: ` mr-jobhistory-daemon.sh start historyserver`
1.配置link
2.配置job
3.Start job

从hdfs导出到mysql
/bd14/exptomysql

csv是用逗号隔开,

  [1]: https://www.github.com/wxdsunny/images/raw/master/1510109173049.jpg