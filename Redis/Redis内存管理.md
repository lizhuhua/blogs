# 内存管理

## 1.内存上限控制
  Redis使用maxmemory参数限制Redis实际使用的内存（used_memory）<br>
  修改maxmemory命令: `config set maxmemory *GB`<br>
  修改maxmemory可以达到扩容的目的，但如果内存超出物理内存时需要采用在线`迁移数据`或者`复制切换服务器`来扩容。
  
## 2.内存回收策略
  内存使用达到maxmemory上限时触发内存溢出控制策略<br>
  * 删除过期键对象
  
