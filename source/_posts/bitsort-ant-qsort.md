title: 位排序和快排
tags:
  - algorithm
id: 723
categories:
  - Algorithm
date: 2015-05-04 21:57:53
---

这已经不是一个新话题了，但是从开始实现一个C++位排序和快排，我还是花费了2个多小时，这里就记下自己的一点点体会啦。

首先，题目来自《编程珠玑》第一章，主要是做位排序，同时和快排做比较：

1\. 实现位逻辑运算，实现位向量，并用该位向量实现1,000,000个数字的排序，数字最大是10,000,000

2\. 实现1000,000个数字的快排序

3\. 实现生成小于n且没有重复的k个整数，这里n=10,000,000, k = 1,000,000

<!--more-->

### 1\. 位排序

[cpp]
#include &lt;iostream&gt;
#include &lt;string&gt;
#include &lt;fstream&gt;

// 这里以32位的int来实现位向量，初始化的开销可能会比较大
class BitVector {
public:
    BitVector(int size) {
        int n = (1 + (size / BITS_PER_WORD));
        buffer = new int[n];
        for (int i = 0; i &lt; n; ++i) {
            buffer[i] = 0;
        }
    }
    ~BitVector() {
        delete[] buffer;
    }
    void set(int i) {
        buffer[(i &gt;&gt; SHIFT)] |= 1 &lt;&lt; (i &amp; MASK);
    }
    void clr(int i) {
        buffer[(i &gt;&gt; SHIFT)] &amp;= ~(1 &lt;&lt; (i &amp; MASK));
    }
    int test(int i) {
        return buffer[(i &gt;&gt; SHIFT)] &amp; (1 &lt;&lt; (i &amp; MASK));
    }

private:
    int* buffer;
    static const int BITS_PER_WORD = 32;
    static const int MASK = 0x1f;
    static const int SHIFT = 5;
};

const int BitVector::BITS_PER_WORD;
const int BitVector::MASK;
const int BitVector::SHIFT;

class Solution {
public:
    // 输入是要排序的文件名，同时需要给出输出文件名称
    // range是数字的范围
    void sort(
            int range,
            const std::string&amp; input_file_name,
            const std::string&amp; output_file_name) {
        BitVector bits_vector(range);
        std::ifstream fin(input_file_name);
        int i = 0;
        while (fin &gt;&gt; i) {
            bits_vector.set(i);
        }
        std::ofstream fout(output_file_name);
        for (int i = 0; i &lt; range; ++i) {
            if (bits_vector.test(i)) {
                fout &lt;&lt; i &lt;&lt; std::endl;
            }
        }
    }
};

int main(int argc, char const *argv[]) {
    Solution s;
    int range = 10000000;
    s.sort(range, &quot;numbers_input.txt&quot;, &quot;numbers_output.txt&quot;);
}
[/cpp]

### 2\. Quick Sort

[cpp]
class Solution {
public:
    void sort(
            const std::string&amp; input_file_name,
            const std::string&amp; output_file_name) {
        std::vector&lt;int&gt; arr;
        int number = 0;
        std::ifstream fin(input_file_name);
        while(fin &gt;&gt; number) {
            arr.push_back(number);
        }
        qsort(&amp;arr, 0, arr.size() - 1);
        std::ofstream fout(output_file_name);
        for (int i = 0; i &lt; arr.size(); ++i) {
            fout &lt;&lt; arr[i] &lt;&lt; std::endl;
        }
    }

private:
    void qsort(std::vector&lt;int&gt;* arr, int p, int r) {
        if (p &gt;= r) {
            return;
        }
        int q = partition(arr, p, r);
        qsort(arr, p, q - 1);
        qsort(arr, q + 1, r);
    }

    int partition(std::vector&lt;int&gt;* arr, int p, int r) {
        int i = p;
        int j = i - 1;
        for (; i &lt; r; ++i) {
            if (arr-&gt;at(i) &lt; arr-&gt;at(r)) {
                ++j;
                std::swap(arr-&gt;at(i) , arr-&gt;at(j));
            }
        }
        ++j;
        std::swap(arr-&gt;at(j) , arr-&gt;at(r));
        return j;
    }
};
[/cpp]

### 3\. 生成范围是[0,n)的k个随机数

该算法首先生成顺序的n个数字，再遍历该数组，将当前索引i和 random(i, n)生成的索引对应的数字交换，保证生成不重复的随机序列，生成1000000个范围为10000000的数字5.3s可以完成

[python]
import sys
import random
def swap(list, i, j):
    tmp = list[i]
    list[i] = list[j]
    list[j] = tmp

def generate():
    number_limit = int(sys.argv[1])
    n = int(sys.argv[2])
    list = []
    for i in range(number_limit):
        list.append(i)
    f = open('numbers_input.txt', 'w')
    for i in range(n):
        random_index = random.randint(i, number_limit - 1)
        swap(list, i, random_index)
        f.write(str(list[i]) + '\n')
    f.close()
generate()
[/python]

### 4.性能比较

运行的环境是Mac OSX 10.10，内存DDR3 4G，1.8GHz Inter Core i5
1\. 位排序总运行时间是: 3.98s, 去掉读写文件的时间，0.47s完成排序
2\. 快排序总运行时间: 4.71s, 去掉读写文件时间，0.63s完成排序
总之，快排的优势在于不限制排序数字范围，位排序限定范围，性能比快排高效。