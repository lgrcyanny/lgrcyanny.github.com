title: HBase Architecture Analysis Part1(Logical Architecture)
tags:
  - Big Data
  - Hadoop
  - HBase
id: 625
categories:
  - Hadoop
  - HBase
date: 2014-03-13 21:18:32
---

## 1\. Overview

Apache HBase is an open source column-oriented database. **It is often described as a sparse, consistent, distributed, multi-dimensional sorted map.** HBase is modeled after Google’s “Bigtable: A distributed Storage System for Structured Data”, which can host very large tables with billions of rows, X millions of columns.
<!--more-->
HBase is a No-SQL database, and it has very different paradigms from traditional RDBMS, which speaks SQL and enforce relationships upon data. HBase stores structured and semi-structured data with key-value style. Each row in HBase is located by** &lt;Rowkey, Column Family, Column Qualifier, Version&gt;.**

HBase stores data distributed on Hadoop HDFS shipped with random data access. HDFS is Hadoop Distributed File System, which stores big data sets with streaming style. It has high reliability, scalability and availability but the cons is that HDFS split data into blocks (64MB or 128MB), it can handle large chucks of data very well, but not very well for small piece of data and low-latency data access. In other words, HDFS isn’t fit for online real time read and write, and that’s the meaning of HBase. HBase goal is to make use of cloud distributed data storage for online real time data access.
The blog analyzes the architecture of HBase with the method of “4+1” views by Kruthten.

## 2\. Use Case View

[![Image and video hosting by TinyPic](http://i60.tinypic.com/346rmhh.jpg)](http://tinypic.com?ref=346rmhh)
Figure 2.1 User Case View of HBase
The figure 2.1 shows the use case view of HBase. Since from the architecture view, the figure gives more details about the main features of HBase:
As shown on figure 2.1, the main features of HBase are as follows:
**1\. For User:**

*   **Read/Write big data in real time:** HBase provides transparent and simple interface for user. User has no need to care about details about data transforming.
*   **Manage Data Model:** User Need to create and manage column-oriented data model about business logic, and HBase provide Interface for them to do management.
*   **Manage System:** User can manually configure and monitor the status of HBase
** 2\. For the System**

*   **Store data distributed on HDFS:** HBase has great scalability and reliability based on HDFS.
*   ** Data random online, real time access:** HBase provides block cache and index on files for real time queries.
*   **Automatic Data Sharding:** HBase partitions big data automatically on distributed node.
*   **Automatic failover:** Since hardware failure on clusters is inevitable, HBase can failover automatically for high reliability and availability.

## 3\. Logical Architecture

### 3.1 High Level Architecture and Style

[![Image and video hosting by TinyPic](http://i60.tinypic.com/2eezt6x.jpg)](http://tinypic.com?ref=2eezt6x)
Figure 3.1 High Level Architecture for HBase
As shown in Figure3.1, the HBase architecture is layered overall, layered architecture style is a typical style for database.
**1\. Layered Architecture Style**
➢** Client Layer:** The layer is for user, HBase provides easy-to-use java client API, and ships with non-java API as well, such as shell, Thrift, Avro and REST.
➢ **HBase Layer:** The layer is the core logic of HBase. Overall, HBase has two important components: HMaster and RegionServer. We will discuss them later.
Moreover, another important component is Zookeeper, which is a distributed coordination service modeled after Google’s Chubby. It’s another open source project under Apache.
➢ **Hadoop HDFS Layer:** HBase build on top of HDFS, which provides distributed data storage and data replication.

**2\. Master-Slave Architecture Style**
For distributed applications, how to handle data blocks is a key point. There are two typical styles: **centralized and non-centralized**. Centralized mode is very available for scalability and load balancer, but there is Single Pont of Failure(SPOF) problem and performance bottleneck on the master node. Non-centralized mode has no SPOF problem and performance bottleneck, but not very easy to do load balance and there is data consistent problem.
HBase utilizes Centralized Mode: Master-Slave Style. BTW, Hadoop is also master-slave. The architecture style in Hadoop ecosystem is kind of consistent master-slave style.
[![Image and video hosting by TinyPic](http://i62.tinypic.com/ieff35.jpg)](http://tinypic.com?ref=ieff35)
Figure 3.2 Master-Slave Architecture Style of HBase
As shown in figure 3.2:

*   **HMaster** is the master in such style, which is responsible for RegionServer monitor, region assignment, metadata operations, RegionServer Failover etc. In a distributed cluster, HMaster runs on HDFS NameNode.
*   **RegionServer** is the slave, which is responsible for serving and managing regions. In a distributed cluster, it runs on HDFS DataNode.
*   **Zookeeper** will track the status of Region Server, where the root table is hosted. Since HBase 0.90.x, it introduces an even more tighter integration with Zookeeper. The heartbeat report from Region Server to HMaster is moved to Zookeeper, that is zookeeper has the responsibility of tracking Region Server status. Moreover, Zookeeper is the entry point of client, which enable query Zookeeper about the location of the region hosting the –ROOT- table.

### 3.2 HBase Data Model

HBase data model is Column-Family-Oriented. Some special characteristics of HBase data model is as follows:
➢ **Key-Value store:** Each cell in a table is specified by the four dimensional key: . The table will not store null values like RDBMS, it works well for large sparse table.
➢ **Sorted by key, index only on key.** Row key design is the only most important thing in HBase Schema Design.
➢ **Each cell can store different versions,** specified by Timestamp by default
➢ The column qualifier is** schema-less,** can be changed during run-time
➢ The column qualifier and value is treated a**s arbitrary bytes**
[![Image and video hosting by TinyPic](http://i58.tinypic.com/8wmlhj.jpg)](http://tinypic.com?ref=8wmlhj)
Table 3.1 The logical view of data model
As shown in Table 3.1, is a logical view of a big table, the table is parse, each row has a unique row key, and the table has three column family: cf1, cf2, cf3\. cf1 has two qualifiers: q1-1, q1-2\. q1-1 on timestamp t1 has value v1.
[![Image and video hosting by TinyPic](http://i61.tinypic.com/14smvbr.jpg)](http://tinypic.com?ref=14smvbr)
Table 3.2 The physical view of data model
In RDBMS, a big table as shown in Table 3.1 will store a lot of null values, but for HBase is not, key-value styles make HBase store only non-empty values as shown in Table3.2\. It’s the physical view, a table is partitioned by Column Family, each Column Family is stored in a HStore in a table region on a Region Server.
Since the main purpose of the report is architecture analysis, the table design of HBase will not discuss in detail, you can refer to the “HBase in Action”

### 3.3 HBase Distributed Mode

#### 3.3.1 Big Table Splitting

HBase can run on local file system, called “standalone mode” for development and testing. For production, HBase should run on cluster on top of HDFS. Big tables are split and stored across the cluster.
[![Image and video hosting by TinyPic](http://i62.tinypic.com/16h0rdk.jpg)](http://tinypic.com?ref=16h0rdk)
Figure 3.3 Region split on big table
A table is split into roughly equal size. Regions are assigned to Region Servers across the cluster. And Region Servers host roughly equal number of regions. As shown in Figure3.3, Table A is split into four regions, each of which is assigned to a Region Server.

#### 3.3.2 Region Server

As shown in figure3.4, the Region Server Architecture. It contains several components as follows:

*   **One Block Cache**, which is a LRU priority cache for data reading.
*   **One WAL(Write Ahead Log):** HBase use Log-Structured-Merge-Tree(LSM tree) to process data writing. Each data update or delete will be write to WAL first, and then write to MemStore. WAL is persisted on HDFS.
*   **Multiple HRegions:** each HRegion is a partition of table as we talk about in 3.3.1.
*   **In a HRegion:** Multiple HStore: Each HStore is correspond to a Column Family
*   **In a HStore:**  **One MemStore:** store updates or deletes before flush to disk. **Multiple StoreFile**, each of which is correspond to a HFile
*   **A HFile** is immutable, flushed from MemStore, persisted on HDFS
[![Image and video hosting by TinyPic](http://i60.tinypic.com/c28l.jpg)](http://tinypic.com?ref=c28l)
Figure3.4 Region Server Architecture

#### 3.3.3 -ROOT- and .META table

Since table are partitioned and store across the cluster. How can the client find which region hosting a specific row key range? There are two special catalog tables, -ROOT- and .META. table for this.
➢**.META. table**: host the region location info for a specific row key range. The table is stored on Region Servers, which can be split into as many region as required.
➢**-ROOT- table:** host the .META. table info. There is only one Region Server store the –ROOT- table. And the Root region never split into more than one region.
The –ROOT- and .META. table structure logically looks as a B+ tree as shown in figure 3.5.
The RegionServer RS1 host the –ROOT- table, the .META. table is split into 3 regions: M1, M2, M3, hosted on RS2, RS3, RS1\. Table T1 contains three regions, T2 contains four regions. For example, T1R1 is hosted on RS3, the meta info is hosted on M1.
[![Image and video hosting by TinyPic](http://i60.tinypic.com/soa649.jpg)](http://tinypic.com?ref=soa649)
Figure 3.5 –ROOT-, .META., and User table viewed as B+ tree

#### 3.3.4 Region Lookup

How can client find where the –ROOT- table is? Zookeeper does this work, and how to find the region where the target row is. Even though it belongs to process architecture, it is related to the section, hence we discuss it in advance. let’s look at an example as shown in figure3.6.
1\. Client query Zookeeper: where is the –ROOT-? On RS1.
2\. Client request RS1: Which meta region contains row: T10006? META1 on RS2
3\. Client request RS2: Which region can find the row T10006? Region on RS3
4\. Client get the from the region on RS3
5\. Client cache the region info, and is refreshed until the region location info changed.
[![Image and video hosting by TinyPic](http://i60.tinypic.com/30ml56q.jpg)](http://tinypic.com?ref=30ml56q)
Figure 3.6 Region Lookup Process

### 3.4 HBase Communication Protocol

[![Image and video hosting by TinyPic](http://i61.tinypic.com/2zggmrk.jpg)](http://tinypic.com?ref=2zggmrk)
Figure 3.7 HBase Communication Protocol
In HBase 0.96 release, HBase has moved its communication protocol to** Protocol Buffer.** Protocol buffer is a method of serializing structured data developed by Google. Google uses it for almost all of its internal RPC protocol and file formats.
Protocol buffer involves an Interface Description Language for cross language service like Apache Thrift.
Basically, the communication between sub-systems of HBase is RPC, which is implemented in Protocol Buffer.
The main protocols are as follows:
➢ **MasterMonitor Protocol:** client use it to monitor the status of HMaster
➢ **MasterAdmin Protocol:** client use it to do management for HMaster, such as region manually management, table meta info management.
➢** Admin Protocol:** client use it communicate with HRegionServer, to do admin work, such as region split, store file compact, WAL management.
➢** Client Protocol:** Client use it to read/write to HRegionServer
➢ **RegionServerStatus Protocol**: HRegionServer use it to communicate with HMaster: including the request and response of server startup, server fatal error, server status report.

### 3.5 Log-Structured Merge-Trees(LSM-trees)

No-SQL database usually uses LSM-trees as data storage process architecture. HBase is no exception. As we all known, RDBMS adopts B+ tree to organize its indexes, as shown in Fugure 3.3\. These B+ trees are often 3-level n-way balance trees. The nodes of a B+ tree are blocks on disk. So for a update by RDBMS, it likely needs 5 times disk operation. (3 times for B+ tree to find the block of the target row, 1 time for target block read, and 1 time for data update).
On RDBMS, data is written randomly as heap file on disk, but random data block decrease read performance. That’s why we need B+ tree index. B+ tree is fit well for data read, but is not efficient for data updates. Given the large distributed data, B+ tree is not the competitor for LSM-trees so far.
[![Image and video hosting by TinyPic](http://i58.tinypic.com/7286l0.jpg)](http://tinypic.com?ref=7286l0)
Figure3.8 B+ tree

LSM-trees can be viewed as n-level merge-trees. It transforms random writes into sequential writes using logfile and in-memory store. Figure3.9 shows data write process of LSM-trees.
[![Image and video hosting by TinyPic](http://i59.tinypic.com/2mw8nky.jpg)](http://tinypic.com?ref=2mw8nky)
Figure 3.9 LSM-trees

➢ **Data Write(Insert, update):** Data is written to logfile sequentially first, then to in-memory store, where data is organized as sorted tree, like B+ tree. When the in-memory store is filled up, the tree in the memory will be flushed to a store file on disk. The store files on disk is arranged like B+ tree, as the C1 Tree shown in Figure 3.9 . But store files are optimized for sequential disk access.
➢**Data Read: I**n-memory store is searched first. Then search the store files on disk.
➢**Data Delete:** Give a data record a “delete marker”, system background will do housekeeping work by merging some store files into a larger one to reduce disk seeks. A data record will be deleted permanently during the housekeeping.
LSM-trees’ data updates are operated in memory, no disk access, it’s faster than B+ tree. When the data read is always on the data set that is written recently, LSM-trees will reduce disk seeks, and improve performance. When disk IO is the cost we must consider, LSM-trees is more suitable than B+ tree.