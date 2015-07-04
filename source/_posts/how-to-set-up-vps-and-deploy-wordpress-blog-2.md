title: 如何配置VPS并发布WordPress Blog?(第二篇)
tags:
  - Linux
  - VPS
id: 210
categories:
  - Linux
date: 2013-11-11 20:00:14
---

完成了VPS的LAMP环境的配置，接下来的工作就是要将WordPress Blog部署到VPS上。方法多种多样，例如可以在VPS上配置FTP，用FTP将文件传输到VPS上，以后就可以直接在VPS上写博客了，一次性的买卖，方便，听起来很不错，但是不安全，如果哪天VPS Down了数据就没了。

我采取的办法是在本地写博客，然后发布到GitHub,然后VPS端Clone到本地，这样做的原因如下：

[more...]

*   WordPress Blog作为轻量的博客，主要发布文字，图片是用第三方的图床，博客本身的数据量不大。
*   FTP传输速度不理想。
*   GitHub在VPS端运行很快，VPS毕竟是服务器，网速快，因此部署Blog很快。
*   版本控制，本地有数据备份，以后要迁移VPS也可以轻松搞定。
*   WordPress的Blog数据是写入MySQL的，因此发布WordPress时的一个大问题就是要处理MySQL的数据，其实基本的mysqldump就可以了。

### VPS 配置GIT

[bash]
rpm -Uvh http://dl.fedoraproject.org/pub/epel/5/i386/epel-release-5-4.noarch.rpm // 这个在前面的php安装时输入过了，可以忽略
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-5.rpm // 这个在前面的php安装时输入过了，可以忽略
yum install git-core -y
git --version
[/bash]
一定配置一下SSH，这样不会在git clone时报错，当然建立github repository和使用github都是默认掌握了。
[Set up git](https://help.github.com/articles/set-up-git#platform-linux "Set up Git for linux") 

[Enable SSH Git](https://help.github.com/articles/generating-ssh-keys "Generate ssh key")</p>

### VPS MySQL权限配置

刚装MySQL时运行过mysql_secure_installation，如果所有选项都yes了，那么VPS的mysql只能是本地的root可以访问，blog访问不了，这一步很关键，也让我头疼了很久。

进入MySQL
[bash]
mysql -u root -p
[/bash]
[sql]
CREATE USER 'monty'@'localhost' IDENTIFIED BY 'some_password';
GRANT ALL PRIVILEGES ON *.* TO 'monty'@'localhost';
CREATE USER 'monty'@'%' IDENTIFIED BY 'some_password';
GRANT ALL PRIVILEGES ON *.* TO 'monty'@'%;
[/sql]
**watch out**

我在这一步遇到了问题：当运行
[sql]
GRANTALL PRIVILEGES ON *.* TO 'monty'@'localhost';
[/sql]
报错：Access denied for user 'root'@'localhost'。后来在stackoverflow上找到了答案：[access-denied-for-user-rootlocalhost-while-attempting-to-grant-privileges](http://stackoverflow.com/questions/8484722/access-denied-for-user-rootlocalhost-while-attempting-to-grant-privileges)

我采用了二楼的方法如下就可以了：
[bash]
$ su - mysql
$ rm -rf /var/lib/mysql/*
$ mysql_install_db
$ /etc/init.d/mysql restart
[/bash]
然后再重置root密码：
[bash]
mysqladmin -u root password
[/bash]

### VPS MySQL CharSet配置

为了防止中文乱码，将VPS的MySQL的CharSet设置为utf8，当然WordPress的mysqldump的SQL文件中的每一个表都强制指定了CharSet为utf8，如果略去这一步也是可以的。
[bash]
vim /etc/my.cnf
[/bash]
在文本中的相应位置添加
[text]
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
character-set-server = utf8
collation-server = utf8_unicode_ci
[client]
default-character-set = utf8
[mysql]
default-character-set = utf8
[/text]
验证，进入MySQL输入：
[sql]
SHOW VARIABLES LIKE 'character%';
SHOW VARIABLES LIKE 'collation%';
[/sql]

### 创建博客数据库

进入mysql，创建数据库
[bash]
mysql -u root -p
[/bash]
[sql]
mysql&gt; create database yourblogdatabase;
[/sql]

### 本地Blog GitHub Set Up

首先在GitHub网站上创建一个Repository，[GitHub of Cyanny Live Blog](https://github.com/lgrcyanny/CyannyLive "GitHub of Cyanny Live Blog")
在本地的Blog中：
[bash]
cd ~/Sites/Blog //打开文件夹
git init
git remote add origin git@github.com:lgrcyanny/CyannyLive.git  // github的ssh地址
git pull origin master
[/bash]

### 本地Blog MySQL数据导出备份

我直接写了一个脚本，备份MySQL的数据，脚本backup.sh如下:
[shell]
# !/bin/bash
export DATABASE_DIR=~/Sites/myblog/wp-content/backup-db
cd $DATABASE_DIR
mysqldump -u root -pyourpassword dbname &gt; temp-db.sql
# sed 命令替换在sql文件中Blog的http地址，这一步非常重要，否则在VPS上打不开，
# 原因很简单，本地是Localhost访问，VPS是域名访问，WordPress为每一个Post生成的访问地址是写死的
sed 's%http://cyanny/myblog%http://www.cyanny.com%g' temp-db.sql \&gt; db.sql
# 数据压缩一下，会从上百KB变成几十个KB
tar -czvf db.tar.gz db.sql
# 删除中间文件
rm db.sql
rm temp-db.sql
[/shell]
运行backup.sh
[bash]
sh backup.sh
[/bash]

### 本地Blog htaccess配置

博客的访问地址，如果设置过Permalinks，这一步就很重要，否则在VPS端地址找不到
[bash]
cp .htaccess htaccess.prod
[/bash]
编辑htaccess.prod
[text]
&lt;IfModule mod_rewrite.c&gt;
RewriteEngine On
RewriteBase /
RewriteRule ^index&amp;#46;php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
&lt;/IfModule&gt;
[/text]

### 本地Blog的其他配置

编辑.gitignore
[text]
wp-config.php
uploads/&amp;#42;
wp-cache/&amp;#42;
wp-content/cache
.htaccess
[/text]

wp-config.php配置
[bash]
cp wp-config.php wp-config-prod.php  //本地的数据库配置和VPS上的不一样，因此要在wp-config-prod.php中设置VPS的DB信息
[/bash]

### 本地Blog 发布到 GitHub

[bash]
git add .
git commit -a -m 'message'
git push origin master
[/bash]

### VPS端 Enable htaccess overwirte

因为Blog的Post地址会被overwrite，所以需要VPS端具有htaccess overwrite的功能，这个Bug让我抓狂很久。

进入VPS
[bash]
vim /etc/httpd/conf/httpd.conf
[/bash]
编辑httpd.conf
[text]
// 找到下面这一段
&lt;Directory /var/www/html&gt;
  Options Indexes FollowSymLinks MultiViews
  // 更改为 AllowOverride All
  AllowOverride None
  Order allow,deny
  allow from all
&lt;/Directory&gt;
[/text]

### VPS端的发布脚本

为了方便VPS端将WordPress从GitHub Clone到本地, 设置文件权限，DB数据导入，我创建了一个简单地发布脚本

deploy.sh, 这个脚本放在VPS上
[bash]
# !/bin/bash
export BLOG_ROOT_DIR=/var/www/html/cyannyblog
export BLOG_DATABASES_DIR=$BLOG_ROOT_DIR/wp-content/backup-db
# Git update
rm -Rf cyannyblog
git clone git@github.com:lgrcyanny/CyannyLive.git cyannyblog  # 更换为自己的GitHub的地址
# Change mod, grant access right
chmod -R 755 $BLOG_ROOT_DIR
chmod -R 777 $BLOG_ROOT_DIR/wp-content
# Add config
rm -f $BLOG_ROOT_DIR/wp-config.php
cp $BLOG_ROOT_DIR/wp-config-prod.php $BLOG_ROOT_DIR/wp-config.php
# Add .htaccess
cd $BLOG_ROOT_DIR
rm -f .htaccess
cp htaccess.prod .htaccess
# Import data to mysql 不用担心数据冲突，因为mysqldump出来的表都会先drop再导入
cd $BLOG_DATABASES_DIR
tar -xvf db.tar.gz
mysql -u yourdbuser -pyourpassword -h localhost dbname &lt; db.sql
[/bash]

在VPS上运行deploy.sh
[bash]
sh deploy.sh
[/bash]

### VPS端将DocumentRoot指向Blog文件夹

[bash]
vim /etc/httpd/conf/httpd.conf
[/bash]
编辑httpd.conf:
[text]
DocumentRoot &quot;/var/www/html/blog&quot;
[/text]
我想这不是一个好办法，不过先将就一下。如果要在VPS上放置多个网站，DocumentRoot当然不能只是Blog，应该可以有一些Virtual的方法的，之后我会改进。目前这样是因为DNS不能解析到某一个路径

### 结束

Okay, 发布流程部署结束，接下来就可以访问啦。
以后每写一遍Post，或者本地有什么更新只需三个步骤就可以完成发布：

1\. 本地 run backup.sh

2\. GitHub Commit

3\. VPS run deploy.sh, 当然未来还可以用cronjob，这样VPS端的发布问题就不用手工了，鉴于不是频繁发布Post，手工操作也无妨。

### 后记

1\. 2014-08-17
VPS版本换了Centos7, MySQL替换为MariaDB, 但是DB不稳定,今天终于挂了
[bash]
systemctl restart mariadb 
[/bash]
启动失败,最后修复如下:
[bash]
systemctl stop mariadb
rm -rf /var/lib/mysql
rm -rf /var/log/mariadb/
yum list mariadb
yum remove maridadb mariadb-server
# yum remove all list about mariadb
yum install mariadb mariadb-server
reboot
systemctl enable mariadb
mkdir -p /var/lib/mysql
chown -R mysql:mysql /var/lib/mysql
systemctl start mariadb
yum install php-mysql
systemctl restart httpd.service
[/bash]