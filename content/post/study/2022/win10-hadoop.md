---
title: "Win10环境下Hadoop安装和配置"
date: 2022-03-24T16:56:59+08:00
categories: ["环境配置"]
tags: ["Hadoop", "大数据", "配置"]
---
## Win10环境下安装
1. 下载安装JDK1.8，并配置环境变量，注意：JAVA_HOME环境变量配置的路径不要包含空格，C盘中的Program Files目录名称可用PROGRA~1代替
2. 下载Hadoop镜像安装文件(我下载的3.2.2)，下载地址：<a href="https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/" target="_blank">镜像文件</a>
   ![Hadoop Image](http://source.icodego.com/2022/hadoop-3.2.2.jpg)
3. 解压Hadoop镜像文件到磁盘目录下，注意：可以解压在非C盘下，存储目录不要包含中文和空格
4. 配置HADOOP_HOME环境变量，并在系统环境变量Path中添加Hadoop环境变量
   ![Hadoop Home](http://source.icodego.com/2022/hadoop_home.png)
   ![Hadoop Home Bin](http://source.icodego.com/2022/hadoop_home_bin.jpg)
5. 打开cmd窗口，输入hadoop version命令验证
   ![Hadoop Version](http://source.icodego.com/2022/hadoop_version.png)
   备注： 若出现 Error: JAVA_HOME is incorrectly set. Please update D:\bigdata\hadoop-3.2.2\etc\hadoop\hadoop-env.cmd的报错，则是因为JAVA_HOME环境变量配置的路径含有空格的原因，请参考步骤1
6. Hadoop伪分布式部署配置  
    a. 下载windows专用二进制文件和工具类依赖库: hadoop在windows上运行需要winUtils支持和hadoop.dll等文件  
    <a href="https://github.com/cdarlint/winutils" target="_blank">https://github.com/cdarlint/winutils</a>  
    在github仓库中找到对应版本的二进制库hadoop.dll和winutils.exe文件，然后把文件拷贝到hadoop解压的bin目录中去  
    ***注意:  hadoop.dll等文件不要与hadoop冲突，若出现依赖性错误可以将hadoop.dll放到C:\Windows\System32下一份***  
    b.修改etc目录下的core-site.xml文件
```xml
<configuration>
  <property>
      <name>dfs.replication</name>
      <value>1</value>
  </property>
  <property>
      <name>dfs.namenode.name.dir</name>
      <value>/D:/bigdata/hadoop-3.2.2/data/dfs/namenode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>/D:/bigdata/hadoop-3.2.2/data/dfs/datanode</value>
  </property>
</configuration>
```
***注意：windows目录路径要改成使用正斜杠，且磁盘名称最前面也需要一个正斜杠***  
    c. 修改hdfs-site.xml配置文件  
```xml
<configuration>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/D:/bigdata/hadoop-3.2.2/data</value>
    <description>存放临时数据的目录，即包括NameNode的数据</description>
  </property>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>
</configuration>
```
***注意：windows目录路径要改成使用正斜杠，且磁盘名称最前面也需要一个正斜杠***  
    d. 节点格式化  
    在cmd窗口执行命令：`hdfs namenode -format`  
    执行成功结果:  
    ![namenode format](http://source.icodego.com/2022/namenode_format.png)
    ![namenode data](http://source.icodego.com/2022/namenode_data.png)  
    多出data文件夹  
7. 启动&关闭Hadoop  
    a. 进入Hadoop的sbin目录下执行start-dfs.cmd启动Hadoop  
    b. Web界面查看HDFS信息，在浏览器输入http://localhost:9870/，可访问NameNode  
   ![namenode information](http://source.icodego.com/2022/namenode_information.png)  
    c. 执行stop-dfs.cmd关闭Hadoop
