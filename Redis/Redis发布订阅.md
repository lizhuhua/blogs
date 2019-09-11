# Redis发布订阅
## 命令
发布消息：`publish channel message`<br>
订阅消息：`subscribe channel [channel]`<br>
取消订阅：`unsubscribe [channel]`<br>
按照模式订阅 `psubscribe pattern [pattern]`<br>
按照模式取消订阅 `punsubscribe [pattern]`<br>
查询订阅：`pubsub channels [pattern]`<br>
查看频道订阅数：`pubsub numsub [channel]`<br>
查看模式订阅数：`pubsub numpat`<br>
和很多专业的消息队列系统（例如Kafka、RocketMQ）相比，Redis的发布订阅略显粗糙，例如无法实现消息堆积和回溯，但胜在足够简单。<br>
