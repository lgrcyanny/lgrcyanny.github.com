title: Nodejs HBase0.96 Hadoop2.2.0 Thrift2配置与使用
tags:
  - Hadoop
  - HBase
  - Learn
id: 598
categories:
  - Hadoop
  - HBase
date: 2014-02-27 16:00:42
---

项目如果没有采用Java开发，难道就不能用HBase了么？程序猿不会善罢甘休的，有什么语言就会有什么API存在，我还觉得用Java配置时各种缺包错误很烦呢，记得《数学之美》中曾说道：“做技术有术和道两个层面”，知道HBase的架构和一些底层细节是"道"，而使用各种配置和API开发应用则是"术"，而我们就来试试非Java连接HBase。
HBase的第三方接口有Shell, Java, REST和Thrift，可以参考《HBase in Action》chapter 6, REST接口比较慢，使用起来并没有Thrift好。而你可能疑惑什么是Thrift:
<!--more-->
_
Thrift is a “software framework, for scalable cross-language services development…with a code generation engine to build services that work efficiently and seamlessly between C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, JavaScript, Node.js…” Thrift allows us to create a thin API in NodeJS to communicate with HBase using a thin socket protocol. The advantages over REST are that the connection stays alive and the protocol is thinner than XML.
_
Apache Thrift是一个proxy，一个基于RPC的中间件，通过该接口，我们可以采用任何第三方语言连接HBase, 由此可见软件中间件封装底层detail的魅力。HBase提供C++, Java, Python, Node.js的Thrift文件。就Node.js来说，HBase提供了Thrift1和Thrift2， 官方的说法是Thrift2会逐步取代Thrift1， Thrift1我用了，发现命名不规范，而且CRUD并没有全部跑通，之后换用Thrift2，试验后成功了。
HBase Thrift API文档很少，官方只找到一个[Python的老版本API](http://wiki.apache.org/hadoop/Hbase/ThriftApi "Python HBase API"), 参考过的博客中采用Thrift2和Node.js的，Google一圈也没有，因此在这里把自己的一些小试验写下来，分享给朋友们，如有不对之处，还请指正。

### 一、环境配置

1\. 系统要求
*nix系统，我是Max OX 10.9，因此在配置上也是以Mac的配置为准。

2\. 安装Hadoop2.2.0和HBase 0.96.1，并下载HBase 0.96.1的源码到本地，并解压到本地
可以参考我之前的两个配置文章：
[Set up Hadoop 2.2 and HBase 0.96 part1](http://cyanny/myblog/2014/02/06/set-hadoop-hbase-part1/ "Set up Hadoop 2.2 and HBase 0.96 part1")
[Set up Hadoop 2.2 and HBase 0.96 part2](http://cyanny/myblog/2014/02/06/set-hadoop-hbase-part2/ "Set up Hadoop 2.2 and HBase 0.96 part2")

3\. Mac需要安装brew, Ubuntu用apt-get就可以

4\. 安装thrift
[shell]
$ brew install thrift #Maybe cost a lot of time
$ thrift -version
Thrift version 0.9.1
[/shell]

5\. 下载 [Node-HBase-Thrift2]
这是我写的使用HBase Thrift2接口的Demo，可以使用该Demo进行测试。
[shell]
$ git clone https://github.com/lgrcyanny/Node-HBase-Thrift2
[/shell]

6\. 生成HBase Thrift的相关文件
[shell]
$ cd Node-HBase-Thrift2
$ thrift --gen js:node path-to-hbasesrc/hbase-0.96.1-src/hbase-hrift/src/main/resources/org/apache/hadoop/hbase/thrift2/Hbase.thrift
#then you will get gen-nodejs directory
[/shell]
编译生成连接Thrift的js文件，需要使用HBase 0.96.1源码
BTW, 在你下载的Node-HBase-Thrift2包中，已经有生成的gen-nodejs文件夹，如果你在生成Thrift相关文件时出现任何问题，可以尝试使用这gen-nodejs文件夹下编译好的文件。

8\. npm安装thrift包
[shell]
$ npm install thrift
[/shell]

9\. 打开HDFS，HBase, HBase Thrift2接口
[shell]
$ start-dfs.sh
$ start-hbase.sh
$ hbase thrift2 start -f #Use framed transport， This transport is required when using a non-blocking server. It sends data in frames, where each frame is preceded by length information. Node.js is no-blocking server, we must use &quot;-f&quot; option, or connection lost.
$ jps
23835 HMaster
23764 HQuorumPeer
24109 ThriftServer
23556 SecondaryNameNode
24264 Jps
23384 NameNode
23927 HRegionServer
23463 DataNode
[/shell]

10.插入测试数据
[shell]
$ hbase shell
&gt; put 'users', 'TheRealMT', 'info:email', 'mark@gmail.com'
&gt; put 'users', 'TheRealMT', 'info:passoword', 'abc123'
&gt; put 'users', 'TheRealMT', 'info:name', 'Mark Twain'
&gt; put 'users', 'wwzyhao', 'info:count', 1
&gt; put 'users', 'wwzyhao', 'info:passowrd', 7818271
&gt; put 'users', 'wwzyhao', 'info:email', 7818271
&gt; put 'users', 'wwzyhao', 'info:gender', 'male'
&gt; scan 'users'
ROW                                      COLUMN+CELL
 TheRealMT                               column=info:email, timestamp=1392811829718, value=mark@gmail.com
 TheRealMT                               column=info:name, timestamp=1392811829718, value=Mark Twain
 TheRealMT                               column=info:password, timestamp=1392814983344, value=abc123
 wwzyhao                                 column=info:count, timestamp=1393489157537, value=male
 wwzyhao                                 column=info:email, timestamp=1393407097540, value=sks66782@gmail.com
 wwzyhao                                 column=info:gender, timestamp=1393489039523, value=male
 wwzyhao                                 column=info:password, timestamp=1393416922377, value=7612111
2 row(s) in 0.0560 seconds
[/shell]</p>

### 二、使用Thrift2

为什么不是Thrift1呢？确实，HBase 0.96.1中提供了两个编译文件thrift, thrift2, 前者我们称之为thrift1，我最初使用的是Thrift1，二者最大的痛处是没有文档，官方文档寥寥无几，有一种孤军深入的感觉。不过，API们都是相似的，语义相同，语言不同罢了，这时候代码就是文档。Thrift1的代码规范不如Thrift2，Thrift2更接近Java实现的API调用方式，在使用Thrift1后觉得不好用，就弃用之，投奔Thrift2了。
在完成了配置后，我们开始代码了：
HBase提供的CRUD操作是经典的：Put, Get, Delete, Scan和Increment

#### 1\. HBase Get

[javascript]
var thrift = require('thrift');
var HBase = require('./gen-nodejs/THBaseService');
var HBaseTypes = require('./gen-nodejs/hbase_types');

var connection = thrift.createConnection('localhost', 9090, {
  transport: thrift.TFramedTransport,
  protocol: thrift.TBinaryProtocol
});

connection.on('connect', function () {
  console.log('connected');
  var client = thrift.createClient(HBase, connection);

  // row is rowid, columns is array of TColumn, please refer to hbase_types.js
  var tGet = new HBaseTypes.TGet({row: 'TheRealMT',
    columns: [new HBaseTypes.TColumn({family: 'info'})]});
  client.get('users', tGet, function (err, data) {
    if (err) {
      console.log(err);
    } else {
      console.log(data);
    }
    connection.end();
  });

});

connection.on('error', function(err){
  console.log('error', err);
});
[/javascript]

我在demo中分文件展示了这5种操作，因为每个操作是以event的形式绑定到connection上，如果将所有的操作都写到：
[javascript]
connection.on('connect', function () {
  console.log('connected');
  var client = thrift.createClient(HBase, connection);
  // ...
});
[/javascript]
运行时connection.end()后，会出现"Write after end"的Error，我觉得分文件封装是更好的方式。

#### 2\. HBase Put

这个操作是向HBase写入数据, 包括更新和插入。
[javascript]
connection.on('connect', function () {
  console.log('connected');
  var client = thrift.createClient(HBase, connection);

  var tPut = new HBaseTypes.TPut({row: 'wwzyhao',
    columnValues: [
    new HBaseTypes.TColumnValue({family: 'info', qualifier: 'hobbies', value: 'music'}),
    new HBaseTypes.TColumnValue({family: 'info', qualifier: 'name', value: 'Thomas Zhang'})
    ]});
  client.put('users', tPut, function (err) {
    if (err) {
      console.log(err);
      return;
    }
    console.log('success');
    connection.end();
  });
});
[/javascript]

#### 3\. HBase Delete

[javascript]
connection.on('connect', function () {
  console.log('connected');
  var client = thrift.createClient(HBase, connection);

  // Please run the command in shell first: put 'users', 'wwzyhao', 'info:gender', 'male'
  var tDelete = new HBaseTypes.TDelete({row: 'wwzyhao',
   columns: [new HBaseTypes.TColumn({family: 'info', qualifier: 'gender'})]});
  client.deleteSingle('users', tDelete, function (err) {
    if (err) {
      console.log(err);
      return;
    } else {
      console.log('delete success');
    }
    connection.end();
  });
});
[/javascript]

#### 4\. HBase Scan

Get操作本质上是基于Scan的，Get只能获取单行的数据，Scan可以获取多行的数据。
[javascript]
connection.on('connect', function () {
  console.log('connected');
  var client = thrift.createClient(HBase, connection);&lt;/p&gt;

  var tScanner = new HBaseTypes.TScan({startRow: 'TheRealMT',
    columns: [new HBaseTypes.TColumn({family: 'info'})]});
  client.openScanner('users', tScanner, function (err, scannerId) {
    if (err) {
      console.log(err);
      return;
    }
    console.log('scannerid : ' + scannerId);
    client.getScannerRows(scannerId, 10, function (serr, data) {
      if (serr) {
        console.log(serr);
        return;
      }
      console.log(data);
    });
    client.closeScanner(scannerId, function (err) {
      if (err) {
        console.log(err);
      }
    });
    connection.end();
  });
});
[/javascript]

#### 5\. HBase Increment

[javascript]
connection.on('connect', function () {
  console.log('connected');
  var client = thrift.createClient(HBase, connection);

  // put 'users', 'wwzyhao', 'info', 'count', 1
  var tIncrement = new HBaseTypes.TIncrement({
    row:'wwzyhao',
    columns: [new HBaseTypes.TColumn({family: 'info', qualifier: 'count'})]
  });
  client.increment('users', tIncrement, function (err) {
    if (err) {
      console.log(err);
      return;
    }
    console.log('increment success.');
    connection.end();
  });
});
[/javascript]

### References

1.[Getting Started with HBase and Thrift for Node](http://dailyjs.com/2013/07/04/hbase/)
2\. HBase in Action
3\. [building-apache-thrift-on-mac-os-x](http://blog.evernote.com/tech/2012/12/20/building-apache-thrift-on-mac-os-x/)
4\. [Thrift API](https://wiki.apache.org/hadoop/Hbase/ThriftApi "Thrift API")