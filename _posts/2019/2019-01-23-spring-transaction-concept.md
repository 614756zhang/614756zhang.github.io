---
layout: post
title: spring事务概念
category: spring
tags: [spring]
keywords: Spring,spring,事物,Transaction
---

# 一、事务概述
###### 1. 事务有四个特性（ACID）：
原子性、一致性、隔离性、持久性
###### 2. spring事务支持：
Spring可以使用编程式事务，也可以使用声明式事务。一般是用声明式

# 二、Spring事务管理器

Spring只提供事务管理器的接口，具体内容有各个事务管理器来实现常用的事物管理器：JDBC事务、Hibernate事务、JPA事务(Java持久化API事务)、JTA事务(Java原生API事务)

**spring提供的内置事务管理器实现:**

- DataSourceTransactionManager数据源事务管理器，提供对单个javax.sql.DataSource事务管理，用于Spring JDBC抽象框架、iBATIS或MyBatis框架的事务管理

- HibernateTransactionManager
用于集成Hibernate框架时的事务管理；
该事务管理器只支持Hibernate3+版本，且Spring3.0+版本只支持Hibernate3.2+版本

# 三、事务属性
**事务属性有五个方面： 隔离级别，传播行为，事务超时时间，回滚规则，是否只读。**
 
#### 1. 隔离级别（isolation level）
首先先说一下数据库的操作可能出现的几个不确定情况：
- 更新丢失(Lost update)：同时对某条数据更新，后者覆盖了前者，造成跟新丢失；
- 脏读（Dirtyreads）：一个事务读取到了另一个事务未提交的数据操作结果；
- 不可重复读（Non-repeatable Reads）：同一条数据读两次，结果不同；
- 幻读（Phantom Reads）：两次查询，后者比前者少或多了数据

*“脏读”、“不可重复读”和“幻读”，其实都是数据库读一致性问题*

**事务隔离级别：**
- 未提交读取（Read Uncommitted）：允许脏读取，但不允许更新丢失。Spring标识：ISOLATION_READ_UNCOMMITTED。
- 已提交读取（Read Committed） ：允许不可重复读取，但不允许脏读取。Spring标识：ISOLATION_READ_COMMITTED。
- 可重复读取（Repeatable Read） ：禁止不可重复读取和脏读取，可能出现幻读数据。Spring标识：ISOLATION_REPEATABLE_READ。
- 序列化（Serializable） ：提供严格的事务隔离，要求事务序列化执行
    Spring标识：ISOLATION_SERIALIZABLE。

*Spring中同时提供一个标识：ISOLATION_DEFAULT，表示使用后端数据库默认的隔离级别，比如Sql Server , Oracle，MySQL的默认隔离级别是Repeatable read。*

| 隔离级别             | 含义         |
| :------------------- | :------------|
|ISOLATION_DEFAULT| 使用后端数据库默认的隔离级别|
|ISOLATION_READ_UNCOMMITTED| 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读|
|ISOLATION_READ_COMMITTED| 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生|
|ISOLATION_REPEATABLE_READ| 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生|
|ISOLATION_SERIALIZABLE| 最高的隔离级别，完全服从ACID的隔离级别，确保阻止脏读、不可重复读以及幻读，也是最慢的事务隔离级别，因为它通常是通过完全锁定事务相关的数据库表来实现的|

#### 2. 传播行为
**事务传播行为（propagation behavior）指的就是当一个事务方法被另一个事务方法调用时，	这个事务方法应该如何进行**。 例如：methodA事务方法调用methodB事务方法时，methodB是继续在调用者methodA的事务中运行呢，还是为自己开启一个新事务运行，这就是由methodB的事务传播行为决定的。
| 传播行为             | 含义         |
| :------------------- | :------------|
|PROPAGATION_REQUIRED| 表示当前方法必须运行在事务中。如果当前事务存在，方法将会在该事务中运行。否则，会启动一个新的事务|
|PROPAGATION_SUPPORTS| 表示当前方法不需要事务上下文，但是如果存在当前事务的话，那么该方法会在这个事务中运行|
|PROPAGATION_MANDATORY| 表示该方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常|
|PROPAGATION_REQUIRED_NEW| 表示当前方法必须运行在它自己的事务中。一个新的事务将被启动。如果存在当前事务，在该方法执行期间，当前事务会被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager|
|PROPAGATION_NOT_SUPPORTED| 表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager|
|PROPAGATION_NEVER| 表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常|
|PROPAGATION_NESTED| 表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与PROPAGATION_REQUIRED一样。注意各厂商对这种传播行为的支持是有所差异的。可以参考资源管理器的文档来确认它们是否支持嵌套事务|
 
#### 3. 事务超时时间
即字面意义，是对事务超时的控制，长时间的事物会占用数据库资源，设置超时时间，时间内没有执行完，会自动回滚结束，不会一直等待。

#### 4. 是否只读
字面意义，是否为只读事务，readOnly = true，只读事务。由于只读事务不存在数据的修改，因此数据库将会为只读事务提供一些优化手段，例如Oracle对于只读事务，不启动回滚段，不记录回滚log。

#### 5. 回滚规则
哪些异常会导致事务回滚而哪些不会。默认情况下，事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚