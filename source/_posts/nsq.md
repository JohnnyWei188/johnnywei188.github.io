---
title: nsq的简介
date: 2018-11-03
tags:
  - Go 
  - golang
  - nsq 
categories:
  - 技术
---

#### nsq是什么？ 
nsq是由golang开发的一个轻量级高性能、消息无序的分布式实时消息队列平台。
<!-- more -->

#### nsq有哪些特点呢？
- 分布式拓扑结构，可以避免单点故障
- 低延时的消息传递
- 可以水平的扩展
- 组合的负载均衡和多播消息路由
- 运行时的生产者和消费者自动发现
- ……

#### nsq包含了哪些东西？
nsq主要包含了3个组件， nsqd、nsqadmin、nsqlookupd。

其中nsqd是在一个最基本的节点服务，负责接受消息、以及把消息分发给客户端。nsqd启动后监听2个端口，4151(http)和4150(tcp)，客户端可以通过它们以http或tcp 2种协议和nsqd交互，进行消息的消费或者生产。

nsqlookupd是一个守护进程，它负责管理所有nsqd基本节点的拓扑信息，启动的时候也监听2个端口，4161(http)和4160(tcp)，其中，nsqd通过nsqdlookupd的4160端口，上报自己的节点信息，而4161端口，则是提供给nsqadmin后台查询、管理整个nsqd集群。

nsqadmin是一个http服务，服务启动时监听4171端口并通过4171端口提供了一个web UI界面，通过这个界面可以很方便的管理整个nsq平台，包括有多少个nsqd节点，对应的端口，当前消费队列，topic，channel等等。

#### nsq中的topic和channel概念
1. 第一个说的就是topic，其实就是指的某一类消息，比如，我们有支付类的消息，或者是用户登录类的消息。每个不同种类的消息我们可以建不同的topic
  
2. 而channel，其实是基于topic来说的，同一个topic下，可以有很多channel，也就是可以有很多的消费者来消费同一个topic下的消息，每个channel接收到的消息都是一样的，等价的的。nsq会负责把topic下的每一条消息分发给每一个channel
