title: I look down on binary search algorithm
date: 2017-02-21 09:29:49
tags: Algorithm
---


One day, I wanted to use binary search in one of my feature in my project. My friend said the algorithm was not easy to implement bug free. I did't believe that. I spent 10min to write it.

<!--more-->

```scala
 def search(list: Array[Int], start: Int, end: Int, x: Int): Option[Int] = {
    if (start <= end) {
      val middle = (end - start) / 2
     if (list(middle) == x) {
        Some(middle)
      } else if (list(middle) > x) {
        search(list, middle + 1, end, x)
      } else {
        search(list, start, middle - 1, x)
      }
    } else {
      None
    }
  }

  def main(args: Array[String]): Unit = {
    val list = Array(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
    println(search(list, 0, list.size - 1, 5))
  }
```

Ooh, definitly my code has bug, yes I admitted that it was not very easy to implement binary search bug free.
I revised it.

```scala
def search(list: Array[Int], start: Int, end: Int, x: Int): Option[Int] = {
    if (start <= end) {
      val middle = (end - start) / 2 + start // bug 1, without plus start
      if (list(middle) == x) {
        Some(middle)
      } else if (list(middle) > x) {   // bug2, when middle bigger than x, not search middle+1,end
        search(list, start, middle - 1, x)
      } else {
        search(list, middle + 1, end, x)
      }
    } else {
      None
    }
  }

  def main(args: Array[String]): Unit = {
    val list = Array(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
    println(search(list, 0, list.size - 1, 5))
    println(search(list, 0, list.size - 1, 11))
    println(search(list, 0, list.size - 1, 8))
  }

```

```
output:
Some(4)
None
Some(7)
```

It was an interesting problem. I should tain my programming skills more.