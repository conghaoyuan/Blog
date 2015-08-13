---
layout: post
title: Linux和Unix 相关操作系统包管理器介绍
categories:
- 随笔
---

# Linux
-------------
## [RedHat](http://baike.baidu.com/view/1139590.htm target="_blank")
## [CentOS](http://baike.baidu.com/view/26404.htm target="_blank")
## [Debian](http://baike.baidu.com/view/40687.htm target="_blank")
## [Ubuntu](http://baike.baidu.com/view/4236.htm target="_blank")

**Ubuntu**（乌班图)是一个以桌面应用为主的Linux操作系统，是基于linux的免费开源桌面PC操作系统，基于 **Debian** GNU/Linux。作为Debian的衍生版，Ubuntu同样采用**`dpkg`**进行软件包管理。

#####介绍
* **`dpkg`**是用来安装.deb文件,但不会解决模块的依赖关系,且不会关心ubuntu的软件仓库内的软件,可以用于安装本地的deb文件。
* **`apt`**会解决和安装模块的依赖问题,并会咨询软件仓库, 但不会安装本地的deb文件, apt是建立在dpkg之上的软件管理工具。
* **`aptitude`**与 apt-get 一样，是 Debian 及其衍生系统极其强大的包管理工具。与 apt-get 不同的是，aptitude在处理依赖问题上更佳一些。举例来说，aptitude在删除一个包时，会同时删除本身所依赖的包。这样，系统中不会残留无用的包，整个系统更为干净。

#####使用

安装软件包：

* dpkg -i package.deb &nbsp;&nbsp;#安装本地软件包，不解决依赖关系
* apt-get install package &nbsp;&nbsp;#在线安装软件包
* apt-get install package --reinstall  &nbsp;&nbsp;#重新安装软件包
* aptitude install package &nbsp;&nbsp;#在线安装软件包
* apitude reinstall package  &nbsp;&nbsp;#重新安装软件包

移除软件包:

* dpkg -r package  &nbsp;&nbsp;#删除软件包
* dpkg -P &nbsp;&nbsp;#删除软件包及配置文件
* apt-get remove package &nbsp;#删除软件包
* apt-get remove package --purge &nbsp;&nbsp;#删除软件包及

配置文件

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





