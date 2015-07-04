title: 找一个数组中第2小的元素
tags:
  - algorithm
  - Learning
id: 250
categories:
  - Algorithm
date: 2013-09-12 20:09:43
---

找出一个数组中的最小元素或者最大元素，遍历是最优解,即O(n)，我想这是会编程的人都会的简单算法。而如何找出第二小的元素呢，先排序，然后再选么，这样的算法最优也是O(n * lg n)，还是一次遍历来的更快，思想如下：

假定数组A有n个元素,元素可以有重复：

1\. 初始化一个数组res[2]，变量i, i为遍历时使用。

2\. 当n为奇数时, res[0] = A[0], res[1] = A[1], i = 1;

3\. 当n为偶数时, res[0] = min {A[0], A[1]}, res[1] = max{A[0], A[1]}, i = 2;

4\. 接下来每次取数组A中的两个元素，较小的元素如果小于res[0]比较，则res[1] = res[0], 再赋值 res[0] = small；

5\. 如果res[0] == res[1]或者较大的元素小于res[1], 则res[1] = large;[more...]

以上算法可以O(3n/2 - 2)时间，即O(n)的时间内选出第二小的元素。

[Github Source Code](https://github.com/lgrcyanny/Algorithm/blob/master/src/com/algorithm/orderstatistics/SecondSmallest.java)
[java]
public int selectSecondSmallest() {
    int[] res = new int[2];
    int n = data.length;
    int i = 2;
    if (n % 2 == 0) {
        i = 2;
        if (data[0] &lt; data[1]) {
            res[0] = data[0];
            res[1] = data[1];
        } else {
            res[0] = data[1];
            res[1] = data[0];
        }
    } else {
        i = 1;
        res[0] = data[0];
        res[1] = data[1];
    }
    for ( ;i &lt; n; i += 2) {
        int large = getLarge(data[i], data[i + 1]);
        int small = getSmall(data[i], data[i + 1]);
        if (small &lt; res[0]) {
            res[1] = res[0]; // Maybe res[0] is the second minimum, don't forget the tiny step
            res[0] = small;
        } else if (small &lt; res[1]) {
            res[1] = small;
            continue;
        }
        if (large &lt; res[1] || res[0] == res[1]) {
            res[1] = large;
        }
    }
    System.out.println(&quot;The smallest is &quot; + res[0]);
    System.out.println(&quot;The second minimun is &quot; + res[1]);
    return res[1];
}
[/java]

### 后记

这个算法很简单，但是算法导论的答案给的太复杂了，[用了树的算法](http://blog.csdn.net/mishifangxiangdefeng/article/details/7687460)，这么诡异而复杂，算是领教了。这个算法可以扩展到找第i个小的元素。