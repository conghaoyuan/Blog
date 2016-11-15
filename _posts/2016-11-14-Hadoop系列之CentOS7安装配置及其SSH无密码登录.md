---
layout: post
title: Hadoop系列之CentOS7安装配置及其SSH无密码登录
categories:
- 大数据
---

<div class="message">
  由于最近在进行关于Hadoop的学习，开始重新搭建一套Hadoop的环境，和之前自己搭建的环境的区别在于操作系统的版本，本次采用的为CentOS7.2的版本，7相对于6的版本改动较大。一些命令都不太一样，其中在其内核中加了Docker，因此在后期装相关软件时不需要重新安装，应该说也是Hadoop生态体系后期迁移的一个重点。故本次搭建采用CentOS7.2的最新版本。
</div>

## 1.部署环境及相关版本软件

* 操作系统：MAC OS X EI Caption 10.11.6  
* CPU：2.7 GHz Intel Core i5
* 内存：8 GB 1867 MHz DDR3
* 虚拟机：Parallels Desktop 10
* 操作系统：[CentOS-7-x86_64-DVD-1511.ISO](https://www.centos.org/download/)
* JDK：[jdk-8u111-linux-x64.tar.gz](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
* 其他软件：
	hadoop-2.7.1.tar.gz    hbase-1.1.7-bin.tar.gz    apache-hive-1.2.1-bin.tar.gz    zookeeper-3.4.6.tar.gz
	spark-1.5.1-bin-hadoop2.6.tgz    apache-storm-0.9.5.tar.gz
    apache-flume-1.6.0-bin.tar.gz    sqoop-1.4.6.tar.gz    thrift-0.8.0.tar.gz
    mongodb-linux-x86_64-rhel70-3.2.10.tgz    redis-3.2.5.tar.gz


## 2.CentOS7 安装

PD这个软件在mac上算是特别好用的虚拟机了，用的当然是破解版的，目前有PD10的破解版，建议别升级，升级后不能用，也别将mac 升级成 mac sierra，升级后很多破解版的软件无法使用，吐槽一下苹果新版的MBP，那叫一个讽刺，自己的手机无法直接连接自己的电脑。

<img width="600px" src="/images/161114/16111401.png"/>
<img width="600px" src="/images/161114/16111402.png"/>
这里要往上选择一下第一个安装，这样不用检查，省去很多时间。
<img width="600px" src="/images/161114/16111403.png"/>
<img width="600px" src="/images/161114/16111404.png"/>
<img width="600px" src="/images/161114/16111405.png"/>
其中要设置一下时区，以及root的密码，以及一个用户及密码，这里所有的设置的用户为：hadoop，因为Hadoop环境建议不要直接用root用户直接去安装。这里有一点为分区的设置为题，就用默认的就行了。
<img width="600px" src="/images/161114/16111406.png"/>
CentOS7安装有个好处为配置与安装分离，安装完基础的软件包后，再进行系统的配置。
<img width="600px" src="/images/161114/16111407.png"/>

以上为CentOS7安装完毕，然后就可以重启登录了。第一次登录可用root用户登录并且设置。
------------------

	
### (1).设置防火墙
在CentOS6.*的版本及以前，service iptables stop
在CentOS7中
	
	systemctl stop firewalld.service      #停止firewall
	systemctl disable firewalld.service   #禁止firewall开机启动
	
即可，具体详细设置可看笔记[CentOS7防火墙设置]()
### (2).hostname设置
查看当前机器名：
	
	hostname

默认为：
<img width="600px" src="/images/161114/16111407.png"/>
修改`/etc/hostname`文件
	
	vi /etc/hostname
	删除：localhost.localdomain
	添加：Master.Hadoop
	保存退出:wq

###(3).配置ip
CentOS6.*查看ip地址命令为：`ifconfig`，CentOS7修改为：`ip addr`
因为所用PD虚拟机的网络设置为共享网络，故只需要在eth0配置文件将开机启动设置为`yes`即可，因每增加一台虚拟机，PD变会将ip进行自增，且下次开机不会改变。
	
	vi /etc/sysconfig/network-scripts/ifcfg-eth0
	将ONBOOT=no 变为 ONBOOT=yes
	保存退出:wq
	/etc/init.d/network restart    即可

此时，`ip addr`即可查看相应的ip。
本人搭建了三台虚拟机，其ip分别为：
	
	10.211.55.13    master
	10.211.55.14    slave1
	10.211.55.15    slave2

以上为CentOS7的安装及其配置，接下来为SSH无密码登录配置。
-----------------


<<<<<<< HEAD
=======


	

>>>>>>> aa11b4012fed4acdfaa2c912d767ce2143d1cc7a
