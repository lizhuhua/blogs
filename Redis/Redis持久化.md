# Redis持久化
## RDB
RDB持久化是把进程数据生成快照保存到硬盘的过程。 <br>
触发机制：<br>
* 手动触发：
  save命令: 阻塞当前Redis服务器，知道RDB过程完成位置<br>
  bgsave命令：优化save命令，Redis进程执行fork操作创建子进程，RDB持久化过程由子进程负责。阻塞只发生在fork阶段<br>
* 自动触发: 
  使用save相关配置 `save m n`。m秒内数据集存在n次修改<br>
  从节点执行全量复制操作，直接点自动执行bgsave<br>
  执行debug reload命令重新加载redis时,
  执行shutdown，如果未启动AOF，则自动执行bgsave
## AOF
AOF持久化是以独立日志方式记录每次写命令。重启时重新执行AOF
* 命令写入
  写入内容直接是文本协议格式，具有很好的兼容性、可读性。<br>
  写AOF文件命令追加到aof_buf,而不是直接追加到硬盘。<br>
* 文件同步
  AOF提供多种同步策略，有参数appendfsync控制<br>
  1.always 命令写入aof_buf后调用系统fsync操作同步到AOF文件，fsync完成后线程返回<br>
  2.everysec 命令写入aof_buf后调用系统write操作，write完成后线程返回。fsync同步文件操作由专门线程每秒调用一次<br>
  3.no 命令写入aof_buf后调用系统write操作，不对AOF文件做同步，同步硬盘操作由系统负责。<br>
  建议：<br>
  使用everysec（默认配置）兼顾性能和数据安全性，但当系统宕机理论会出现丢失1秒数据。no同步AOF周期不可控，数据安全性没法保证。always每次写入都同步AOF，磁盘写入烧性能。 <br>
* 重写机制
  AOF重写是为了解决命令不断写入AOF,文件越来越大问题。引入重写机制压缩文件体积，减少内存消耗<br>
  1.超时数据不再写入文件<br>
  2.只保存最终数据的写入命令到新的AOF<br>
  3.多条写命令可以合并一个<br>
  触发：<br>
  手动触发： bgrewriteaof命令<br>
  自动触发： 根据auto-aof-rewrite-min-size（AOF重写时文件最小体积，默认64MB）和auto-aof-rewrite-percentage参
数确定自动触发时机。<br>
* 重启加载
AOF和RDB文件都可以用于服务器重启时的数据恢复,AOF优先级高于RDB<br>
* 文件校验
错误AOF无法启动<br>
## 问题定位与优化
1.fork操作，物理机支持；放宽AOF自动触发<br>
2.子进程（AOF/RDB重写）开销监控和优化：CPU/内存/硬盘<br>
3.AOF追加阻塞 对比上一次fsync超过2秒阻塞，最多可能丢失2秒数据。主要优化系统硬盘负载<br>
4.单机多实例，建议做隔离控制，避免CPU和IO资源竞争<br>
