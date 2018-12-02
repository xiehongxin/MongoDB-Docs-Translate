# 副本集数据同步

- [副本集数据同步](#%E5%89%AF%E6%9C%AC%E9%9B%86%E6%95%B0%E6%8D%AE%E5%90%8C%E6%AD%A5)
    - [Initial Sync](#initial-sync)
        - [Initial Sync的条件：](#initial-sync%E7%9A%84%E6%9D%A1%E4%BB%B6)
        - [Initial Sync的具体过程：](#initial-sync%E7%9A%84%E5%85%B7%E4%BD%93%E8%BF%87%E7%A8%8B)
    - [Replication](#replication)

为了维护数据的一致性，副本集的从节点通过Inital Sync或者复制其它成员的数据的方式来同步。MongoDB使用两种方式的数据同步： Inital Sync 全量拉取数据及 replication 增量拉取。

注：并不是用Inital Sync这种方式就不会用到Replication，后面会讲到。

## Initial Sync
Sync策略通过拷贝副本集的一个节点的所有数据到另外一个节点，实现数据一致性

**下面的内容来自于“运维那点事”，链接如下， 特此说明**  
http://www.ywnds.com/?p=3295

### Initial Sync的条件：
1.Secondary上oplog为空，比如新加入的空节点。

2.local.replset.minvalid集合里_initialSyncFlag标记被设置。当initial sync开始时，同步线程会设置该标记，当initial sync结束时清除该标记，故如果initial sync过程中途失败，节点重启后发现该标记被设置，就知道应该重新进行initial sync。

3.BackgroundSync::_initialSyncRequestedFlag被设置。当向节点发送resync命令时，该标记会被设置，此时会强制重新initial sync。


### Initial Sync的具体过程：  
1.minValid集合设置_initialSyncFlag（db.replset.minvalid.find()）。  
2.获取同步源当前最新的oplog时间戳t0。  
3.从同步源克隆所有的集合数据。  
4.获取同步源最新的oplog时间戳t1。  
5.同步t0~t1所有的oplog。(因为全量拉取数据耗时是特别长的，这期间oplog会继续增加)  
6.获取同步源最新的oplog时间戳t2。   
7.同步t1~t2所有的oplog。   
8.从同步源读取index信息，并建立索引（除了_id ，这个之前已经建立完成）。   
9.获取同步源最新的oplog时间戳t3。  
10.同步t2~t3所有的oplog。   
11.minValid集合清除_initialSyncFlag，initial sync结束。  

当完成了所有操作后，该节点将会变为正常的状态secondary。

To perform an initial sync, see [Resync a Member of a Replica Set.](https://docs.mongodb.com/manual/tutorial/resync-replica-set-member/)

## Replication  
initial sync结束后，Secondary会建立到Primary上local.oplog.rs的tailable cursor，不断从Primary上获取新写入的oplog，并应用到自身。Tailable cursor每次会获取到一批oplog，Secondary采用多线程重放oplog以提高效率，通过将oplog按照所属的namespace进行分组，划分到多个线程里，保证同一个namespace的所有操作都由一个线程来replay，以保证统一namespace的操作时序跟primary上保持一致（如果引擎支持文档锁，只需保证同一个文档的操作时序与primary一致即可）。

注： 
1.从MongoDB3.2开始，副本集成员会有一个vote字段，vote为1的不能从vote为0的同步数据  
2.延迟成员和隐藏成员也不会被同步  