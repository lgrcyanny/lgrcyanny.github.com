title: Set up Hadoop 2.2 and HBase 0.96 part1
tags:
  - Hadoop
  - HBase
  - Learning
id: 574
categories:
  - Hadoop
  - HBase
date: 2014-02-06 11:27:54
---

过完春节，新年开始了，闲暇的时光弄了一下Hadoop和HBase，之前也配置过Hadoop，不过是1.x的版本和现在2.2的版本不一样了，HBase的官网推荐使用Hadoop2.x，配置HBase确实花费了点时间，网上的各种教程相似但各异，自己遇到的问题和方法也值得记下来，下次再配置时也可以查看一下。
[Hadoop官方Guide](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html)的配置不够详细，需要参考各种博客，以下是我个人配置的方法：

### 配置Hadoop

1\. 操作系统
我用的是MacOS 10.9， 尽量在配置时使用Linux系统如Ubuntu，用Windows需要下载安装Cygwin，可能会麻烦些。
[more...]
2\. 下载Java
到Oracle下载J2SE，1.6版本以上，下载最新版1.7即可。
在Mac下安装后，Java的Home路径是：
"/Library/Java/JavaVirtualMachines/jdk1.7.0_17.jdk/Contents/Home/"
而不是很多博客中用的：
"/System/Library/Frameworks/JavaVM.framework/Versions/CurrentJDK"
应该是MacOS10.8后有更新了。

3\. 下载Hadoop 2.X
我用的版本是Hadoop2.2.0
[点击下载 hadoop-2.2.0.tar.gz](http://apache.fayea.com/apache-mirror/hadoop/common/stable2/ "Download Hadoop 2.2.0")

[shell]
$ tar xzvf  hadoop-2.2.0.tar.gz
$ mv hadoop-2.2.0 hadoop
[/shell]
我安装hadoop到路径/Users/lgrcyanny/hadoop，即当前Mac用户的Home目录下，为了方便没有创建新的用户hadoop，在配置时可以自己将“lgrcyanny”替换为自己的用户名。

4\. 配置ssh localhost
Hadoop的Master和Slave的通信采用ssh，单机版的Hadoop也需要ssh。
Ubuntu用户可以"apt-get install ssh"， mac用户自带ssh
[shell]
$ ssh-keygen -t rsa # 如果之前配置过GitHub，rsa key是存在的，请不要覆盖即可
$ ssh lgrcyanny@localhost mkdir -p .ssh
$ cat .ssh/id_rsa.pub | ssh lgrcyanny@localhost 'cat &gt;&gt; .ssh/authorized_keys'
$ ssh lgrcyanny@localhost &quot;chmod 700 .ssh; chmod 640 .ssh/authorized_keys&quot;
$ ssh localhost
Last login: Thu Feb  6 10:22:40 2014
[/shell]

5\. 配置环境变量
[shell]
$ cd ~
$ vim .bashrc
#Java properties
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_17.jdk/Contents/Home/
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=.:$JAVA_HOME/bin:$JRE_HOME/bin:$PATH

#Hadoop variables
export HADOOP_INSTALL=/Users/lgrcyanny/hadoop  # Please change to your installation path
export PATH=$PATH:$HADOOP_INSTALL/bin
export PATH=$PATH:$HADOOP_INSTALL/sbin
export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_HOME=$HADOOP_INSTALL
export HADOOP_HDFS_HOME=$HADOOP_INSTALL
export HADOOP_YARN_HOME=$HADOOP_INSTALL
$ source .bashrc
[/shell]

6\. 编辑hadoop-env.sh
[shell]
$ cd ~/hadoop/etc/hadoop
$vim hadoop-env.sh
# config JAVA_HOME and HADOOP_OPTS
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_17.jdk/Contents/Home/

# HADOOP_OPTS is to get rid of warning message &quot;Unable to load realm info from SCDynamicStore &quot;
export HADOOP_OPTS=&quot;-Djava.security.krb5.realm= -Djava.security.krb5.kdc= -Djava.security.krb5.conf=/dev/null&quot;
[/shell]

7\. 查看hadoop version
[shell]
$ hadoop version
Hadoop 2.2.0
Subversion https://svn.apache.org/repos/asf/hadoop/common -r 1529768
Compiled by hortonmu on 2013-10-07T06:28Z
Compiled with protoc 2.5.0
From source with checksum 79e53ce7994d1628b240f09af91e1af4
This command was run using /Users/lgrcyanny/hadoop/share/hadoop/common/hadoop-common-2.2.0.jar
[/shell]

8\. 编辑core-site.xml
[shell]
$ cd ~/hadoop
$ mkdir -p mydata/tmp
$ vim etc/hadoop/core-site.xml
# config as following
&lt;configuration&gt;
  &lt;property&gt;
     &lt;name&gt;fs.default.name&lt;/name&gt;
     &lt;value&gt;hdfs://localhost:9000&lt;/value&gt;
  &lt;/property&gt;

  &lt;property&gt;
    &lt;name&gt;hadoop.tmp.dir&lt;/name&gt;
    &lt;value&gt;/Users/lgrcyanny/hadoop/mydata/tmp&lt;/value&gt;
  &lt;/property&gt;
&lt;/configuration&gt;
[/shell]

9\. 编辑yarn-site.xml
Hadoop 2.x的特点就是引入了YARN框架，这是和之前Hadoop 1.x不同的方面
[shell]
$ cd ~/hadoop
$ vim etc/hadoop/yarn-site.xml
&lt;configuration&gt;
  &lt;property&gt;
     &lt;name&gt;yarn.nodemanager.aux-services&lt;/name&gt;
     &lt;value&gt;mapreduce_shuffle&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
     &lt;name&gt;yarn.nodemanager.aux-services.mapreduce.shuffle.class&lt;/name&gt;
     &lt;value&gt;org.apache.hadoop.mapred.ShuffleHandler&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;yarn.nodemanager.resource.memory-mb&lt;/name&gt;
    &lt;value&gt;10240&lt;/value&gt;
    &lt;description&gt;the amount of memory on the NodeManager in GB&lt;/description&gt;
  &lt;/property&gt;
&lt;/configuration&gt;
[/shell]

10\. 编辑mapred-site.xml
[shell]
$ cd ~/hadoop
$ mkdir -p mydata/mapred/temp
$ mkdir -p mydata/mapred/local
$ mv etc/hadoop/mapred-site.xml.template etc/hadoop/mapred-site.xml
$ vim etc/hadoop/mapred-site.xml
&lt;configuration&gt;

  &lt;property&gt;
     &lt;name&gt;mapreduce.framework.name&lt;/name&gt;
     &lt;value&gt;yarn&lt;/value&gt;
  &lt;/property&gt;

  &lt;property&gt;
    &lt;name&gt;mapreduce.cluster.temp.dir&lt;/name&gt;
    &lt;value&gt;/Users/lgrcyanny/hadoop/mydata/mapred/temp&lt;/value&gt;
    &lt;description&gt;The temp dir for map reduce&lt;/description&gt;
    &lt;final&gt;true&lt;/final&gt;
  &lt;/property&gt;

  &lt;property&gt;
    &lt;name&gt;mapreduce.cluster.local.dir&lt;/name&gt;
    &lt;value&gt;/Users/lgrcyanny/hadoop/mydata/mapred/local&lt;/value&gt;
    &lt;description&gt;The local dir for map reduce&lt;/description&gt;
    &lt;final&gt;true&lt;/final&gt;
  &lt;/property&gt;

&lt;/configuration&gt;
[/shell]

11\. 编辑hdfs-site.xml
[shell]
$ cd ~/hadoop
$ mkdir -p mydata/hdfs/namenode
$ mkdir -p mydata/hdfs/datanode
$ vim etc/hadoop/hdfs-site.xml
&lt;configuration&gt;
  &lt;property&gt;
     &lt;name&gt;dfs.replication&lt;/name&gt;
     &lt;value&gt;1&lt;/value&gt;
   &lt;/property&gt;
   &lt;property&gt;
     &lt;name&gt;dfs.namenode.name.dir&lt;/name&gt;
     &lt;value&gt;file:/Users/lgrcyanny/hadoop/mydata/hdfs/namenode&lt;/value&gt;
   &lt;/property&gt;
   &lt;property&gt;
     &lt;name&gt;dfs.datanode.data.dir&lt;/name&gt;
     &lt;value&gt;file:/Users/lgrcyanny/hadoop/mydata/hdfs/datanode&lt;/value&gt;
   &lt;/property&gt;
&lt;/configuration&gt;
[/shell]

12\. Format HDFS
[shell]
$ hadoop namenode -format
......
14/02/06 13:22:41 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at Cyanny-MacBook-Air.local/192.168.1.103
************************************************************/
[/shell]

13\. 启动Hadoop
[shell]
$ start-dfs.sh
$ start-yarn.sh
$ jps #  Java Virtual Machine Process Status Tool
10377 Jps
10224 NodeManager
10146 ResourceManager
9155 SecondaryNameNode
9064 DataNode
8990 NameNode
[/shell]
Hadoop 1.x采用start-all.sh启动，而Hadoop2.x拆分为start-dfs.sh, start-yarn.sh，我想是因为如果使用HBase时，只需要HDFS的服务，而一般不需要YARN的服务，这样就不会占用太多的内存。

14\. Web查看Hadoop
查看HDFS的状态：http://localhost:50070
查看YARN的状态: http://localhost:8088

15\. 测试Hadoop，使用Hadoop的WordCount example
[shell]
$ cd ~/hadoop
$ hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.2.0.jar pi 2 5
Number of Maps  = 2
Samples per Map = 5
14/02/06 13:28:39 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Wrote input for Map #0
Wrote input for Map #1
Starting Job
14/02/06 13:28:40 INFO client.RMProxy: Connecting to ResourceManager at /0.0.0.0:8032
14/02/06 13:28:41 INFO input.FileInputFormat: Total input paths to process : 2
14/02/06 13:28:41 INFO mapreduce.JobSubmitter: number of splits:2
.........
[/shell]
你可以通过http://localhost:8088/cluster 查看MapReduce的运行状态

Congratulations, Hadoop 2.2.0安装完毕，接下来我们就要开始安装HBase了。

### Trouble Shooting

1\. 当启动Hadoop，jps查看后没有datanode怎么办？
某一天我遇到这个问题，方法是将datanode和namenode, 以及tmp文件夹删除，重新format，再重启Hadoop。
[shell]
$ cd ~/hadoop
$ stop-dfs.sh
$ stop-yarn.sh
$ rm -r mydata/hdfs/datanode
$ rm -r mydata/hdfs/namenode
$ rm -r mydata/tmp
$ mkdir -p mydata/tmp
$ hadoop namenode -format
$ start-dfs.sh
$ start-yarn.sh
[/shell]

#### 参考资料

[[1] setting-hadoop-2.2.0-on-ubuntu-12-lts](http://javatute.com/javatute/faces/post/hadoop/2014/setting-hadoop-2.2.0-on-ubuntu-12-lts.xhtml)