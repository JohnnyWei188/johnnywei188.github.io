---
title: elasticsearch几个简单的查询 
date: 2018-11-12
tags:
  - Elasticsearch 
  - Go 
categories:
  - 技术
---
最近因为项目接触了下elasticsearch， 其实以前一直有所接触，PHP项目的时候有写过几个demo，现在转到go这边，又写了几个demo，比较认真的看了下es的文档。
<!-- more -->

##### 安装
安装我都是通过docker-compose来安装，顺便也安装了一下kibana，es装了个集群，就是安装之后本机有点卡。以下是compose的配置

```
elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.4.2
    container_name: elasticsearch
    environment:
        - cluster.name=docker-cluster
        - bootstrap.memory_lock=true
        - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
        memlock:
            soft: -1
            hard: -1
    volumes:
        - esdata1:/usr/share/elasticsearch/data
    ports:
        - 9200:9200
    networks:
        - webnet

elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.4.2
    container_name: elasticsearch2
    environment:
        - cluster.name=docker-cluster
        - bootstrap.memory_lock=true
        - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
        - "discovery.zen.ping.unicast.hosts=elasticsearch"
    ulimits:
        memlock:
            soft: -1
            hard: -1
    volumes:
        - esdata2:/usr/share/elasticsearch/data
    networks:
        - webnet

kibana:
    image: docker.elastic.co/kibana/kibana:6.2.4
    volumes:
        - ./kibana.yml:/usr/share/kibana/config/kibana.yml
    ports:
        - 5601:5601
    depends_on:
        - elasticsearch
        - elasticsearch2
    networks:
        - webnet

```


##### 记录了几个curl查询的语句

```
- 查询所有
curl -H 'Content-Type: application/json' 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{
    "query": {
        "match_all": {}
    }
}'

- match 匹配查询 全文搜索查询
curl -H 'Content-Type: application/json' 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{
    "query": {
        "match": {
            "message": "No.1"
        }
    }
}'

- match 匹配查询 精确查询，针对的是保存的精确值的字段，如数字、日期、布尔或者一个 not_analyzed 字符串字段
curl -H 'Content-Type: application/json' 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{
    "query": {
        "match": {
            "retweets": 0  
        }
    }
}'

- multi_match  多个字段进行匹配查询
curl -H 'Content-Type: application/json' 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{
    "query": {
        "multi_match": {
            "query": "No.1",
            "fields": ["user", "message"]
        }
    }
}'

- range查询  查询找出那些落在指定区间内的数字或者时间
- gt   大于
- gte  大于等于
- lt   小于
- lte  小于等于
curl -H 'Content-Type: application/json' 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{
    "query": {
        "range": {
            "retweets": {
                "gte": 0,
                "lt": 1
            }
        }
    }
}'

- term查询用于精确值匹配，这些精确值可能是数字、时间、布尔或者那些 not_analyzed 的字符串
//格式一
curl -H 'Content-Type: application/json' 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{
    "query": {
        "term": {
            "message": "test123"
        }
    }
}'

//格式二 这样也可以搜索   效果一样
curl -H 'Content-Type: application/json' 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{
    "query": {
        "term": {
            "message": {
                "value": "test123"
            }
        }
    }
}'

//term对输入的文本不进行分析，而进行精确匹配   这里查询出来的结果为Null
curl -H 'Content-Type: application/json' 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{
    "query": {
        "term": {
            "message": "No.1"
        }
    }
}'

//对比match  这里macth则可以进行文本分析 进行搜索出message 为"To Be No.1"的结果
curl -H 'Content-Type: application/json' 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{
    "query": {
        "match": {
            "message": "No.1"
        }
    }
}'

- terms查询，它允许你指定多值进行匹配。如果这个字段包含了指定值中的任何一个值，那么这个文档满足条件
curl -H 'Content-Type: application/json' 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{
    "query": {
        "terms": {
            "message": ["test123", "test456", "To Be No.1"]
        }
    }
}'

- exists查询  查询含有user字段的记录
curl -H 'Content-Type: application/json' 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{
    "query": {
        "exists": {
            "field": "user"
        }
    }
}'

- 6.x没有missing查询  使用must_not 查询不含有user字段的记录
curl -H 'Content-Type: application/json' 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{
    "query": {
        "bool": {
            "must_not": {
                "exists": {
                    "field": "user"
                }
            }
        }
    }
}'

- 写入一条消息
curl -H 'Content-Type: application/json' -XPOST 'http://localhost:9200/twitter/tweet?pretty=true' -d '
{
    "message": "test789"
}'
```

##### 写了个demo，遇到一个问题

在初始化es客户端的时候，如果是集群，可以使用elastic.SetSniff(true)进行所有可用es节点的嗅探，但是我本地却不行，原因是因为，我本地部署在docker的环境里，所有的es节点暴露的端口都是在docker的网络内，所以嗅探失败，导致es客户端连接不上。
我们可以看下nodes对应的IP都是docker网络环境下的IP地址
```
 curl 'http://localhost:9200/_cat/nodes?pretty=true'
172.21.0.5 33 97 2 0.13 0.10 0.11 mdi * EYP6Nad
172.21.0.8 34 97 2 0.13 0.10 0.11 mdi - sj-9rAe
```

##### 垃圾代码在这里
https://github.com/JohnnyWei188/es-demo



