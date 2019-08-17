---
layout: post
title: ELK安装部署（ Elasticsearch + Logstash + Filebeat + Kibana）
category: deploy
tags: [ELK安装部署]
keywords: ELK,Elasticsearch,Logstash,Filebeat,Kibana,deploy,安装部署
---
## 前言：
**架构图：**

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019/2019-08-11-deploy-ELK.jpg)

## 一、环境准备
### 三台虚拟机
    192.168.50.6  
    192.168.50.8  
    192.168.50.9

### 机器部署说明
    192.168.50.6、192.168.50.8安装elasticsearch 192.168.50.6为主节点，192.168.50.8为数据节点
    192.168.50.6安装filebeat
    192.168.50.8安装logstash
    192.168.50.9安装kibana

### ELK组件版本
    filebeat：6.3.2
    logstash：6.3.2
    elasticsearch：6.3.2
    kibana：6.3.2
    jdk1.8
    tomcat8
### 组件下载：
下载官网地址：https://www.elastic.co/cn/downloads/  
历史版本下载：https://www.elastic.co/cn/downloads/past-releases  
官网下载6.3.2版本的组件压缩包

## 二、组件部署安装
### 1、Elasticsearch安装部署
<div>
<iframe marginwidth=0 marginheight=0 width="100%" height=200 src="deploy-Elasticsearch.html" frameborder="no"></iframe>
</div>

### 2、Kibana安装部署
 <a href="deploy-Kibana.html">详见Kibana安装部署安装部署</a>
 <div id="Kibana"></div>
### 3、Logstash安装部署
 <a href="deploy-Logstash.html">详见Logstash安装部署</a>
### 4、Filebeat安装部署
 <a href="deploy-Filebeat.html">详见Filebeat安装部署</a>

## 三、组件连接
### 1、各组件说明：
- Elasticsearch是搜索引擎，需要做集群处理，也是需要做优化和性能升级的关键组件；
- Kibana是前台展示，目的是将从Elasticsearch中获取到的检索数据可视化展示和操作；
- Logstash 可做日志收集和过滤，但由于其相对的重量级（安装包大，运行占内存高等），现有大多只用它过滤日志，而非采集；
- Filebeat是日志采集，监控到指定日志文件添加后采集到其配置对应的输出方，轻量级，对被采集的日志和应用无影响，无感知；

### 2、各组件配置说明：
- Elasticsearch ，配置分配好集群的mater和node节点分配、node节点的主备、数据分片的规则（有需要的话）即可；
- Kibana 配置好自己的ip和端口以及Elasticsearch集群地址就行了；
- Logstash  配置好日志输入（输入方式、监听端口）、日志过滤规则、日志输出（Elasticsearch集群地址）
- Filebeat 配置采集日志文件的地址，采集到日志的输出方（Logstash、中间件、Elasticsearch）

### 3、启动顺序：
Elasticsearch --> Kibana --> Logstash --> Filebeat
