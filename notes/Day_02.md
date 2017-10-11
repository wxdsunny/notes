---
title: Day_02
tags:
grammar_cjkRuby: true
---


##

**集群中的角色**:也就是进程或者功能名称,
hadoop是统称

``` haml
hdfs下面有三个角色:(主从架构的集群)
 - namenode(主节点)
 - datanode(子节点)
 - secondarynamenode(附属进程,一般在主节点)
```


``` haml
yarn下面两个角色:(主从架构的集群)
 - resourceManager(主节点)
 - nodeManager (子节点)
```

**hdfs和yarn中的进程互相不影响**

**主从架构**

``` haml
公平节点的缺点:
 - 公平的节点间的通信会出现多余的通信
 - 通信时会发生两个节点去同一个,先到的节点会锁住块,则另一个会再去别的块,就会浪费时间,会产生抢占式浪费资源
主从架构缺点:
 - 主节点产生故障则子节点也没有作用了
```

**高可用的方式用来解决主节点故障**


  在hadoop中看不到实时的报错,则需要知道出现故障的原因,则在日志中可以找到原因(日志目录logs)
  
![][1]

看日志从尾部看,执行`tail -200`指令,表示看后200行

不抛异常,看日志级别,warn,erro等

查看Hadoop是否启动成功
 - web端:在浏览器中输入主机的ip或者在window中配置的hostname名加50070端口`http://master:50070`

![][2]

![][3]

![][4]

![][5]

![][6]

快照:(备份是为了数据安全,在生产环境下是必须的)
   虚拟机中的快照:把指定时间的的所有数据内容和状态备份和存储下来
   


  [1]: https://www.github.com/wxdsunny/images/raw/master/1507688049872.jpg
  [2]: https://www.github.com/wxdsunny/images/raw/master/1507688964607.jpg
  [3]: https://www.github.com/wxdsunny/images/raw/master/1507689071769.jpg
  [4]: https://www.github.com/wxdsunny/images/raw/master/1507689314125.jpg
  [5]: https://www.github.com/wxdsunny/images/raw/master/1507689380660.jpg
  [6]: https://markdown.xiaoshujiang.com/img/spinner.gif "[[[1507689469076]]]"