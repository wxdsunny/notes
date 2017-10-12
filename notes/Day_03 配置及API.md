---
title: Day_03 配置及API
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


## 配置Hadoop插件
### eclipse安装后配置

1.配置编码格式

![][1]

2.配置jdk

![][2]

3.配置Maven的核心配置

![][3]


## 使用HadoopAPI代码

### 联邦(Federation)
>1.namenode中元数据过多,导致内存不够,则解决办法为使用联邦
>2.多个namenode 但是多个之间存储的元数据是不同的,互不影响的
>3.namespace会分配多个空间给namenode,则文件存放后会加上namespace的命名空间名作为前缀
>4.通过namespace确定文件的元数据所在的位置


>JAVA类有两种数据形式:功能型(注重方法)和数据型(注重属性(字段))
>1 对于重功能型的要看其方法

==FileSystem==:(重功能的JAVA类)

### 查看API文档
>1.class的描述
>2.class的constructor Summary(构造方法的概述)
>3.oStaticMethod(方法)、factory、build

### Maven工程的创建
 
 ![][4]

>==运行时报错==


  [1]: https://www.github.com/wxdsunny/images/raw/master/1507810762679.jpg
  [2]: https://www.github.com/wxdsunny/images/raw/master/1507811742712.jpg
  [3]: https://www.github.com/wxdsunny/images/raw/master/1507811910122.jpg
  [4]: https://www.github.com/wxdsunny/images/raw/master/1507812724522.jpg