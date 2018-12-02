# 分片键的选择

- [分片键的选择](#%E5%88%86%E7%89%87%E9%94%AE%E7%9A%84%E9%80%89%E6%8B%A9)
    - [什么是分片键](#%E4%BB%80%E4%B9%88%E6%98%AF%E5%88%86%E7%89%87%E9%94%AE)
    - [分片键跟索引的难舍难分](#%E5%88%86%E7%89%87%E9%94%AE%E8%B7%9F%E7%B4%A2%E5%BC%95%E7%9A%84%E9%9A%BE%E8%88%8D%E9%9A%BE%E5%88%86)
        - [前缀索引](#%E5%89%8D%E7%BC%80%E7%B4%A2%E5%BC%95)
        - [唯一索引](#%E5%94%AF%E4%B8%80%E7%B4%A2%E5%BC%95)
    - [分片键的约束](#%E5%88%86%E7%89%87%E9%94%AE%E7%9A%84%E7%BA%A6%E6%9D%9F)
    - [分片的方法](#%E5%88%86%E7%89%87%E7%9A%84%E6%96%B9%E6%B3%95)
    - [分片键的选择](#%E5%88%86%E7%89%87%E9%94%AE%E7%9A%84%E9%80%89%E6%8B%A9-1)



## 什么是分片键
引用下上一篇文章的内容：

分片键是数据分区，或者数据切割的前提， 从而实现数据分发到不同服务器上，减轻服务器的负担。

对于分片键，它也不是什么高级的东东，它就是文档里面的一个字段，但是这个字段不是普通的字段，有一定的要求
- 它必须在所有文档中都出现
- 它必须是集合的一个索引，可以是单索引或复合索引的前缀索引（后面会讲到）
- 它的值一但定下，就不能改变，或者修改

## 分片键跟索引的难舍难分

因为作为分片键的字段必须是索引，也就注定它与索引之间的难舍难分。

那既然分片键必须为文档里面的索引，那是不是所有索引都可以呢？

其实不是的，可以作为分片键的索引只有“单索引”和“复合索引的前缀索引”， 关于前缀索引请查看下文
（原文：All sharded collections must have an index that supports the shard key; i.e. the index can be an index on the shard key or a compound index where the shard key is a prefix of the index.）


### 前缀索引
索引前缀，或者说前缀索引，其实就是复合索引的一个子集。

举个例子，比如你有这样的一个复合索引 { "item": 1, "location": 1, "stock": 1 }

则该索引的子索引有：
- { item: 1 }
- { location: 1 }
- { stock: 1 }
- { item: 1, location: 1 }
- { item: 1, stock: 1 }
- { location: 1, stock: 1 }
- { "item": 1, "location": 1, "stock": 1 }

但前缀索引只有：
- { item: 1 }
- { item: 1, location: 1 }

查询操作支持的索引有以下：
- { item: 1 }
- { item: 1, location: 1 }
- { item: 1, stock: 1 }  # 我也不太懂，好像勉强算前缀索引，但是查询效率不高
- { "item": 1, "location": 1, "stock": 1 }  # 全索引

### 唯一索引

唯一索引的三种情况：
- 被指定为分片键的索引
- 被指定为分片键的前缀索引
- 默认索引_id

注：
>如果_id字段不是分片键的一部分，则_id索引仅对每个分片强制执行唯一性约束，而不是跨分片。

例如，考虑一个分片集合（带有分片键{x：1}），它跨越两个分片A和B.因为_id键不是分片键的一部分，所以该集合可以在分片A中具有_id值为1的文档和另一个在分片B中具有_id值1的文档。

唯一索引的意义(不太懂)：
- For a to-be-sharded collection, you cannot shard the collection if the collection has other unique indexes.
0 For an already-sharded collection, you cannot create unique indexes on other fields.


注：您不能在哈希索引上指定唯一约束。



## 分片键的约束
- 分片键的大小不能超过512字节
- 分片键的索引不能是多索引，文本索引或地理空间位置索引（geospatial index）
- 分片键不能被修改（修改分片键将会遭受灭顶之灾，因为要推倒重来）
- 分片键最好避免单调递增，这会使插入操作都指向单个分片，从而产生吞吐量的瓶颈（可用哈希索引避免这个问题）
- [更多约束](https://docs.mongodb.com/manual/reference/limits/#limits-shard-keys)



## 分片的方法

```python
# 1.指定要分片的数据库
mongos> sh.enableSharding("testdb")
# 2.指定要分片的集合及分片键 sh.shardCollection( namespace, key )
# 2.1 如果集合不存在， mongodb会创建一个唯一索引，并将其作为分片键
mongos> sh.shardCollection("testdb.mycoll2", {x: 1})
{
	"collectionsharded" : "testdb.mycoll2",
	"collectionUUID" : UUID("a60bcf4b-0c9f-4a63-bac9-8f26d3ffe44a"),
	"ok" : 1,
    ...
}
mongos> db.mycoll2.getIndexes()
[
	{
		"v" : 2,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_",
		"ns" : "testdb.mycoll2"
	},
	{
		"v" : 2,
		"key" : {
			"x" : 1
		},
		"name" : "x_1",
		"ns" : "testdb.mycoll2"
	}
]
# 如果集合存在，则必须先指定索引，再使用sh.shardCollection()方法


```


## 分片键的选择
- 基数
- 频率
- 变化率

理想的分片键能够使数据均匀分布在数据块中，而数据块又均匀地分布在集群上。

详情参见：
http://www.mongoing.com/understand-mongodb-shardkey


