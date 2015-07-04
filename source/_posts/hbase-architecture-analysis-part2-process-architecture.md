title: HBase Architecture Analysis Part2(Process Architecture)
tags:
  - Big Data
  - Hadoop
  - HBase
id: 632
categories:
  - Hadoop
  - HBase
date: 2014-03-13 22:00:11
---

## 4\. Process Architecture

### 4.1 HBase Write Path

The client doesn’t write data directly into HFile on HDFS. Firstly it writes data to WAL(Write Ahead Log), and Secondly, writes to MemStore shared by a HStore in memory.
<!--more-->
[![Image and video hosting by TinyPic](http://i58.tinypic.com/20i77l0.jpg)](http://tinypic.com?ref=20i77l0)
Figure4.1 HBase Write Path

**MemStore** is a write buffer(64MB by default). When the data in MemStore accumulates its threshold, data will be flush to a new HFile on HDFS persistently. Each Column Family can have many HFiles, but each HFile only belongs to one Column Family.
**WAL** is for data reliability, WAL is persistent on HDFS and each Region Server has only on WAL. When the Region Server is down before MemStore flush, HBase can replay WAL to restore data on a new Region Server.
A data write completes successfully only after the data is written to WAL and MemStore.

### 4.2 HBase Read Path

As shown in Figure 4.2, it’s the read path of HBase.
1\. Client will query the MemStore in memory, if it has the target row.
2\. When MemStore query failed, client will hit the BlockCache.
3\. After the MemStore and BlockCache query failed, HBase will load HFiles into memory which may contain the target row info.
The MemStore and BlockCache is the mechanism for real time data access for distributed large data.
**BlockCache is a LRU(Lease Recently Used) priority cache.** Each RegionServer has a single BlockCache. It keeps frequently accessed data from HFile in memory to reduce disk data reads. The “Block”(64KB by default) is the smallest index unit of data or the smallest unit of data that can be read from disk by one pass.
For random data access, small block size is preferred, but block index consumes more memory. And for sequential data access, large block size is better, fewer index save more memory.

[![Image and video hosting by TinyPic](http://i61.tinypic.com/290x9qt.jpg)](http://tinypic.com?ref=290x9qt)
Figure 4.2 HBase Read Path

### 4.3.HBase Housekeeping: HFile Compaction

The data of each column family is flush into multiple HFiles. Too many HFiles means many disk data reads and lower the read performance. Therefore, HBase do HFile compaction periodically.
[![Image and video hosting by TinyPic](http://i62.tinypic.com/2nk85xj.jpg)](http://tinypic.com?ref=2nk85xj)
Figure4.3 HFile Compaction

**➢Minor Compaction**
It happens on multiple HFiles in one HStore. Minor compaction will pick up a couple of adjacent small HFiles and rewrite them into a larger one.
The process will keep the deleted or expired cells. The HFile selection standard is configurable. Since minor compaction will affect HBase performace, there is an upper limit on the number of HFiles involved (10 by default).
**➢Major Compaction**
Major Compaction compact all HFiles in a HStore(Column Family) into one HFile. It is the only chance to delete records permanently. Major Compaction will usually have to be triggered manually for large clusters.
Major Compaction is not region merge, it happens to HStore which will not result in region merge.

### 4.4 HBase Delete

When HBase client send delete request, the record will be marked “tombstone”, it is a “predicate deletion”, which is supported by LSM-tree. Since HFile is immutable, deletion isn’t available for HFile on HDFS. Therefore, HBase adopts major compaction(Section 4.3) to clean up deleted or expired records.

### 4.5 Region Assignment

It is a main task of HMaster:
1\. HMaster invoke the AssignmentManager to do region assignment.
2\. AssignmentManager checks the existing region assignments in .META.
3\. If region assignment is valid, then keep the region.
4\. If region assignment is invalid, then the LoadBalancerFactory will create a DefaultLoadBalancer
5\. DefaultLoadBalancer will assign the new region randomly to a RegionServer
6\. Update the assignment to .META.
7\. The RegionServer open the new region

### 4.6 Region Split

Region split is the work of RegionServer, not participated by HMaster. When a region in a RegionServer accumulates over size threshold, RegionServer will split the region into half.
1\. RegionServer offline the region to be split.
2\. RegionServer split the region into two half
3\. Update new daughter regions info to .META.
4\. Open the new daughter regions.
5\. Report the split info to HMaster.

### 4.7 Region Merge

A RegionServer can’t have too many regions, because too many regions bring large cost of memory and lower the performance of RegionServer. Meanwhile, HMaster can’t handle load balance with too many regions.
Therefore, when the number of regions is over a threshold, region merge is trigged. Region Merge is joined by RegionServer and HMaster.
1\. Client send RPC region merge request to HMaster
2\. HMaster moves regions together to the same RegionServer where the more heavily loaded region resided.
3\. HMaster send request to the RegionServer to run region merge.
4\. The RegionServer offline the regions to be merged.
5\. The RegionServer Merge the regions on the local file system.
6\. Delete the meta info of the merging regions on .META. table, add new meta info to .META. table.
7\. Open the new merged region
8\. Report the merge to HMaster

### 4.8 Auto Failover

#### 4.8.1 RegionServer Failover

When a region server fails, HMaster will do failover automatically.
1\. RegionServer is down
2\. HMaster will detect the unavailable RegionServer when there is no heartbeat report.
3\. HMaster start region reassignment, detect that the region assignment is invalid, then re-assign the regions like the region assignment sequence.

#### 4.8.2 Master Failover

In centralized architecture style, the SPOF is a problem, you may ask How does HBase will do when HMaster failed? There two safeguards for SPOF:
**1\. Multi-Master environment**
HBase can run in multi-master cluster. There is only one active master, when the master is down, the remaining master will take over the master role.
It looks like Hadoop HDFS new feature high availability, a standby NameNode will take over the NameNode role when the active one failed.
**2\. Catalog tables are on RegionServers**
When client send read/write request to HBase, it talks directly to RegionServer, not HMaster. Hence, the cluster can be steady for a time without HMaster. But it requires that the HMaster failover as soon as possible, HMaster is the vital of the cluster.

#### 4.9 Data Replication

For data reliability, HBase replicates data blocks across the cluster. This is achieved by Hadoop HDFS.
By default, there 3 replicas:
1\. The First replica is written to local node.
2\. The second replica is written to a random node on a different rack.
3\. The third replica is on a random node on the same rack
HBase stores HFiles and WAL on HDFS. When RegionServer is down, new region assignment can be done by read replicas of HFile and replay the replicas the WAL. In short, HBase has high reliability.