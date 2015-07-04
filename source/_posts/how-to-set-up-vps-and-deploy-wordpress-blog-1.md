title: 如何配置VPS并发布WordPress Blog?(第一篇)
tags:
  - Linux
  - VPS
id: 188
categories:
  - Linux
date: 2013-11-11 20:02:33
---

最近将VPS放到digitalocean上, 因为之前的VPS被关停了,所以再找一个, 相信这次会稳定了,之前的服务器是CentOS5.9,而现在的版本是CentOS7,以下的一些配置可能不正确了,可以参考这个链接[LAMP with Centos 7.0](http://www.howtoforge.com/apache_php_mysql_on_centos_7_lamp "lamp with centos7.0")

### 目标

发布个人的WordPress博客网站，配置LAMP环境，并用GitHub来发布Blog，不用FTP，会很慢。
[more...]

### 连接VPS

我在Mac下，所以很方便的连上VPS, 为了方便管理直接用root登陆, 第一次配置安全性先不考虑，用root方便些
[bash]
ssh -l root ip.address
[/bash]
如果在Windows下，下载一个putty就可以很方便的连上VPS，SSH端口是22，VPS默认可以用SSH连接。

### 有用的命令

[bash]
cat /etc/*release*    #查看Linux的版本
uname -a              # 查看内核的版本
ifconfig -a           # 查看网卡情况
nman localhost        # 查看端口的情况
iptables -vL          # 查看防火墙的情况
netstat -aln | grep 80  # 查看网络连接情况
init 6                  # 重启
passwd                  # 修改当前用户的密码，可以该root密码
[/bash]

### Linux running level

Linux下的运行级别，一个常识

0 – halt

1 – Single user mode 

2 – Multiuser, without NFS 

3 – Full multiuser mode 

4 – unused 

5 – X11 

6 – reboot 

[bash]
init running-level  # 触发运行级别，关机，重启各种方便
[/bash]

### 关闭selinux

selinux经常捣乱，先把它关了，还要永久关
[bash]
getenforce                  # 查看selinux的状态
sestatus                    # 一样，查看selinux状态
vim /etc/sysconfig/selinux  # 将SELINUX 改为disabled状态，然后重启
setenforce 0                # 只能暂时改变selinux状态，不能永久更改，只有上面的方法可以
init 6                      # 重启
[/bash]

这个在DigitalOcean的镜像中不需要配置.

### 安装Apache

[bash]
yum -y install httpd 
chkconfig --levels 235 httpd on  # 将httpd进程设置为开机启动
service httpd start              # 此时可以用ip访问VPS，如http://129.29.10.10，如果可以访问便安装成功
[/bash]
Apache在CentOS的DocumentRoot是“/var/www/html”, 配置文件在"/etc/httpd/config/httpd.conf"

无奈，第一个问题遇到了，我的Apache访问不了，纠结半天，原来是防火墙iptables的问题。

### 打开防火墙

CentOS的iptables默认只让注册过的端口通过，而80端口没有注册过，不能通过防火墙。
[bash]
vim /etc/sysconfig/iptables
# 将 “-A RH-Firewall-1-INPUT -j REJECT --reject-with icmp-host-prohibited” 这行注释了，保存退出
service iptables restart
[/bash]
BTW,如果采用以下的方法enable 80 端口，我今天成功了, 以前没有成功不知为何，不过上面的方法是肯定好的，下面的方法是一个安全点的办法。
[bash]
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT  // 添加443 SSL端口未来会用
service iptables save                          // /etc/sysconfig/iptables 文件会被重写
service iptables restart
[/bash]
此时再访问http://youripaddress就会有apache界面了。

在DigitalOcean中,不需要配置本项.

### 安装MySQL

[bash]
yum install -y mysql mysql-server
chkconfig --levels 235 mysqld on
service mysqld start
mysql_secure_installation         // 设置root密码，我这次谨慎了一下，没有全部选yes，大家看情况，记住自己的root密码
mysql -uroot -p                  // 进入MySQL，看看玩玩 
[/bash]

在DigitalOcean中安装的是MariaDB, 兼容MySQL, 具体看参考链接.[LAMP with Centos 7.0](http://www.howtoforge.com/apache_php_mysql_on_centos_7_lamp "lamp with centos7.0")

### 安装PHP

我被坑过，CentOS 5.9的yum的默认包库中PHP是5.1.6，后来WordPress跑不了(至少5.2.4)，所以需要更新一下yum。

如果想了解更详细的信息，可以看看这个链接
[Install Apache, MySQL 5.5.32 &amp; PHP 5.5.0 on RHEL/CentOS 6.4/5.9 &amp; Fedora 19-12](http://www.tecmint.com/install-apache-mysql-php-on-redhat-centos-fedora/ "Install Apache, MySQL 5.5.32 & PHP 5.5.0 on RHEL/CentOS 6.4/5.9 & Fedora 19-12")
[bash]
yum info name of module  # 查看某个包的信息
rpm -Uvh http://dl.fedoraproject.org/pub/epel/5/i386/epel-release-5-4.noarch.rpm
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-5.rpm
yum --enablerepo=remi,remi-test install -y php php-common
yum --enablerepo=remi,remi-test install php-mysql php-pgsql php-pecl-mongo php-sqlite php-pecl-memcache php-pecl-memcached php-gd php-mbstring php-mcrypt php-xml php-pecl-apc php-cli php-pear php-pdo  # 安装PHP会用到的module
service httpd restart
vim /var/www/html/phpinfo.php
[/bash]
输入：
[php]
&lt;?php
  phpinfo();
?&gt;
[/php]
再访问http://ipaddress/phpinfo.php， 查看php的状态

### VPS DNS解析

有了VPS，再买一个域名吧，我在godaddy上买了一个域名，用DNSPod来解析，国内的DNS解析，会快一些。

需要在godaddy上将NameServer更改为:

[text]
F1G1NS1.DNSPOD.NET
F1G1NS2.DNSPOD.NET
[/text]
再到DNSpod上注册，填写IP，等信息，就能简单完成DNS解析，从此以后就可以用域名访问VPS了。

### 本章结束

OKay, that's it! LAMP环境安装好了，一切就绪，坐等发布WordPress了，请看[如何配置VPS并发布WordPress Blog?(第二篇)](http://cyanny/myblog/2013/11/11/how-to-set-up-vps-and-deploy-wordpress-blog-2/ "如何配置VPS并发布WordPress Blog?(第二篇)")