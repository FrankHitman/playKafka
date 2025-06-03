# playKafka

## Concept
### Broker
Servers: Kafka is run as a cluster of one or more servers that can span multiple datacenters or cloud regions. Some of these servers form the storage layer, called the brokers.

### Event 消息事件
Producers generate event

event:
- key
- value
- timestamp

Producers and consumers are decoupled and agnostic of each other 生产者和消费者互相解耦和不依赖对方的，不用互相等待

### Topics
负责存储 event，保证序列化

Events are organized and durably stored in topics

Topics in Kafka are always multi-producer and multi-subscriber

events are not deleted after consumption，虽然不删除，但是可以定义过期时间，到期自动删除

### Partition 分区
Topics are partitioned, meaning a topic is spread over a number of "buckets" located on different Kafka brokers

不同生产者生产的events中如果拥有相同key的会被分配并附加到同一个 “分区” 中。
Kafka保证订阅特定key的消费者会找到那个“分区”，并且可以保证读取event的顺序和写入顺序一样。

### replication 备份
multiple brokers that have a copy of the data just in case things go wrong


## References
- https://kafka.apache.org/intro

