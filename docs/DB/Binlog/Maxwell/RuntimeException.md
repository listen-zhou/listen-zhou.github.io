# 运行时异常解决

## 1.Couldn't find table cmfmgqualifications in database winddb

```java
2022-02-22 18:24:05:663] [ERROR] - com.zendesk.maxwell.util.TaskManager.stop(TaskManager.java:42) - cause: 
java.lang.RuntimeException: Couldn't find table cmfmgqualifications in database winddb
	at com.zendesk.maxwell.replication.TableCache.processEvent(TableCache.java:31) ~[maxwell-1.28.2.jar:1.28.2]
	at com.zendesk.maxwell.replication.BinlogConnectorReplicator.getTransactionRows(BinlogConnectorReplicator.java:502) ~[maxwell-1.28.2.jar:1.28.2]
	at com.zendesk.maxwell.replication.BinlogConnectorReplicator.getRow(BinlogConnectorReplicator.java:626) ~[maxwell-1.28.2.jar:1.28.2]
	at com.zendesk.maxwell.replication.BinlogConnectorReplicator.work(BinlogConnectorReplicator.java:178) ~[maxwell-1.28.2.jar:1.28.2]
	at com.zendesk.maxwell.util.RunLoopProcess.runLoop(RunLoopProcess.java:34) ~[maxwell-1.28.2.jar:1.28.2]
	at com.zendesk.maxwell.Maxwell.startInner(Maxwell.java:247) ~[maxwell-1.28.2.jar:1.28.2]
	at com.zendesk.maxwell.Maxwell.start(Maxwell.java:175) ~[maxwell-1.28.2.jar:1.28.2]
	at com.zendesk.maxwell.Maxwell.main(Maxwell.java:273) ~[maxwell-1.28.2.jar:1.28.2]

```

解决方法：升级到最新版，当前版本有bug，大概问题是，表被删除后又重建，但是应用没有刷新表配置列表，导致无法找到该表。

## 2.发送到kafka topic分区数据倾斜

修改配置config.properties

```ini
#######修改partition_by，解决kafka数据倾斜######
kafka_partition_hash=murmur3
producer_partition_by=primary_key
```

