---
layout: post
title: "[译]zookeeper概述"
date: 2021-06-28 23:27:24+08:00
categories: work tech
---
[原文链接](https://zookeeper.apache.org/doc/r3.7.0/zookeeperOver.html)

**-----分割线-----**

## ZooKeeper 概述
[toc]

---
### ZooKeeper 一种针对分布式程序的分布式协同服务

ZooKeeper是一种分布式的，开源的分布式程序协调服务。他提供了一个简单原语的集合，分布式程序可以根据他实现服务高层次的同步，配置管理，分组和命名。他被设计为易于编程，使用类似文件系统目录树形结构的数据结构。在java中运行，同时有java和c的绑定。

协调器服务臭名昭著的难以做对。特别容易导致竞速条件和死锁。ZooKeeper背后的动机是减少分布式系统从头实现协调器服务。

### 设计目标

**ZooKeeper是简单的。**ZooKeeper允许分布式进程通过一个共享的层次分明的命名空间相互协调，这个共享的层次分明的命名空间类似标准的文件系统。命名空间包含数据寄存器-成为 - znodes，在ZooKeeper的说法中 - 类似于文件和目录。不同于典型的文件系统被设计为存储，ZooKeeper的数据存储在内存中，这意味着ZooKeeper可以获得高吞吐量和低延迟。

ZooKeeper的实现注重高性能，高可用，严格有序访问。ZooKeeper性能方向的表现意味着他可以用作大型的，分布式系统。可用性方面保证其不会陷入单点故障。严格有序意味着复杂同步原语可以在客户端实现。

**ZooKeeper**是主从结构。类似于他所协调的分布式进程，ZooKeeper本身旨在通过一组成为集合(ensemble)的主机进行复制。
![ZooKeeper ensemble](https://zookeeper.apache.org/doc/r3.7.0/images/zkservice.jpg)
组成ZooKeeper服务的各个服务必须都互相知道。他们维护了一个内存的状态镜像，以及持久存储中的事务日志和快照。只要大多数服务器可用，ZooKeeper的服务就可用。

客户端链接一个单独的ZooKeeper服务。这个客户端维护了一个TCP链接，这个TCP链接发送请求，得到回应，得到监视事件，发送心跳。如果这个tcp断掉，他将链接一个不同的服务。

**ZooKeeper是有序的。**ZooKeeper用一个反映所有ZooKeeper事务的数字来标记每次更新。随后的操作可以使用这个顺序去实现高层次的抽象，比如同步原语。

**ZooKeeper是快速的。**他在"读取主导(read-dominant)"工作集中特别快。ZooKeeper服务运行在数以千计的机器上，他在读取比写入更频繁的场景下表现最好，比率大概10:1。


### 数据结构以及分层命名空间

ZooKeeper提供的命名空间和标准文件系统非常像。名字是被斜线(/)分隔的路径元素的一个序列。每个ZooKeeper的命名空间被一个路径标志。

ZooKeeper的分层命名空间。
![ZooKeeper's Hierarchical namespace](https://zookeeper.apache.org/doc/r3.7.0/images/zknamespace.jpg)

### 节点和临时节点

不同于标准的文件系统，ZooKeeper命名空间的每个节点可以附带数据以及子节点。就像是一个文件系统，允许文件作为目录。(ZooKeeper被设计为存储协调数据：状态信息，配置，地点信息等，因此存在于每个节点的信息通常很小，从字节到kb)。我们使用术语*znode*来明确我们正在讨论的ZooKeeper数据节点。


