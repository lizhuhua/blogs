# 内存管理
## 1.内存上限控制
  Redis使用maxmemory参数限制Redis实际使用的内存（used_memory）<br>
  修改maxmemory命令: `config set maxmemory *GB`<br>
  修改maxmemory可以达到扩容的目的，但如果内存超出物理内存时需要采用在线`迁移数据`或者`复制切换服务器`来扩容。
## 2.内存回收策略
  * 删除过期键对象
  惰性删除：客户端访问已过期的键时，会执行删除操作返回空。缺点：如一直没有访问会导致内存不能及时释放<br>
  定时任务删除：默认采用慢模式,每个数据库空间随机检查20个键，超过25%过期执行删除，执行时间超过25毫秒之后执行快模式，快模式超时时间为1毫秒<br>
  * 内存溢出控制策略
  内存使用达到maxmemory上限时触发内存溢出控制策略，策略受maxmemory-policy参数控制<br>
  noeviction：默认策略，只读<br>
  volatile-lru：根据LRU算法删除设置了超时属性的键，直到腾出足够空间为止。如没有可删除的键对象，回退noeviction策略<br>
  allkeys-lru：根据LRU算法删除键，直到腾出足够空间为止<br>
  volatile-random：随机删除过期属性，直到腾出足够空间为止<br>
  allkeys-random：随机删除所有键，直到腾出足够空间为止<br>
  volatile-ttl：根据键值对象的ttl属性，删除最近将要过期的数据。如果没有，回退noeviction策略<br>
  建议：Redis工作在maxmemory>used_memory状态下，避免频繁内存回收开销。<br>
  触发内存回收方式：将maxmemory设置为used_memory之下，可触发。修改策略为非noeviction可达到快速回收的效果，但会导致数据丢失和短暂的阻塞问题。<br>
## 3.内存优化
  * 建议Key为数字或者长度小于等于39字节的字符串
  * 建议控制K-V的长度，当V为json/xml格式时可以尝试压缩工具处理。推荐（Snappy、GZIP）
  * 共享对象池，对象池为（0-9999）整数。开启maxmemory和LRU淘汰策略后对象池无效。
  * 减少字符串的append/setrange操作，直接使用set。降低预分配带来的内存浪费和内存碎片化
  * 当V为json可转成hash类型，设置hash-max-ziplistvalue，可节省内存。（时间换空间）
  * 使用ziplist压缩编码优化hash、list等结构，注重效率和空间的平衡。
  * 使用intset编码优化整数集合。
  * 使用ziplist编码的hash结构降低小对象链规模。


