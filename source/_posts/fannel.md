---
title: Fannel相关介绍 
date: 2018-08-28 00:21:18
tags:
  - Go 
  - Fannel
  - Kubernetes 
categories:
  - 技术
---

##### Fannel是什么？ 
Fannel是使用UDP实现的一种覆盖网络(Overlay)，即表示运行在一个网上的网。并不依靠IP地址来传递信息，而是采用一种映射机制，把IP地址和identifiers来做映射来资源定位。也就是将TCP数据包装在另一种网络包里面进行路由转发和通信。
< !--more-->

##### Fannel的运用场景是什么？
Flannel是CoreOS团队针对Kubernetes设计的一个网络规划服务，它的功能就是让集群中不同节点主机创建的Docker容器都具有全局唯一的虚拟IP, 并且能让他们相互ping通。

##### Fannel主要解决了什么问题？
试想，默认的Docker，在每个物理机节点上的Docker服务都会分别负责所在节点容器的IP分配，那这样就会导致一个问题，就是不同节点上不同容器可能会获得相同的IP地址。
例如：
物理机w1和物理机w2中分别有容器r1和容器r2，那怎么通过容器r1去访问容器r2呢？它们之间是没办法互相访问的。 再者，w1和w2如果都是使用Docker的默认配置的话，那r1和r2的IP默认应该都是172.17.0.2，服务注册的话不都注册相同的IP吗？ 那r1访问r2的时候，会不会很疑惑的发现r2的IP地址和自己一样？

##### Fannel实现 
flannel 使用etcd存储配置数据和子网分配信息。flannel 启动之后，后台进程首先检索配置和正在使用的子网列表，然后选择一个可用的子网，然后尝试去注册它。
etcd也存储这个每个主机对应的ip。flannel 使用etcd的watch机制监视/coreos.com/network/subnets下面所有元素的变化信息，并且根据它来维护一个路由表。为了
提高性能，flannel优化了Universal TAP/TUN设备，对TUN和UDP之间的ip分片做了代理。
