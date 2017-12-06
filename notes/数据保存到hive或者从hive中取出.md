---
title: 数据保存到hive或者从hive中取出
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

## 从hdfs上导入数据到hive中
### 从Linux上进行

> - 在hive中创建一个表,根据文件的划分决定字段

![在hive中创建一个可以读取文件的表][1]

> - 把hdfs上的数据加载到hive表中 
> load data inpath '/opt/Software/put/download' into table hdfs_hive;
> - 再查看一下是否导入 elect * from hdfs_hive;






  [1]: https://www.github.com/wxdsunny/images/raw/master/1512572573324.jpg