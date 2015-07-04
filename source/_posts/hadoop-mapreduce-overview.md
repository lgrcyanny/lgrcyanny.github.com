title: Hadoop MapReduce Overview
id: 536
categories:
  - Hadoop
date: 2013-12-05 15:20:25
tags:
---

### MapReduce Introduction

**MapReduce is a parallel programming model and an associated implementation for processing and generating large data sets. The MapReduce model consists of two phrases: map and reduce.** A map task is to process a key/value pair to generate a set of intermediate key/value pairs, and a reduce task is to merge all intermediate values associated with the same intermediated key.[more...]

Hadoop MapReduce is based on the MapReduce paper in 2006\. This processing model is automatic parallelization and distribution. It provides a clean abstraction for programmers. MapReduce programs are usually written in java, and can be written in any scripting language like Ruby, Python, PHP using Hadoop Streaming, or in C++ using Hadoop Pipes. MapReduce abstracts all the ‘housekeeping’ away from the developer. Developer can concentrate simply on writing the Map and Reduce functions.

### MapReduce Data Flow

[![hd6.1](http://cyanny/myblog/wp-content/uploads/2013/11/hd6.1.png)Figure1 MapReduce Data Flow](http://cyanny/myblog/wp-content/uploads/2013/11/hd6.1.png)
As shown in Figure1, A MapReduce process consists of two phrases: map phrase and the reduce phrase. Let’s consider them in detail.

#### The Mapper

A MapReduce job is a unit of work that the client wants to be run, including the input data, the MapReduce program and the configuration information. Hadoop runs the job by dividing it into tasks: map tasks and reduce tasks.
Hadoop divides the input data into fixed-size pieces call input splits or splits. Each map task runs on each split, which runs the user-defined map function. All of the map tasks runs in parallel. Usually, the split size is a HDFS block, 64MB or 128MB. 
**If the file is less than 64MB or 128MB, it will not be split. And the file will occupy one block, results in a waste of storage.**
Map tasks usually runs on its local HDFS data, or the data near the node that runs the map task. Data Locality saves bandwidth and decreases dependencies. 
The input value for map task is key/value pair. For example, in the WordCount example by Hadoop, the input value for map task: the key is the line offset whining the file, which we can ignore in our map function, the value is the line in the file.
In the map function, developers will process the value of each line, make sure the output is key/value pair, WorkCount again, the output for map function is like ‘<apple, 1>, <pear, 1>…’, key is the word, value is 1.
**Map tasks output is written to the local disk, not to HDFS, then the reduce task will use these intermediate output to do merge work.** Because storing these intermediate data to HDFS with replication would be overkill. And if the map task fails before the reduce task consume the output, Hadoop will automatically start another map task on another node that will re-create the output. 

#### The Reducer

After map tasks done, the job tracker will start the reduce task. The reducer input is the intermediate mapper output.
Between the map task and the reduce task is the well known shuffle and sort. Hadoop will sort the intermediate map output by key. And each reduce task will run on map output with the same key. In WordCount example, the same key means the same word, like ‘<apple, 1> … <apple, 1>’, will be assigned to a reduce task, the reduce function just sum up the value and calculate the word count for ‘apple’.
Reduce tasks don’t have the advantage of data locality, the sorted map output have to be transferred across the network to the node where the reduce task is running.
Then the reduce task will merge the data with the user-defined reduce function. The reduce output is normally stored in HDFS for reliability. For each reduce output block, the first replica is stored on the local node, while the other two replicas are stored on off-rack nodes.
**We can see that no reduce task can start until every map task has finished. Will the mapper become a bottleneck? Hadoop uses the ‘Speculative execution’ to mitigate against this:**
-  If a Mapper appears to be running significantly more slowly than the others, a new instance of the Mapper will be started on another node, operating on the same data.
-  The results of the first Mapper to finish will be used
-  Hadoop will kill off the Mapper which is still running

#### The Combiner

Often, Mappers produce large amounts of intermediate data, which have to be transferred to Reducers that will result in a lot of network traffic.
To minimize the data transferred between Mapper and Reducer, Hadoop introduces the combiner function to be run on the map output, and combine the Mapper output and generate the Reducer Input.
Combiner is like a ‘Mini-Reducer’, runs locally on a the same node as Mapper. The output from the combiner is sent to the Reducers. Combiners decrease the amount of the network traffic required during the shuffle and sort phase. 

Resources:
[Part 0 Hadoop Overview](http://cyanny/myblog/2013/12/05/hadoop-overview/ "Hadoop Overview")
[Part 1 Hadoop HDFS Review](http://cyanny/myblog/2013/12/05/hadoop-hdfs-review/ "Hadoop HDFS Review")
[Part 2 Hadoop HDFS Federation](http://cyanny/myblog/2013/12/05/hadoop-hdfs-federation/ "Hadoop HDFS Federation")
[Part 3 Hadoop HDFS High Availability(HA)](http://cyanny/myblog/2013/12/05/hadoop-hdfs-high-availability/ "Hadoop HDFS High Availability(HA)")
[Part 4 Hadoop MapReduce Overview](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-overview/ "Hadoop MapReduce Overview")
[Part 5 Hadoop MapReduce 1 Framework](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-1-framework/ "Hadoop MapReduce 1 Framework")
[Part 6 Hadoop MapReduce 2 (YARN)](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-2-yarn/ "Hadoop MapReduce 2 (YARN)")
[Part 7 Hadoop isn’t Silver Bullet](http://cyanny/myblog/2013/12/05/hadoop-isnt-silver-bullet/ "Hadoop isn’t Silver Bullet")