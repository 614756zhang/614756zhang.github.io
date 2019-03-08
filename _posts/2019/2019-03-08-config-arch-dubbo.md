---
layout: post
title: dubbo.properties配置文件路径修改
category: config
tags: [dubbo]
keywords: dubbo,dubbo.properties,dubbo配置文件路径,架构,config
---
## 一、问题：
### 修改dubbo.properties加载路径
在使用dubbo中，我们用到dubbo.properties配置文件，但dubbo默认的是加载classpath 根目录下的dubbo.properties，这显示不能满足所有的项目架构，我需要更加灵活的放置dubbo.properties的路径，那么就有了该问题，如何更改默认的加载路径？

## 二、解决：
**阅读官方文档后得到以下信息：**

*Dubbo 将自动加载 classpath 根目录下的 dubbo.properties，可以通过JVM启动参数 -Ddubbo.properties.file=xxx.properties 改变缺省配置位置。*

那么解决的方法来了，修改启动参数
### 方法1-修改JVM启动参数：

修改JVM启动参数，在现有启动参数后面添加-Ddubbo.properties.file=xxx.properties

**文件路径配置示例：**
```
    -Ddubbo.properties.file="/D:\software\apache-tomcat\apache-tomcat-8.5.35-mrm\config/dubbo.properties"
```
*注意：由于dubbo jar包里面ConfigUtil.class类中对路径有判断，当为/开头时，为文件路径，否者就是classpath路径；文件路径前一定要加"/",否者dubbo会按classpath路径去加载。ps：此只对window路径，Linux路径是默认/开头的*

**classpath路径配置示例：**
```
    -Ddubbo.properties.file="config/dubbo.properties"
```
这里配置的是加载classpath根目录下的config下的dubbo.properties

### 方法2-自定义ContextListener：
方法1简单快捷，直接修改命令参数即可，但显然修改命令参数，在后期需要实施，测试等相关人员的修改，也需要知道具体路径，这中间有许多沟通问题，在交付中显示不是很方便，那么就有了该方法，自定义ContextListener，我们在项目启动时，通过代码来改变环境变量即可

**代码示例：**
```java
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
import com.alibaba.dubbo.common.Constants;
public class DubboServletContextListener implements ServletContextListener  {

	@Override
	public void contextInitialized(ServletContextEvent sce) {
		// TODO Auto-generated method stub Ddubbo.properties.file
		String dubboPath = sce.getServletContext().getInitParameter("dubboConfiguration");
		System.out.println("*********dubbo.properties dir:" + dubboPath);
		if (dubboPath.startsWith("classpath:")) {
			dubboPath = dubboPath.replace("classpath:", "").trim();
			if (dubboPath.startsWith("/")) {
				dubboPath = dubboPath.substring(1);
			}
		}else{
			dubboPath = dubboPath.replace("file:", "").trim();
			if (!dubboPath.startsWith("/")) {
				//dubbo jar包里面ConfigUtil.class类中对路径有判断，当为/开头时，为文件路径，否者就是classpath路径 
				//window 获取到的路径没有/，linux服务器获得路径前自动有/，故为window路径补个/
				dubboPath = "/" + dubboPath; 
			}
		}
        System.setProperty(Constants.DUBBO_PROPERTIES_KEY, dubboPath);
	}
	@Override
	public void contextDestroyed(ServletContextEvent sce) {
		// TODO Auto-generated method stub
		
	}

}
```

**xml配置示例：**

在web.xml中添加以下内容
```xml
<!-- dubbo配置文件路径 -->
	<context-param>
	    <param-name>dubboConfiguration</param-name>
	    <param-value>${catalina.home}/config/dubbo.properties</param-value>
	</context-param>
	<listener>
	    <listener-class>com.tydic.config.DubboServletContextListener</listener-class>
	</listener>
```
param-value中配置dubbo.properties的路径即可，这里以classpath:开头的为配置的classpath路径，其他的为文件路径，${catalina.home}取的tomcat根目录

我这里的配置的是加载tomcat根目录下的config下的dubbo.properties