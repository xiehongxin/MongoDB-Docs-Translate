# 数据库和集合

## 目录
- [数据库](#databases)
- [集合](#collections)

## Databases
>\# 选择一个数据库
use myDB

>\# 创建一个数据库（MongoDB的数据库不必显示创建，只要你使用一个数据库，并插入一条数据，便会自动生成数据库）
use myNewDB
db.myNewCollection1.insertOne( { x: 1 } )

[数据名字限制（不能起什么名字）](https://docs.mongodb.com/manual/reference/limits/#restrictions-on-db-names)

## Collections

MongoDB用collection存储文档，collection的结构有点像关系型数据库的二维表

### 默认创建
>db.myNewCollection2.insertOne( { x: 1 } )  
db.myNewCollection3.createIndex( { y: 1 } )

### 显示创建(可以指定文档的一些参数，比如文档的大小)
>db.createCollection()  
[参数说明](https://docs.mongodb.com/manual/reference/command/collMod/#dbcmd.collMod)


### 文档验证
默认情况下，MongoDB不要求其每个文档具有相同的模式; 即单个集合中的文档之间的字段与数据类型不一定要一样。

但是，从MongoDB 3.2开始，您可以在更新和插入操作期间强制执行集合的[文档验证规则](https://docs.mongodb.com/manual/core/schema-validation/)（字段及类型一致） 有关详细信息，请参阅[模式验证](https://docs.mongodb.com/manual/core/schema-validation/)。

### 修改文档的结构
如果你想要修改文档的结构，比如增加，删除，修改一个字段，使用update方法即可。

