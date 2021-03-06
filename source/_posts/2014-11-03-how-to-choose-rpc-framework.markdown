---
layout: post
title: "How to choose RPC framework"
date: 2014-11-03 15:16:21 +0800
comments: true
categories: [rpc]
keywords: rpc, thrift, zerorpc
description: Why we need RPC, and discuss how to choose by comparing Apache Thrift and ZeroRPC.
---

RPC 是 remote procedure call 的缩写，意指调用远程进程的方法。这里的远程需要广义的理解，有时为了协议的统一即使调用本地进程也叫做 RPC，所以 RPC 可简单理解为进程间通信。

在选择 RPC 框架之前，要搞清楚为什么需要 RPC？ RPC 主要是为了解决服务化架构中客户端与服务端以及服务之间通讯的问题。在最早接触 RPC 的时候，我一直有一个疑问： RESTful API 不就搞定了么？为什么需要 RPC？直到深入实践了 RPC，我自己总结了 RPC 与 RESTful API 最大的几个不同。

<!-- more -->

首先，RESTful API 经常纠结的问题就是，到底在什么样的资源粒度上开放接口？到底需要哪些接口？而在 RPC 中这个问题被很大程度上淡化了，因为 RPC 使用起来几乎和本地方法没有太多区别。

其次，RESTful API 在应用层使用 HTTP 协议，这使得传输数据受到限制，实践中早些年流行过 xml，现在 JSON 应该是标准做法了。但这些在大量读写时，都会消耗较大的流量，构造这些结构必然消耗额外的带宽。而 RPC 的协议都是由各框架自己定义的，目前还没有形成标准，以 Apache Thrift （以下简称 Thrift）为例，使用二进制的编码大大降低了数据大小。

最后，也是我认为最重要的区别， RPC 可以实现异步请求，而受限于 HTTP 协议的 RESTful API 则无法实现这个功能。而异步请求让整个请求过程变得非阻塞，比如在一个 PHP 进程中将互不依赖的几个数据请求变成异步执行，这样执行时间取决于最慢的请求而不是它们相加。

基于以上 3 点，我们非常确定需要选择一个 RPC 框架，比较流行的有 Thrift, Google Protocol Buffer, Avro。我将它们归为一类，并以 Thrift 为代表。但我倾向于另外一个选择：基于 ZeroMQ 和 Msgpack 的 ZeroRPC。下面以 Thrift 和 ZeroRPC 的对比来解释我的选择，这些对比并不涉及易用性或者安全性，仅仅从核心的信息传递角度来比较。

在 RPC 框架中，最重要的 3 个核心组件： transport，protocol，encoding。

Transport，即传输方式。Thrift 使用 socket，而 ZeroRPC 使用 ZeroMQ —— 一个丰富扩展过的 socket 类库。在传输方式的灵活性上，ZeroRPC 明显胜过一筹。

Protocol，即通讯协议。Thrift 使用自己定义的 Tprotocol，协议并不复杂，以 byte 长度来规定消息格式。比如前 4 个 byte 来表示状态码。而基于 ZeroMQ 的扩展， ZeroRPC 可以使用消息帧（Frames）来构建更加灵活的通讯协议。

Encoding，即序列化或简单理解为数据压缩。Thrift 的文档中并没有透露过多关于序列化算法的细节，而 ZeroRPC 使用 Msgpack 作为序列化/反序列化的工具。 Msgpack 本身是一个优秀的开源项目，功能更加强大。

除了以上 3 个维度，我认为 ZeroRPC 还有另外一个明显的优势，ZeroRPC 是由 2 款开源软件组合而成，这意味着可以灵活的替换为其他类似的组件。比如 nanomsg 来替代 ZeroMQ，只要 SDK 封装的足够抽象，可以无缝的切换类库。

说了这么多，其实 ZeroRPC 最大的优势就是 ZeroMQ，给 socket 加上了更加丰富的可能性。但 ZeroRPC 没有提供 PHP 的客户端，我可能需要根据它的协议来自己构建一个。
