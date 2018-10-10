---
layout:     post
title:      "java基础之集合三"
subtitle:   "map,arrays类,可变参数,多线程"
date:       2018-08-11 12:00:00
author:     "Chamber"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
    - 集合
    - 多线程
    - Java基础
    - 数据类型
---
# 使用Map的注意点
1. 什么类型可以充当key的值?
    1. 让key实现Comparable接口
    2. 创建一个比较器,比较器是实现了Comparator接口 <br> *(例如:String,Integer,double)<br>
    不能充当的类型:ArrayList(没有实现Comparable),数组*
2. 元素能不能充当Key与元素本身有关,与元素的内部无关


# 增强for循环:
> 可以循环数组,collection.Map

结构: 
```
for(元素类型 变量:数组./Collection)
    循环体
}
```

###### 注意:

对于Map要进行特殊处理,要先转化成对应的Set,才能使用增强For循环


# Arrays类
> java.util.Arrays 类能方便地操作数组，它提供的所有方法都是静态的

具有以下功能：

- 给数组赋值：通过 fill 方法。
- 对数组排序：通过 sort 方法,按升序。
- 比较数组：通过 equals 方法比较数组中元素值是否相等。
- 查找数组元素：通过 binarySearch 方法能对排序好的数组进行二分查找法操作。


> 常用方法如下:

<table class="reference">
	<tbody>
		<tr>
			<th style="width:76px;">
				序号</th>
			<th style="width:501px;">
				方法和说明</th>
		</tr>
		<tr>
			<td style="width:76px;">
				1</td>
			<td style="width:501px;">
				<strong>public static int binarySearch(Object[] a, Object key)</strong><br>
				用二分查找算法在给定数组中搜索给定值的对象(Byte,Int,double等)。数组在调用前必须排序好的。如果查找值包含在数组中，则返回搜索键的索引；否则返回 (-(<em>插入点</em>) - 1)。</td>
		</tr>
		<tr>
			<td style="width:76px;">
				2</td>
			<td style="width:501px;">
				<strong>public static boolean equals(long[] a, long[] a2)</strong><br>
				如果两个指定的 long 型数组彼此<em>相等</em>，则返回 true。如果两个数组包含相同数量的元素，并且两个数组中的所有相应元素对都是相等的，则认为这两个数组是相等的。换句话说，如果两个数组以相同顺序包含相同的元素，则两个数组是相等的。同样的方法适用于所有的其他基本数据类型（Byte，short，Int等）。</td>
		</tr>
		<tr>
			<td style="width:76px;">
				3</td>
			<td style="width:501px;">
				<strong>public static void fill(int[] a, int val)</strong><br>
				将指定的 int 值分配给指定 int 型数组指定范围中的每个元素。同样的方法适用于所有的其他基本数据类型（Byte，short，Int等）。</td>
		</tr>
		<tr>
			<td style="width:76px;">
				4</td>
			<td style="width:501px;">
				<strong>public static void sort(Object[] a)</strong><br>
				对指定对象数组根据其元素的自然顺序进行升序排列。同样的方法适用于所有的其他基本数据类型（Byte，short，Int等）。</td>
		</tr>
	</tbody>
</table>	

# 可变参数

> 简化代码,方便操作

###### 注意点:
1. 给可变参数传值的实参,可以直接是多个值
2. 当包括可变参数在内有多个参数时,可变参数**必须放在最后面**,而且一个方法只能同时有一个可变参数
3. 当可变的参数方法与固定参数方法是**重载**关系的时候,**优先调用固定参数方法**

#### 可变长参数的定义

> 在具有可变长参数的方法中可以把参数当成数组使用，例如可以循环输出所有的参数值。

```
print(String... args){

   for(String temp:args)

      System.out.println(temp);

}
```

# Collection集合类

> 集合工具类,封装了集合的操作

- 要求:
1. 数据可以重复
2. 数据可以排序

> 常用方法:

```
public class Demo1 {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<String>();
        list.add("java");
        list.add("php");
        list.add("python");
        list.add("html");
        list.add("BigData");
        //自然顺序
        System.out.println("自然排序"+list);

        /**
         * 按照字典排序--默认都是升序
         * 注意:要想实现字典排序,必须是此案了Comparable接口的compareTo方法.
         */
        Collections.sort(list);
        System.out.println("字典排序"+list);

        /**
         * 按照从短到长排序,默认的方法无法实现,所以需要写自己的比较器.
         */
        ComWithLength com = new ComWithLength();
        Collections.sort(list,com);
        System.out.println("从短到长排序:"+list);
        /**
         * 按照从长到短排序,利用现有的比较器反转得到新的比较器
         */
        Comparator<String> order = Collections.reverseOrder(com);
        Collections.sort(list,order);
        System.out.println("从长到短:"+list);

        /**
         * reverse()默认的是字典排序的反转比较器
         */
        Comparator<String> objectComparator = Collections.reverseOrder();
        Collections.sort(list,objectComparator);
        System.out.println("字典排序反转:"+list);
        
        //现有顺序的反转
        Collections.reverse(list);
        System.out.println("现有顺序反转:" + list);
        
        //求最大值--按照字典顺序
        String max = Collections.max(list);
        System.out.println("字典排序的最大值:" + max);
        
        //按照自己的规则--从短到长
        String max1 = Collections.max(list,com);
        System.out.println("从短到长的顺序是:" + list);
  
    }
}
/**
 * 按照从短到长排序的比较器
 */
class ComWithLength implements Comparator<String>{

    @Override
    public int compare(String o1, String o2) {

        return o1.length()-o2.length();
    }
}

```

# 多线程

 **程序:** 一个可执行的文件<br>
 
 **进程;** 一个正在运行的程序,也可以理解成在内存中开辟了一块儿空间.<br>
 
 **线程:** 负责程序的运行,可以看做一条执行的通道或执行单位,所以我们通常将进程
      的工作理解成线程的工作.<br>

**进程中可不可以没有线程?** <br>
      必须有线程,至少有一个,当有一个线程的时候我们称之为单线程(唯一的线程就是主线程),当有一个以上的线程同时存在的时候我们称为多线程.<br>
 
 **任务区:** 我们将线程工作的地方称为任务区.
 每一个线程都有一个任务区,任务区通过对应的方法产生作用.

**JVM默认是多线程吗?**<br>
是的,至少有两个线程
1. 主线程: 任务区:main函数
2. 垃圾回收线程:任务区是:finalize函数

**多线程的作用:**<br>
为了实现同一时间干多件事情

**线程的生命周期:**<br>
线程是随着任务的开始而开始,结束而结束,只要任务没有结束,县城就不会结束,当线程还在工作的时候,进程就不会结束.
