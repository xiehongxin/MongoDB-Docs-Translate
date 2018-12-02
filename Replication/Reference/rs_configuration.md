# 副本集配置

- [副本集配置](#%E5%89%AF%E6%9C%AC%E9%9B%86%E9%85%8D%E7%BD%AE)
	- [副本集配置的小例子](#%E5%89%AF%E6%9C%AC%E9%9B%86%E9%85%8D%E7%BD%AE%E7%9A%84%E5%B0%8F%E4%BE%8B%E5%AD%90)
		- [查看配置](#%E6%9F%A5%E7%9C%8B%E9%85%8D%E7%BD%AE)
		- [修改配置](#%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE)
	- [副本集配置的字段说明](#%E5%89%AF%E6%9C%AC%E9%9B%86%E9%85%8D%E7%BD%AE%E7%9A%84%E5%AD%97%E6%AE%B5%E8%AF%B4%E6%98%8E)

前面提到Mongo通过副本集实现备份，或者说数据的高可用，但是默认的配置跟我们的实际配置并不会那么契合，那如何查看和设置副本集的配置文件呢？

## 副本集配置的小例子
### 查看配置
```python
1.通过 rs.conf() 查看（任何数据库都可执行）
rs0:PRIMARY> rs.conf()
{
	"_id" : "rs0",
	"version" : 3,
	"protocolVersion" : NumberLong(1),
	"writeConcernMajorityJournalDefault" : true,
...
}

2.通过 replSetGetConfig 查看（只能在admin执行）
rs0:PRIMARY> db.runCommand( { replSetGetConfig: 1 } )
{
	"operationTime" : Timestamp(1543108272, 1),
	"ok" : 0,
	"errmsg" : "replSetGetConfig may only be run against the admin database.",
	"code" : 13,
	"codeName" : "Unauthorized",
	"$clusterTime" : {
		"clusterTime" : Timestamp(1543108272, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}

虽然上面也返回了一些信息，但这些其实是报错信息，报错信息提示见 errmsg 字段，它提示这个命令应该在admin数据库执行。

\# 切换到admin并执行，果然出现了配置信息
rs0:SECONDARY> use admin
switched to db admin
rs0:SECONDARY> db.runCommand( { replSetGetConfig: 1 } )
{
	"config" : {
		"_id" : "rs0",
		"version" : 3,
		"protocolVersion" : NumberLong(1),
		"writeConcernMajorityJournalDefault" : true,
		"members" : [
```

### 修改配置
修改配置的话可以采用 rs.reconfig() 方法，比如我们把 electionTimeoutMillis 这个字段（选举超时时间）的值由10000改为9000（官网建议这个值不要超过12000）。
注： 修改配置必须在主节点上进行
```python
rs0:PRIMARY> cfg = rs.conf();
rs0:PRIMARY> cfg.settings.electionTimeoutMillis = 9000;
9000
rs0:PRIMARY> rs.reconfig(cfg)
{
	"ok" : 1,
	"operationTime" : Timestamp(1543109593, 1),
	"$clusterTime" : {

```
注： 修改配置的过程可能会导致主节点不可用，这时也无法写入数据到主节点，所以建议在空闲时间修改配置，当修改完毕后会重新选举主节点。

## 副本集配置的字段说明
```python
# 用户==节点==实例
_id(最上层的)： 副本集的名字
version： 配置文件的版本号，每修改一次配置文件，该版本号递增
configsvr： （没找到该字段）
protocolVersion： 应该是协议版本，从4.0起，该值为1而不是0
writeConcernMajorityJournalDefault： 还没搞懂
members： 副本集的用户，该字段的值是一个数组，索引从0开始
members[n]._id：标志副本集的用户，该值的范围从 0-255, 这也意味副本集的用户数量上限为255, 该_id 一旦设置，不能再改变
members[n].host：指定IP和端口。当所有mongod实例都在本机上运行时，IP可以设置为localhost，否则的话设置为localhost会出错。
members[n].arbiterOnly：当该值为true时代表此用户是一个仲裁节点
members[n].buildIndexes：当该值为true时，该用户会建立索引。该值的设置只能在添加用户的时候指定（用rs.add()）, 一旦用户添加完毕，该值不能再被修改。
当该用户有查询操作时，该值不能设置为false。
members[n].hidden：当该值为true时，会隐藏该实例（用户），在该实例上也无法进行查询操作。
members[n].priority：表明哪些节点有成为主节点的资格（主节点和从节点的值为1, 仲裁节点值为0）
members[n].tags： 还没搞懂
members[n].slaveDelay：还没搞懂
members[n].votes： 在副本集选举为主节点时拥有的票数
settings：设置选项（对整个副本集有效）
settings.chainingAllowed：当设置为true时，从节点只能从主节点同步数据；当设置为false时，可以从另外的从节点同步数据
settings.getLastErrorDefaults：还没搞懂
settings.getLastErrorModes：还没搞懂
settings.heartbeatTimeoutSecs：心跳超时时间，默认10s（各节点每隔一段时间会发送一个心跳包来检测节点是否可用，如果在这个时间内节点没有响应，则认为这个节点不可用）
settings.electionTimeoutMillis：选举超时时间，默认10s（当主节点不可用，经过这个时间后，从节点会开始进行选举）
settings.catchUpTimeoutMillis：当选举出一个新节点后，从其它节点同步最新数据的限制时间
settings.catchUpTakeoverDelayMillis：还没搞懂
settings.heartbeatIntervalMillis：心跳包发送的频率（每隔多少毫秒发送一次）
settings.replicaSetId：副本集的ID

```
本人英语能力有限，若有翻译错误的地方还望在评论中指正，不胜感激！

建议看完此文章再看下官方文档，写得更详细

参考：
https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.settings
