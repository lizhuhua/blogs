# 缓存设计
## 缓存更新策略
* LRU/LRF/FIFO 算法剔除:一致性最差,维护成本低
* 超时剔除：一致性较差,维护成本较低
* 主动更新：一致性强，维护成本高
建议：<br>
    低一致性业务建议配置最大内存和算法剔除<br>
    高一致性业务可以结合使用超时剔除和主动更新。
## 缓存穿透优化
* 缓存空对象： 当未命中时，但保存null到缓存层。内存空间消耗大
* 布隆过滤器拦截： 将Key用布隆过滤器保存，下次访问是先判断布隆过滤器是否有值，一定程度保护存储层
## 无底洞优化： 
* 串行命令：遍历所有Key，执行O(keys)次IO
* 串行IO： 合并Key,执行O(nodes)次IO
* 并行IO： 合并Key,多线程执行O(nodes)次IO
* hash_tag： 将多个key强制分配到一个节点，减少IO（1）。
## 雪崩优化
* 保证缓存层服务高可用 Redis Sentinel/Redis Cluster
* 依赖隔离组件为后端限流并降级 Hystrix
## 热点Key重建优化
* 互斥锁（mutex key）： 只允许一个线程重建缓存，可能出现死锁情况
* 永不过期 ：过期后重新异步构建缓存，构建期间可能出现数据不一致情况
