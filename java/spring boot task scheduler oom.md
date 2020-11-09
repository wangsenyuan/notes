### 背景

某天早上到公司以后，就收到了很多error mail，都是调用service x的timeout。客户服务群里面也反馈有很多不可用的情况。意识到情况不妙。通过grafana查看该服务的情况，最主要的发现是jvm平均GC时间都有500MS了。  
登录的k8s集群去查看服务实例的情况，在当时的情况下，k8s通过liveness检测，自动重启了该服务的两个实例； 
然后服务就马上恢复了。  
上面的表现，应该是OOM了。grafana JVM的统计也显示那段时间内出现明显的断层。

### 查找问题

服务中使用了多个guava loading cache。但是这些基本可以被排除，因为它们的数量都很有限，全部加载也不会超过1W个。
然后想到系统里面有使用一个BlockingQueue。它的作用是接收到请求后，先缓存起来，然后另外一端有个线程再消费处理。如果输入端很快，而消费端很慢，确实会造成积压。但是通过查看log，基本也排除了它的嫌疑，从log里面看，消费处理的时间基本在10ms内就完成了。
然后就想最近有啥变化吗？（之前没有发生过这种情况） 
想到是最近做了一个优化，为了提供消息处理的速度，把之前用来audit所有请求的一个操作，使用TaskScheduler异步去记录。事实证明确实是这个改动造成的。
1. TaskScheduler 是把任务包装多层后放到了一个BlockingQueue里去。
2. 然后有可用的线程时，调度执行。但是spring-boot里面的spring.task.scheduling.pool.size 默认是1。前面使用blockingqueue的时候，也用的是同一个taskscheduler，已经把这1个thread给占用了。所以所有的audit任务都没有被调度，只是被单纯的放到了一个queue里。日积月累就OOM了。这也能解释为啥两个实例同时OOM了。
   
### 总结
1. 使用TaskScheudler的时候记得配置spring.task.scheduling.pool.size
2. 平时要多关注服务实例情况。