---
title: Docker的安装使用
date: 2020-11-11 21:12:27
tags: [Docker,Linux]
---

## Docker简介

**Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。**它是目前最流行的 Linux 容器解决方案。

Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。

<!--more-->

## 安装使用

1. Docker要求CentOS系统内核版本高于3.10,查看CentOS内核版本:

   ```shell
   # uname -r
   ```

2. 安装

    ```shell
    # yum install docker
    ```

3. 启动

   ```shell
   # systemctl start docker
   ```

4. 将docer设为开机启动

   ```shell
   # systemctl enable docker
   ```

5. 设置镜像
   - 登录阿里云 ----容器镜像服务-------镜像工具-------镜像加速器

## 常用操作

镜像层是只读的，容器层是可写的。

### 镜像操作

|                             命令                             |       作用       |
| :----------------------------------------------------------: | :--------------: |
|                     docker search 关键字                     |       检索       |
|                    docker pull 镜像名:tag                    |       拉取       |
|                        docker images                         | 查询已下载的镜像 |
|                     docker rmi  image-id                     |     删除镜像     |
| docker commit -m="description" -a="author" container-id image-name:tag-name |     制作镜像     |

### 容器操作

|                             命令                             |                             作用                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| docker run --name container-name -d image-name\|\|eg:docker run --name mytomcat -d tomcat:latest | --name:自定义容器名\|\|-d:后台运行\|\|image-name:指定镜像模板 |
|                          docker ps                           |            查看运行中的容器\|\|-a可以查看所有容器            |
|           docker stop container -name/container-id           |                           停止容器                           |
|          docker start container -name/container-id           |                           启动容器                           |
|                    docker rm container-id                    |                           删除容器                           |
|                         -p 6379:6379                         |                    主机端口映射到容器端口                    |
|           docker logs container-name/container-id            |                         查看容器日志                         |
|    docker exec -it 容器ID /bin/bash------attach(exit退出)    |                         进入new容器                          |
|                        exit/Ctrl+P+Q                         |                           退出容器                           |

## 容器卷

| 命令                                                         | 作用           |
| ------------------------------------------------------------ | -------------- |
| docker run -v /dir/dir:/dir/dir:rw[ro]                       | 挂载           |
| docker inspect container-id                                  | 查看是否挂载上 |
| docker run -it --privileged=true --volumes-from u1 --name u2 ubuntu | 从u1继承卷到u2 |

## 高级篇



## 镜像创建

### MySQL

```shell
$ docker run -p 3306:3306 --name mysql \
 -v /mydata/mysql/conf:/etc/mysql \
 -v /mydata/mysql/log:/var/log/mysql \
 -v /mydata/mysql/data:/var/lib/mysql \
 -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
 
 $ docker run -p 3306:3306 --name mysql \
 -v /mydata/mysql/conf:/etc/mysql/conf.d \
 -v /mydata/mysql/log:/var/log \
 -v /mydata/mysql/data:/var/lib/mysql \
 --privileged=true \
 -e MYSQL_ROOT_PASSWORD=root -d mysql:8.0 \
 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode-ci
 
 
# 数据挂载:docker rm 将容器删除那么数据就丢失了,将容器中的目录挂载到宿主机，以保证数据的安全
```

说明:

 -p 3306:3306	将容器的3306端口映射到主机的3306端口

 -e MYSQL_ROOT_PASSWORD=密码		初始化root用户密码

 -d 后台启动

 --privileged=true：容器内的root拥有真正root权限，否则容器内root只是外部普通用户权限

 -v /mydata/mysql/conf:/etc/mysql	将配置文件挂载到主机

 -v /mydata/mysql/log:/var/log/mysql	将日志文件挂载到主机

 -v /mydata/mysql/data:/var/lib/mysql	将配置文件挂载到主机

​	

### Redis 

```shell
$ mkdir -p /mydata/redis/conf
$ touch /mydata/redis/conf/redis.conf

$ docker run --name redis -p 6379:6379 \
-v /mydata/redis/data:/data \
-v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
-d redis redis-server /etc/redis/redis.conf

#为避免重启丢失数据
#/mydata/redis/conf/redis.conf 添加 appendonly yes(启用AOF持久化) 新版本默认
```

​	

### ElasticSearch

```shell

$ docker run --name kibana -e ELASTICSEA	RCH_HOSTS=http://172.24.0.1:9200 -p 5601:5601 -d kibana:7.6.2 //命令行指定

$ docker run --name kibana -p 5601:5601 -v E:/mydata/kibana/config/:/usr/share/kibana/config -d kibana:7.6.2	//配置文件指定


$ docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" -v E:/mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v E:/mydata/elasticsearch/data:/usr/share/elasticsearch/data -v E:/mydata/elasticsearch/plugings:/usr/share/elasticsearch/plugins -d elasticsearch:7.6.2
```

### Nginx

```shell
#先创建空的把conf目录移出挂载
$ docker run -p 80:80 --name nginx -d nginx:latest
$ docker container cp nginx:/etc/nginx .
#正式
$ docker run -p 80:80 --name nginx -v E:/mydata/nginx/html:/usr/share/nginx/html -v E:/mydata/nginx/logs:/var/log/nginx -v E:/mydata/nginx/conf.d:/etc/nginx/conf.d -d nginx:latest
```

### RabbitMQ

```shell
$ docker run -d --name rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 rabbitmq:management

```

