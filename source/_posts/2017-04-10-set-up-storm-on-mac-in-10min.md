title: Set Up Apache Storm On Mac In 10min
date: 2017-04-10 20:19:23
tags: apache storm
---

Storm is a great real time streaming system. Recently, my project is about spark streaming. I want to learn storm either to know more about streaming system. Okay, let's fire up.
Today I tried to install storm cluster on my local mac.
It was easy to install. It will cost you about 10min.

<!--more-->

## 1. install zookeeper

- download [zookeeper-3.4.9](http://www.apache.org/dyn/closer.cgi/zookeeper/)
- configure conf/zoo.cfg as follows:

```shell
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# replace it as your local dir
dataDir=/Users/lgrcyanny/Codelab/zookeeper/zookeeper-3.4.9/zkdata
# the port at which the clients will connect
clientPort=2181
```

## 2. install storm

- download [latest storm 1.1.0](http://storm.apache.org/downloads.html)
- configure conf/storm.yaml as follows:

```shell
storm.zookeeper.servers:
    - "localhost"
#storm.zookeeper.port:2181

storm.local.dir: "/Users/lgrcyanny/Codelab/storm/apache-storm-1.1.0/storm-local"

#
# nimbus.seeds: ["host1", "host2", "host3"]
#
nimbus.seeds: ["localhost"]

supervisor.slots.ports:
    - 6700
    - 6701
    - 6702
    - 6703
```

to understand these config, please refer to: [Setting-up-a-Storm-cluster.html](http://storm.apache.org/releases/1.1.0/Setting-up-a-Storm-cluster.html)

## 3. start stom
```shell
# start nimbus
./bin/storm nimbus
# start supervisor for workers
./bin/storm supervisor
# start ui
./bin/storm ui
```
open http://localhost:8080, you will see storm started:
![start storm](http://wx1.sinaimg.cn/large/761b7938ly1fehv2miunpj21kw0ps0we.jpg)

