---
title: spring-cloud-netflix使用
date: 2020-11-23 17:01:24
tags: [Spring]
---

### SpringCloud简介

springCloud是基于SpringBoot的一整套实现微服务的框架。他提供了微服务开发所需的配置管理、服务发现、断路器、智能路由、微代理、控制总线、全局锁、决策竞选、分布式会话和集群状态管理等组件。

<!--more-->

### 基本组件

Eureka: 服务注册和发现组件
Ribbon: 负载均衡
Feign: 远程接口的声明式调用
Hystrix: 服务的熔断,降级,监控
![](https://gitee.com/LazyC-99/blog-images/raw/master/img/20201120150542.png)

#### Eureka(注册)

- 服务端配置

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

```yml
server:
	port: 5000
eureka:
  instance:
    hostname: localhost     #配置当前Eureka服务的主机地址
  client:
    register-with-eureka: false     #当前服务本身就是注册中心,不用注册自己
    fetch-registry: false           #不用从注册中心取回信息
    service-url:
      defaultZone: http://${eureka.instance.hostname}/${server.port}/eureka #设置与Eureka交互的地址查询服务和注册服务
```

```java
@EnableEurekaServer//开启服务
public class EurekaMainType {
    public static void main(String[] args) {
        SpringApplication.run(EurekaMainType.class,args);
    }
}
```

- 客户端配置

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:5000/eureka  #Eureka服务地址
  instance:
  	instance-id: xxx		#修改服务名称
  	prefer-ip-address: true		#访问路径显示ip地址
spring:
  application:
    name: xxx
```

```java
@EnableEurekaClient
public class ProviderMainType {
    public static void main(String[] args) {
        SpringApplication.run(ProviderMainType.class,args);
    }
}
```

#### Ribbon(负载均衡)

依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

```java
@Configuration
public class SpringCloudConfig {
    //让RestTemplate具有负载均衡的功能,通过Ribbon访问provider集群
    @LoadBalanced
    @Bean 
    public RestTemplate getRestTemplate () {
        return new RestTemplate();
    }
}
```

#### Feign(声明式调用)

依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>
```

```java
@EnableFeignClients
public class CosumerMainType {
    public static void main(String[] args) {
        SpringApplication.run(CosumerMainType.class,args);
    }
}
```

Feigin依赖有Ribbon, 不需要配置Ribbon

![Feign(声明式调用)](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201120150725105.png)

@FeignClient注解

value:

- 表示当前接口和一个Provider对应,注解中value属性指定要用的Provider的微服务名称
- 要求@RequestMapping注解映射的地址一致
- 要求方法声明一致
- 用来获取请求参数的@RequstParam, @PathVariable, @RequestBody不能省略, 两边一致

fallbackFactory:

- 指定Provider不可用时提供的备用方案的工厂类型

@EnableFeignClients

- 启用Feign客户端功能

#### Hystrix(熔断,降级,监控)

依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

##### 熔断(Provider)

```java
//指定当前方法出问题时调用的方法
@HystrixCommand(fallbackMethod = "MethodName)
@RequestMapping("/")
```

```java
@EnableCircuitBreaker	//开启熔断支持
public class ProviderMainType {
    public static void main(String[] args) {
        SpringApplication.run(ProviderMainType.class,args);
    }
}
```

##### 降级(Consumer)

- 实现Consumer端服务降级功能
- 实现FallbackFactory接口要传入@FeignClient标记的接口类型
- 在create()方法中返回@FeignClient注解标记的接口类型的对象,当Provider调用失败后,会执行这个对象对应的方法
- 使用@Component将当前对象加入IOC容器

```java
@Component	//大坑	
public class MyFallBackFactory implements FallbackFactory<EmployeeRemoteService> {
    @Override
    public EmployeeRemoteService create(Throwable throwable) {
        return new EmployeeRemoteService() {
            @Override
            .....
        };
    }
}
```

启用

在Consumer的配置文件中启用

```yml
feign:
  hystrix:
    enabled: true
```

##### 监控(Provier)

依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

配置

```yml
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```

启动类上加@EnableHystrixDashboard注解启动仪表盘功能

#### Zuul(网关)

添加依赖并使用@EnableZuulProxy开启

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```

```java
@EnableZuulProxy
```

```properties
zuul:
  prefix: /prefix
  ignored-services: 微服务名称/"*"
  routes:
    mydept.serviceId: 微服务名称
    mydept.path: /mydept/**		//映射
```



访问: http://127.0.0.1:zuul端口 / 微服务名称 / 具体功能地址

**ZuulFilter过滤类**

`shouldFilter`：返回一个boolean类型来判断该过滤器是否要执行，所以通过此函数可实现过滤器的开关。

`filterType`：返回一个字符串代表过滤器的类型，在zuul中定义了四种不同生命周期的过滤器类型，具体如下：

- `pre`：可以在请求被路由之前调用
- `route`：在路由请求时候被调用
- `post`：在route和error过滤器之后被调用
- `error`：处理请求时发生错误时被调用

`filterOrder`：通过int值来定义过滤器的执行顺序

`run`：过滤器的具体实现功能。

#### SpringCloud Config

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

```properties

#Server端
spring:
  application: 
    name: xx
  cloud:
    config:
      server:
        git:
          uri: git@github.com:/xxxx/git		#git配置文件仓库
          
#==============================================================
#Client端 
#创建bootstrap.yml 系统级配置文件
spring:
  cloud:
    config:
      name: xxxx
      profile: dev
      lable: master
      uri: http://config.com	#SpringCloud Config服务的地址 
      
#创建application.yml 系统级配置文件
spring:
  cloud:
    config:
      name: xxxx		#名字一致
```

```java
@EnableConfigServer
```



### 遇到的问题

##### fallback method wasn't found

 备用方法 和 原方法 的参数类型，个数不同造成的

##### Unable to connect to Command Metric Stream.

dashboard无法显示,控制台显示http://xxx.xxx.xx:xxx/hystrix.stream is not in the allowed list of proxy host names.  If it should be allowed add it to hystrix.dashboard.proxyStreamAllowList. 

解决: 配置

```yml
hystrix:
  dashboard:
    proxy-stream-allow-list: "localhost"
```

