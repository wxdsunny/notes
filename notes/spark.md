---
title: spark
tags: spark,特点,运行
grammar_cjkRuby: true
---


## 数据处理应用场景

> - 批处理
> 	- bi分析
> 	- olap
> - 流处理(处理的时间很短)
> 	- 实时需求
> - 交互式处理
> 	- 开发调试
> 	- 数据挖掘
> 	- 模型计算
> 	- 机器学习

## spark特点
> - 速度快,提供 Cache 机制来支持需要反复迭代的计算或者多次数据共享，减少数据读取的 IO 开销。
> - Spark 提供广泛的数据集操作类型（20 + 种）
> - 可以融合多种语言,Spark 支持 Java,Python 和 Scala API
> - Spark 可以使用 YARN 作为他的集群管理器,读取 HDFS，HBase 等一切 Hadoop 的数据

## 数据处理方式

> - 函数式数据处理,使用某种语言的api,通过方法的调用完成数据的处理(底层数据处理)
> - sql(如hive),新版本里用SparkSQL,对流处理也可以用sql

## 运行

> - 到处运行,可以运行在很多分布式框架上,而且不需要搭建spark集群,只要有hadoop集群就可以分布式运行
> - 节约硬件,不需要单独搭建spark的集群
> - 不需要去维护spark集群,只需要维护hadoop
> - 整理数据能力:HDFS,Cassandra,Hbase,S3
> - 处理数据格式多样化
> - 跨环境

## Spark运行过程

![Spark运行][1]

> - Application:一个完整的数据处理过程(最核心的两个部分Driver和Executor)
> 	- Driver
> 		- 把程序解析编译成有向无环图
> 		- 首先运行Driver,运行Application的main函数，一般在其中创建SparkContext
> 		- 一个Driver可能有一个或多个job,一个job可能被划分为多个Task(本质上是函数),Task被分配到某个Executor上执行,Stage是Task的一个粗密度的划分
> 	- Executor
> 		- 执行Driver分配的Task
> 		- 把数据保存到磁盘
> 	- Cluster Manager
> 		-  在分布式集群上获取资源的外部服务（Standalone、Mesos、Yarn）
> 	- Worker Node 
> 		- 具体执行的节点

## Spark数据处理过程:

> - Spark程序数据处理过程,是在Driver上进行计息编译的,在具体对数据处理的时候需要一个触发点来触发执行计算过程
> - 这个触发点就是rdd的action方法
> - rdd处理方法(函数)有两种
> 	- transformation 转换 对数据进行计算转换抽取,值记录处理过程而不真正计算数据,例如,一行变多行,一个单词变kv对,把一个key下的所有value累计
> 	- action   行为   对数据进行读值,触发对数据的计算过程,触发后会把action前面的所有的transformation启动执行,一般不改变数据,只是对数据的结果值进行读取

## 创建spark的Maven工程

> intellj中的程序直接运行在spark的standalong模式下
> - 创建artifact(项目编译打包)
> - 修改conf的master设置: new SparkConf().setMaster("spark://master:7077").setAppName("word count")
> - 在sc相爱添加intellj打的包,打包过程 : build ---->build artifacts ---->build   sc.addJar("D:\\develop\\idea\\workspase\\SparkAndScalamaven\\out\\artifacts\\SparkAndScalamaven_jar\\SparkAndScalamaven.jar")

 # 代码:
 
``` java
package com.zhiyou100

import org.apache.spark.{SparkConf, SparkContext}

/**
  * Created by Administrator on 2017/11/20.
  */
object WordCount {
  def main(args: Array[String]): Unit = {
    //创建SparkConf对象
    //设置分布式运行平台,和app名称
    //使用master指定运行平台,yarn,standalong,mesos,(前三个生产环境下使用)local(开发调试时使用)
    //local(单线程) local[N](n个线程) local[*](本地的cup有多少个核心就启动多少个线程)
//    val conf = new SparkConf().setMaster("local[2]").setAppName("word count")
    val conf = new SparkConf().setMaster("spark://master:7077").setAppName("word count")
    //构建SparkContext对象
    val sc = new SparkContext(conf)
    sc.addJar("D:\\develop\\idea\\workspase\\SparkAndScalamaven\\out\\artifacts\\SparkAndScalamaven_jar\\SparkAndScalamaven.jar")
    //加载数据源,获取RDD对象
    //file:///path/to/name加载本地文件
    //把core-site拷进来就会把hdfs当作默认的文件系统
    val fileRdd = sc.textFile("/user_info.txt")
    //数据处理开始
    val wordRdd = fileRdd.flatMap(line => line.split("\\s"))
    val result = wordRdd.map(x => (x,1)).reduceByKey((x1,x2) => x1 + x2)
    fileRdd.foreach(println)
    //result.saveAsTextFile("/spark/one")
    //简化函数
   /* fileRdd.flatMap(_.split("\\s")).map((_,1))
           .reduceByKey(_+_)
           .saveAsTextFile("/spark/one1")*/
    sc.stop()
  }
}

```


  [1]: https://www.github.com/wxdsunny/images/raw/master/1511161975415.jpg