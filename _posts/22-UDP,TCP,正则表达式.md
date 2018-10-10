---
layout:     post
title:      "java基础之TCP.UDP"
subtitle:   "java基础之TCP.UDP"
date:       2018-08-22 12:00:00
author:     "Chamber"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
    - TCP
    - Java基础
    - 数据类型
---
# 网络通信

 -|TCP | UDP
:-|:-|:-
1.|是简历在连接的基础上			|			建立在非连接的基础上   
2.|安全性高					|			安全性低     
3.|传输速度低					|			传输速度高    
4.|适合传输数据量大的数据		|			数据量小的数据      

##### 端口号:
同一台主机上的每一个服务器都拥有自己的端口号,取值范围(0,65535),常用的端口:80,8080

##### 注意点:
1. 要保证客户端和服务器端的端口号一致    
2. 要保证同一台主机上的不同服务器端口号不同

# UDP

## 客户端

创建流程:
1. 创建UDP通信的对象-socket对象:对应的类是DatagramSocket.(用于UDP数据的发送与接收)
2. 数据的准备-封装包:DatagramPacket(数据包,包括相关的属性,数据)
3. 发送数据,通过send方法
4. 关闭socket对象
```
 //* 1.创建UDP通信的对象-socket对象:对应的类是DatagramSocket.(用于UDP数据的发送与接收)
DatagramSocket datagramSocket = new DatagramSocket();
 //* 2.数据的准备-封装包:DatagramPacket(数据包,包括相关的属性,数据)
/*
 * 打包
 * 第一个参数:包的内容
 * 第二个参数:数据的长度
 * 第三个参数:主机对象
 * 第四个参数:端口号
 */
byte[] buf = "bingbing".getBytes();
DatagramPacket datagramPacket = new DatagramPacket(buf, buf.length, InetAddress.getLocalHost(), 20000);
 //* 3.发送数据,通过send方法
datagramSocket.send(datagramPacket);
 //* 4.关闭socket对象
datagramSocket.close();
```

## 服务器端

创建流程:
1. 创建socket对象,并绑定端口号
2. 创建包对象,创建空数组,准备接收传来的数据
3. 接收数据
4. 关闭相关的对象

```
 //* 1.创建socket对象,并绑定端口号
DatagramSocket datagramSocket = new DatagramSocket(20000);
 //* 2.创建包对象,创建空数组,准备接收传来的数据
byte[] buf = new byte[1024];
DatagramPacket datagramPacket = new DatagramPacket(buf, buf.length);
 //* 3.接收数据--当服务器运行起来,这个方法会一直处于监听状态
datagramSocket.receive(datagramPacket);

//将数据从包中取出
byte[] data = datagramPacket.getData();
System.out.println(new String(data));//打印接收的数据
 //* 4.关闭相关的对象
datagramSocket.close();
```

# TCP

## 客户端

在客户端与服务器端通信的时候,对于客户端既要进行输入又要进行输出,所以在Socket对象的内部就内置了输入流和输出流,当进行数据传输的时候,将数据放入socke对象的内部,将socket对象传到服务器端,相当于在客户端与服务器端建立了一个通道,两端使用同一个socket对象.
```
//1.创建Socket对象并绑定服务器的地址和端口
Socket socket = new Socket(InetAddress.getLocalHost(), 22000);

//2.准备数据
String data = "BigData1712,你好";

//3.获取socket内部的输出流
OutputStream outputStream = socket.getOutputStream();

//4.将数据写入网络
outputStream.write(data.getBytes());

//接收从服务器传回的数据
InputStream inputStream = socket.getInputStream();

//将内容写到控制台
byte[] arr = new byte[1023];
int num = inputStream.read(arr);
System.out.println(new String(arr,0,num));

//5.关闭资源
socket.close();
```

## 服务器端

```
//1.创建ServerSocket对象并绑定接口
ServerSocket serverSocket  = new ServerSocket(22000);

//2.接收套接字的连接,保证客户端与服务器端使用同一个socket对象---一直处于监听的状态
Socket socket =  serverSocket.accept();

//3.获取输入流
InputStream inputStream = socket.getInputStream();

//4.将内容写到控制台
byte[] arr = new byte[1023];
int num = inputStream.read(arr);
System.out.println(new String(arr,0,num));

//实现将服务器的数据写回客户端
OutputStream outputStream = socket.getOutputStream();
outputStream.write("你好,BigData1712".getBytes());

//5.关闭资源
serverSocket.close();
```
一个实例,实现大小写转换服务器
(使用的是各种流)

# 正则表达式
实例1:判断QQ是否合法
```
String qq = "07283904";
String regex = "[1-9]\\d{4,12}";
boolean b1 = qq.matches(regex);
System.out.println(b1);
```

## 匹配
```
//匹配
public static void piPei(){
//实例:可以在h和l之间有0个或多个o
//		String s = "schooooooool";
//		String regex = "scho*l";//o*:代表0个或多个o     o+:代表1个或多个o

//实例:匹配手机号码:18910909090
String s = "18910909090";
String regex = "1[345789]\\d{9}";

boolean b = s.matches(regex);
System.out.println(b);
}
```
## 切割
```
//切割
public static void qieGe(){
//实例:使用,进行切割
//		String s = "axogj,owjgoiwrj,sgoiwrjoigjw,orisjgwr,jog34wjr";
//		String regex = ",";
//		String[] strings = s.split(regex);

//要求按照空格切割
//		String s = "sdjof sjfsj      rjgojj          jgoj      jfoesjew  sd";
//		String regex = " +";
//		String[] strings = s.split(regex);

//要求使用.进行切割
String s = "sdjof.srj.gojjg.ofoe.sjesd";
String regex = "\\.";//.默认代表任意字符,要想使用.需要进行转义\\.
String[] strings = s.split(regex);

for (String string : strings) {
	System.out.println("string:"+string);
}
}
```
## 替换

```
//替换
public static void tiHuan(){
	//要求:将连续超过三个数字的部分进行替换---****
	String s = "sjfwfj4444sajose645sdgsjgrj3329889jsfjgowjsg888888888jsfjs";
	String regex = "\\d{4,}";
	String place = "****";
	
	String newString = s.replaceAll(regex, place);
	System.out.println(newString);
}
```

## 获取

```
//获取
public static void huoQu(){
	//要求:获取连续超过四个字母的子字符串
	String s = "sjoiffs   sjfj   sdfgjj       sdjijs    djd    d    jwejfe";
	String regex = "[a-zA-Z]{5,}";
	//相当于将正则表达式进行了简单的转化,但是Pattern本身不具有获取数据的能力
	Pattern pattern =  Pattern.compile(regex);
	
	//具有获取数据的能力
	Matcher matcher = pattern.matcher(s);
	
//		matcher.find();//判断是否有符合当前表达式的内容
//		matcher.group();//是获取内容
	while (matcher.find()) {
		String string = matcher.group();
		System.out.println(string);
	}
}
```