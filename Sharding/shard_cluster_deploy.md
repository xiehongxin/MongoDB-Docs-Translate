# 分片集群的部署


分片集群的部署有两种方式，一是把参数写在命令行中，而是把参数写在配置文件中，这两种方式是等价的，下面的链接是命令行参数与配置文件参数对应表：
https://docs.mongodb.com/manual/reference/configuration-file-settings-command-line-options-mapping/#conf-file-command-line-mapping


下面讲述的是如何在单机上部署我们的测试分片集群（采用配置文件方式）：

## 1.配置说明

组件名 | IP | 端口 | 备注
------- | ------- | ------- | -------
configserver | 127.0.0.1 | 27001 | 最先启动，副本集单成员形式
shard1 | 127.0.0.1 | 27003 | 第二启动，副本集单成员形式
shard2 | 127.0.0.1 | 27005 | 第三启动，副本集单成员形式
mongos | 127.0.0.1 | 27007 | 最后启动

## 2.创建对应的文件夹：
```python
mkdir -p /home/hongxin/mongodb/conf
mkdir -p /home/hongxin/mongodb/mongos/log
mkdir -p /home/hongxin/mongodb/config/data
mkdir -p /home/hongxin/mongodb/config/log
mkdir -p /home/hongxin/mongodb/shard1/data
mkdir -p /home/hongxin/mongodb/shard1/log
mkdir -p /home/hongxin/mongodb/shard2/data
mkdir -p /home/hongxin/mongodb/shard2/log
```

## 3.新建config server 副本集

注：本次为了测试，该副本集只包含一个成员，实际生产环境应至少有三个成员）


### 配置文件说明
从MongoDB2.6开始，配置文件开始采用YAML的格式（YAML支持空格，不支持Tab）
```python
# 2.6版本之前的写法（键值对）
configsvr = true
replSet=configs

# 2.6版本以后的写法
sharding:
   clusterRole: configsvr
replication:
   replSetName: <replica set name>
```

配置参数说明： https://docs.mongodb.com/manual/reference/configuration-options/

### 新建config.conf文件
```python
>vim /home/hongxin/mongodb/conf/config.conf
# 填入以下内容
systemLog: 
   destination: file 
   path: /home/hongxin/mongodb/config/log/configsrv.log 
   logAppend: true 
storage: 
   dbPath: /home/hongxin/mongodb/config/data 
   journal: 
      enabled: true 
processManagement: 
   pidFilePath: /home/hongxin/mongodb/config/log/configsrv.pid 
   fork: true 
net: 
   bindIp: 127.0.0.1 
   port: 27001 
sharding: 
   clusterRole: configsvr   # 代表此进程是config server
replication:  
  replSetName: configs  # 副本集名称

```

### 运行并初始化
```python
# 创建进程
[hongxin@localhost conf]$ mongod -f /home/hongxin/mongodb/conf/config.conf

2018-12-02T15:29:15.608+0800 I CONTROL  [main] Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'
about to fork child process, waiting until server is ready for connections.
forked process: 9587
child process started successfully, parent exiting
# 连接 
[hongxin@localhost conf]$ mongo --host 127.0.0.1 --port 27001
# 初始化，默认创建一个成员
>rs.initiate() 
# 如果要创建多个成员，参见下面
rs.initiate(
  {
    _id: "<replSetName>",
    configsvr: true,
    members: [
      { _id : 0, host : "cfg1.example.net:27019" },
      { _id : 1, host : "cfg2.example.net:27019" },
      { _id : 2, host : "cfg3.example.net:27019" }
    ]
  }
)

```


## 4.新建shard副本集
shard的创建方式与config server大同小异
### 新建shard1.conf文件
```python
systemLog: 
   destination: file 
   path: /home/hongxin/mongodb/shard1/log/shard1.log 
   logAppend: true 
storage: 
   dbPath: /home/hongxin/mongodb/shard1/data 
   journal: 
      enabled: true 
processManagement: 
   pidFilePath: /home/hongxin/mongodb/shard1/log/shard1.pid 
   fork: true 
net: 
   bindIp: 127.0.0.1 
   port: 27003 
sharding: 
   clusterRole: shardsvr 
replication: 
  replSetName: shard1

```
### 创建连接及初始化
```python
[hongxin@localhost conf]$ mongod -f /home/hongxin/mongodb/conf/shard1.conf
[hongxin@localhost conf]$ mongo --host 127.0.0.1 --port 27003
>rs.initiate() 
```

shard2 的创建与上面类似

注： shard 副本集的名字与 config server的名字不能一样！


## 5.新建mongos
### 新建 mongos.conf
注意 mongos.conf 没有存储选项
```python
systemLog: 
   destination: file 
   path: /home/hongxin/mongodb/mongos/log/mongos.log 
   logAppend: true  
processManagement: 
   pidFilePath: /home/hongxin/mongodb/mongos/log/mongos.pid 
   fork: true 
net: 
   bindIp: 127.0.0.1 
   port: 27007 
sharding: 
   configDB: configs/127.0.0.1:27001

```

### 创建连接及初始化

```python
[hongxin@localhost conf]$ mongos -f /home/hongxin/mongodb/conf/mongos.conf

[hongxin@localhost conf]$ mongo --host 127.0.0.1 --port 27007

mongos> sh.addShard("shard1/127.0.0.1:27003, 127.0.0.1:27005")

```

**至此，单机分片集群已搭建完毕**


## 测试
```python
mongos> use testdb
switched to db testdb
mongos> db.mycoll.insertOne({id: 1, "test": "test_shard"})
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5c039517071e1ba7e339fe24")
}
# 首先要让数据库启动分片
mongos> sh.enableSharding("testdb")
{
	"ok" : 1,
	"operationTime" : Timestamp(1543738671, 3),
	"$clusterTime" : {
		....
}
mongos> db.mycoll.find({})
{ "_id" : ObjectId("5c039517071e1ba7e339fe24"), "id" : 1, "test" : "test_shard" }
# 因为集合不为空，所以在执行shardCollection前要先创建索引，如果集合为空，会自动创建索引
mongos> db.mycoll.createIndex({id: 1})
{
	"raw" : {
		"shard1/127.0.0.1:27003" : {
			"createdCollectionAutomatically" : false,
			"numIndexesBefore" : 1,
			"numIndexesAfter" : 2,
			"ok" : 1
		}
	},
	...
}
# 查看索引
mongos> db.mycoll.getIndexes()
[
	{
		"v" : 2,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_",
		"ns" : "testdb.mycoll"
	},
	{
		"v" : 2,
		"key" : {
			"id" : 1
		},
		"name" : "id_1",
		"ns" : "testdb.mycoll"
	}
]
# 使用 _id 作为分片键
mongos> sh.shardCollection("testdb.mycoll", {_id:1})
{
	"collectionsharded" : "testdb.mycoll",
	"collectionUUID" : UUID("cffb9588-52df-4df4-bb1a-9e461602cabd"),
	"ok" : 1,
	...
}
# 因为先使用 _id 作为分片键，再使用 id 会报错
mongos> sh.shardCollection("testdb.mycoll", {id:1})
{
	"ok" : 0,
	"errmsg" : "sharding already enabled for collection testdb.mycoll with options { _id: \"testdb.mycoll\", lastmodEpoch: ObjectId('5c039722733b2ba5a2bafebd'), lastmod: new Date(4294967296), dropped: false, key: { _id: 1.0 }, unique: false, uuid: UUID(\"cffb9588-52df-4df4-bb1a-9e461602cabd\") }",
	"code" : 23,
	"codeName" : "AlreadyInitialized",
  ...
}

# 插入100000条数据
mongos> for (var i = 1; i <= 100000; i++) db.mycoll.save({id:i,"test1":"test_shard"});
WriteResult({ "nInserted" : 1 })

# 查看结果（不知道是不是因为分片键为_id的缘故，数据并没有分散到shard2中）
mongos> db.mycoll.stats()
```



## 后期运维
mongodb的启动顺序是，先启动配置服务器，在启动分片，最后启动mongos.  

mongod -f /usr/local/mongodb/conf/config.conf  
mongod -f /usr/local/mongodb/conf/shard1.conf  
mongod -f /usr/local/mongodb/conf/shard2.conf  
mongod -f /usr/local/mongodb/conf/shard3.conf  
mongod -f /usr/local/mongodb/conf/mongos.conf  

关闭时，直接killall杀掉所有进程  
killall mongod  
killall mongos  


参考：
https://docs.mongodb.com/manual/tutorial/deploy-shard-cluster/index.html  
http://www.ityouknow.com/mongodb/2017/08/05/mongodb-cluster-setup.html  
