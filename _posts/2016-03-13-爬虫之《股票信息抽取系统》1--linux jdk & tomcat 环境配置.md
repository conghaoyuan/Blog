---
layout: post
title: 爬虫之《股票信息抽取系统》1--linux jdk & tomcat 环境配置
categories:
- JAVA
---

<div class="message">
环境说明，个人学习网络爬虫，后期会涉及到Nutch 和 Lucene，个人环境为 Mac 上装有 Paralles Desktop 10 虚拟机，虚拟了一个 CentOS6.6 系统，准备在CentOS中搭建jdk+tomcat+svn。
</div>

###一. 下载解压安装jdk 
在mac上下载进入oracle官网下载jdk，亦可直接下载如下：

	wget http://download.oracle.com/otn-pub/java/jdk/8u73-b02/jdk-8u73-linux-i586.tar.gz
	
下载后执行命令

	scp jdk-8u73-linux-i586.tar.gz root@10.37.129.3:/usr/local/src
	
将jdk复制到Linux /usr/local/src下，可以打开mac另一terminal窗口，ssh登录到linux
	
	ssh root@10.37.129.3
	
前期是自己的mac和Linux环境ping通，这个不再这里赘述，如果不通可以修改自己的linux ip，这里用的是hots-only链接方式，mac虚拟网卡为vnic1

系统进入后，可以在/usr/local/src下看到jdk的包，将其安装在/usr/local/下

	cd /usr/local/src
	cp jdk-8u73-linux-i586.tar.gz ../
	tar zxvf jdk-8u73-linux-i586.tar.gz
	
解压后便可看到一个jdk1.8.0_73文件夹，接下来为jdk配置环境变量。有三种形式，<b>推荐第二种</b>；

###二. 配置jdk环境变量 

```

1. PATH环境变量。作用是指定命令搜索路径，在shell下面执行命令时，它会到PATH变量所指定的路径中查找看是否能找到相应的命令程序。我们需要把 jdk安装目录下的bin目录增加到现有的PATH变量中，bin目录中包含经常要用到的可执行文件如javac/java/javadoc等待，设置好 PATH变量后，就可以在任何目录下执行javac/java等工具了。 
2. CLASSPATH环境变量。作用是指定类搜索路径，要使用已经编写好的类，前提当然是能够找到它们了，JVM就是通过CLASSPTH来寻找类的。我们 需要把jdk安装目录下的lib子目录中的dt.jar和tools.jar设置到CLASSPATH中，当然，当前目录“.”也必须加入到该变量中。 
3. JAVA_HOME环境变量。它指向jdk的安装目录，Eclipse/NetBeans/Tomcat等软件就是通过搜索JAVA_HOME变量来找到并使用安装好的jdk。 

```
####三种配置环境变量的方法
######1. 修改/etc/profile文件 
如果你的计算机仅仅作为开发使用时推荐使用这种方法，因为所有用户的shell都有权使用这些环境变量，可能会给系统带来安全性问题。 

	vim /etc/profile 
	在profile文件末尾加入： 
	export JAVA_HOME=/usr/local/jdk1.8.0_73
	export PATH=$JAVA_HOME/bin:$PATH
	export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 
	保存退出后
	source profile	更新配置文件

注解 :

a. 要将 /usr/local/jdk1.8.0_73改为你的jdk安装目录 
b. linux下用冒号“:”来分隔路径 
c. $PATH / $CLASSPATH / $JAVA_HOME 是用来引用原来的环境变量的值 
在设置环境变量时特别要注意不能把原来的值给覆盖掉了，这是一种 
常见的错误。 
d. CLASSPATH中当前目录“.”不能丢,把当前目录丢掉也是常见的错误。 
e. export是把这三个变量导出为全局变量。 
f. 大小写必须严格区分。 

######2. 修改.bash_profile文件 (推荐)

这种方法更为安全，它可以把使用这些环境变量的权限控制到用户级别，如果你需要给某个用户权限使用这些环境变量，你只需要修改其个人用户主目录下的.bash_profile文件就可以了。 用文本编辑器打开用户目录下的.bash_profile文件,在.bash_profile文件末尾加入： 

	export JAVA_HOME=/usr/local/jdk1.8.0_73
	export PATH=$JAVA_HOME/bin:$PATH
	export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
	保存退出后更新配置文件
	source .bash_profile

#####3. 直接在shell下设置变量 
不赞成使用这种方法，因为换个shell，你的设置就无效了，因此这种方法仅仅是临时使用，以后要使用的时候又要重新设置，比较麻烦。 
只需在shell终端执行下列命令： 

	export JAVA_HOME=/usr/local/jdk1.8.0_73
	export PATH=$JAVA_HOME/bin:$PATH
	export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

###三. 测试jdk 
在terminal中输入

	java -version 出现

	java version "1.8.0_73"
	Java(TM) SE Runtime Environment (build 1.8.0_73-b02)
	Java HotSpot(TM) Client VM (build 25.73-b02, mixed mode)
	 
配置成功

###四. 下载解压安装tomcat

跟jdk同样的方法，我这里下载的是apache-tomcat-8.0.28版本的。用scp 命令复制到Linux上。

	scp apache-tomcat-8.0.28.tar.gz root@10.37.129.3:/usr/local/src
	
ssh进入Linux

	cd /usr/local/src
	cp apache-tomcat-8.0.28.tar.gz ../
	tar zxvf apache-tomcat-8.0.28.tar.gz
	建立软链
	ln -s apache-tomcat-8.0.28 tomcat
	
这样便于日后更新tomcat版本。

###五. 配置tomcat

打开/usr/local/tomcat/bin

	cd /usr/local/tomcat/bin
	vim catalina.sh
	在第一行添加
	CATALINA_HOME=/usr/local/apache-tomcat-8.0.28/
	保存退出
	
最好在.bash_prifile下进行配置tomcat，这样启动tomcat就不用单独进入tomcat目录。

	export CATALINA_HOME=/usr/local/tomcat
	
	在PATH的JAVA_HOME后边加上CATALINA_HOME/bin
	
	export PATH=$JAVA_HOME/bin:$CATALINA_HOME/bin:$PATH
	
这样在任何目录下都可以使用startup.sh 来启动tomcat和shutdown.sh 来关闭tomcat

###六. 配置完无法访问问题

如果输入上面url访问失败，即tomcat启动失败，请用下面的方法来尝试处理

####1. 权限问题，用户权限和文件是否有可执行权限。

	a.普通用户权限一般不足，用chmod命令给用户加权限,我是用root用户来进行安装的，因此没有遇到这个问题。
	b.文件的权限不够，大部分时候是没有可执行权限。我在安装过程中失败后，给下面文件(xxxxx.bin）文件夹中所有文件赋予了可执行权限。可用下面的命令。
	
	# chmod 777 "文件名" (如：#chmod 777 startup.sh)
####2. 防火墙和端口问题 查看tomcat的8080端是否开启
首先确定是不是防火墙问题，可以运行下面命令将防火墙服务关闭，然后再访问看是否正常。如果正常，说明是防火墙问题，我安装过程就是遇到这个问题，后来发现时防火墙问题，用下面方法解决掉了。

	关闭服务器的防火墙服务命令
	service iptables stop
	
	开启服务器的防火墙服务命令
	service iptables start
	
	编辑和开启防火墙相应端口命令
	vim /ect/sysconfig/iptables
	查看端口是否被占用，查看端口命令
	netstat -pan|gerp 8080
	
	查看Tomcat进程命令
	ps -ef|grep tomcat
	
	杀死一个进程命令
	kill 进程id (注：呵呵，感觉比windows下简单多了,kill you, hehe)
	
	查看系统初始所有服务命令
	cd /etc/rc.d/init.d
	ls
	
	挂载服务，删除服务，服务列表可以通过下面命令查看到
	chkconfig -h

配置完后再mac得chrome下输入10.37.129.3:8080即可访问,亦可在/etc/hosts下配置域名指定10.37.129.3 ，访问如下：

<img width="600px" src="/images/160313/tomcat.png" /> 

