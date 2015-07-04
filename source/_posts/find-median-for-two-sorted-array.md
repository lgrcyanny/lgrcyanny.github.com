title: 查找两个排序数组的中位数
tags:
  - algorithm
id: 705
categories:
  - Algorithm
date: 2015-04-03 22:07:01
---

题目是：求两个排序数组的中位数。
设：两个排序的数组a[m], b[n]，求a和b数组的中位数。
算法是：
mid_a是数组a的中位数index，同理mid_b是数组b的中位数索引
1\. 如果a[mid_a] == b[mid_b] 中位数为a[mid_a]

2\. 如果a[mid_a] &lt; b[mid_b], 递归查找a(mid_a + 1, m - 1), b(0, mid_b)，因为a[mid_a]比较小，不可能作为下一次查询的中位数

3\. 如果a[mid_a] &gt; b[mid_b], 递归查找a(0, mid_a), b(mid_b + 1, n - 1)，因为b[mid_b]比较小，不可能作为下一次查询的中位数

4\. 当只少于4个元素需要查找时递归停止，merge这少于4的元素，求出中位数。这里需要考虑奇偶数的情况，只剩下2个或3个元素，不如只考虑4个简单。

注意：这里求上中位数，当n为奇数时，中位数是唯一的，出现位置为n/2；当n为偶数时候，存在两个中位数，数组index从0开始，位置分别为n/2 - 1（上中位数）和n/2（下中位数）。

<!--more-->

[cpp]
#include &lt;iostream&gt;
using namespace std;

class Solution {
public:
  int find_median(int a[], int p, int q, int b[], int r, int s) {
    int size_a = q - p + 1;
    int size_b = s - r + 1;
    int total_size = size_a + size_b;
    if (total_size &lt;= 4) {
      // calcuate median for less than 4 elements
      int i = p;
      int j = r;
      int num = (total_size + 1) / 2;
      for (int k = 1; k &lt; num; k++) {
        // A fake merge without copying, just move index
        if (i &lt;= q &amp;&amp; j &lt;= s) {
          if (a[i] &lt;= b[j]) {
            ++i;
          } else {
            ++j;
          }
        } else if (i &gt; q) {
          ++j;
        } else if (j &gt; s) {
          ++i;
        }
      }
      int median = 0;
      if (i &lt;= q &amp;&amp; j &lt;= s) {
        median = a[i] &lt; b[j] ? a[i] : b[j];
      } else if (i &gt; q){
        median = b[j];
      } else if (j &gt; s) {
        median = a[i];
      }
      return median;
    }
    int mid_a = get_median_index(a, p, q);
    int mid_b = get_median_index(b, r, s);
    if (a[mid_a] == b[mid_b]) {
      return a[mid_a];
    } else if (a[mid_a] &lt; b[mid_b]) {
      // mid_a is less, no need to be seached later, no possible to be median
      return find_median(a, mid_a + 1, q, b, r, mid_b);
    } else {
      // mid_b is less, no need to be seached later, no possible to be median
      return find_median(a, p, mid_a, b, mid_b + 1, s);
    }
  }

private:
  int get_median_index(int a[], int p, int q) {
    int n = q - p + 1;
    int mid = n / 2;
    if (n % 2 == 0) {
      return p + mid - 1;
    } else {
      return p + mid;
    }
  }
};

int main(int argc, char const *argv[])
{
  int a[] = {5, 6, 7, 8};
  int m = sizeof(a) / sizeof(int);
  int b[] = {1, 2, 3, 4};
  int n = sizeof(b) / sizeof(int);
  Solution s;
  int median = s.find_median(a, 0, m - 1, b, 0, n - 1);
  cout &lt;&lt; median &lt;&lt; endl;
  return 0;
}
[/cpp]