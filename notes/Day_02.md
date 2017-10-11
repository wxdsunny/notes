---
title: Day_02
tags:
grammar_cjkRuby: true
---


##

**集群中的==角色==**:也就是进程或者功能名称,
`hadoop`是统称

==hdfs==下面有三个角色:(主从架构的集群)
 - `namenode`(主节点)
 - `datanode`(子节点)
 - `secondarynamenode`(附属进程,一般在主节点)


==yarn==下面两个角色:(主从架构的集群)
 - `resourceManager`(主节点)
 - `nodeManager` (子节点)

**hdfs和yarn中的进程互相不影响**

==**主从架构**==

==公平节点的缺点:==
 - 公平的节点间的通信会出现多余的通信
 - 通信时会发生两个节点去同一个,先到的节点会锁住块,则另一个会再去别的块,就会浪费时间,会产生抢占式浪费资源

==主从架构缺点:==
 - 主节点产生故障则子节点也没有作用了

**高可用的方式用来解决主节点故障**


  在hadoop中看不到实时的报错,则需要知道出现故障的原因,则在日志中可以找到原因(日志目录logs)
  
![][1]

看日志从尾部看,执行`tail -200`指令,表示看后200行

不抛异常,看日志级别,warn,erro等

查看Hadoop是否启动成功
 - web端:在浏览器中输入主机的ip或者在window中配置的hostname名加50070端口`master:50070`

![][2]

![][3]

![][4]

![][5]

![][6]

快照:(备份是为了数据安全,在生产环境下是必须的)
   虚拟机中的快照:把指定时间的的所有数据内容和状态备份和存储下来
  

 - yarn:在浏览器中输入主机的ip或者在window中配置的hostname名加8088端口`master:8088`

## hdfs
  ### 分布式文件存储
 - 原理:把文件分割,分别存到不同的块中
 - ==块block==:
      1.默认block存储大小为128M,不宜过大过小
	  2. block包含有id,
	  3.尺寸,
	  4. 归属哪个文件,
	  5.块存储的位置
	  6.散布在集群中的不同的节点上
 - ==元数据==(描述数据的数据,还可以进行数据分析,质量控制)
   1 在数据库中存储元数据的表为==数据字典表==
   2 数据库中记录表中数据的为==元组==
   3 关系型数据库中有==模式==和==元组==
   4 nosql中只有==元组==
 
 - ==特点:==
 1.保存多个副本
     - 没有副本的话就会产生读取失败,会出现不稳定,风险性高,在生产环境下就是可用性低
     - 副本的方式来解决稳定性问题,其中一个节点出现故障,hdfs就会去找它的副本,再转到副本上操作
	 
   2.副本保存原则
     - 存储在不同的物理节点上()
     - 机架感知(第一个副本保存在一个机架后,其他副本会优先在其他机架中备份)
     - 副本丢失或宕机自动恢复。默认存3份
     - 运行在廉价的机器上。
     - 适合大数据的处理,不宜放过多的小文件,大量小文件会加重内存负担


   3.运行架构

![][7]

  - `namenode`
   1.处理请求,
   2.分配处理任务,
   3.负责心跳连接(namenode 发送的心跳信号,如果datanode响应就返回信号,如果没有响应就出故障了),
   4.均衡(自动均衡,手动均衡),
   5.副本
   6.(负荷最重)
   7.(最重要的是元数据的处理)
  - `datanode`
    1.数据的读写请求执行
    2.数据的保存操作
  - `SecondaryNameNode`
    1.元数据的备份

  


  [1]: https://www.github.com/wxdsunny/images/raw/master/1507688049872.jpg
  [2]: https://www.github.com/wxdsunny/images/raw/master/1507688964607.jpg
  [3]: https://www.github.com/wxdsunny/images/raw/master/1507689071769.jpg
  [4]: https://www.github.com/wxdsunny/images/raw/master/1507689314125.jpg
  [5]: https://www.github.com/wxdsunny/images/raw/master/1507689380660.jpg
  [6]: https://www.github.com/wxdsunny/images/raw/master/1507689469076.jpg
  [7]: https://www.github.com/wxdsunny/images/raw/master/1507694087255.jpg