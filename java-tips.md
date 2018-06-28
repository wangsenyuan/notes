* 1\. LinkedHashMap
  * 1. LinkedHashMap，使用一个双向列表将Entries连接起来，提供可预期的迭代顺序；默认顺序是插入顺序；
  * 2. 以下构造函数的accessOrder为true时，Entries将会按照accessOrder连接起来，从Least Recent Access到Most Recent Acess; 所以，用于实现LRU cache;

  ```
  LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder)

  ```

  ```

  A special constructor is provided to create a linked hash map whose order of iteration is the order in which its entries were last accessed, from least-recently accessed to most-recently (access-order). This kind of map is well-suited to building LRU caches. Invoking the put, putIfAbsent, get, getOrDefault, compute, computeIfAbsent, computeIfPresent, or merge methods results in an access to the corresponding entry (assuming it exists after the invocation completes). The replace methods only result in an access of the entry if the value is replaced. The putAll method generates one entry access for each mapping in the specified map, in the order that key-value mappings are provided by the specified map's entry set iterator. No other methods generate entry accesses. In particular, operations on collection-views do not affect the order of iteration of the backing map.

  The removeEldestEntry(Map.Entry) method may be overridden to impose a policy for removing stale mappings automatically when new mappings are added to the map.

  ```

   * 3. Iteration over the collection-views of a LinkedHashMap requires time proportional to the size of the map, regardless of its capacity. Iteration over a HashMap is likely to be more expensive, requiring time proportional to its capacity.

* 2. HashMap
  * 1. 解决Hash Collision的方法是用链表，即将相同Hash的放在一个bucket里，通过链表连接；
  * 2. 当数据足够多的时候，超过了loadFactor（默认是0.75)时，会resize, 原始大小的2倍, 原有的entry会重新插入新的table;

* 3. ConcurrentHashMap
  * 1. 不允许null做为Key和Value
    如下为containsKey的实现, 如果和containsKey实现一样， 判断是否能找到保存key的Node, 那么就需要加锁；使用判断value是否为空，可以不加锁；这个应该是value不能为空的原因；

    ```Java
    public boolean containsKey(Object key) {
        return get(key) != null;
    }
    ```
    ```Java
    public boolean containsKey(Object key) {
        return getNode(hash(key), key) != null;
    }

    ```
  * 2. get时，不加锁; put时，新值将加在链表的末端, 在链表的头节点加锁（如果存在);
  * 3. 通过key的hash获取，在table中定位，通过access volatile的方式，使得一个线程对该entry的变化，可以最快被其他线程看到；

  ```Java
    @SuppressWarnings("unchecked")
    static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
        return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
    }

    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
    }

    static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
        U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
    }

  ```

  当size为2的整数倍时，计算bucket的下标，可以用下面的位运算快速进行:

  ```Java

  (n - 1) & h

  ```

* 4. AtomicReference
  * 1. AtomicReference可用于在两个线程间无锁交换数据
  ```Java


    public static void main(String[] args) throws InterruptedException {
        AtomicReference<Integer> ref = new AtomicReference<>();
        Integer val = new Integer(0);
        ref.set(val);

        Consumer consumer = new Consumer(ref);

        new Thread(consumer).start();

        Thread.sleep(2000);

        ref.compareAndSet(val, new Integer(10));
        Thread.sleep(100);
    }


    static class Consumer implements Runnable {
        private final AtomicReference<Integer> ref;
        private int last;

        Consumer(AtomicReference<Integer> ref) {
            this.ref = ref;
        }

        @Override
        public void run() {
            int prev = last;
            while ((last = ref.get()) == prev) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("read " + last + "from ref");
            }
            System.out.println("value updated to " + last);
        }
    }

  ```
  
* 5. Vector和ArrayList的实现大体是类似的，内部都是一个数组，当capacity不足时，增大数组；主要的区别时Vector的操作上面加了sychronized锁，是线程安全的。