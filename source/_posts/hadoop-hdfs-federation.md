title: 'Hadoop HDFS Federation '
tags:
  - Hadoop
  - Learning
  - Research
id: 528
categories:
  - Hadoop
date: 2013-12-05 14:48:10
---

In traditional HDFS architecture, there is only one NameNode in a cluster, which maintains all of the namespace and block map. Regarding that Hadoop cluster is becoming larger and larger one enterprise platform and every file and block information is in NameNode RAM, when there are more than 4000 nodes with many files, the NameNode memory will reach its limit and it becomes the limiting factor for cluster scaling. In Hadoop 2.x release series, Hadoop introduces HDFS Federation, which allows a cluster to scale by adding NameNodes, each of which manages a portion of the filesystem namespace.[more...]
[![hd4.1](http://cyanny/myblog/wp-content/uploads/2013/11/hd4.1.png)Figure1 HDFS Federation](http://cyanny/myblog/wp-content/uploads/2013/11/hd4.1.png)
As shown in Figure1, there are many federated NameNodes which manages its own namespace. The NameNodes are independent and don’t require coordination with each other. The DataNodes are used as common storage for blocks. Each DataNode registers with all the NameNodes in the cluster and send periodic heartbeats and block report and handles commands from the NameNodes.
Each NameNode is responsible for two tasks: Namespace management and Block Management. For the two tasks, mange NameNode manages its own Namespace Volume, which consists of two parts:

*   Namespace: the metadata for each file and block
*   Block Pool: it is a set of blocks that belong to a single namespace. The Block Pool is in charge of block management task, including processing block reports and maintaining location of blocks.
Block pool storage is not partitioned , so DataNodes register with each NameNode in the cluster and store blocks from multiple block pools. The NameNode namespace must to generate block ID for new blocks.
A NameSpace Volume is a self-contained uinit . Each NameNode has no need to contact other NameNode and it’s failure will not influence other NameNode.
To access a federated HDFS, clients use client-side mount tables to map file paths to NameNodes. A new identifier ClusterID is added to all the nodes in the cluser. Federated NameNodes is configured by ViewFileSystem and the viewfs://URIs.
The HDFS Federation brings several benefits:

- NameSpace Scalability: It will support large clusters with many files, just add more NameNode to scale namespace.
- Performance: Break the limitation of single node, more NameNodes means more read/write operations and the throughput is improved.
- Isolation: A single NameNode has no isolation in multi user environment. While multiple NameNodes can provide isolated NameSpace for both production and experiment application. 

Resources:
[Part 0 Hadoop Overview](http://cyanny/myblog/2013/12/05/hadoop-overview/ "Hadoop Overview")
[Part 1 Hadoop HDFS Review](http://cyanny/myblog/2013/12/05/hadoop-hdfs-review/ "Hadoop HDFS Review")
[Part 2 Hadoop HDFS Federation](http://cyanny/myblog/2013/12/05/hadoop-hdfs-federation/ "Hadoop HDFS Federation")
[Part 3 Hadoop HDFS High Availability(HA)](http://cyanny/myblog/2013/12/05/hadoop-hdfs-high-availability/ "Hadoop HDFS High Availability(HA)")
[Part 4 Hadoop MapReduce Overview](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-overview/ "Hadoop MapReduce Overview")
[Part 5 Hadoop MapReduce 1 Framework](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-1-framework/ "Hadoop MapReduce 1 Framework")
[Part 6 Hadoop MapReduce 2 (YARN)](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-2-yarn/ "Hadoop MapReduce 2 (YARN)")
[Part 7 Hadoop isn’t Silver Bullet](http://cyanny/myblog/2013/12/05/hadoop-isnt-silver-bullet/ "Hadoop isn’t Silver Bullet")