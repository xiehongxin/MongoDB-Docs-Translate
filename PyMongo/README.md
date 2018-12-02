# PyMongo

pymongo用 db.collection.find()出来是个游标,怎么才能转成数据?
```python

>>> data = coll.find({})
>>> type(data)
<class 'pymongo.cursor.Cursor'>
>>> data
<pymongo.cursor.Cursor object at 0x7fde4699acc0>

# 1. 用 for 循环
>>> data2 = coll.find({})
>>> for i in data2:
...     print(i)
... 
{'_id': ObjectId('5c01f8344fbbcfdb099452e4'), 'y': 1.0}

# 2. 用 list 
>>> list(data)
[{'_id': ObjectId('5c01f8344fbbcfdb099452e4'), 'y': 1.0}]

```


pymongo实现自增长id的方法
>待补充

pymongo根据_id查询数据
```python
# 来源： http://www.sharejs.com/codes/python/6401

# 如果pymongo的版本号小于2.2，使用下面的语句导入ObjectId
from pymongo.objectid import ObjectId

# 如果pymongo的版本号大于2.2，则使用下面的语句
from bson.objectid import ObjectId

# 查询代码如下：
>db.collection.find_one({'_id':ObjectId('50f0d76347f4ec148890ef1e')})

```

pymongo查看字段是否存在
```python
# 来源： https://blog.csdn.net/mijiang_the_cat/article/details/78333055

>>> from pymongo import MongoClient
>>> client = MongoClient('127.0.0.1')
>>> coll = client['testdb']['new_not_capped_coll']
# y字段是存在的
>>> coll.find({"y": {'$exists': True}}).count()
# x字段是不存在的
>>> coll.find({"x": {'$exists': True}}).count()
0

```

pymongo的多进程问题
https://www.cnblogs.com/dplearning/p/7779266.html  
https://segmentfault.com/q/1010000004867588   
https://www.cnblogs.com/zhangjpn/p/7775258.html   


获取倒数的数据 (-1 代表倒数，1代表正数)
db.financial.find().sort({report_date:-1}).limit(10);

删除一定时间的数据
 [mongodb：如何获取最后N条记录？](https://codeday.me/bug/20170221/3985.html)