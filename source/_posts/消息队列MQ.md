---
title: 消息队列MQ
date: 2023-02-06 15:50:02
tags: [MQ,中间件]
---

# Message Queue

>  MQ是消息中间件，是一种在分布式系统中应用程序借以传递消息的媒介，常用的有ActiveMQ，RabbitMQ(erlang)，kafka。
>
> 使用场景:  异步处理、应用解耦、流量控制 

<!--more-->

- JMS (Java Message Service) JAVA消息服务:

  基于JVM消息代理的规范。ActiveMQ、HornetMQ是JMS实现

- AMQP (Advanced Message Queuing Protocol)

  高级消息队列协议，也是一个消息代理的规范，兼容JMS

  RabbitMQ是AMQP的实现

|              | JMS (Java Message Service)                                   | AMQP (Advanced Message Queuing Protocol)                     |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 定义         | Java api                                                     | 网络线级协议                                                 |
| 跨语言       | 否                                                           | 是                                                           |
| 跨平台       | 否                                                           | 是                                                           |
| Model        | 两种消息模型:<br />(1) Peer-2-Peer<br />(2) Pub/sub          | 提供了五种消息模型:<br />(1) . direct exchange<br />(2). fanout exchange<br />(3). topic change<br/>(4) . headers exchange<br />(5). system exchange<br/>本质来讲．后四种和JMS的pub/sub模型没有太大差别，仅是在路由机制上做了更详细的划分; |
| 支持消息类型 | 多种消息类型:TextMessage, MapMessage, Message.....           | byte[]: 当实际应用时,有复杂的消息,可以将消息序列化后发送。   |
| 综合评价     | JMS定义了JAVA API层面的标准; 在java体系中，多个client均可以通过JMS进行交互，不需要应用修改代码, 但是其对跨平台的支持较差; | AMQP定义了wire-level层的协议标准;天然具有跨平台、跨语言特性。 |

## RabiitMQ

### 核心概念

- Message - 消息

  消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key_(路由键)、priority(相对于其他消息的优先权)、delivery-mode(指出该消息可能需要持久性存储)等。

- Publisher - 消息的生产者

  也是一个向交换器发布消息的客户端应用程序。

- Exchange - 交换器

  用来接收生产者发送的消息并将这些消息路由给服务器中的队列。
  Exchange有4种类型: direct(默认)，fanout, topic,和headers，不同类型的Exchange转发消息的策略有所区别

  ![image-20230206170910912](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20230206170910912.png)

![image-20230206174720569](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20230206174720569.png)

## ActiveMQ 

特点：
1、支持多种语言编写客户端
2、对spring的支持，很容易和spring整合
3、支持多种传输协议：TCP,SSL,NIO,UDP等
4、支持AJAX
消息形式：
1、点对点（queue）
2、一对多（topic）

![](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/20210707211328.png)

![](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/20210707211339.png)

### Java实现

1. 创建连接工厂

```java
public static void main(String[] args) throws JMSException {
        //1.创建连接工厂
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory("admin","admin","tcp://127.0.0.1:61616");
        //2.通过连接工厂，获得连接并启动访问
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        //3.创建会话Session(参数：事务|签收)
        Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
        //4.创建目的地(具体是队列queen还是主题topic)
        Queue queue = session.createQueue("queue01");

    }
```

2. 生产者

```java
    //5.创建消息生产者
    MessageProducer messageProducer = session.createProducer(queue);
    //6.生产消息发送到MQ队列
    for (int i = 1;i<=3;i++){
        TextMessage textMessage =session.createTextMessage("msg--"+i);
        messageProducer.send(textMessage);
    }
    //关闭资源
    messageProducer.close();
    session.close();
    connection.close();
    System.out.printf("******发送完成");
```

3. 消费者:　两种消费者方式

- 同步阻塞方式（receive()）

  订阅者或接收者调用MessageConsumer的receive()方法接收消息，receive方法在能够接收到消息之前(超时之前)将一直阻塞。

- 异步非阻塞方式（监听器onMessage()）

  订阅者或接受者通过调用MessageConsumer的setMessgeLisener(MessageListener listener) 注册一个消息监听器，当消息到达之后，系统自动调用监听器MessageListener的onMessage(Message message)方法。

```Java
//5.创建消息消费者
MessageConsumer messageConsumer = session.createConsumer(queue);
while (true){
    TextMessage textMessage = (TextMessage) messageConsumer.receive();//.receive(timeout)
    if (null!= textMessage){
        System.out.println("****消费者接收到消息："+textMessage.getText());
    }else{
        break;
    }
}
```

### SpringBoot整合

yml配置

```yml
spring:
	activemq:
      broker-url: tcp://127.0.0.1:61616
      user: admin
      password: admin
    jms:
      pub-sub-domain: false 		#false = Queue	true = topic
```

配置类

```java
@Configuration
@EnableJms  //启动消息队列（启动类）
//@EnableJmsScheduling    //开启定投
public class ActiveConfig{
    @Value("${myqueue}")
    private String myQueue;
    @Bean
    public Queue queue(){
        return new ActiveMQQueue(myQueue)
    }
}
```

生产者代码

```java
@Component
public class Queue_Produce{
    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;
    @Autowired
    private Queue queue;//destination

    //消息发送
    public void produceMsg(){
        jmsMessagingTemplate.convertAndSend(queue,"message");
    }
    //间隔定投
    @Scheduked(fixedDelay = 3000)
    public void produceMsgScheduled{
        jmsMessagingTemplate.convertAndSend(queue,"message");
    }
}
```

消费者代码

```java
@Component
public class Queue_Consumer{
    @JmsListener(destination = "${myqueue}")
    public void receive(String message) {
        ....
        System.out.println("消费者接收到消息...");
    }
}
```

### 进阶

https://www.bilibili.com/video/BV164411G7aB?p=44
