# 哨兵模式
## 功能
* 监控：Sentinel节点会定期检测Redis数据节点、其余Sentinel节点是否可达<br>
* 通知：Sentinel节点会将故障转移结果通知应用方<br>
* 主节点故障转移：实现从节点晋升为主节点并维护后续正确的主从关系<br>
## 实现原理
* 三个定时监控任务<br>
1）每10秒，每个Sentinel节点会向主从节点发送info命令获取最新拓扑结构<br>
2）每2秒，每个Sentinel节点回想Redis数据节点的——sentinel_:hello频道上发送该Sentinel节点对于主节点的判断以及当前Sentinel节点信息。用以发现新Sentinel节点以及为后续选举领导提供依据<br>
3）每秒，每个Sentinel节点会向各节点发送一条ping命令，用作心跳检测。<br>
* 主观下线和客观下线<br>
主观下线：Sentinel节点进行心跳检测，超过down-after-milliseconds没有进行有效回复，因为是单个节点主观判断，存在误判<br>
客观下线：当Sentinel主观下线节点是主节点时，该节点会通过sentinel ismaster-down-by-addr命令向其他Sentinel节点询问对主节点的判断，当超过<quorun>个数认为确实有问题，则表示客观下线<br>
* 领导者节点选举<br>
通过Raft算法进行选举，大致思路是：<br>
每个节点都有资格成为领导者，当确认主节点主管下线时，会向其他节点发送sentinel ismaster-down-by-addr命令，要求自己成为领导。接受者在此之前未同意其他的此命令，将同意，否则拒绝。如票选数大于等于max(quorum，num（sentinels）/2+1)，将成为领导，否则进入下一次选举<br>
