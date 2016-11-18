---
layout: post
title: Hadoop系列之（四）Hive安装配置
categories:
- 大数据
---
<div class="message">
	1.Hive 是建立在 Hadoop  上的数据仓库基础构架。它提供了一系列的工具，可以用来进行数据提取转化加载（ETL ），这是一种可以存储、查询和分析存储在 Hadoop  中的大规模数据的机制。Hive 定义了简单的类 SQL  查询语言，称为 QL ，它允许熟悉 SQL  的用户查询数据。同时，这个语言也允许熟悉 MapReduce  开发者的开发自定义的 mapper  和 reducer  来处理内建的 mapper 和 reducer  无法完成的复杂的分析工作。
	2.Hive是SQL解析引擎，它将SQL语句转译成M/R Job然后在Hadoop执行。
	3.Hive的表其实就是HDFS的目录/文件，按表名把文件夹分开。如果是分区表，则分区值是子文件夹，可以直接在M/R Job里使用这些数据。
</div>

因为Hive为SQL的解析引擎，故需要数据库作为`存储引擎`，Hive默认使用内嵌`derby`数据库作为存储引擎，但`Derby`引擎的缺点为：一次只能打开一个会话，因此后边我们会把`Mysql`作为外置存储引擎，多用户可同时访问。
这里要说明一下，Hive只需要在一台机器上安装即可，原则上Hive可以在任意一台机器上安装，有的可能考虑Master的服务较多，会选择一台datanode进行安装，在这里我们就选择在Master上安装即可，同样，Mysql也只需要在Master上安装。

## 1.安装mysql

这里采用的是yum安装，注意一下，使用yum必须虚拟机能上外网，否则没法用。CentOS7的yum源中默认没有mysql，所以要先下载mysql的repo源。我们所有的软件包都放在`/usr/local/src`下，所以同样进入此目录下载安装。`root用户登录`。如果原先机器上已经安装mysql或者自带mysql，可以通过`rpm -e进行卸载`。

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

### 1).启动mysql

	systemctl start mysqld
	systemctl status mysqld      #查看mysql启动状态

如图所示：
<img width="800px" src="/images/161116/mysqlstart.png"/>

### 2).设置开机启动

	systemctl enable mysqld
	systemctl daemon-reload

如图所示：
<img width="400px" src="/images/161116/mysqlreboot.png"/>
### 3).修改默认root密码

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

### 4).配置默认编码utf8

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

### 5).设置允许远程用户登录访问mysql

如果不进行配置，在进行

	mysql -u root -h Master.Hadoop -p

登陆时，会出现错误：

	ERROR 1130 (HY000): Host 'Master.Hadoop' is not allowed to connect to this MySQL server

两种方法进行修改，选一种即可：

方法一、本地登入mysql，更改 "mysql" 数据库里的 "user" 表里的 "host" 项，将"localhost"改为"%"
	
	mysql -u root -proot
	mysql>use mysql;
	mysql>update user set host = '%' where user = 'root';
	mysql>select host, user from user;
	mysql>FLUSH PRIVILEGES

方法二、直接授权(推荐)
	
	mysql -u root -proot 
	mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'youpassword' WITH GRANT OPTION;
	mysql>FLUSH PRIVILEGES

<img width="600px" src="/images/161117/mysqlremotelogin.png"/>
<img width="600px" src="/images/161117/mysqlremotesetup.png"/>
操作完后切记执行以下命令刷新权限 
	
	FLUSH PRIVILEGES

其他：
	默认配置文件路径：  /etc/my.cnf  
	日志文件：/var/log//var/log/mysqld.log  
	服务启动脚本：/usr/lib/systemd/system/mysqld.service  
	socket文件：/var/run/mysqld/mysqld.pid

## 2.安装Hive

### 1).安装Hive

	cd /usr/local/src
	ls
	cp apache-hive-1.2.1-bin.tar.gz /usr
	tar zxvf apache-hive-1.2.1-bin.tar.gz
	mv apache-hive-1.2.1 hive
	chown -R hadoop:hadoop hive
	rm apache-hive-1.2.1-bin.tar.gz
	ll

<img width="600px" src="/images/161117/hivesrctar.png"/>
<img width="600px" src="/images/161117/hivezxvf.png"/>

### 2).配置Hive

#### (1).配置/etc/profile

	vi /etc/profile
	最后添加
	#set hive environment
	export HIVE_HOME=/usr/hive
	export HIVE_CONF_HOME=/usr/hive/conf
	export PATH=$PATH:$HIVE_HOME/bin
	保存退出:wq
	source /etc/profile

<img width="600px" src="/images/161117/hiveprofile.png"/>

查看hive所有的配置文件，并将一下四个配置文件重命名,如图所示：
<img width="600px" src="/images/161117/hiveconfls.png"/>
要对`hive-log4j.properties`和`hive-site.xml`进行配置。

#### (2).配置hive-log4j.properties

	vi /usr/hive/conf/hive-log4j.properties
	在hive.log.dir处设置
	hive.log.dir=/home/hadoop/hive-1.2.1/logs
	其中hive-1.2.1/logs目录同hadoop、zookeeper、hbase一样需要创建，后边会讲统一创建

<img width="600px" src="/images/161117/hiveconfproper.png"/>

#### (3).配置hive-site.xml

在此配置文件中需要配置的地方较多，需要配置:

* hive.metastore.warehouse.dir     #指定hive的数据存储目录，指定的是HDFS上的位置，默认值：/user/hive/warehouse，为了便于管理，hive-1.2.1文件下创建文件warehouse,在335行左右。
* hive.exec.scratchdir             #指定hive的临时数据目录，默认位置为：/tmp/hive-${user.name}。为了便于管理，hive-1.2.1文件下创建文件scratchdir。在47行左右。
* hive.exec.local.scratchdir       #hadoop用户下的，在51行左右
* hive.querylog.location           #log目录，在1320行左右
* hive.downloaded.resources.dir    #在56行左右，如果不配置的话会报错。
* javax.jdo.option.ConnectionURL   #指定hive连接的数据库的数据库连接字符串，在396行左右。
* javax.jdo.option.ConnectionDriverName   #指定驱动的类入口名称，在790行左右
* javax.jdo.option.ConnectionUserName     #数据库用户名，在815行左右。
* javax.jdo.option.ConnectionPassword     #数据库密码，在381行左右

分别进行配置：

hive.metastore.warehouse.dir,需要创建`/home/hadoop/hive-1.2.1/warehouse`目录

	<property>
    	<name>hive.metastore.warehouse.dir</name>
    	<value>/home/hadoop/hive-1.2.1/warehouse</value>
    	<description>location of default database for the warehouse</description>
    </property>

配置如图所示：
<img width="600px" src="/images/161117/hivesitewarehouse.png"/>

hive.exec.scratchdir，需要创建`/home/hadoop/hive-1.2.1/scratchdir`目录

    <property>
    	<name>hive.exec.scratchdir</name>
    	<value>/home/hadoop/hive-1.2.1/scratchdir</value>
    	<description>HDFS root scratch dir for Hive jobs which gets created with write all (733) permission. For each connecting user, an HDFS scratch dir: ${hive.exec.scratchdir}/&lt;username&gt; is created, with ${hive.scratch.dir.permission}.</description>
    </property>

配置如图所示：
<img width="600px" src="/images/161117/hivesitescra.png"/>

hive.exec.local.scratchdir，需要创建`/home/hadoop/hive-1.2.1/scratchdir`目录

    <property>
    	<name>hive.exec.local.scratchdir</name>
    	<value>/home/hadoop/hive-1.2.1/scratchdir</value>
    	<description>Local scratch space for Hive jobs</description>
    </property>

配置如图所示：
<img width="600px" src="/images/161117/hivesitescralocal.png"/>

hive.querylog.location，需要创建`/home/hadoop/hive-1.2.1/logs`目录

	<property>
    	<name>hive.querylog.location</name>
    	<value>/home/hadoop/hive-1.2.1/logs</value>
    	<description>Location of Hive run time structured log file</description>
    </property>

配置如图所示：
<img width="600px" src="/images/161117/hivesitelog.png"/>

hive.downloaded.resources.dir，需要创建`/home/hadoop/hive-1.2.1/resources/`目录

	<property>
    	<name>hive.downloaded.resources.dir</name>
    	<value>/home/hadoop/hive-1.2.1/resources/${hive.session.id}_resources</value>
    	<description>Temporary local directory for added resources in the remote file system.</description>
    </property>

配置如图所示：
<img width="600px" src="/images/161117/hivesiteresource.png"/>
如果不对其进行配置，启动hive时会出现如下错误：
<img width="600px" src="/images/161117/hivesiterecourseerror.png"/>

javax.jdo.option.ConnectionURL
	
	<property>
    	<name>javax.jdo.option.ConnectionURL</name>
    	<value>jdbc:mysql://localhost:3306/hivedb?createDatabaseIfNotExist=true</value>
    	<description>JDBC connect string for a JDBC metastore</description>
    </property>

原始状态为Derby引擎，如图所示：
<img width="600px" src="/images/161117/hivesiteurl1.png"/>
修改为mysql引擎。此处的远程登录地址为Master.Hadoop，而不是localhost，这就是为什么需要在mysql配置中添加远程访问配置了。
<img width="600px" src="/images/161117/hivesiteurl2.png"/>

javax.jdo.option.ConnectionDriverName

	<property>
    	<name>javax.jdo.option.ConnectionDriverName</name>
    	<value>com.mysql.jdbc.Driver</value>
    	<description>Driver class name for a JDBC metastore</description>
    </property>

配置如图所示：
<img width="600px" src="/images/161117/hivesitedriver.png"/>

javax.jdo.option.ConnectionUserName

	<property>
    	<name>javax.jdo.option.ConnectionUserName</name>
    	<value>root</value>
    	<description>Username to use against metastore database</description>
    </property>

配置如图所示：
<img width="600px" src="/images/161117/hivesiteusername.png"/>

javax.jdo.option.ConnectionPassword  

	<property>
    	<name>javax.jdo.option.ConnectionPassword</name>
    	<value>MyNewPass123!</value>
    	<description>password to use against metastore database</description>
    </property>

配置如图所示：
<img width="600px" src="/images/161117/hivesitepass.png"/>

#### (4).将mysql驱动jar包导入/usr/hive/lib下

提前将`mysql-connector-java-5.1.39-bin.jar`传到/usr/local/src下。

	cd /usr/local/src
	ls
	cp mysql-connector-java-5.1.39-bin.jar /usr/hive/lib

<img width="600px" src="/images/161117/hivemysqlconnector.png"/>

至此，hive-site.xml配置完成，还是比较麻烦的，下边开始启动hive服务。

### 3).启动测试Hive

因为之前已经将hive配置到环境变量里边，直接在命令行中输入`hive`即可。
	
	hive

首次输入可能会出现如下警告：
<img width="600px" src="/images/161117/hivesitestart1.png"/>
首先恭喜，出现这个的时候说明已经安装成功了，这是警告不是错误，以后使用是不影响的。大概的意思就是说建立ssl连接，但是服务器没有身份认证，这种方式不推荐使用。可以通过修改`hive-site.xml`配置文件进行修改。还是javax.jdo.option.ConnectionURL这项：改为：

	<property>
    	<name>javax.jdo.option.ConnectionURL</name>
    	<value>jdbc:mysql://Master.Hadoop:3306/hivedb?useSSL=false</value>
    	<description>JDBC connect string for a JDBC metastore</description>
    </property>

如图所示：
<img width="600px" src="/images/161117/hivesiteurl3.png"/>
再进行启动，即可看到没有警告。

	hive

<img width="600px" src="/images/161117/hivesitestart2.png"/>
继续在hive内输入：

	show databases
	show tables

如图所示表明hive安装成功。
<img width="600px" src="/images/161117/hivestart3.png"/>



### 4).Hive与HBase、Zookeeper进行整合

<img width="600px" src="/images/161117/hivesitezoo.png"/>
<img width="600px" src="/images/161117/hivezoocp.png"/>
### 5).Hive与HBase整合测试


更新中。。。。。。
至此，Hive安装配置完成，下一步配置storm+spark


看在我辛苦截图敲代码的份上，喜欢的话打赏一下吧，哈哈。

<img width="400px" src="/images/shoukuan.png"/>