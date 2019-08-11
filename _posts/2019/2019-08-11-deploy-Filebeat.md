---
layout: post
title: Filebeat安装部署
category: deploy
tags: [ELK安装部署]
keywords: ELK,Kibana,deploy,Filebeat安装部署,ELK安装部署
---
##### （1）、直接下载或上传安装包
##### （2）、解压安装包到指定目录(/home/elk)
```
tar zxvf filebeat-6.3.2-linux-x86_64.tar.gz -C /home/elk/
cd  /home/elk/filebeat-6.3.2-linux-x86_64
```
##### （3）、配置filebeat .yml
主要配置采集那些日志文件（文件路径），输出到哪里（elsticsearch、logstash、kibana）这里我们关闭output.elasticsearch，打开output.logstash，将收集到的信息推送到logstash。
filebeat.yml常用配置：
1）定义日志文件路径
```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    – /var/log/*.log
```
2）输出给elsticsearch（不输出给logstash）（建议输出给logstash）
```
output.elasticsearch:
  hosts: ["192.168.1.42:9200"]
```
3）如果打算用kibana来展示filebeat数据，需要配置为kibana终端
```
setup.kibana:
  host: “localhost:5601″
```
4）如果elasticsearch和kibana增加了安全配置，你需要在配置文件中指定认证信息，
举例如下：
```
output.elasticsearch:
  hosts: ["myEShost:9200"]
  username: “filebeat_internal”
  password: “{pwd}” 
setup.kibana:
  host: “mykibanahost:5601″
  username: “my_kibana_user” 
    password: “{pwd}”
```
5）配置输出给logstash
```
output.logstash:
  hosts: ["192.168.50.8:5044"]
```
##### （4）、启动命令
```
cd  /home/elk/filebeat-6.3.2-linux-x86_64
./filebeat -e -c ./filebeat.yml
```