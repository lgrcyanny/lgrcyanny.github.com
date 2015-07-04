title: Hadoop Overview
tags:
  - Hadoop
  - Learning
  - Research
id: 370
categories:
  - Hadoop
date: 2013-12-05 11:36:08
---

### The motivation for Hadoop

Apache Hadoop is an open source distributed computing framework for large-scale data sets processing. Doug Cutting, who is the creator of Apache Lucene Project, creates it. 

The name of Hadoop is a made-up name, which is the nickname of a stuffed yellow elephant of the kid of Doug. Hadoop has its origins in Apache Nutch, an open source web search engine, and it is built based on work done by Google in early 2000s, specifically on Google papers describing the Google File System (GFS) published in 2003, and MapReduce published in 2004\. 

Hadoop moved out of Nutch to form an independent in Feb. 2006\. Up to now, Hadoop has been powered by many companies, such as Yahoo who claims it has the biggest Hadoop cluster in the world with more than 42000 nodes, LinkedIn who has 4100 nodes, Facebook who has 1400 nodes, Taobao who has the biggest cluster in China with more than 2000 nodes. [more...]

### The problems for traditional big data processing

Why these companies adopt Hadoop? The reason is simple, that we live in a big data age. We are flooded with big data every day on the Internet. Consider that: Facebook hosts approximately 10 billion photos, taken up on PB storage. Every second on eBay, a total merchandise value of 1400 dollars is traded and 10 million new items are listed on eBay every day. Ancestry.com, the genealogy site, store around 2.5 PB of Data.

How to make use of these big data to make analysis? For traditional methods, use only one machine to process computation, which needs faster processor and RAM. Even though the CPU power doubles every 18 months according to Moores’s Law, it hasn’t meet the big data analysis needs. Yet distributed system evolved to allow developers to use multiple machines for a single job, like MPI, PVM and Condor. However, programing for these traditional distributed systems is complex, you have to deal with these problems:

*   It’s difficult to deal with partial failures of the system. Developers spend more time designing for failure than they do actually working on the problem itself.
*   Finite and precious bandwidth must be available to combine data from different disks and transfer time is very slow for big data volume.
*   Data exchange requires synchronization.
*   Temporal dependencies are complicated.

### How can Hadoop save big data analysis

What really counts is big data. Traditional distributed computing can’t handle big data in a decent way, but Hadoop can. Lets see what Hadoop brings to us: 

*   Hadoop provide partial failure support. Hadoop Distributed File System (HDFS) can store large data sets with high reliability and scalability.
*   HDFS provide great fault tolerance. Partial Failure will not result in the failure of the entire system. And HDFS provide data recoverability for partial failure.
*   Hadoop introduce MapReduce, which spares programmers from low-level details, like partial failure. The MapReduce framework will detect failed tasks and reschedule them automatically.
*   Hadoop provide data locality. The MapReduce framework tries to collocate data with the compute nodes. Data is local, and tasks are separated with no dependence on each other. So the shared-nothing and data locality architecture can save more bandwidth and solve the complicated dependence problem

In summary, Hadoop is a great big data processing tool.

### Hadoop Basic Concepts and Core Concepts

The core concepts for Hadoop are to distribute the data as it is initially stored in the system. That is data locality, individual nodes can work on data local to these nodes, and no data transfer over the network is required for initial processing.
Here are the basic concepts for Hadoop:

*   Applications are written in high-level code. Developers don’t worry about network programming, temporal dependencies etc.
*   Nodes talk to each other as little as possible. Developers should not write code which communicates between nodes, that is ‘Shared-Nothing’ architecture.
*   Data is spread among machines in advance. Computation happens where the data is stored, wherever possible, just as near the node as possible. Data is replicated multiple times on the system for increased availability and reliability.

### Hadoop High-Level Overview

Hadoop consists of two important components: HDFS and MapReduce.
HDFS is Hadoop Distributed File System, which is a distributed file system designed to store large data sets and streaming data sets on commodity hardware with high scalability, reliability and availability.

MapReduce is a distributed processing framework designed to operate on large data stored on HDFS, which provides a clean interface for developers.

A set of machines running HDFS and MapReduce is known as a Hadoop Cluster. An individual machine in a cluster is known as nodes. The nodes play different roles. There are two kind important nodes: Master Nodes and Slave Nodes.

There are 5 important daemons on these nodes. 

[![hd2.1](http://cyanny/myblog/wp-content/uploads/2013/11/hd2.1.png)Figure1 Hadoop High-Level Architecture](http://cyanny/myblog/wp-content/uploads/2013/11/hd2.1.png)

As shown in Figure, Hadoop is comprised of 5 separate daemons:

*   NameNode, which holds the metadata for HDFS.
*   Secondary NameNode, which performs housekeeping functions for NameNode, and isn’t a backup or hot standby for the NameNode.
*   DataNode, which stores actual HDFS data blocks. In Hadoop, a large file is split into 64M or 128M blocks.
*   JobTracker, which manages MapReduce jobs, distributes individual tasks to machines running.
*   TaskTracker, which initiates and monitors each individual Map and Reduce tasks.

Each daemon runs on its own Java Virtual Machine (JVM), no Nodes can run 5 daemons at the same time.  

Master Nodes runs the NameNode, Secondary NameNode, JobTracker daemons. And only one of each of these daemons runs on the cluster.
Slave Nodes run the DataNode and TaskTracker daemons. A slave node will run both of these daemons. All of these Slave Nodes run in parallel, each on their own part of the overall dataset locally.

Just for very small clusters, the NameNode, JobTracker and the Secondary NameNode run on a single machine. However, when there are beyond 20-30 nodes. It’s better to run each of them on individual nodes. 

### Hadoop Ecosystem

Hadoop has a family of related projects based on the infrastructure for distributed computing and large-scale data processing. The core projects are apache open source projects. Except for HDFS and MapReduce, some hadoop related projects are:

*   Ambari, a Hadoop management and cluster monitoring system.
*   Avro, a serialization system for efficient, cross-language RPC and persistent data storage.
*   Pig, a data flow language and execution environment for exploring very large datasets, which runs on HDFS and MapReduce clusters.
*   HBase, provide random, realtime read/write access to BigData. The project built a column-oriented database, host very large tables ---- billions of rows X millions of columns. The project is based on Google’s paper BigTable: A Distributed Storage System for Structured Data.
*   Hive, which is distributed data warehouse. Hive manages data stored in HDFS and provides a query language based on SQL, which is translated by runtime engine to MapReduce Job.
*   Zookeeper, a distributed, highly available coordination service. It provides centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services, which make build distributed applications more efficient.
*   Oozie, a service for running and scheduling workflows of Hadoop jobs, which includes MapReduce, Pig, Hive and sqoop jobs.
*   Mahout, a scalable machine learning libraries based on Hadoop HDFS and MapReduce.

Resources:
[Part 0 Hadoop Overview](http://cyanny/myblog/2013/12/05/hadoop-overview/ "Hadoop Overview")
[Part 1 Hadoop HDFS Review](http://cyanny/myblog/2013/12/05/hadoop-hdfs-review/ "Hadoop HDFS Review")
[Part 2 Hadoop HDFS Federation](http://cyanny/myblog/2013/12/05/hadoop-hdfs-federation/ "Hadoop HDFS Federation")
[Part 3 Hadoop HDFS High Availability(HA)](http://cyanny/myblog/2013/12/05/hadoop-hdfs-high-availability/ "Hadoop HDFS High Availability(HA)")
[Part 4 Hadoop MapReduce Overview](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-overview/ "Hadoop MapReduce Overview")
[Part 5 Hadoop MapReduce 1 Framework](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-1-framework/ "Hadoop MapReduce 1 Framework")
[Part 6 Hadoop MapReduce 2 (YARN)](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-2-yarn/ "Hadoop MapReduce 2 (YARN)")
[Part 7 Hadoop isn’t Silver Bullet](http://cyanny/myblog/2013/12/05/hadoop-isnt-silver-bullet/ "Hadoop isn’t Silver Bullet")