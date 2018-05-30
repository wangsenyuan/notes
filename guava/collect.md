* 1\. ImmutableList
  * 1.1\. subList(from, to)
       * 1.1.1\. 特殊处理size为0，1的情况；
       * 1.1.2\. 返回的是原List的一个view，并不真正持有数据;
* 2\. Multiset
  * 2.1\. TreeMultiset 内部使用了AVL Tree来保存数据
          java.util.TreeSet 内部使用了TreeMap，TreeMap使用了红黑树实现
  * 2.2\. ConcurrentHashMultiset 利用了ConcurrentMap 和 AtomicInteger