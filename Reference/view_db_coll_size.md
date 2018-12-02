# 查看数据库和集合大小

来源：https://www.jb51.net/article/52517.htm

```python
# 数据库
> db.stats(); 
{ 
  "db" : "test",        //当前数据库 
  "collections" : 3,      //当前数据库多少表 
  "objects" : 4,        //当前数据库所有表多少条数据 
  "avgObjSize" : 51,      //每条数据的平均大小 
  "dataSize" : 204,      //所有数据的总大小 
  "storageSize" : 16384,    //所有数据占的磁盘大小 
  "numExtents" : 3, 
  "indexes" : 1,        //索引数 
  "indexSize" : 8176,     //索引大小 
  "fileSize" : 201326592,   //预分配给数据库的文件大小 
  "nsSizeMB" : 16, 
  "dataFileVersion" : { 
    "major" : 4, 
    "minor" : 5 
  }, 
  "ok" : 1 
} 

# 集合
> db.mycoll.stats(); 
{ 
  "ns" : "test.posts", 
  "count" : 1, 
  "size" : 56, 
  "avgObjSize" : 56, 
  "storageSize" : 8192, 
  "numExtents" : 1, 
  "nindexes" : 1, 
  "lastExtentSize" : 8192, 
  "paddingFactor" : 1, 
  "systemFlags" : 1, 
  "userFlags" : 0, 
  "totalIndexSize" : 8176, 
  "indexSizes" : { 
    "_id_" : 8176 
  }, 
  "ok" : 1 
} 
```