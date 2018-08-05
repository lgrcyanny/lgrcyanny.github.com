layout: posts
title: 'Java Performance Toolbox'
date: 2018-08-04 18:24:34
tags: java performance
---

On Satuaday, I learned **The Java Performance Definitive Guide[chapter 3]**, here is a brief summary about Java Performance Toolbox.

## System Monitoring Tools

### 1. CPU Usage

vmstat: Report virtual memory statistics, vmstat reports information about processes, memory, paging, block IO, traps, disks and cpu activity
vmstat [options] [delay [count]]
<!--more-->
```
vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 71322688 314388 127644112    0    0     0     7    0    0  9  1 90  0  0
 0  0      0 71322816 314388 127644112    0    0     0     0 5590 7874  1  0 99  0  0
 0  0      0 71322592 314388 127644144    0    0     0     0 5418 7226  0  0 99  0  0
 0  0      0 71323208 314388 127644144    0    0     0    12 4952 7199  0  0 100  0  0
 0  0      0 71323600 314388 127644144    0    0     0   104 5253 7262  1  0 99  0  0
```
Tips:
+ CPU time is the first thing to examine when looking at performance of an application.
+ The goal in optimizing code is to drive the CPU usage up (for a shorter period of time), not down.
+ Understand why CPU usage is low before diving in and attempting to tune an application.


### 2. Disk Usage

iostat: Report Central Processing Unit (CPU) statistics and input/output statistics for devices and partitions.

+ %user: Show the percentage of CPU utilization that occurred while executing at the user level (application).
+ %system: Show the percentage of CPU utilization that occurred while executing at the system level (kernel).
+ rrqm/s: The number of read requests merged per second that were queued to the device
+ avgrq-sz: The average size (in sectors) of the requests that were issued to the device
+ %util: Percentage of elapsed time during which I/O requests were issued to the device (bandwidth utilization for the device). Device saturation occurs when this value is close to 100%
```
iostat -xm 5
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           9.40    0.00    0.52    0.01    0.00   90.07

Device:         rrqm/s   wrqm/s     r/s     w/s    rMB/s    wMB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sda               0.00     0.58    0.15    4.49     0.00     0.14    61.65     1.31  283.55    0.22  292.90   0.06   0.03
```

### 3. Network Usage

netstat: Print network connections, routing tables, interface statistics, masquerade connections, and multicast memberships
```
netstat -s
```

The book use `nicstat`, which is not built-in Linux server, it can be installed by yum.
```
nicstat 5
```
Be careful that the bandwidth is measured in bits per second, but tools generally report bytes per second


## Java Monitoring Tools
### 1. Basic JVM INFO
```
jcmd process_id VM.uptime
jcmd process_id VM.system_properties
jcmd process_id VM.version
jcmd process_id VM.command_line
jinfo -sysprops process_id
jinfo -flags process_id
jinfo -flag PrintGCDetails process_id
```
Some tuning flags can be set by jcmd and jinfo in command line, such as manageable options and C2 diagnostic (the flag provides diagnostic output for the compiler engineers to understand how the compiler is functioning).
```
jinfo -flag -PrintGCDetails process_id  # turns off PrintGCDetails
jinfo -flag PrintGCDetails process_id
```

tips:
+ jcmd can be used to find the basic VM information—include the value of all the tuning flags—for a running application.
+ Default flag values can be found by including -XX:+PrintFlagsFinal on a command line. This is useful for determining the default ergonomic settings of flags on a particular platform.
+ jinfo is useful for inspecting (and in some cases changing) individual flags.

### 2. Thread Info

```
jstack process_id
jcmd process_id Thread.print
```

### 3. Class Info

```
jconsole
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
```

### 4. Heap Info

```
jmap -heap process_id
jmap -dump:[live,] format=b, file=filename process_id
jhat -port 7000 dump_file
```

Another way, add hprof option to java process

```
java -agentlib:hprof=help
HPROF: Heap and CPU Profiling Agent (JVMTI Demonstration Code)
hprof usage: java -agentlib:hprof=[help]|[<option>=<value>, ...]
Option Name and Value  Description                    Default
---------------------  -----------                    -------
heap=dump|sites|all    heap profiling                 all
cpu=samples|times|old  CPU usage                      off
monitor=y|n            monitor contention             n
format=a|b             text(txt) or binary output     a
file=<file>            write data to file             java.hprof[{.txt}]
net=<host>:<port>      send data over a socket        off
depth=<size>           stack trace depth              4
interval=<ms>          sample interval in ms          10
cutoff=<value>         output cutoff point            0.0001
lineno=y|n             line number in traces?         y
thread=y|n             thread in traces?              n
doe=y|n                dump on exit?                  y
msa=y|n                Solaris micro state accounting n
force=y|n              force output to <file>         y
verbose=y|n            print messages about dumps     y
```

```
-agentlib:hprof=cpu=samples,lineno=y # for cpu
-agentlib:hprof=heap=sites,lineno=y # for heap
```
Some notes:
+ heap=sites, sites is a sorted list of allocation sites.  This identifies the most heavily allocated object types, and the TRACE at which those allocations occurred.
+ cpu=samples,  is a statistical profile of program execution.  The VM  periodically samples all running threads, and assigns a quantum to active TRACEs in those threads.
+ cpu=time, is a profile of program execution obtained by measuring the time spent in individual methods (excluding the time spent in callees), as well as by counting the number of times each method is called

[hprof ref](https://docs.oracle.com/javase/8/docs/technotes/samples/hprof.html)


### 5. Heap Dump Processing
Heap dumps can be captured from the jvisualvm GUI, or from the command line using jcmd or jmap.
Or you can use Eclipse Memory Analzyer Tool.


## Java Profiling Tools

### 1. Tools types

+ Sampling profilers
Sampling-based profilers are the most common profiler. There may be error in sampling profiler's result. The way to minimize these errors is to profile over a longer period of time, and to reduce the time interval between samples.

+ Instrumented profilers
Instrumented profilers yield more information about an application, but can possibly have a greater effect on the application than a sampling profiler.
Instrumented profilers should be set up to instrument small sections of the code, a few classes or packages. That limits their impact on the application’s performance.


## 2. JMC(Java Mission Control)
It's a great profiling tool built-in JDK(jdk 7 or higher). On local machine, just type `jmc` command, the jmc UI will show.
![jmc ui](http://wx3.sinaimg.cn/mw690/761b7938ly1ftz2nfllbmj21kw0xz4j5.jpg)

Then how to connect jmc to remote Linux server.
+ Firstly, add jmx configurations to Linux java process
```
-XX:+UnlockCommercialFeatures -XX:+FlightRecorder -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.port=8091 -Dcom.sun.management.jmxremote.rmi.port=8091 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false
```

+ Secondly, config local jmc connection
fill the server and port, click Finished. That's all.
![config local jmc connection](http://wx3.sinaimg.cn/mw690/761b7938ly1ftz2nkhtqvj20sw0pcwis.jpg)

[jmc help guides](https://docs.oracle.com/javacomponents/jmc-5-5/jmc-user-guide/toc.htm)

*Use JMC the dump files:*
firstly, add `-XX:+UnlockCommercialFeatures -XX:+FlightRecorder` to application
secondly, type these commands:
```
jcmd process_id JFR.start
jcmd process_id JFR.dump filename=path
jcmd process_id JFR.stop
```

## Summary
+ System monitoring tools: vmstat, iostat, netstat
+ Java built-in tools: jinfo, jcmd, jmap, jhat, jstat, jconsole, jvisualvm, jmc, jhprof
+ No perfert tools for everything, when do profiling work, use right tools right applications

## Reference
[Java Performance: The Definitive Guide](https://www.amazon.com/Java-Performance-Definitive-Guide-Getting/dp/1449358454/ref=sr_1_1?ie=UTF8&qid=1533475568&sr=8-1&keywords=java+performance+definitive+guide)