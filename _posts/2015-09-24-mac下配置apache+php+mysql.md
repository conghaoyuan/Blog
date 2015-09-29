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

```
Mac 用户需要注意若您的系统版本低于 Mac OS X，StuffIt 会把文件解压为 Mac 格式。因为 PHP 似乎不支持 Mac 式的换行符（“\r”），所以在将文件上传到服务器之前您必须用 BBEdit 将所有 phpMyAdmin 脚本重新保存为 Unix 格式。

```
1. 从 phpmyadmin.net 下载页选择一个合适的版本。有些版本只有英语，有些包含了所有语言。假设您选择了一个名字类似于 phpMyAdmin-x.x.x -all-languages.tar.gz 的版本。
2. Ensure you have downloaded a genuine archive, see Verifying phpMyAdmin releases.
3. 解开这个压缩包（包括子目录）：在您网站服务器的文档根目录中执行 tar -xzvf phpMyAdmin_x.x.x-all-languages.tar.gz。如果您不能直接访问服务器，请先把这些文件解压到您自己的电脑上，等完成第 4 步之后，再通过 ftp 等方式将文件上传到您的网站服务器。
4. 确保所有的脚本都有正确的所有者（若 PHP 运行于安全模式，脚本间所有者的不同将会导致问题）。参见 4.2 What’s the preferred way of making phpMyAdmin secure against evil access? 和 1.26 I just installed phpMyAdmin in my document root of IIS but I get the error “No input file specified” when trying to run phpMyAdmin.。
5. 现在开始设置您的安装。两种方法。以前，用户只能手动编辑一份 config.inc.php 文件，但现在我们为那些喜欢使用图形界面安装的用户提供了一个向导式的安装脚本。手动创建 config.inc.php 仍然是一个快速安装的方法且一些高级功能也需要手动编辑该文件。
6. 配置config.inc.php文件


```
<?
php
$cfg['blowfish_secret'] = 'ba17c1ec07d65003';  // use here a value of your choice

$i=0;
$i++;
$cfg['Servers'][$i]['auth_type']     = 'cookie';
?>

或者，若您不想每次都登录：

<?php

$i=0;
$i++;
$cfg['Servers'][$i]['user']          = 'root';
$cfg['Servers'][$i]['password']      = 'cbb74bc'; // use here your password
$cfg['Servers'][$i]['auth_type']     = 'config';
?>
```
注意：把mysql服务打开

其实具体详细的配置就看人家phpmyadmin的文档吧 ，比我们的都详细 [连接](https://phpmyadmin-chinese-china.readthedocs.org/en/latest/setup.html)

另外给大家推荐个好用的桌面版工具，如windows下的navigate 。[Sequel Pro](http://www.sequelpro.com/)