---
layout:     post
title:      "java基础之内部类,异常处理"
subtitle:   "内部类,异常处理"
date:       2018-08-03 12:00:00
author:     "Chamber"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
    - Java基础
    - 异常处理
    - 内部类
    - 数据类型
---
# Object类

### equals()

> 指示其他某个对象是否于此对象"相等"(默认比较的是对象的地址)

### getClass()

> 获取当前类的字节码文件

### hashCode()

> 获取当前对象的哈希值(内存当中的地址)

### toString()

> 默认打印包名加哈希值

# 内部类

> 定义在A类中的B类就是内部类,B类在A类中的地位与其他的成员相同(可以相互调用)

- 创建方法:
- 方法是:外部类的对象.new内部类构造方法



```
public class Demo2 {
	public static void main(String[] args) {
		Outer outer = new Outer();
		//创建的内部类的对象;方法是:外部类的对象.new内部类构造方法
		//Inner在外部是不可见的,必须使用外部类的对象去使用内部类
		Outer.Inner inner = outer.new Inner();
		inner.play();
	}
}
class Outer{
	int age = 10;
	
	class Inner{
		int height = 20;
		
		public void play() {
			System.out.println("Inner-play"+age);
		}
	}
	public void show() {
		System.out.println("Outer-show");
	}
}
```

- 作用:

通过内部类让java简介实现了多继承

```
class A{
	
}
class B{
	
}
class X extends A{
	class Y extends B{
		
	}
}
```
### 局部内部类

> 定义在一个方法中的类

###### 作用范围:

从定义它开始到所在的方法结束

###### 作用:

对于当前的show方法,相当于将他的一部分功能面向对象了,形成了局部内部类,既保证了代码的私有化又对代码进行了整理,增加了可读性,操作性,简化代码,增加了复用性

###### 了解:
局部变量和内部类在一起的时候,默认这个局部变量是final的,作用范围无形中扩大了


# 匿名内部类(对象)

> 定义在一个类方法中的你名字类对象,属于局部内部类

### 1. 匿名对象

```
//创建匿名对象,完成方法的调用
		new Test1().test1Method();
```

### 2. 匿名子类对象


创建匿名子类对象的方式:
```
//创建匿名子类对象
		//第一种方法
		//使用已有的子类生成匿名子类对象并调用
		new SubTest1().test1Method();;
		//第二种方法
		//直接创建没有名字的Test1类的匿名子类对象
		//构成 : new + 父类的名字/父接口的名字 + (){匿名子类的执行体}
		new Test1() {
			//重写的Test1类的方法
			public void test1Method() {
				System.out.println("匿名的子类方法");
			}
		}.test1Method();
```

### 匿名内部类

###### 创建匿名内部类对象的注意点:
1. 匿名内部类对象必须有父类或者父接口


# 异常

> 异常就是程序中出现不正常的情况

- 异常的由来

程序在运行的过程中出现了不正产给的情况,程序把它看成了对象,提取了属性,行为(名字,原因,位置等信息)形成了各种异常类

- 异常的分类:

(throwable)
1. Error(错误):运行中出现严重错误.不需要我们进行更改.
2. Exception:运行中出现的不严重的错误,我们可以尝试去更改.

###### Exception分类:
1. 第一种分类
    1. 系统异常:系统定义好的,我们直接使用
   2. 自定义异常:需要我们自己定义
2. 第二种分类
    1.编译异常:在编译阶段抛出异常,处理异常
    2. 运行时异常:在运行阶段抛出异常,处理异常



<br><br><br><br><br><br>