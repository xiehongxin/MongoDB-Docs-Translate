# 副本/复制

- [副本/复制](#%E5%89%AF%E6%9C%AC%E5%A4%8D%E5%88%B6)
    - [冗余与数据可用性](#%E5%86%97%E4%BD%99%E4%B8%8E%E6%95%B0%E6%8D%AE%E5%8F%AF%E7%94%A8%E6%80%A7)
    - [MongoDB的复制](#mongodb%E7%9A%84%E5%A4%8D%E5%88%B6)
        - [没有仲裁节点的情况](#%E6%B2%A1%E6%9C%89%E4%BB%B2%E8%A3%81%E8%8A%82%E7%82%B9%E7%9A%84%E6%83%85%E5%86%B5)
        - [有仲裁节点的情况](#%E6%9C%89%E4%BB%B2%E8%A3%81%E8%8A%82%E7%82%B9%E7%9A%84%E6%83%85%E5%86%B5)
    - [异步复制](#%E5%BC%82%E6%AD%A5%E5%A4%8D%E5%88%B6)
    - [自动容错](#%E8%87%AA%E5%8A%A8%E5%AE%B9%E9%94%99)
    - [读取操作](#%E8%AF%BB%E5%8F%96%E6%93%8D%E4%BD%9C)
    - [事务](#%E4%BA%8B%E5%8A%A1)
    - [其它特点](#%E5%85%B6%E5%AE%83%E7%89%B9%E7%82%B9)

MongoDB的副本集（replica set）其实是一组 mongod 进程，这组进程维护着相同的数据集，副本集提供冗余和高可用性，是所有生产部署的基础。

副本集允许有多个mongod进程，即多个节点，这些节点会选举出一个主节点，主节点会同步数据到从节点，所以主节点和从节点数据是一致的（不发生意外的情况下）


## 冗余与数据可用性  
副本提供冗余并提高数据可用性，通过在不同数据库服务器上提供多个数据副本（数据一样）。副本可提供一定程度的容错能力，以防止单机宕机带来的数据丢失。

在某些情况下，副本可以提高数据的读取速度，这是因为客户端可以将读取请求分散到不同服务器上（不同副本集），从而实现读写分离。另外，你还可以设置一些专用的副本集，用于灾难恢复，报告或备份。

## MongoDB的复制  
副本集一般包含几个节点，在这些节点中，会有一个节点被选举为主节点，没被选举到的则为从节点，另外，我们还可以在从节点中指定一个当仲裁节点（看个人需要）

### 没有仲裁节点的情况

![没有仲裁节点的架构图](http://hdoc.scau.edu.cn/Public/Uploads/2018-11-29/5bff9efce0434.png)

                                    没有仲裁节点的架构图

对于写入请求，只能写入到主节点；读取请求的话，默认也写到主节点，但可以设置从“从节点”读取。

对主节点数据集的操作会写入到 operation log中，即oplog。随后从节点会复制oplog并把这些操作应用到从节点的数据集中，从而保持数据的一致。

### 有仲裁节点的情况
![有仲裁节点的架构图](http://hdoc.scau.edu.cn/Public/Uploads/2018-11-29/5bff9ef43e900.png)

                                      有仲裁节点的架构图

这是副本集的另一种部署模式，即将其中一个mongod实例设置为仲裁节点。仲裁节点并没有包含数据集，也不参与主节点的选举，它的作用只是通过响应其它副本集成员的心跳（heartbeat）和选举请求来维护副本集中的仲裁（也就是选出新的主节点）。比如当主节点宕机，而从节点数量为偶数时，仲裁者就可以投出关键性的一票。

## 异步复制  
从节点将oplog应用到数据集的操作是异步的，详情查看 [Replica Set Oplog]() 和 [Replica Set Data Synchronization]()

## 自动容错
当主节点超出 electionTimeoutMillis（默认10s） 还没有响应从节点，则视主节点不可用，其它合格的从节点将会进行主节点的选举。

注意！在还没选出新的主节点之前，该副本集是无法进行写入操作的。对于读请求，如果该请求是分配到从节点的，则可以进行读取。

选举的中位时间（median time）一般不应该超过12s, 也就是12000毫秒。这个时间可以通过来设置 electionTimeoutMillis 来修整。

## 读取操作  
默认情况下，客户端从主节点读取数据。当然，我们也可以设置从“从节点”读取数据，不过要注意的是，这时候读取的数据不一定是最新的，详情参考 [Read Preference](https://docs.mongodb.com/manual/core/read-preference/).

另外，包含读取操作的事务操作必须在主节点进行！

这里还有一点read concern的内容，本人还未理解，详情请看官网

## 事务
1. 从MongoDB4.0开始，副本集开始支持多文档事务（ multi-document transactions）
2. 多文档事务中如果包含读取操作，则必须在主节点进行
3. 给定事务中的所有操作必须路由（route to）到同一个用户（member）


## 其它特点
https://docs.mongodb.com/manual/replication/