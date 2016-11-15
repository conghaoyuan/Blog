---
layout: post
title: Hadoop系列之（三）Zookeeper与HBase安装配置
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

## 2.HBase安装配置

更新中。。。。。

### 1.复制解压hbase-1.1.7-bin.tar.gz

### 2.配置hbase-env.sh

### 3.配置hbase-site.xml

### 4.配置regionservers

### 5.将hbase发送到其他slave

### 6.开启HBase服务

## 3.Hive配置

至此，hadoop配置完成，下一步配置storm+spark

