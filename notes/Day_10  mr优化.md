---
title: Day_10  mr优化
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


> 1.0中:jobtracher作用:
> - 维护子节点连接
> - 用来执行用户发送过来的mr程序
> - 解析job配置
> - 生成调度schedule
> - 分配task给tasktraker执行
> - 监控接收tasktraker执行的进度和状态信息
> - 整个mr执行结束,jobtraker返回客户端执行状态,才执行结束
> 
> 在2.0中
> - yarn把mapreducer分为两部分:resoucemanage和nodemanage
> - resoucemanage
> 	- 包括资源管理(集群计算资源管理,cpu,内存,网络带宽)
> 	- 集群资源调度
> 	- 第一个container运行的都是ApplicationMaster
> 	- 调度由resoucemanage执行
> - ApplicationMaster 
> 	- job的运行管理
> 	- mapreducer数量
> 	- 自己写mapreducertask的jar
> 	- 监控整个job中每一个task的运行状态
> 	- 分配到任意一个nodemanager执行
> - 不再局限于只执行mapreducer任务
> - 集群的container数量有限的
> - job数量的提升会让container数量达到饱和

mr优化:

![][1]
