# 副本集成员

MongoDB中的副本集是一组提供冗余和高可用性的mongod进程。副本集有三种成员：
- 主节点
- 从节点：
- 仲裁节点： 选出主节点

## 主节点
接收所有的写入请求


## 从节点
- 备份，读写分离等
- 阻止从节点成为主节点，将其当作一个数据备份中心，详情参见[Priority 0 Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-priority-0-member/)
- 阻止从该节点读取数据，达到流量的隔离，详情请见[Hidden Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-hidden-member).
- 作为一个历史数据快照，方便宕机时从其恢复数据，详情请见[Delayed Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-delayed-member).

## 仲裁节点
- 不存储数据
- 不参与选举（priority被设为0, 1代表有选举的权利）


一个副本集的最小配置建议是包含三个节点， 有两种选择：
第一， 一个主节点+两个从节点；
第二，一个主节点+一个从节点+一个仲裁节点


参考文档：
https://docs.mongodb.com/manual/core/replica-set-members/
