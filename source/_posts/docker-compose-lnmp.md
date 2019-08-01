---
title: 使用docker-compose来构建一个本地的lnmp开发环境 
date: 2018-11-15
tags:
  - Docker 
  - Lnmp 
categories:
  - 技术
---
最近一直都在做微服务的开发，因为Docker容器技术的出现，为微服务架构提供了更为便利的条件，我们可以拆分我们的业务为一个一个小的单元进行部署，每个单元相互独立。微服务具有分布式系统的特性，比如横向伸缩性，服务发现，负载均衡，故障转移和高可用等。另外我们可能还有多版本支持，灰度发布，服务降级等。本文主要说的是怎么使用docker来构建一个本地的lnmp开发环境。

<!-- more -->

因为lnmp环境是需要我们部署多个服务，比如php, nginx, redis, mysql等，所以这里是通过使用docker官方下的一个项目叫Compose的工具来实现服务的快速编排。 `Compose是Docker官方的一个开源项目，主要负责是Docker容器集群的快速编排，官方文档说的是Compose可以定义和运行多个容器应用，通过docker-compose.yaml文件可以配置和定义你的服务。`

#### docker-compose.yaml文件配置
```
version: '3'
services: 
    nginx:
        build: ./nginx 
        ports:
            - "80:80"
        volumes:
            - "./nginx/volume-nginx/project:/etc/nginx/project"
            - "./nginx/volume-nginx/lualib:/etc/nginx/lualib" 
            - "./nginx/volume-nginx/nginx.conf:/etc/nginx/nginx.conf"
            - "./nginx/volume-nginx/logs:/etc/nginx/logs"
            - "./nginx/volume-nginx/vhosts:/etc/nginx/vhosts"
            - "./nginx/volume-nginx/ssl:/etc/nginx/ssl"
            - "~/wwwroot:/Users/Johnny/wwwroot"
        networks:
            - webnet

    php:
        build: ./php 
        ports:
            - "9000:9000"
        volumes:
            - "~/wwwroot:/Users/Johnny/wwwroot"
            - "./php/php-fpm.d:/usr/local/etc/php-fpm.d"
            - "./php/logs:/usr/local/etc/logs"
        networks:
            - webnet

    etcd-browser:
        build: ./etcd-browser 
        ports:
            - "8000:8000"
        environment:
            ETCD_HOST: 10.10.134.33
            ETCD_PORT: 2379
        networks:
            - webnet

    mysql:
        build: ./mysql 
        ports:
            - "3306:3306"
        volumes:
            - "./mysql-data:/var/lib/mysql"
        environment:
            MYSQL_ROOT_PASSWORD: 123456
        networks:
            - webnet

    adminer:
        image: adminer
        restart: always
        ports:
            - 8080:8080

    redis:
        build: ./redis 
        ports:
            - "6379:6379"
        networks:
            - webnet

    memcached:
        build: ./memcached
        ports:
            - "11211:11211"
        networks:
            - webnet

networks:
    webnet:
```
###### docker-compose.yaml文件存在的意义和相关配置含义 
我们通过docker-compose.yaml文件就可以直接为一个docker应用定义它的相关服务、网络、和容器卷，可以非常方便的就为我们搭建起多个容器的应用。所以，用docker-compose来构建我们的lnmp环境就在合适不过。配置的相关含义：
```
version: '3'
services: 
    mysql:
        build: ./mysql 
        ports:
            - "3306:3306"
        volumes:
            - "./mysql-data:/var/lib/mysql"
        environment:
            MYSQL_ROOT_PASSWORD: 123456
        networks:
            - webnet
```
- version: 3  全局配置，表示的是当前yaml文件格式版本为3，你也可以指定为1，或者2，因为docker引擎在不断的升级，每一次发布机会都有新的特性和新的API，所以我们通过指定Yaml文件版本，可以指定当前配置适用于docker的版本
- services:  是一个对象，用以定义服务列表，可以定义多个
- mysql:  指定的服务名称，这个名称可以随便取
- build: ./mysql 构建指定的服务，可以是一个对象，如果直接是一个字符串的话，则指定的是构建目录，目录可以是相对路径和绝对路径，相对路径是基于docker-compose.yaml文件的目录, 如果build定义的是一个对象的话，可以像这样：
```
 mysql:
    build:
      context: ./mysql  //定义构建目录
      dockerfile: Dockerfile-alternate   //可选的参数，用以指定另外的Dockerfile，默认是用context目录下的Dockerfile
      args:  //可选的参数，设置构建参数
        buildno: 1
```
- ports:  //指定端口映射关系80->80，可以指定多组的端口映射
  - 80:80
  - 9000:9000
- volumes:  //本地目录./mysql-data挂载为docker容器中的/var/lib/mysql，可以挂载多组
- networks: //指定加入的网络名称，多个容器可以共享一个网络, 还可以设置网络的别名，如下，在相同网络下的其他容器，可以使用其别名进行网络连接: 如下： 
    webnet:
```
- networks: 
    webnet:
      aliases:
       - databases
	legacy:
      aliases:
       - mysql
```
- environment:  //设置容器启动的时候环境变量
   MYSQL_ROOT_PASSWORD: 123456
- 其他还有很多参数，可以参考文档，这里列举的只是几个基础的参数


#### 这个是我使用docker构建lnmp的目录结构
```
├── docker-compose.yaml
├── etcd-browser
│   ├── Dockerfile
│   ├── LICENSE
│   ├── README.md
│   ├── etcd-browser.service
│   ├── frontend
│   └── server.js
├── memcached
│   └── Dockerfile
├── mysql
│   └── Dockerfile
├── mysql-data
│   ├── auto.cnf
│   ├── ca-key.pem
│   ├── ca.pem
│   ├── client-cert.pem
│   ├── client-key.pem
│   ├── ib_buffer_pool
│   ├── ib_logfile0
│   ├── ib_logfile1
│   ├── ibdata1
│   ├── ibtmp1
│   ├── mysql
│   ├── performance_schema
│   ├── private_key.pem
│   ├── public_key.pem
│   ├── server-cert.pem
│   ├── server-key.pem
│   ├── sys
│   ├── test
│   └── xfile
├── nginx
│   ├── Dockerfile
│   ├── start.sh
│   └── volume-nginx
├── php
│   ├── Dockerfile
│   ├── logs
│   └── php-fpm.d
└── redis
    └── Dockerfile
```
#### docker-compose的一些常用命令 
列出启动的服务
```
▶ docker-compose ps
        Name                       Command               State            Ports
-----------------------------------------------------------------------------------------
docker_adminer_1        entrypoint.sh docker-php-e ...   Up      0.0.0.0:8080->8080/tcp
docker_etcd-browser_1   nodejs server.js                 Up      0.0.0.0:8000->8000/tcp
docker_memcached_1      docker-entrypoint.sh memcached   Up      0.0.0.0:11211->11211/tcp
docker_mysql_1          docker-entrypoint.sh mysqld      Up      0.0.0.0:3306->3306/tcp
docker_nginx_1          nginx -g daemon off;             Up      0.0.0.0:80->80/tcp
docker_php_1            docker-php-entrypoint php-fpm    Up      0.0.0.0:9000->9000/tcp
docker_redis_1          docker-entrypoint.sh redis ...   Up      0.0.0.0:6379->6379/tcp
```
重新启动指定的某个服务
```
▶ docker-compose restart nginx
Restarting docker_nginx_1 ... done
```
构建服务, 我这是都已经构建好了的，build默认使用的是当前目录下的docker-compose.yml或者是docker-compose.yaml，我们可以可以使用-f参数指定docker-compose的配置文件
```
▶ docker-compose build
Building nginx
Step 1/1 : FROM nginx:latest
 ---> 40960efd7b8f
Successfully built 40960efd7b8f
Successfully tagged docker_nginx:latest
Building php
…………
………………
………………………
```
进入到某个已经运行的docker容器里面,
```
▶ docker-compose exec nginx bash
root@daeafec686cf:/# ls -a
.  ..  .dockerenv  Users  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```
打印容器运行的日志
```
▶ docker-compose logs -f nginx
Attaching to docker_nginx_1
```
还有一些其他的命令，可以参考文档


#### 参考文档
- https://docs.docker.com/compose/compose-file/
- https://github.com/JohnnyWei188/docker-compose-lnmp.git


