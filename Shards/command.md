# 分片: 有关分片集群设置的一些命令

## sh.addShard()
### 定义

sh.addShard(\<url\>)  #必须在mongos实例上运行此方法。

将分片副本集添加到分片群集。

|参数|类型|描述|
|-------|-------|-------|
|host|string|碎片副本集的至少一个成员的副本集名称，主机名和端口。任何其他副本集成员主机名必须以逗号分隔。例如：<replica_set> / <hostname> <：port>，<hostname> <：port>，...|

### 使用格式
`sh.saddShard （“<replica_set> / <hostname> <：port>” ）`

### 注意事项
#### 平衡
将分片添加到分片群集时，会影响所有现有分片集合的群集分片之间的块的平衡 。平衡器将开始迁移块，以便群集实现平衡。
版本2.6中已更改：块迁移可能会对磁盘空间产生影响。从MongoDB 2.6开始，源分片默认自动归档迁移的文档。
#### 隐藏成员

> 您不能在提供的种子列表中 包含隐藏的成员sh.addShard()。


## sh.addShardTag()
### 定义
sh.addShardTag(shard, tag)  #仅在连接到mongos实例时使用

在3.4版中更改：此方法sh.addShardToZone()在MongoDB 3.4中别名。下面指定的功能仍适用于MongoDB 3.2。MongoDB 3.4提供区域分片作为标记感知分片的后继。

将分片与标签或标识符关联。MongoDB使用这些标识符将位于标记范围内的块指向特定分片。sh.addTagRange() 将块范围与标记范围相关联。

|参数|类型|描述|
|-------|-------|-------|
|shard|string|要为其指定特定标记的分片的名称。|
|tag|string|要添加到分片的标记的名称。

### 使用格式
```angular2
sh.addShardTag （“shard0000” ， “NYC” ）
sh.addShardTag （“shard0001” ， “LAX” ）
sh.addShardTag （“shard0002” ， “NRT” ）
```

## sh.addShardToZone()









