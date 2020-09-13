title: Akka http notes
date: 2020-09-13 21:47:31
tags: Akka, Scala
---

在快3年多的Scala项目编程中, Akka是我见过的比较高质量的scala库, 其核心抽象是一种基于Actor的编程模型, 同时在这个核心抽象上, 提供一组工具库, 用户只需要按Actor形式写业务逻辑, 框架会帮你处理好底层的消息传递, 高并发和IO问题. Akka在工业场景下, 很接底气, 比如有很多微服务, 服务的性能各有差异, 这时候你需要整合这些微服务, 完成比如广告投放, 在线推荐, 事故检测等业务, Akka的业务抽象就会有很大的用处.

而最近系统看了Akka-HTTP, 我个人比较喜欢这个库在meta-programming方面的应用, akka-http把一个老生常谈的HTTP库实现的很优雅, 设计和抽象值得推敲, 时间有限, 就看了一周, 以下是一些最近对我帮助比较大的总结, 如果以后有空会继续完善

## 1.Akka HTTP 优势
定位: 用于处理复杂业务的Library, 不是一个MVC Framework(such as Play)
* DSL with convenient pathMatchers
* Streaming: 流式传输, 速率限制
* Interacting with actor easy

<!--more-->

## 2.核心数据结构和抽象
* HTTP Module
    * HttpRequest
        * a method (GET, POST, etc.)
        * a URI (see URI model for more information)
        * a seq of headers
        * an entity (body data): HttpEntity
            * HttpEntity类型
                * Strict: 消息体小, 内存可放, String or ByteString
                * Default: Streaming Data Source, know data size
                * Chunked: unknown length
                * Multipart.BodyPart: streaming entity, unknown length
            * 大小控制: max-content-length
        * a protocol
    * HttpResponse
        * a status code
        * a Seq of headers
        * an entity (body data)
        * a protocol
    * Headers
        * content-type: 在HttpEntity中定义
        * Content-length
        * User-agent: 自动添加
        * Date: 自动添加
    * Other supporting types and tools:
        * Uri, HttpMethods, MediaTypes, StatusCodes, Attributes
        * Parsing/Rendering
* URI Module
    * model, handling special characters
        * foo://example.com:8042/over/there?name=ferret#nose
        * scheme://authority/path/query?fragment
    * Extract directives
        * Uri.query()
        * PathDirectives
* Marshalling/Unmarshalling
    * Marshalling(Serialization/Pickling): Convert high level object to low level wire format
        * Function: A => Future[List[Marshalling[B]]]
            * Future: 支持异步
            * List: 序列化后的对象, 提供多种格式
            * Marshalling[B]: 可以访问MediaType, HttpCharset这些属性; 延迟Marshalling的调用, 执行content negotiation
        * 常见的Mashallers
            * type ToEntityMarshaller[T] = Marshaller[T, MessageEntity]
            * type ToByteStringMarshaller[T] = Marshaller[T, ByteString]
            * type ToHeadersAndEntityMarshaller[T] = Marshaller[T, (immutable.Seq[HttpHeader], MessageEntity)]
            * type ToResponseMarshaller[T] = Marshaller[T, HttpResponse]
            * type ToRequestMarshaller[T] = Marshaller[T, HttpRequest]
        * Support Implicit Resolution
    * Unmarshalling(Deserialization)
        * From low level MessageEntity to high level type T
        * Unmarshaller[A, Future[B]]
        * Support implicit resolution
    * Sparay-Json support


## 3.Server API
* High Level: Routing DSL, content negotiation, static content serving
    * https://doc.akka.io/docs/akka-http/current/routing-dsl/index.html
    * 设计思路:
        * 更可读, 可维护性更好, 可组合, 可扩展的API构建方式
        * 以Directives构建Route Tree
        * Route Tree可以转换为底层的Flow
    * Route的接口定义:
        * type Route = RequestContext => Future[RouteResult]
            * RequestContext: 提供比HttpRequest更丰富的内容
            * RouteResult: 是一个ADT(algebraic data type), 一般从RouteDirectives创建(complete, reject, redirect)
        * Composing Route
            * transformation: 代理给inner route处理
            * filtering: condition and reject
            * chaining: concat指令, 构建Route Tree, 不建议使用~符号
        * Sealing a route
            * 可以返回response的route
    * Directives是构建route的核心
        * 创建route
            * val route: Route = { ctx => ctx.complete("yeah") }
            * val route: Route = _.complete(“yeah")
            * val route = complete("yeah")
        * 其他功能
            * Transform/Filter/Extract incoming RequestContext
            * Chain RouteResult
            * Complete a request

## 4.代码重要的模块
* Akka-http: High level functionality, dsl, marshalling/unmarshalling
* Akka-http-core: low level http protocols
* Akka-http-tools
    * Akka-http-testkit
    * Akka-http2-support
    * Akka-http-spray-json
    * Akka-http-xml

## 5.执行层
* 基于akka stream
* Timeout
    * Client/Server: IdleTimeout: 链接如果一段时间没有request和response
    * Client:
        * akka.http.client.connecting-timeout
        * akka.http.host-connection-pool.max-connection-lifetime: This timeout configures a maximum amount of time, while the connection can be kept open
    * Server:
        * akka.http.server.request-timeout 20s
        * akka.http.server.bind-timeout
        * Linger timeout: server implementation will keep a connection open after all data has been delivered to the network layer
* Caching
    * Based on caffeine(https://github.com/ben-manes/caffeine/)
    * Design Idea:
        * 避免羊群效应
        * Cache future
    * 应用: Frequency-biased LFU cache



