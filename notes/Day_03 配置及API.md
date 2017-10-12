---
title: Day_03 配置及API
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

	* [配置Hadoop插件](#配置hadoop插件)
		* [eclipse安装后配置](#eclipse安装后配置)
	* [使用HadoopAPI代码](#使用hadoopapi代码)
		* [联邦(Federation)](#联邦federation)
		* [查看API文档](#查看api文档)
		* [Maven工程的创建](#maven工程的创建)
	* [==运行时报错==](#运行时报错)
	* [API的JAVA代码](#api的java代码)
		* [创建FileSystem对象](#创建filesystem对象)
		* [用FileSystem对象执行方法](#用filesystem对象执行方法)
			* [在HDFS上新建和删除文件](#在hdfs上新建和删除文件)
			* [将数据写入HDFS或读取到本地](#将数据写入hdfs或读取到本地)
			* [将文件以流的形式上传或下载](#将文件以流的形式上传或下载)
			* [查看文件](#查看文件)

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
public static final Configuration CONF = new Configuration();
    public static FileSystem fileSystem;
    
    static{
    	try {
			fileSystem = FileSystem.get(CONF);
		} catch (Exception e) {
			System.out.println("无法连接HDFS,请检查配置");
			e.printStackTrace();
		}
    }
```
### 用FileSystem对象执行方法

#### 在HDFS上新建和删除文件

``` java
新建一个文件:

 public static void creatFolder(String floderName) throws Exception {
    	Path path = new Path(floderName);
    	if(fileSystem.exists(path)){
			System.out.println("文件夹已存在");
		}else{
			fileSystem.mkdirs(path);
		}
	}
	
删除文件:

 public static void deleteFile(String fileName) throws Exception {
    	Path path = new Path(fileName);
    	if(!fileSystem.exists(path)){
    		System.out.println("给定路径不存在");
    	}else{
    		fileSystem.delete(path, true);
    	}
	}
	
删除时文件夹必须存在,而且第二个参数为true,代表递归删除,如果是文件的话设置为true或者false都可以
```

#### 将数据写入HDFS或读取到本地

``` java
写入数据:

public static void write(String fileName,String content) throws Exception {
		Path path = new Path(fileName);
		if(fileSystem.exists(path)){
			System.out.println("文件"+fileName+"已存在");
		}else{
			FSDataOutputStream outputStream = fileSystem.create(path);
			outputStream.writeUTF(content);
			//outputStream.write(content.getBytes());
			outputStream.flush();
			outputStream.close();
		}
	}
	
   writeUTF()方法是写入一个String类型的数据
	  
	  
读取到本地:

 public static void redFile(String fileName) throws Exception {
		Path path = new Path(fileName);
		if(!fileSystem.exists(path) || fileSystem.isDirectory(path)){
			System.out.println("给定路径:"+path+"不存在或者是个文件夹");
		}else{
			FSDataInputStream inputStream = fileSystem.open(path);
			String readUTF = inputStream.readUTF();
			System.out.println(readUTF);
		}
	}
	
	readUTF()返回一个String类型
	写入的时候和读取的时候必须是一直的,用writeUTF()写就要用readUTF()读,不然会报错
```
#### 将文件以流的形式上传或下载

``` java
1 以流上传:

public static void byFSOutputStream(String localSrc,String hdfsDst) throws Exception {
		Path hdfsPath = new Path(hdfsDst);
		FileInputStream fileInputStream = new FileInputStream(localSrc);
		FSDataOutputStream dataOutputStream = fileSystem.create(hdfsPath);
		byte [] b = new byte [1024*5];
		int len = 0;
		while((len = fileInputStream.read(b))!=-1){
			dataOutputStream.write(b, 0, len);
		}
		fileInputStream.close();
	}
	byFSOutputStream("E:/三国演义.txt", "/test04");
	
	从local用输入流读取,用HDFS的输出流写入HDFS上,传入的hdfsDst必须是已存在的文件夹,而且如果文件夹已经存在,想存入文件夹下,则必须给文件起名字,如果直接/test04,则会给文件起的名字为test04
	
	还可以用 IOUtils来进行上传:
	
	public static void copyfile(String localSrc,String hdfsDst) throws Exception {
		Path hdfsPath = new Path(hdfsDst);
		FileInputStream fileInputStream = new FileInputStream(localSrc);
		FSDataOutputStream dataOutputStream = fileSystem.create(hdfsPath);
	    IOUtils.copyBytes(fileInputStream, dataOutputStream, 4096, true);
	}
	copyfile("E:/三国演义.txt", "/test03/c.txt");
	
	参数为输入流,输出流,缓存大小和流的关闭,设置为true就会进行流的关闭,为false就不会关闭流.
	这个util的方法做到直接用输入流读取用输出流写入,就不用再读取到数组中进行循环写入了
	
2 以流下载:

public static void byFSInputStream(String localSrc,String hdfsDst) throws Exception {
		Path path = new Path(hdfsDst);
		FileOutputStream outputStream = new FileOutputStream(localSrc);
		FSDataInputStream dataInputStream = fileSystem.open(path);
		byte [] b = new byte [1024*5];
		int len = 0;
		while((len = dataInputStream.read(b))!=-1){
			outputStream.write(b, 0, len);
		}
		outputStream.close();
	}
	byFSInputStream("E:/abc/", "/test03/a.txt");
	
	下载的时候也需要给文件起名字,而且如果中间有文件夹必须真实存在,E:/abc/如果文件夹不存在则会以abc为文件名来创建
	
	也可以用 IOUtils来进行下载:
	
	public static void downfile(String localSrc,String hdfsDst) throws Exception {
		Path hdfsPath = new Path(hdfsDst);
		FileOutputStream outputStream = new FileOutputStream(localSrc);
		FSDataInputStream dataInputStream = fileSystem.open(hdfsPath);
		IOUtils.copyBytes(dataInputStream, outputStream, 4096, true);
	}
	downfile("E:/abc/abc.txt", "/test03/a.txt");
	
```
#### 查看文件

``` java
1 查看指定目录下的文件

public static void catalog(String fileName) throws Exception {
		FileSystem fileSystem = getFileSystem();
		Path path = new Path(fileName);
		FileStatus[] listStatus = fileSystem.listStatus(path);
		for (FileStatus fileStatus : listStatus) {
			System.out.println(fileStatus.getPath());
		}
	}
	catalog("/test01");
	
	这样只能查看指定目录下的所有文件和文件夹,但是文件夹的子目录看不了,所以会有递归查看
	
2 递归查看文件
    可以用数组递归:
	
	public static void catalog(String fileName) throws Exception {
		FileSystem fileSystem = getFileSystem();
		Path path = new Path(fileName);
		FileStatus[] listStatus = fileSystem.listStatus(path);
		for (FileStatus fileStatus : listStatus) {
			if(fileStatus.isDirectory()){
				catalog(fileStatus.getPath().toString());
			}else{
				System.out.println(fileStatus.getPath());
			}
		}
	}
	catalog("/");

   也可以用listFiles(path, true)进行递归:

    public static void catalog(String fileName) throws Exception {
		Path path = new Path(fileName);
		RemoteIterator<LocatedFileStatus> listFiles = fileSystem.listFiles(path, true);
		while(listFiles.hasNext()){
			LocatedFileStatus next = listFiles.next();
			Path newPath = next.getPath();
			System.out.println("文件地址为:"+newPath);
		}
	}
	catalog("/");
	
	listFiles(path, true)这个方法传入一个路径,如果路径为文件夹,且第二个参数为false,则只返回路径下的文件和文件夹,但不会返回子文件夹,当为true时就会把路径下的所有文件和子文件都返回.
	
```





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