# Capped Collections

## 目录
- [概览](#overview)
- [创建Capped Collection](#create-capped-collection)
- [限制及建议](#restrictions-and-recommendations)
- [相关操作](#procedures)

## Overview
Capped collections 是一种大小固定的集合，他基于插入排序算法，支持高吞吐量的文档插入和文档查询操作。

Capped collections 的工作原理有点像循环缓冲区，一旦它存储的空间满了，新的文档便会覆盖旧的文档。

## Create Capped Collection
```python
# 创建的语法： db.createCollection(<name>, options)

> use testdb
switched to db testdb
# 创建时必须指定大小
> db.createCollection('testdb.mycoll', {capped:true})
{
	"ok" : 0,
	"errmsg" : "specify size:<n> when capped is true",
	"code" : 14832,
	"codeName" : "Location14832"
}
# 创建一个大小为5242880bytes， 文档数为5000的collection
> db.createCollection('mycoll', {capped : true, size : 5242880, max : 5000})
{ "ok" : 1 }
> show collections
mycoll

```

有关创建的详细信息，参见[这里](https://docs.mongodb.com/manual/reference/method/db.createCollection/#db.createCollection)

## Restrictions and Recommendations

### 更新操作

如果你计划在一个capped集合更新里面的文档，则创建索引可以避免全表扫描


### 文档大小

从3.2版本开始， 如果你执行的更新或替换操作改变了capped集合的大小，这个操作将会失败

### 文档删除

我们是不能删除capped集合里面的文档的，如果要删除文档，只能把这个capped集合删除并重新创建一个新的

[drop() 方法](https://docs.mongodb.com/manual/reference/method/db.collection.drop/#db.collection.drop)

### 分片
capped 集合不能分片

### 有效查询
Use natural ordering to retrieve the most recently inserted elements from the collection efficiently. This is (somewhat) analogous to tail on a log file

### 聚合操作的$out
The aggregation pipeline operator $out cannot write results to a capped collection.

## Procedures
### 创建Capped Collection
创建步骤如上

### 查询Capped Collection
如果 find 方法没有指定查询方式，则默认是正向排序，即返回最早插入的文档

如果要反向查询，则可以用下列方法
>db.mycoll.find().sort( { $natural: -1 } )


### 判断是否为Capped Collection
```python
> db.mycoll.isCapped()
true
```

### 将普通集合转化为Capped Collection
```python
# 创建普通集合
> db.new_not_capped_coll.insertOne({'y': 1})
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5c01f8344fbbcfdb099452e4")
}
> db.new_not_capped_coll.isCapped()
false
# 转化为capped集合
> db.runCommand({"convertToCapped": "new_not_capped_coll", size: 1000});
{ "ok" : 1 }
> db.new_not_capped_coll.isCapped()
true

```

### 设置数据过期清理
该功能跟redis键的过期清理类似，详情参见[这里](https://docs.mongodb.com/manual/tutorial/expire-data/)

### Tailable Cursor
You can use a tailable cursor with capped collections. Similar to the Unix tail -f command, the tailable cursor “tails” the end of a capped collection. As new documents are inserted into the capped collection, you can use the tailable cursor to continue retrieving documents.

See [Tailable Cursors](https://docs.mongodb.com/manual/core/tailable-cursors/) for information on creating a tailable cursor.
