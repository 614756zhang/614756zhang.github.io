---
layout: post
title: java自定义注解
category: java
tags: [java]
keywords: java,注解,自定义注解,annotation
---
## 一、创建自定义注解

**创建格式：**

*//引入元注解*

@Target( ElementType.METHOD)//说明注解的作用对象

@Retention(RetentionPolicy.RUNTIME)//说明注解的保留时间
```java
public @interface 注解名 {
    定义体
}
```

**使用@interface自定义注解时，自动继承了java.lang.annotation.Annotation接口**

## 二、元注解说明
**元注解的作用为负责注解其他注解，此为java 1.5 后提供的，定义了4种元注解：**

- ###### @Target                  表示支持注解的程序元素的种类
- ###### @Retention             表示注解类型保留时间的长短
- ###### @Documented        表示使用该注解的元素应被javadoc或类似工具文档化
- ###### @Inherited               表示一个注解类型会被自动继承

##### 1. @Target 用于描述注解的使用范围其取值有（取值的类为ElementType）：

- CONSTRUCTOR:用于描述构造器
- FIELD:用于描述域
- LOCAL_VARIABLE:用于描述局部变量
- METHOD:用于描述方法
- PACKAGE:用于描述包
- PARAMETER:用于描述参数
- TYPE:用于描述类、接口(包括注解类型) 或enum声明

##### 2. @Retention 表示需要在什么级别保存该注释信息，用于描述注解的生命周期其取值有（取值的类为RetentionPoicy）：
- SOURCE:在源文件中有效（即源文件保留）
- CLASS:在class文件中有效（即class保留）
- RUNTIME:在运行时有效（即运行时保留）
    
##### 3. @Documented用于描述其它类型的注解
该注解被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。Documented是一个标记注解，没有成员。
##### 4. @Inherite元注解是一个标记注解
@Inherited阐述了某个被标注的类型是被继承的。如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。

## 三、定义体
**定义注解的参数示例：**

```java
public String value() default "";
```

1. 方法修饰符只能为public或默认；
2. default为默认值（默认值可不写，public String value() 也可以）；
3. 返回类型只能为基本数据类型，String，枚举（enum），Class，Annotation 以及以上的数组
4. 如果只有一个参数，参数名为value（推荐这样写），多个参数则写多个方法，参数名为定义的参数名，例：
```java
 public String name();
 public int age();
```

## 四、注解使用示例：
**定义注解**
```java
@Target(ElementType.TYPE)
@Retention(RetentionPoicy.RUNTIME)
public @interface ClassAnnotation{
    String value();
}
```
```java
@Target(ElementType.METHOD)
@Retention(RetentionPoicy.RUNTIME)
public @interface MethodAnnotation{
    String name();
    public int age() default 1;
}
```
**注解使用**
```java
@ClassAnnotation("userAnnotation")
public Class UseAnnotation{
    @MethodAnnotation(name="zhangtest")
    public void test(){
         System.out.println("123");
    }

    @MethodAnnotation(name="peng234",age=12)
    public void test1(){
         System.out.println("34535");
    }
}
```
**（1）、使用时，直接@+你定义的注解名加到要使用的类或方法上即可；**

**（2）、括号里写参数，若未写则使用注解定义的默认值；**

**（3）、若自定义注解只有一个参数，并命名为value，则括号里不用指定参数名称，直接写参数值 ；例如上面的ClassAnnotation注解**

## 五、提取注解参数
```java
public static void main(String[] args) throws Exception {  
        // 提取到被注解的方法Method，这里用到了反射的知识  
		Class cl = Class.forName("com.test.AnnotationTest");
		ClassAnnotation classAnnotation= (ClassAnnotation) cl.getAnnotation(ClassAnnotation.class); 
		// 得到注解的参数  
        System.out.println(classAnnotation.value());  

        Method method = cl.getDeclaredMethod("test");  
        // 从Method方法中通过方法getAnnotation获得我们设置的注解  
        MethodAnnotation methodAnnotation = method.getAnnotation(MethodAnnotation.class); 
        // 得到注解的俩参数  
        System.out.println(methodAnnotation.name());  
        System.out.println(methodAnnotation.age());  

        Method method1 = cl.getDeclaredMethod("test1");  
        // 从Method方法中通过方法getAnnotation获得我们设置的注解  
        MethodAnnotation methodAnnotation1 = method1.getAnnotation(MethodAnnotation.class); 
        // 得到注解的俩参数  
        System.out.println(methodAnnotation1.name());  
        System.out.println(methodAnnotation1.age());  

    }  
```

## 六、注解使用场景
##### 1.  spring的相关注解
spring的相关注解是日常使用频率比较高的，使用了spring的相关注解，是大大提高了工作效率，是使开发人员从繁琐的xml配置文件中解脱出来，其注解的工作原理大致流程为，先在包中定义自己的注解，然后你在工程中需要的地方声明注解，然后在配置文件中开启注解（准确的说应该是开启扫描），spring在启动的时候会对你指定的包下的所有类进行扫描，扫描出自己的注解和注解参数，然后处理初始化bean，而后将得到的bean交由spring的容器管理
##### 2. javaEE的相关注解
原理类同上述spring
##### 3. 自定义的注解使用
使用自定义注解可根据实际工程需要自行设计使用，这里提供一个本人接触过得关于接口的调用的一个自定义的注解使用场景：

1、暴露给外围接口调用，有多个场景的多个接口，考虑安全问题和所有接口协议类型、结构一致问题，在暴露一个接口地址的前提下，支持所有的接口调用；

2、暴露一个接口，然后在这个接口中解析报文中的接口编码，然后扫描所有的已标注注解，将接口编码与注解参数匹配，匹配到即将报文反射调用到该方法上得到结果，然后返回结果给外围

[个人博客主页](https://614756zhang.github.io/zhangpeng/) - [微博](http://weibo.com/614756zhang) - [Github](https://github.com/614756zhang)