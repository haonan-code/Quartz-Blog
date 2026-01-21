**架构图：**
```nginx
Producer → Topic → Partition → Broker → Consumer Group
```
- 分区保证**顺序性**
- 消费组支持**水平扩展**
- 存储在磁盘，**持久化可靠**
**使用场景：**

| 场景       | 描述                      |
| -------- | ----------------------- |
| 用户行为日志收集 | 实时采集海量日志数据              |
| 实时数据分析   | Kafka → Spark/Flink 流处理 |
| 秒杀订单流转   | 通过 partition 顺序处理订单     |

















