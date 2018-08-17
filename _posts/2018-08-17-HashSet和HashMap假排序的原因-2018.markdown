---
layout:     post
title:      "HashSet和HashMap假排序的原因"
subtitle:   " HashSet是无序的,这个无序指的是:存入的顺序和取出的顺序不同"
date:       2018-08-17 10:00:00
author:     "Chamber"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - java
---


> 在学习Java的时候了解到HashSet是无序的,这个无序指的是:存入的顺序和取出的顺序不同<br>

下图说明了HashSet的存储结构

![image](https://note.youdao.com/favicon.ico)


很好理解,当球放入桶的时候,在里面是随机打乱的,取的时候也没有顺序可言.

我们在使用HashSet的时候都知道，Set的特性是元素唯一，并且乱序存储。但是在实际时候的时候。会有下面的情况发生：

> 源码如下:

```
 publicvoid test1() throws InterruptedException{
long randomSeed= System.currentTimeMillis() ;
Random random = new Random(45) ;
Set randomSet = new HashSet() ;
System.out.println("开始放入：");
for(int i = 0 ; i < 100 ; i++){
   Integer in = random.nextInt(10) ;
   System.out.print(in+" , ");
   randomSet.add(in) ;
   TimeUnit.MILLISECONDS.sleep(2);
} 
System.out.println() ;
System.out.println("结果:");
System.out.println(randomSet.toString()) ;
}


```

> 以上代码的输出：

```
//开始放入：
//9 , 1 , 1 , 0 , 7 , 1 , 1 , 3 , 4 , 3 , 0 , 9 , 0 , 8 , 0 , 6 , 4 , 5 , 0 , 7 ,2 , 1 , 4 , 4 , 8 , 9 , 7 , 0 , 9 , 1 , 2 , 3 , 1 , 2 , 4 , 3 , 9 , 0 , 7 , 0 ,1 , 1 , 2 , 4 , 3 , 1 , 7 , 3 , 2 , 1 , 9 , 0 , 2 , 2 , 2 , 3 , 6 , 5 , 8 , 3 ,1 , 0 , 3 , 4 , 0 , 1 , 3 , 2 , 4 , 8 , 7 , 1 , 5 , 1 , 4 , 1 , 7 , 9 , 3 , 4 ,5 , 3 , 7 , 1 , 9 , 0 , 0 , 3 , 4 , 7 , 4 , 1 , 8 , 7 , 4 , 3 , 8 , 9 , 2 , 2, 

//结果:
//[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

- 结果显示,是排序过的

这和我们所了解到的内容完全不一样,经过查阅源码得知,hashset其实是通过hashmap来实现

通过查阅源码,发现map中的put实际如下:

```
public Vput(K key, V value) {
    if (key== null)
       return putForNullKey(value);
    int hash = hash(key.hashCode());//在这里获取key 的map值
    int i = indexFor(hash, table.length);
    for(Entry<K,V> e = table[i]; e != null; e = e.next) {
       Object k;
        if(e.hash == hash && ((k = e.key) == key || key.equals(k))) {
           V oldValue = e.value;
           e.value = value;
           e.recordAccess(this);
            return oldValue;
        }
    }
   modCount++;
   addEntry(hash,key, value, i);
   returnnull;
}

```

然后通过这个Hash值计算到它在table中的位置。然后通过addEntry进行保存。这里的key.hashCode()方法就是最开始导致HashSet().toString()排序假象的元凶。如果key是Integer类型。那么他们的hashCode计算结果是其本身。下面的Integer的hashCode()源码：

```
    public int hashCode() {
         return value;
    }
```

这就导致了在散列分布的时候，实际上hashMap中的key值就是随机写入的Integer类型的值。也就是为什么toString()方法返回了一个看似正向排序的结果。但如果按照以上的说法，这种“排序”应该是正确的，为什么说是假象呢？是因为在每一回进行addEntry(hash, key, value, i);的时候，都会判断当前的table长度是否足够hashcode算出来的索引，即i值，如果它超出了索引，就将table的大小乘2，用来扩充实际的大小。上面的例子中，只取了前10个元素。那么他们不存在增大的部分，看起来像是被排序了。