* 1\. redis memeory usage
  * 1.1\. 可以通过 info memory 查看内存使用情况
  ```
used_memory:2939472
used_memory_human:2.80M
used_memory_rss:733184
used_memory_rss_human:716.00K
used_memory_peak:3005424
used_memory_peak_human:2.87M
used_memory_peak_perc:97.81%
used_memory_overhead:1145970
used_memory_startup:980752
used_memory_dataset:1793502
used_memory_dataset_perc:91.57%
total_system_memory:8589934592
total_system_memory_human:8.00G
used_memory_lua:37888
used_memory_lua_human:37.00K
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
mem_fragmentation_ratio:0.25
mem_allocator:libc
active_defrag_running:0
lazyfree_pending_objects:0

  ```

  * 1.2\. used_memory redis内存分配器分配的内存总量，包括swap空间；
  * 1.3\. used_memory_rss 操作系统分配给redis的内存总量; 除了分配器分配的内存之外，used_memory_rss还包括进程运行本身需要的内存、内存碎片等，但是不包括swap;
  * 1.4\. mem_fragmentation_ratio used_memory_rss/used_memory;
    mem_fragmentation_ratio一般大于1，且该值越大，内存碎片比例越大。mem_fragmentation_ratio<1，说明Redis使用了虚拟内存，由于虚拟内存的媒介是磁盘，比内存速度要慢很多，当这种情况出现时，应该及时排查，如果内存不足应该及时处理，如增加Redis节点、增加Redis服务器的内存、优化应用等
    