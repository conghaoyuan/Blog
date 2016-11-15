---
layout: post
title: Hadoop系列之（二）JDK和Hadoop安装配置
categories:
- 大数据
---

<div class="message">
  
</div>

## 1.JDK安装配置
之前在有篇博客是搭建apache tomcat + nutch + solr的已经讲过jdk的详细搭建，此次在这里采用第一种搭建方式，即在`/etc/profile`里进行环境变量的配置。

### 1).JDK解压安装
我所有的软件包，全部在mac上通过terminal下的scp发送到master上了，全部放在`/usr/local/src`下，如下图：
<img width="600" src="/images/161115/srcsoftlist.png" />



### 2).profile配置

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

### 3).所有slave节点配置
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




