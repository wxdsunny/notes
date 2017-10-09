---
title: hadoop与集群搭建
tags: 
grammar_cjkRuby: true
---

 1. 安装虚拟机,并把虚拟机的网络模式设置为NAT(桥接模式是网关和window用的是相同的,也就有可能用局域网的网关,则ip相同的概率会增大,而NAT是本机上提供的虚拟网关,不会和别人的ip重复,比较安全)
   查看NAT模式的网关

  ![enter description here][1]
  
  ![enter description here][2]
  
 2. 设置ip为静态地址

  ![enter description here][3]

　执行`vi /etc/sysconfig/network-scripts/ifcfg-eth0`
　
　![enter description here][4]
　
  再执行`service network restart`让其生效
  
  ![enter description here][5]
  
 3. 修改主机名(修改主机名是为了区分与别的主机,不然全是localhost就区分不出来了)

  ![enter description here][6]
  
  执行 `hostname test01`生效`test01`是修改后的主机名
  
 4. 创建ip和域名的映射(有了映射后可以直接写名字就不用写ip了)

  ![enter description here][7]

 5. 安装JDK
  把JDK放到`/opt/Software/Java/`中,解压
  再配置环境变量 `vim /etc/profile`在`profile`文件中追加环境变量

  ![enter description here][8]

  再执行`source /etc/profile`重新加载文件
  可以执行`java -version`查看JAVA环境是否配好
 6. 配置Hadoop环境变量(有可能会找不到系统配置的环境变量)
  在`etc/hadoop`下都是Hadoop的配置文件
  配置`hadoop-env.sh`中配置jdk

  ![enter description here][9]

 7. 


  [1]: https://www.github.com/wxdsunny/images/raw/master/1507557450802.jpg "1507557450802.jpg"
  [2]: https://www.github.com/wxdsunny/images/raw/master/1507557560527.jpg "1507557560527.jpg"
  [3]: https://www.github.com/wxdsunny/images/raw/master/1507557941819.jpg "1507557941819.jpg"
  [4]: https://www.github.com/wxdsunny/images/raw/master/1507558565934.jpg "1507558565934.jpg"
  [5]: https://www.github.com/wxdsunny/images/raw/master/1507559241598.jpg "1507559241598.jpg"
  [6]: https://www.github.com/wxdsunny/images/raw/master/1507558940918.jpg "1507558940918.jpg"
  [7]: https://www.github.com/wxdsunny/images/raw/master/1507559731814.jpg "1507559731814.jpg"
  [8]: https://www.github.com/wxdsunny/images/raw/master/1507561539150.jpg "1507561539150.jpg"
  [9]: https://www.github.com/wxdsunny/images/raw/master/1507562725324.jpg "1507562725324.jpg"