# 安全: 副本集群安全认证

## 环境介绍
已有一个副本集群并且正在运行中
## 正式开始
### 1、查找副本集 master 节点
```angular2
shard1:PRIMARY> rs.isMaster()  #如果你连接的节点是PRIMARY状态就代表已经是Master节点
{
	"hosts" : [   #成员节点
		"192.168.0.92:27017",
		"192.168.0.93:27017",
		"192.168.0.94:27017"
	],
	"setName" : "shard1",   #副本集名字
	"setVersion" : 3,
	"ismaster" : true,  #是不是master节点
	"secondary" : false,  #是不是副本节点
	"primary" : "192.168.0.93:27017", # master节点
	"me" : "192.168.0.93:27017",  
	"electionId" : ObjectId("7fffffff0000000000000005"),
	"maxBsonObjectSize" : 16777216,
	"maxMessageSizeBytes" : 48000000,
	"maxWriteBatchSize" : 1000,
	"localTime" : ISODate("2019-04-29T07:03:58.670Z"),
	"maxWireVersion" : 4,
	"minWireVersion" : 0,
	"ok" : 1
}
```
### 2、在 master 节点创建一个 root 用户
>注意：给一个相对权限比如：管理用户和角色的权限或者给一个最大权限。否则集群权限开启后可能自己会面临权限不足的尴尬
```angular2
shard1:PRIMARY> db.createUser(
... {
...   user: "root", 
...   pwd: "root", 
...   roles: [ { role: "root", db: "admin"}]})
```
### 3、生成密钥文件
```angular2
root@General:~# openssl rand -base64 756 > mongo.key
```
### 4、传输密钥文件并设置文件权限为 400
```angular2
root@General:~# ansible mongocluster -m copy -a "src=/root/mongo.key dest=mongodb-linux-x86_64-rhel70-3.2.22/key"
root@General:~# ansible mongocluster -m shell -a "chmod 400 /usr/local/mongodb-linux-x86_64-rhel70-3.2.22/key"
```
### 5、修改 mongodb 配置文件
```angular2
[root@node92 mongodb-linux-x86_64-rhel70-3.2.22]# vi config/shard.conf 
#各个节点配置文件增加以下参数
...
security:
   authorization: enabled
   keyFile:  /usr/local/mongodb-linux-x86_64-rhel70-3.2.22/key/mongo.key
...

```
### 6、重启 Mongo 实例  验证权限
```angular2
[root@node92 mongodb-linux-x86_64-rhel70-3.2.22]# bin/mongo 192.168.0.93:27017
MongoDB shell version: 3.2.22
connecting to: 192.168.0.93:27017/test
shard1:PRIMARY> show dbs;
2019-04-29T03:14:58.311-0400 E QUERY    [thread1] Error: listDatabases failed:{
	"ok" : 0,
	"errmsg" : "not authorized on admin to execute command { listDatabases: 1.0 }",
	"code" : 13
} :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
Mongo.prototype.getDBs@src/mongo/shell/mongo.js:62:1
shellHelper.show@src/mongo/shell/utils.js:781:19
shellHelper@src/mongo/shell/utils.js:671:15
@(shellhelp2):1:1
#进行权限验证
shard1:PRIMARY> use admin
switched to db admin
shard1:PRIMARY> db.auth("root","root")
1
shard1:PRIMARY> show dbs;
admin  0.000GB
local  0.000GB
test   0.000GB
```