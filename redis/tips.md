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
* 2\. redis 数据类型
  redis支持5种数据类型，字符串，哈希字典，列表，集合，有序集合；

* 3\. redis transaction
  MULTI, EXEC, DISCARD and WATCH are the foundation of transactions in Redis. They allow the execution of a group of commands in a single step, with two important guarantees:
  * 3.1\. 
    All the commands in a transaction are serialized and executed sequentially. It can never    
    happen that a request issued by another client is served in the middle of the execution of a Redis transaction. This guarantees that the commands are executed as a single isolated operation.
  * 3.2\.
    Either all of the commands or none are processed, so a Redis transaction is also atomic. The EXEC command triggers the execution of all the commands in the transaction, so if a client loses the connection to the server in the context of a transaction before calling the MULTI command none of the operations are performed, instead if the EXEC command is called, all the operations are performed. When using the append-only file Redis makes sure to use a single write(2) syscall to write the transaction on disk. However if the Redis server crashes or is killed by the system administrator in some hard way it is possible that only a partial number of operations are registered. Redis will detect this condition at restart, and will exit with an error. Using the redis-check-aof tool it is possible to fix the append only file that will remove the partial transaction so that the server can start again.
  * 3.3\.
    Starting with version 2.2, Redis allows for an extra guarantee to the above two, in the form of optimistic locking in a way very similar to a check-and-set (CAS) operation
  * 3.4\.
    During a transaction it is possible to encounter two kind of command errors:
    * 3.4.1\.
      A command may fail to be queued, so there may be an error before EXEC is called. For instance the command may be syntactically wrong (wrong number of arguments, wrong command name, ...), or there may be some critical condition like an out of memory condition (if the server is configured to have a memory limit using the maxmemory directive).
    * 3.4.2\.
      A command may fail after EXEC is called, for instance since we performed an operation against a key with the wrong value (like calling a list operation against a string value).
    Clients used to sense the first kind of errors, happening before the EXEC call, by checking the return value of the queued command: if the command replies with QUEUED it was queued correctly, otherwise Redis returns an error. If there is an error while queueing a command, most clients will abort the transaction discarding it.
    However starting with Redis 2.6.5, the server will remember that there was an error during the accumulation of commands, and will refuse to execute the transaction returning also an error during EXEC, and discarding the transaction automatically.

    如果是在Multi阶段产生了ERROR，即使这个ERROR被client忽略了，继续提交其他的command，并执行了EXEC，EXEC仍然会失败掉；
    
    Errors happening after EXEC instead are not handled in a special way: all the other commands will be executed even if some command fails during the transaction.

    如果是在EXEC阶段，某条任务的失败，不会影响其他任务的执行；
  * 3.5\.
    WATCH is used to provide a check-and-set (CAS) behavior to Redis transactions.
    WATCHed keys are monitored in order to detect changes against them. If at least one watched key is modified before the EXEC command, the whole transaction aborts, and EXEC returns a Null reply to notify that the transaction failed.
    ```

    WATCH mykey
    val = GET mykey
    val = val + 1
    MULTI
    SET mykey $val
    EXEC
    ```



  