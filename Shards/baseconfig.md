# 集群: 分片副本集群基础配置

## 环境介绍

|系统版本|服务器ip地址|分片集群角色|端口号|
|-------|-------|-------|-------|
|CentOS Linux release 7.6.1810 (Core)|192.168.0.92|Master|30000|
|CentOS Linux release 7.6.1810 (Core)|192.168.0.92|config|20000|
|CentOS Linux release 7.6.1810 (Core)|192.168.0.92|shard1|27017|
|CentOS Linux release 7.6.1810 (Core)|192.168.0.92|shard2|27018|
|CentOS Linux release 7.6.1810 (Core)|192.168.0.92|shard3|27019|
|CentOS Linux release 7.6.1810 (Core)|192.168.0.93|config|20000|
|CentOS Linux release 7.6.1810 (Core)|192.168.0.93|shard1|27017|
|CentOS Linux release 7.6.1810 (Core)|192.168.0.93|shard2|27018|
|CentOS Linux release 7.6.1810 (Core)|192.168.0.93|shard3|27019|
|CentOS Linux release 7.6.1810 (Core)|192.168.0.94|config|20000|
|CentOS Linux release 7.6.1810 (Core)|192.168.0.94|shard1|27017|
|CentOS Linux release 7.6.1810 (Core)|192.168.0.94|shard2|27018|
|CentOS Linux release 7.6.1810 (Core)|192.168.0.94|shard3|27019|

首先根据 [副本: 副本集集群基础配置](../Replication/baseconfig.md) 创建出三个副本集集群,副本集名称分别为shard1、shard2、shard3。

## 正式开始

### 1、配置 mongos 分片集路由节点
```angular2
[root@node92 mongodb-linux-x86_64-rhel70-3.2.22]# cat config/mongos.conf 
systemLog:
   destination: file
   path: "/usr/local/mongodb-linux-x86_64-rhel70-3.2.22/logs/mongos.log"
   logAppend: true
processManagement:
   fork: true
net:
   bindIp: 192.168.0.92
   port: 30000
sharding:
  configDB: 192.168.0.92:20000,192.168.0.93:20000,192.168.0.94:20000
  chunkSize: 64
```

### 2、配置 configsvr 分片集配置节点

>注意：从3.4开始，mongod 不再支持将已弃用的镜像实例用作配置服务器（SCCC）。在将分片群集升级到3.4之前，必须将配置服务器从SCCC转换为CSRS。可以将配置集节点作为副本集群实现数据的一致性。

```angular2
[root@node92 mongodb-linux-x86_64-rhel70-3.2.22]# cat config/config.conf 
systemLog:
   destination: file
   path: "/usr/local/mongodb-linux-x86_64-rhel70-3.2.22/logs/config.log"
   logAppend: true
storage:
   dbPath: "/usr/local/mongodb-linux-x86_64-rhel70-3.2.22/data"
   directoryPerDB: true
   journal:
      enabled: true
processManagement:
   fork: true
net:
   bindIp: 192.168.0.92
   port: 20000
sharding:
   clusterRole: configsvr

```

### 3、在各个副本集节点添加分片集群角色
以 node92 副本集 shard1 为例，其他副本集节点依次添加。
```angular2
[root@node92 mongodb-linux-x86_64-rhel70-3.2.22]# cat config/shard1.conf 
...
sharding:
  clusterRole: shardsvr
...
```

### 4、重启各副本集节点 启动mognos和configsvr
```angular2
#举例启动 依次启动各 Mongo Cluster Node。
[root@node92 mongodb-linux-x86_64-rhel70-3.2.22]# bin/mongos -f config/mongos.conf
[root@node92 mongodb-linux-x86_64-rhel70-3.2.22]# bin/mongod -f config/config.conf
```
### 5、在 mongos 节点添加副本集节点为分片节点
```angular2
[root@node92 mongodb-linux-x86_64-rhel70-3.2.22]# bin/mongo 192.168.0.92:30000
MongoDB shell version: 3.2.22
connecting to: 192.168.0.92:30000/test
mongos> sh.addShard("shard1/192.168.0.92:27017")
mongos> sh.addShard("shard2/192.168.0.92:27018")
mongos> sh.addShard("shard3/192.168.0.92:27019")
```

### 6、搭建完成测试即可
