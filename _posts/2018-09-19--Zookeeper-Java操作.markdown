---
layout:     post
title:      "Zookeeper的java操作"
subtitle:   "Zookeeper的java操作"
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

- 一、zookeeper的Java操作
  - 1.Java操作的基本流程
  - 2.JavaAPI
    - 2.1  创建节点
    - 2.2  删除节点
    - 2.3  修改节点内容
    - 2.4 查询节点
      - 2.4.1 查询节点内容
      - 2.4.2 查询连接的sessionID
      - 2.4.3 节点的所有子节点
- 二、zookeeper的监控器

<!-- /MarkdownTOC -->

# 一、zookeeper的Java操作

## 1.Java操作的基本流程

0. 设置pom文件,增加依赖

   ```xml
   <!-- zookeeper依赖 -->
   <dependency>
       <groupId>org.apache.zookeeper</groupId>
       <artifactId>zookeeper</artifactId>
       <version>3.4.7</version>
       <type>pom</type>
   </dependency>
   ```


1. 连接服务端

> 首先获取客户端对象,后续对zookeeper的所有操作都是基于此

```java
//创建连接端口的URI
String zkURI = "hadoop1:2181";
//获取客户端对象
ZooKeeper zkCli = new ZooKeeper(zkURI, timeOut, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
                System.out.println("==========" + watchedEvent.getType() + "====" + watchedEvent.getState());
            }
        });
```

2. 操作

例如:创建临时节点

```java
/**
     * 使用给定的zk客户端创建指定名称、内容、类型的znode
     * @param zkCli 给定的zk客户端
     * @param znodeAbsPath 要创建的节点的绝对路径
     * @param znodeContent 要创建的节点的内容
     * @param isEPHEMERAL 节点的类型是否是临时节点
     * @return 创建操作是否成功
     */
    public static boolean createZnode(ZooKeeper zkCli,String znodeAbsPath,String znodeContent,boolean isEPHEMERAL){
        CreateMode znodeType = null;
        if (isEPHEMERAL){
            //创建临时节点
            znodeType = CreateMode.EPHEMERAL;
        }else {
            //创建永久节点
            znodeType = CreateMode.PERSISTENT;
        }
        try {
            String a = zkCli.create(znodeAbsPath,znodeContent.getBytes(),ZooDefs.Ids.OPEN_ACL_UNSAFE,znodeType);
            System.out.println(a);
            return true;
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return false;
    }
```

3. 关闭客户端对象

```java
zkCli.close();
```

## 2.JavaAPI

### 2.1  创建节点

```java
String create(final String path, byte data[], List<ACL> acl, CreateMode createMode)
    //第一个参数: 将要创建的文件的绝对路径
    //第二个参数: 节点内容的字节数组
    //第三个参数: ZooDefs.Ids.OPEN_ACL_UNSAFE  表示开放权限
    //第四个参数: 创建模式: 临时节点  CreateMode.EPHEMERAL
    //           永久节点  CreateMode.PERSISTENT
或:
void create(final String path, byte data[], List<ACL> acl,CreateMode createMode,  StringCallback cb, Object ctx)
```

例如:

```java
public static boolean createZnode(ZooKeeper zkCli,String znodeAbsPath,String znodeContent,boolean isEPHEMERAL){
        CreateMode znodeType = null;
        if (isEPHEMERAL){
            znodeType = CreateMode.EPHEMERAL;
        }else {
            znodeType = CreateMode.PERSISTENT;
        }
        try {
            String a = zkCli.create(znodeAbsPath,znodeContent.getBytes(),ZooDefs.Ids.OPEN_ACL_UNSAFE,znodeType);
            System.out.println(a);

            return true;
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return false;
    }
```



### 2.2  删除节点

```java
public void delete(final String path, int version)
    //第一个参数:  将要删除的文件的绝对路径
    //第二个参数:  删除的版本(节点信息中的dataVersion,-1是最后一次更改信息)
```

### 2.3  修改节点内容

```java
Stat setData(final String path, byte data[], int version)
  //第一个参数:  将要修改的文件的就对路径
    //第二个参数:  修改内容的字节数组
    //第三个参数:  修改的版本(节点信息中的dataVersion,-1是最后一次更改信息)
    //返回值:    返回修改后文件的节点信息
```

**getData()方法仅仅监控对应节点的一次数据变化，无论是数据修改还是删除！若要每次对应节点发生变化都被监测到，那么每次都得先调用getData()方法获取一遍数据！**

### 2.4 查询节点

#### 2.4.1 查询节点内容

```java
byte[] getData(String path, boolean watch, Stat stat)
    //第一个参数:  将要修改的文件的就对路径
    //第二个参数:  如果设为true,则下一次修改节点会触发watcher
    //第三个参数:  设置接收将要查看文件的节点信息
    //返回值:    节点内容的字节数组
byte[] getData(final String path, Watcher watcher, Stat stat)
    //第一个参数:  将要修改的文件的就对路径
    //第二个参数:  设置监控器
    //第三个参数:  设置接收将要查看文件的节点信息
    //返回值:    节点内容的字节数组
    
```

#### 2.4.2 查询连接的sessionID

```java
zkCli.getSessionId()
```

#### 2.4.3 节点的所有子节点

```java
List<String> childrens = zkCli.getChildren(znodeAbsPath, false);
for (int i = 0; i < childrens.size(); i++) {
                System.out.println(childrens.get(i));
}
```

# 二、zookeeper的监控器

> 可以对任意节点进行监控

```
package org.qianfeng.bigdata.zookeeper;

import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;

/**
 * @description zookeeper监控者
 * @author: Mmy mamingyume@foxmail.com
 * @create: 2018-09-19 12:02:57
 **/
public class ZKWatcher implements Watcher {

    @Override
    public void process(WatchedEvent watchedEvent) {
//        五种事件类型
//        None(-1),
//        NodeCreated(1),
//        NodeDeleted(2),
//        NodeDataChanged(3),
//        NodeChildrenChanged(4);

        if (watchedEvent.getType().getIntValue()==1){
            System.out.println(watchedEvent.getPath() + "节点增加了");
        }

        if (watchedEvent.getType().getIntValue()==2){
            System.out.println(watchedEvent.getPath() + "节点删除了~~~~~~~~~~~");
        }

        if (watchedEvent.getType().getIntValue()==3){
            System.out.println(watchedEvent.getPath() + "节点内容发生变化了");
        }

        if (watchedEvent.getType().getIntValue()==4){
            System.out.println(watchedEvent.getPath() + "节点的子节点发生变化了");
        }
    }
}
```

**实例:**

...

当数据改变的时候，那么一个Watch事件会产生并且被发送到客户端中。但是客户端只会收到一次这样的通知，如果以后这个数据再次发生改变的时候，之前设置Watch的客户端将不会再次收到改变的通知，因为Watch机制规定了它是一个一次性的触发器。