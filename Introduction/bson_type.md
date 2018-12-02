# BSON类型

## 目录
- [BSON 类型](#bson-type)
- [BSON 几种特殊类型](#bson-specicial-type)


[BSON](https://docs.mongodb.com/manual/reference/glossary/#term-bson) 是一种二进制序列化后的格式，用于存储文档并被MongoDB用来远程调用。BSON的规范在 [bsonspec.org](http://bsonspec.org/) 网站

## BSON Type
BSON主要有以下类型：  

Type | Number | Alias | Notes  
-----|--------|-------|------  
Double | 1 | “double” |    
String | 2 | “string” |    
Object | 3 | “object” |  
Array | 4 | “array” |  
Binary data | 5 | “binData” |  
Undefined | 6 | “undefined” | Deprecated.
ObjectId | 7 | “objectId” |  
Boolean | 8 | “bool” |  
Date | 9 | “date” |  
Null | 10 | “null” |  
Regular Expression | 11 | “regex” |  
DBPointer | 12 | “dbPointer” | Deprecated.
JavaScript | 13 | “javascript” |  
Symbol | 14 | “symbol” | Deprecated.
JavaScript (with scope) | 15 | “javascriptWithScope” |  
32-bit integer | 16 | “int” |  
Timestamp | 17 | “timestamp” |  
64-bit integer | 18 | “long” |  
Decimal128 | 19 | “decimal” | New in version 3.4.
Min key | -1 | “minKey” |  
Max key | 127 | “maxKey”

利用上述类型，我们可以通过 [$type](https://docs.mongodb.com/manual/reference/operator/query/type/#op._S_type) 这个操作器来选择出我们想要的类型的数据，比如你有一个字段 tag， 它的类型是不确定的，有可能为int，也有可能为string，用 $type 可以筛选出指定类型的数据。
> db.news_society.find({'tag': {$type: "int"}})  # 注意int加双引号


如果我们要查看字段的类型，可以通过 typeof 命令获得
```python
> a = db.news_society.findOne({'tag': {$type: "int"}})
{
	"_id" : "8b6b01f0e590b3e0f3891ef3242c9b0e",
	"tag" : 1,
	"num" : 1,
	"url" : "https://news.qq.com/a/20161116/5491d4e8b86463b3d3ee2944c77c2b35.htm",
	"content" : "西风挟雨声翻浪。恰洗尽、黄茅瘴。老惯人间齐得丧。千岩高卧，五湖归棹，替却凌烟像。故人小驻平戎帐，白羽腰间气何壮。我老渔樵君将相。小槽红酒，晚香丹荔，记取蛮江上。0000",
	"article_title" : "fff",
	"timestamp" : 1542017925.373029
}
> typeof a._id
string
> typeof a.tag
number

```

如果我们要把BSON类型转化为JSON，可以参考[这里](https://docs.mongodb.com/manual/reference/mongodb-extended-json/)


## BSON Specicial Type

下面讲述的是BSON几种特殊类型：

### ObjectId
ObjectIds很小（个人认为并不小），可能有点独特，但是生成和排序非常快。（ObjectIds are small, likely unique, fast to generate, and ordered.）

ObjectId的值由12个字节组成，分别如下：
- 一个大小为4个字节的时间戳，Unix时间
- 一个大小为5个字节的随机数
- 一个大小为3个字节的计数器，从随机数开始算起

在MongoDB中，每个集合里面的文档都需要一个“_id”字段（类型为ObjectId）作为主键。如果你插入的文档缺少_id这个字段，则MongoDB会自动生成一个，生成方法如上。

上述方式适用于“更新文档操作（update）， 且条件为 “upsert：true”的情况。

通过使用_id这种唯一键的好处：
- 通过 ObjectId.getTimestamp() 获取插入时的时间戳（注意时区）
- 根据 _id 排序大致相当于按创建时间排序


为什么可以根据_id排序？  
ObjectId 看起来虽然像是经过hash的值，但是实际上它是一组十六进制的字符串，前4位是时间戳，而时间戳是不断递增的，所以这组数会不断变大，当然，这只是一种大致排序，而不是精确排序，原因有二：
- 时间戳的精确度只精确到秒，但一秒内是可以创建多个_id的，所以同一秒创建的_id无法排序
- 这个时间戳是由客户端生成的，所以不同系统产生的时间也会有区别

参考：  
[MongoDB深究之ObjectId](https://www.cnblogs.com/xjk15082/archive/2011/09/18/2180792.html)  
[ObjectId()](https://docs.mongodb.com/manual/reference/method/ObjectId/#ObjectId)

### String
BSON的字符串类型采用UTF-8编码。在序列化和反序列化BSON时，每种编程语言的驱动程序都会将它们自身的字符串编码转换为UTF-8。这样可以轻松地将大多数国际字符存储在BSON字符串中。 此外，MongoDB的 正则查询（$ regex）也支持UTF-8。

注：通过UTF-8编码的字符串，用sort()方法大多数情况是没问题的，但是因为sort()方法底层使用的是 C++ 的strcmp的api， 用这个方法处理某些字符串可能会出错。

### Timestamps
BSON的Timestamp类型比较特殊，主要给内部使用，它跟下面讲到的Date类型是没有多大关系的。Timestamp的大小为64位：
- 前32位是一个Unix时间的时间戳
- 后面的32位是一个给定时间内（秒）操作的递增序数

在单个mongod实例中，时间戳值始终是唯一的。

在副本集中，oplog包含一个ts字段。此字段中的值反映了使用BSON时间戳值的操作时间。

如果在顶级字段中插入包含空BSON时间戳的文档，MongoDB服务器将使用当前时间戳值替换该空时间戳。例如，如果您创建插入带有时间戳值的文档，如以下操作：

**顶级字段：** 比如在 {"ts": {"time": 13422}, "_id": "2drf43"}中，ts 和 _id 就是顶级字段

**操作时间：** 暂时还未理解

```python
>var a = new Timestamp();
>db.test.insertOne( { ts: a } );

>db.test.find()
{ "_id" : ObjectId("542c2b97bac0595474108b48"), "ts" : Timestamp(1412180887, 1) }
```

而如果空时间戳的字段非顶级字段，则保留空时间戳


### Date
BSON的Date类型是大小为64位的整型时间戳，也是Unix时间，精确到毫秒。
需要注意的是，Date采用UTC时间，所以有时区问题

```python
# 只有Date() 方法有加上时区
> date1 = Date()
Thu Nov 29 2018 23:53:07 GMT+0800 (CST)
> date2 = new Date()
ISODate("2018-11-29T15:53:29.403Z")
> date3 = ISODate()
ISODate("2018-11-29T15:53:41.685Z")
> date4 = new ISODate()
ISODate("2018-11-29T15:53:54.962Z")

# toString()方法会自动转换时区
> date1s = date1.toString()
Thu Nov 29 2018 23:53:24 GMT+0800 (CST)
> date2s = date2.toString()
Thu Nov 29 2018 23:53:29 GMT+0800 (CST)
> date3s = date3.toString()
Thu Nov 29 2018 23:53:41 GMT+0800 (CST)
> date4s = date4.toString()
Thu Nov 29 2018 23:53:54 GMT+0800 (CST)

> date1s = date1.getMonth()  # 注意月份是从0开始的
10
```
疑问： Date居然是整型的？

参考：   
[一些时间的概念与区分(UTC、GMT、LT、TAI等)](https://www.cnblogs.com/LubinLew/p/Knownledge_Time.html)

