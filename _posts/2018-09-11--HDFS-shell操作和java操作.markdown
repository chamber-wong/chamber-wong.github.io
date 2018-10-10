---
layout:     post
title:      "HDFS的shell操作和Java操作"
subtitle:   ""
date:       2018-09-11 12:00:00
author:     "Chamber"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - hadoop
    - HDFS
---
<!-- MarkdownTOC -->

- 一、hdfs的原理
  - **1、Block数据**
  - **2、NameNode**
  - **3、Secondary NameNode**
  - **4、DataNode**
- 二、hdfs的操作
  - 1、写文件
    - 1.1、流程,原理
    - **1.2、Rack aware（机架感知）**
  - 2、读文件
  - 3、HDFS-可靠性
- 三、hdfs的进程启动，停止
  - 0、格式化
  - 1、启动相关
  - 2、关闭相关
- 四、hdfs的shell操作
  - 1、命令行参数列表
  - 2、常用命令和命令案例
  - 3、Hadoop回收站trash
- 五、Java操作hdfs
  - 1、使用maven创建一个maven工程
    - 0、安装maven
    - 1、配置pom.xml文件
  - 2.Java操作HDFS对象
    - 1.基本操作流程
    - 2.获取根目录下所有文件
    - 3.创建目录
    - 4.删除目录
    - 5.上传文件
    - 6.读取文件
    - 7.按行读取文件
    - 8.删除文件

<!-- /MarkdownTOC -->

# 一、hdfs的原理

------

![hdfs](https://7n.w3cschool.cn/attachments/image/wk/hadoop/hdfs-architecture.png)

## **1、Block数据**

1. 基本存储单位，一般大小为64M(2.x之后的默认为128M)（配置大的块主要是因为：1）减少搜寻时间，一般硬盘传输速率比寻道时间要快，大的块可以减少寻道时间；2）减少管理块的数据开销，每个块都需要在NameNode上有对应的记录；3）对数据块进行读写，减少建立网络的连接成本）
2. 一个大文件会被拆分成一个个的块，然后存储于不同的机器。如果一个文件少于Block大小，那么实际占用的空间为其文件的大小
3. 基本的读写S#x5355;位，类似于磁盘的页，每次都是读写一个块
4. 每个块都会被复制到多台机器，默认复制3份

## **2、NameNode**

1. 存储文件的metadata，运行时所有数据都保存到内存，整个HDFS可存储的文件数受限于NameNode的内存大小
2. 一个Block在NameNode中对应一条记录（一般一个block占用150字节），如果是大量的小文件，会消耗大量内存。同时map task的数量是由splits来决定的，所以用MapReduce处理大量的小文件时，就会产生过多的map task，线程管理开销将会增加作业时间。处理大量小文件的速度远远小于处理同等大小的大文件的速度。因此Hadoop建议存储大文件
3. 数据会定时保存到本地磁盘，但不保存block的位置信息，而是由DataNode注册时上报和运行时维护（NameNode中与DataNode相关的信息并不保存到NameNode的文件系统中，而是NameNode每次重启后，动态重建）
4. NameNode失效则整个HDFS都失效了，所以要保证NameNode的可用性

## **3、Secondary NameNode**

1. 定时与NameNode进行同步（定期合并文件系统镜像和编辑日&#x#x5FD7;，然后把合并后的传给NameNode，替换其镜像，并清空编辑日志，类似于CheckPoint机制），但NameNode失效后仍需要手工将其设置成主机

## **4、DataNode**

1. 保存具体的block数据
2. 负责数据的读写操作和复制操作
3. DataNode启动时会向NameNode报告当前存储的数据块信息，后续也会定时报告修改信息
4. DataNode之间会进行通信，复制数据块，保证数据的冗余性

# 二、hdfs的操作

## 1、写文件

### 1.1、流程,原理

![write](https://7n.w3cschool.cn/attachments/image/wk/hadoop/hdfs-write.png)

1.客户端将文件写入本地磁盘的N#x4E34;时文件中

2.当临时文件大小达到一个block大小时，HDFS client通知NameNode，申请写入文件

3.NameNode在HDFS的文件系统中创建一个文件，并把该block id和要写入的DataNode的列表返回给客户端

4.客户端收到这些信息后，将临时文件写入DataNodes

- 4.1 客户端将文件内容写入第一个DataNode（一般以4kb为单位进行传输）
- 4.2 第一个DataNode接收后，将数据写入本地磁盘，同时也传输给第二个DataNode
- 4.3 依此类推到最后一个DataNode，数据在DataNode之间是通过pipeline的方式进行复制的
- 4.4 后面的DataNode接收完数据后，都会发送一个确认给前一个DataNode，最终第一个DataNode返回确认给客户端
- 4.5 当客户端接收到整个block的确认后，会向NameNode发送一个最终的确认信息
- 4.6 如果写入某个DataNode失败，数据会继续写入其他的DataNode。然后NameNode会找另外一个好的DataNode继续复制，以保证冗余性
- 4.7 每个block都会有一个校验码，并存放到独立的文件中，以便读的时候来验证其完整性

5.文件写完后（客户端关闭），NameNode提交文件（这时文件才可见，֘#x5982;果提交前，NameNode垮掉，那文件也就丢失了。fsync：只保证数据的信息写到NameNode上，但并不保证数据已经被写到DataNode中）

### **1.2、Rack aware（机架感知）**

通过配置文件指定机架名和DNS的对应关系

假设复制参数是3，在写入文件时，会在本地的机架保存一份数据，然后在另外一个机架内保存两份数据（同机架内的传输速度快，从而提高性能）

整个HDFS的集群，最好是负载平衡的，这样才能尽量利用集群的优势

## 2、读文件

![read](https://7n.w3cschool.cn/attachments/image/wk/hadoop/hdfs-read.png)

1. 客户端向NameNode发送读取请求
2. NameNode#x8FD4;回文件的所有block和这些block所在的DataNodes（包括复制节点）
3. 客户端直接从DataNode中读取数据，如果该DataNode读取失败（DataNode失效或校验码不对），则从复制节点中读取（如果读取的数据就在本机，则直接读取，否则通过网络读取）

## 3、HDFS-可靠性

1. DataNode可以失效

   DataNode会定时发送心跳到NameNode。如果ղ#x5728;一段时间内NameNode没有收到DataNode的心跳消息，则认为其失效。此时NameNode就会将该节点的数据（从该节点的复制节点中获取）复制到另外的DataNode中

2. 数据可以毁坏

   无论是写入时还是硬盘本身的问题，只要数据有问题（读取时通过校验码来检测），都可以通过其他的复制节点读取，同时还会再复制一份到健康的节点中

3. NameNode不可靠

# 三、hdfs的进程启动，停止

## 0、格式化

```shell
hadoop namenode -format
```

格式化文件系统

## 1、启动相关

```shell
start-dfs.sh
#启动hdfs服务
start-yarn.sh
#启动yarn服务
start-all.sh
#启动hafs和yarn服务

./sbin/hadoop-deamon.sh start [namenode|...]
#单独启动服务
```

## 2、关闭相关

```shell
stop-dfs.sh
#停止hdfs服务
stop-yarn.sh
#停止yarn服务
stop-all.sh
#停止hdfs和yarn服务

./sbin/hadoop-deamon.sh stop [namenode|...]
#单独停止服务
```



------

# 四、hdfs的shell操作

基本格式:

```shell
[hadoop@min1 ~]$ hadoop fs [-参数] [-URL] [路径]
    url指的是hdfs的地址:例如:hdfs://hadoop1:9000/input
    路径也可以是hdfs的地址.
```

## 1、命令行参数列表

> 基本和shell命令相同

​   [-appendToFile <localsrc> ... <dst>]

​        [-cat [-ignoreCrc] <src> ...]

​        [-checksum <src> ...]

​        [-chgrp [-R] GROUP PATH...]

​        [-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]

​        [-chown [-R][OWNER][:[GROUP]] PATH...]

​        [-copyFromLocal [-f][-p] <localsrc> ..

​        [-copyToLocal [-p][-ignoreCrc] [-crc] <src> ... <localdst>]

​        [-count [-q] <path> ...]

​        [-cp [-f][-p] <src> ... <dst>]

​        [-createSnapshot <snapshotDir> [<snapshotName>]]

​        [-deleteSnapshot <snapshotDir> <snapshotName>]

​        [-df [-h][ ...]]

​        [-du [-s][-h] <path> ...]

​        [-expunge]

​        [-get [-p][-ignoreCrc] [-crc] <src> ... <localdst>]

​        [-getfacl [-R] <path>]

​        [-getmerge [-nl] <src> <localdst>]

​        [-help [cmd ...]]

​        [-ls [-d][-h] [-R][ ...]]

​        [-mkdir [-p] <path> ...]

​        [-moveFromLocal <localsrc> ... <dst>]

​        [-moveToLocal <src> <localdst>]

​        [-mv <src> ... <dst>]

​        [-put [-f][-p] <localsrc> ... <dst>]

​        [-renameSnapshot <snapshotDir> <oldName> <newName>]

​        [-rm [-f][-r|-R] [-skipTrash] <src> ...]

​        [-rmdir [--ignore-fail-on-non-empty] <dir> ...]

​        [-setfacl [-R][{-b|-k} {-m|-x } ]|[--set <acl_spec> <path>]]

​        [-setrep [-R][-w] <rep> <path> ...]

​        [-stat [format] <path> ...]

​        [-tail [-f] <file>]

​        [-test -[defsz] <path>]

​        [-text [-ignoreCrc] <src> ...]

​        [-touchz <path> ...]

​        [-usage [cmd ...]]

## 2、常用命令和命令案例

- **cat**

  功能：将路径指定文件的内容输出到stdout。

  使用方法：

  ```shell
  hadoop fs -cat URI [URI …]    
  ```

  示例：

  ```shell
  hadoop fs -cat hdfs://host1:port1/file1   hdfs://host2:port2/file2
  #由于配置了core-site.xml，所以可以省略在hdfs对应的url

  hadoop fs -cat file:///file3 /user/hadoop/file4

  #返回值：成功返回0，失败返回-10。
  ```

  ​

- **chgrp**

  功能：

  ​ 改变文件所属的组。使用-R将使改变在目录结构下递归进行。命令的使用者必须是文件的所有者或者超级用户。更多的信息请参见HDFS权限用户指南。

  使用方法：

  ```shell
   hadoop fs -chgrp [-R] GROUP URI [URI ...]
  ```

  示例：

  ```shell
  hadoop fs -chgrp -R tearcher /flume    #递归的改变/flume目录下的所有文件的所属组为名为“tearcher”的组
  ```

  ​


- **chmod**

  功能：

  ​ 改变文件的权限，使用-R将使改变在目录结构下递归进行。命令的使用者必须是文件的所有者或者超级用户。

  使用方法：

  ```shell
  hadoop fs -chmod [-R] <MODE[,MODE]... | OCTALMODE> URI [URI …]
  ```

  示例：

  ```shell
   hadoop fs -chmod -R  777  /flume  #使/flume的权限对于所有用户拥有可读、可写、可执行权限
  ```

  ​


- **chown**

  功能：改变文件的拥有者，使用-R使改变在目录结构下递归进行。命令的使用者必须是超级用户。

  使用方法：

  ```shell
  hadoop fs -chown [-R] [OWNER]   [ :[GROUP]]  URI 
  ```

  示例：

  ```shell
  hadoop fs -chown -R hadoop_mapreduce:teacher /flume #递归的改变/flume目录下的所有文件的所属主为hadoop_mapreduce  所属组为teacher
  ```

  ​

- **put**

  功能：从本地文件系统中复制单个或多个源路径到目标文件系统。也支持从标准输入中读取输入写入目标文件系统。

  使用方法：

  ```shell
  hadoop fs -put <localsrc> ... <dst>
  ```

  示例：

  ```shell
  hadoop fs -put localfile /user/hadoop/hadoopfile

  hadoop fs -put localfile1 localfile2 /user/hadoop/hadoopdir

  hadoop fs -put localfile hdfs://host:port/hadoop/hadoopfile

  hadoop fs -put - hdfs://host:port/hadoop/hadoopfile 

  #从标准输入中读取输入。

  #返回值：成功返回0，失败返回-1。
  ```

  ​              　　　   

- **get**

  功能：复制文件到本地文件系统。可用-ignorecrc选项复制CRC校验失败的文件。使用-crc选项复制文件以及CRC信息。

  使用方法：

  ```shell
  hadoop fs -get -ignorecrc <src> <localdst> 
  ```

  示例：

  ```shell
  hadoop fs -get /user/hadoop/file   localfile

  hadoop fs -get hdfs://host:port/user/hadoop/file    localfile

  #返回值：成功返回0，失败返回-1。
  ```

  ​               　　　 

- **copyFromLocal    put**

  使用方法：

  ```shell
  hadoop fs -copyFromLocal <localsrc> URI
  ```

  除了限定源路径是一个本地文件外，和put命令相似。

  ​

- **copyToLocal    get**

  使用方法：

  ```shell
  hadoop fs -copyToLocal -ignorecrc URI <localdst>
  ```

  除了限定目标路径是一个本地文件外，和get命令类似。

  ​

- **cp**

  功能：将文件从源路径复制到目标路径。这个命令允许有多个源路径，此时目标路径必须是一个目录。

  使用方法：

  ```shell
  hadoop fs -cp URI [URI …] <dest>
  ```

  示例：

  ```shell
  hadoop fs -cp /user/hadoop/file1 /user/hadoop/file2

  hadoop fs -cp /user/hadoop/file1 /user/hadoop/file2 /user/hadoop/dir

  #返回值：成功返回0，失败返回-1。
  ```

  ​

- **du**

  功能：显示目录中所有文件的大小，或者当只指定一个文件时，显示此文件的大小。

  使用方法：

  ```shell
  hadoop fs -du URI [URI …]
  ```

  示例：

  ```shell
  hadoop fs -du /user/hadoop/dir1 /user/hadoop/file1 hdfs://host:port/user/hadoop/dir1 

  #返回值：成功返回0，失败返回-1。 

  ```

  ​

- **dus**

  功能：显示文件的大小。

  使用方法：

  ```shell
  hadoop fs -dus <args>
  ```

  示例：

  ```shell
  hadoop fs -dus /flume/app
  ```

  ​

- **ls**

  功能：

  如果是文件，则按照如下格式返回文件信息：

  ​ 文件名 <**副本数**>   文件大小    修改日期     修改时间   权限  用户ID    组ID 

  如果是目录，则返回它直接子文件的一个列表，就像在Unix中一样。目录返回列表的信息如下：

  ​ 目录名 <dir>   修改日期   修改时间   权限   用户ID   组ID 

  使用方法：

  ```shell
  hadoop fs -ls <args>
  ```

  示例：

  ```shell
  hadoop fs -ls /user/hadoop/file1  /user/hadoop/file2   hdfs://host:port/user/hadoop/dir1 /nonexistentfile 

  #返回值：成功返回0，失败返回-1。 
  ```

  ​

- **lsr**

  功能：ls命令的递归版本。类似于Unix中的ls -R。

  使用方法：

  ```shell
  hadoop fs -lsr <args> 
  ```

  ​

- **mkdir**

  功能：接受路径指定的uri作为参数，创建这些目录。其行为类似于Unix的mkdir -p，它会创建路径中的各级父目录。

  使用方法：

  ```shell
  hadoop fs -mkdir <paths> 
  ```

  示例：

  ```shell
  hadoop fs -mkdir /user/hadoop/dir1 /user/hadoop/dir2

  hadoop fs -mkdir hdfs://host1:port1/user/hadoop/dir hdfs://host2:port2/user/hadoop/dir

  #返回值：成功返回0，失败返回-1。
  ```

  ​

- **movefromLocal**

  功能：和put方法类似，从本地移动文件到hdfs

  使用方法：

  ```shell
  hadoop fs -moveFromLocal <src> <dst>          　　　　
  ```

  示例：

  ```shell
  hadoop fs -moveFromLocal /usr/local/* /user/flume/
  ```

  ​

- **mv**

  功能：将文件从源路径移动到目标路径。这个命令允许有多个源路径，此时目标路径必须是一个目录。不允许在不同的文件系统间移动文件。

  使用方法：

  ```shell
  hadoop fs -mv URI [URI …] <dest>
  ```

   示例：

  ```shell
  hadoop fs -mv /user/hadoop/file1 /user/hadoop/file2

  hadoop fs -mv hdfs://host:port/file1 hdfs://host:port/file2 hdfs://host:port/file3 hdfs://host:port/dir1   #(多集群环境—不常用)

  #  返回值：成功返回0，失败返回-1。
  ```

  ​

- **rm**

  功能：删除指定的文件。只删除非空目录和文件。请参考rmr命令了解递归删除。

  使用方法：

  ```shell
  hadoop fs -rm URI [URI …]
  ```

  示例：

  ```shell
  hadoop fs -rm hdfs://host:port/file /user/hadoop/emptydir

  #返回值：成功返回0，失败返回-1。
  ```

  ​

- **rmr**

  功能：delete的递归版本。

  使用方法：hadoop fs -rmr URI [URI …]

  示例：

  ```shell
  hadoop fs -rmr /user/hadoop/dir

  hadoop fs -rmr hdfs://host:port/user/hadoop/dir

  返回值：成功返回0，失败返回-1。

  ```

  ​

- **setrep**

  功能：改变一个文件的副本系数。-R选项用于递归改变目录下所有文件的副本系数。

  使用方法：

  ```shell
  hadoop fs -setrep [-R] <path>
  ```

  示例：

  ```shell
  hadoop fs -setrep -w 3 -R /user/hadoop/dir1

  #返回值：成功返回0，失败返回-1。
  ```

  ​

- **stat**

  功能：返回指定路径的统计信息。

  使用方法：hadoop fs -stat URI [URI …]

  示例：

  ```shell
  hadoop fs -stat path

  #返回值：成功返回0，失败返回-1。
  ```

  > stat命令详解：
  >
  > 当向HDFS上写文件时，可以通过设置**dfs.block.size**配置项来设置文件的block size，这导致HDFS上不同文件的block size是不同的。有时候我们需要知道HDFS上某个文件的block size，比如想知道该该文件作为job的输入会创建几个map等。Hadoop FS Shell提供了一个**-stat**选项可以达到目的。-**stat**选项的使用格式是：
  >
  > ​       hadoop fs –stat [format]
  >
  > 其中可选的*format*的形式：
  >
  > ​   **%b**：打印文件大小（目录为0）
  >
  > ​   **%n**：打印文件名
  >
  > ​   **%o**：打印block size （我们要的值）
  >
  > ​   **%r**：打印备份数
  >
  > ​   **%y**：打印UTC日期 yyyy-MM-dd HH:mm:ss
  >
  > ​   **%Y**：打印自1970年1月1日以来的UTC微秒数
  >
  > ​   **%F**：目录打印directory, 文件打印regular file
  >
  > 当使用**-stat**选项但不指定*format*时候，只打印文件创建日期，相当于**%y**：
  >
  > ​   **hadoop fs -stat /liangly/teradata/part-00099**
  >
  > ​       输出内容：  2018-04-20 08:03:49
  >
  > 下面的例子打印文件的block size和备份个数：
  >
  > ​   **hadoop fs -stat "%o %r" /liangly/teradata/part-00099**
  >
  > ​       输出内容：67108864 3
  >
  > 从打印结果可以看到文件*/liangly/teradata/part-00099*的block size是64m，有3个备份。
  >
  > ​

- **tail**

  功能：将文件尾部1K字节的内容输出到stdout。支持-f选项，行为和Unix中一致。

  使用方法：

  ```shell
  hadoop fs -tail [-f] URI
  ```

  示例：

  ```shell
  hadoop fs -tail pathname

  #返回值：成功返回0，失败返回-1。
  ```

  ​

- **test**

  功能：测试检查目录或者文件是否存在

  ​      选项：

  ​ -e 检查文件是否存在。如果存在则返回0。

  ​ -z 检查文件是否是0字节。如果是则返回0。 

  ​ -d 如果路径是个目录，则返回1，否则返回0。

  使用方法：

  ```shell
  hadoop fs -test -[ezd] URI
  ```

  示例：

  ```shell
  hadoop fs -test -e filename
  ```

  ​

- **text**

  功能：将文本文件或某些格式的非文本文件通过文本格式输出。允许的格式是zip和TextRecordInputStream。

  使用方法：

  ```shell
  hadoop fs -text <src> 
  ```

  ​

- **touchz**

  功能：创建一个0字节的空文件。

  使用方法：

  ```shell
  hadoop fs -touchz URI [URI …] 
  ```

  示例：

  ```shell
  hadoop -touchz pathname

  #返回值：成功返回0，失败返回-1。
  ```

  ​

- **distcp**

  功能：利用distcp在hdfs间传送数据

  示例：

  ```shell
  $ hadoop distcp -overwrite hdfs://192.168.2.31:9000/tmp/pageview.log hdfs://hadoop01:9000/tmp/pageview.log
  #两个不同hdfs之间传递数据
  ```

  ​

- **getmerge**

  功能：接受一个源目录和一个目标文件作为输入，并且将源目录中所有的文件连接成本地目标文件。addnl是可选的，用于指定在每个文件结尾添加一个换行符。

  使用方法：

  ```shell
  hadoop fs -getmerge <src> <localdst> [addnl]
  ```

  示例

  > 假设在你的hdfs集群上有一个/user/hadoop/output目录
  >
  > 里面有作业执行的结果（多个文件组成）part-000000,part-000001,part-000002
  >
  > 然后你想把所有的文件合拢来一起看 可以使用命令：
  >
  > ​   hadoop fs  -getmerge  /user/hadoop/output   local_file
  >
  > 然后就可以在本地使用vi local_file查看内容了

## 3、Hadoop回收站trash

  Hadoop回收站trash，默认是关闭的。 建议最好还是把它提前开开，否则误操作的时候，就欲哭无泪了

- 修改conf/core-site.xml,增加复制代码

  ```xml
  <property> 
    <name>fs.trash.interval</name> 
    <value>1440</value> 
    <description>Number of minutes between trash checkpoints. If zero, the trash feature is disabled. 
    </description> 
  </property>
  ```

  默认是0.单位分钟。这里我设置的是1天（60*24） 

  删除数据rm后，会将数据move到当前文件夹下的.Trash目录

- 测试 

  - 新建目录input

    ```shell
    hadoop fs -mkdir /input
    ```

    ​

  - 上传文件

    ```shell
      hadoop fs -copyFromLocal /data/soft/file0* /input
    ```

    ​

  - 删除目录input

    ```shell
     hadoop fs -rmr /input 
      #Moved to trash: hdfs://master:9000/user/hadoop/.Trash/Current/input/
    ```

    ​

  - 参看当前目录

    ```shell
     hadoop fs -ls /user/hadoop/

      #Found 2 items 
      #drwxr-xr-x - root supergroup 0 2011-02-12 22:17 /user/hadoop/.Trash
    ```

     发现input删除后，多了一个目录.Trash

  - 恢复刚刚删除的目录

    ```shell
     hadoop fs -mv /user/hadoop/.Trash/Current/input /
    ```

    ​

  - 检查恢复的数据

    ```shell
    hadoop fs -ls input 

        #  Found 2 items 

        # -rw-r--r-- 3 root supergroup 22 2018-04-12 17:40 /input/file01 

        #  -rw-r--r-- 3 root supergroup 28 2018-04-12 17:40 /input/file02
    ```

    ​

  - 删除.Trash目录（清理垃圾）

    ```shell
     hadoop fs -rmr /user/hadoop/.Trash

      # Deleted hdfs://master:9000/user/hadoop/.Trash   
    ```

    ​


- **expunge**

  功能：清空回收站。请参考HDFS设计文档以获取更多关于回收站特性的信息。默认关闭

  使用方法：

  ```shell
  hadoop fs -expunge    #就是将 /user/hadoop/.Trash目录里的内容清空
  ```

  ​

# 五、Java操作hdfs

## 1、使用maven创建一个maven工程

### 0、安装maven

0.1 下载安装maven（[Apache Maven官网](https://maven.apache.org/download.cgi)）

0.2 解压，配置环境变量

0.3 设置本地库的位置（安装目录/config/setting.xml）

### 1、配置pom.xml文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.qf</groupId>
  <artifactId>QF_Online</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>QF_Online</name>
  <url>http://maven.apache.org</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <hadoop.version>2.7.1</hadoop.version>
  </properties>

  <dependencies>
   <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    <!-- jdk依赖 -->
    <dependency>
    	<groupId>jdk.tools</groupId>
    	<artifactId>jdk.tools</artifactId>
    	<version>1.7</version>
    	<scope>system</scope>
    	<systemPath>${JAVA_HOME}/lib/tools.jar</systemPath>
    </dependency>
    <dependency>
	    <groupId>org.apache.zookeeper</groupId>
	    <artifactId>zookeeper</artifactId>
	    <version>3.4.6</version>
	    <type>pom</type>
	</dependency>
    
    <!-- hadoop start -->
	<dependency>
	    <groupId>org.apache.hadoop</groupId>
	    <artifactId>hadoop-common</artifactId>
	    <version>${hadoop.version}</version>
	</dependency>

	<dependency>
	    <groupId>org.apache.hadoop</groupId>
	    <artifactId>hadoop-client</artifactId>
	    <version>${hadoop.version}</version>
	</dependency>
	
		<dependency>
	    <groupId>org.apache.hadoop</groupId>
	    <artifactId>hadoop-hdfs</artifactId>
	    <version>${hadoop.version}</version>
	</dependency>
	<!-- hadoop end -->
  </dependencies>
</project>

```

## 2.Java操作HDFS对象

### 1.基本操作流程

```java
//获取配置信息(可以在此设置hdfs的配置,例如core-site.xml)
Configuation conf = new Configuration();
//conf.set("fs-defaultFS", "hdfs://192.168.99.101:9000");设置连接方式
//设置hdfs连接地址
URI uri = new URI("hdfs://192.168.1.200:9000");
//创建FileSystem对象
FileSystem fileSystem = FileSystem.get(uri, conf,"hadoop");//伪装成hadoop用户登录
//FileSystem fileSystem = FileSystem.get(conf);对应直接设置连接方式的conf

fileSystem.copeFromLocal("d:\test.txt","\input");
fileSystem.close();
```

- **注意:** 所有包都来自于apache.hadoop下

### 2.获取根目录下所有文件

```java
public void getFileStatus() throws FileNotFoundException, IllegalArgumentException, IOException {
        //获取路径根目录下的所有对象信息
        FileStatus[] listStatus = fileSystem.listStatus(new Path("/"));
        for (FileStatus fileStatus : listStatus) {
            System.out.println(fileStatus);
        }
    }
```

### 3.创建目录

```java
public void mkdir() throws IllegalArgumentException, IOException {
    //3.在根目录下创建目录
    boolean mkdirs = fileSystem.mkdirs(new Path("/myhdfs"));//用Java操作hdfs创建的文件夹-目录
    System.out.println(mkdirs);
}
```

### 4.删除目录

```java
public void delete() throws IllegalArgumentException, IOException {
    //4.删除目录
    //删除不存在的目录或文件会返回false
    boolean delete = fileSystem.delete(new Path("/myhdfs"), true);
    System.out.println(delete);
}
```

### 5.上传文件

```java
public void uploadData() throws IllegalArgumentException, IOException {
    //5.上传文件-将Windows下hosts文件上传
    FSDataOutputStream create = fileSystem.create(new Path("/hosts1"));
    FileInputStream fsIn = new                                   FileInputStream("C:\\Windows\\System32\\drivers\\etc\\hosts");
    //最后一个参数true是直接把fsIn流关闭,如果为false则不关闭流
    IOUtils.copyBytes(fsIn, create, conf, true);
}

```

### 6.读取文件

```java
public void readFile() throws IOException {
    //6.读取文件
    FSDataInputStream open = fileSystem.open(new Path("/hosts1"));
    IOUtils.copyBytes(open, System.out, conf, true);
}
```



### 7.按行读取文件

```java
public void readFile2() throws IllegalArgumentException, IOException {
    //7.按行读取文件
    FSDataInputStream fsin = fileSystem.open(new Path("/input/test1.txt"));
    byte RL = '\r';//回车
    byte NL = '\n';//换行
    byte[] buffer = new byte[1024*64];
    int length = fsin.read(buffer);
    int start_pos = 0;
    int pos = 0;
    String str = null;
    for (pos = 0;pos < length; pos++) {
        //当为回车或者换行符时
        if (buffer[pos] == RL || buffer[pos] == NL) {
            //根据起始坐标start_pos和当前坐标生成字符串
            str = new String(buffer, start_pos, pos);
            System.out.println(str);

            pos++;//当前坐标加1
            start_pos = pos;//将新的位置赋给起始坐标
        }
    }
}
```

### 8.删除文件

```java
public void deleteFile() throws IllegalArgumentException, IOException {
    boolean deleteFile = fileSystem.delete(new Path("/hosts1"), true);//删除文件
    System.out.println(deleteFile);
}
```



