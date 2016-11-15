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

### 2).Hadoop配置

#### (1).第一步，hadoop-env.sh配置

#### (2).第二步，core-site.xml配置

#### (3).第三步，hdfs-site.xml配置

#### (4).第四步，mapred-site.xml配置

#### (5).第五步，yarn-site.xml配置

#### (6).第六步，slaves配置

#### (7).第七步，profile配置Hadoop命令（可省）

### 3).发送给所有slave节点并进行配置
scp /etc/hosts root@Slave1.Hadoop:/etc/
scp -r /usr/java/ root@Slave1.Hadoop:/usr/
scp -r /usr/hadoop/ root@Slave1.Hadoop:/usr/
scp /etc/profile root@Slave1.Hadoop:/etc/
scp -r hadoop-2.7.1/ hadoop@Slave2.Hadoop:~/
登录Slave，
source /etc/profile
su & cd /usr
chown -R hadoop:hadoop hadoop/



### 4).启动验证

#### (1).格式化HDFS文件系统

#### (2).启动Hadoop

#### (3).验证Hadoop




