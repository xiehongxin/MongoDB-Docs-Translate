# 副本集Oplog

- [副本集Oplog](#%E5%89%AF%E6%9C%AC%E9%9B%86oplog)
    - [什么是Oplog](#%E4%BB%80%E4%B9%88%E6%98%AFoplog)
    - [Oplog大小](#oplog%E5%A4%A7%E5%B0%8F)
    - [查看Oplog的数据](#%E6%9F%A5%E7%9C%8Boplog%E7%9A%84%E6%95%B0%E6%8D%AE)
    - [几种需要更大的Oplog的情况](#%E5%87%A0%E7%A7%8D%E9%9C%80%E8%A6%81%E6%9B%B4%E5%A4%A7%E7%9A%84oplog%E7%9A%84%E6%83%85%E5%86%B5)
        - [同时更新大量的文档](#%E5%90%8C%E6%97%B6%E6%9B%B4%E6%96%B0%E5%A4%A7%E9%87%8F%E7%9A%84%E6%96%87%E6%A1%A3)
        - [删除了与插入时相同大小的数据](#%E5%88%A0%E9%99%A4%E4%BA%86%E4%B8%8E%E6%8F%92%E5%85%A5%E6%97%B6%E7%9B%B8%E5%90%8C%E5%A4%A7%E5%B0%8F%E7%9A%84%E6%95%B0%E6%8D%AE)
        - [大量In-Place更新](#%E5%A4%A7%E9%87%8Fin-place%E6%9B%B4%E6%96%B0)
    - [Oplog的状态](#oplog%E7%9A%84%E7%8A%B6%E6%80%81)
    - [自己的一些思考](#%E8%87%AA%E5%B7%B1%E7%9A%84%E4%B8%80%E4%BA%9B%E6%80%9D%E8%80%83)
  

## 什么是Oplog
oplog（operations log）位于MongoDB的 local 数据库， 是该数据库下的一个特殊的capped collecton， 这个collection包含了修改数据集等操作的回滚记录。这个集合的大小是固定的，当空间用完时新记录会覆盖旧记录。

>注意: 从MongoDB4.0开始，不像其它capped collection， oplog 的大小可以突破我们所配置的oplog大小，从而避免删除 [majority commit point.](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/#replSetGetStatus.optimes.lastCommittedOpTime)

MongoDB在操作主节点时，会记录相应的操作到主节点的oplog。从节点随后就会用异步的方式，复制主节点oplog的内容，并把oplog的操作应用到自身的数据集。副本集的所有成员都有自己的一个oplog，这个oplog的数据在local数据库下的[oplog.rs](https://docs.mongodb.com/manual/reference/local-database/#local.oplog.rs) 集合里面。

为了促进复制，副本集的所有成员都会发送心跳包给其它成员（通过ping的方式，你在oplog里面也可以看到这些心跳信息），任何从节点都可以从其它节点导入oplog的数据。

oplog里面的每个操作应该是幂等的（可百度下幂等的含义，这意味者多次写入跟一次写入的效果是一样的）


## Oplog大小
当你启动一个member，或者说mongod实例时，如果没有指定oplog大小（通过添加 --oplogSize， 单位MB），则会创建一个默认大小的oplog。

oplog的大小是跟系统和存储引擎相关的。

对于Unix 和 Windows 系统

| Storage Engine | Default Oplog Size | Lower Bound | Upper Bound |
| --- | --- | --- | --- |
| [In-Memory Storage Engine](https://docs.mongodb.com/manual/core/inmemory/) | 5% of physical memory | 50 MB | 50 GB |
| [WiredTiger Storage Engine](https://docs.mongodb.com/manual/core/wiredtiger/) | 5% of free disk space | 990 MB | 50 GB |
| [MMAPv1 Storage Engine](https://docs.mongodb.com/manual/core/mmapv1/) | 5% of free disk space | 990 MB | 50 GB |

对于64位 macOS系统

| Storage Engine| Default Oplog Size|
| --- | --- |
|[In-Memory Storage Engine](https://docs.mongodb.com/manual/core/inmemory/) | 192 MB of physical memory
| [WiredTiger Storage Engine](https://docs.mongodb.com/manual/core/wiredtiger/) | 192 MB of free disk space
| [MMAPv1 Storage Engine](https://docs.mongodb.com/manual/core/mmapv1/) | 192 MB of free disk space

在大多数情况下， 默认大小的oplog是足够的。不过oplog也有不够用的情况，比如你的主节点oplog用的是默认大小，且可以存储24小时内的操作，那么从节点就可以在停止复制24小时后仍能追赶上主节点，而不需要重新获取全部数据。如果说从节点在24小时后开始追赶数据，那么不好意思主节点的oplog已经滚动覆盖了，把从节点没有执行的那条语句给覆盖了。这个时候为了保证数据一致性从节点就会终止复制。（从节点停止复制后，数据跟主节点就会不一致，这时需要人工resync）然而，大多数复制集中的操作没有那么频繁，oplog可以存放远不止上述的时间的操作记录。

在mongod进程创建oplog之前，我们可以通过指定 --oplogSize 来指定oplog的大小，一旦创建之后，则需要通过 [replSetResizeOplog](https://docs.mongodb.com/manual/reference/command/replSetResizeOplog/#dbcmd.replSetResizeOplog) 命令来修改，该命令的修改是动态的，即不需要重启mongod服务


```python
# 具体操作如下：
# 查看oplog原来的大小
>use local
>db.oplog.rs.stats().maxSize
# 修改大小
>use admin
# size的单位是MB
>db.adminCommand({replSetResizeOplog:1, size: 1638})
```
注：
1.该命令必须在admin数据库下执行  
2.用 replSetResizeOplog 修改 mongod实例的oplog 只对 Wired Tiger 数据存储引擎有效。  
3.通过replSetResizeOplog修改oplog有效，但是修改replication.oplogSizeMB的大小对oplog的大小是无影响的  
4.通过 replSetResizeOplog 改变的只是副本集其中一个成员的oplog大小，你必须在所有成员上执行此命令才会改变所有成员的oplog  
5.减少节点中的 oplog 的大小会从中删除数据， 这可能会导致从该节点同步数据的副本集的数据变得过时（没有得到更新）， 要重新同步这些数据，参见[Resync a Member of a Replica Set](https://docs.mongodb.com/manual/tutorial/resync-replica-set-member/)  



## 查看Oplog的数据
```python
# 一天内 oplog.rs 这个collection的数据量，大部分都是自动生成的（心跳包，每隔10s产生一个， 44800/60/60 = 12.4小时）
rs0:SECONDARY> db.oplog.rs.find().count()
4482

# oplog.rs 这个collection 数据的大小
rs0:SECONDARY> db.oplog.rs.dataSize()
496420

# oplog.rs 这个collection 分配的空间大小（包括未使用的控件）
rs0:SECONDARY> db.oplog.rs.storageSize()
196608

# oplog的最大值，这里显示的是1.4G, 我磁盘容量正好在28G左右，所以28G*0.05=1.4G
rs0:SECONDARY> db.oplog.rs.stats().maxSize
NumberLong(1419572019)   # 单位为bytes

```

## 几种需要更大的Oplog的情况
###  同时更新大量的文档
Oplog为了保证 幂等性 会将多项更新（multi-updates）转换为一条条单条的操作记录。这就会在数据没有那么多变动的情况下大量的占用oplog空间。

### 删除了与插入时相同大小的数据

如果我们删除了与我们插入时同样多的数据，数据库将不会在硬盘使用情况上有显著提升，但是oplog的增长情况会显著提升。

### 大量In-Place更新

如果我们会有大量的in-place更新，数据库会记录下大量的操作记录，但此时硬盘中数据量不会有所变化。

## Oplog的状态
要查看oplog状态（包括操作的大小和时间范围），请查看 [rs.printReplicationInfo（）](https://docs.mongodb.com/manual/reference/method/rs.printReplicationInfo/#rs.printReplicationInfo)方法。有关oplog状态的更多信息，请查看 ["检查Oplog的大小"](https://docs.mongodb.com/manual/tutorial/troubleshoot-replica-sets/#replica-set-troubleshooting-check-oplog-size)。

在各种异常情况下,对从节点的oplog的更新可能比我们估计的时间要慢一些，通过在从节点使用 [db.getReplicationInfo()](https://docs.mongodb.com/manual/reference/method/db.getReplicationInfo/#db.getReplicationInfo)  方法可以让我们评估当前副本集的状态及是否存在任何意外的复制延迟。

有关更多信息，请参见[Replication Lag](https://docs.mongodb.com/manual/tutorial/troubleshoot-replica-sets/#replica-set-replication-lag)


## 自己的一些思考
1.每个节点都有oplog， 假设硬盘20G，那么oplog应该在1G左右，如果有三个节点，且都在同一个机器上，那岂不是占了3G？   
答：对的，用空间换取数据的高可用   

2.从节点是怎么复制oplog的内容的？全部复制还是看时间复制？   
答：oplog每个操作/记录都有一个时间戳，个人认为是根据时间复制   

3.从节点同步主节点oplog的过程？   
答：假设一， 当主节点的oplog非常大，即从节点有非常多的时间去同步主节点oplog的内容，这种情况一般不会有什么问题   
假设二， 当主节点的oplog比较小，而且从节点宕机了，而主节点的oplog是不断增长的，**这时就会出现从节点的最新时间戳小于主节点的最旧时间戳**，如果继续同步的话必定会导致数据不一致，所以这时候从节点会停止复制oplog的内容，采用 Inital Sync 这种方式重新同步数据。   
假设三，当添加一个新的从节点时，从节点的最新时间戳必定小于主节点的最旧时间戳，所以也是采用 Inital Sync这种方式同步。  

4.单机模式是否会有oplog？  
答：没有  

5.从节点复制主节点oplog的频率？  
答：后台线程进行OPLOG复制到从节点，这个频率是非常高的，比日志刷盘频率还要高，从节点会一直监听主节点，OPLOG一有变化就会进行复制操作（所以除非从节点宕机，否则一般数据都能对得上）  

6.重点强调一下oplog只记录改变数据库状态的操作。比如，查询就不存储在oplog中   


参考：  
https://www.cnblogs.com/Joans/p/7723554.html  
http://www.ywnds.com/?p=3295  
https://docs.mongodb.com/manual/reference/command/replSetResizeOplog/#dbcmd.replSetResizeOplog  
 
