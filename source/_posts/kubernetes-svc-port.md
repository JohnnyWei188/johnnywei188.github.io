---
title: kubernetes中service中nodePort,port,targetPort含义
date: 2019-12-02 15:08:18
tags:
  - Go 
  - kubernetes 
categories:
  - 技术
---

##### Service中经常遇到需要配置port的情况，一不小心就会搞迷糊 
我们在配置kubernetes的svc的时候，经常回遇到需要暴露端口的情况，需要非常清楚各个port代表的是什么意思。
<!--more-->

```
kind: Service
apiVersion: v1
metadata:
  name: port-example-svc
spec:
  # Make the service externally visible via the node
  type: NodePort 

  ports:
    # Which port on the node is the service available through?
    # 在这个节点上, 可以通过哪个端口请求这个service
    - nodePort: 31234

    # Inside the cluster, what port does the service expose?
    # 在集群里面, 这个svc暴露了哪个端口给别人调用
    - port: 8080

    # Which port do pods selected by this service expose?
    # 目标端口, 指的是这个svc暴露的是对应pods的哪个端口
    - targetPort: 

  selector:
```
