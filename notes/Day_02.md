---
title: Day_02
tags:
grammar_cjkRuby: true
---


## 概论

### **集群中的==角色==**:也就是进程或者功能名称,
`hadoop`是统称

==hdfs==下面有三个角色:(主从架构的集群)
 - `namenode`(主节点)
 - `datanode`(子节点)
 - `secondarynamenode`(附属进程,一般在主节点)


==yarn==下面两个角色:(主从架构的集群)
 - `resourceManager`(主节点)
 - `nodeManager` (子节点)

**hdfs和yarn中的进程互相不影响**

### ==**主从架构**==

==公平节点的缺点:==
 - 公平的节点间的通信会出现多余的通信
 - 通信时会发生两个节点去同一个,先到的节点会锁住块,则另一个会再去别的块,就会浪费时间,会产生抢占式浪费资源

==主从架构缺点:==
 - 主节点产生故障则子节点也没有作用了

**高可用的方式用来解决主节点故障**

### ==错误信息==
  1  在hadoop中看不到实时的报错,则需要知道出现故障的原因,则在日志中可以找到原因(日志目录logs)
  
![][1]

  2 看日志从尾部看,执行`tail -200`指令,表示看后200行

  3 不抛异常,看日志级别,warn,erro等

### ==查看Hadoop是否启动成功==
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
 - ==**原理**==:把文件分割,分别存到不同的块中
 - ==**块block**==:
      1.默认block存储大小为128M,不宜过大过小
	  2. block包含有id,
	  3.尺寸,
	  4. 归属哪个文件,
	  5.块存储的位置
	  6.散布在集群中的不同的节点上
 - ==**元数据**==(描述数据的数据,还可以进行数据分析,质量控制)
   1 在数据库中存储元数据的表为==数据字典表==
   2 数据库中记录表中数据的为==元组==
   3 关系型数据库中有==模式==和==元组==
   4 nosql中只有==元组==
 
 - ==**特点**:==
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
   6.==.负荷最重==
   7.==最重要的是元数据的保存处理==:
          a. 元数据的保存路径
		  
	  ![][8]
		  b.namenode启动过程:
		    1 读取加载fsimage到内存
			2 加载edits
			3 存储文件时,文件的元数据以事务的过程把元数据信息放到两个地方(内存,edits:元数据更新的操作记录和操作内容)
			4 第二次启动时, 读取加载fsimage到内存, 加载edits,按edits里面的操作过程再操作一遍,启动完后,内存里会有一个最全的最新的元数据数据,则会删除fsimage,再新建一个fsimage,把元数据的数据保存到里面,然后再把edits删除,再创建一个空的edits,然后运行的话就会把元数据放如edits中
			5 每次重新启动 都会循环
			6 每个元数据的更新都会保存到新的edits中
			7 长时间的不启动会造成edits里面内容非常大
		c.secondarynamenode:
		    1 会监控namenode
			2 当edits中数据量达到一定量时,拷贝到secondarynamenode
			3 创建一个新的edits
			4 然后还会把fsimage拷贝到secondarynamenode
			5 再把edits和fsimage合并,包括程序的内存和所有元数据的内容
			6 再把合并后的放入新建的fsimage,再把这个fsimage给namenode中的fsimage
			7 safe model保证了在启动过程中不会被操作,启动结束后safe model会退出,就可以对元数据进行操作了
		d.元数据不在
		    1 当fds启动,会启动namenode 还会启动datanode
			2 datanode启动后会自动检查data.dir目录下的所有的block信息
			3 汇报给namenode
 - `datanode`
    1.数据的读写请求执行
    2.数据的保存操作
 - `SecondaryNameNode`
    1.元数据的备份
    
	### HDFS优缺点
	
 #### HDFS优点
 - 高容错性
    1.数据会自动保存多个副本(如果其中节点出现故障,则会找副本,如果发现副本没有两个,就会再找个节点,保存一份副本)
    2.副本丢失后,自动回复
 - 适合批处理
    1.移动的计算和操作
    2.数据位置暴露

 - 适合大数据的处理
    1.GB,TB，PB甚至更大
    2.百万规模以上的文件数量
    3.lOK＋节点
 - 
  #### HDFS缺点
  

 - 低延迟数据访问
    1.   毫秒级读取
    2.   低延迟与高吞吐量

 - 小文件存取(不能过多)
     1.占用NameNode内存空间
     2.寻址时间超过读取时间
 - 并发写入、文件随即修改(不支持)
     1. 一个文件同时只能由一个写入者
     2.  仅支持append(指追加)
     3.  不支持随机修改,只支持顺序写

 - 

  [1]: https://www.github.com/wxdsunny/images/raw/master/1507688049872.jpg
  [2]: https://www.github.com/wxdsunny/images/raw/master/1507688964607.jpg
  [3]: https://www.github.com/wxdsunny/images/raw/master/1507689071769.jpg
  [4]: https://www.github.com/wxdsunny/images/raw/master/1507689314125.jpg
  [5]: https://www.github.com/wxdsunny/images/raw/master/1507689380660.jpg
  [6]: https://www.github.com/wxdsunny/images/raw/master/1507689469076.jpg
  [7]: https://www.github.com/wxdsunny/images/raw/master/1507694087255.jpg
  [8]: https://www.github.com/wxdsunny/images/raw/master/1507702150255.jpg