title: 再叙堆排序
tags:
  - algorithm
  - Learning
  - sort
id: 104
categories:
  - Algorithm
date: 2013-08-10 21:45:36
---

这个暑假，开始看《算法导论》，这是一本很厚的书，不过本着坚持就是胜利的原则，我选择一点点坚持，最近看到了堆排序。这是自己曾经靠背和记才慢慢理解的算法，而今重温，发现这个排序其实很简单，只需完成三点:

1\. 实现MaxHeapify函数, 也就是以递归或者循环的方式，让每一节点保持最大堆的性质。这是堆排序的核心函数，写好了，就万事大吉了。

2\. Build-Max-Heap, 利用MaxHeapify函数构建一个最大堆。

3\. HeapSort,实现堆排： 先构建最大堆，再循环将第一个元素和最后一个元素交换，即把最大的元素换到最后，heapSize减1，最后对第一个元素调用MaxHeapify函数，保持最大堆的性质。

看起来，堆排很好，最佳和最坏的复杂度都是O(n*lgn)，但实际应用中quicksort是一个比堆排更好的算法,速度比堆排更快。但堆的数据结构对实现优先级队列有很大意义。
[Source Code](https://github.com/lgrcyanny/Algorithm/blob/master/src/com/algorithm/heap/HeapSort.java "Source Code")