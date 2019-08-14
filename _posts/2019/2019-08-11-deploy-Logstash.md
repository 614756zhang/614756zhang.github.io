---
layout: post
title: Logstash安装部署
category: deploy
tags: [ELK安装部署]
keywords: ELK,Kibana,deploy,Logstash安装部署,ELK安装部署
---
## Logstash安装步骤
##### （1）、直接下载或上传安装包
##### （2）、解压安装包到指定目录(/home/elk)
```
tar -zxvf logstash-6.3.2.tar.gz -C /home/elk/
cd  /home/elk/logstash-6.3.2
```
##### （3）、创建自己的配置文件
```
vi config/first-pipeline.conf
# 日志导入
input {
  beats {
    #指定监听端口
    port => 5044
    #要监听的ip地址，默认0.0.0.0；
    host => '0.0.0.0'
  }
}
# 日志筛选匹配处理
filter {
}
# 日志匹配输出
output {
     #可以同时输出到多个终端
     #筛选过滤后的内容输出到终端显示
     stdout { codec => "rubydebug" }
    
     #导出到elasticsearch
     elasticsearch {
        # 导出格式为json
        codec => "json"
        # ES地址+端口
        hosts => ["192.168.50.6:9200"]
        # 设置索引，可以使用时间变量
        index => "logstash-slow-%{+YYYY.MM.dd}"
        # ES如果有安全认证就使用账号密码验证，无安全认证就不需要
        #user => "admin"
        #password => "xxxxxx"
     }   
}
```
测试配置
```
./bin/logstash -f /home/elk/logstash-6.3.2/config/first-pipeline.conf --config.test_and_exit 
--config.test_and_exit 
```
出现Configuration OK 字样即为可以

##### （4）、启动命令
```
cd  /home/elk/logstash-6.3.2
./bin/logstash -f /home/elk/logstash-6.3.2/config/first-pipeline.conf --config.reload.automatic


nohup /home/elk/logstash-6.3.2/bin/logstash -f /home/elk/logstash-6.3.2/config/first-pipeline.conf --config.reload.automatic &
```
--config.reload.automatic选项启用动态重载配置功能


##### （5）、问题解决：
- 问题1：Java HotSpot(TM) 64-Bit Server VM warning: INFO: os::commit_memory(0x00000000c5330000, 986513408, 0) failed; error='Cannot allocate memory' (errno=12)  
原因：由于默认分配jvm空间大小较大，实际内存不够  
解决方案：
1、增加服务器内存（土豪专用）
2、修改默认jvm空间分配（实用）
cd  /home/elk/logstash-6.3.2
vi config/jvm.options 
-Xms1g
-Xmx1g
改为
-Xms512m
-Xmx512m

