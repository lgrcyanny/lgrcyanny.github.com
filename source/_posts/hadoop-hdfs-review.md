title: Hadoop HDFS Review
tags:
  - Hadoop
  - Learning
  - Research
id: 388
categories:
  - Hadoop
date: 2013-12-05 12:00:32
---

### HDFS Basic Concepts

Hadoop Distributed File System, know as HDFS, is a distributed file system designed to store large data sets and streaming data sets on commodity hardware with high scalability, reliability and availability. 

HDFS is written in Java based on the Google File System (GFS). HDFS has many advantages compared with other distributed file systems:

**1\. Highly fault-tolerant**

Fault-tolerant is the core architecture for HDFS. Since HDFS can run on low-cost and unreliable hardware, the hardware has a non-trivial probability of failures. HDFS is designed to carry on working without a noticeable interruption to the user in the face of such failure. HDFS provides redundant storage for massive amounts of data and Heartbeat for failure detection. 
When one node fails, the master will detect it and re-assign the work to a different node. Restarting a node doesn’t need communicating with other data node. When failed node restart it will be added the system automatically. If a node appears to run slowly, the master can redundantly execute another instance of the same task, know as ‘speculative execution’.[more...]

**2\. Streaming Data Access**

HDFS is designed with the most efficient data processing pattern: a write once, read-many times pattern. HDFS split large files into blocks, usually 64MB or 128MB. HDFS performs best with a modest number of large files. It prefers millions, rather than billions of files, and each of which is 100MB or more. No random writes to files is allowed. Also HDFS is optimized for large, steaming reads of files. No Random reads is allowed. 

A data set is generated and copied from source and replicated spread the HDFS system. It is efficient to load data from HDFS for big data analysis.

**3\. Large data sets**

Large data means files that are MB, GB or TB in size. There are Hadoop clusters today that store PB data. HDFS support high aggregate data bandwidth and scale to hundreds of nodes in a single cluster. It also supports tens of millions of files processing.

### HDFS NameNode and DataNodes Architecture

HDFS has a master/slave architecture. As shown in Figure1\. 

The **master node** is the NameNode, which managers the file system namespace, file metadata and regulates the access interface for files by clients. A cluster has only one NameNode, which maintain the map of file metadata and  the location of blocks. 

The **slave nodes** are the DataNodes. A cluster has many DataNodes, which holds the actual data blocks.

[![hd3.1](http://cyanny/myblog/wp-content/uploads/2013/11/hd3.1.png)Figure1 HDFS Architecture](http://cyanny/myblog/wp-content/uploads/2013/11/hd3.1.png)

### NameNode

NameNode maintains the namespace tree, which is logical location and the mapping of file blocks to DataNodes, which is the physical location.

The NameNode executes file system namespace operations like opening, closing, and renaming files and directories. It provides POSIX interface for client, so that user can access HDFS data with Unix like commands and no need to know about the function of NameNode and DataNodes. It is kind of abstraction which decrease the complexity for data access. 

The NameNode is the pivot in a cluster. A single NameNode greatly simplifies the architecture of HDFS. NameNode holds all of its metadata in RAM for fast access. It keeps a record of change on disk for crash recovery.

However, once the NameNode fails, the cluster fails. The Single Point of Failure, known as SPOF is really a bottleneck for NameNode. Hadoop 2.0 introduces NameNode Federation and High Availability to solve the problem. We will discuss these in later section.

### NameNode Data Persistent

In case of NameNode crash, the namespace information and metadata updates are stored persistently on the local disk in the form of two files: the namespace image called **FsImage** and the **EditLog**.

FsImage is an image file persistently stores the file system namespace, including the file system tree and the metadata for all the files and the directories in the tree, the mapping of blocks to files and file system properties.

EditLog is a transaction log to persistently record every change that occurs to file system metadata.

However, we have to know there is one thing that is not persistent. The block locations we can call it Blockmap, which is stored in NameNode in-memory, not persistent on the local disk. Blockmap is reconstructed from DataNodes when the system starts by the Blockreport of DataNodes.

When the system starts up, NameNode will load FsImage and EditLog from the local disk, applies the transactions from the EditLog to the in-memory representation of the FsImage, and flushes out the new version of FsImage on disk. Then it truncates the old EditLog since its transactions has been applied to the FsImage. This process is called a **checkpoint**.

### Secondary NameNode

To increase the reliability and availability of the NameNode, a separate daemon known as the Secondary NameNode takes care of some **housekeeping** tasks for the NameNode. Be careful that the Secondary NameNode is not a backup or a hot standby NameNode.

The housekeeping wok is to periodically merge the namespace image FsImage with the EditLog to prevent the EditLog becoming to large. The Secondary NameNode runs on a single Node, which is a separate physical machine with as much memory and CPU requirements as the NameNode.

The Secondary NameNode keeps a copy of the merged namespace image in case of the NameNode fails. But the time lags will result in data loss certainly. And during the NameNode recovery, the reconstruction of the Blockmap will cost too much time. So Hadoop works out the problem with Hadoop High Availability solution.

### DataNodes

The DataNodes holds all the actual data blocks. In sum up, it has three functions: 

*   Serves read and requests from the file system clients.
*   Provides block operations, like creation, deletion and replication upon instruction from the NameNode.
*   Make data Blockreport to NameNode periodically with lists of blocks that they are storing.

In enterprise Hadoop deployment, each DataNode is a java program run on a separate JVM, and one instance of DataNode on one machine. 

In addition, NameNode is a java program run on a single machine. Written in java provides Hadoop good portability.

### The Communication Protocols

HDFS client, NameNode and DataNodes communication protocols are TCP/IP. Client Protocol is TCP connection between a client and NameNode. The DataNode Protocol is the connection between the NameNode and the DataNodes. HDFS makes an abstraction wraps for the Client Protocol and the DataNode Protocol, which called Remote Procedure Call (RPC). NameNode never initiates any RPCs, only responds to RPC requests from clients or DataNodes.

### HDFS Files Organization

In traditional concepts, a disk has a block size, which is the minimum amount of data that it can read or write. Disk blocks are normally 512 bytes. While HDFS has the concept of block as well, which is a much larger unit – 64MB by default.

HDFS large files are chopped up into 64MB or 128MB blocks. This brings several benefits:

*   It can take advantage of any of the disks in the cluster, when the file is larger than any single disk.
*   Making the unit of abstraction of a block simplifies the storage subsystem. HDFS will deal with blocks, rather than a file. Since blocks are a fixed size, it’s easy to calculate how many can be stored on a give disk and eliminate metadata concerns.
*   Normally, a map task will operate on one local block at a time. Bocks spare the complexity of dealing with files.
*   It’s easy to do data replication with blocks for providing fault tolerance and availability. Each block is replicated multiple times. Default is to replicate each block 3 times. Replicas are stored on different nodes.
*   Block data fits well for streaming data. Files are written once and read many times. Blocks minimize the cost of seeking files.

Although files are split into 64MB or 128MB blocks ， if a file is smaller than this full 64MB/128MB will not be split.

### HDFS Data Replication

HDFS each data block has a replica factor, which indicates how many times it should be replicated. Normally, the replica factor is three. Each block is replicated three times and spread on three different machines across the cluster. This provides efficient MapReduce processing because of good data locality. 
[![hd3.2](http://cyanny/myblog/wp-content/uploads/2013/11/hd3.2.png)Figure2 Block Replication](http://cyanny/myblog/wp-content/uploads/2013/11/hd3.2.png)
As shown in Figure2, the NameNode holds the metadata for each map of file and blocks, including filename, replica factor and block-id etc. Block data are spread on different DataNodes.

### Data Replica Placement

Block replica placement is not random. Regarding of the reliability and the performance, HDFS policy is to put one replica on one node in the local rack, the second one on a node in a different remote rack and the third one on a different node in the same remote rack. 
[![hd3.3](http://cyanny/myblog/wp-content/uploads/2013/11/hd3.3.png)Figure 3 HDFS Racks](http://cyanny/myblog/wp-content/uploads/2013/11/hd3.3.png)
As shown in Figure 3, different DataNodes on different racks, Rack 0 and Rack1\. Large HDFS instance run on a cluster of machines that commonly spread across many racks. Network bandwidth for intra-racks is greater than inter-racks. So the HDFS replica policy cuts the inter-rack write traffic and improves write performance. One third of replicas are on one node, two third of replicas are on one rack, the other third are distributed across the remaining racks. This policy guarantees the reliability.

### Data Replication Pipeline

[![hd3.4](http://cyanny/myblog/wp-content/uploads/2013/11/hd3.4.png)Figure 4 Data Replication Pipeline](http://cyanny/myblog/wp-content/uploads/2013/11/hd3.4.png)
As shown in Figure 4, it is the data flow of writing data to HDFS. A client request to request a file does not reach the NameNode immediately. It will follow the steps:
Step 1, the HDFS client caches the file data into a temporary local file until the local file accumulates data worth over one HDFS block size.
Step 2, the client contacts the NameNode, requests add a File.
Step 3, the NameNode inserts the file name into the file system tree and allocates a data block for it. Then responds to the client request with the identity of the DataNodes and the destination data block.
Step 4, suppose the HDFS file has a replication factor of three. The client retrieves a list of DataNodes, which contains that will host a replica of that block. The clients flushes the block of data from the local temporary file to the first DataNode.
Step 5,.the first DataNode starts receiving the data in small portions like 4KB, writes each portion to its local repository and transfers that portion to the second DataNode. Then the second DataNode retrieves the data portion, stores it and transfers to the third DataNode. Data is pipelined from the first DataNode to the third one.
Step 6, when a file is closed, the remaining un-flushed data in temporary local file is transferred to the DataNode. Then the client tells the NameNode that the file is closed.
Step 7, the NameNode commits the file creation operation into a persistent one. Be careful if the NameNode dies before the file is closed, the file is lost.
So far, we can see that file caching policy improves the writing performance. HDFS is write-once-read-many-times. When a client wants to read a file: It should contacts the NameNode to retrieves the file block map, including block id, block physical location. Then it communicates directly with the DataNodes to read data. The NameNode will not be a bottleneck for data transfer.

Resources:
[Part 0 Hadoop Overview](http://cyanny/myblog/2013/12/05/hadoop-overview/ "Hadoop Overview")
[Part 1 Hadoop HDFS Review](http://cyanny/myblog/2013/12/05/hadoop-hdfs-review/ "Hadoop HDFS Review")
[Part 2 Hadoop HDFS Federation](http://cyanny/myblog/2013/12/05/hadoop-hdfs-federation/ "Hadoop HDFS Federation")
[Part 3 Hadoop HDFS High Availability(HA)](http://cyanny/myblog/2013/12/05/hadoop-hdfs-high-availability/ "Hadoop HDFS High Availability(HA)")
[Part 4 Hadoop MapReduce Overview](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-overview/ "Hadoop MapReduce Overview")
[Part 5 Hadoop MapReduce 1 Framework](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-1-framework/ "Hadoop MapReduce 1 Framework")
[Part 6 Hadoop MapReduce 2 (YARN)](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-2-yarn/ "Hadoop MapReduce 2 (YARN)")
[Part 7 Hadoop isn’t Silver Bullet](http://cyanny/myblog/2013/12/05/hadoop-isnt-silver-bullet/ "Hadoop isn’t Silver Bullet")