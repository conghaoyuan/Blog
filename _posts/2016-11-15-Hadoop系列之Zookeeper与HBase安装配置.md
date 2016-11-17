---
layout: post
title: Hadoop系列之（三）Zookeeper、HBase与Hive安装配置
categories:
- 大数据
---

<div class="message">
  Zookeeper管理Hadoop集群中的NameNode，HBase中HBaseMaster的选举，Servers之间状态同步等。单只HBase中ZooKeeper实例负责的工作就有：存储HBase的Schema，实时监控HRegionServer,存储所有Region的寻址入口，当然还有最常见的功能就是保证HBase集群中只有一个Master。
</div>

## 1.Zookeeper安装配置
zookeeper的配置相对简单，配置原理同Hadoop相同，即在Master上安装配置完成后，scp到所有的Slave上。

### 1).复制解压zookeeper-3.4.6.tar.gz
用root用户登录,将`zookeeper-3.4.6.tar.gz`拷贝到`/usr`下，解压缩。

	cp /usr/local/src/zookeeper-3.4.6.tar.gz /usr
	cd /usr
	tar zxvf zookeeper-3.4.6.tar.gz
	chown -R hadoop:hadoop zookeeper
	ll

`ll`即可看到zookeeper的状态，如图所示：
<img width="600px" src="/images/161115/zookeepercpzxvf.png"/>
<img width="600px" src="/images/161115/zookeeperllchown.png"/>

### 2).配置在datanode节点建myid
退出root用户，在家目录hadoop用户下创建相关版本的zookeeper-3.4.6目录文件，并在该目录下创建data目录，用于存放myid

	cd 
	mkdir zookeeper-3.4.6
	mkdir zookeeper-3.4.6/data
	vi myid
	插入1，保存退出:wq

内容如下：
<img width="600px" src="/images/161115/zookeeperdatapwd.png"/>
<img width="600px" src="/images/161115/zookeepermyid.png"/>

### 3).配置zoo.cfg
进入到zookeeper的配置文件目录下，将`zoo_sample.cfg` 改为`zoo.cfg`,打开进行配置

	cd /usr/zookeeper/conf/
	mv zoo_sample.cfg zoo.cfg
	vi zoo.cfg

在16行的dataDir设置为上一步的data路径 

	dataDir=/home/hadoop/zookeeper-3.4.6/data

在21行添加如下配置

	server.1=Master.Hadoop:2888:3888
    server.2=Slave1.Hadoop:2888:3888
    server.3=Slave2.Hadoop:2888:3888

这里的server.id 对应myid中设置的数字
如：这里有三台机器装了zookeeper，那么对应的myid分别为：
    
    Master.Hadoop 中的myid为1
    Slave1.Hadoop 中的myid为2
    Slave2.Hadoop 中的myid为3

将35行和39行的注释打开

	autopurge.snapRetainCount=3
	autopurge.purgeInterval=1

zoo.cfg配置完成，保存退出即可，如图所示：
<img width="600px" src="/images/161115/zookeeperconf.png"/>
<img width="600px" src="/images/161115/zookeepercfg.png"/>

### 4).将zookeeper发送到其他slave并修改权限

同hadoop的配置一样，将`/usr`下的`zookeeper`和`/home/hadoop/zookeeper-3.4.6`文件分别发送到所有的Slave机器上,可发送到root用户。

	scp -r /usr/zookeeper root@Slave1.Hadoop:/usr
	scp -r /usr/zookeeper root@Slave2.Hadoop:/usr
	scp -r /home/hadoop/zookeeper-3.4.6 hadoop@Slave1.Hadoop:/home/hadoop/
	scp -r /home/hadoop/zookeeper-3.4.6 hadoop@Slave2.Hadoop:/home/hadoop/

发送完成后，分别root登录到Slave1和Slave2，修改权限和myid

	cd /usr
	chown -R hadoop:hadoop zookeeper
	
	vi /home/hadoop/zookeeper-3.4.6/data/myid
	将Slave1.Hadoop中的myid设置为2
	将Slave2.Hadoop中的myid设置为3

<img width="600px" src="/images/161115/zookeepermyid.png"/>

### 5).开启Zookeeper服务

配置完成后，分别在装有zookeeper的三台机器上启动zookeeper

	cd /usr/zookeeper/bin
	./zkServer.sh start
	jps

如果三台机器都有`QuorumPeerMain`进程，则说明zookeeper启动成功。

<img width="600px" src="/images/161115/zookeeperstart.png"/>

### 6).查看Zookeeper服务状态
zookeeper通过选举机制来选出`leader`和`follower`,因此机器必须为奇数，偶数会出问题。相关的选举机制及其原理，请参照[zookeeper原理](http://cailin.iteye.com/blog/2014486/)
分别在三台机器上查看zookeeper状态

	./zkServer.sh status

<img width="600px" src="/images/161115/zookeeperstatus.png"/>
<img width="600px" src="/images/161115/zookeeperstatus2.png"/>

至此，zookeeper配置成功。
===================

<div class="message">
  HBase – Hadoop Database，是一个高可靠性、高性能、面向列、可伸缩的分布式存储系统，利用HBase技术可在廉价PC Server上搭建起大规模结构化存储集群。HBase是Google Bigtable的开源实现，类似Google Bigtable利用GFS作为其文件存储系统，HBase利用Hadoop HDFS作为其文件存储系统;Google运行MapReduce来处理Bigtable中的海量数据，HBase同样利用Hadoop MapReduce来处理HBase中的海量数据;Google Bigtable利用 Chubby作为协同服务，HBase利用Zookeeper作为对应来源.
</div>

## 2.HBase安装配置

### 1.复制解压hbase-1.1.7-bin.tar.gz

root用户登录，将`hbase-1.1.7-bin.tar.gz`拷贝到`/usr`目录下，解压后，将`hbase-1.1.7`重命名为`hbase`，删除`hbase-1.1.7-bin.tar.gz`包,给`hbase`添加读权限。

	cd /usr/loacl/src
	ls
	cp hbase-1.1.7-bin.tar.gz /usr/        #将hbase安装包拷贝到安装目录
	cd /usr & ls
	tar zxvf hbase-1.1.7-bin.tar.gz        #解压缩
	mv hbase-1.1.7 hbase                   #重命名
	rm hbase-1.1.7-bin.tar.gz            
	chown -r hadoop:hadoop hbase           #改权限

如下图操作：
<img width="600px" src="/images/161116/hbasetargz.png"/>
<img width="600px" src="/images/161116/hbasechown.png"/>

接下来要进行配置，共需要配置三个文件：`hbase-env.sh` `hbase-site.xml` 和 `regionservers`.如图所示：
<img width="600px" src="/images/161116/hbaseconf.png"/>

### 2.配置hbase-env.sh
`hbase-env.sh`的配置有几个坑，第一个为`HBASE_PID_DIR`，第二个为`PermSize`设置问题，如果第一个不设置的话，在启动hbase的时候会出现一个错误提示

	/usr/hbase/bin/hbase-daemon.sh:行213: /tmp/hbase-hadoop-master.pid: 权限不够

由于`The directory where pid files are stored. /tmp by default.`默认存放hbase-hadoop-master.pid的目录为`/tmp`，而这个目录为系统级目录，需要root权限，且每次系统重启后`/tmp`目录内容清空。故需要重新设置一下，后边会讲。第二个如果不设置的话，也会出现警告提示：

	Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0
	Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
	........

因此，也需要在`hbase-env.sh`配置文件中进行配置，后面讲。

在`hbase-env.sh`配置文件中，需要配置`三个`地方。第一个为`JDK`目录，第二个为`HBASE_PID_DIR`，第三个为`PermSize`.

	cd /usr/hbase/conf
	vi hbase-env.sh
	1.在28行配置jdk路径
	export JAVA_HOME=/usr/java/jdk1.8.0_111
	2.在122行配置pid目录
	export HBASE_PID_DIR=/home/hadoop/hbase-1.1.7/tmp
	3.在46行将PermSize的两个配置关掉
	#export HBASE_MASTER_OPTS="$HBASE_MASTER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m"
	#export HBASE_REGIONSERVER_OPTS="$HBASE_REGIONSERVER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m"

至此，hbase-env.sh配置完成。保存退出即可`:wq`，这是正确的配置方法，如图：

jdk配置：
<img width="600px" src="/images/161116/hbasejdk.png"/>

pid_dir配置：
<img width="600px" src="/images/161116/hbasepid.png"/>
这里的pid路径需要自己创建
<img width="600px" src="/images/161116/hbasepiddir.png"/>
如果不配置的话，启动时会出现如下错误：
<img width="600px" src="/images/161116/hbaseerrorpid.png"/>

permsize配置：
<img width="600px" src="/images/161116/hbaseenvpermsize1.png"/>
入托不进行配置的话，启动时会出现如下错误：
<img width="600px" src="/images/161116/hbasepermsizeerror.png"/>

### 3.配置hbase-site.xml

	vi hbase-site.xml
	添加如下配置：
	<property>
            <name>hbase.rootdir</name>
            <value>hdfs://Master.Hadoop:9000/hbase</value>
    </property>
    <property>
            <name>hbase.cluster.distributed</name>
            <value>true</value>
    </property>
    <property>
            <name>hbase.zookeeper.quorum</name>
            <value>Master.Hadoop,Slave1.Hadoop,Slave2.Hadoop</value>
    </property>
    <property>
            <name>hbase.zookeeper.property.dataDir</name>
            <value>/home/hadoop/zookeeper-3.4.6/data</value>
    </property>
    保存退出:wq

其中`hbase.zookeeper.property.dataDir`的目录为之前配置`zookeeper`的data目录。
<img width="600px" src="/images/161116/hbase-site.png"/>

### 4.配置regionservers

	vi regionservers
	添加：
	Master.Hadoop
	Slave1.Hadoop
	Slave2.Hadoop

<img width="200px" src="/images/161116/hbaseregoin.png"/>

### 5.将hbase发送到其他slave

和hadoop、zookeeper的配置一样，在master上配置完成后，将其相关文件发送到所有的slave下，并且修改其权限。

	scp -r /usr/hbase root@Slave1.Hadoop:/usr
	scp -r /usr/hbase root@Slave2.Hadoop:/usr
	提示让输入密码后发送过去
	scp -r /home/hadoop/hbase-1.1.7 hadoop@Slave1.Hadoop:/home/hadoop
	scp -r /home/hadoop/hbase-1.1.7 hadoop@Slave2.Hadoop:/home/hadoop

分别进入Slave1和Slave2，修改`/usr/hbase`的权限问题，root用户登录
	
	cd /usr
	ll
	chown -R hadoop:hadoop hbase/

修改完成后，即完成hbase配置，下边开启服务并验证。

### 6.开启HBase服务

进入master的hbase

	cd /usr/hbase/bin
	./start-hbase.sh
	jps
	查看所有机器，会出现HRegionServer进程，则hbase启动成功

<img width="800px" src="/images/161116/hbasestart.png"/>
<img width="300px" src="/images/161116/hbaseslave.png"/>

至此，HBase安装配置完成，可以装Hive了。
================

<div class="message">
	1.Hive 是建立在 Hadoop  上的数据仓库基础构架。它提供了一系列的工具，可以用来进行数据提取转化加载（ETL ），这是一种可以存储、查询和分析存储在 Hadoop  中的大规模数据的机制。Hive 定义了简单的类 SQL  查询语言，称为 QL ，它允许熟悉 SQL  的用户查询数据。同时，这个语言也允许熟悉 MapReduce  开发者的开发自定义的 mapper  和 reducer  来处理内建的 mapper 和 reducer  无法完成的复杂的分析工作。
	2.Hive是SQL解析引擎，它将SQL语句转译成M/R Job然后在Hadoop执行。
	3.Hive的表其实就是HDFS的目录/文件，按表名把文件夹分开。如果是分区表，则分区值是子文件夹，可以直接在M/R Job里使用这些数据。
</div>

## 3.Hive安装配置
因为Hive为SQL的解析引擎，故需要数据库作为`存储引擎`，Hive默认使用内嵌`derby`数据库作为存储引擎，但`Derby`引擎的缺点为：一次只能打开一个会话，因此后边我们会把`Mysql`作为外置存储引擎，多用户可同时访问。

### 1).安装Hive

这里要说明一下，Hive只需要在一台机器上安装即可，原则上Hive可以在任意一台机器上安装，有的可能考虑Master的服务较多，会选择一台datanode进行安装，在这里我们就选择在Master上安装即可，同样，Mysql也只需要在Master上安装。

#### (1).安装Hive

#### (2).配置Hive

#### (3).启动测试Hive




### 2).安装mysql

这里采用的是yum安装，注意一下，使用yum必须虚拟机能上外网，否则没法用。CentOS7的yum源中默认没有mysql，所以要先下载mysql的repo源。我们所有的软件包都放在`/usr/local/src`下，所以同样进入此目录下载安装。`root用户登录`。

	cd /usr/local/src
	wget http://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
	rpm -ivh mysql-community-release-el7-5.noarch.rpm
	yum install mysql-server

<img width="600px" src="/images/161116/mysqlyum.png"/>
yum install后会有几次然提示选择y/NO,这里都输入y同意即可。以上为mysql的yum安装方式，按照以上命令安装即可，一共需要下载33个包共192M，根据网速的不同，安装的速度也不同。详细其他如图所示：

<img width="800px" src="/images/161116/mysqlyumsearch.png"/>
<img width="800px" src="/images/161116/mysqlyuminstall1.png"/>
<img width="800px" src="/images/161116/mysqlyuminstall2.png"/>
至此，mysql已经安装成功。接下来要对其进行配置：启动mysql、设置开机启动、修改默认root密码、配置默认编码utf8

(1).启动mysql

	systemctl start mysqld
	systemctl status mysqld      #查看mysql启动状态

如图所示：
<img width="800px" src="/images/161116/mysqlstart.png"/>

(2).设置开机启动

	systemctl enable mysqld
	systemctl daemon-reload

如图所示：
<img width="400px" src="/images/161116/mysqlreboot.png"/>
(3).修改默认root密码

	grep "password" /var/log/mysqld.log      #复制临时密码
	mysql -u root -p
	粘贴临时密码回车即可登录到mysql
	mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass123!'; 
	或者
	mysql> set password for 'root'@'localhost'=password('MyNewPass123!');

通过此步骤可查看mysql的临时密码，查找出来后将其复制
<img width="800px" src="/images/161116/mysqltmppass.png"/>
mysql的登录，将复制的密码粘贴
<img width="800px" src="/images/161116/mysqllogin.png"/>
登陆进去后要进行密码重置，mysql5.6以上增加了密码规则，默认为Medium，需要在8个字符以上，且包含大小写字母数字等。
<img width="600px" src="/images/161116/mysqlpassset.png"/>
改了密码后可以查看密码的验证规则
<img width="600px" src="/images/161116/mysqlvalipass.png"/>

(4).配置默认编码utf8

	vi /etc/my.cnf
	在最后插入
	character_set_server=utf8
	init_connect='SET NAMES utf8'
	保存退出即可:wq
	重启mysql
	systemctl restart mysqld

查看默认的字符集
<img width="600px" src="/images/161116/mysqldefaultchar.png"/>
在my.cnf中进行修改配置
<img width="400px" src="/images/161116/mysqlmy.png"/>
配置完成后保存退出，重启mysql，再登录进去，即可查看现在的字符集
<img width="600px" src="/images/161116/mysqldefaultchar2.png"/>

其他：
	默认配置文件路径：  /etc/my.cnf  
	日志文件：/var/log//var/log/mysqld.log  
	服务启动脚本：/usr/lib/systemd/system/mysqld.service  
	socket文件：/var/run/mysqld/mysqld.pid
	
### 2.安装Hive



至此，Zookeeper、HBase、Hive安装配置完成，下一步配置storm+spark

