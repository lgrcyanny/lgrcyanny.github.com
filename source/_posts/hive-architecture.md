title: Hive架构笔记
tags:
  - Hadoop
  - Hive
id: 696
categories:
  - Hadoop
  - Hive
date: 2014-08-16 23:32:27
---

Hive是基于Hadoop的大数据查询引擎, 存储于HDFS上的数据,要求开发者采用MR才能进行计算, 但MR是low-level的分布式计算,对于复杂的分析job, 多重MR是很复杂的, Hive致力于让用户采用传统的RDBMS的SQL查询处理分布式的大数据.
<!--more-->

### 1\. 架构

[![Image and video hosting by TinyPic](http://i60.tinypic.com/2qs2w6e.jpg)](http://tinypic.com?ref=2qs2w6e)

(1) external interface: 提供用户接口，WebUI, CLI, JDBC, ODBC
(2) Thrift Server：为JDBC, ODBC 提供服务
(3) MetaStore: 存储元数据信息
(4) Driver：控制HQL的生命周期， 包括

*   Compiler: 将HQL编译为Operator DAG的执行计划
*   Optimizer: 对Operator DAG进行基于规则的优化
*   Executor: 执行Optimizer最终生成的DAG MR
(5) Hadoop：MR， HDFS： 执行physical plan（DAG）

### 2\. Hive与传统数据库的区别

虽然Hive提供的HQL和RDBMS很类似,并且也致力于向标准SQL靠近, 但还是有一些区别:

(1) Hive Data Model and Typed System

*   Hive支持传统数据类型,如int , float, double, string, 也支持struct, array, map
*   Hive 提供对struct, array, map的field访问的语法,例如”a[1]”, “a.b.c”
*   Hive提供SerDe Interface, 可以支持用户自定义的文件格式的读写
(2) HQL

*   Hive的子查询只能在from子句中, from可以在select前面
*   Hive join只支持等值join
*   Hive支持multi insert
*   HQL中可以自定义Map和Reduce脚本
*   Hive引入了Partition的概念, 例如数据总是按day, hour产生时,可以分配event_day, event_hour的partition, 每一个查询针对每一个partition操作,就不会加载所有的数据,加快查询,提高性能

### 3\. Hive MetaStore and Data Model

Hive MetaStore提供元数据信息的抽象
(1) DataBase
类似传统的数据库概念，是表的namespace，默认的table放入’default’ database

(2) Table
数据表
重要信息包括：schema, location, SerDe, input_format, out_format等

(3) Partition
每个表的数据可以有分区, 每个partition key需要在create table时指定,例如: event_day, event_hour等
每个Partition可以有自己的schema和location.

(4) Bucket
Table和Partition的Storage Descriptor可以按指定的column bucket(分桶), 默认以hash分桶, 分桶可以优化map join, table sample操作.
Bucket是Table或Partition所在directory下的file

Hive Metastore信息可以存储在本地Derby数据库,或MySQL, Hive采用DataNucleus进行ORM

### 4\. Driver

目标：将HQL解析为DAG DAG的物理执行计划
步骤：
(1) Compiler的Parser将HQL解析为AST(Abstract Syntax Tree), Hive中采用Antlr进行词法解析和语法解析

(2)Semantic Analyzer, 语义分析,检查Table是否存在, 类型, 表达式等是否合法
语义分析将AST解析为中间表示(Intermediate Representation), 即Query Block(QB) tree,
QB tree 是一个tree的结构, 目的是方便将AST转化为Operator DAG.

(3) Logical Plan Generator将QB tree转化为logical plan, 即Operator DAG

(4) Optimizer优化Operator DAG, 进行基于规则的算子变换(transformation), 每一个变换会实现5个接口, Node, GraphWalker, Dispatcher, Rule 和Processor, Opritimizer提供的优化包括:

*   列裁剪
*   partition裁剪
*   谓词下推, 更早地filter数据, 从源头减少数据量
*   合并多个共享join key的join算子
*   Join Reordering: 在reduce时, 让small table常驻内存
*   基于用户的Hint的优化, 包括:

    *   Repartitioning of data to handle skews in group by processing: 对于有倾斜性的数据, reducer分配到的数据不均匀,影响性能, 做聚集时, 例如sum, 可以将group by拆分为两个map reduce阶段, 第一个阶段hash randomly分配数据,并部分聚集, 减少第二个阶段的聚集量.
    *   map side join(将小的表存储在内存中)
    *   Hash based partial aggregations in the mappers, 将需要聚集的行存在内存hash table中
    *   Physical Plan Generator 将Operator DAG转化为Physical Plan, 包括多个阶段的MR和HDFS tasks

*   Execution Engine 按照物理计划的依赖关系执行task

### 5\. 相关的

*   Hive基于成本的优化
*   Scope, 微软的SQL Like language
*   Pig, allows users to write declarative scripts to process data
*   Dremel, Google’s Query Engine
*   Apache Drill
*   Spark
*   MR