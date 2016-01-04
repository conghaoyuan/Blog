---
layout: post
title: mac 安装opencv
categories:
- 随笔
---

1.可用homebrew进行安装

```
brew update
brew tap homebrew/homebrew-science
brew install opencv
```
用macport同理


现在问题来了，brew的方法在Yosemite下可能出现问题。
在yosemite系统下brew install opencv之前安装cmake的一步会失败。而自己安装了cmake，brew并不会识别到。对于这种brew失效的情况.

2.源码安装

* 首先确保Xcode已安装且已经安装command line tool，Xcode5之后的版本应该直接自带了。
* 然后开始安装，先安装cmake，直接官网下源码。解压后cd到解压的文件夹，然后

```
./bootstrap 
make 
make install
```
于是cmake就搞定了。


* 然后安装opencv，还是[官网下源码包解压](http://opencv.org/)，进入解压后的文件夹

```
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D BUILD_PYTHON_SUPPORT=ON -D BUILD_EXAMPLES=ON ..
make
sudo make install
```
于是opencv就装上了。

* 然后设置一下路径，在home下，也就是～下

```
cd
vim .bash_profile
在.bash_profile中添加
export PYTHONPATH=/usr/local/lib/python2.7/site-packages/
保存退出
source.bash_profile
```

这个设置的路径不一定相同，“python2.7”这种视版本而定。如果文件夹正确的话里面会有cv2.so。
最后在～下执行

``搞定``

国内mac安opencv的教程基本都是brew法，brew一不灵了，顿时事情就麻烦了。这里把非brew的直接安装的方法一并总结，以后再有人遇到问题就不用麻烦了。