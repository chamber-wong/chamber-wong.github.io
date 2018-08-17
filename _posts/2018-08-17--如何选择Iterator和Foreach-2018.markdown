---
layout:     post
title:      "如何选择Iterator和Foreach"
subtitle:   " 既然有了foreach这种简单的方法,谁还会去用迭代器,简直就是有毛病"
date:       2018-08-17 10:00:00
author:     "Chamber"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
---


```
Map<String, String> map = new HashMap<>();
map.put("01", "java");
map.put("03", "php");
map.put("05", "BigData");
map.put("02", "python");

System.out.println(map);
```
> 想要对map进行遍历的时候,有很多种方法选择

- 遍历方法一   

```
//通过Set<K> keySet()  
Set<String> set1 = map.keySet();
//获取迭代器
Iterator<String> iterator = set1.iterator();
while (iterator.hasNext()) {
	String key =  iterator.next();
	
	//再去获取value
	System.out.println("key:"+key+"   value:"+map.get(key));
}
```
- 遍历方法二
```
//Set<Map.Entry<K,V>> entrySet() 
//先得到装着entry的set
Set<Map.Entry<String, String>> set = map.entrySet();
//获取迭代器
Iterator<Map.Entry<String, String>> iterator2 = set.iterator();
while (iterator2.hasNext()) {
	Map.Entry<String, String> entry = iterator2.next();
	//entry.setValue("hah");
	//再去获取key,value
	System.out.println("key:"+entry.getKey()+"   value:"+entry.getValue());
}
System.out.println(map);
```

- 遍历方法三
```
//使用foreach  
for (Entry<String, String> entry : map.entrySet()) {
	System.out.println("key:"+entry.getKey()+"   value:"+entry.getValue());
}
```

**显而易见**使用foreach非常的方便,极大提高了效率

> 既然有了这种简单的方法,谁还会去用迭代器,简直就是有毛病

### 但是,事情不是想象中的那么简单:

经过多方调研发现,foreach是一种"语法糖",所谓语法糖就是通过编译器或者其它手段优化了代码，给使用带来了遍历。也就是,Java给开发者的福利.那么foreach的使用条件是什么呢?

##### 要求：数组或java.lang.Iterable。

所以,在内部foreach是直接调用了Iterable迭代器,这样我们就可以想象到,如果在foreach中对collection进行遍历的时候,就不能使用collection.remove(),下面就行一些验证:

```
for (String i : list) {
    System.out.println(i);
    list.remove(i); //问题出现
} 
```
这里抛出了```ConcurrentModificationException```异常

- 这样就可以使用Iterator的遍历方法

代码如下:
```
Iterator it=list.iterator();
while (it.hasNext()){
    System.out.println(it.next());
    it.remove(); // 通过
}
```
这里使用的是Iterator自带的remove方法.不会出现问题

# 总结
1. 当只进行遍历查询的时候可以使用foreach来简化代码
2. 如果涉及到删除操作,foreach会出现问题,必须使用Iterator
