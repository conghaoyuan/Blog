---
layout: post
title: Hadoop系列之（二）JDK和Hadoop安装配置
categories:
- 大数据
---

<div class="message">
  配置jdk和hadoop的原则为，先将Master安装并且配置好，然后再统一将其发送给所有的Slave，Slave如果需要单独配置则单独改变。
</div>

## 1.JDK安装配置
之前在有篇博客是搭建apache tomcat + nutch + solr的已经讲过jdk的详细搭建，此次在这里采用第一种搭建方式，即在`/etc/profile`里进行环境变量的配置。

### 1).JDK解压安装
我所有的软件包，全部在mac上通过terminal下的scp发送到master上了，全部放在`/usr/local/src`下，如下图：
<img width="600" src="/images/161115/srcsoftlist.png" />
切换到`root`用户，在`/usr`下创建`java`目录，将jdk包拷贝到java目录下，解压

	mkdir /usr/java
	cp /usr/local/src/jdk-8u111-linux-x64.tar.gz /usr/java       #将jdk包拷贝到java目录下
	tar zxvf jdk-8u111-linux-x64.tar.gz        #解压jdk包
	rm jdk-8u111-linux-x64.tar.gz              #解压完成后，将其删除
	ll         #查看解压后的jdk包

即可看到如下目录

<img width="600" src="/images/161115/jdklist.png" />

接下来为在`profile`中设置环境变量

### 2).profile配置
打开`/etc/profile`文件，在文件最后加入如下代码：

	vi /etc/profile
	#set java environment
	export JAVA_HOME=/usr/java/jdk1.8.0_111
	export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
	export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin

如图：
<img width="600" src="/images/161115/jdkpath.png" />

添加完成后保存退出`:wq`,然后`source /etc/profile` 让配置生效
<img width="600" src="/images/161115/sourceprofile.png" />

查看jdk是否配置成功

	java -version

显示如下，表示配置成功
<img width="600" src="/images/161115/javaversion.png" />

## 2.Hadoop安装配置

### 1).Hadoop解压安装

将`/usr/local/src/hadoop-2.7.1.tar.gz`拷贝到`/usr`下并将其解压，并将其分配给hadoop用户读的权限，需用root用户登录

	cp /usr/local/src/hadoop-2.7.1.tar.gz /usr
	tar zxvf hadoop-2.7.1.tar.gz
	mv hadoop-2.7.1.tar.gz hadoop 
	rm hadoop-2.7.1.tar.gz
	chown -R hadoop:hadoop hadoop/

<img width="600" src="/images/161115/hadoopzxvf.png" />	
<img width="600" src="/images/161115/hadooplist.png" />	

### 2).Hadoop配置

Hadoop 所有的配置文件全部在`/usr/hadoop/etc/hadoop`下，进行相应的配置时可用`vi`编辑器进行打开配置。其中主要配置其中的5歌文件，如下所示：

<img width="600" src="/images/161115/hadoopetc.png" />	

#### (1).第一步，hadoop-env.sh配置

在24行

	# The java implementation to use.
	# export JAVA_HOME=${JAVA_HOME}
	export JAVA_HOME=/usr/java/jdk1.8.0_111	

如下所示：
<img width="600" src="/images/161115/hadoopenv.png" />	

#### (2).第二步，core-site.xml配置

	<configuration>
        <property>
        	<name>fs.defaultFS</name>
			<value>hdfs://Master.Hadoop:9000</value>
        </property>
        <property>
			<name>hadoop.tmp.dir</name>
            <value>file:/home/hadoop/hadoop-2.7.1/tmp</value>
        </property>
        <property>
			<name>io.file.buffer.size</name>
            <value>131702</value>
        </property>
	</configuration>

如图所示：

<img width="600" src="/images/161115/hadoopcore-site.png" />	

#### (3).第三步，hdfs-site.xml配置

	<configuration>
        <property>
			<name>dfs.namenode.name.dir</name>
            <value>/home/hadoop/hadoop-2.7.1/hdfs/name</value>
        </property>
        <property>
            <name>dfs.namenode.data.dir</name>
            <value>/home/hadoop/hadoop-2.7.1/hdfs/data</value>
        </property>
        <property>
            <name>dfs.replication</name>
            <value>2</value>
        </property>
        <property>
            <name>dfs.namenode.secondary.http-address</name>
            <value>Master.Hadoop:9001</value>
        </property>
        <property>
            <name>dfs.webhdfs.enabled</name>
            <value>true</value>
        </property>
        <property>
            <name>dfs.permissions</name>
            <value>false</value>
        </property>
	</configuration>

如图所示：
<img width="600" src="/images/161115/hadoophdfs-site.png" />	

#### (4).第四步，mapred-site.xml配置

需要将`mapred-site.xml.template`重命名为`mapred-site.xml`

	<configuration>
        <property>
			<name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
        <property>
            <name>mapreduce.jobhistory.address</name>
            <value>Master.Hadoop:10020</value>
        </property>
        <property>
            <name>mapreduce.jobhistory.webapp.address</name>
            <value>Master.Hadoop:19888</value>
        </property>
	</configuration>

如图所示：
<img width="600" src="/images/161115/hadoopmapred-site.png" />	

#### (5).第五步，yarn-site.xml配置

	<configuration>
		<!-- Site specific YARN configuration properties -->
        <property>
        	<name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
        <property>
			<name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>
            <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>
        <property>
            <name>yarn.resourcemanager.address</name>
            <value>Master.Hadoop:8032</value>
        </property>
        <property>
            <name>yarn.resourcemanager.scheduler.address</name>
            <value>Master.Hadoop:8030</value>
        </property>
        <property>
            <name>yarn.resourcemanager.resource-tracker.address</name>
			<value>Master.Hadoop:8031</value>
        </property>
        <property>
            <name>yarn.resourcemanager.admin.address</name>
            <value>Master.Hadoop:8033</value>
        </property>
        <property>
            <name>yarn.resourcemanager.webapp.address</name>
            <value>Master.Hadoop:8088</value>
        </property>
        <property>
            <name>yarn.nodemanager.resource.memory-mb</name>
            <value>2048</value>
        </property>
	</configuration>

如图所示：
<img width="600" src="/images/161115/hadoopyarn-site1.png" />	
<img width="600" src="/images/161115/hadoopyarn-site2.png" />

#### (6).第六步，slaves配置

	Slave1.Hadoop
	Slave2.Hadoop

如图所示
<img src="/images/161115/hadoopslaves.png" />	
注意：slaves 文件只是在Master节点上有用，其他Slave节点没用，但复制过去时带着也无妨。

#### (7).第七步，profile配置Hadoop命令（可省）

	#set hadoop enviroment
	export HADOOP_HOME=/usr/hadoop
	export PATH=$PATH:$HADOOP_HOME/bin

如图所示：
<img width="600" src="/images/161115/hadoopprofile.png" />	

### 3).发送给所有slave节点并进行配置

先将Master配置好的各项文件发给所有的Slave，然后再单独对Slave的相关文件进行设置。

1.将Master的`hosts`文件发给Slave

	scp /etc/hosts root@Slave1.Hadoop:/etc/
	scp /etc/hosts root@Slave2.Hadoop:/etc/

2.将Master的`JDK`发给Slave

	scp -r /usr/java/ root@Slave1.Hadoop:/usr/
	scp -r /usr/java/ root@Slave2.Hadoop:/usr/

3.将Master的`hadoop`发送给Slave

	scp -r /usr/hadoop/ root@Slave1.Hadoop:/usr/
	scp -r /usr/hadoop/ root@Slave2.Hadoop:/usr/

4.将Master的`profile`发送给Slave

	scp /etc/profile root@Slave1.Hadoop:/etc/
	scp /etc/profile root@Slave2.Hadoop:/etc/

5.将Master创建的`hadoop-2.7.1`目录发送到Slave

	scp -r hadoop-2.7.1/ hadoop@Slave2.Hadoop:~/

6.登录所有的Slave进行配置，让`profile`生效，给`hadoop`文件增加hadoop用户读的权限。

	source /etc/profile
	su & cd /usr
	chown -R hadoop:hadoop hadoop/

至此，所有的安装配置工作完成，接下来要进行验证是否配置成功。

### 4).启动验证

#### (1).格式化HDFS文件系统

在"Master.Hadoop"上使用普通用户hadoop进行操作。（备注：只需一次，下次启动不再需要格式化，只需 start-all.sh）

	hadoop namenode -format

如图所示表示格式化成功：

<img width="600" src="/images/161115/hadoopformat1.png" />	
<img width="600" src="/images/161115/hadoopformat2.png" />

#### (2).启动Hadoop

进入到`cd /usr/hadoop/bin`目录下

	./start-all.sh

可以通过以下启动日志看出，首先启动namenode 接着启动datanode1，datanode2，…，然后启动secondarynamenode。再启动yarn，resourcemanager,nodemanager.

启动 hadoop成功后，在 Master 中的 tmp 文件夹中生成了 dfs 文件夹，在Slave 中的 tmp 文件夹中均生成了 dfs 文件夹和 nm-local-dir 文件夹。
<img width="600" src="/images/161115/hadoopstart.png" />	

#### (3).验证Hadoop

通过`jps`查看进程
Master上查看：

	jps

含有：
	
	8515 SecondaryNameNode
	8325 NameNode
	9448 Jps
	8667 ResourceManager

进程，如图所示，表示master运行成功。
<img src="/images/161115/hadoopmasterjps.png" />

Slave上查看：
含有：

	12338 Jps
	11884 NodeManager
	11775 DataNode

进程，如图所示，表示slave上运行成功。
<img src="/images/161115/hadoopslave1jps.png" />
<img src="/images/161115/hadoopslave2jpg.png" />	

还可使用

	[hadoop@Master hadoop]$ hadoop dfsadmin -report

来查看hadoop集群状态。

回到mac上打开chrome浏览器，输入`10.211.55.13:8088`  `10.211.55.13:50070`
可查看相关网页版状态。

<img src="/images/161115/hadoopweb8088.jpg" />
<img src="/images/161115/hadoopweb50070.jpg" />

至此，hadoop配置完成，下一步配置Zookeeper+Hbase+Hive

