---
layout: post
title: Kibana安装部署
category: deploy
tags: [ELK安装部署]
keywords: ELK,Kibana,deploy,Kibana安装部署,ELK安装部署
---
##### （1）、直接下载或上传安装包
##### （2）、解压安装包到指定目录(/home/elk)
```
tar zxvf tar -zxvf kibana-6.3.2-linux-x86_64.tar.gz -C /home/elk/
cd  /home/elk/kibana-6.3.2-linux-x86_64
```
##### （3）、修改配置文件：kibana.yml
vi config/kibana.yml  
主要改两项
```
#配置本机ip
server.host: "192.168.50.9"
#配置es集群url
elasticsearch.url: "http://192.168.50.6:9200"
```
##### （4）、启动命令
```
cd  /home/elk/kibana-6.3.2-linux-x86_64
./bin/kibana
```
##### （5）、验证
ps：注意开启端口5601
访问：192.168.50.9:5601
能打开就是成功了