title: Programming In Scala Overview Key Note
date: 2016-03-06 18:01:42
tags:
    - Language
    - Scala
---

过去半年里，都在忙着spark相关的项目，主要的编程语言是scala，前些年主要用的是c++，刚转到scala上时，有点不适应函数式编程语言的思想，现在已经半年多过去了，觉得scala真的是awesome，简洁有力的表达，整合了functional programming和object programming，并且设计良好。
去年上了Martin Odersky的编程课<i>Functional Programming In Scala</i>, 觉得挺感兴趣，学习了很多，想在scala语言上看得更多，更深入些，我选择了Scala的权威著作[Programming In Scala](http://www.artima.com/shop/programming_in_scala_2ed), 虽然这书有快900页，但我想挑战下，与其每天泡在各种新闻资讯里，还不如看看书泡在书里。每天都看看书，觉得挺好，有些笔记和心得就写在博客里，毕竟工作忙，写博客的时间很少，但我想还是应该多总结和留下写想法。之后阅读Spark的源码，我也会坚持写下些东西，看过了和写下来总还是不一样嘛。
<!--more-->

# Scala Overview

## 1. Scala is a Scalable Language

- Scala is a blend of object-oriented and functional programing language
- Grow new types, such as BigInt
- Grow new control structures, such as actor based api

## 2. What makes scala scalable
fusion object-oriented and functional programming

- **Object-Oriented**
    - it combines data with operations under a formalized interface. So objects have a lot to do with language scalability: the same techniques apply to the construction of small as well as large program
    - Scala is an object-oriented language in pure form: every value is an object and every operation is a method call.
    -  An example is Scala’s traits. Traits are like interfaces in Java, but they can also have method implementations and even fields
- **Functional**
    - Functional programming fundation was raid in lonzo Church’s lambda calculus, in the 1930. The first functional programming language is Lisp, created in the late 1950s, other functional programming languages are Scheme, SML, Erlang, Haskell, OCaml, and F#.
    - Functional programming two main ideas:
        - Firstly, **functional are first class values**
            - You can pass functions as arguments to other functions, return them as results from functions, or store them in variables. You can also define a function inside another function, just as you can define an integer value inside a function
            - Functions that are first-class values provide a convenient means for abstracting over operations and creating new control structures.
        - Secondly, **operations of a program should map input values to output values rather than change data in place.**
            - methods should not have any side effects
                -  **Referentially transparent**, which means that for any given input the method call could be replaced by its result without affecting the program’s semantics
            - encourage immutable data structures and referentially transparent methods

## 3. Why chooose scala?

- **Compatibility**
    - compile to JVM bytecode, run on jvm
    - compatible with java types, reuse java types
    - implicity conversions : support string.toInt,
    - java can call scala code
- **Brevity, scala is concise**
    - reduction on list
    - Avoid biolerplate in java, such as avoid class getter and setter, default constructor
    - Type inference
    - tools in library, can be used as trait
- **High-level abstractions**
    - Scala helps you manage complexity by letting you raise the level of abstraction in the interfaces you design and use.
    For example, The Scala code treats the same strings as higher-level sequences of characters that can be queried with predicates
    ```scala
        val tr = str.exists(_.isUpper)
    ```
    - Functional literals are lightweight
- **Advanced static type system**
    - it allows you to parameterize types with generics, to combine types using intersections, and to hide details of types using abstract types.
    - Although some argues that static typed language is verbose and not flexible, In scala  **verbosity** is avoided through type inference and **flexibility** is gained through pattern matching and several new ways to write and compose types.
    - Advantages of static typing system:
        - Verifiable properties: Static type systems can prove the absence of certain run-time errors. Reduce the number of unit tests.
        - Safe refactoring : make changes to a codebase with a high degree of confidence
        - Documentation: Static types are program documentation that is checked by the compiler for correctness.
            - Scala has a very sophisticated type inference system that lets you omit almost all type information that’s usually considered annoying

## 4.Scala roots
Although only a few features of Scala are genuinely new; most have been already applied in some form in other languages. Its design models many languages, such as SmallTalk, Ruby, Algol, Simula, OCaml, Haskell etc.
Scala’s innovations come primarily from how its constructs are put together
Scala is also not the first language to integrate functional and object-oriented programming, although it probably goes furthest in this direction

Given Scala's innovations:
-   its abstract types provide a more object-oriented alternative to generic types,
-   its traits allow for flexible component assembly,
-   its extractors provide a representation-independent way to do pattern matching.

## 5. Starting Programming
[Programming In Scala](https://github.com/lgrcyanny/ScalaPractice/tree/master/ProgrammingInScala)
读书的时候写写代码，都是书里的例子，用maven构建的scala progject。
