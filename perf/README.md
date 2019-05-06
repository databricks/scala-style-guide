
1. [Performance](#perf)
    * [Microbenchmarks](#perf-microbenchmarks)
    * [Traversal and zipWithIndex](#perf-whileloops)
    * [Option and null](#perf-option)
    * [Scala Collection Library](#perf-collection)
    * [private[this]](#perf-private)

## <a name='perf'>Performance</a>

For the vast majority of the code you write, performance should not be a concern. However, for performance sensitive code, here are some tips:

### <a name='perf-microbenchmarks'>Microbenchmarks</a>

It is ridiculously hard to write a good microbenchmark because the Scala compiler and the JVM JIT compiler do a lot of magic to the code. More often than not, your microbenchmark code is not measuring the thing you want to measure.

Use [jmh](http://openjdk.java.net/projects/code-tools/jmh/) if you are writing microbenchmark code. Make sure you read through [all the sample microbenchmarks](http://hg.openjdk.java.net/code-tools/jmh/file/tip/jmh-samples/src/main/java/org/openjdk/jmh/samples/) so you understand the effect of deadcode elimination, constant folding, and loop unrolling on microbenchmarks.


### <a name='perf-whileloops'>Traversal and zipWithIndex</a>

Use `while` loops instead of `for` loops or functional transformations (e.g. `map`, `foreach`). For loops and functional transformations are very slow (due to virtual function calls and boxing).
```scala

val arr = // array of ints
// zero out even positions
val newArr = list.zipWithIndex.map { case (elem, i) =>
  if (i % 2 == 0) 0 else elem
}

// This is a high performance version of the above
val newArr = new Array[Int](arr.length)
var i = 0
val len = newArr.length
while (i < len) {
  newArr(i) = if (i % 2 == 0) 0 else arr(i)
  i += 1
}
```

### <a name='perf-option'>Option and null</a>

For performance sensitive code, prefer `null` over `Option`, in order to avoid virtual method calls and boxing. Label the nullable fields clearly with Nullable.
```scala
class Foo {
  @javax.annotation.Nullable
  private[this] var nullableField: Bar = _
}
```

### <a name='perf-collection'>Scala Collection Library</a>

For performance sensitive code, prefer Java collection library over Scala ones, since the Scala collection library often is slower than Java's.

### <a name='perf-private'>private[this]</a>

For performance sensitive code, prefer `private[this]` over `private`. `private[this]` generates a field, rather than creating an accessor method. In our experience, the JVM JIT compiler cannot always inline `private` field accessor methods, and thus it is safer to use `private[this]` to ensure no virtual method call for accessing a field.
```scala
class MyClass {
  private val field1 = ...
  private[this] val field2 = ...

  def perfSensitiveMethod(): Unit = {
    var i = 0
    while (i < 1000000) {
      field1  // This might invoke a virtual method call
      field2  // This is just a field access
      i += 1
    }
  }
}
```


