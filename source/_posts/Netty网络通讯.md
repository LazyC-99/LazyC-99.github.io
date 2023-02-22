---
title: Netty网络通讯
date: 2022-08-02 16:03:23
tags: [IO,网络]
---

## Netty网络通讯

### IO模型

- BIO(传统阻塞IO): 服务器实现模式为一个连接一个线程

  ![image-20220802163623607](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20220802163623607.png)

- NIO(同步非阻塞IO): 面向缓冲区(块)编程，服务器实现模式为一个线程处理多个请求(连接),请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求就进行处理

![image-20220802163844851](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20220802163844851.png)

1. 每个channel都会对应一个Bufer
2. Selector 对对应一个线程，一个线程对应多个channel(连接)
3. 该图反应了有三个channel注册到该selector 
4. 程序切换到哪个channel是有事件决定的,Event是一个重要的概念
5. Selector会根据不同的事件，在各个通道上切换
6. Buffer就是一个内存块,底层是有一个数组
7. 数据的读取写入是通过Buffer，BIO中要么是输入流，或者是输出流，不能双向，NIO的Buffer是可以读也可以写，需要flip()方法切换
8. channel是双向的，可以返回底层操作系统的情况，比如Linux，底层的操作系统通道就是双向的.

### 缓冲区(Buffer)

| 属性     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| Capacity | 容量:即可以容纳的最大数据量;在缓吊区创建时被设并且不能改变   |
| Limit    | 表示缓冲区的当前终点，不能对缓冲区超过极限的位置迸行读写操作。且极限是可修改的 |
| Position | 位置,下一个要被读或写的元素的索引,每次读亨缓冲区数据时都会改变改值,为下茨读写作准备 |
| Mark     | 标记                                                         |

NIO还提供了MappedByteBuffer，可以让文件直接在内存（堆外的内存）中进行修改，而如何同步到文件由NIO来完成

### 通道(Channel)

1. NIO的通道类似于流，但有些区别如下:

   - 通道可以同时进行读写，而流只能读或者只能写·通道可以实现异步读写数据

   - 通道可以从缓冲读数据，也可以写数据到缓冲
2. Channel在NIO中是一个接口
3. 常用Channel类有: FileChannel, DatagramChannel, ServerSocketChannel, SocketChannnel
4. FileChannel用于文件数据读写, DatagramChannel 用于UDP的数据读写, ServerSocketChannel, SocketChannnel用于TCP的数据读写

- 文件读写

 ![image-20220805163842676](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20220805163842676.png)

- 文件复制

![image-20220805163939115](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20220805163939115.png)

5. NIO还支持通过多个Buffer(即 Buffer数组)完成读写操作，即Scattering 和 Gathering
   - Scattering : 将数据写入到buffer时，可以采用buffer数组，依次写入
   - Gathering : 从buffer读取数据时，可以采用buffer数组，依次读

### 选择器(Selector)

1. Selector能够检测多个注册的通道上是否有事件发生，如果有事件发生，便获取事件然后针对每个事件进行相应的处理。实现一个单线程管理多个通道，也就是管理多个连接和请求。
2. 只有在 连接/通道 真正有读写事件发生时，才会进行读写，大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程, 避免了多线程之间的上下文切换导致的开销
3. 

