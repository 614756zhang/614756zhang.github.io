---
layout: post
title: 单元测试-SpringTest与JUnit整合
category: spring
tags: [SpringTest]
keywords: SpringTest,JUnit
---

**前言：** 测试是一个开发人员在开发后必需的环节，单元测试在实际工作中有很频繁的应用，当你只想测试某一模块而非启动整个项目时，单元测试就体现了它的意义了，本文叙述一种SpringTest与JUnit结合的单元测试方法。

### 一、环境背景
- IDE：eclipse
- 项目构建工具：gradle
- 项目框架：spring+activiti（本人研究工作流临时搭建的demo）

### 二、SpringTest和JUnit引入
在原有工程的build.gradle中的dependencies下加入SpringTest和JUnit的jar包依赖关系：
```gradle
    testCompile 'junit:junit:4.12'
    testCompile 'org.springframework:spring-test:4.1.5.RELEASE'
```
然后重新gradle编译一下工程，待工程下载对应jar后即可

*上诉为gradle工程的方式，maven工程请添加maven格式的依赖配置，普通工程请直接下载jar到工程下（所需jar包junit-4.12.jar、hamcrest-core-1.3.jar、spring-test-4.1.5.RELEASE.jar）*

### 三、单元测试配置
##### 1、要测试的类上加测试注解
```java
@RunWith(SpringJUnit4ClassRunner.class) //使用junit4进行测试
@ContextConfiguration(locations={"classpath:spring-activiti.xml"}) //加载配置文件 
@TransactionConfiguration(transactionManager = "transactionManager", defaultRollback = true)
@Transactional
public class DeploymentProcessTest {

}
```
**注解说明：**
- **RunWith:** 运行器，若不配置参数则默认使用JUnit4.class，本demo使用spring-test的SpringJUnit4ClassRunner.class，这个运行器集成了spring的一些功能，使用起来更方便

- **ContextConfiguration:** 加载配置文件,测试该类时需要加载那些配置文件，例如本demo中则需要加载流程引擎的一些配置；若有多个配置文件需加载，可配置多个，以逗号分开，例如：@ContextConfiguration(locations={"classpath:config-a.xml","classpath:config-b.xml"})

- **TransactionConfiguration:** 数据库回滚的注解,Spring的项目中做单元测试时,声明单元测试事物回滚；***TransactionConfiguration注解为较老的版本，因本demo使用的spring版本为4.1.5.RELEASE，故使用该名称，高版本的Spring框架中（Spring4.2以后），@TransactionConfiguration已经标注为过时的注解，查看官方文档会发现，替代的方式为： @Rollback***

- **Transactional：** spring事物的注解，可参看spring事物文章了解

**使用说明：** 若单元测试处无需控制事物或更本就没有事物，可不写后面两个注解（TransactionConfiguration、Transactional）

##### 2、给具体执行测试的方法上加测试注解
```java
@RunWith(SpringJUnit4ClassRunner.class) //使用junit4进行测试
@ContextConfiguration(locations={"classpath:spring-activiti.xml"}) //加载配置文件 
@TransactionConfiguration(transactionManager = "transactionManager", defaultRollback = true)
@Transactional
public class DeploymentProcessTest {

	@Autowired //自动注入
	private ProcessEngine processEngine;
	
	@Before
	public void beforeDeploymentProcessDefinitionTest() {
		System.out.println("执行deploymentProcessDefinitionTest方法前执行");
	}
	
	@After
	public void afterDeploymentProcessDefinitionTest() {
		System.out.println("执行deploymentProcessDefinitionTest方法后执行");
	}
	
	/** 部署流程定义 */
	@Test
	public void deploymentProcessDefinitionTest() {
	    RepositoryService repositoryService = processEngine.getRepositoryService();
	    // 创建一个部署对象DeploymentBuilder，用来定义流程部署的相关参数
	    DeploymentBuilder deploymentBuilder = repositoryService.createDeployment();
	    // 添加部署的名称
	    deploymentBuilder.name("process-zhangpengTest");
	    // 添加process.bpmn20.xml
	    deploymentBuilder.addClasspathResource("process.bpmn20.xml");
	    // 部署流程定义
	    Deployment deployment = deploymentBuilder.deploy();
	    System.out.println("部署ID：" + deployment.getId());//1
	    System.out.println("部署名称：" + deployment.getName());
	}
}
```
在你需要测试的方法上加Test注解即可

**说明：** 若测试某个方法直接加Test注解即可，若要在测试该方法前后执行其他方法，例如执行A方法时要查询数据库，需在执行A方法前创建数据库连接，在执行A方法后关闭数据连接，那么就需要Before和After注解，Before标注的方法会在Test标注的方法前执行，After会在Test标注的方法后执行，每次都是；

## 四、eclipse中的使用
这里示例用debug执行说明，run的是一样的，在该类中右击>Debug as > JUnit Test 即可

而后可在eclipse的Juint框中查看执行结果
![]({{ site.url }}{{ site.baseurl }}/assets/images/2019/2019-02-19-SpringTest-Junit-1.png)
同时可查看控制台输出日志

