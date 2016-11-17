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

## 3.Hive安装配置
因为Hive为SQL的解析引擎，故需要数据库作为`存储引擎`，Hive默认使用内嵌`derby`数据库作为存储引擎，但`Derby`引擎的缺点为：一次只能打开一个会话，因此后边我们会把`Mysql`作为外置存储引擎，多用户可同时访问。
这里要说明一下，Hive只需要在一台机器上安装即可，原则上Hive可以在任意一台机器上安装，有的可能考虑Master的服务较多，会选择一台datanode进行安装，在这里我们就选择在Master上安装即可，同样，Mysql也只需要在Master上安装。

### 1).安装mysql

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
	

### 2).安装Hive


#### (1).安装Hive

#### (2).配置Hive

#### (3).启动测试Hive



至此，Zookeeper、HBase、Hive安装配置完成，下一步配置storm+spark