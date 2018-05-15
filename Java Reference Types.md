Java有四种引用类型
1. 强引用：
  通过new操作符获得的，都是强引用；当持有该引用的对象存活的情况下， 被引用的对象也会存活；
2. SoftReference
  通过以下方式获得的是软引用; 弱引用在内存充足的情况下不会被回收；只有当回收完Garbage的时候，内存仍然不足的情况下，弱引用对象会被回收掉；
  
  ```Java

  Object obj = new Object();
  SoftReference<Object> sf = new SoftReference<Object>(obj);
  
  ```

3. WeakReference
   通过以下方式获得弱引用；WeakReference referent只能存活到下次GC前。

  ```Java

  Object obj = new Object();
  WeakReference<Object> sf = new WeakReference<Object>(obj);
  
  ```

4. PhantomReference
   PhantomReference创建后，就无法通过get()获得referent；使用它的一个作用是和ReferenceQueue一起使用，以在对象在被GC时，获得通知

5. ReferenceQueue
   在构建Soft,Weak和PhantomReference的时候可以传入一个ReferenceQueue, 当对象要被GC时，引用会被放入Queue，从而可以做一些操作;
  
6. 测试代码:

```Java


    public static void main(String[] args) {
        testSoftReference();
        testWeakReference();
        testPhantomReference();
    }

    public static void testSoftReference() {
        System.out.println("Test SoftReference");
        Object obj = new Object();
        ReferenceQueue<Object> que = new ReferenceQueue<>();

        SoftReference<Object> ref = new SoftReference<>(obj, que);

        // only soft reference left
        obj = null;

        System.out.println("before GC is reference holding referent ? " + (ref.get() != null));
        System.out.println("start GC");

        Runtime.getRuntime().gc();

        System.out.println("is reference still holding referent ? " + (ref.get() != null));

        System.out.println("is reference queue has object appended? " + (que.poll() != null));
    }

    public static void testWeakReference() {
        System.out.println("Test WeakReference");
        Object obj = new Object();
        ReferenceQueue<Object> que = new ReferenceQueue<>();

        WeakReference<Object> ref = new WeakReference<>(obj, que);

        obj = null;

        System.out.println("Before GC, is weak reference holding referent? " + (ref.get() != null));
        System.out.println("start GC");

        Runtime.getRuntime().gc();

        System.out.println("is reference still holding referent ? " + (ref.get() != null));

        System.out.println("is reference queue has object appended? " + (que.poll() != null));
    }

    public static void testPhantomReference() {
        System.out.println("Test PhantomReference");
        Object obj = new Object();
        ReferenceQueue<Object> que = new ReferenceQueue<>();

        PhantomReference<Object> ref = new PhantomReference<>(obj, que);

        obj = null;

        System.out.println("Before GC, is phantom reference holding referent? " + (ref.get() != null));
        System.out.println("start GC");

        Runtime.getRuntime().gc();

        System.out.println("is reference still holding referent ? " + (ref.get() != null));

        System.out.println("is reference queue has object appended? " + (que.poll() != null));
    }

```

测试输出:

```

Test SoftReference
before GC is reference holding referent ? true
start GC
is reference still holding referent ? true
is reference queue has object appended? false
Test WeakReference
Before GC, is weak reference holding referent? true
start GC
is reference still holding referent ? false
is reference queue has object appended? true
Test PhantomReference
Before GC, is phantom reference holding referent? false
start GC
is reference still holding referent ? false
is reference queue has object appended? true

```