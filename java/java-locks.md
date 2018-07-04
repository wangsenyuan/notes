1. 公平锁和非公平锁
   公平锁是指按照申请锁的顺序来获取锁；非公平锁即不保证先申请先获得，优点是可以提供吞吐量；
   ReentrantLock默认是非公平的，可以通过构造函数传入true，使其公平；
   syncrhonized是非公平锁
2. 互斥锁和读写锁
   互斥锁也是独享锁，指一旦锁被获取以后，其他线程必须等待。
   读写锁，一般读写，写写互斥，读读不互斥。在读比较多的情况下， 提供并发效率
   ReentrantLock是互斥锁，ReadWriteLock是读写锁；
3. ReentrantLock也可以通过Condition实现临时的共享锁；如下的代码所示：当items满的时候，put将会await; 直到其他的线程notify；这里的关键是在condition上面调用await, 该线程将会释放锁。以使其他的线程可以获得锁，进入critical section;

```

class BoundedBuffer {
   final Lock lock = new ReentrantLock();
   final Condition notFull  = lock.newCondition(); 
   final Condition notEmpty = lock.newCondition(); 

   final Object[] items = new Object[100];
   int putptr, takeptr, count;

   public void put(Object x) throws InterruptedException {
     lock.lock();
     try {
       while (count == items.length)
         notFull.await();
       items[putptr] = x;
       if (++putptr == items.length) putptr = 0;
       ++count;
       notEmpty.signal();
     } finally {
       lock.unlock();
     }
   }

   public Object take() throws InterruptedException {
     lock.lock();
     try {
       while (count == 0)
         notEmpty.await();
       Object x = items[takeptr];
       if (++takeptr == items.length) takeptr = 0;
       --count;
       notFull.signal();
       return x;
     } finally {
       lock.unlock();
     }
   }
 }

```

```

Conditions (also known as condition queues or condition variables) provide a means for one thread to suspend execution (to "wait") until notified by another thread that some state condition may now be true. Because access to this shared state information occurs in different threads, it must be protected, so a lock of some form is associated with the condition. The key property that waiting for a condition provides is that it atomically releases the associated lock and suspends the current thread, just like Object.wait.

```

4. 悲观锁和乐观锁
   悲观锁适合写比较多的情况，主要通过各种lock，sychronized等实现
   乐观锁适合读比较多的情况，通过Compare And Swap实现，主要是各种Atomic类

5. Semaphone
   信号量可被看做分配了多把钥匙的锁；比如某个酒吧，最多可以服务100人，那么可以在门口设置一个守卫，依次（或者邀请）放入(semaphone.accquire)，但一旦人满，则后面的人只能等待；直到里面的人出来(semaphone.relase)
   Semaphone也可以设置为公平或不公平，默认不公平；通过提供permits，可以设置上限；可以设置为负数，当为负数时，必须先调用release。

```
    A counting semaphore. Conceptually, a semaphore maintains a set of permits. Each acquire() blocks if necessary until a permit is available, and then takes it. Each release() adds a permit, potentially releasing a blocking acquirer. However, no actual permit objects are used; the Semaphore just keeps a count of the number available and acts accordingly.

```