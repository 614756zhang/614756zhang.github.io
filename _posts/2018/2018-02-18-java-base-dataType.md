---
layout: post
title: java封装类和基本数据类型的比较
category: java
tags: [java]
keywords: java,dataType,封装类,基本数据类型
---

# java封装类和基本数据类型的比较


**本文为本人学习心得笔记，如有不对的地方，望不吝指出**

**在实际代码开发中总是会碰到封装类和基本数据类型的比较，但相同的值比较结果是否为true，总是似是而非，本人对几种类型的比较做了以下实验：**


## 一、 Integer和int的比较
-----------------------
**代码为：**
```java
int num1 = 127;
Integer num2 = 127;
Integer num3 = num1;
Integer num4 = 128;
Integer num5 = 128;
int num6 = 128;
System.out.println("num1 == num2 的结果为："+(num1 == num2));
System.out.println("num2 == num3 的结果为："+(num2 == num3));
System.out.println("num4 == num5 的结果为："+(num4 == num5));
System.out.println("num5 == num6 的结果为："+(num5 == num6));
```
**结果为：**
```java
num1 == num2 的结果为：true
num2 == num3 的结果为：true
num4 == num5 的结果为：false
num5 == num6 的结果为：true
```
**结果分析：**

 1. num1 == num2 的结果为：true，可见Integer对象和int数据类型比较，只要数值相同，==比较的结果为true；**其比较原理为：先将num2对象调用其intValue方法拆箱成基本数据类型int，然后在用拆箱后的int类型的值和num1比较，故其结果为true；**另：其几种类型和其封装类比较亦是如此；
 
 2. num2 == num3 的结果为：true，这可能会奇怪，这个明明是两个对象的比较，怎么会true呢，其实这个是Integer的缓存原因，在`Integer num2 = 127;`这句代码赋值中，前面定义的是封装类声明，而后面是基本数据类型int的值，按理说是会报错的，但java有自己的自动拆箱（unboxing）&自动装箱(boxing)机制，此时会将后面的int基本数据类型装箱封装成Integer对象，**而装箱调用的是Integer.valueOf(int i)方法**，说了这么多终于说到重点了，缓存就是在valueOf这个方法中的，让我们先看这段代码：
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2018/2018-02-18-java-base-dataType-1.jpg)
当i的值在low和high之间时，是不创建对象的，直接在缓存中取，这里low为-128，high为127，**即当int值为-128~127之间时，自动装箱时，是不创建对象的，直接从缓存中获取**，因为num2和num3在自动装箱时均是在缓存中获取的，故均为同一个对象，所以同对象相比较当然是true了；其它封装类的比较亦是如此（Double和Float除外，这两个没有缓存）

 3. num4 == num5 的结果为：false，经过上述2中的解释，这个结果的原因就显然易得了，因为128不在缓存的范围内，所以各自创建了对象，不同对象相比较，结果为false；
 
 4. num5 == num6 的结果为：true，此为验证int和Integer值相同相比较是否受缓存限制，结果为true，显然是不受的，，但此步有点鸡肋了，细想这个的1中已经解释清楚了，是拆箱，拆箱后就都是基本数据类型了；
 

## 二、 Long和int的比较
---------------------
关于Long和long的比较久不在赘述了，其相关比较和实验一中相似
**代码为：**
```java
Long lon = 130L;
int in = 130;
System.out.println("lon == in 的结果为："+(lon == in));
```

**结果为：**
```java
lon == in 的结果为：true
```
**结果分析：**
 1. 其实这个也很好理解，，只是在其中多了个基本数据类型的隐式转换，先是将Long对象拆箱为long基本数据类型，然后in会隐式转换为long，然后比较，所以结果为true；
 2. 这里不单单是Long和int，只要是封装类和基本数据类型相比较，只要实际值相同，结果一般都应为true(Boolean 除外)；


三、隐式转换
------

由低--->高，可以隐式自动转换，数据类型将自动提升。
其转换顺序为：
byte--->short--->int--->long--->float--->double
char-->int-->.......(后面同上)