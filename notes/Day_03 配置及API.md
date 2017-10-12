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

## ==运行时报错==

>1.这是hdfs依赖的jar包没有导入成功

![][5]

>解决办法有:
>a 把对应的jar包在Maven仓库中找到,删除重新导入
>b 把Maven仓库中org——>apache中的hadoop文件删除,重新导一次

>2.写入的时候和读取时方法不一致

![][6]

![][7]

>解决办法:
>重新再写入一遍,然后用一致的方法读出来


>3.用户没有权限
>会报entry in command string: null chrnod 0644错

![][8]

>解决方法:


>(1) 用	copyToLocalFile(boolean delSrc, Path src, Path dst, boolean useRawLocalFileSystem)
>boolean delSrc表示是否下载后删除原来的文件, boolean useRawLocalFileSystem是否使用原始本地文件系统

>(2).配置
a.在GitHub上下载hadoop-common-2.6.0-bin-master.zip
b.解压后拷贝hadoop.dll到c:\Windows\system32文件夹下面
c.在windows中配置hadoop的环境变量
   i.创建变量HADOOP_HOME
   ii.值为hadoop-common-bin的安装目录

![][9]

>iii.在Path的环境变量中追加hadoop变量%HADOOP_HOME%\bin;

![][10]

## API的JAVA代码

### 创建FileSystem对象

``` java
    创建FileSystem对象,该对象拥有HDFS的操作的方法
```
![][11]

``` java
Configuration conf = new Configuration();
FileSystem fileSystem = FileSystem.get(conf);
```
### 用FileSystem对象执行方法

#### 在HDFS上新建一个文件夹

``` java
 public static void creatFolder(String floderName) throws Exception {
    	Path path = new Path(floderName);
    	if(fileSystem.exists(path)){
			System.out.println("文件夹已存在");
		}else{
			fileSystem.mkdirs(path);
		}
	}
```
#### 在




  [1]: https://www.github.com/wxdsunny/images/raw/master/1507810762679.jpg
  [2]: https://www.github.com/wxdsunny/images/raw/master/1507811742712.jpg
  [3]: https://www.github.com/wxdsunny/images/raw/master/1507811910122.jpg
  [4]: https://www.github.com/wxdsunny/images/raw/master/1507812724522.jpg
  [5]: https://www.github.com/wxdsunny/images/raw/master/1507813583548.jpg
  [6]: https://www.github.com/wxdsunny/images/raw/master/1507813731818.jpg
  [7]: https://www.github.com/wxdsunny/images/raw/master/1507815568354.jpg
  [8]: https://www.github.com/wxdsunny/images/raw/master/1507815702592.jpg
  [9]: https://www.github.com/wxdsunny/images/raw/master/1507816170804.jpg
  [10]: https://www.github.com/wxdsunny/images/raw/master/1507816259959.jpg
  [11]: https://www.github.com/wxdsunny/images/raw/master/1507817995732.jpg