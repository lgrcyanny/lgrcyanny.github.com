title: Learning Akka
date: 2015-10-07 21:54:48
tags:
  - Akka
  - Scala
---

最近学习了scala，主要是跟着**Functional Programming in Scala**的课程学习的，scala主要的用处还是在spark上，关于spark也看了一些源码，处于继续探索中。

并发时的消息通信是处理spark这样的分布式系统的关键。spark主要依赖scala社区的akka进行消息通讯。在spark1.2的代码中可以看到很多actor的影子，然而spark1.4中，都是封装为EndPoint，但Actor的思想和模式还在。

基于Actor的消息通讯模型是akka的核心。Akka为处理并发、容错和可扩展性的分布式问题提供了一套基于Actor模型的库。并发的消息通讯中，抽象是关键的方面。Actor模型是1973年由Carl Hewitt在论文"Actor Model of Computation- Scalable Robust Information Systems"中提出, 后被应用于爱立信公司研发的Elang语言，爱立信公司应用Actor模型开发了高并发和可靠的通信系统。

<!--more-->

###1. Akka的重要的4个方面
*Actor*

* 为处理并发，并行问题提供了简单和high-level的抽象
* event-driven的异步非阻塞通讯
* 轻量的actor进程，几百万个actor只占heap 1G左右

*容错性*

* 具有"let-it-crash"语意的监督者层级结构
* 不同的监督者层级可以分布在不同的JVM中，提供真正容错的系统
* 系统可以自我恢复，never stop

*寻址透明*

* Actor在Akka内部采用统一的寻址方式，屏蔽底层细节
* Actor是纯消息传递，且为异步消息传递

*可持久化*

* Actor收到的消息会被存储在邮箱中，收到的消息可以被持久化，actor可以被迁移到其他节点，重新回复并重启。

总之，Akka的设计围绕着以下三个核心展开：

* Scale-up 并发
* Scal-out(Remoting)，可扩展性
* 容错性

###2. 永远不变的HelloWorld
[Hello World](https://github.com/lgrcyanny/LearningAkka)
*Greeter*
Actor必须实现receive方法, 通过hello world例子可以看到actor的入门很简单。
```scala
import akka.actor.Actor
object Greeter {
  case object Greet
  case object Done
}
class Greeter extends Actor {
  def receive = {
    case msg@Greeter.Greet =>
      println(s"Hello Akka, receive $msg")
      sender() ! Greeter.Done
  }
}
```

*HelloWorld Actor*
负责启动Greeter Actor
```scala
class HelloWorld extends Actor {
  override def preStart = {
    val greeter = context.actorOf(Props[Greeter], "greeter")
    greeter ! Greeter.Greet
  }

  override def receive = {
    case Greeter.Done =>
      context.stop(self)
  }
}
```

*Main启动*

```scala
object Main {
  def main(args: Array[String]) {
    akka.Main.main(Array(classOf[HelloWorld].getName))
  }
}
```
Akka的主要学习还是看官方的文档，akka的特定用法看api很不错。
以上就是akka的入门的一些学习吧，写的仓促，如有不对的地方还请指正。最近很忙，也没能好好写博客。
akka的深入篇，会持续更新。
