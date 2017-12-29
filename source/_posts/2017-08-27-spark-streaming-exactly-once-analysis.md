title: spark streaming exactly-once analysis
date: 2017-08-27 11:31:58
tags: spark streaming
---

最近对Spark Streaming接触比较多，主要关注的是streaming的准确性方面的需求, 忙了快半年，不禁想问为什么需要在exactly-once上花费这么多时间呢。streaming和batch的处理逻辑有什么区别呢？我觉得streaming更适合一些简单的过滤，能在100ms以内能算完的逻辑，而这些逻辑用batch也可以算完，为什么要streaming呢？用户们更希望的是更快。如果batch也能满足低延迟的需求，streaming系统就不需要了。而问题是为什么我们需要一个单独的streaming系统？
<!--more-->

 生产环境中的版本是1.6，spark streaming的API在1.6上是基于RDD的DStream API，相比Structured Streaming，更稳定和成熟些。而我们的用户们，比较关心的是streaming系统
 
 开源里广泛使用的Streaming系统是Storm和Flink。Storm早期用record ack的方式保证at-least once，但没有提供exactly-once的保证，后面又有了storm trident.

## Spark Streaming Receiver模式没有exactly-once保证
 
 
## Flink中的Exactly-Once保证  

## Storm的Eactly-Once保证
### Storm Architecture
![storm architecture](http://wx3.sinaimg.cn/mw690/761b7938ly1fizgwunfr6j21kw0n7e6n.jpg)
以Storm on Yarn来说明Storm的架构：

+ client将jar包通过yarn上传
+ 在一台NodeManager上启动Nimbus，这是master节点，负责管理StormTopology, 分发task，心跳等
+ 其他的NodeManager上启动Supervisor, 相当于slave节点，管理storm worker
	+  在每个supervisor上，可以启动多个worker进程，每个worker进程可以运行多个task，task是多线程的，由worker管理。这些task运行的就是Spout或Bolt定义的操作
+  Zookeeper, Storm运行时状态的管理

Storm方面算是简单调研，理解不是很深入，具体的参考[官方文档](http://storm.apache.org/releases/1.1.1/Setting-up-a-Storm-cluster.html)

### Storm exactly-once

####1. Storm Transactional Topologies(deprecated)
Strom 0.7版本中，实现了[transctional topologies](http://storm.apache.org/releases/1.1.1/Transactional-topologies.html)来保证exactly-once, 
**1.transactional phrases 类似两阶段事务机制**

+ The processing phase: this is the phase that can be done in parallel for many batches 可以并发执行计算partial result
+ The commit phase: The commit phases for batches are strongly ordered. So the commit for batch 2 is not done until the commit for batch 1 has been successful. 保证batch的提交是按顺序的

来个直观的，用户需要构建的Topology如下:

```java
MemoryTransactionalSpout spout = new MemoryTransactionalSpout(DATA, new Fields("word"), PARTITION_TAKE_PER_BATCH);
TransactionalTopologyBuilder builder = new TransactionalTopologyBuilder("global-count", "spout", spout, 3);
builder.setBolt("partial-count", new BatchCount(), 5)
        .shuffleGrouping("spout");
builder.setBolt("sum", new UpdateGlobalCount())
        .globalGrouping("partial-count");
```

**2.关键点**

+ 一个拓扑里只有一个Transactional Spout，其实现是由一个单线程的Coordinator Spout + 多个Emitter Bolt组成。利用Storm的ACK Framework机制，判断一个batch是否执行完成
+ Committer Bolt 可以有多个，需要收到Transactional Spout的commit信息才会执行commit


####2. Storm Trident Topologies
storm 1.1的版本中，引入新的Trident API解决exactly-once，这是transactional topologies的升级版本。API的易用性改善，exactly-once也是采用事务机制

**1.exactly once**

* Tuples are processed as small batches 采用micor-batch机制
* Each batch of tuples is given a unique id called the "transaction id" (txid). If the batch is replayed, it is given the exact same txid. 每个batch有唯一的batchid
* State updates are ordered among batches. That is, the state updates for batch 3 won't be applied until the state updates for batch 2 have succeeded. 按txid顺序提交


**2.trident example**
[github example](https://github.com/lgrcyanny/LearningStorm/blob/master/src/main/scala/com/learning/storm/TridentTest.scala)

```scala
  val topology = new TridentTopology()
  // define spout
  val spout = new FixedBatchSpout(new Fields("sentence"), 3,
      new Values("the cow jumped over the moon"),
      new Values("the man went to the store and bought some candy"),
      new Values("four score and seven years ago"),
      new Values("how many apples can you eat"))
    spout.setCycle(true)

  val wordsCount: TridentState = topology.newStream("wordsSpout", spout)
      .each(new Fields("sentence"), new Split(), new Fields("word"))
      .groupBy(new Fields("word"))
      .persistentAggregate(new MemoryMapState.Factory(), new Count(), new Fields("count"))
      .parallelismHint(6)
```

###**总结**

Storm里为了exacly once，需要做到：

+ 源端可重放
+ batch要有唯一的txid
+ commit时按顺序提交，类似事务的两阶段提交

<hr/>







