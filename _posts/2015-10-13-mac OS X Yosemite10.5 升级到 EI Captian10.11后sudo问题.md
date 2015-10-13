---
layout: post
title: mac OS X Yosemite10.5 升级到 EI Captian10.11后sudo问题
categories:
- 随笔
---


今天把 Mac 升级了 OS x EI Captian10.11 ，10.11增加了很多新的功能，个人试着分屏挺好用。  

更新系统后大多数软件都没有丢失，试了几个，brew 等。但是gcc不行。包括之前电脑上配置的apache和虚拟主机都不行了。所以要重新配置一下。  

在重装yaf的时候，最后执行 sudo make install 时出现错误，提示yaf.so 复制不到/usr/lib/php/extensions/no-debug-non-zts-20121212/ 目录。  

单独执行  
sudo cp yaf.so /usr/lib/php/extensions/no-debug-non-zts-20121212/
也会提示 Operation not permitted
我试过 sudo -s ，也是不行  


查了一下，原因为： OS X El Capitan中，在内核下引入了Rootless机制，以下路径：

/System
/bin
/sbin
/usr (except /usr/local)

均属于Rootless范围，即使root用户无法对此目录有写和执行权限，只有Apple以及Apple授权签名的软件（包括命令行工具）可以修改此目录。

要么思考你这个操作的意义之后，使用其他方式完成你的操作
要么关闭Rootless（非开发者一般不推荐，或者建议执行后再次开启）


```
关闭Rootless权限的方法  
1. 开机按住Command + R键，让电脑进入恢复模式  
2. 打开终端，在终端中键入：csrutil disable 并回车  
3. 重新启动电脑进入普通模式即可。  
```

亲测有效。
