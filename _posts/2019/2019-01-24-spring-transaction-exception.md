---
layout: post
title: spring事物异常回滚
category: spring
tags: [spring]
keywords: spring,事物,Transaction,exception
---
## 一、异常分类（从事物方面看）：

**1.运行时异常(uncheckException)**,即RuntimeException异常及其子类，如空指针异常、强转异常等，这种异常是可以写代码不捕获的，但运行时会抛出异常
	
**2.已检查异常(checkException)**，即已知异常，必须处理，不处理的话是不能进行编译的，但是checkException不是一个具体的异常类型，它只是一个概念；
	
## 二、spring的事务边界

**spring的事务边界是在调用业务方法之前开始的，业务方法执行完毕之后来执行commit or rollback**

当方法结束时（也许正常结束，也许异常中断结束），spring执行commit or rollback默认取决于是否抛出runtime异常
- 有运行时异常(uncheckException) 即有RuntimeException异常抛出，则执行rollback，回滚数据并继续将异常抛向上一级方法；
- 无运行时异常(uncheckException)，则执行commit，数据提交，方法结束

简言之抛出RuntimeException并在你的业务方法中没有catch处理的话，事务会回滚。

一般不需要在业务方法中catch异常，如果非要catch，在做完你想做的工作后（比如关闭文件等）一定要抛出RuntimeException，
否则在事物方法中就无运行时异常(uncheckException)，spring会认为你的操作完成并且无异常发生，将你的操作commit,这样就会产生脏数据，所以你的catch代码是画蛇添足。

**如：**
```java
try {  
    //bisiness logic code  
} catch(Exception e) {  
    //handle the exception  
} 
```
 
由此可以推知，在spring中如果某个业务方法被一个整个try catch包裹起来并异常捕获内部消化处理，则这个业务方法也就等于脱离了spring事务的管理，
因为没有任何异常会从业务方法中抛出！全被捕获并吞掉，导致spring异常抛出触发事务回滚策略失效。
不过，如果在catch代码块中采用页面硬编码的方式使用spring api对事务做显式的回滚，这样写也未尝不可（即调用spring api 手动回滚）。
**如：**
```java
try {  
    //bisiness logic code  
} catch(Exception e) {  
	//handle the exception 
    //我把异常处理了，，然后我就不抛出去，，我自己处理，我自己回滚
	TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
} 
```
 
## 三、Spring的AOP事务即声明式事务管理
**spring的AOP事务默认是针对checkException不回滚，uncheckException回滚**

也就是默认对RuntimeException()异常或是其子类进行事务回滚，其他异常不回滚；
如果使用try-catch捕获抛出的unchecked异常后没有在catch块中采用页面硬编码的方式使用spring api对事务做显式的回滚，则事务不会回滚，
“将异常捕获,并且在catch块中不对事务做显式提交=生吞掉异常” ，要想捕获非运行时异常则需要如下配置：

**解决办法：** 
1. 在针对事务的类中抛出RuntimeException异常，而不是抛出Exception。
2. 增加rollback-for，里面写自己的exception，例如自己写的exception


## 四、基于注解的事务

Transactional的异常控制，默认也是checkException不回滚，uncheckException回滚

**如果配置了rollbackFor 和 noRollbackFor 且两个都是用同样的异常，那么遇到该异常，还是回滚**

rollbackFor和noRollbackFor配置也许不会含盖所有异常，对于遗漏的继续按照checkException不回滚，uncheckException回滚

[个人博客主页](https://614756zhang.github.io/zhangpeng/) - [微博](http://weibo.com/614756zhang) - [Github](https://github.com/614756zhang)
