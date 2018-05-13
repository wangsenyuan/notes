1. New, Ready, Running, Blocked, Terminated
2. 状态迁移图
  ![lifecyle](./threadstates.jpg?raw=true "Lifecyle")
3. 当调用thread.start()时，线程进入Ready状态，等待分配CPU资源
   当分配了CPU资源，开始执行run方法时，线程处于Running状态；
   当等待IO资源，或者获取锁（等待），或者调用Thread.sleep(), Object.wait方法时，线程进入Blocked；
   当等待的条件满足时，或者获得了锁，sleep时间到，其他线程调用notify时，线程重新进入Ready状态；
   只有ready状态可以进入running；
   当任务完成，从run方法退出，线程进入Terminated

4. Java threads v.s OS threads
   Java threads可以分为两个层次，第一个是定义，另外一个运行；只有运行的java threads和OS threads有关；
   运行时的Java Threads和OS Threads如何映射，主要看JVM得实现；大体有下面两种：
   1. JVM负责调度
   ```
   For Green threads, the JVM will choose which thread to run whenever the operating system gives CPU time to the JVM.
   ```
   2. JVM将threads映射为native threads, 由OS调用
   ```
    If the JVM supports native threads (common these days), they are mapped onto real threads as offered by the operating system. They will mostly run in parallel until one of multiple conditions:

      JVM needs to perform garbage collection;
      Threads reach synchronized methods (not necessarily all threads);
      Threads reach barriers in the JVM, and need to wait for the JVM servicing their request (for example, memory allocation).
   ```
   