---
layout: post
title: spring事务配置使用
category: spring
tags: [spring]
keywords: spring,config,事物,Transaction
---
## 一、spring事物简介
**spring事物的用法有两类四种：**
**1. 编程式事务管理(基于Java编程控制)**
（1）、利用TransactionTemplate，只需在配置中配置一个事物模板，在使用是获取实例即可
**2. 声明式事务管理(基于Spring的AOP配置控制)**
（2）、基于TransactionProxyFactoryBean的方式，需要为每个进行事务管理的类,配置一个TransactionProxyFactoryBean进行增强.（使用较少）
（3）、基于XML配置(经常使用) ，  一旦配置好之后,类上不需要添加任何东西
（4）、基于注解(配置简单，经常使用)， 在applicationContext.xml中开启事务注解配置，在目标组件类中使用@Transactional

## 二、spring事物使用
### 1、编程式事务管理-事物模板
**使用示例：**
**配置：**
```xml
<!-- 配置事务管理器 -->  
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
        <property name="dataSource" ref="dataSource" />  
    </bean>  
    
	<!-- 1、模板事务-->
	<!-- 此处配置了事务模板，在代码中使用时，获取到bean的模板实例即可-->
	<!--事务模板 -->
 	<bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
 		<property name="transactionManager" ref="transactionManager" /> 
 		
		<!-- ISOLATION_DEFAULT 表示由使用的数据库决定  -->
 		<property name="isolationLevelName" value="ISOLATION_DEFAULT" /> 
		<property name="propagationBehaviorName" value="PROPAGATION_REQUIRED" /> 
	</bean> 
```
**代码使用：**
```java
//获取事务模板实例
	@Autowired
	TransactionTemplate transactionTemplate;
	
	//获取数据持久层实例
	@Autowired
	TransactionDAO transactionDAO;
	public int test(){
		System.out.println("进入了模板事务的测试方法~！");
		try{
		   transactionTemplate.execute(new TransactionCallback() {
				@Override
				public Object doInTransaction(TransactionStatus ts) {
					System.out.println("进入了事务！");
					Map map = new HashMap();
					int a = transactionDAO.insertone(map);
					System.out.println("执行了sql！");
					//报错，回滚
					Object b = null;
//					b.equals("123");
					System.out.println("回滚失败！");
					return 1;
				}
			});
		}catch(Exception e){
			e.printStackTrace();
		}
		System.out.println("结束了模板事务的测试方法~！");
		return 1 ;
	}
```
-  使用时在配置文件中配置事务模板，代码中使用直接获取模板实例即可，调用模板实例的execute的方法，并在匿名内部类中写业务逻辑，在匿名类中的一但有异常，会将匿名类中的所有已做的数据库操作回滚
- 配置中的isolationLevelName（隔离属性）和propagationBehaviorName（传播行为），配置内容后面一并说明

### 2、基于TransactionProxyFactoryBean的方式，不常用，现在先不研究了；
简单看了一下，，要每个DAO层都要配置一下，有点费事，，不好用，可以不看了
### 3、基于XML配置，用aop的切面织入，好处是添加或修改事务不需改任何代码，只改配置即可
**使用示例**
**配置：**
```xml
    <!-- 配置事务管理器 -->  
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
        <property name="dataSource" ref="dataSource" />  
    </bean> 
<!-- 2、aop配置事务 -->
	<!-- 基于切面配置事务，对代码无入侵，添加和修改配置即可实现事务控制 -->
	<!-- 配置事务通知 -->
	<tx:advice id="aopAdvice" transaction-manager="transactionManager">
		<!-- 配置事务的传播特性、超时时间、隔离级别、是否只读、回滚规则以及次事务作用匹配的方法名 -->
		<tx:attributes>
			<!-- 匹配test开头的方法，传播特性为REQUIRED（前事务存在，方法将会在该事务中运行。否则，会启动一个新的事务） 
				 隔离级别为默认级别（即数据库默认的隔离级别）-->
			<tx:method name="test*" propagation="REQUIRED" isolation="DEFAULT"/>
		</tx:attributes>
	</tx:advice>
	<!-- 配置事务切入点，构成切面 -->
	<aop:config>
		<!-- 配置方法1  -->
		<!-- 切入点 -->
		<aop:pointcut expression="execution(* com.tydic.zhang.testTransaction.XmlAOPTransaction.*(..))" id="aopPointcut" />
		<!-- 通知连接切入点 -->
		<aop:advisor advice-ref="aopAdvice" pointcut-ref="aopPointcut"/>
		<!-- 配置方法2  -->
		<!-- <aop:advisor advice-ref="" pointcut="execution(* com.com.tydic.zhang.testTransaction.*.*(..))"/> -->
	</aop:config>
```
代码无需修改，，配置中配置的匹配表达式会找到对应的方法添加事务
此种用法的好处是对代码零入侵，全配置，若后面想修改事务仅修改配置即可，缺点可能就是配置上的理解吧，要有aop的基础知识和实现原理的了解，方能更好的理解此处配置

### 4、基于注解，开启注解，在代码方法上添加注解即可
**使用示例**
**配置：**
```xml
<!-- 配置事务管理器 -->  
 <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
        <property name="dataSource" ref="dataSource" />  
    </bean> 
<!-- 3、注解 -->
<!--开启注解，使用注释事务 -->  
<tx:annotation-driven  transaction-manager="transactionManager" /> 
```
 ***MyBatis自动参与到spring事务管理中，只要org.mybatis.spring.SqlSessionFactoryBean引用的数据源与DataSourceTransactionManager引用的数据源一致即可，否则事务管理会不起作用。***
**代码：**

```java

```

**@Transactional注解
@Transactional属性** 

| 属性 | 类型| 描述|
| :------| :-----| :-----|
| value |String | 可选的限定描述符，指定使用的事务管理器|
|propagation |enum: Propagation |可选的事务传播行为设置|
|isolation |enum: Isolation |可选的事务隔离级别设置|
|readOnly |boolean |读写或只读事务，默认读写|
|timeout |int (in seconds granularity) |事务超时时间设置|
|rollbackFor |Class对象数组，必须继承自runtime exception|导致事务回滚的异常类数组|
|rollbackForClassName |类名数组，必须继承自runtime exception|导致事务回滚的异常类名字数组|
|noRollbackFor |Class对象数组，必须继承自runtime exception|不会导致事务回滚的异常类数组|
|noRollbackForClassName |类名数组，必须继承自runtime exception|不会导致事务回滚的异常类名字数组|
 **@Transactional 可以作用于接口、接口方法、类以及类方法上**。当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。

 虽然 @Transactional 注解可以作用于接口、接口方法、类以及类方法上，但是 Spring 建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。另外， ***@Transactional 注解应该只被应用到 public 方法上，这是由 Spring AOP 的本质决定的。如果你在 protected、private 或者默认可见性的方法上使用 @Transactional 注解，这将被忽略，也不会抛出任何异常。***
 
 **默认情况下，只有来自外部的方法调用才会被AOP代理捕获，也就是，类内部方法调用本类内部的其他方法并不会引起事务行为，即使被调用方法使用@Transactional注解进行修饰。**