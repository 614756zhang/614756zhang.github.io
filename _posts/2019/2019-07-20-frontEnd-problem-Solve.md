---
layout: post
title: 解决IE浏览器处理返回JSON数据提示下载问题
category: frontEnd
tags: [IE浏览器兼容]
keywords: JSON,IE浏览器,提示下载
---


### 一、问题

Ajax请求后台，后台返回json数据，在IE浏览器弹出XXX.json下载提示，不能正确接收Json数据，谷歌浏览器无此问题

### 二、原因分析

此时response中header的ContentType为application/json，该类型只有新浏览器才会兼容，IE未兼容该类型，故把返回数据当成一个文件，提示下载

### 三、问题解决

1. 修改或强制指定后台response中header的ContentType为text/html
```java
response.setContentType("text/html;charset=UTF-8");
```

2. 前台修改请求参数中的dataType为html
```java
var options  = {
			url:url,
			dataType:'html',
			//dataType:'json',
			type:'post',
	        success:function(response){
				var json = eval("(" + response + ")");
			}
		}
```