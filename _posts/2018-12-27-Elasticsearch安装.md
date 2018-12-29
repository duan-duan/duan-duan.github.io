---
title: elasticsearch安装
date: 2018-12-27 14:29:53
categories:
- elasticsearch
- elk
tags:
- elasticsearch
- elk
---


1.查看系统环境：

```shell
[root@Elasticserch ~]# hostname 
Elasticserch
[root@Elasticserch ~]# cat /etc/redhat-release 
CentOS release 6.5 (Final)
[root@Elasticserch ~]# uname -r
2.6.32-431.el6.x86_64
```

3.安装Elasticserch节点：
```shell
[root@Elasticserch ~]# tar xf elasticsearch-6.3.1.tar.gz 
[root@Elasticserch ~]# mv elasticsearch-6.3.1 /usr/local/elasticsearch
[root@Elasticserch ~]# cd /usr/local/elasticsearch/
[root@Elasticserch elasticsearch]# cd config/
[root@Elasticserch config]# cp elasticsearch.yml{,.default}     #备份配置文件，防止修改错误
```
​    4.修改配置文件：
```shell
[root@Elasticserch config]# vim elasticsearch.yml
cluster.name: my-es-cluster           #集群的名称
node.name: node-1                 #节点的名称
path.data: /usr/local/elasticsearch/data   #数据路径
path.logs: /usr/local/ elasticsearch /logs    #日志路径
bootstrap.memory_lock: false          #这行去掉注释把ture改成false，不改会造成服务启动报错
bootstrap.system_call_filter: false      #添加这行，否则启动会报错。
配置上述两行的原因：
这是在因为Centos6不支持SecComp，而ES5.2.0默认bootstrap.system_call_filter为true进行检测，所以导致检测失败，失败后直接导致ES不能启动。
network.host: 192.168.8.102          # elasticsearch主机地址
http.port: 9200                        #端口号
discovery.zen.ping.unicast.hosts: ["node-1"]  #启动新节点通过主机列表发现。
discovery.zen.minimum_master_nodes: 1    #总节点数


[root@elasticsearch ~]# vim /etc/security/limits.d/90-nproc.conf
*          soft    nproc     4096   #默认1024，改成4096


[root@elasticsearch ~]# vim /etc/sysctl.conf
#末尾追加否则服务会报错。
vm.max_map_count=655360

[root@elasticsearch ~]# sysctl -p   #使上述配置生效
```
​      5.创建elasticsearch运行的用户：
```shell
[root@Elasticserch config]# useradd elasticsearch 
[root@Elasticserch config]# chown -R elasticsearch.elasticsearch /usr/local/elasticsearch/
```
​    6.修改文件句柄数：
```shell
[root@Elasticserch config]# vim /etc/security/limits.conf
#添加下面内容：
* soft nofile 65536

* hard nofile 131072

* soft nproc 2048

* hard nproc 4096
```

 7.切换用户启动服务。 nohup  bin/elasticsearch -d &

```shell
[root@Elasticserch config]# su - elasticsearch
[elasticsearch@Elasticserch ~]$ cd /usr/local/elasticsearch/
[elasticsearch@elasticsearch elasticsearch]$ bin/elasticsearch &
注：如果启动错误请看下上述配置过程黄色标记的部分是否有配置错误或者没有配置。
```
​     8.查看服务是否启动成功
```shell
[root@elasticsearch ~]# netstat -anpt
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
tcp        0      0 0.0.0.0:111                 0.0.0.0:*                   LISTEN      1018/rpcbind        
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      1152/sshd           
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      1228/master         
tcp        0      0 0.0.0.0:3616                0.0.0.0:*                   LISTEN      1036/rpc.statd      
tcp        0      0 192.168.8.102:22            192.168.8.254:64255         ESTABLISHED 1727/sshd           
tcp        0     64 192.168.8.102:22            192.168.8.254:64756         ESTABLISHED 2617/sshd           
tcp        0      0 :::34155                    :::*                        LISTEN      1036/rpc.statd      
tcp        0      0 :::111                      :::*                        LISTEN      1018/rpcbind        
tcp        0      0 :::9200                     :::*                        LISTEN      2180/java           
tcp        0      0 :::9300                     :::*                        LISTEN      2180/java           
tcp        0      0 :::22                       :::*                        LISTEN      1152/sshd           
tcp        0      0 ::1:25                      :::*                        LISTEN      1228/master
```
​    9.简单测试
```shell
[root@elasticsearch ~]# curl 10.75.229.83:9200
{
  "name" : "RN2SQVB",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "TSeoUl-vROuJGR7vfuSuBw",
  "version" : {
    "number" : "6.5.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "8c58350",
    "build_date" : "2018-11-16T02:22:42.182257Z",
    "build_snapshot" : false,
    "lucene_version" : "7.5.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```
出现如上结果表示ES配置完成，并且没有问题