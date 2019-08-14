---
layout: post
title: Elasticsearch安装部署
category: deploy
tags: [ELK安装部署]
keywords: ELK,Elasticsearch,deploy,Elasticsearch安装部署,ELK安装部署
---
## Elasticsearch安装步骤

##### （1）、直接下载或上传安装包  
##### （2）、解压安装包到指定目录(/home/elk)  
```
tar -zxvf elasticsearch-6.3.2.tar.gz -C /home/elk/
cd  /home/elk/elasticsearch-6.3.2
```
##### （3）、修改config/elasticsearch.yml配置  
**集群配置修改：**
```
vi config/elasticsearch.yml
```
主节点配置：
```
cluster.name: cluster-node  # 集群中的名称
node.name: master-node  # 该节点名称
node.master: true  # 意思是该节点为主节点
node.data: false  # 表示这不是数据节点
network.host: 0.0.0.0  # 监听全部ip，在实际环境中应设置为一个安全的ip
http.port: 9200  # es服务的端口号
discovery.zen.ping.unicast.hosts: ["192.168.50.6", "192.168.50.8"] # 配置自动发现
#Memory
bootstrap.memory_lock: false 
bootstrap.system_call_filter: false
```
数据节点配置：
```
cluster.name: cluster-node  # 集群中的名称
node.name: data-node-1  # 该节点名称
node.master: false  # 
node.data: true  # 表示该节点为数据节点
network.host: 0.0.0.0  # 监听全部ip，在实际环境中应设置为一个安全的ip
http.port: 9200  # es服务的端口号
discovery.zen.ping.unicast.hosts: ["192.168.50.6", "192.168.50.8"] # 配置自动发现
#Memory
bootstrap.memory_lock: false 
bootstrap.system_call_filter: false
```

##### （4）、把下面**问题解决的设置**提前设置了，不然报错了还要解决  
##### （5）、启动elasticsearch
root用户无法启动elasticsearch，需要使用非root用户,
这里我需要新建用户，非root用户忽略创建用户
```
#创建用户组
groupadd elkgroup
#创建用户及文件授权
useradd -d /home/elk elkuser -g elkgroup

#切换用户
su elkuser
启动elasticsearch
/home/elk/elasticsearch-6.3.2/bin/elasticsearch -d
(加-d，则表示后端运行)

关闭elasticsearch
jps 
kill -9 进程号
 ```
##### （6）、验证

- 在浏览器中输入http://192.168.50.6:9200/ 可看到单节点信息即为启动成功
- 在浏览器中输入主节点的http://192.168.50.6:9200/_cluster/health?pretty，
- 可集群的健康检查及查看集群信息
http://192.168.50.6:9200/_cluster/state?pretty  可查询集群及其节点的详细内容

以上用curl 命令也可以，
```
curl 'http://192.168.50.6:9200/_cluster/health?pretty'
bash-4.1$ curl 'http://192.168.50.6:9200/_cluster/health?pretty'
{
  "cluster_name" : "my-application",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 2,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 0,
  "active_shards" : 0,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

##### （7）、问题解决：
- 问题1：java.lang.UnsupportedOperationException: seccomp unavailable:requires kernel 3.5+ with CONFIG_SECCOMP and CONFIG_SECCOMP_FILTER compiled in
只是警告，主要是因为Linux版本过低造成的，警告不影响使用，可以忽略

- 问题2：node validation exception
[4] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]  
原因：无法创建本地文件问题,用户最大可创建文件数太小，修改配置  
解决方案：切换到root用户，编辑limits.conf配置文件， 添加类似如下内容：
```
vi /etc/security/limits.conf
添加如下内容:
#elasticsearch添加配置
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
```
- 问题3：
max number of threads [1024] for user [es] likely too low, increase to at least [4096]  
原因：无法创建本地线程问题,用户最大可创建线程数太小  
解决方案：切换到root用户，进入limits.d目录下，修改90-nproc.conf 配置文件。
```
vi /etc/security/limits.d/90-nproc.conf
找到如下内容：
* soft nproc 1024
#修改为
* soft nproc 4096
```

- 问题4：
max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]  
原因：最大虚拟内存太小  
解决方案：切换到root用户下，修改配置文件sysctl.conf
```
vi /etc/sysctl.conf
添加下面配置：
#elasticsearch添加配置
vm.max_map_count=655360
```
并执行命令：
sysctl -p
然后重新启动elasticsearch，即可启动成功。

- 问题5：
 system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk  
原因： Centos6不支持SecComp，而ES5.2.0默认bootstrap.system_call_filter为true  
解决方案：在elasticsearch.yml中配置bootstrap.system_call_filter为false，
注意要在Memory下面: 
bootstrap.memory_lock: false 
bootstrap.system_call_filter: false

- 问题6：
Exception in thread "main" java.nio.file.AccessDeniedException: 
/home/elk/elasticsearch-6.3.2/config/jvm.options  
原因：使用非 root用户启动ES，该用户的文件权限不足  
解决方案：添加用户对该文件的权限  chown -R elkuser:elkgroup  /home/elk

- 问题7：Java HotSpot(TM) 64-Bit Server VM warning: INFO: os::commit_memory(0x00000000c5330000, 986513408, 0) failed; error='Cannot allocate memory' (errno=12)  
原因：由于elasticsearch默认分配jvm空间大小为2g，实际内存不够  
解决方案：
1、增加服务器内存（土豪专用）
2、修改elasticsearch默认jvm空间分配（实用）
cd  /home/elk/elasticsearch-6.3.2
vi config/jvm.options 
-Xms2g
-Xmx2g
改为
-Xms512m
-Xmx512m