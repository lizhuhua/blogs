
# 内存消耗
## 1.Redis内存统计
> 
  命令：
> > 
    info memory 表示显示内存相关信息
> 
  结果选项：
> > 
    used_memory : Redis分配的总内存（字节）
    used_memory_rss : Redis进程占用操作系统的内存（字节）
    used_memory_peak : Redis的内存消耗峰值（字节）
    used_memory_lua : Lua引擎所消耗的内存大小
    used_fragmentation_ratio : 内存碎片比率  used_memory_rss/used_memory  
    mem_allocator : Redis使用的内存分配器。在编译时指定，可以是 libc 、jemalloc或者tcmalloc，默认是jemalloc。
>
  重点：
> >
    当内存碎片比率>1，说明used_memory_rss相比used_memory多的部分没有用于数据存储，而是被碎片所消耗，碎片率较高。
    当内存碎片比率<1，这种情况一般是把Redis内存交换到硬盘导致。由于硬盘速度<<内存速度，Redis性能会变差，甚至僵死。
    一般为1.03左右
## 2.内存消耗划分
自身内存、对象内存、缓冲内存（used_memory）
内存碎片（used_memory_rss，used_memory）
> 
对象内存：
> > 
    存储用户所有数据。K_V形式保存，K为字符串（避免使用过长），V包含字符串、列表、哈希、集合、有序集合
> 
缓冲内存：（客户端缓冲、复制积压缓冲区、AOF缓冲）
> > 
    客户端缓冲：所有接入到Redis服务器TCP连接的输入输出缓冲，输入缓冲最大空间为1G,超过将断开。输出通过参数client-output-buffer-limit控制
    复制积压缓冲区：为实现部分复制功能提供的一个可重用固定大小缓冲区，通过repl-backlog-size参数控制，可设置较大，所有节点共享此空间
    AOF缓冲区：用于重写时缓存最近的写入命令，通常较小
> 
内存碎片：
> > 
    数据存入内存分配器分配的内存块时，内存块多余无法存储其他数据的区域叫做内存碎片。
    碎片率较高解决方案：
    数据对齐：数据尽量采用数字类型或者固定长度字符串
    安全重启：启动节点内存水片会重新整理，可以利用高可用框架（Sentinel/Cluster）,将碎片率过高的主节点转移为从节点，进行安全重启
## 3.子进程内存消耗
主要执行AOF/RDB重写时创建子进程内存消耗。当Redis执行fork操作时子进程内存占用量对外表现与父进程相同。父进程处理写请求时会复制一份副本完成写操作，子进程读取fork时父进程的内存快照。<br>
>
内存消耗较高，解决方案：
> >
    要设置sysctl vm.overcommit_memory=1允许内核可以分配所有的物理内存，防止Redis进程执行fork时因系统剩余内存不足而失败。
    排查当前系统是否支持并开启THP，如果开启建议关闭，防止copy-onwrite期间内存过度消耗。
