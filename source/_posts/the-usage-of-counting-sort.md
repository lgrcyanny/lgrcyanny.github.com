title: 计数排序的应用
tags:
  - algorithm
  - sort
id: 175
categories:
  - Algorithm
date: 2013-08-18 20:00:44
---

比较排序是我们常见的排序方法，如Insertion Sort, Merge Sort, Quick Sort, Heap Sort都有O(n*lgn)的下界，而如何让一个排序能在最短的线性时间O(n)内排序呢？就要求助于Counting Sort计数排序，该算法的基本思想是对数组中出现的数字进行计数，通过计数计算出每一个数字的排位，并按排位给出结果。计数排序明显的在时间上优于快排，以空间换时间，这个排序思想很像在编程珠玑的开篇中提到的按位排序，计数排序还有个好处是稳定的。

计数排序的应用主要是基数排序(Radix Sort),但是我试过排序一百万个数字，基数排序的时间虽然是O(n),但可能是常系数的因子太大，执行时间并不比快排理想，只能算是为了学习嘛，试验一下了。

虽然计数排序看着很快，万物都有利有弊，计数排序有一个前提，需要知道输入数据的分布，即最大值和最小值，如果不知道的话，还是快排最好。而基数排序则更适用于输入的数据位数都相同。

对于计数排序，其代码如下：[more...]
[java]
public void sort() {
  count = new int[k + 1];
  results = new int[data.length];
  int i;
  for (i = 0; i &lt; count.length; i++) {
    count[i] = 0;
  }
  // 对数组进行计数
  for (i = 0; i &lt; data.length; i++) {
    count[data[i]]++;
  }
  // 计算每一个数字的排名
  for (i = 1; i &lt; count.length; i++) {
    count[i] += count[i - 1];
  }
  // 按排名输出到结果数组中
  for (i = data.length - 1; i &gt;= 0; i--) {
    results[count[data[i]] - 1] = data[i];
    count[data[i]]--;
  }
}
[/java]

## 计数排序的应用一

**算法导论思考题8-3-a**

_给定一个整数数组，其中不同的整数中包含的数字个数可能不同，但该数组中，所有整数中的总的数字数为n。说明如何在O(n)的时间内对该数组排序。_

分析： 该数组的每个数字的位数不相同，直接用radix sort不好，需要做一些改进。其方法是用counting sort，按整数的位数给整数排序，再对相同位数的一段整数用基数排序，算法时间O(n)。

[Source Code](https://github.com/lgrcyanny/Algorithm/blob/master/src/com/algorithm/lineartimesort/VaringLengthNumberSort.java)

## 计数排序的应用二

**算法导论思考题8-3-b**

_给定一个字符串数组，其中不同的串包含的字符数可能不同，但所有的串中总的字符个数为n。说明如何在O(n)的时间内对该数组进行排序_

分析：该问题的难度出现在字符串上，字符串的排序不像数字，位数多的比位数少的大，字符串的排序需要按字典顺序来。这个算法的基本思想是先对所有的字符串按首字母进行Counting Sort, 然后再递归地对相同首字母的字符串的第二个字符进行Counting Sort，以此类推，递归排序第三，第四...个字符。

算法的边界问题容易出错，debug花了我两个小时，不过能编出来实属不易。

[Source Code](https://github.com/lgrcyanny/Algorithm/blob/master/src/com/algorithm/lineartimesort/VaringLengthStringSort.java)