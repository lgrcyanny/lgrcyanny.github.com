title: Set up Hadoop 2.2 and HBase 0.96 part2
tags:
  - Hadoop
  - HBase
id: 588
categories:
  - Hadoop
  - HBase
date: 2014-02-06 14:09:21
---

在完成[Hadoop配置](http://cyanny/myblog/2014/02/06/set-hadoop-hbase-part1/ "set-hadoop-hbase-part1")后，我们可以开始HBase的安装和配置了。
对于HBase，我只想说走对了路就成功了一半，选对了版本就省事好多。之前下载的是0.94版，按照官方的配置，连Standalone都跑不通，纠结了半天，放弃治疗，选用0.96版本，一切顺利。

### 配置HBase Standalone模式

1\. 前提条件
MacOS 10.9
Java安装好
Standalone模式是单机的，基于Local FileSystem，不需要Hadoop，用于开发或测试。
[more...]
2\. 下载HBase
[hbase-0.96.1-hadoop2-bin.tar.gz ](http://apache.fayea.com/apache-mirror/hbase/hbase-0.96.1/)

3\. 解压安装
[shell]
$ tar xzvf hbase-0.96.1-hadoop2-bin.tar.gz  
$ mv hbase-0.96.1-hadoop2 hbase
[/shell]
同样需要把hbase安装到Home目录下~/hbase, 对于我是/Users/lgrcyanny/hbase

4\. 配置环境变量
[shell]
$ vim ~/.bashrc
#Config HBase
export HBASE_HOME=/Users/lgrcyanny/hbase
export PATH=$PATH:$HBASE_HOME/bin
[/shell]

5\. 编辑hbase-site.xml
[shell]
$ cd ~/hbase
$ mkdir -p mydata/hbase
$ mkdir -p mydata/zookeeper
$ vim conf/hbase-site.xml
&lt;configuration&gt;
  &lt;property&gt;
    &lt;name&gt;hbase.rootdir&lt;/name&gt;
    &lt;value&gt;file:///Users/lgrcyanny/hbase/mydata/hbase&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hbase.zookeeper.property.dataDir&lt;/name&gt;
    &lt;value&gt;/Users/lgrcyanny/hbase/mydata/zookeeper&lt;/value&gt;
  &lt;/property&gt;
&lt;/configuration&gt;
[/shell]

6\. 编辑hbase-env.sh
[shell]
$ cd ~/hbase
$ vim conf/hbase-env.sh
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_17.jdk/Contents/Home/
export HBASE_OPTS=&quot;-Djava.security.krb5.realm= -Djava.security.krb5.kdc= -Djava.security.krb5.conf=/dev/null&quot;
[/shell]

7\. 关于/etc/hosts
对于Ubuntu的用户，需要修改/etc/hosts，将127.0.1.1改为127.0.0.1，因为HBase的环回地址默认是127.0.0.1，但官方的quikstart中提到0.96以后的版本不需要修改，MacOS中确实不用修改，之前用0.94的版本，因为环回地址的问题总是报错也无法修复，这是0.94的bug。如果是Ubuntu用户，可以尝试不修改hosts看能不能跑通。

8\. 启动并测试HBase
[shell]
$ start-hbase.sh
starting master, logging to /Users/lgrcyanny/hbase/logs/hbase-lgrcyanny-master-Cyanny-MacBook-Air.local.out
$ hbase shell
&gt; status
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/Users/lgrcyanny/hbase/lib/slf4j-log4j12-1.6.4.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/Users/lgrcyanny/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
2014-02-06 14:27:29,472 WARN  [main] util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
1 servers, 0 dead, 3.0000 average load
&gt; create 'test', 'cf'
&gt; put 'test', 'row1', 'cf:a', 'value1'
&gt; list 
TABLE                                                                                                                                                         
test                                                                                                                                                          
1 row(s) in 0.1060 seconds

=&gt; [&quot;test&quot;]
&gt; scan 'test'
ROW                                      COLUMN+CELL                                                                                                          
 row1                                    column=cf:a, timestamp=1391653468438, value=value1                                                                   
1 row(s) in 0.0650 seconds
&gt; exit
$ stop-hbase.sh
stopping hbase.................
[/shell]

如果上面的步骤完成，HBase的Standalone模式就安装成功。

### 配置HBase单机伪分布式

1\. 修改hbase-site.xml
[shell]
$ cd ~/hbase
$ vim conf/hbase-site.xml
&lt;configuration&gt;
  &lt;property&gt;
    &lt;name&gt;hbase.rootdir&lt;/name&gt;
    &lt;value&gt;hdfs://localhost:9000/hbase&lt;/value&gt;
  &lt;/property&gt;

  &lt;property&gt;
    &lt;name&gt;hbase.cluster.distributed&lt;/name&gt;
    &lt;value&gt;true&lt;/value&gt;
  &lt;/property&gt;

  &lt;property&gt;
    &lt;name&gt;hbase.zookeeper.quorum&lt;/name&gt;
    &lt;value&gt;localhost&lt;/value&gt;
  &lt;/property&gt;

  &lt;property&gt;
    &lt;name&gt;dfs.replication&lt;/name&gt;
    &lt;value&gt;1&lt;/value&gt;
  &lt;/property&gt;

  &lt;property&gt;
    &lt;name&gt;hbase.zookeeper.property.clientPort&lt;/name&gt;
    &lt;value&gt;2181&lt;/value&gt;
  &lt;/property&gt;

  &lt;property&gt;
    &lt;name&gt;hbase.zookeeper.property.dataDir&lt;/name&gt;
    &lt;value&gt;/Users/lgrcyanny/hbase/mydata/zookeeper&lt;/value&gt;
  &lt;/property&gt;
&lt;/configuration&gt;
[/shell]

2\. 启动Hadoop和HBase
[shell]
$ start-dfs.sh
$ start-hbase.sh
localhost: starting zookeeper, logging to /Users/lgrcyanny/hbase/logs/hbase-lgrcyanny-zookeeper-Cyanny-MacBook-Air.local.out
starting master, logging to /Users/lgrcyanny/hbase/logs/hbase-lgrcyanny-master-Cyanny-MacBook-Air.local.out
localhost: starting regionserver, logging to /Users/lgrcyanny/hbase/logs/hbase-lgrcyanny-regionserver-Cyanny-MacBook-Air.local.out
$ jps
11107 HQuorumPeer
11427 Jps
11180 HMaster
11276 HRegionServer
9155 SecondaryNameNode
9064 DataNode
8990 NameNode
[/shell]
不需要启动yarn, 启动HDFS即可，我们可以看到HBase的伪分布式模式中，启动了内嵌的Zookeeper，启动了Master和RegionServer。

3\. 查看状态
查看Master Status： http://localhost:60010/master-status
查看Region Server Status： http://localhost:60030/rs-status

4\. 测试HBase
可以按照Standalone模式中的第8步，打开hbase shell进行测试。

到这里，如果顺利那么就一切都配置好了，可能花费了你1~2个小时，还是要恭喜，我前后花费了2天。
Anyway，探索的过程还是很愉快，Good Luck！

#### 参考

[HBase Quick Start](http://hbase.apache.org/book/quickstart.html)