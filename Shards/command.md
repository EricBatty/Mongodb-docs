# 分片: 有关分片集群设置的一些命令

## sh.addShard()
### 定义

sh.addShard(\<url\>)  

>必须在mongos实例上运行此方法。

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
sh.addShardTag(shard, tag)  

>仅在连接到mongos实例时使用

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
### 定义
sh.addShardToZone(shard, zone)

>在连接到mongos实例时使用。

版本3.4中的新功能：将分片与区域关联。MongoDB将此分片与给定区域相关联。区域覆盖的块被分配给与区域关联的分片。

|参数|类型|描述|
|-------|-------|-------|
|shard|string|要关联到zone的分片的名称|
|zone|string|要关联到shard的区域的名称|

### 行为
您可以将区域与多个分片关联，分片可以与多个分区关联。

### 范围
MongoDB有效地忽略了没有至少一个与之关联的分片键值范围的区域。

要将一系列分片键值与区域相关联，请使用该 sh.updateZoneKeyRange()方法。

### 安全
对于使用身份验证运行的分片群集，您必须作为其权限包含update在config.shards集合或config数据库上的用户进行身份验证。

clusterAdmin或clusterManager内置角色有发放相应的权限sh.addShardToZone()。有关详细信息，请参阅 基于角色的访问控制手册页。

### 示例
下面的示例将三个区域，NYC，LAX，和NRT，关联到每一个碎片:
```angular2
sh.addShardTag （“shard0000” ， “NYC” ）
sh.addShardTag （“shard0001” ， “LAX” ）
sh.addShardTag （“shard0002” ， “NRT” ）
```
分片可以与多个区域关联。以下示例关联 LGA到shard0000：
```angular2
sh.addShardToZone （“shard0000” ， “LGA” ）
```
shard0000与LGA区域和JFK区域相关联。在平衡群集中，MongoDB将任一区域覆盖的读取和写入路由到 shard0000。

## sh.addTagRange()
### 定义

>在连接到mongos实例时使用

sh.addTagRange(namespace, minimum, maximum, tag)

在3.4版中更改：此方法sh.updateZoneKeyRange()在MongoDB 3.4中别名。下面指定的功能仍适用于MongoDB 3.2。MongoDB 3.4提供区域分片作为标记感知分片的后继。

|参数|类型|描述|
|-------|-------|-------|
|namespace|string|要标记的分片集合的命名空间。|
|minimum|document|要包含在标记中的分片键范围的最小值。最小值是包含性匹配。以形式指定最小值<fieldname>:<value>。该值必须与分片键具有相同的BSON类型。|
|maximum|document|要包含在标记中的分片键范围的最大值。最大值是独家匹配。以形式指定最大值<fieldname>:<value>。该值必须与分片键具有相同的BSON类型。|
|tag|string|用于附加由minimum 和maximum参数指定的范围的标记的名称。|

使用sh.addShardTag()以确保均衡迁移存在指定范围内的一个特定的碎片或一组碎片的文件。

### 行为

#### 界限
区域范围始终包括下边界并且不包括上边界。
#### 删除的集合
如果使用标签范围添加标签范围 sh.addTagRange()，然后删除集合或其数据库，则MongoDB不会删除标记关联。如果稍后创建具有相同名称的新集合，则旧标记关联将应用于新集合。
### 示例
给定分片键，以下操作在纽约州创建一个涵盖邮政编码的标签范围：{state: 1, zip: 1}

```angular2
sh.addTagRange( "exampledb.collection",
                { state: "NY", zip: MinKey },
                { state: "NY", zip: MaxKey },
                "NY"
              )
```

## sh.disableBalancing()
### 说明

>在mongos实例上 运行

sh.disableBalancing(namespace)

禁用指定分片集合的平衡器。这不会影响同一群集中其他分片集合的块的平衡 。

|参数|类型|描述|
|-------|-------|-------|
|namespace|string|集合的名称空间。|

## sh.enableBalancing()

### 说明

>在mongos实例上 运行。

sh.enableBalancing(namespace)

为分片集合的指定命名空间启用平衡器。

|参数|类型|描述|
|-------|-------|-------|
|namespace|string|集合的名称空间|

>sh.enableBalancing()没有开始 平衡。相反，它允许在下次平衡器运行时平衡此集合。


## sh.disableAutoSplit()


版本3.4中的新功能。

禁用config.settings 集合中的autosplit标志。当为分片群集启用自动分割时，MongoDB会根据分块所代表的分片键值自动分割块，以防止块变得过大。
默认情况下启用自动拆分。

## sh.enableAutoSplit()

>只能在mongos实例使用 

sh.enableAutoSplit()

版本3.4中的新功能。

为分片群集启用自动分割。 sh.enableAutoSplit()启用config.settings集合中的autosplit标志 。

默认情况下启用自动拆分。

## sh.enableSharding()
### 定义
sh.enableSharding(database)

在指定的数据库上启用分片。这不会自动对任何集合进行分片，但可以使用开始分片集合sh.shardCollection()。

|参数|类型|描述|
|-------|-------|-------|
|database|string|数据库分片的名称。将名称括在引号中。|

## sh.getBalancerHost()

从版本3.4开始不推荐使用：从3.4开始，平衡器在CSRS的主要版本上运行。CSRS的主要使用名为“ConfigServer”的进程ID来保持“平衡器”锁定。这个锁永远不会释放。

## sh.getBalancerState()

sh.getBalancerState()启用平衡器true时返回， 如果禁用平衡器则返回false。这并不反映平衡操作的当前状态：用于sh.isBalancerRunning()检查平衡器的当前状态。

## sh.removeTagRange()
### 定义

> 在连接到mongos实例时使用。

sh.removeTagRange(namespace, minimum, maximum, tag)

3.4版中更改：此方法sh.removeRangeFromZone()在MongoDB 3.4中别名。下面指定的功能仍适用于MongoDB 3.2。MongoDB 3.4提供区域分片作为标记感知分片的后继。

3.0版中的新功能。
删除一系列分片键值到使用该sh.addShardTag()方法创建的分片标记。

|参数|类型|描述|
|-------|-------|-------|
|namespace|string|要标记的分片集合的命名空间。|
|minimum|document|要包含在标记中的分片键范围的最小值。最小值是包含性匹配。以形式指定最小值<fieldname>:<value>。该值必须与分片键具有相同的BSON类型。|
|maximum|document|要包含在标记中的分片键范围的最大值。最大值是独家匹配。以形式指定最大值<fieldname>:<value>。该值必须与分片键具有相同的BSON类型。|
|tag|string|用于附加由minimum 和maximum参数指定的范围的标记的名称。|
使用sh.removeShardTag()以确保未使用的或过期的范围被删除，因此块是根据需要平衡。

### 示例
给定分片键，以下操作将删除覆盖纽约州邮政编码的现有标记范围：{state: 1, zip: 1}
```angular2
sh.removeTagRange( "exampledb.collection",
                { state: "NY", zip: MinKey },
                { state: "NY", zip: MaxKey },
                "NY"
              )
```

## sh.removeRangeFromZone()
### 定义
sh.removeRangeFromZone(namespace, minimum, maximum)

版本3.4中的新功能。

删除一系列分片键值与区域之间的关联 。

|参数|类型|描述|
|-------|-------|-------|
|namespace|string|要与区域关联的分片集合的命名空间，必须对集合进行分片才能使操作成功。。|
|minimum|document|分片键值范围的包含下限。以形式指定分片键的每个字段。该值必须与分片键具有相同的BSON类型。<fieldname> : <value>|
|maximum|document|分片键值范围的独占上限。以形式指定分片键的每个字段。该值必须与分片键具有相同的BSON类型。<fieldname> : <value>|

使用sh.removeRangeFromZone()删除未使用之间的关联，过时或相互冲突的范围和区域。

如果没有范围匹配传递给的最小和最大边界 removeShardFromZone()，则不会删除任何内容。

### 行为
sh.removeShardFromZone() 不会删除与指定范围关联的区域。

#### 平衡器
删除范围和区域之间的关联将删除约束，以保持该区域内的分片上的范围所覆盖的块。在下一轮平衡器期间，平衡器可以迁移先前由区域覆盖的块。

#### 安全
对于使用身份验证运行的分片群集，您必须以其权限包括以下内容的用户进行身份验证
- find在config.shards集合或config 数据库上
- find在config.tags集合或config 数据库上
- update在config.tags集合或config 数据库上
- remove在config.tags集合或config 数据库上

在clusterAdmin或clusterManager内置角色有发放相应的权限sh.removeRangeFromZone()

### 示例
给定exampledb.collection带有分片键的分片集合，以下操作将删除具有下限 和上限的范围：{ a : 1 }{ a : 10 }
```angular2
sh.removeRangeFromZone( "exampledb.collection",
                { a : 1 },
                { a : 10 }
              )

```
在min和max必须在目标范围内的完全范围一致。以下操作尝试删除先前创建的范围，但指定为绑定：{ a : 0 }{ a : 10 }
```angular2
admin = db.getSiblingDB("admin")
admin.runCommand(
   {
      updateZoneKeyRange : "exampledb.collection",
      min : { a : 0 },
      max : { a : 10 },
      zone : null
   }
)
```
而的范围和包括现有的范围，这是不完全匹配，并且因此 不会删除任何东西。{ a : 0 }{ a : 10 }sh.removeRangeFromZone()

#### 符合分片键

给定exampledb.collection带有分片键的分片 集合，以下操作将删除具有下限和上限的范围：{ a : 1, b : 1 }{ a : 1, b : 1}{ a : 10, b : 10 }

```angular2
sh.removeRangeFromZone( "exampledb.collection",
                { a : 1, b : 1 },
                { a : 10, b : 10 }
              )
```
给定前面的示例，如果存在具有下限和上限的现有范围，则操作不会删除该范围，因为它不是传递给的最小值和最大值的精确匹配。{ a : 1, b : 5 }{ a : 10, b : 1 }sh.removeRangeFromZone()


