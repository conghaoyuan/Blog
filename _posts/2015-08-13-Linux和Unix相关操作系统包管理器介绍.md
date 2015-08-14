---
layout: post
title: Linux和Unix 相关操作系统包管理器介绍
categories:
- 随笔
---

# Linux
<div class="message">
在 GNU/Linux( 以下简称 Linux) 操作系统中，RPM 和 DPKG 为最常见的两类软件包管理工具，他们分别应用于基于 RPM 软件包的 Linux 发行版本和 DEB 软件包的 Linux 发行版本。软件包管理工具的作用是提供在操作系统中安装，升级，卸载需要的软件的方法，并提供对系统中所有软件状态信息的查询。
RPM 全称为 Redhat Package Manager，最早由 Red Hat 公司制定实施，随后被 GNU 开源操作系统接受并成为很多 Linux 系统 (RHEL) 的既定软件标准。与 RPM 进行竞争的是基于 Debian 操作系统 (UBUNTU) 的 DEB 软件包管理工具－ DPKG，全称为 Debian Package，功能方面与 RPM 相似。二者之具体比较不在本文范围之内。
</div>
-------------
## <a href="http://baike.baidu.com/view/1139590.htm" target="_blank">RedHat</a>&&<a href="http://baike.baidu.com/view/26404.htm" target="_blank">CentOS</a>

**RedHat**最早发行的个人版本的Linux，自从Red Hat 9.0版本发布后，RedHat 公司就不再开发桌面版的 Linux发行套件，Red Hat Linux停止了开发，而将全部力量集中在服务器版的开发上，也就是 (RHEL)Red Hat Enterprise Linux 版。2004年4月30日，Red Hat公司正式停止对Red Hat 9.0版本的支援，标志著Red Hat Linux的正式完结。原本的桌面版Red Hat Linux发行套件则与来自开源社区的 Fedora 计划合并，成为 Fedora Core 发行版本。目前Red Hat分为两个系列：由Red Hat公司提供收费技术支持和更新的Red Hat Enterprise Linux(RHEL)，以及由社区开发的免费的Fedora Core。`特点：面向个人桌面应用系统，采用基于rpm/yum管理软件包。`    
**CentOS** 是Linux发行版之一，它是来自于RHEL(Red Hat Enterprise Linux)依照开放源代码规定释出的源代码所编译而成,而且在RHEL的基础上修正了不少已知的 Bug ，相对于其他 Linux 发行版，其稳定性值得信赖。由于出自同样的源代码，因此有些要求高度稳定性的服务器以CentOS替代商业版的Red Hat Enterprise Linux使用。两者的不同，在于CentOS并不包含封闭源代码软件。

#### 介绍
* **`rpm`** 用来安装下载到本地的rpm包，通常表现为以 .rpm 扩展名结尾的文件，例如 package.rpm 。对其操作，需要使用 rpm 命令。rpm包的安装有一个很大的缺点就是文件的关联性太大，有时候装一个软件要安装很多其他的软件包.
* **`yum`** 能在线下载并安装rpm包,能更新系统,且还能自动处理包与包之间的依赖问题,这个是rpm 工具所不具备的。YUM的另一个功能是进行系统中所有软件的升级。如上所述，YUM的RPM包来源于源空间，在RHEL中由/etc/yum.repos.d/目录中的.repo文件配置指定。YUM的系统配置文件位于/etc/yum.conf。

#### rpm使用

rpm命令参数：

* -q在系统中查询软件或查询指定rpm包的内容信息
* -i在系统中安装软件
* -U在系统中升级软件
* -e在系统中卸载软件
* -h用#(hash)符显示rpm安装过程
* -v详述安装过程
* -p表明对RPM包进行查询，通常和其它参数同时使用，如：
* -qlp查询某个RPM包中的所有文件列表, 查看软件包将会在系统里安装哪些部分
* -qip查询某个RPM包的内容信息,系统将会列出这个软件包的详细资料，包括含有多少个文件、各文件名称、文件大小、创建时间、编译日期等信息。

安装rpm包:

* rpm -ivh package.rpm

升级rpm包:

* rpm -Uvh package.rpm

卸载rpm包:

* rpm -ev package

查询已安装rpm包:

* rpm -qa｜grep package
* rpm -qf <文件名> &nbsp;&nbsp;#快速判定某个文件属于哪个软件包
* rpm -Va  &nbsp;&nbsp;#Linux将为你列出所有损坏的文件

#### yum使用

软件库中搜索软件包:

* yum search package-name

获得详细的输出信息:

* yum provides httpd[package-name]

提供所有软件组列表:

* yum grouplist
* yum groupinstall PHP Support

只下载rpm不安装:

* yum install yum-downloadonly &nbsp;&nbsp;#首先安装yum插件downloadonly
* yum install httpd-devel -downloadonly
* yum install httpd-devel -downloadonly -downloaddir=/opt &nbsp;&nbsp;#默认情况下软件包会被下载到/var/cache/yum/目录，但是你可以添加额外选项将其下载到指定位置

安装RPM包:

* yum -y install package-name

升级rpm包:

* yum -y update &nbsp;&nbsp;#升级过程中所有问答形式
* yum update &nbsp;&nbsp;#升级所有可升级软件
* yum update package-name &nbsp;&nbsp;#升级指定软件

卸载rpm包:

* yum remove package-name

列出已安装rpm包:

* yum list installed

列出可用的软件库，通过它们可以查询、安装和更新软件包:

* yum repolist 

列出系统中可升级的所有软件:

* yum check-update

将下载的软件包和存储在cache中的header清除

*yum clean

yum其他:

```
yum常用的源
1) 自动选择最快的源
由于yum中有的mirror速度是非常慢的，如果yum选择了这个mirror，这个时候yum就会非常慢，对此，可以下载fastestmirror插件，它会自动选择最快的mirror：
#yum install yum-fastestmirror
配置文件：（一般不用动）/etc/yum/pluginconf.d/fastestmirror.conf
你的yum镜像的速度测试记录文件：/var/cache/yum/timedhosts.txt
(2)使用图形界面的yum
如果觉得命令行的yum不方便，那么可以使用图形化的yumex，这个看起来更方便，因为可以自由地选择软件仓库：
#yum install yumex
然后在系统工具中就可以看到yum extender了。实际上系统自带的“添加/删除程序“也可以实现图形化的软件安装，但有些yumex的功能它没有。
```

## <a href="http://baike.baidu.com/view/40687.htm" target="_blank">Debian</a>&&<a href="http://baike.baidu.com/view/4236.htm" target="_blank">Ubuntu</a>

**Debian** 项目众多内核分支中以Linux宏内核为主，而且 Debian开发者 所创建的操作系统中绝大部分基础工具来自于GNU工程 ，因此 “Debian” 常指Debian GNU/Linux。是社区类Linux的典范，是迄今为止最遵循GNU规范的Linux系统。
**Ubuntu**（乌班图)是一个以桌面应用为主的Linux操作系统，是基于linux的免费开源桌面PC操作系统，基于 **Debian** GNU/Linux。作为Debian的衍生版，Ubuntu同样采用**`dpkg`**进行软件包管理。

##### 介绍
* **`dpkg`**是用来安装.deb文件,但不会解决模块的依赖关系,且不会关心ubuntu的软件仓库内的软件,可以用于安装本地的deb文件。
* **`apt`**会解决和安装模块的依赖问题,并会咨询软件仓库, 但不会安装本地的deb文件, apt是建立在dpkg之上的软件管理工具。
* **`aptitude`**与 apt-get 一样，是 Debian 及其衍生系统极其强大的包管理工具。与 apt-get 不同的是，aptitude在处理依赖问题上更佳一些。举例来说，aptitude在删除一个包时，会同时删除本身所依赖的包。这样，系统中不会残留无用的包，整个系统更为干净。

##### 使用

dpkg命令参数：

* -l 在系统中查询软件内容信息
* --info 在系统中查询软件或查询指定 rpm 包的内容信息
* -i 在系统中安装 / 升级软件
* -r 在系统中卸载软件 , 不删除配置文件
* -P 在系统中卸载软件以及其配置文件

安装deb软件包：

* dpkg -i package.deb &nbsp;&nbsp;#安装本地软件包，不解决依赖关系
* sudo apt-get install package &nbsp;&nbsp;#在线安装软件包
* sudo apt-get install package --reinstall  &nbsp;&nbsp;#重新安装软件包
* aptitude install package &nbsp;&nbsp;#在线安装软件包
* apitude reinstall package  &nbsp;&nbsp;#重新安装软件包

移除软件包:

* dpkg -r package  &nbsp;&nbsp;#删除软件包
* dpkg -P &nbsp;&nbsp;#删除软件包及配置文件
* sudo apt-get remove package &nbsp;#删除软件包
* sudo apt-get remove package --purge &nbsp;&nbsp;#删除软件包及配置文件
* aptitude remove package &nbsp;#删除软件包
* apitude purge package &nbsp;&nbsp;#删除软件包及配置文件

自动移除软件包:

* sudo apt-get autoremove &nbsp;&nbsp;#删除不再需要的软件包
注：aptitude 没有，它会自动解决这件事

清除下载的软件包:

* sudo apt-get clean &nbsp;&nbsp;#清除 /var/cache/apt/archives 目录
* aptitude clean &nbsp;&nbsp;#同上
* sudo apt-get autoclean &nbsp;&nbsp;#清除 /var/cache/apt/archives 目录，不过只清理过时的包
* aptitude autoclean &nbsp;&nbsp;#同上

编译相关:   

* sudo apt-get source package &nbsp;&nbsp;#获取源码
* sudo apt-get build-dep package &nbsp;&nbsp;#解决编译源码 package 的依赖关系
* aptitude build-dep package &nbsp;&nbsp;#解决编译源码 package 的依赖关系

更新源:

* sudo apt-get update &nbsp;&nbsp;#更新源
* aptitude update &nbsp;&nbsp;#同上

更新系统:

* sudo apt-get upgrade &nbsp;&nbsp;#更新已经安装的软件包
* aptitude safe-upgrade &nbsp;&nbsp;#同上
* sudo apt-get dist-upgrade &nbsp;&nbsp;#升级系统
* aptitude full-upgrade &nbsp;&nbsp;#同

Ubuntu中的高级包管理方法apt-get:  

<img width="600px" src="/images/15813/aptitude.png"/>  

常用的APT命令参数:

<img width="600px" src="/images/15813/apt.png">

-------------------


# Unix

<div>
UNIX分为两大类，分别是由厂商支持的systemV 系统和BSD系统，具体有：<br/>
SYSTEM V 系统：<br/>
<a href="http://baike.baidu.com/view/3247518.htm" target="_blank">SCO UNIX</a> &nbsp;&nbsp;&nbsp;&nbsp;
<a href="http://baike.baidu.com/view/58963.htm" target="_blank">HP UNIX</a> &nbsp;&nbsp;&nbsp;&nbsp;
<a href="http://baike.baidu.com/subview/329359/5113665.htm" target="_blank">SUN UNIX (SOLARIS )</a>&nbsp;&nbsp;&nbsp;&nbsp;
<a href="http://baike.baidu.com/view/3155886.htm" target="_blank">IBM UNIX (AIX)</a><br/><br/>

BSD系统：<br/>
<a href="http://baike.baidu.com/view/21459.htm" target="_blank">FreeBSD</a>&nbsp;&nbsp;&nbsp;&nbsp;
<a href="http://baike.baidu.com/view/337596.htm" target="_blank">OpenBSD</a>&nbsp;&nbsp;&nbsp;&nbsp;
<a href="http://baike.baidu.com/view/288469.htm" target="_blank">NetBSD</a>&nbsp;&nbsp;&nbsp;&nbsp;
<a href="http://baike.baidu.com/view/24778.htm" target="_blank">APPle UNIX(MAC OS bsd内核）</a><br/>
开发中用到最多的便是Mac os X ,故只介绍mac的包管理器。<b>Homebrew</b>&&<b>Macports</b>
</div>
-------------

## <a href="http://baike.baidu.com/view/24778.htm" target="_blank">Mac</a>

##### 介绍

* **`Homebrew`** 是(Ruby)[https://www.ruby-lang.org/en/downloads/]开发的智能包管理系统,能够判断已有的依赖组件,不用重新下载一套组件,Homebrew 本身是使用Git管理的,升级非常方便.
* **`Macports`** 是自成一派,他的所有组件都会安装到/opt/目录下,带来的问题就是很多系统已经存在的组件也会重新下载,费时费空间.

--------------
#### Homebrew安装使用

##### 安装 
	ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"
##### 自检
	brew doctor
##### 常用命令(以安装php为例)

* brew update                        #更新brew可安装包，建议每次执行一下
* brew search php55                  #搜索php5.5
* brew tap josegonzalez/php          #安装扩展<gihhub_user/repo>   
* brew tap                           #查看安装的扩展列表
* brew install php55                 #安装php5.5
* brew remove  php55                 #卸载php5.5
* brew upgrade 						 #升级所有package
* brew upgrade php55                 #升级php5.5
* brew options php55                 #查看php5.5安装选项
* brew info    php55                 #查看php5.5相关信息
* brew home    php55                 #访问php5.
* brew deps    php55                 #显示php55包依赖
* brew list							 #列出brew安装的软件
* brew services list                 #查看系统通过 brew 安装的服务
* brew services cleanup              #清除已卸载无用的启动配置文件
* brew services restart php55        #重启php-fpm
* brew --cache					     #查看下载的package存放目录  /Library/Caches/Homebrew
* brew server *						 #启动web服务器，可以通过浏览器访问http://localhost:4567/ 来同网页来管理包  
	注:如果提示 

    Error: Sinatra required but not found

    To install: /usr/bin/gem install sinatra

    安装一下sinatra：sudo /usr/bin/gem install sinatra

##### 删除Homebrew
```
cd `brew –prefix`       #找到 Homebrew

rm -rf Cellar			#删除Homebrew 安装的程序包和库

brew prune				#删除无效 Link	

rm -rf Library .git .gitignore bin/brew README.md share/man/man1/brew

rm -rf ~/Library/Caches/Homebrew   #删除缓存
```

------------
#### Macports安装使用

##### 安装

1.访问官方网站[http://www.macports.org/install.php]，下载相关系统dmg安装,一步一步即可。  
2.通过source安装,可打开[http://distfiles.macports.org/MacPorts/]查看macports版本信息，选定一个版本安装即可。如下载最新版本2.3.3

	wget http://distfiles.macports.org/MacPorts/MacPorts-2.3.3.tar.gz
	tar zxvf MacPorts-2.3.3.tar.gz
	cd MacPorts-2.3.3.tar.gz
	./configure && make && sudo make install
	cd ../
	rm -rf MacPorts-2.3.3*

然后将/opt/local/bin和/opt/local/sbin添加到$PATH搜索路径中,在编辑/etc/profile文件中，加上

	export PATH=/opt/local/bin:$PATH
	export PATH=/opt/local/sbin:$PATH

##### MacPorts使用

* port help  selfupdate   #查看某个（selfupdate）指令的帮助说明
* sudo port selfupdade    #同步本地和全球的软件树,有必要时,同时升级mac port自己.
* sudo port sync          #同步本地和全球的ports tree,但不检查自己是否有更新.
* port list     #列出当前所有的可用软件,如果想查找是否有自己想要的软件时,还是使用search指令方便一些
* port search   #模糊搜索,可以匹配软件名字和描述,还有更高级的用法,具体看port help search
* port info     #查看一款软件的详细信息 port info php
* port deps		#查看一款软件的依赖关系 port deps apache2
* port variants #在安装软件前,用这个命令查看软件是否有多个版本.再选择安装一个合适的版本
* sudo port install  #安装软件命令,安装前最好使用variants命令查看是否有多个不同版本.
* port clean    #删除一些编译软件时留下的临时文件.
* port uninstall  #卸载软件命令,如果这个软件依赖与另外的一款软件,默认不删除它依赖的软件,
* port -f uninstall vile	#使用参数 -f (force) 可以强行删除它依赖的软件.
* port contents		#显示软件安装后的文件列表.
* port installed	#列出全部或者指定的已经安装的软件.
* port outdated		#查看已经安装的软件是否有更新,在执行这个指令前,先执行selfupdate 或者 sync更新软件树
* port upgrade		#更新软件,默认一起更新它依赖的所有软件,如果想不更新它依赖的软件,使用 -n 参数,默认不删除旧软件版本,只是使旧软件变成无效状态,如果想要一起删除旧软件,使用 －u 参数

        port upgrade gnome
        port -n upgrade gnome
        更新所有的可更新软件
        sudo port upgrade outdated
        更新软件同时删除旧版本软件
        port -u upgrade vile
* port dependents	#查看哪些软件时依赖与这个软件的.删除一个软件时候，最好先执行一下这个命令


##### 删除MacPorts

```
sudo port -f uninstall installed

sudo rm -rf

/opt/local

/Applications/DarwinPorts

/Applications/MacPorts

/Library/LaunchDaemons/org.macports.*

/Library/Receipts/DarwinPorts*.pkg

/Library/Receipts/MacPorts*.pkg

/Library/StartupItems/DarwinPortsStartup

/Library/Tcl/darwinports1.0

/Library/Tcl/macports1.0

~/.macports
```
