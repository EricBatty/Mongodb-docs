# 复制: 复制集集群基础配置

## 环境介绍

|系统版本|服务器ip地址|MongoDB版本|端口号|
|-------|-------|-------|-------|
|CentOS Linux release 7.6.1810 (Core)|192.168.0.92|mongodb-linux-x86_64-rhel62-3.6.5.tgz|27017|
|CentOS Linux release 7.6.1810 (Core)|192.168.0.93|mongodb-linux-x86_64-rhel62-3.6.5.tgz|27017|
|CentOS Linux release 7.6.1810 (Core)|192.168.0.94|mongodb-linux-x86_64-rhel62-3.6.5.tgz|27017|

## 正式开始
本实践其中有使用ansible工具便于快速部署。
### 1、安装MongoDB
```angular2
$ wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-3.2.22.tgz
$ ansible -i inventory mongocluster -m copy -a 'src=/Users/jinxiaozhang/mongodb-linux-x86_64-rhel62-3.6.5.tgz dest=/usr/local/'
$ ansible -i inventory mongocluster -m shell -a 'tar zxvf /usr/local/mongodb-linux-x86_64-rhel62-3.6.5.tgz -C /usr/local/
```
### 2、创建我们所需要的目录
```angular2
$ ansible -i inventory mongocluster -m shell -a 'mkdir /usr/local/mongodb-linux-x86_64-rhel70-3.2.22/data' 
$ ansible -i inventory mongocluster -m shell -a 'mkdir /usr/local/mongodb-linux-x86_64-rhel70-3.2.22/shard1' 
$ ansible -i inventory mongocluster -m shell -a 'mkdir /usr/local/mongodb-linux-x86_64-rhel70-3.2.22/logs' 
$ ansible -i inventory mongocluster -m shell -a 'mkdir /usr/local/mongodb-linux-x86_64-rhel70-3.2.22/config' 
$ ansible -i inventory mongocluster -m shell -a 'mkdir /usr/local/mongodb-linux-x86_64-rhel70-3.2.22/key' 
```
### 3、创建配置文件
```angular2

[root@node92 mongodb-linux-x86_64-rhel70-3.2.22]# cat shard1.conf
systemLog:
  destination: file
  path: "/usr/local/mongodb-linux-x86_64-rhel70-3.2.22/logs/shard1.log"
  logAppend: true
storage:
  dbPath:   /usr/local/mongodb-linux-x86_64-rhel70-3.2.22/data/shard1
  engine: wiredTiger
#  oplogSize: 51200
processManagement:
  fork: true
net:
  bindIp: 192.168.0.92 #其他两台机器分别改为192.168.0.93 ; 192.168.0.94
  port: 27017
replication:
  replSetName:  shard1
```
### 4、启动服务
```angular2
[root@node92 mongodb-linux-x86_64-rhel70-3.2.22]# bin/mongod -f config/shard1.conf
[root@node93 mongodb-linux-x86_64-rhel70-3.2.22]# bin/mongod -f config/shard1.conf
[root@node94 mongodb-linux-x86_64-rhel70-3.2.22]# bin/mongod -f config/shard1.conf
```
### 5、初始化副本集群
```angular2
[root@node92 mongodb-linux-x86_64-rhel70-3.2.22]# bin/mongo 192.168.0.92:27017
> rs.initiate() #初始化一个副本集群
```
### 6、向副本集群添加节点成员
```angular2
[root@node92 mongodb-linux-x86_64-rhel70-3.2.22]# bin/mongo 192.168.0.92:27017
> rs.add("192.168.0.93:27017")
> rs.add("192.168.0.94:27017")
```
### 7、查看副本集群状态
```angular2
shard1:PRIMARY> rs.status()
{
	"set" : "shard1",
	"date" : ISODate("2019-04-29T06:48:45.798Z"),
	"myState" : 1,
	"term" : NumberLong(5),
	"heartbeatIntervalMillis" : NumberLong(2000),
	"members" : [
		{
			"_id" : 0,
			"name" : "192.168.0.92:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 10961,
			"optime" : {
				"ts" : Timestamp(1556509844, 1),
				"t" : NumberLong(5)
			},
			"optimeDate" : ISODate("2019-04-29T03:50:44Z"),
			"lastHeartbeat" : ISODate("2019-04-29T06:48:44.593Z"),
			"lastHeartbeatRecv" : ISODate("2019-04-29T06:48:45.714Z"),
			"pingMs" : NumberLong(1),
			"syncingTo" : "192.168.0.94:27017",
			"configVersion" : 3
		},
		{
			"_id" : 1,
			"name" : "192.168.0.93:27017",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 10988,
			"optime" : {
				"ts" : Timestamp(1556509844, 1),
				"t" : NumberLong(5)
			},
			"optimeDate" : ISODate("2019-04-29T03:50:44Z"),
			"electionTime" : Timestamp(1556509548, 1),
			"electionDate" : ISODate("2019-04-29T03:45:48Z"),
			"configVersion" : 3,
			"self" : true
		},
		{
			"_id" : 2,
			"name" : "192.168.0.94:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 10988,
			"optime" : {
				"ts" : Timestamp(1556509844, 1),
				"t" : NumberLong(5)
			},
			"optimeDate" : ISODate("2019-04-29T03:50:44Z"),
			"lastHeartbeat" : ISODate("2019-04-29T06:48:45.700Z"),
			"lastHeartbeatRecv" : ISODate("2019-04-29T06:48:44.725Z"),
			"pingMs" : NumberLong(0),
			"syncingTo" : "192.168.0.93:27017",
			"configVersion" : 3
		}
	],
	"ok" : 1
}

```