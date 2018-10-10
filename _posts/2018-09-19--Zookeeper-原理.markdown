---
layout:     post
title:      "Zookeeper原理详解"
subtitle:   "Zookeeper原理详解"
date:       2018-09-19 12:00:00
author:     "Chamber"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - hadoop
    - zookeeper
    - HDFS
    - HA高可用
---
<!-- MarkdownTOC -->

- 一、zookeeper概述
  - 1.1  什么是zookeeper
  - 1.2 zookeeper服务的应用场景
  - 1.3 zookeeper集群特性
  - 1.4 zookeeper数据结构
  - 1.5节点类型
  - 1.6 Zookeeper 数据结构特点
  - 1.7 如何使用
- 二、zookeeper集群的配置
  - 2.1 集群安装
  - 2.2 集群的启动
  - 2.3 集群的测试使用

<!-- /MarkdownTOC -->

# 一、zookeeper概述

## 1.1  什么是zookeeper

- Zookeeper是一个分布式协调服务；就是为用户的分布式应用程序提供协调服务

- zookeeper是为别的分布式程序服务的

- Zookeeper本身就是一个分布式程序（只要有半数以上节点存活，zk就能正常服务）

  - Zookeeper集群的角色：  Leader 和  follower  （Observer）

- zookeeper在底层最核心的两个功能：

  - 管理(存储，读取)用户程序提交的数据
  - 并为用户程序提供数据节点监听服务

## 1.2 zookeeper服务的应用场景

- 主从协调
- 服务器节点动态上下线
- 统一配置管理
- 分布式共享锁
- 统一名称服务等等

## 1.3 zookeeper集群特性

- Zookeeper：一个leader，多个follower组成的集群
- 全局数据一致：每个server保存一份相同的数据副本，client无论连接到哪个server，数据都是一致的
- 分布式读写，更新请求转发，由leader实施
- 更新请求顺序进行，来自同一个client的更新请求按其发送顺序依次执行
- 数据更新原子性，一次数据更新要么成功，要么失败
- 实时性，在一定时间范围内，client能读到最新数据

## 1.4 zookeeper数据结构

- 层次化的目录结构，命名符合常规文件系统规范

- 每个节点在zookeeper中叫做znode,并且其有一个唯一的路径标识

- 节点Znode可以包含数据和子节点（但是EPHEMERAL类型的节点不能有子节点）

- 客户端应用可以在节点上设置监视器

## 1.5节点类型

- Znode有两种类型：

  - 短暂（ephemeral）（断开连接自己删除）
  - 持久（persistent）（断开连接不删除）

- Znode有四种形式的目录节点（默认是persistent ）

  - PERSISTENT  持久类型
  - PERSISTENT_SEQUENTIAL（持久序列类型/test0000000019 ）sequential
  - EPHEMERAL  短暂类型
  - EPHEMERAL_SEQUENTIAL 短暂序列类型

     创建znode时设置顺序标识，znode名称后会附加一个值，顺序号是一个单调递增的计数器，由父节点维护

- 在分布式系统中，顺序号可以被用于为所有的事件进行全局排序，这样客户端可以通过顺序号推断事件的顺序

## 1.6 Zookeeper 数据结构特点

1. 每个子目录项如 NameService 都被称作为 znode，这个 znode     是被它所在的路径唯一标识，如 Server1 这个 znode 的标识为 /NameService/Server1
2. znode 可以有子节点目录，并且每个 znode 可以存储数据，注意 EPHEMERAL类型的目录节点不能有子节点目录
3. znode 是有版本的，每个 znode 中存储的数据可以有多个版本，也就是一个访问路径中可以存储多份数据
4. znode 可以是临时节点，一旦创建这个 znode 的客户端与服务器失去联系，这个 znode     也将自动删除，Zookeeper 的客户端和服务器通信采用长连接方式，每个客户端和服务器通过心跳来保持连接，这个连接状态称为 session，如果     znode 是临时节点，这个 session 失效，znode 也就删除了
5. znode 的目录名可以自动编号，如 App1 已经存在，再创建的话，将会自动命名为 App2
6. znode     可以被监控，包括这个目录节点中存储的数据的修改，子节点目录的变化等，一旦变化可以通知设置监控的客户端，这个是 Zookeeper     的核心特性，Zookeeper 的很多功能都是基于这个特性实现的，后面在典型的应用场景中会有实例介绍

## 1.7 如何使用

 	Zookeeper作为一个分布式的服务框架，主要用来解决分布式集群中应用系统的一致性问题，它能提供基于类似于文件系统的目录节点树方式的数据存储，但是 Zookeeper并不是用来专门存储数据的，它的作用主要是用来维护和监控你存储的数据的状态变化。通过监控这些数据状态的变化，从而可以达到基于数据的集群管理，



# 二、zookeeper集群的配置

## 2.1 集群安装

Zookeeper使用java编写，运行在jvm上，所以需要提前安装并配置好好java环境，推荐Oracle jdk1.7及以上版本。

- [ ] 官网：http://zookeeper.apache.org/

- [ ] 下载地址：http://apache.opencas.org/zookeeper/  从官方网站上下载tar.gz包

- [ ] 集群规划

  | 主机名（hostname） | 安装软件        | 运行进程       |
  | ------------------ | --------------- | -------------- |
  | min1               | zookeeper-3.4.7 | QuorumPeerMain |
  | min2               | zookeeper-3.4.7 | QuorumPeerMain |
  | min3               | zookeeper-3.4.7 | QuorumPeerMain |


- [ ] 安装步骤

  1. 在min1机器上安装zookeeper-3.4.7

     1. 上传zookeeper-3.4.7.tar.gz

     2. 解压到目录（/usr/local/zookeeper） 

    ```shell
    cd /usr/local/zookeeper
    tar -zvxf  zookeeper-3.4.7.tar.gz -C /usr/local/zookeeper --strip-components 1
    ```

     3. 配置

    4. 添加一个zoo.cfg配置文件

       ```shell
       cd /usr/local/zookeeper/conf
       mv zoo_sample.cfg  zoo.cfg
       #在创建/home/hadoopdata/zkdata文件夹
       mkdir -p /home/hadoopdata/zkdata
       ```

    5. 修改配置文件（zoo.cfg）

       ```shell
       vi zoo.cfg    #添加如下内容
       	
           tickTime=2000
           initLimit=10
           syncLimit=5
           clientPort=2181
           #为zookeeper指定一个工作目录，此目录要手动创建
           dataDir=/home/hadoopdata/zkdata
       
           #指定集群中各个机器之间地址及通信端口
           #3888是选举端口 2888是leader和follower通信端口
           #注意 域名（如min1） 都要在各个机器/etc/hosts文件中配置了
           #在zoo.cfg最后添加如下内容
           server.1=min1:2888:3888
           server.2=min2:2888:3888
           server.3=min3:2888:3888
       
       ```

    6. 在上面server.N对应的主机的“dataDir=/home/hadoopdata/zkdata”目录下**创建一个myid文件，里面内容是N**（      server.1对应的min1里面的myid文件内容为1 

       server.2对应的hadoop2里面的myid文件内容为2 

       server.3对应的hadoop3里面的myid文件内容为3）

       ```shell
       cd /home/hadoopdata/zkdata
       touch myid        #创建一个myid的文件
       echo  "1" > myid  #写入的内容要对应，本机是min1 设置的名称是server.1 因此写入时，要将1写入mydid
       ```

  7. 设置zookeeper日志文件的zookeeper.out存放目录

     ```shell
     vi /usr/local/zookeeper/bin/zkEnv.sh   #在如下位置设置
     ```

     ```sh
     then
         ZOO_LOG_DIR="/home/hadoopdata/zkdata"
     fi
     ```

  8. 将在hadoop1配置好的zk拷贝到hadoop2和hadoop3节点

     ```shell
     scp -r  /usr/local/zookeeper  min2:/usr/local/
     #在min2的机器中修改myid的值
     cd /home/hadoop/develop_env/zookeeper-3.4.7/data
     echo  "2" > myid
     
     scp -r  /home/hadoop/develop_env/zookeeper-3.4.7/  min3:/home/hadoop/develop_env/
     #在min3的机器中修改myid的值
     cd /home/hadoop/develop_env/zookeeper-3.4.7/data
     echo  "3" > myid     
     ```

## 2.2 集群的启动

- 配置环境变量

  ```shell
  export ZK_HOME=/usr/local/zookeeper
  export PATH=$PATH:$ZK_HOME/bin:
  ```

- 分别在三台机器中启动zookeeper

  ```shell
  zkServer.sh start   #启动zookeeper
  ```

- 分别在三台机器中执行jps 显示如下进程，则启动成功

  ```shell
  [root@hadoop1 bin]# jps
  13170 Jps
  11405 QuorumPeerMain
  ```

- 查看zk状态

  ```shell
  zkServer.sh status   #启动zookeeper
  ```

  在三台机器中分别显示如下信息：

  ```shell
  [root@hadoop1 bin]# zkServer.sh status
  JMX enabled by default
  Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
  Mode: follower
  ```

  ```shell
  [root@hadoop2 ~]# zkServer.sh status
  JMX enabled by default
  Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
  Mode: leader
  ```

  ```shell
  [root@hadoop3 ~]# zkServer.sh status
  JMX enabled by default
  Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
  Mode: follower
  ```


## 2.3 集群的测试使用

在任何一台机器上执行如下命令进入zk客户端的shell 

```shell
zkCli.sh  
```

执行完zkcli.sh命令后进入的界面效果如下：

```zookeeper
Welcome to ZooKeeper!
JLine support is enabled
2018-09-19 18:59:14,508 [myid:] - INFO  [main-SendThread(localh                                    ost:2181):ClientCnxn$SendThread@849] - Socket connection establ                                    ished to localhost/127.0.0.1:2181, initiating session
2018-09-19 18:59:14,530 [myid:] - INFO  [main-SendThread(localh                                    ost:2181):ClientCnxn$SendThread@1207] - Session establishment c                                    omplete on server localhost/127.0.0.1:2181, sessionid = 0x165f1                                    65d9f50000, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0]
```



- 通过help命令查看zk客户端的shell命令帮助

  ```zookeeper
  [zk: localhost:2181(CONNECTED) 0] help
  ZooKeeper -server host:port cmd args
          stat path [watch]
          set path data [version]
          ls path [watch]
          delquota [-n|-b] path
          ls2 path [watch]
          setAcl path acl
          setquota -n|-b val path
          history
          redo cmdno
          printwatches on|off
          delete path [version]
          sync path
          listquota path
          rmr path
          get path [watch]
          create [-s] [-e] path data acl
          addauth scheme auth
          quit
          getAcl path
          close
          connect host:port
  
  ```

- zk服务器中也有类似 linux中的目录结构，”/“ 是它的目录根，在这个根下可以创建任何一个key、value对，其中key值就相当于一个linux中子目录名，但是这个子目录是可以有值的，就是那个value。这个key-value对在zk中叫一个node 这个node可以有子孙node。

  - 在根下创建一个node节点key为age， value为18

    ```shell
    create /age 18
    ```

  - 显示“根”下所有node

    ```shell
    ls  /
    ```

  - 获得一个node的value值

    ```shell
    get /age
    ```
