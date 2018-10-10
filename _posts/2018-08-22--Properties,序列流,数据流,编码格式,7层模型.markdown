---
layout:     post
title:      "java基础之TCP,正则"
subtitle:   "java基础之TCP,正则"
date:       2018-08-22 12:00:00
author:     "Chamber"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
    - TCP
    - 正则表达式
    - Java基础
    - 数据类型
---
# Properties

> 实质上是一个Map集合,存储的是属性,属性以键值对的形式存在.内部的键值对中的键和值都必须是字符串,不需要泛型

#### 为什么要使用Properties?
A:因为使用与流结合


#### 优点:
1. 以键值对的形式存储数据
2. 内部针对属性的存储封装了大量的专有方法:load,store,list


## Properties的基础
```
//1.Properties的基础
public static void fun1() {
    //创建Properties对象
    Properties properties = new Properties();
    
    //存储
    properties.setProperty("01", "java");
    properties.setProperty("02", "html");
    
    //取值
    System.out.println(properties.getProperty("01"));
    
    //里面放的是所有的key
    Set<String> set = properties.stringPropertyNames();
    //遍历
    //第一种方式
    for (String string : set) {
    	System.out.println("增强for循环:"+string+"   value:"+properties.getProperty(string));
    }
    
    //第二种方式:迭代器
    Iterator<String> iterator = set.iterator();
    while (iterator.hasNext()) {
    	String string = (String) iterator.next();
    	System.out.println("迭代器:"+string+"   value:"+properties.getProperty(string));
    }
    
    //注意点:一:当key不存在的时候,value返回的是后面的默认值
    String value = properties.getProperty("05", "iOS");
    System.out.println("value:"+value);
    
    //注意点二:跟Map一样,key是唯一的,同一个key后面的值会覆盖前面的值.
    properties.setProperty("01", "python");
    System.out.println(properties.getProperty("01"));
}
```

## 获取系统的属性集合
```
//2.获取系统的属性集合
public static void fun2() {
	Properties properties = System.getProperties();
	
	//遍历属性集合
	Set<String> set = properties.stringPropertyNames();
	Iterator<String> iterator = set.iterator();
	while (iterator.hasNext()) {
		String key = (String) iterator.next();
		System.out.println("key:"+key+"   value:"+properties.getProperty(key));
	}
	
	//改变内部的值
	properties.setProperty("file.encoding", "GBK");
	System.out.println(properties.getProperty("file.encoding"));//GBK
	
	//在获取一遍
	Properties properties1 = System.getProperties();
	//原理:会先到内存中找属性集合的对象,如果有,直接使用.如果没有,会重新初始化一个新的对象,并获取属性集合.
	System.out.println(properties1.getProperty("file.encoding"));//GBK
	
	//直接将内容打印到控制台
	properties.list(System.out);
}
```
## 模拟系统的属性集合操作

```
//3.模拟系统的属性集合操作
public static void fun3() throws IOException {
	//创建属性集合对象
	Properties properties = new Properties();
	
	//创建输入流--temp1.txt模拟的系统的属性集合文件
	FileReader fileReader = new FileReader("temp1.txt");
	
	//将输入流的内容存储到属性集合文件中--通过load方法实现
	properties.load(fileReader);
	
	//实现打印到控制台
	properties.list(System.out);
	
	//更改内容
	properties.setProperty("01", "buxiakele");
	
	//将内容写回磁盘,创建写出流
	FileWriter fileWriter = new FileWriter("temp1.txt");
	
	//开始写回--第二个参数是注释说明
	properties.store(fileWriter, "dajiael");
	
	//关闭流
	fileReader.close();
	fileWriter.close();
}
```

# 序列流

> 把多个输入流的内容一次性的打印(操作)---字节流

```
    //创建三个输入流
		FileInputStream fileInputStream1 = new FileInputStream("src\\com\\qianfeng\\test\\Demo2.java");
		FileInputStream fileInputStream2 = new FileInputStream("src\\com\\qianfeng\\test\\Demo2.java");
		FileInputStream fileInputStream3 = new FileInputStream("src\\com\\qianfeng\\test\\Demo1.java");
		
		//将三个输入流放入序列流
		//方式一:先放入一个Vector
//		Vector<FileInputStream> vector = new Vector<>();
//		vector.add(fileInputStream1);
//		vector.add(fileInputStream2);
//		vector.add(fileInputStream3);
//		
//		//得到枚举器
//		Enumeration<FileInputStream> e1 = vector.elements();
		
		//方式二:先放入一个list
		ArrayList<FileInputStream> list = new ArrayList<>();
		list.add(fileInputStream1);
		list.add(fileInputStream2);
		list.add(fileInputStream3);
		
		//将集合转换成枚举
		Enumeration<FileInputStream> e2 = Collections.enumeration(list);
		
		//创建序列流对象并关联相关的文件--参数是一个枚举器
		//SequenceInputStream sequenceInputStream = new SequenceInputStream(e1);
		SequenceInputStream sequenceInputStream = new SequenceInputStream(e2);
		
		//创建输出流
		FileOutputStream fileOutputStream = new FileOutputStream("temp2.txt");
		
		//读写
		byte[] arr = new byte[1024];
		int num;
		while ((num = sequenceInputStream.read(arr)) != -1) {
			fileOutputStream.write(arr, 0, num);
			
			fileOutputStream.flush();
		}
		
		sequenceInputStream.close();
		fileOutputStream.close();
```

# 数据流:字节流

1. DataInputStream:  数据输入流  

2. DataOutputStream:  数据输出流
                     
**注意:** 数据流要与字节输入流,输出流配合使用

```
public static void writeData() throws IOException {
	DataOutputStream dataOutputStream = new DataOutputStream(new FileOutputStream("temp3.txt"));
	
	//写
	dataOutputStream.writeInt(97);//4个字节      00000000 00000000 000000000 011000001  00000001
	dataOutputStream.writeBoolean(true);//1个   
	dataOutputStream.write(33);//1个
	dataOutputStream.writeDouble(34.56);//8个
	
	//关闭流
	dataOutputStream.close();
}

public static void readData() throws IOException {
	DataInputStream dataInputStream = new DataInputStream(new FileInputStream("temp3.txt"));
	
	//这里的boolean型和int型的数据,在读的时候由于与之前写的顺序相反了,所以读取的数据错误
	/*
	 * 注意点:1.读的顺序要与写的顺序一致   2.类型保持一致
	 */
	System.out.println(dataInputStream.readBoolean());// 00000000
	System.out.println(dataInputStream.readInt());//00000000 000000000 011000001  00000001
	
	System.out.println(dataInputStream.readByte());
	System.out.println(dataInputStream.readDouble());
	
	dataInputStream.close();
}
```

# 内存流(byte数组流)

```
/*
 * 内存流(byte数组流):
 * ByteArrayInputStream:写入内存,在内部有一个数组,数据被放在这里面
 * ByteArrayOutputStream:将数据取出,放在字节数组里面
 */

//创建输入流,关联一个byte型的数组,作为缓冲区数据
ByteArrayInputStream bais = new ByteArrayInputStream("hello world".getBytes());

//创建输出流-不需要指定参数
ByteArrayOutputStream baos = new ByteArrayOutputStream();

byte[] arr = new byte[1024];
int num;
while ((num = bais.read(arr)) != -1) {
	baos.write(arr, 0, num);
}

System.out.println(new String(arr));

bais.close();
baos.close();

//注意:将流关闭了之后,还可以调用方法,不会报错.
baos.write(45);
```

# 序列化流

> 是将短期存储的数据实现长期存储

#### 数据的存储分成两类:                                        
1.短期存储:存放在内存中,随着程序的关闭而释放---对象,集合,变量,数组            
2.长期存储:存储在磁盘中,即使程序关闭了,数据仍然存在------文件              
                                                  
#### 序列化:
将数据从内存放入磁盘,可以实现数据的长久保存--数据持久化的手段              
#### 反序列化:
将数据从磁盘放回内存                                   
                                                  
#### 进行序列化的步骤:--通过对象的序列化讲解                             
1. 创建一个类                                           
2. 使用对应的流将对象存入磁盘中----序列化----ObjectOutputStream     
3. 使用对应的流将对象从磁盘中取出放回内存--反序列化------ObjectInputStream
4. 关闭流                                             
                                                  
> 注意点:序列化流在工作时也要关联对应的输入流和输出流                        


#### 创建类用于序列化                                                                               
类通过实现 java.io.Serializable 接口以启用其序列化功能。未实现此接口的类将无法使其任何状态序列化或反序列化。                      
可序列化类的所有子类型本身都是可序列化的。序列化接口没有方法或字段，仅用于标识可序列化的语义。                                        
                                                                                       
>  #### 解释:                                                                                   
>  一个类如果没有实现Serializable,进行序列化会报异常:NotSerializableException
>  
>  实现了Serializable接口的类可以达到的目的:                                                           
>  1. 可以进行序列化
>  2. 进行序列化的类的元素都必须支持序列化
>  3. 接口本身没有方法或字段,只是用来表示可序列化的语义
                                                                                       
#### 注意点:                                                                                
 1. ClassNotFoundException:当前的类没有找到<br>
 分析:将Person对象进行序列化之后,将Person类删除,再进行反序列化的时候出现了异常<br>
 原因:反序列化在执行的时候依赖字节码文件,当类没有了,字节码文件无法创建,反序列化失败

 2. java.io.InvalidClassException  无效的类<br>
 出现的原因:没有声明自己的serialVersionUID,而使用系统的.在进行反序列化的时候,类被改动了,系统认为现在的类已经不是原来的类了(在使用系统的id进行识别的时候,重写给Person设置了id),认为此类无效                                      
                                                                                       
 3. **使用系统的serialVersionUID与自定义的ID的区别?**<br>
 使用系统的,序列化和反序列化,id不能手动设置,使用的是编译器默认生成的,一旦类发生了改动,id会重新赋值                                 
 使用自定义的,序列化和反序列化,id不会发生改变,所以当反序列化的时候,即使对Person类进行了一些改动,也能继续反序列化                        
                                                                                       
 4. 总结序列化,反序列化工程的注意点:<br>
 a. 合理使用序列化流和反序列化流,要与输入流与输出流配合使用                                                        
 b. 进行序列化的类一定要实现Serializable接口,只要实现了接口就可以序列化.包括集合,包装类等                                  
 c. 进行序列化的类要保证当前类与内部的类都要实现Serializable接口

```
class Person  implements Serializable{
    /**
	 * generated:由编译器自动生成的,后面加L表示long型
	 */
	private static final long serialVersionUID = -7224641225172644265L;
//	/**
//	 * default:UID是由用户自己指定的,默认值是1L
//	 */
//	private static final long serialVersionUID = 1L;
	private String name;
	private int age;
	private int height;
	public String toString() {
		return "Person [name=" + name + ", age=" + age + "]";
	}
	public Person(String name, int age) {
		super();
		this.name = name;
		this.age = age;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
}
public class Demo6 {
	public static void main(String[] args) throws FileNotFoundException, IOException, ClassNotFoundException {
		//写出--序列化
		//writeFile();
		//读入--反序列化
		readFile();
	}
	
	//写出--序列化
	public static void writeFile() throws FileNotFoundException, IOException{
		//创建序列化流并关联文件
		ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("temp4.txt"));
		
		//调用方法实现序列化
		//序列化后的内容不能直接查看,要想查看进行反序列化
		objectOutputStream.writeObject(new Person("bingbing", 20));
		
		objectOutputStream.close();
	}
	//读入--反序列化
	public  static  void readFile() throws FileNotFoundException, IOException, ClassNotFoundException {
		ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("temp4.txt"));
		
		//实现反序列化
		Object object = objectInputStream.readObject();
		
		//向下转型
		//Person person = (Person)object;
		/*
		 * ClassNotFoundException:当前的类没有找到
		 * 分析:将Person对象进行序列化之后,将Person类删除,再进行反序列化的时候出现了异常
		 * 原因:反序列化在执行的时候依赖字节码文件,当类没有了,字节码文件无法创建,反序列化失败
		 */
		System.out.println(object);
		
		objectInputStream.close();
	}
```

# 编码格式

开发中的编码                                                                           
常用的字符集:一个汉字:GBK:2个字节       ISO8859-1:1个字节       utf-8:3个字节    unicode:2个字节(内部编码) 

#### 创建输出流并关联文件
```
//   第一个参数是字节输出流     第二个参数是:输出时指定的编码格式
OutputStreamWriter outputStreamWriter = new OutputStreamWriter(new FileOutputStream("utf8.txt"),"utf-8");
```
#### 读入流
```
InputStreamReader inputStreamReader = new InputStreamReader(new FileInputStream("utf8.txt"),"utf-8");
```

#### 使用字符串
```
//使用GBK编码,GBK解码
String str1 = "你好";
byte[] str1b = str1.getBytes("GBK");
System.out.println(new String(str1b,"GBK"));//你好
System.out.println(Arrays.toString(str1b));//[-60, -29, -70, -61]
```

# 网络编程

## 7层模型

1. 应用层
 
与其它计算机进行通讯的一个应用，它是对应应用程序的通信服务的。
例如，一个没有通信功能的字处理程序就不能执行通信的代码，
从事字处理工作的程序员也不关心OSI的第7层。但是，如果添加了一个
传输文件的选项，那么字处理器的程序员就需要实现OSI的第7层。
示例：TELNET，HTTP，FTP，NFS，SMTP等。

2. 表示层

这一层的主要功能是定义数据格式及加密。例如，FTP允许你选择以二进制
或ASCII格式传输。如果选择二进制，那么发送方和接收方不改变文件的内容。
如果选择ASCII格式，发送方将把文本从发送方的字符集转换成标准的ASCII后
发送数据。在接收方将标准的ASCII转换成接收方计算机的字符集。示例：加密，ASCII等。

3. 会话层

它定义了如何开始、控制和结束一个会话，包括对多个双向消息的控制和管理，
以便在只完成连续消息的一部分时可以通知应用，从而使表示层看到的数据是连续的，
在某些情况下，如果表示层收到了所有的数据，则用数据代表表示层。示例：RPC，SQL等。

4. 传输层

这层的功能包括是否选择差错恢复协议还是无差错恢复协议，及在同一主机上对不同
应用的数据流的输入进行复用，还包括对收到的顺序不对的数据包的重新排序功能。
示例：TCP，UDP，SPX。

5. 网络层

这层对端到端的包传输进行定义，它定义了能够标识所有结点的逻辑地址，还定义了
路由实现的方式和学习的方式。为了适应最大传输单元长度小于包长度的传输介质，
网络层还定义了如何将一个包分解成更小的包的分段方法。示例：IP，IPX等。

6. 数据链路层

它定义了在单个链路上如何传输数据。这些协议与被讨论的各种介质有关。示例：ATM，FDDI等。

7. 物理层

OSI的物理层规范是有关传输介质的特性标准，这些规范通常也参考了其他组织制定的标准。
连接头、帧、帧的使用、电流、编码及光调制等都属于各种物理层规范中的内容。
物理层常用多个规范完成对所有细节的定义。示例：Rj45，802.3等。
