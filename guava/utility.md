* 1. Don't use NULL;
    * 1.1 null 会 造成NPE
    * 1.2 null 经常会带来歧义, 比如某个map.get(key) => null, 无法直接判断是该key不存在，还是值就是null；
    * 1.3 可以用Optional代替null

* 2. Orderings
    * 2.1 抽象类定义方法，具体的类实现细节
    * 2.2 static工厂方法返回按照不同方式排序的子类，比如natural(), from(Comparator)等；
    * 2.3 实例方法，使用Wrapper模式，wrap原有的排序算法，实现链式，复杂的排序方法；如reverse, nullsFirst, nullsLast, compond等

* 3. java.util.Arrays.deepToString(Object[])

* 4. ComparisonChain

  ```
    public int compareTo(Foo that) {
      return ComparisonChain.start()
         .compare(this.aString, that.aString)
         .compare(this.anInt, that.anInt)
         .compare(this.anEnum, that.anEnum, Ordering.natural().nullsLast())
         .result();
    }

  ```
  按照compare调用的顺序，依次比较，直到找到第一个非0的结果。
  实现方式，非常clever; 大体是如下图所示的一个状态图：
  
  ![comparisonchain](../images/ComparisonChain.jpg?raw=true "ComparisonChain")

  比较结果不为0之前， 将一直处于ACTIVE状态；一旦到达LESS或GREATER状态，结果将不再改变:

  ```Java

  ComparisonChain classify(int result) {
          return (result < 0) ? LESS : (result > 0) ? GREATER : ACTIVE;
  }

  ```

* 5. Class.isInstance v.s instanceOf
    * 1. from java doc: This method is the dynamic equivalent of the Java language instanceof operator.
    * 2. 当要检查的类确定已知时，两个都可以用，但用instanceOf更直观
  ```Java
  if (foo instanceof ThisClassIKnow)
  ```
  ```Java
  if (ThisClassIKnow.class.instanceOf(foo))
  ```
    * 3. 当要检查的类不确定时，只能用isInstance方法:
  ```Java
  public boolean areObjectsAssignable(Object left, Object right) {
    return left.getClass().isInstance(right);
  }
  ```

* 6. Throwables.getRootCause(throwable) 考虑了stacktrace出现环的情况。never thought about that before, clever

```Java
  public static Throwable getRootCause(Throwable throwable) {
    // Keep a second pointer that slowly walks the causal chain. If the fast pointer ever catches
    // the slower pointer, then there's a loop.
    Throwable slowPointer = throwable;
    boolean advanceSlowPointer = false;

    Throwable cause;
    while ((cause = throwable.getCause()) != null) {
      throwable = cause;

      if (throwable == slowPointer) {
        throw new IllegalArgumentException("Loop in causal chain detected.", throwable);
      }
      if (advanceSlowPointer) {
        slowPointer = slowPointer.getCause();
      }
      advanceSlowPointer = !advanceSlowPointer; // only advance every other iteration
    }
    return throwable;
  }

```