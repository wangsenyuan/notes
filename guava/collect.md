* 1\. ImmutableList
  * 1.1\. subList(from, to)
       * 1.1.1\. 特殊处理size为0，1的情况；
       * 1.1.2\. 返回的是原List的一个view，并不真正持有数据;
* 2\. Multiset
  * 2.1\. TreeMultiset 内部使用了AVL Tree来保存数据
          java.util.TreeSet 内部使用了TreeMap，TreeMap使用了红黑树实现
  * 2.2\. ConcurrentHashMultiset 利用了ConcurrentMap 和 AtomicInteger
* 3\. EnumMap
  * 3.1\. Key的类型必须是enum
  * 3.2\. 使用keys数组保存key， 不用hash索引，而是使用ordinal()作为key的索引；在构造EnumMap的时候，必须提供key的类型；keys将会被enum的值预先填充好
  * 3.3\. value存在一个vals数组中，和keys使用相同的下标；
  * 3.4\. put/remove不会改变keys，而只会将vals对应的值改变；value不能为空；
