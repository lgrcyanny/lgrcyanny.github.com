title: Eight Queens Problem In Scala
date: 2016-09-27 09:15:28
tags:
    - Scala
    - Algorithm
---

I have dedicated in **Programming in Scala** for about 4 months. My work is busy, but I can't give up reading more books.
Scala is a fabulous language, both object oriented and functional.
Eight qeens problem can be expressed in scala easily and concise.
<!--more-->

## Eight Queens Problem
Given a standard chess-board, place eight queens such that no queen is in check from any other (a queen can check another piece if they are on the same column, row, or diagonal)

## Solutions

The problem in scala is recursively.

1. First each solution is a List[(Row, Column)]

		- Each element is a coordinated, the queen position in each row
		- The coordicate for row k comes first, followed by row `k-1`, `k-2`, ... 0

2. Use a Set[List[Row, Column]], represent all solutions
3. To place next `k+1` qeen, we iterate all solutions, if match the condition, yield another list


## Lets run the code
```scala
/**
    * the coordinates of the queen in row k comes first in each List[(ROW, Column)], followed
    * by k -1, k - 2, ..., 0 and so on
    *
    * @param n
    * @return
    */
  def queens(n: Int): Set[List[(Row, Column)]] = {
    def placeQueen(k: Int): Set[List[(Row, Column)]] = {
      if (k < 0) {
        Set(List())
      } else {
        for {
          queens <- placeQueen(k - 1)
          column <- 0 until n
          queen = (k, column)
          if isSafe(queen, queens)
        } yield queen :: queens
      }

    }

    def isSafe(queen: (Row, Column), queens: List[(Row, Column)]): Boolean = {
      queens.forall { placedQueen =>
        placedQueen._1 != queen._1 &&
          placedQueen._2 != queen._2 &&
          (Math.abs(placedQueen._1 - queen._1) !=
            Math.abs(placedQueen._2 - queen._2))
      }
    }
    placeQueen(n - 1)
  }
```

## Github Link

[Eight Queens](https://github.com/lgrcyanny/ScalaPractice/blob/master/ProgrammingInScala/src/main/scala/com/chapter23/EightQueens.scala)

```shell
	git clone https://github.com/lgrcyanny/ScalaPractice.git

	mvn clean package

	cd ProgrammingInScala

	scala -cp target/programming-in-scala-1.0-SNAPSHOT.jar com.chapter23.EightQueens
```


