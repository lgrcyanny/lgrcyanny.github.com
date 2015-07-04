title: Hadoop isn’t Silver Bullet
tags:
  - Hadoop
  - Learning
  - Research
id: 547
categories:
  - Hadoop
date: 2013-12-05 18:28:45
---

Hadoop is a great framework for distributed large data computing. But Hadoop is not the silver bullet. Hadoop fits not very well in such cases as follow:

#### 1\. Low-latency Data Access

Applications that require real-time query, and low-latency access to data in tens of milliseconds will not work well with Hadoop. 
Hadoop is not a substitute for a database. Database index records that will gains low-latency and fast response. 
But if you really want to replace the database for real time needs, try HBase, which is a column-oriented database for random and real time read/write.[more...]

#### 2\. Structured Data

Hadoop is not fit for structured data with strong relationship. Hadoop works well for semi-structured and unstructured data. It stores data in files, doesn’t index them like RDBMS. Therefore, each ad hoc query for Hadoop is processed by MapReduce job which will bring the latency cost. 

#### 3\. When data isn’t that big

How big the data is big enough for Hadoop? The answer is TB or PB. When your analytics data is only tens of GB, Hadoop is heavy. Don’t follow the fashion and use Hadoop, just follow your requirements. 

#### 4\. Too many small files

When there are too many small files, the NameNode will hit its memory limit where the block map and the metadata are hosted. And to handle the NameNode bottleneck, Hadoop introduces HDFS Federation.

#### 5\. Too many writers and too much file updates

HDFS is in write-once-and-read-many-times way. When there is too much files update needs, Hadoop won’t support that.

#### 6\. MapReduce may not the best choice

MapReduce is a simple programming model in parallel. But for MapReduce parallelism, you need to make sure each MR job and the data where the job runs on is independent from all the others. Every MR shouldn’t have dependencies. 
But if you want to do some data sharing during MR, you can do like this:
- Iteration: run multiple MR jobs, with the output of one being the input of the next MR.
- Shared state information. But don’t share information in memory, since each MR job is run on single JVM. 

Resources:
[Part 0 Hadoop Overview](http://cyanny/myblog/2013/12/05/hadoop-overview/ "Hadoop Overview")
[Part 1 Hadoop HDFS Review](http://cyanny/myblog/2013/12/05/hadoop-hdfs-review/ "Hadoop HDFS Review")
[Part 2 Hadoop HDFS Federation](http://cyanny/myblog/2013/12/05/hadoop-hdfs-federation/ "Hadoop HDFS Federation")
[Part 3 Hadoop HDFS High Availability(HA)](http://cyanny/myblog/2013/12/05/hadoop-hdfs-high-availability/ "Hadoop HDFS High Availability(HA)")
[Part 4 Hadoop MapReduce Overview](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-overview/ "Hadoop MapReduce Overview")
[Part 5 Hadoop MapReduce 1 Framework](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-1-framework/ "Hadoop MapReduce 1 Framework")
[Part 6 Hadoop MapReduce 2 (YARN)](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-2-yarn/ "Hadoop MapReduce 2 (YARN)")
[Part 7 Hadoop isn’t Silver Bullet](http://cyanny/myblog/2013/12/05/hadoop-isnt-silver-bullet/ "Hadoop isn’t Silver Bullet")