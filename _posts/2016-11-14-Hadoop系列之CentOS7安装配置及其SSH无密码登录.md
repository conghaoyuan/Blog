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
----------------------

	
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
<img width="200px" src="/images/161114/16111408.png"/>
修改`/etc/hostname`文件
	
	vi /etc/hostname
	删除：localhost.localdomain
	添加：Master.Hadoop
	保存退出:wq


### (3).配置ip
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
-------------------

## 3.SSH无密码验证配置

<div class="message">
  Hadoop运行过程中需要管理远端Hadoop守护进程，在Hadoop启动以后，NameNode是通过SSH（Secure Shell）来启动和停止各个DataNode上的各种守护进程的。这就必须在节点之间执行指令的时候是不需要输入密码的形式，故我们需要配置SSH运用无密码公钥认证的形式，这样NameNode使用SSH无密码登录并启动DataName进程，同样原理，DataNode上也能使用SSH无密码登录到NameNode。
</div>

默认安装好的系统上已经安装了ssh和rsync，可以通过以下命令查看：

	rpm –qa | grep openssh
	rpm –qa | grep rsync
	如果提示没有安装，则需安装，如果安装跳过该步
	yum install ssh   #安装SSH协议
	yum install rsync  #rsync是一个远程数据同步工具，可通过LAN/WAN快速同步多台主机间的文件）
	systemctl restart  sshd.service  #启动服务

<div class="message">
  SSH无密码原理：Master（NameNode | JobTracker）作为客户端，要实现无密码公钥认证，连接到服务器Salve（DataNode | Tasktracker）上时，需要在Master上生成一个密钥对，包括一个公钥和一个私钥，而后将公钥复制到所有的Slave上。当Master通过SSH连接Salve时，Salve就会生成一个随机数并用Master的公钥对随机数进行加密，并发送给Master。Master收到加密数之后再用私钥解密，并将解密数回传给Slave，Slave确认解密数无误之后就允许Master进行连接了。这就是一个公钥认证过程，其间不需要用户手工输入密码。重要过程是将客户端Master复制到Slave上。
  分为两部分：`Master无密码登录所有的Slave` 和  `所有的Slave无密码登录到Master`    
</div>

	这句话一定要看，一定要看，因为关系到配置ssh效率的问题。
	原理说白了就是：需要将Master的公钥id_rsa.pub追加到所有的Slave的authorized_keys里边，将所有的Slave中的id_rsa.pub追加到Master的authorized_keys里边。
	因此先配置Master和所有的Slave机器的sshd_config
	vi /etc/ssh/sshd_config
	将下边三个选项进行配置
	RSAAuthentication yes # 启用 RSA 认证
	PubkeyAuthentication yes # 启用公钥私钥配对认证方式
	AuthorizedKeysFile .ssh/authorized_keys # 公钥文件路径（和上面生成的文件同）
	然后分别在所有机器的hadoop用户的~/目录下分别建立.ssh文件夹并将其权限设为700 chmod 700 .ssh 
	~/ 目录为登录hadoop用户后，直接cd下的目录

### 1).Master无密码登录所有的Slave

1.在Master上生成密码对
在Master节点上执行：

	ssh-keygen -t rsa -P ''

这条命是生成其无密码密钥对，询问其保存路径时直接回车采用默认路径。生成的密钥对：id_rsa和id_rsa.pub，默认存储在"/home/hadoop/.ssh"目录下
<img src="/images/161115/16111501.png"/>
查看"/home/hadoop/"下是否有".ssh"文件夹，且".ssh"文件下是否有两个刚生产的无密码密钥对。
<img src="/images/161115/16111502.png"/>
2.接着在Master节点上做如下配置，把id_rsa.pub追加到授权的key里面去。

	cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

<img src="/images/161115/16111503.png"/>
3.修改文件"authorized_keys"权限
权限的设置非常重要，因为不安全的设置安全设置，会让你不能使用RSA功能

	chmod 600 ~/.ssh/authorized_keys

<img src="/images/161115/16111504.png"/>
4.使用hadoop普通用户验证是否成功

	ssh localhost

<img src="/images/161115/16111505.png"/>

从上图中得知无密码登录本级已经设置完毕，接下来的事儿是把公钥复制所有的Slave机器上。使用下面的命令格式进行复制公钥：

	scp ~/.ssh/id_rsa.pub 远程用户名@远程服务器IP:~/
	例如：
	scp ~/.ssh/id_rsa.pub hadoop@10.211.55.14:~/.ssh
	scp ~/.ssh/id_rsa.pub hadoop@10.211.55.15:~/.ssh

然后分别登录Slave1和Slave2两台机器，将`id_rsa.pub` 追加到`authorized_keys`中，并将其权限设置为`600`后删除id_rsa.pub

	cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
	chmod 600 ~/.ssh/authorized_keys
	rm ~/.ssh/id_rsa.pub

5.在Master上hadoop用户登录Slave上的Hadoop用户验证是否成功

	ssh hadoop@10.211.55.14
	ssh hadoop@10.211.55.15
<img src="/images/161115/16111506.png"/>
不在提示让输入密码，表示Master登录所有Slave节点ssh配置成功。


### 2).Slave无密码登录所有的Master


