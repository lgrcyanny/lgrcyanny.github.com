title: Hadoop HDFS High Availability(HA)
tags:
  - Hadoop
  - Learning
  - Research
id: 532
categories:
  - Hadoop
date: 2013-12-05 14:54:59
---

Reliability, Scalability and Availability are the most important three features for a distributed file system. In HDFS, persistent metadata, using the Secondary NameNode to create checkpoint against data loss, using block replication across the cluster against DataNode failure, all of these brings HDFS high reliability. And the master slave architecture wins HDFS great scalability. [more...]
However, HDFS has always had a well-known Single Point of Failure (SPOF) which impacts HDFS’s availability. HDFS fits well for ETL or batch-processing workflows, but in the past few years HDFS begin to be used for real time job, such as HBase. To recover a single NameNode, an administrator starts a new primary NameNode with FsImage and replays its EditLog, waits for Blockreport from DataNodes to leave safe mode. On large clusters with many files and blocks, it will take 30 minutes or more time to recovery. Long time recovery will impact the productivity of internal users and perhaps results in downtime visible to external users.
For these reasons, Hadoop community began a new feature for HDFS called High Availability Name Node (HA Name Node) in 2011\. The HA makes use of an active and a standby NameNode. When the active NameNode fails, the hot standby NameNode will take over serving the role of an active NameNode without a significant interruption. 

### HA Architecture

[![hd5.1](http://cyanny/myblog/wp-content/uploads/2013/11/hd5.1.png)Figure1 HDFS High Availability Architecture](http://cyanny/myblog/wp-content/uploads/2013/11/hd5.1.png)
As shown in Figure 1, HA architecture has several changes
- The NameNodes must use highly available shared storage to share EditLog, such as Network File System (NFS), or BookKeeper. The active NameNode will write its EditLog on the Shared dir, and the Standby NameNode polls the shared log frequently and then applies to its in-memory so as to has the most complete and up-to-date file system state in memory.
- DataNodes must send Blockreports with block locations to both active and standy NameNodes. Because the block mappings are stored in a NameNode’s memory and not on a disk to increase data access performance.
- Clients must be configured to handle NameNode failover, using a mechanism that is transparent to users. 
The HA guarantees if the active NameNode fails, the standby one will take over quickly in tens of seconds, since it has the latest state available in memory: the latest EditLog entries and an up-to-date block mapping.

### Failover

The transition from the active NameNode to the standby NameNode is controlled by the failover controller. Each NameNode runs a failover controller to monitor its namenode failure, which is a heartbeating mechanism. Failover is triggered when a NameNode fails.  

After NameNode failover, the Client must be informed which is active NameNode now. Client Failover is handled transparently by the client library. The HDFS client supports the configuration for multiple network addresses, one for each NameNode. The NameNode is identified by a single logical URI which is mapped the two network addresses of the HA Name Nodes via client-side configuration. The client will retry the two addresses until the active NameNode is found. 

### Fencing

After Failover, HDFS makes use of the fencing technique to make sure that the previous died active NameNode is prevented from doing any damage and causing corruption. These mechanisms includes killing the NameNode’s process, revoking its access to the shared storage directory and disabling its network port via a remote management command. If these techniques fail, HDFS will use STONITH, that is “shoot the other node in the head”, which uses a specialized power distribution unit to forcibly power down the host machine. 

Resources:
[Part 0 Hadoop Overview](http://cyanny/myblog/2013/12/05/hadoop-overview/ "Hadoop Overview")
[Part 1 Hadoop HDFS Review](http://cyanny/myblog/2013/12/05/hadoop-hdfs-review/ "Hadoop HDFS Review")
[Part 2 Hadoop HDFS Federation](http://cyanny/myblog/2013/12/05/hadoop-hdfs-federation/ "Hadoop HDFS Federation")
[Part 3 Hadoop HDFS High Availability(HA)](http://cyanny/myblog/2013/12/05/hadoop-hdfs-high-availability/ "Hadoop HDFS High Availability(HA)")
[Part 4 Hadoop MapReduce Overview](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-overview/ "Hadoop MapReduce Overview")
[Part 5 Hadoop MapReduce 1 Framework](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-1-framework/ "Hadoop MapReduce 1 Framework")
[Part 6 Hadoop MapReduce 2 (YARN)](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-2-yarn/ "Hadoop MapReduce 2 (YARN)")
[Part 7 Hadoop isn’t Silver Bullet](http://cyanny/myblog/2013/12/05/hadoop-isnt-silver-bullet/ "Hadoop isn’t Silver Bullet")