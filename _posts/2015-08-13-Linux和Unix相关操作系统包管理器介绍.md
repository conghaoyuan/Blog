---
layout: post
title: Linux和Unix 相关操作系统包管理器介绍
categories:
- 随笔
---
<div class="message">
在 GNU/Linux( 以下简称 Linux) 操作系统中，RPM 和 DPKG 为最常见的两类软件包管理工具，他们分别应用于基于 RPM 软件包的 Linux 发行版本和 DEB 软件包的 Linux 发行版本。软件包管理工具的作用是提供在操作系统中安装，升级，卸载需要的软件的方法，并提供对系统中所有软件状态信息的查询。
RPM 全称为 Redhat Package Manager，最早由 Red Hat 公司制定实施，随后被 GNU 开源操作系统接受并成为很多 Linux 系统 (RHEL) 的既定软件标准。与 RPM 进行竞争的是基于 Debian 操作系统 (UBUNTU) 的 DEB 软件包管理工具－ DPKG，全称为 Debian Package，功能方面与 RPM 相似。二者之具体比较不在本文范围之内。
</div>
# Linux
-------------
## <a href="http://baike.baidu.com/view/1139590.htm" target="_blank">RedHat</a>&&<a href="http://baike.baidu.com/view/26404.htm" target="_blank">CentOS</a>

**RedHat**最早发行的个人版本的Linux，自从Red Hat 9.0版本发布后，RedHat 公司就不再开发桌面版的 Linux发行套件，Red Hat Linux停止了开发，而将全部力量集中在服务器版的开发上，也就是 (RHEL)Red Hat Enterprise Linux 版。2004年4月30日，Red Hat公司正式停止对Red Hat 9.0版本的支援，标志著Red Hat Linux的正式完结。原本的桌面版Red Hat Linux发行套件则与来自开源社区的 Fedora 计划合并，成为 Fedora Core 发行版本。目前Red Hat分为两个系列：由Red Hat公司提供收费技术支持和更新的Red Hat Enterprise Linux(RHEL)，以及由社区开发的免费的Fedora Core。`特点：面向个人桌面应用系统，采用基于rpm/yum管理软件包。`
**CentOS** 是Linux发行版之一，它是来自于RHEL(Red Hat Enterprise Linux)依照开放源代码规定释出的源代码所编译而成,而且在RHEL的基础上修正了不少已知的 Bug ，相对于其他 Linux 发行版，其稳定性值得信赖。由于出自同样的源代码，因此有些要求高度稳定性的服务器以CentOS替代商业版的Red Hat Enterprise Linux使用。两者的不同，在于CentOS并不包含封闭源代码软件。

#### 介绍
* **`rpm`** 用来安装下载到本地的rpm包，通常表现为以 .rpm 扩展名结尾的文件，例如 package.rpm 。对其操作，需要使用 rpm 命令。rpm包的安装有一个很大的缺点就是文件的关联性太大，有时候装一个软件要安装很多其他的软件包.
* **`yum`** 能在线下载并安装rpm包,能更新系统,且还能自动处理包与包之间的依赖问题,这个是rpm 工具所不具备的。

#### 使用

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
* apt-get install package &nbsp;&nbsp;#在线安装软件包
* apt-get install package --reinstall  &nbsp;&nbsp;#重新安装软件包
* aptitude install package &nbsp;&nbsp;#在线安装软件包
* apitude reinstall package  &nbsp;&nbsp;#重新安装软件包

移除软件包:

* dpkg -r package  &nbsp;&nbsp;#删除软件包
* dpkg -P &nbsp;&nbsp;#删除软件包及配置文件
* apt-get remove package &nbsp;#删除软件包
* apt-get remove package --purge &nbsp;&nbsp;#删除软件包及配置文件
* aptitude remove package &nbsp;#删除软件包
* apitude purge package &nbsp;&nbsp;#删除软件包及配置文件

自动移除软件包:

* apt-get autoremove &nbsp;&nbsp;#删除不再需要的软件包
注：aptitude 没有，它会自动解决这件事

清除下载的软件包:

* apt-get clean &nbsp;&nbsp;#清除 /var/cache/apt/archives 目录
* aptitude clean &nbsp;&nbsp;#同上
* apt-get autoclean &nbsp;&nbsp;#清除 /var/cache/apt/archives 目录，不过只清理过时的包
* aptitude autoclean &nbsp;&nbsp;#同上

编译相关:   

* apt-get source package &nbsp;&nbsp;#获取源码
* apt-get build-dep package &nbsp;&nbsp;#解决编译源码 package 的依赖关系
* aptitude build-dep package &nbsp;&nbsp;#解决编译源码 package 的依赖关系

更新源:

* apt-get update &nbsp;&nbsp;#更新源
* aptitude update &nbsp;&nbsp;#同上

更新系统:

* apt-get upgrade &nbsp;&nbsp;#更新已经安装的软件包
* aptitude safe-upgrade &nbsp;&nbsp;#同上
* apt-get dist-upgrade &nbsp;&nbsp;#升级系统
* aptitude full-upgrade &nbsp;&nbsp;#同


# Unix
-------------





