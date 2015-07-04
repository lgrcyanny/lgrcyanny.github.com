title: "Warshall's Algorithm for Cycle Dectection"
tags:
  - algorithm
  - Learning
id: 239
categories:
  - Algorithm
date: 2013-09-09 21:03:54
---

今天上系统分析与设计课，老师提到了Warshall算法做函数调用关系的Cycle Detection, 想起当年的离散，这一章还真没有什么印象了。翻开陈年旧书看看，就跟看见陌生人一样，基础还是需要在加强一下啦。今天上课提到的几个点，比如死锁，Dijkstra Algorithm，DFS，Unix多级反馈调度，Android卡顿缘由等，自己没能第一时间想出来，就留作以后闲暇时光google的题目吧。现在花了一个小时，用Java把Warshall算法实现了，总觉得用一个小时是不是有点长了。

### Warshall算法要义：

1.  邻接矩阵: 表示图关系的矩阵，看看离散数学就懂了

2.  传递闭包：从邻接矩阵的每一个顶点出发，求出的所有顶点的到达情况，该矩阵就是传递闭包。说是闭包是因为它满足自反性、对称性和传递性。

3.  Warshall算法就是从邻接矩阵中求出传递闭包。[more...]

    例如:

    邻接矩阵为：

    [0, 1, 0, 0,

    1, 0, 1, 0,

    0, 0, 0, 1,

    0, 0, 0, 0]

    传递闭包为：

    [1, 1, 1, 1

    1, 1, 1, 1,

    0, 0, 0, 1,

    0, 0, 0, 0]

    算法很简单，直接上一个代码描述：
[java]
public void warshallDetect(int[] matrix) {
closure = matrix;
for (int k = 0; k &lt; n; k++) {
  for (int i = 0; i &lt; n; i++) {
    for (int j = 0; j &lt; n; j++) {
      if (closure[i, k] == 1 &amp;&amp; closure[k, j] == 1) {
        closure[i, j] = 1
      }
    }
  }
}
}
[/java]
程序的意思是：
M(0): 初始矩阵，从每一点到另一个点不能有中间节点，即最初的矩阵。

    M(1): 在M(0)基础上，从每一个顶点出发，让第1个顶点作为中间节点，检测是否有关系。

    M(2): 在M(1)基础上，从每一个顶点出发，让第2个顶点作为中间节点，检测是否有关系。

    M(k): 在M(k-1)基础上，从每一个顶点出发，让第k个顶点作为中间节点，检测是否有关系。

    M(n): 在M(n - 1)基础上，从每一个顶点出发，让第n个顶点作为中间节点，检测是否有关系。

    最终生成一个传递闭包，检测哪些顶点有循环调用就显而易见了。

    [源代码在此。](https://github.com/lgrcyanny/Algorithm/blob/master/src/com/algorithm/warshall/CycleDetection.java "Warshall Cycle Detection Source Code")

    [参考博客](http://www.cnblogs.com/lpshou/archive/2012/04/27/2473109.html "参考博客")