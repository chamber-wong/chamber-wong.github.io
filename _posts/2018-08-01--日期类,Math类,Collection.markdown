---
layout:     post
title:      "java基础之工具类"
subtitle:   "日期类,Math类,collection"
date:       2018-08-01 12:00:00
author:     "Chamber"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
    - 工具类
    - Java基础
    - 数据类型
---
# Date日期类

> java.util包提供了date类来封装当前的日期和时间.Date类提供了两个构造函数来实例化Date对象.

第一个构造函数使用使用当前的时间来初始化对象
```
Date()
```

第二个构造函数接收一个参数,改参数是从1970.1.1起的毫秒数
```
Date(long millisec)
```

Date对象创建以后,可以调用下面的方法:


# DateFormat类

> 只有系统提供的4种格式

1. short
2. long
3. full
4. default

```
//第一个参数是设置日期的格式,第二个是设置时间的格式
DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.SHORT, DateFormat.SHORT);
String date = dateFormat.format(new Date());
System.out.println(date);
//--->18-8-8 下午3:04

```


# 使用SimpleDateFormat格式化日期

> SimpleDateFormat 是一个以语言环境敏感的方式来格式化和分析日期的类。SimpleDateFormat 允许你选择任何用户自定义日期时间格式来运行。

实例:
```
import java.util.*;
import java.text.*;
 
public class DateDemo {
   public static void main(String args[]) {
 
      Date dNow = new Date( );
      SimpleDateFormat ft = new SimpleDateFormat ("E yyyy.MM.dd 'at' hh:mm:ss a zzz");
 
      System.out.println("Current Date: " + ft.format(dNow));
   }
}
```

# Math类

```
Math.abs(-44);//求绝对值
Math.floor(3.4);//向下取整
Math.ceil(3.4);//向上取整
Math.random();//取随机数 默认的范围是[0,1)
```

# Random类

**实例:取[0,100)之间的整数
```
Random random = new Random();
random.nextInt();//获取一个随机的int类型的数
random.nextInt(100);//获取[0,100)之间的整数
```

# 集合
- 数组:<br>
可以存储不同类型的多个数据,数据类型可以是简单数据类型,也可以是引用数据类型<br>
**缺点:**   创建的是一个定值,只能存储固定长度的数据,一旦存满了,就不能再继续存储
- 集合<br>
可以存储不同类型的多个数据,但是只能存储引用数据类型<br>
**缺点:** 只能存储引用数据类型<br>
**优点:** 存储空间会随和存储数据的增大而增大,所以可以更加喝了的利用内存空间,方法也很多

- 存储的分类:
1. 短期存储:一旦计算机关闭,存储的内容会被立刻释放----变量,对象,数组,集合
2. 长期存储:直接存储在磁盘上,可以长久的保存,数据不会随着计算机的关闭而消失.----.mp4,.mp3,.txt等文件

## Collection:---接口
方法:

增加：
```
       1：add()   将指定对象存储到容器中

                      add 方法的参数类型是Object 便于接收任意对象

       2：addAll() 将指定集合中的元素添加到调用该方法和集合中
```
删除：
```
       3：remove() 将指定的对象从集合中删除

       4：removeAll() 将指定集合中的元素删除
```
修改
```
       5：clear() 清空集合中的所有元素
```
判断
```
       6：isEmpty() 判断集合是否为空

       7：contains() 判断集合何中是否包含指定对象

           

       8：containsAll() 判断集合中是否包含指定集合

                            使用equals()判断两个对象是否相等 
```

获取:  
```
        9：int size()    返回集合容器的大小
```

转成数组
```
        10： toArray()   集合转换数组
```

#### 迭代器

- 获取迭代器对象
```
//1.获取迭代器对象
Iterator<String> iterator = collection.iterator();
//2.通过方法实现遍历
while (iterator.hasNext()) {
	String string = iterator.next();
	System.out.println(string);
}
```

**注意点**

```
//1.直接在此使用第一次的iterator金香槟简历,遍历失败,因为当前指针已经指向了集合的最后
//再次使用hasnext会直接返回false.所以如果想再次遍历,要重新获取迭代器对象.
while (iterator.hasNext()) {
	String string = (String) iterator.next();
	
}

//2.集合可以同时存储不同类型的数据
collection.add(1);

//3.再次遍历--当集合中存在不同了类型的数据是,需要进行容错处理和向下转型
Iterator iterator2 = collection.iterator();
while (iterator2.hasNext()) {
	Object object = iterator2.next();
	
	if (!(object instanceof String)) {
		throw new ClassCastException();
	}
	
	//向下转型
	String string = (String)object;
	System.out.println(string);
}
```

#### List:---接口

> 存储的数据是有序的,可以重复的

###### ArrayList:---类

> 底层的数据结构是数组,线程不安全的.:特点:查找速度快,添加删除速度慢

###### Vector:---类

> 底层的数据结构是数组,线程安全的.:特点:查找速度快,添加删除速度慢

###### LinkedList:---类

> 底层是链表,线程不安全的.特点:查找速度慢,添加删除速度快

#### Set:---接口

> 存储的数据是无序的,不可以重复的

###### HashSet:---类

###### TreeSet---类

## Map:---接口

###### HashMap:---类

###### TreeMap:---类
