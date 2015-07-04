title: HBase Architecture Analysis Part 3 Pros and Cons
tags:
  - Big Data
  - Hadoop
  - HBase
id: 638
categories:
  - Hadoop
  - HBase
date: 2014-03-13 22:13:20
---

## 5\. HBase Physical Architecture

Figure 5.1 shows the deployment view for HBase cluster:
HBase is the master-slave cluster on top of HDFS. The classic deployment is as follows:
➢** Master node:** one HMaster and one NameNode running on a machine as the master node.
➢ **Slave node:** Each node is running one HRegionServer and one DataNode. And each node report status to the master node and Zookeeper.
➢** Zookeeper:** HBase is shipped with ensemble Zookeeper, but for large clusters, using existing Zookeeper is better. Zookeeper is crucial, the HMaster and HRegionServers will register on Zookeeper.
➢ **Client**: There can be many clients to access HRegionServer, like Java Client, Shell Client, Thrift Client and Avro Client

<!--more--> [![Image and video hosting by TinyPic](http://i58.tinypic.com/6eg29d.jpg)](http://tinypic.com?ref=6eg29d)
Figure 5.1 HBase Deployment View

## 6.HBase Pros

**1\. Master-Slave Architecture**
HBase build on top HDFS, HDFS is mater-slave architecture, HBase follows this style. It will improve the interoperability between HBase and HDFS.
This style do good for load balance, the HMaster takes over the load balance job, assign regions to region servers and auto failover dead RegionServers.

**2\. Real-time , random big data access**
HDFS handles large blocks of data well, but small blocks of data with real-time access is not efficient. HBase uses LSM-trees as data storage architecture. Data is written into WAL first, then to Mem Store. WAL is in case of data lost before Mem Store is flushed to HDFS. Mem Store will be flushed to HDFS when it’s filled up. The mechanism insures that data write is operated in memory, no need of HDFS disk access, which improves write performance.
Meanwhile, data read accesses Mem Store first, then to Block Cache, and last the HFiles, so for random data access, if there is cache in memory, the read performance is very well.
In short, HBase brings big data online, it is efficient for real-time operations on big data(TB, PB);

**3\. Column-Oriented data model for big sparse table**
HBase is NO-SQL, column-family-oriented, each column family is stored together in HStore. The key-value data model stores only non-empty values, no null values. So for large spares table, the disk usage is great in HBase, not like RDBMS storing large quantities of null values.

**4\. LSM-trees vs. B+ tree**
HBase data storage architecture is LSM-trees. A big feature of LSM-trees is the merge housekeeping, which merge small files into a larger one to reduce disk seek. The housekeeping is a background process, which will not impact real-time data access.
LSM-trees transform random data access into sequential data access, which improves read performance. But B+ tree in RDBMS requires more disk seeks for data modifications, which is very efficient for big data process.

**5\. Row-level Atomic**
HBase has no strictly ACID features like RDBMS, how to insure data consistent in HBase and how to avoid dirty read. HBase is row-level automic, in other words, each Put operation on a given row either success or fail. Meanwhile, CheckAnd*, Increment* operations are also atomic.
The scan operation across the table is not on a snapshot of a table, if the data is updated before a Scan operation, the updated version is read by Scan, It ensures data consistent.
HBase is good at loose coupled data and not high requirements for consistent. Row-level atomic is a mechanism to improve data consistent, but data consistent is really a shortage of HBase.

**6\. High Scalability**
HBase data is stored across the cluster, the cluster has many HRegionServer, which can be scaled. HBase build on top of HDFS, HDFS is high scalability, that is the DataNode can be scaled. HDFS brings great scalability to HBase. Moreover, the master-slave architecture do well in data scalability.

**7\. High Reliability**
The HBase cluster is built on commodity hardware, which will be dead at any time. HBase data is replicated across the cluster. Data replication is done by HDFS. HDFS stores 3 replications for each block: on the same node, on another node on the same rack, on another node on another rack. The feature brings high reliability for HBase.

**8\. Auto Failover**
HMaster is in charge of auto failover work. When HMaster detects the dead HRegionServer. It will find that the region assignment is invalid, and it will do region assignment again. Auto failover brings great availability for HBase, which is no need of manually HRegionServer failover.

**9\. Data auto sharding**
Since data is stored distributed across the cluster. HBase must makes use of all of the machines in the cluster. HRegionServer stores many HRegions. When a HRegion size reaches its threshold, HRegionServer will split it into half.
In addition, when the number of regions is over the threshold that the HMaster can handle, the HMaster will merge small regions into a larger one. Auto sharding brings HBase great load balance and keeps programmer away from the details of distributed data sharding.

**10\. Simple Client Interface**
HBase provides 5 basic data operations : Put, Get, Delete, Scan and Increment. The Interface is easy to use. HBase provide java interface, and non java interface: Thrift, Avro, REST and Shell. Each interface is transparent for developers to do data operations on HBase.

## 7\. HBase Cons

**1\. Single Point of Failure (SPOF)**
The master-slave architecture always has SPOF, HBase is no exception. When there is only on HMaster in a cluster, if the cluster has more than 4000 nodes, the HMaster is the performance bottleneck. Even though client talks directly to HRegionServer, when the HMaster is down, the cluster can be “steady” for a short time, HMaster long time down will bring down the cluster.
It is possible to start multi HMaster in a cluster, but only one is active, which is also a performance bottleneck.

**2\. No transaction**
As we talked before, HBase data consistent is a shortage. Interrow operations are not atomic, which will bring dirty read. HBase is good at louse coupled data but not good at highly rational data records. That is an important point we must be aware of.

**3\. No Join, if you want, use MapReduce**
HBase has no join operation, so data model has to be redesign when migrate from RDBMS, which is a big headache. For highly rational data records, HBase will make you sick. But if you really want implement join, you can use MapReduce, please refer to “HBase in Action”.

**4\. Index only on key, sorted by key, but RDBMS can be indexed on arbitrary column**
The column-oriented data model is key-value style, HBase only has index on key: &lt;rowkey, column family, qualifier, version&gt;, rowkey design is the most important thing.
The data model paradigm for HBase is quite different for RDBMS. HBase has no index on value. Scan with columnvalue filter for big data is very slow. But RDBMS can be indexed on arbitrary column.
If you want to improve index for HBase, you can use MapReduce to build Inverted Index.

**5\. Security Problem**
Finally, HBase has security problem. RDBMS like MySQL has authentication and authorization feature, different user has different data access authority. But HBase has no such feature yet. But we can see that, improve security will lower the performance, If the online application can insure the security, HBase will has no need to care about that.

**Refereces**
[1] George, Lars. HBase: the definitive guide. O'Reilly Media, Inc., 2011.
[2] Dimiduk, Nick, Amandeep Khurana, and Mark Henry Ryan. HBase in action. Manning, 2013.
[3] Chang, Fay, et al. "Bigtable: A distributed storage system for structured data."ACM Transactions on Computer Systems (TOCS) 26.2 (2008): 4.
[4] White, Tom. Hadoop: The Definitive Guide: The Definitive Guide. O'Reilly Media, 2009.
[5] Kruchten, Philippe B. "The 4+ 1 view model of architecture." Software, IEEE12.6 (1995): 42-50.
[6] Apache HBase, “The Apache HBase Reference Guide”, apache.org, 2013