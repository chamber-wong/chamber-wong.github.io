---
layout:     post
title:      "MapReduce的Java操作和IDEA中的环境搭建详解"
subtitle:   "Zookeeper的Javaapi,和IDEA中的环境搭建"
date:       2018-09-20 12:00:00
author:     "Chamber"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - hadoop
    - mapreduce
---
[TOC]



# 一、IDEA编写环境搭建

## 1、安装maven

IDEA自带maven，只需要设置仓库位置即可

> 操作

file -> setting -> Build,Execution,Deployment -> Maven -> Local repository

设置成自定义的仓库位置

## 2、新建Maven项目

file -> New -> project -> Maven

## 3、设置pom文件

**注意：** pom文件中可以视情况缺省

```xml
<dependencies>

        <!-- MapReduce的测试单元mrunit -->
        <dependency>
            <groupId>org.apache.mrunit</groupId>
            <artifactId>mrunit</artifactId>
            <version>1.0.0</version>
            <classifier>hadoop2</classifier>
            <scope>test</scope>
        </dependency>

        <!-- junit测试单元 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>


        <!-- zookeeper依赖 -->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.7</version>
            <type>pom</type>
        </dependency>

        <!--Hadoop依赖-->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.9.0</version>
        </dependency>
        <!--Hive依赖-->
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <version>2.3.3</version>
        </dependency>
        <!--Spark依赖-->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.11</artifactId>
            <version>2.3.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming_2.11</artifactId>
            <version>2.3.1</version>
        </dependency>
        <!--kafka&spark依赖-->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql-kafka-0-10_2.11</artifactId>
            <version>2.3.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming-kafka-0-10_2.11</artifactId>
            <version>2.3.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>0.11.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.htrace</groupId>
            <artifactId>htrace-core</artifactId>
            <version>3.1.0-incubating</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <!--maven编译java的相关设置-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.7.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <!--&lt;!&ndash; maven编译scala的相关设置 &ndash;&gt;-->
            <!--<plugin>-->
                <!--<groupId>net.alchim31.maven</groupId>-->
                <!--<artifactId>scala-maven-plugin</artifactId>-->
                <!--<version>3.4.1</version>-->
                <!--<executions>-->
                    <!--<execution>-->
                        <!--<id>scala-compile-first</id>-->
                        <!--<phase>process-resources</phase>-->
                        <!--<goals>-->
                            <!--<goal>add-source</goal>-->
                            <!--<goal>compile</goal>-->
                        <!--</goals>-->
                    <!--</execution>-->
                <!--</executions>-->
            <!--</plugin>-->
            <!--maven打jar包（无论是scala还是java）时候指定主类的插件-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <mainClass>wang.chamber.bigdata.mapreduce.topn.job.TopNJob</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

## 4、设置远程执行jar包操作

1、拷贝配置文件

将Hadoop根目录/etc/hadoop中的core-site.xml、log4j.properties、mapred-site.xml、yarn-site.xml拷贝到IDEA中的resources文件夹中

## 5、打包

在IDEA右侧的maven中依次执行

clean -> package

## 6、执行mapreduce程序

在IDEA右上角运行按钮的左边点击edit configuration在其中设置参数，即可运行

# 二、maprecue的Java操作

<a id="1、编写步骤"></a>
## 1、编写步骤

1. 用户编写的程序分成三个部分：Mapper，Reducer，Driver(提交运行mr程序的客户端)
2. Mapper的输入数据是KV对的形式（KV的类型可自定义）
3. Mapper的输出数据是KV对的形式（KV的类型可自定义）
4. Mapper中的业务逻辑写在map()方法中
5. map()方法（maptask进程）对每一个<K,V>调用一次
6. Reducer的输入数据类型对应Mapper的输出数据类型，也是KV
7. Reducer的业务逻辑写在reduce()方法中
8. Reducetask进程对每一组相同k的<k,v>组调用一次reduce()方法
9. 用户自定义的Mapper和Reducer都要继承各自的父类
10. 整个程序需要一个Drvier来进行提交，提交的是一个描述了各种必要信息的job对象

<a id="2、经典的wordcount程序编写"></a>
## 2、经典的wordcount程序编写

1. 需求：有一批文件（规模为TB级或者PB级），如何统计这些文件中所有单词出现次数

   如有三个文件，文件名是qf_course.txt、qf_stu.txt 和 qf_teacher

   qf_course.txt内容：

   ```
   php java linux
   bigdata VR
   C C++ java web
   linux shell
   ```

   qf_stu.txt内容：

   ```
   tom jim lucy
   lily sally
   andy
   tom jim sally
   ```

   qf_teacher内容：

   ```
   jerry Lucy tom
   jim
   ```

2. 方案

   - 分别统计每个文件中单词出现次数 - map（）
   - 累加不同文件中同一个单词出现次数 - reduce（）

3. 实现代码

   - [ ] 创建一个简单的maven项目

   - [ ] 添加hadoop client依赖的jar，pom.xml主要内容如下：

     ```xml
     <dependencies>
     		<dependency>
     			<groupId>org.apache.hadoop</groupId>
     			<artifactId>hadoop-client</artifactId>
     			<version>2.7.1</version>
     		</dependency>
     		
     		<dependency>
     			<groupId>junit</groupId>
     			<artifactId>junit</artifactId>
     			<version>4.11</version>
     			<scope>test</scope>
     		</dependency>		
     </dependencies>
     ```


<a id="3、编写代码"></a>
## 3、编写代码

<a id="mapper类"></a>
### mapper类

     ```java
     import java.io.IOException;
     
       import org.apache.hadoop.io.IntWritable;
       import org.apache.hadoop.io.LongWritable;
       import org.apache.hadoop.io.Text;
       import org.apache.hadoop.mapreduce.Mapper;
     
       /**
        * Maper里面的泛型的四个类型从左到右依次是：
        * 
        * LongWritable KEYIN: 默认情况下，是mr框架所读到的一行文本的起始偏移量，Long,  类似于行号但是在hadoop中有自己的更精简的序列化接口，所以不直接用Long，而用LongWritable 
        * Text VALUEIN:默认情况下，是mr框架所读到的一行文本的内容，String，同上，用Text
        *
        * Text KEYOUT：是用户自定义逻辑处理完成之后输出数据中的key，在此处是单词，String，同上，用Text
        * IntWritable VALUEOUT：是用户自定义逻辑处理完成之后输出数据中的value，在此处是单词次数，Integer，同上，用IntWritable
        */
       public class WordcountMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
     
        /**
        *在MapReduce程序运行的时候执行一次
        *此次执行在map方法之前
        **/
        @Override
        protected void setup(Context context) throws IOException, InterruptedException {
            super.setup(context);
        }
     
       	/**
       	 * map阶段的业务逻辑就写在自定义的map()方法中
       	 * maptask会对每一行输入数据调用一次我们自定义的map()方法
       	 */
       	@Override
       	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
       		
       		//将maptask传给我们的一行的文本内容先转换成String
       		String line = value.toString();
       		//根据空格将这一行切分成单词
       		String[] words = line.split(" ");
       	
       		/**
       		 *将单词输出为<单词，1> 
       		 *如<lily,1> <lucy,1>  <c,1> <c++,1> <tom,1> 
       		 */
       		for(String word:words){
       			//将单词作为key，将次数1作为value，以便于后续的数据分发，可以根据单词分发，以便于相同单词会到相同的reduce task
       			context.write(new Text(word), new IntWritable(1));
       		}
       	}
       	/**
         * 当一个输入分片（默认等同于一个块的大小）处理完毕后，由MR框架调用一次
         * @param context
         * @throws IOException
         * @throws InterruptedException
         */
        @Override
        protected void cleanup(Context context) throws IOException, InterruptedException {
            context.write(new Text("ca"),new LongWritable(countall));
        }
       }
     ```


​     

<a id="reduce类有setup方法和cleanup方法"></a>
### reduce类

     ```java
      import java.io.IOException;
     
       import org.apache.hadoop.io.IntWritable;
       import org.apache.hadoop.io.Text;
       import org.apache.hadoop.mapreduce.Reducer;
     
       /**
        * Reducer里面的泛型的四个类型从左到右依次是：
        * 	Text KEYIN: 对应mapper输出的KEYOUT
        * 	IntWritable VALUEIN: 对应mapper输出的VALUEOUT
        * 
        * 	KEYOUT, 是单词
        * 	VALUEOUT 是自定义reduce逻辑处理结果的输出数据类型，是总次数
        */
       public class WordcountReducer extends Reducer<Text, IntWritable, Text, IntWritable>{
     
       	/**
       	 * <tom,1>
       	 * <tom,1>
       	 * <linux,1>
       	 * <banana,1>
       	 * <banana,1>
       	 * <banana,1>
       	 * 入参key，是一组相同单词kv对的key
       	 * values是若干相同key的value集合
       	 * 如 <tom,[1,1]>   <linux,[1]>   <banana,[1,1,1]>
       	 */
       	@Override
       	protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
     
       		int count=0;  //累加单词的出现的次数
       		
       		for(IntWritable value:values){
       			count += value.get();
       		}
       		context.write(key, new IntWritable(count));	
       	}	
       }
     ```


​     

<a id="driver类（也叫job类）"></a>
### Driver类（也叫job类）

```java
package wang.chamber.bigdata.mapreduce.topn.job;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.DoubleWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import wang.chamber.bigdata.mapreduce.topn.mapper.TopNMapper;
import wang.chamber.bigdata.mapreduce.topn.reducer.TopNReducer;
import wang.chamber.bigdata.mapreduce.topn.util.MyArrayWritable;

import java.io.IOException;

/**
 * @description 程序入口
 * @author: wang chunbo
 * @create: 2018-09-20 19:53:56
 **/

public class TopNJob {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        if (args.length != 2){
            System.out.println("参数数量不够");
        }
        else {
            String inputPath = args[0];
            String outputPath = args[1];
            if (inputPath == "" || outputPath == null || outputPath == ""){
                System.out.println("传入参数错误,请重试");
                System.exit(1);
            }else{
                System.setProperty("HADOOP_USER_NAME", "root");
                Configuration configuration = new Configuration();
                configuration.set("fs.defaultFS", "hdfs://mycluster");
                configuration.set("dfs.nameservices", "mycluster");
                configuration.set("dfs.ha.namenodes.mycluster", "nn1,nn2");
                configuration.set("dfs.namenode.rpc-address.mycluster.nn1", "min1:8020");
                configuration.set("dfs.namenode.rpc-address.mycluster.nn2", "min2:8020");
                configuration.set("dfs.client.failover.proxy.provider.mycluster", "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider");
                //MR&YARN相关配置
                configuration.set("mapreduce.framework.name", "yarn");
                configuration.set("yarn.resourcemanager.address", "http://min2:8032");
                configuration.set("mapreduce.app-submission.cross-platform", "true");//允许跨平台提交jar包
                configuration.set("mapreduce.job.jar", "G:\\JavaSE\\Hadoop\\Hadoop\\target\\Hadoop-TopN.jar");
                configuration.set("yarn.resourcemanager.scheduler.address", "http://min2:8030");
                configuration.set("n",n);
                
                //设置作业对象
                Job job = Job.getInstance(configuration);
                //设置作业名字，可以在mapreduce中看到正在执行任务的名字
                job.setJobName("TopN");
                //作业驱动类的字节码文件
                job.setJarByClass(TopNJob.class);
				
                //设置mapper的字节码文件
                job.setMapperClass(TopNMapper.class);
                //设置mapper的输出Key的字节码文件
                job.setMapOutputKeyClass(Text.class);
                //设置mapper的输出value的字节码文件
                job.setMapOutputValueClass(MyArrayWritable.class);

                //设置reducer的字节码文件
                job.setReducerClass(TopNReducer.class);
                //设置reducer的输出Key的字节码文件
                job.setOutputKeyClass(Text.class);
                //设置reducer的输出value的字节码文件
                job.setOutputValueClass(DoubleWritable.class);

                //设置接收的两个参数
                FileInputFormat.addInputPath(job,new Path(inputPath));
                FileOutputFormat.setOutputPath(job,new Path(outputPath));

                System.exit(job.waitForCompletion(true)?0:1);
            }
        }
    }
}

```
<a id="4、运行步骤"></a>
## 4、运行步骤

<a id="1-将此程序打包-名为wordcountjar"></a>
### 1.打包
将此程序打包  名为wordcount.jar  

  

### 2. 上传

上传wordcount.jar到名为min1机器的/home/hadoop目录下

​    

<a id="3-在hdfs上创建文件夹“wordcountinput”，并将三个文件（qf_coursetxt、qf_stutxt-和-qf_teacher）上传到hdfs的“wordcountinput”目录下"></a>
### 3. 上传数据

在hdfs上创建文件夹“/wordcount/input”，并将三个文件（qf_course.txt、qf_stu.txt 和 qf_teacher）上传到hdfs的“/wordcount/input”目录下

    ```shell
    hadoop  fs  mkdir  -p  /wordcount/input
    hadoop  fs  –put  qf_course.txt  /wordcount/input
    hadoop  fs  –put  qf_stu.txt  /wordcount/input
    hadoop  fs  –put  qf_teacher.txt  /wordcount/input
    ```

<a id="4-在homehadoop下启动wordcountjar运行"></a>
### 4. 在Hadoop中执行jar包

在/home/hadoop下启动wordcount.jar运行

    ```shell
    hadoop jar wordcount.jar 包名.WordcountDriver /wordcount/input  /wordcount/output
    ```
    ###  5. 在hadoop的/wordcount/output下生成两个文件  如下：
    
    	_SUCCESS      //表示计算成功
    
    	part-r-00000  //处理结果文件 

<a id="6-查看结果"></a>
### 6. 查看结果    

    ```shell
    hadoop fs -cat /wordcount/output/part-r-00000		#结果如下
    	Hello 4
    	ketty 2
    	tom   2
    	jim   1
    	word  1
    ```


<a id="5、程序分析"></a>
## 5、程序分析

      1. 将文件拆分成splits，由于测试用的文件较小，所以每个文件为一个split，并将文件按行分割形成<key,value>对，下图所示。这一步由MapReduce框架自动完成，其中偏移量（即key值）包括了回车所占的字符数（Windows/Linux环境不同）。


      2. 将分割好的<key,value>对交给用户定义的map方法进行处理，生成新的<key,value>对，下图所示。


      3. 得到map方法输出的<key,value>对后，Mapper会将它们按照key值进行排序，并执行Combine过程，将key至相同value值累加，得到Mapper的最终输出结果。下图所示。
    
      4. Reducer先对从Mapper接收的数据进行排序，再交由用户自定义的reduce方法进行处理，得到新的<key,value>对，并作为WordCount的输出结果，下图所示。
