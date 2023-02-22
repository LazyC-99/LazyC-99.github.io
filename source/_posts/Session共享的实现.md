---
title: Session共享的实现
date: 2020-12-01 14:40:47
tags:
---

### Session共享

 Session 共享是指在一个浏览器访问多个 Web 服务时，服务端的 Session 数据需要共享。

<!--more-->

#### cookie加密的方式保存在客户端

- 优点：减轻服务器端压力

- 缺点：受到cookie大小限制，因为每次请求会在头部附带cookie信息，占用一定的带宽。另外，这种方式在用户禁用cookie的情况下无效。这种方式不常用。

#### 服务器间同步

服务器间同步比如tomcat集群：通过配置tomcat，实现session共享。每个tomcat都会在局域网中广播自己的session信息，同时监听其他tomcat广播的session，一旦自己的session发生变化，其他的tomcat能够感知到的，同时就可以同步自己的session和它一样。

- 缺点：当集群服务器数量比较大如200台，每一台服务器的tomcat都需要广播自己的session，同时监听另外199台，此时，服务器的大量资源都用来处理session同步的事情，用户正常的访问就会受到影响。要视部署的tomcat集群数量等来定是否使用这种方式。

#### 基于分布式缓存

基于分布式缓存的session共享机制

使用redis取代session保存用户信息，这种方式比较常用

### SpringSession

Spring Session 是 Spring 的项目之一。Spring Session 提供了一套创建和管理 Servlet HttpSession 的方案，默认采用外置的 Redis 来存储 Session 数据，以此来解决 Session 共享的 问题。

#### 基本实现原理

SpringSession从底层全方位"接管"了Tomcat对Session的管理

![SpringSession基本实现原理](https://gitee.com/LazyC-99/blog-images/raw/master/img/20201201155030.png)

##### SessionRepositoryFilter

利用Filter原理,在每次请求到达目标方法之前,将原生HttpSetvletRequest / HttpSetvletResponse对象包装为SessionRepository / ResponseWrapper 