title: 分布式系统总结part4
tags:
  - Learning
id: 435
categories:
  - Distributed System
  - Life
date: 2013-11-24 15:04:39
---

### Petri网

#### 1\. 以生产者消费者为例子

Petri三元组PN=(P,T,F)即，库所，变迁，转换弧

Preset表示前集,如果x是变迁，那么preset表示某个变迁的输入库所集合，如果x是某个库所那么preset表示某个库所的输入变迁。

Postset表示后集，如果x是变迁，那么postset表示某个变迁的输出库所集合，如果x是库所，那么postset表示某个库所的输出变迁集合。[more...]
[![Distributed System14](http://cyanny/myblog/wp-content/uploads/2013/11/Distributed-System14-580x398.png)](http://cyanny/myblog/wp-content/uploads/2013/11/Distributed-System14.png)

#### 2\. Marking， Enable， Firing

M(p)表示当前所有库所的Token数量, 是一个快照.
[![Distributed System16](http://cyanny/myblog/wp-content/uploads/2013/11/Distributed-System16-580x290.png)](http://cyanny/myblog/wp-content/uploads/2013/11/Distributed-System16.png)
通过这个图，可以很简单地看出 M(p)的计算等式.
当库所的token都到位后，变迁就enable了，如上图左边t2是enable的, t1没有enable， 如果Firing点火，就会转到另一个Marking下，如图中右边的图是点火后的Marking。

### 3.Handle

PP—handled : 某一个Place到另一个Place有两条不相交的路径

TT handled：某一个Transition和另一个Transition有两条不相交的路径

PT—handled：某一个Place到另一个Transition有两条不相交的路径

TP—handled：某一个Transition到另一个Place有两条不相交的路径

Well-handled：只有TT和PP，没有PT，TP。 因为PT会有死锁的风险， TP会造出某一个Place内部的Token无限增长

### 4.State Machine和Marked Graph

[![Distributed System17](http://cyanny/myblog/wp-content/uploads/2013/11/Distributed-System17-580x349.png)](http://cyanny/myblog/wp-content/uploads/2013/11/Distributed-System17.png)

*   Marked Graph：库所只有单个输入和单个输出
*   State Machine: 每个变迁只有单个输入和单个输出，这是一个有限状态机
*   T-component：如果一个大的Petri可以找到一个子网符合marked graph的要求，就是 T-component, 如果所有的T-component的集合可以覆盖整个Petri网，就是T-coverability。
*   P-component：是一个大的网络中的一个Petri子网，满足state machine的，如所有的P-component可以覆盖整个Petri网，则称Petri网P-coverability

### 5\. 活性和有界性

*   活性：从M0状态出发，对于任何一个M和t转换后的M'，可以让任何变迁T有可能点火enable.什么是不活呢，就是死锁，但是不死锁，不等价于活性
*   有界性：不论状态如何变化，token数量都不会无限增长，保持在一定数量
*   Petri是well-formed的：存在M0使这个Petri网有活性并且有界

### 6\. Free-choice structure

自由选择结构：如果两个输入库所交集不为空，蕴含两个输入库所相等

t1和t2是对等的选择关系.

自由选择结构可以表达并发关系和同步关系

给定一个自由选择结构，可以很好地决定它的活性和有界性。

### 7\. 哲学家吃通心粉

有五个哲学家围坐在一圆桌旁，桌子中央有一盘通心面，每人面前有一只空盘子，每两人之间放一把叉子。每个哲学家思考、饥饿、然后，欲吃通心面。为了吃面，每个哲学家必须获得两把叉子，且每人只能直接从自己左边或右边去取叉子。

[![Distributed System15](http://cyanny/myblog/wp-content/uploads/2013/11/Distributed-System15-580x355.png)](http://cyanny/myblog/wp-content/uploads/2013/11/Distributed-System15.png)

### 资源

[分布式系统总结part1 中间件，进程迁移，移动通信失效，名称解析，移动实体定位](http://cyanny/myblog/2013/11/24/distributed-system-review-part1/ "分布式系统总结part1")

[分布式系统总结part2 Lamport同步与向量时间戳，两大选举算法，三大互斥算法](http://cyanny/myblog/2013/11/24/distributed-system-review-part2/ "分布式系统总结part2")

[分布式系统总结part3 复制和一致性(以数据和以客户为中心的一致性)，容错（拜占庭将军问题，两阶段与三阶段提交）](http://cyanny/myblog/2013/11/24/distributed-system-review-part3/ "分布式系统总结part3")

[分布式系统总结part4 Petri网解决哲学家问题和生产者、消费者问题](http://cyanny/myblog/2013/11/24/distributed-system-review-part4/ "分布式系统总结part4")