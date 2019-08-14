---
title: 部署nsq到kubernetes上
date: 2019-08-14
tags:
  - kubernetes 
  - nsq 
categories:
  - 技术
---

#### 介绍一下怎么把nsq部署在k8s集群上
Kubernetes是容器编排生态中的明星，它是一个Google开源的的大规模容器管理系统borg的开源实现。目前我们公司所有的应用都在docker化，几乎所有的应用都部署在k8s平台上。当然包括一些缓存、以及像nsq这的消息队列。我们的nsq就是部署在k8s上。
<!-- more -->

#### 以什么方式部署呢
在介绍如何部署之前，其实应该对k8s要有一定的熟悉，因为部署涉及到k8s中的一些概念，比如node, pod, service, 还有DaemonSet等。后面再写过一篇文章说kubernetes，现在只说如何部署nsq在k8s集群上。

首先我们需要部署nsqd、nsqadmin、nsqlookupd它们的service。部署service的时候，我们可以以clusterIP的方式，每个service分配一个clusterIP，然后分部暴露它们对应的端口，这样我们在启动nsqd和nsqadmin的时候，就可以直接使用nsqlookupd的service clusterIP加对应的端口(4160 tcp 端口接收nsqd的广播信息、4161 http端口给nsqadmin查询)

接着部署nsqd，因为nsq本身就是一个拓扑关系的分布式系统，所以，我们部署nsqd的时候，也是考虑部署多个节点，那部署到多个节点的话，当然也是希望能够分散到每个k8s的node节点上，因此，我们可以使用k8s的DaemonSet控制器来部署nsqd，它会为我们在每个node上都启动一个nsqd服务。

`部署nsqd的时候需要注意的2点就是：`
- 在部署nsqd的时候，因为我们需要把部署的nsqd自身的信息广播到nsqlookupd上，所以，需要在nsqd启动的时候带上自身的IP。
- 考虑如果nsq有数据落地到了磁盘而nsqd重启情况，因为是在容器里面，如果重启那所有的文件数据都被删除，因此，我们还需要把nsq的data文件挂载到主机上。

第一个问题，kubernetes支持在容器启动的时候，可以把pod的信息注入到容器里面，注入可以通过环境变量和Downward API2种方式，这里我们使用环境变量注入

第二个问题，我们使用volume 的hostPath方式，把node主机上的目录挂载到pod内。这样就可以把nsqd的data文件数据持久化。
部署文件类似这样:
```
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: nsqd
spec:
  template:
    metadata:
      labels:
        app: nsq
        component: nsqd
    spec:
      containers:
      - name: nsqd
        image: nsqio/nsq
        imagePullPolicy: Always
        env:
        - name: POD_IP
          valueFrom:
            fieldRef:
               fieldPath: status.podIP
        resources:
          limits:
            cpu: 1000m
            memory: 1024Mi
          requests:
            cpu: 100m
            memory: 128Mi
        ports:
        - containerPort: 4150
          name: tcp
        - containerPort: 4151
          name: http
        volumeMounts:
        - name: data
          mountPath: /data
        command:
          - /bin/sh
          - -c
          - "nsqd --lookupd-tcp-address=10.96.0.3:4160 --broadcast-address=$POD_IP --data-path=/data "
      terminationGracePeriodSeconds: 5
      volumes:
      - name: data
        hostPath:
          path: /data/nsq
```
最后我们以Deployment的方式部署nsqadmin、nsqlookupd。部署好之后，我们当然还希望能在浏览器访问nsqadmin，这时候在通过ingress，把nsqadmin服务暴露出去，给浏览器访问。

所有的yaml文件在这里：https://github.com/JohnnyWei188/nsq-on-k8s/tree/master

