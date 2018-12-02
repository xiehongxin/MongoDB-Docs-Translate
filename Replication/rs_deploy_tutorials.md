# 副本集部署教程

副本集部署可以采用命令行方式，也可以用配置文件方式，本次采用命令行方式。

## 如何创建副本集  
```python
# 1.首先需要先启动mongod服务（Centos7）
systemctl start mongod

# 2.创建文件夹存储数据
mkdir -p /home/hongxin/mongo_rs/rs0-1
mkdir -p /home/hongxin/mongo_rs/rs0-2
mkdir -p /home/hongxin/mongo_rs/rs0-3

# 3.创建三个节点的副本集
mongod --port 2001 --dbpath /home/hongxin/mongo_rs/rs0-1 --replSet rs_test &
mongod --port 2002 --dbpath /home/hongxin/mongo_rs/rs0-2 --replSet rs_test &
mongod --port 2003 --dbpath /home/hongxin/mongo_rs/rs0-3 --replSet rs_test &
# 4.连接到其中一个节点进行初始化
$ mongo --host 127.0.0.1 --port 2001
> rs.initiate({
... _id: "rs_test",
... members: [
... {_id:0,host:"127.0.0.1:2001"},
... {_id:1,host:"127.0.0.1:2002"},
... {_id:2,host:"127.0.0.1:2003"},
... ]})
# 5.查看配置
>rs.conf()
# 6.插入数据测试
rs_test:PRIMARY> db.mycoll.insertOne({"x": 1})
rs_test:PRIMARY> db.mycoll.find({})
{ "_id" : ObjectId("5c03d1339d9cc7cc9fbbf0ed"), "x" : 1 }

# 连接从节点
rs_test:SECONDARY> rs.slaveOk()  # 让从节点能够读取数据
rs_test:SECONDARY> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
testdb  0.000GB
rs_test:SECONDARY> use testdb
switched to db testdb
# 数据已经同步到从节点
rs_test:SECONDARY> db.mycoll.find({})
{ "_id" : ObjectId("5c03d1339d9cc7cc9fbbf0ed"), "x" : 1 }
```