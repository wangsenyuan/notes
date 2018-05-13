1. New, Ready, Running, Blocked, Terminated
2. 状态迁移图
  ![lifecyle](./threadstates.jpg?raw=true "Lifecyle")
3. 当调用thread.start()时，线程进入Ready状态，等待分配CPU资源
   当分配了CPU资源，开始执行run方法时，线程处于Running状态；
   当等待IO资源，或者获取锁（等待），或者调用Thread.sleep(), Object.wait方法时，线程进入Blocked；
   当等待的条件满足时，或者获得了锁，sleep时间到，其他线程调用notify时，线程重新进入Ready状态；
   只有ready状态可以进入running；
   当任务完成，从run方法退出，线程进入Terminated