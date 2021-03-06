---
layout: post
title: "分布式系统架构概述（译）"
date: 2018-05-04 11:58:05 +0800
comments: true
categories: [best practice, translation]
---
我其实很少翻译东西，首先是因为英文和中文水平都有限，很难做到信达雅，其次一般的文章自己看过就结束了，很少有要翻译的冲动。这次要翻译的是一篇概括性很强的文章，讲的是分布式系统中需要了解的基础概念，值得保存下来经常回顾。

有趣的是，这篇文章在 [hacker news](https://news.ycombinator.com/item?id=16852295) 和 [reddit](https://www.reddit.com/r/programming/comments/8cxz7q/distributed_architecture_concepts_i_learned_while/) 上得到了基本相反的评价。我个人还是很认可这种概括性的文章的。就好像大学的课程大纲，有很强的指导作用。

+ [原文](http://blog.pragmaticengineer.com/distributed-architecture-concepts-i-have-learned-while-building-payments-systems/)
+ [译文纠错](https://github.com/liangshan/liangshan.github.io/issues/2)

-------------------------

两年前我做为移动开发工程师加入了 Uber，之前有一些后端开发的经验。我参与实现了 app 的支付模块——这个模块目前正在重写。之后，我开始进入工程管理领域，领导一个团队。这个团队负责很多和支付相关的后端系统，这意味着我更多的接触了后端开发的东西。

在 Uber 工作之前，我几乎没有分布式系统经验。我的教育背景是传统的计算机科学学位，之后做了十年全栈开发。虽然我之前略知一二，但我对分布式概念（如一致性，可用性或幂等性）没有太多的理解或欣赏。

在这篇文章中，我总结了一些构建大规模、高可用的分布式系统时需要了解的基本概念。这个系统就是 Uber 的支付系统，一个负载高达每秒数千次请求的系统，即使部分系统出现故障，关键支付功能也需要正常工作。 这是一个完整的清单吗？ 也许不是。 但如果我早点知道这些的话，会让我的生活变得更轻松。 因此，让我们深入了解 SLA、一致性、数据持久性、消息持久性、幂等性以及我在工作中需要学习的其他一些内容。

<!--more-->

## SLA
对于那些每天处理数百万次事务的大型系统来说，出错几乎是无法避免的。我发现在深入计划一个系统之前，最重要的事情是先定义这个系统如何算「健康」。尽量用一些可衡量的指标来表示「健康」。通用方法是用 [SLA](https://en.wikipedia.org/wiki/Service-level_agreement) 来衡量，即 service level agreements。我见过的最通用的 SLA 是：

+ **可用性（Availability）**：服务正常运行时间的百分比。显而易见的是应该努力让系统的可用性达到 100%，但做到这个目标不仅很难而且很贵。甚至像 VISA、Gmail、互联网供应商这样的大型且重要的系统都无法做到 100% 的可用性。历年来，他们总是会有那么几秒钟、几分钟、或者几小时不可用。对于大多数系统来说，4 个 9 的可用性（即 99.99%，换言之[每年不超过 50 分钟](https://uptime.is/)不可用时间）即可被称为高可用。仅仅是达到这一水平通常已经意味着大量的工作要做了。
+ **准确性（Accuracy）**：系统允许一些数据的丢失或不准确吗？如果允许，可以接受多大比例呢？对于我负责的支付系统来说，准确性的要求是 100%，意味着任何数据都不允许丢失。
+ **容量（Capacity）**：系统预期可以承受多大的负载？这个指标通常使用 QPS(requests per second) 来表达。
+ **延迟（Latency）**：系统应该在多久之后返回结果？95% 以及 [99%](https://www.quora.com/What-is-p99-latency) 用户请求的响应时间是多少？系统通常会有一些噪音数据，所以取 95% 和 99% 请求的延迟时间是现实世界中的可行做法。

**为什么 SLA 对于构建一个大型支付系统如此重要呢？** 我们在构建将要替换当前系统的新系统。为了确保我们所做是正确的，即我们认为新系统是「更好」的，所以我们使用 SLA 来定义指标。其中可用性是我们优先级最高的要求。一旦确定了标准，我们就可以在架构过程中权衡选择来达到它。

## 水平扩展（Horizontal scaling） Vs 垂直扩展（Vertical scaling）
假如一个由新系统支撑的业务持续在增长，那么系统负载也在增加。到了某个节点，现有的设施已经无法支撑更多负载和容量。两种通常会考虑的扩展策略就是水平扩展和垂直扩展。

水平扩展就是通过加机器（或节点）来提升容量。水平扩展是目前分布式系统最流行的扩展方式，尤其是现如今对于集群来说，增加一台机器（或虚拟机）仅仅只是点一个按钮那么简单。

垂直扩展基本上就是「买一台更大更好的机器」，也就是更多核、更多进程、更多内存。对于分布式系统来说，垂直扩展通常没那么常见，因为垂直扩展往往要花费更多钱。但也有些大型网站，比如 Stack Overflow 就有过[成功垂直扩展](https://www.slideshare.net/InfoQ/scaling-stack-overflow-keeping-it-vertical-by-obsessing-over-performance)来满足需求的案例。

**为什么扩展的策略对于构建一个大型支付系统如此重要呢？** 我们早期就确定了支付系统应该是水平扩展的。虽然垂直扩展在某些场景下可行，但就目前市面上昂贵的单台大型主机负载能力而言，我们对它是否能承受目前支付系统的流量持悲观态度，更不用说将来。另外，我们团队有些工程师有大型支付系统工作的经验，当初他们想要使用能买到的最贵的主机来垂直扩展，然而失败了。（译者注：后面这些话好像并没有回答前面的问题）

## 一致性（Consistency）
可用性对于任何系统都是重要的。分布式系统往往构建于一批可用性相对较差的主机上。假设我们的目标是构建 99.999% 的可用性（即每年 5 分钟不可用），而我们使用的机器，平均只有 99.9% 的可用性（每年 8 小时不可用）。那么达到目标可用性的一种直接的方式就是使用一组这样的主机来组成集群。即便是某些节点不可用，其他的会保持可用来达到更高的可用性。

一致性是高可用系统的关键指标。如果系统的所有节点在同一时间返回相同的数据，则认为这个系统是一致的。回到刚才的话题，由于我们使用了一组机器来组成高可用集群，确保系统保持一致就至关重要了。为了确保每个节点都返回相同的信息，它们之间需要消息通信。但是，消息传递可能会失败、会丢失，有些节点甚至不可用。

一致性是一个我花了最多时间去理解和欣赏的概念。有很多一致性的模型，分布式系统中最流行的[几种模型](https://en.wikipedia.org/wiki/Consistency_model)是[强一致性（strong consistency）](https://en.wikipedia.org/wiki/Consistency_model)，[弱一致性（weak consistency）](https://www.cl.cam.ac.uk/teaching/0910/ConcDistS/11a-cons-tx.pdf)和[最终一致性（eventual consistency）](http://sergeiturukin.com/2017/06/29/eventual-consistency.html)。Hackernoon 的这篇[强一致 vs 最终一致](https://hackernoon.com/eventual-vs-strong-consistency-in-distributed-databases-282fdad37cf7)给出了很好的可行的权衡建议。通常来讲，一致性越弱，系统越快，但拿不到最新数据的可能性也会越高。

**为什么一致性对于构建一个大型支付系统如此重要呢？** 系统中的数据要保持一致。但要多一致呢？对于系统的某些组件来说，只有强一致能满足要求。比如，确认一个支付是否已经初始化的数据需要确保强一致性。其他一些组件，也就是那些不太重要的组件，最终一致性是可以考虑的选择。一个很好的例子是近期交易列表，就可以使用最终一致性策略来实现（这意味着最新的交易未必立刻出现在列表里，但换来了更低的延迟和更小的资源消耗）。

## 数据耐久性（Data Durability）
[耐久性](https://en.wikipedia.org/wiki/Durability_\(database_systems\))意味着数据一旦被成功存储就可以一直继续使用。即使系统中的节点下线，崩溃或数据损坏也是如此。

不同的分布式数据库拥有不同级别的耐久性。有些系统支持机器/节点级别的耐久性，有些做到了集群级别而有些系统的耐久性并没有开箱即用。某种形式的数据复制是较为通用的提高耐久性的做法，因为把同一份数据存储在不同的节点上，即使有节点下线，数据仍然可以被访问。[这篇文章](https://drivescale.com/2017/03/whatever-happened-durability/)很好的解释了为什么分布式系统做到耐久性是很大的挑战。

<img width="800px" src="{{ root_url }}/images/custom/data-durability.png" />

**为什么数据耐久性对于构建一个支付系统如此重要呢？** 这个系统的大部分组件而言，如此重要的数据是不允许丢失的。分布式的数据存储需要支持集群级别的数据耐久性，即使实例会崩溃，完成了的交易仍然还在。现如今大部分分布式数据存储服务，诸如 Cassandra、MongoDB、HDFS 或是 Dynamodb 全都支持不同级别的耐久性，并且通过配置都可以支持集群级别的耐久性。

## 消息的持久化（Persistence）和耐久性（Durability）
分布式系统中的节点进行计算、存储和相互发送消息。发送消息的一个关键性指标是消息送达的可靠性。对于重要的系统而言，常常不允许任何消息的丢失。

对于分布式系统而言，消息通讯通常由分布式消息服务完成，比如 RabbitMQ、Kafka。这些消息服务能支持（或配置后支持）不同级别的消息送达可靠性。

消息持久化的意思是当消息服务的节点发生了错误，已经发送的消息仍然会在错误解决之后被处理。消息耐久性则通常用在消息队列这一层。如果一个消息队列声明了耐久性，那么即使队列在消息发送之后掉线，仍然会在重新上线之后收到这条消息。[这里](https://developers.redhat.com/blog/2016/08/10/persistence-vs-durability-in-messaging/)有一篇很好的文章是关于这个话题。

<img width="800px" src="{{ root_url }}/images/custom/msg-durability.png" />

**为什么消息的持久化和耐久性对于构建一个支付系统如此重要呢？** 我们有太多重要的消息经不起丢失，比如乘客刚刚初始化了行程的消息。也就是说我们的消息系统是不允许丢消息的，每条消息都会被投递一次。但是，每条消息精确投递一次和至少投递一次是截然不同的复杂度。我们最终决定了实现一个耐久的消息系统，每条消息至少被投递一次，在底层选择了一个消息总线（最终我们选择了 Kafka）。

## 幂等性（Idempotency）
分布式系统偶尔会出错，比如连接会断开或是请求会超时。客户端经常需要重试这些请求。一个幂等的系统保证了无论同一个请求被执行了多少次，而最终只生效一次（这里特指写入操作，译者注）。一个很好的例子就是支付系统。如果一个客户端请求付钱，操作成功之后由于请求超时客户端重新发起了这个请求，这时幂等的系统不会重复扣费。对于没有考虑幂等的系统，会重复扣费。

出于幂等的考量，分布式系统会引入某种形式的分布式锁策略。这是最早被引入分布式系统的概念之一。假设我们打算引入乐观锁（optimistic locking）来解决并发更新的问题。而为了实现乐观锁，又要求系统的强一致性，因为这样才能在操作时使用某种版本控制来检查是否有另外的操作在进行中。

取决于系统层面的约束和操作的类型，有很多实现幂等性的方式。设计如何实现幂等性是一个不错的挑战，Ben Nadel [写了](https://www.bennadel.com/blog/3390-considering-strategies-for-idempotency-without-distributed-locking-with-ben-darfler.htm)他用过的不同策略，包括分布式锁和数据库约束。在设计分布式系统时，幂等性是最容易被忽略的部分之一，我的团队就因为没有确保某些关键操作正确的幂等性而付出过惨痛教训。

**为什么幂等性对于构建一个支付系统如此重要呢？** 最重要的是：避免重复扣费和重复退款。前面提到了我们的消息系统确保消息至少被投递一次，可以想象所有消息都有可能被投递多次而系统就必须确保幂等性。我们最终选择了使用版本控制和乐观锁来解决这个问题，这个幂等性的系统则使用拥有强一致性的持久化存储作为数据源。

## Sharding 和 Quorum
分布式系统通常会存储巨量的数据，超过了单个节点的能力范围。所以如何在一组机器上存储一批数据呢？最常用的做法就是 [sharding](https://en.wikipedia.org/wiki/Shard_\(database_architecture\))。数据基于某种哈希算法被水平的拆分。尽管大部分分布式系统的 sharding 策略都在底层，但 sharding 是一个学起来很有趣的领域，尤其是 [resharding](https://medium.com/@jeeyoungk/how-sharding-works-b4dec46b3f6)。Foursquare 曾经在 2010 年由于一个 sharding 的边界问题导致了 17 个小时的宕机，之后有一篇[很棒的文章](http://highscalability.com/blog/2010/10/15/troubles-with-sharding-what-can-we-learn-from-the-foursquare.html)分享了问题的根本原因。

许多分布式系统在多个节点上存储数据或进行计算。为了保证操作的一致性，一个基于投票的方案被发明，简言之必须有一定数量的节点都得到相同的结果操作才算成功。这个方案就是 quorum。

**为什么 sharding 和 quorum 对于构建一个支付系统如此重要呢？** 这其实是分布式系统非常基础的两个概念。我个人在我们设置 Cassandra 复制策略的时候第一次遇到它们。Cassandra (或是其他分布式系统) 使用 [quorum](https://docs.datastax.com/en/archived/cassandra/3.x/cassandra/dml/dmlConfigConsistency.html#dmlConfigConsistency__about-the-quorum-level) 和本地的 quorum 来确保集群的一致性。一个有趣的现象是，当我们开会的时候，一旦会议室有足够的人就有人会问：“可以开始了吗？我们有 quorum 机制吗？”（译者注，意思是并非所有人都在，大部分在也可以保证会议效果，和 quorum 一样 :D）

## Actor Model
通常我们使用的一些编程词汇，诸如变量、接口、方法调用等等，都基于单台主机的系统（译者注，我觉得这里想表达的应该是单个进程内部，和后面的做对比）。当讨论分布式系统时，我们需要另外一种方式来表达。一种通用的描述这类系统的模型叫 [actor model](https://en.wikipedia.org/wiki/Actor_model)，我们使用这个模型来描述系统间的通信。这个模型非常流行，是因为它和我们实际生活中的心智模型相匹配，比如人在一个组织中如何互相交流。而另外一个流行的模型叫做 [CSP](https://en.wikipedia.org/wiki/Communicating_sequential_processes) —— communicating sequential processes。

Actor model 基于 actor 如何相互发送信息和做出反应。每个 actor 被定义了有限的一系列行为——创造其他 actor，发送消息或是决定下一步的动作。只需要一些简单的规则，就可以很好的描述一个复杂的分布式系统，甚至在一个 actor 崩溃之后还能自我修复。我推荐 Brian Storti 的这篇 [The actor model in 10 minutes](https://www.brianstorti.com/the-actor-model/) 作为简介。许多编程语言都实现了 actor model 的[类库或框架](https://en.wikipedia.org/wiki/Actor_model#Actor_libraries_and_frameworks)，我们在 Uber 的一些项目中就使用了 [Akka 工具包](https://doc.akka.io/docs/akka/2.4/intro/what-is-akka.html)。

**为什么 actor model 对于构建一个支付系统如此重要呢？** 我们有很多工程师参与构建这个系统，其中很多人都有分布式系统的开发经验。我们决定遵循一种现行标准，而不是自己造轮子。

## Reactive 架构
构建大型分布式系统的时候，目标通常是有弹性的、可伸缩的、可扩展的。可以是支付系统或是其他类似的高负载系统，可以使用相同的模式来实现这些目标。业内人士已经总结并分享了相关工作的最佳实践——而 Reactive 架构是其中最流行和广泛接受的。

想要快速了解 Reative 架构，我建议读读这篇 [Reactive 宣言](https://www.reactivemanifesto.org/) 然后看一下这个 [12 分钟的视频](https://www.lightbend.com/blog/understand-reactive-architecture-design-and-programming-in-less-than-12-minutes)。

**为什么 Reactive 架构 对于构建一个支付系统如此重要呢？** Akka，也就是之前提到过的我们用来搭建支付系统的类库，深受 Reactive 架构的影响。许多我们的工程师对这套最佳实践也非常熟悉，所以遵循这些原则——响应式的、可伸缩的、有弹性的、消息驱动的——来构建这个系统变得非常自然。能有一种模型可以用来回溯和检查事情是否进展顺利是非常有用的，将来再构建新的系统我还会使用这个模型。

## 结束语
我非常幸运的有机会来重建一个及其关键的大型分布式系统：支撑 Uber 的支付系统。我接触到了许多之前不太熟悉的分布式的相关概念。所以我稍作总结，希望能对其他刚刚接触或者继续学习分布式系统的人有帮助。

这篇文章非常集中在如何设计一个系统的架构。还有很多事情可以讲，比如开发、部署、迁移以及如何可靠的运维这些系统。但这些话题需要其他的文章来描述。
