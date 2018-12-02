# 比较/排序时的顺序

## 目录
- Numeric Types
- Strings
- Arrays
- Objects
- Dates and Timestamps
- Non-existent Fields
- BinData


在比较不同BSON类型的值时，MongoDB使用以下比较顺序，从最低到最高：

MinKey (internal type)
Null
Numbers (ints, longs, doubles, decimals)
Symbol, String
Object
Array
BinData
ObjectId
Boolean
Date
Timestamp
Regular Expression
MaxKey (internal type)

详情参见：https://docs.mongodb.com/manual/reference/bson-type-comparison-order/