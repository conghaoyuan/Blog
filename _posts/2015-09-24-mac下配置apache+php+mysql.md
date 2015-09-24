---
layout: post
title: mac下配置apache+php+mysql
categories:
- PHP
---

<div class="message">
Mac OX 自带apache和php，在terminal下输入httpd -v 查看apache版本，本人自带apache为2.4版本。在配置过程中可能会出现很多错误，在网上搜了一些相关的资料也不是很全。特别是配置虚拟主机的时候，老是不成功，现在已经基本解决90%以上问题。
</div>


## 1 配置apache

apache已经自带，只需要开启apache即可：
	
	sudo apachectl start   //启动apache
	sudo apachectl stop    //停止apache
	sudo apachectl restart  //重启apache
	httpd -v   //查看apache版本
	
开启apache后在浏览器地址栏中输入localhost则出现It works，如图：

<image width="600" src="/images/15924/apache.png">

程序的根目录在/Library/WebServer/Documents/ 下的index.html.en, 现在配置虚拟主机（这里的问题最多，并没有网上写的那么简单）。



先一步一步来吧，首先找到httpd.conf 配置文件。

	cd /etc/apache2/
	sudo vim httpd.conf
	
查询vhost，大约在500行左右，将Include前边的#删除。

	# Virtual hosts
	# Include /private/etc/apache2/extra/httpd-vhosts.conf
	
Include /private/etc/apache2/extra/httpd-vhosts.conf

然后返回可以保存退出了。:wq

进入extra/目录。打开httpd-vhosts.conf

	cd extra/ && sudo vim httpd-vhosts.conf
	
	# Virtual Hosts
	#
	# Required modules: mod_log_config

	# If you want to maintain multiple domains/hostnames on your
	# machine you can setup VirtualHost containers for them. Most configurations
	# use only name-based virtual hosts so the server doesn't need to worry about
	# IP addresses. This is indicated by the asterisks in the directives below.
	#
	# Please see the documentation at 
	# <URL:http://httpd.apache.org/docs/2.4/vhosts/>
	# for further details before you try to setup virtual hosts.
	#
	# You may use the command line option '-S' to verify your virtual host
	# configuration.

	#
	# VirtualHost example:
	# Almost any Apache directive may go into a VirtualHost container.
	# The first VirtualHost section is used for all requests that do not
	# match a ServerName or ServerAlias in any <VirtualHost> block.
	#
	#<VirtualHost *:80>
	#    ServerAdmin webmaster@dummy-host.example.com
	#    DocumentRoot "/usr/docs/dummy-host.example.com"
	#    ServerName dummy-host.example.com
	#    ServerAlias www.dummy-host.example.com
	#    ErrorLog "/private/var/log/apache2/dummy-host.example.com-error_log"
	#    CustomLog "/private/var/log/apache2/dummy-host.example.com-access_log" common
	#</VirtualHost>

	#<VirtualHost *:80>
	#    ServerAdmin webmaster@dummy-host2.example.com
	#    DocumentRoot "/usr/	docs/dummy-host2.example.com"
	#    ServerName dummy-host2.example.com
	#    ErrorLog "/private/var/log/apache2/dummy-host2.example.com-error_log"
	#    CustomLog "/private/var/log/apache2/dummy-host2.example.com-access_log" common
	#</VirtualHost>
	
然后就配置呗，人家也都给提示了。
自己创建一个目录，位置自己定。我在自己的家目录下创建的。如下：
<image width="600" src="/images/15924/apache3.png">

然后可以创建一个index.html文件，里边输入一句hello world
接下来在hosts里边加入想要配置的虚拟主机名

	sudo vim /etc/hosts
	127.0.0.1	site.com
	
在httpd-vhost.conf下边加上：

	#www
	<VirtualHost *:80>
    ServerAdmin you@example.com
    DocumentRoot "/Users/yuanconghao/www"
    #site.com 为hosts里边配置的主机名
    ServerName site.com    
    DirectoryIndex index.html index.php
    <Directory "/Users/yuanconghao/www">
        Options Indexes
        AllowOverride All
        Order Allow,Deny
        Allow from All
    </Directory>
    ErrorLog "/private/var/log/apache2/dummy-host2.example.com-error_log"
    CustomLog "/private/var/log/apache2/dummy-host2.example.com-access_log" common
	</VirtualHost>

到这里就算是基本配置完了，网上很多博客也都是这样写的，然后重启apache在浏览器中输入site.com 查看效果吧。是不是会显示该网页无法显示，这个问题很奇怪。按理说不应该出现这种情况。纠结了好久。最后将

	Include /private/etc/apache2/extra/httpd-vhosts.conf
	中的/private去掉
	Include /etc/apache2/extra/httpd-vhosts.conf

现在重启再看，会发现
<image width="600" src="/images/15924/apache4.png" />

这时只需要将httpd.conf里边的Require all denied改为Require all granted

	<Directory />
    	AllowOverride none
    	#Require all denied
    	Require all granted
	</Directory>
	
重启apache出现hello world 即配置成功。

注：有个问题，当写这篇博客的时候我的虚拟主机已经配置成功，然后又重新设置的。当httpd.conf里边的/private加上后访问竟然
能成功。

		Include /private/etc/apache2/extra/httpd-vhosts.conf
		
--------------------------
## 2 配置php

PHP的配置非常简单，就一个事，进到/etc/apache2/目录，编辑httpd.conf，找到LoadModule php5_module libexec/apache2/libphp5.so将其放开注释就行了。

然后sudo apachectl restart 重启，在用户目录的Sites文件夹下，新
建一个index.php,里面echo phpinfo() ，就可以看到效果了： 

<image width="600" src="/images/15924/php.png">

--------------------------
## 3 安装mysql 5.6

这个版本是最新的MySQL，安装方法跟5.5的略有不同。在官网下载mysql-5.6.26-osx10.9-x86_64.dmg，下面是安装方法： 
1，双击安装的时候，不要勾选StartUp Item这一项： 
<image width="600" src="/images/15924/mysql1.png">

安装完毕后，在偏好设置下边看到MySQL，手动开启MySQL服务。 

<image width="600" src="/images/15924/mysql1.png">

我这里已经打开了MySQL服务。下面要将其设置为开机自动启动。

2，默认状态下，我们用mysql的命令每次都要输入全路径，如sudo /usr/local/mysql/support-files/mysql.server start 开启mysql服务，/usr/local/mysql/bin/mysql -v查看mysql版本，得先把bin目录配到环境变量里。切换到用户根目录 ，vim .bash_profile,输入： 
export PATH=”/usr/local/mysql/bin:$PATH” 
保存后，source .bash_profile使环境变量生效。接着就可以直接在终端里输入mysql命令了。 
最后，通过mysqladmin -u root password ‘yourpasswordhere’ 给mysql的root用户设置密码。单引号里的内容就是要设的密码。

备注：有时上面这个命令不能修改root密码，需要借助phpmyadmin来修改。其实mysql这个版本默认的root密码为root。

3，修复socket error的问题。有一个负责mysql 服务器 客户端通讯的socket文件，mysql的这个版本将其放在/tmp目录，但是OSX却默认的找 /var/mysql 这个目录，所以要建个软链接。新建目录 /var/mysql， 然后sudo ln -s /tmp/mysql.sock /var/mysql/mysql.sock 就ok了。

4，让mysql开机自动启动。 
sudo vim /Library/LaunchDaemons/com.mysql.mysql.plist,里面内容输入： 

KeepAlive 

Label 
com.mysql.mysqld 
ProgramArguments 

/usr/local/mysql/bin/mysqld_safe 
–user=mysql 

保存后，修改权限： 
sudo chown root:wheel /Library/LaunchDaemons/com.mysql.mysql.plist 
sudo chmod 644 /Library/LaunchDaemons/com.mysql.mysql.plist 
sudo launchctl load -w /Library/LaunchDaemons/com.mysql.mysql.plist 

这样mysql就ok了。。。。

-----------------------
## 4 phpMyAdmin的安装
