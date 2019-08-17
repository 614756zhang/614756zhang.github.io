---
layout: post
title: Redis安装部署
category: deploy
tags: Redis安装部署]
keywords: Redis安装部署
---
## 一、redis安装
1. 首先上官网下载Redis 压缩包，地址：http://download.redis.io/releases/
本文下载的是4.0.11版本
2. 上传到服务器上，或直接在Linux上下载wget http://download.redis.io/releases/redis-4.0.11.tar.gz
3. 执行解压操作 tar -zxvf redis-4.0.11.tar.gz
4. 移动到指定目录 mv redis-4.0.11 /home/
5. 执行make 对Redis解压后文件进行编译
```
cd  /home/redis-4.0.11
make
```
6. 编译成功后，进入src文件夹，执行make install进行Redis安装
```
cd src
make install
```

## 二、redis部署
1. 将配置文件和常用命令移动到统一文件中(不移也可以，直接执行)  
创建bin和conf文件
```
mkdir -p /home/redis/bin
mkdir -p /home/redis/conf
```
移动文件
```
mv /home/redis-4.0.11/redis.conf  /home/redis/conf
cd  /home/redis-4.0.11/src
mv mkreleasehdr.sh redis-benchmark redis-check-aof redis-check-rdb redis-cli redis-server /home/redis/bin
```
2. 执行Redis-server 命令，启动Redis 服务
```
cd /home/redis/bin
./redis-server
```
这种启动会随着会话关闭而关闭，故不可取
3. 后台启动redis服务
编辑conf文件，将daemonize属性改为yes（表明需要在后台运行）
```
cd /home/redis/conf
vi redis.conf
./redis-server /home/redis/conf/redis.conf
```
4. 后台客户端连接
```
/home/redis/bin/redis-cli
```
5. 停止redis实例
```
/home/redis/bin/redis-cli shutdown
或者
pkill redis-server
```
6. 让redis开机自启
编写启动脚本（暂不介绍）

## 三、Redis的配置
```
daemonize：如需要在后台运行，把该项的值改为yes
pdifile：把pid文件放在/var/run/redis.pid，可以配置到其他地址
bind：指定redis只接收来自该IP的请求，如果不设置，那么将处理所有请求，在生产环节中最好设置该项
port：监听端口，默认为6379
timeout：设置客户端连接时的超时时间，单位为秒
loglevel：等级分为4级，debug，revbose，notice和warning。生产环境下一般开启notice
logfile：配置log文件地址，默认使用标准输出，即打印在命令行终端的端口上
database：设置数据库的个数，默认使用的数据库是0
save：设置redis进行数据库镜像的频率
rdbcompression：在进行镜像备份时，是否进行压缩
dbfilename：镜像备份文件的文件名
dir：数据库镜像备份的文件放置的路径
slaveof：设置该数据库为其他数据库的从数据库
masterauth：当主数据库连接需要密码验证时，在这里设定
requirepass：设置客户端连接后进行任何其他指定前需要使用的密码
maxclients：限制同时连接的客户端数量
maxmemory：设置redis能够使用的最大内存
appendonly：开启appendonly模式后，redis会把每一次所接收到的写操作都追加到appendonly.aof文件中，当redis重新启动时，会从该文件恢复出之前的状态
appendfsync：设置appendonly.aof文件进行同步的频率
vm_enabled：是否开启虚拟内存支持
vm_swap_file：设置虚拟内存的交换文件的路径
vm_max_momery：设置开启虚拟内存后，redis将使用的最大物理内存的大小，默认为0
vm_page_size：设置虚拟内存页的大小
vm_pages：设置交换文件的总的page数量
vm_max_thrrads：设置vm IO同时使用的线程数量
```
