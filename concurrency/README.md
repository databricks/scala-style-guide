1. [Concurrency](#concurrency)
    * [Scala concurrent.Map](#concurrency-scala-collection)
    * [Explicit Synchronization vs Concurrent Collections](#concurrency-sync-vs-map)
    * [Explicit Synchronization vs Atomic Variables vs @volatile](#concurrency-sync-vs-atomic)
    * [Private Fields](#concurrency-private-this)
    * [Isolation](#concurrency-isolation)

## <a name='concurrency'>Concurrency</a>

### <a name='concurrency-scala-collection'>Scala concurrent.Map</a>

__Prefer `java.util.concurrent.ConcurrentHashMap` over `scala.collection.concurrent.Map`__. In particular the `getOrElseUpdate` method in `scala.collection.concurrent.Map` is not atomic (fixed in Scala 2.11.6, [SI-7943](https://issues.scala-lang.org/browse/SI-7943)). Since all the projects we work on require cross-building for both Scala 2.10 and Scala 2.11, `scala.collection.concurrent.Map` should be avoided.


### <a name='concurrency-sync-vs-map'>Explicit Synchronization vs Concurrent Collections</a>

There are 3 recommended ways to make concurrent accesses to shared states safe. __Do NOT mix them__ because that could make the program very hard to reason about and lead to deadlocks.

1. `java.util.concurrent.ConcurrentHashMap`: Use when all states are captured in a map, and high degree of contention is expected.
  ```scala
  private[this] val map = new java.util.concurrent.ConcurrentHashMap[String, String]
  ```

2. `java.util.Collections.synchronizedMap`: Use when all states are captured in a map, and contention is not expected but you still want to make code safe. In case of no contention, the JVM JIT compiler is able to remove the synchronization overhead via biased locking.
  ```scala
  private[this] val map = java.util.Collections.synchronizedMap(new java.util.HashMap[String, String])
  ```

3. Explicit synchronization by synchronizing all critical sections: can used to guard multiple variables. Similar to 2, the JVM JIT compiler can remove the synchronization overhead via biased locking.
  ```scala
  class Manager {
    private[this] var count = 0
    private[this] val map = new java.util.HashMap[String, String]
    def update(key: String, value: String): Unit = synchronized {
      map.put(key, value)
      count += 1
    }
    def getCount: Int = synchronized { count }
  }
  ```

Note that for case 1 and case 2, do not let views or iterators of the collections escape the protected area. This can happen in non-obvious ways, e.g. when returning `Map.keySet` or `Map.values`. If views or values are required to pass around, make a copy of the data.
  ```scala
  val map = java.util.Collections.synchronizedMap(new java.util.HashMap[String, String])

  // This is broken!
  def values: Iterable[String] = map.values

  // Instead, copy the elements
  def values: Iterable[String] = map.synchronized { Seq(map.values: _*) }
  ```

### <a name='concurrency-sync-vs-atomic'>Explicit Synchronization vs Atomic Variables vs @volatile</a>

The `java.util.concurrent.atomic` package provides primitives for lock-free access to primitive types, such as `AtomicBoolean`, `AtomicInteger`, and `AtomicReference`.

Always prefer Atomic variables over `@volatile`. They have a strict superset of the functionality and are more visible in code. Atomic variables are implemented using `@volatile` under the hood.

Prefer Atomic variables over explicit synchronization when: (1) all critical updates for an object are confined to a *single* variable and contention is expected. Atomic variables are lock-free and permit more efficient contention. Or (2) synchronization is clearly expressed as a `getAndSet` operation. For example:
  ```scala
  // good: clearly and efficiently express only-once execution of concurrent code
  val initialized = new AtomicBoolean(false)
  ...
  if (!initialized.getAndSet(true)) {
    ...
  }

  // poor: less clear what is guarded by synchronization, may unnecessarily synchronize
  val initialized = false
  ...
  var wasInitialized = false
  synchronized {
    wasInitialized = initialized
    initialized = true
  }
  if (!wasInitialized) {
    ...
  }
  ```

### <a name='concurrency-private-this'>Private Fields</a>

Note that `private` fields are still accessible by other instances of the same class, so protecting it with `this.synchronized` (or just `synchronized`) is not technically sufficient. Make the field `private[this]` instead.
```scala
// The following is still unsafe.
class Foo {
  private var count: Int = 0
  def inc(): Unit = synchronized { count += 1 }
}

// The following is safe.
class Foo {
  private[this] var count: Int = 0
  def inc(): Unit = synchronized { count += 1 }
}
```


### <a name='concurrency-isolation'>Isolation</a>

In general, concurrency and synchronization logic should be isolated and contained as much as possible. This effectively means:

- Avoid surfacing the internals of synchronization primitives in APIs, user-facing methods, and callbacks.
- For complex modules, create a small, inner module that captures the concurrency primitives.


