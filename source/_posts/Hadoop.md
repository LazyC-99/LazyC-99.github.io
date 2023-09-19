---
title: Hadoop
date: 2023-09-15 14:47:14
tags: [Hadoop,Java]
---

# Hadoop

> Hadoop 框架是用于计算机集群大数据处理的框架，所以它必须是一个可以部署在多台计算机上的软件。部署了 Hadoop 软件的主机之间通过套接字 (网络) 进行通讯。

Hadoop 主要包含 HDFS 和 MapReduce 两大组件，HDFS 负责分布储存数据，MapReduce 负责对数据进行映射、规约处理，并汇总处理结果。

## HDFS 

Hadoop Distributed File System，Hadoop 分布式文件系统，简称 HDFS。

HDFS 用于在集群中储存文件，它所使用的核心思想是 Google 的 GFS 思想，可以存储很大的文件。

### 命名节点 (NameNode)

- NameNode仅存储HDFS的元数据︰文件系统中所有文件的目录树，并跟踪整个集群中的文件，不存储实际数据。
- NameNode知道HDFS中任何给定文件的块列表及其位置。使用此信息NameNode知道如何从块中构建文件。
- NameNode不持久化存储每个文件中各个块所在的datanode的位置信息，这些信息会在系统启动时从DataNode重建。NameNode是Hadoop集群中的单点故障。
- NameNode所在机器通常会配置有大量内存( RAM )。

### 数据节点 (DataNode)

### 副命名节点 (Secondary NameNode)

## MapReduce 
