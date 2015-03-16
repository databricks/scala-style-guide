# Databricks Scala Guide

Spark has over 500 contributors, making it the largest open-source project in Big Data and the most active project written in Scala (to the best of our knowledge). This guide draws from our experience coaching and working with engineers contributing to Spark as well as our Databricks engineering team.

Code is __written once__ by its author, but __read and modified multiple times__ by lots of other engineers. As most bugs actually come from future modification of the code, we need to optimize our codebase for long-term, global readability and maintainability. The best way to achieve this is to write simple code.


## <a name='TOC'>Table of Contents</a>

  1. [Syntatic Style](#syntatic)
    - [Naming Convention](#naming)
    - [Line Length](#linelength)
    - [Rule of 30](#rule_of_30)
    - [Spacing and Indentation](#indent)
    - [Blank Lines (Vertical Whitespace)](#blanklines)
    - [Parentheses](#parentheses)
    - [Curly Braces](#curly)
    - [Long Literals](#long_literal)
    - [Documentation Style](#doc)
    - [Ordering within a Class](#ordering_class)
    - [Imports](#imports)
    - [Pattern Matching](#pattern-matching)
    - [Infix Methods](#infix)
  2. [Scala Language Features](#lang)
    - [apply Method](#apply_method)
    - [Destructuring Binds](#destruct_bind)
    - [Call by Name](#call_by_name)
    - [Multiple Parameter Lists](#multi-param-list)
    - [Symbolic Methods (Operator Overloading)](#symbolic_methods)
    - [Type Inference](#type_inference)
    - [Return Statements](#return)
    - [Recursion and Tail Recursion](#recursion)
    - [Implicits](#implicits)
    - [Exception Handling, i.e. Try vs try](#exception)
    - [Options](#option)
    - [Monadic Chaining](#chaining)
  3. [Concurrency](#concurrency)
    - [Scala ConcurrentMap](#concurrency-scala-collection)
    - [Explicit Synchronization vs Concurrent Collections](#concurrency-sync-vs-map)
    - [Explicit Synchronization vs Atomic Variables vs @volatile](#concurrency-sync-vs-atomic)
    - [Private Fields](#concurrency-private-this)
    - [Futures](#concurrency-futures)
  4. [Performance](#perf)
    - [Microbenchmarks](#perf-microbenchmarks)
    - [Traversal and zipWithIndex](#perf-whileloops)
    - [Option and null](#perf-option)
    - [Scala Collection Library](#perf-collection)
    - [private[this]](#perf-private)
  5. [Java Interoperability](#java)
    - [Java Features Missing from Scala](#java-missing-features)
    - [Traits and Abstract Classes](#java-traits)
    - [Type Aliases](#java-type-alias)
    - [Default Parameter Values](#java-default-param-values)
    - [Multiple Parameter Lists](#java-multi-param-list)
    - [Varargs](#java-varargs)
    - [Implicits](#java-implicits)
    - [Companion Objects, Static Methods and Fields](#java-companion-object)
  6. [Miscellaneous](#misc)
    - [Prefer nanoTime over currentTimeInMillis](#misc_currentTimeInMillis_vs_nanoTime)
    - [Prefer URI over URL](#misc_uri_url)


## <a name='syntatic'>Syntatic Style</a>

### <a name='naming'>Naming Convention</a>

We mostly follow Java's and Scala's standard naming convention. 

- Classes, traits, objects should follow Java class convension, i.e. CamelCase style with the first letter capitalized.
  ```scala
  class ClusterManager

  trait Expression
  ```

- Packages should follow Java package naming conventions, i.e. all-lowercase ASCII letters.
  ```scala
  package com.databricks.resourcemanager
  ```

- Methods/functions should be named in camelCase style.

- Constants should be all uppercase letters and be put in a companion object
  ```scala
  object Configuration {
    val DEFAULT_PORT = 10000
  }
  ```

- Enums should be CamelCase with first letter capitalized.

- Annotations should also follow Java convention, i.e. CamelCase with first letter capitalized. Note that this differs from Scala's official guide.
  ```scala
  final class MyAnnotation extends StaticAnnotation
  ```


### <a name='linelength'>Line Length</a>

- Limit lines to 100 characters.
- The only exceptions are import statements and URLs (although even for those, try to keep them under 100 chars).


### <a name='rule_of_30'>Rule of 30</a>

"If an element consists of more than 30 subelements, it is highly probable that there is a serious problem" - [Refactoring in Large Software Projects](http://www.amazon.com/Refactoring-Large-Software-Projects-Restructurings/dp/0470858923).

In general: 

- A method should contain less than 30 lines of code.
- A class should contain less than 30 methods.


### <a name='indent'>Spacing and Indentation</a>

- Use 2-space indentation in general.
  ```scala
  if (true) {
    println("Wow!")
  }
  ```

- For function declarations, use 4 space indentation for its parameters when they don't fit in a single line. Return types can be either on the same line as the last parameter, or put to next line with 2 space indent.
  ```scala
  def newAPIHadoopFile[K, V, F <: NewInputFormat[K, V]](
      path: String,
      fClass: Class[F],
      kClass: Class[K],
      vClass: Class[V],
      conf: Configuration = hadoopConfiguration): RDD[(K, V)] = {
    // function body
  }
  
  def newAPIHadoopFile[K, V, F <: NewInputFormat[K, V]](
      path: String,
      fClass: Class[F],
      kClass: Class[K],
      vClass: Class[V],
      conf: Configuration = hadoopConfiguration)
    : RDD[(K, V)] = {
    // function body
  }
  ```
  
- For classes whose header doesn't fit in a single line, put the extend on the next line with 2 space indent, and add a blank line after class header.
  ```scala
  class Foo(
      val param1: String,  // 4 space indent for parameters
      val param2: String
      val param3: Array[Byte])
    extends FooInterface  // 2 space here
    with Logging {
    
    def firstMethod(): Unit = { ... }  // blank line above
  }
  ```

- Do NOT use vertical alignment. They draw attention to the wrong parts of the code and make the aligned code harder to change in the future.
  ```scala
  // Don't align vertically
  val plus     = "+"
  val minus    = "-"
  val multiply = "*"
  
  // Do the following
  val plus = "+"
  val minus = "-"
  val multiply = "*"
  ```


### <a name='blanklines'>Blank Lines (Vertical Whitespace)</a>

- A single blank line appears:
  - Between consecutive members (or initializers) of a class: fields, constructors, methods, nested classes, static initializers, instance initializers.
    - Exception: A blank line between two consecutive fields (having no other code between them) is optional. Such blank lines are used as needed to create logical groupings of fields.
  - Within method bodies, as needed to create logical groupings of statements.
  - Optionally before the first member or after the last member of the class (neither encouraged nor discouraged).
- Use one or two blank line(s) to separate class definitions.
- Excessive number of blank lines are discouraged. 


### <a name='parentheses'>Parentheses</a>

- Functions should be declared with parentheses, unless they are accessors that have no side-effect (state mutation, I/O operations are considered side-effects).
- Callsite should follow function declaration, i.e. if a function is declared with parentheses, call with parentheses:
  ```scala
  class Job {
    // Wrong: killJob changes state. Should have ().
    def killJob: Unit

    // Correct:
    def killJob(): Unit
  }
  ```
  Note that this is not just syntatic. It can affect correctness when `apply` is defined in the return object:
  ```scala
  class Foo {
    def apply(): Int
  }
  
  class Bar {
    def foo: Foo
  }
  
  new Bar().foo  // This returns a Foo
  new Bar().foo()  // This returns an Int!
  ```


### <a name='curly'>Curly Braces</a>

Put curly braces even around one-line conditional or loop statements. The only exception is if you are using if/else as an one-line ternary operator that is also side-effect free.
```scala
// Correct:
if (true) {
  println("Wow!")
}
 
// Correct:
if (true) statement1 else statement2

// Correct:
try {
  foo()
} catch {
  ...
}

// Wrong:
if (true)
  println("Wow!")
  
// Wrong:
try foo() catch {
  ...
}
```


### <a name='long_literal'>Long Literals</a>

Suffix long literal values with uppercase `L`. It is often hard to differentiate lowercase `l` from `1`.
```scala
val longValue = 1234L  // Do this

val longValue = 1234l  // Do NOT do this
```


### <a name='doc'>Documentation Style</a>

Use Java docs style instead of Scala docs style.
```scala
/** This is a correct one-liner, short description. */
 
/**
 * This is correct multi-line JavaDoc comment. And
 * this is my second line, and if I keep typing, this would be
 * my third line.
 */
 
/** In Spark, we don't use the ScalaDoc style so this
  * is not correct.
  */
```


### <a name='ordering_class'>Ordering within a Class</a>

If a class is long and has many methods, group them logically into different sections, and use comment headers to organize them.
```scala
class DataFrame {

  ///////////////////////////////////////////////////////////////////////////
  // DataFrame operations
  ///////////////////////////////////////////////////////////////////////////
  
  ...
  
  ///////////////////////////////////////////////////////////////////////////
  // RDD operations
  ///////////////////////////////////////////////////////////////////////////
  
  ...
}
```


### <a name='imports'>Imports</a>

- __Do NOT use wildcard imports__, unless you are importing more than 6 entities. Wildcard imports make the code less robust to external changes.
- Always import packages using absolute paths (e.g. `scala.util.Random`) instead of relative ones (e.g. `util.Random`).
- In addition, sort imports in the following order:
  * `java.*` and `javax.*`
  * `scala.*`
  * Third-party libraries (`org.*`, `com.*`, etc)
  * Project classes (`com.databricks.*` or `org.apache.spark` if you are working on Spark)
- Within each group, imports should be sorted in alphabetic ordering.
- You can use IntelliJ's import organizer to handle this automatically, using the following config:
  ```scala
  import java.*
  import javax.*
  
  import scala.*
  
  import *
  
  import com.databricks.*  // or import org.apache.spark.* if you are working on Spark
  ```


### <a name='pattern-matching'>Pattern Matching</a>

- For functions whose entire body is a pattern match expression, put the match on the same line as the function declaration if possible to reduce one level of indentation.
  ```scala
  def test(msg: Message): Unit = msg match {
    case ...
  }
  ```

- When calling a function with a closure, if there is only one case, put the case on the same line as the function invocation.
  ```scala
  list.zipWithIndex { case (elem, i) =>
    // ...
  }
  ```
  If there are multiple cases, indent and wrap them.
  ```scala
  list.map {
    case a: Foo =>  ...
    case b: Bar =>  ...
  }
  ```
 

### <a name='infix'>Infix Methods</a>

__Do NOT use infix notation__ for methods that aren't symbolic methods (i.e. operator overloading).
```scala
// Correct
list.map(func)
string.contains("foo")

// Wrong
list map (func)
string contains "foo"

// But overloaded operators should be invoked in infix style
arrayBuffer += elem
```


## <a name='lang'>Scala Language Features</a>


### <a name='apply_method'>apply Method</a>

Avoid defining apply methods on classes. These methods tend to make the code less readable, especially for people less familiar with Scala. It is also harder for IDEs (or grep) to trace. In the worst case, it can also affect correctness of the code in surprising ways, as demonstrated in [Parentheses](#parentheses). It is however ok to define them in companion objects as factory methods. 


### <a name='destruct_bind'>Destructuring Binds</a>

Destructuring bind (sometimes called tuple extraction) is a convenient way to assign two variables in one expression.
```scala
val (a, b) = (1, 2)
```

However, do NOT use them in constructors, especially when `a` and `b` need to be marked transient. The Scala compiler generates an extra Tuple2 field that will not be transient for the above example.
```scala
class MyClass {
  // This will NOT work because the compiler generates a non-transient Tuple2 that points to both a and b
  @transient private val (a, b) = someFuncThatReturnsTuple2()
}
```


### <a name='call_by_name'>Call by Name</a>

__Do NOT use call by name__. Use `() => T` explicitly.

Background: Scala allows function parameters to be defined by-name, e.g.
```scala
def print(value: => Int): Unit = {
  println(value)
  println(value + 1)
}

var a = 0
def inc(): Int = {
  a + 1
  a
}

print(inc())
```
in the above code, `inc()` is passed into `print` as a closure and is only executed (twice) in the print method, rather than being passed in as a value `1`. The main problem with call-by-name is that the caller cannot diffentiate between call-by-name and call-by-value, and thus cannot know for sure whether the expression will be executed or not (or maybe worse, multiple times). This is esepcially dangerous for expressions that have side-effect.


### <a name='multi-param-list'>Multiple Parameter Lists</a>

__Do NOT use multiple parameter lists__. They complicate operator overloading, and can confuse programmers less familiar with Scala.


### <a name='symbolic_methods'>Symbolic Methods (Operator Overloading)</a>

__Do NOT use symbolic method names__, unless you are defining them for natural arithmetic operations (e.g. `+`, `-`, `*`, `/`). Under no other circumstances should they be used. Symbolic method names make it very hard to understand the intent of the functions. Consider the following two examples:
```scala
// symbolic method names are hard to understand
channel ! msg  
stream1 >>= stream2

// self-evident what is going on
channel.send(msg)
stream1.join(stream2)
```


### <a name='type_inference'>Type Inference</a>

Scala type inference, especially left-side type inference and closure inference, can make code more concise. That said, there are a few cases where explicit typing should be used:

- __Public methods should be explicitly typed__, otherwise the compiler's inferred type can often surprise you.
- __Implicit methods should be explicitly typed__, otherwise it can crash the Scala compiler with incremental compilation.
- __Variables, closures with non-obvious types should be explicitly typed__. A good litmus test is that explicit types should be used if a code reviewer cannot determine the type in 3 seconds.


### <a name='return'>Return Statements</a>

__Do NOT use return in closures__. `return` is turned into ``try/catch`` of ``scala.runtime.NonLocalReturnControl`` by the compiler. This can lead to unexpected behaviors. Consider the following example:
  ```scala
  def receive(rpc: WebSocketRPC): Option[Response] = {
    tableFut.waitForReady { table =>
      if (table.hasError) {
        return None // Do not do that!
      } else { ... }
    }
  }
  ```
  the `.waitForReady` function takes the anonymous closure `{ table => ... }` and passes it to a future which then passes it to a different thread. This closure eventually throws the `NonLocalReturnControl` exception that is captured __in a different thread__ . It has no effect on the poor function being executed here.
  
However, there are a few cases where `return` is preferred.

- Use `return` as a guard to simplify control flow
  ```scala
  def doSomething(obj: Any): Any = {
    if (obj eq null) {
      return null
    }
    // do something ...
  }
  ```

- Use `return` to terminate a loop early, rather than constructing status flags
  ```scala
  while (true) {
    if (cond) {
      return
    }
  }
  ```

### <a name='recursion'>Recursion and Tail Recursion</a>

__Do NOT use recursion__, unless the problem can be naturally framed recursively (e.g. graph traversal, tree traversal).

For functions that are meant to be tail recursive, apply `@tailrec` annotation to make sure the compiler can check it is tail recursive (you will be surprised how often seemingly tail recursive code is actually not tail recursive due to the use of closures and functional transformations.)

Most code is easier to reason about with a simple loop and explicit state machines. Expressing it with tail recursions (and accumulators) can make it more verbose and harder to understand.  For example, the following imperative code is more readable than the tail recursive version:

```scala
// Tail recursive version.
def max(data: Array[Int]): Int = {
  @tailrec
  def max0(data: Array[Int], pos: Int, max: Int): Int = {
    if (pos == data.length) {
      max
    } else {
      max0(data, pos + 1, if (data(pos) > max) data(pos) else max)
    }
  }
  max0(data, 0, Int.MinValue)
}

// Explicit loop version
def max(data: Array[Int]): Int = {
  var max = Int.MinValue
  for (v <- data) {
    if (v > max) {
      max = v
    }
  }
  max
}
```


### <a name='implicits'>Implicits</a>

__Do NOT use implicits__, unless:
- you are building a domain-specific language
- you are using it for implicit type parameters (e.g. `ClassTag`, `TypeTag`)
- you are using it private to your own class to reduce verbosity of converting from one type to another (e.g. Scala closure to Java closure)

When implicits are used, we must ensure that another engineer who did not author the code can understand the semantics of the usage without reading the implicit definition itself. Implicits have very complicated resolution rules and make the code base extremely difficult to understand. From Twitter's Effective Scala guide: "If you do find yourself using implicits, always ask yourself if there is a way to achieve the same thing without their help."

If you must use them (e.g. enriching some DSL), do not overload implicit methods, i.e. make sure each implicit method has distinct names, so users can selectively import them.
```scala
// Don't do the following, as users cannot selectively import only one of the implicit methods.
object ImplicitHolder {
  def toRdd(seq: Seq[Int]): RDD[Int] = ...
  def toRdd(seq: Seq[Long]): RDD[Long] = ...
}

// Do the following:
object ImplicitHolder {
  def intSeqToRdd(seq: Seq[Int]): RDD[Int] = ...
  def longSeqToRdd(seq: Seq[Long]): RDD[Long] = ...
}
```


## <a name='exception'>Exception Handling, i.e. Try vs try</a>

- Do NOT catch Throwable or Exception. Use `scala.util.control.NonFatal`:
  ```scala
  try {
    ...
  } catch {
    case NonFatal(e) =>
      // handle exception; note that NonFatal does not match InterruptedException
    case e: InterruptedException =>
      // handle InterruptedException
  }
  ```
  This ensures that we do not catch `NonLocalReturnControl` (as explained in [Return Statements](#return-statements)).

- Do NOT use `Try` in APIs, i.e. do NOT return Try in any methods.Prefer explicitly throwing exceptions for abnormal execution and Java style try/catch for exception handling.

  Background information: Scala provides monadic error handling (through `Try`, `Success`, and `Failure`) that facilitates chaining of actions. However, we found from our experience that the use of it often leads to more levels of nesting that are harder to read. In addition, it is often unclear what the semantics are for expected errors vs exceptions because those are not encoded in `Try`. As a result, we discouarge the use of `Try` for error handling. In particular:
  
  As a contrived example:
  ```scala
  class UserService {
    /** Look up a user's profile in the user database. */
    def get(userId: Int): Try[User]
  }
  ```
  is better written as
  ```scala
  class UserService {
    /**
     * Look up a user's profile in the user database.
     * @return None if the user is not found.
     * Throws DatabaseConnectionException when we have trouble connecting to the database/
     */
    @throws(DatabaseConnectionException)
    def get(userId: Int): Option[User]
  }
  ```
  The 2nd one makes it very obvious error cases the caller needs to handle.


### <a name='option'>Options</a>

- Use `Option` when the value can be empty. Compared with `null`, an `Option` explicitly states in the API contract that the value can be `None`.
- When constructing an `Option`, use `Option` rather than `Some` to guard against `null` values.
  ```scala
  def myMethod1(input: String): Option[String] = Option(transform(input))
  
  // This is not as robust because transform can return null, and then myMethod2 will return Some(null).
  def myMethod2(input: String): Option[String] = Some(transform(input))
  ```
- Do not use None to represent exceptions. Instead, throw exceptions explicitly.
- Do not call `get` directly on an `Option`, unless you know absolutely for sure the `Option` has some value.
  

### <a name='chaining'>Monadic Chaining</a>

One of Scala's powerful features is monadic chaining. Almost everything (e.g. collections, Option, Future, Try) is a monad (don't ask me what this means) and operations on them can be chained together. This is an incredibly powerful concept, but chaining should be used sparingly. In particular:

- Do NOT chain (and/or nest) more than 3 operations
- If it takes more than 2 seconds to figure out what the logic is, try hard to think about how you can expression the same functionality without using monadic chaining. As a general rule, watch out for flatMaps.
- A chain should almost always be broken after a flatMap (because of the type change)

A chain can often be made more understandable by giving the intermediate result a variable name, by explicitly typing the variable, and by breaking it down into more procedural style. As a contrived example:
```scala
class People(val data: Map[String, String])
val database = Map[String, People]
// Sometimes the client can store "null" value in the  store "address"

// A monadic chaining approach
def getAddress(name: String): Option[String] = {
  database.get(name).flatMap { elem =>
    elem.data.get("address")
      .flatMap(Option.apply)  // handle null value
  }
}

// A more readable approach, despite much longer
def getAddress(name: String): Option[String] = {
  if (!database.contains(name)) {
    return None
  }
  
  database(name).data.get("address") match {
    case Some(null) => None  // handle null value
    case Some(addr) => Option(addr)
    case None => None
  }
}

```


## <a name='concurrency'>Concurrency</a>

### <a name='concurrency-scala-collection'>Scala ConcurrentMap</a>

__Do NOT use Scala's concurrent collections__. Many methods in Scala's `scala.mutable.ConcurrentMap` are not safe from a concurrency perspective. In particular, `getOrElseUpdate` is __not atomic__.

Instead, use Java's `java.util.concurrent.ConcurrentHashMap` and the `putIfAbsent` method.


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
    def update(key: String, value: String) = synchronized {
      map.put(key, value)
      count += 1
    }
    def getCount: Int = synchronized { count }
  }
  ```

Note that for case 1 and case 2, do not let views or iterators of the collections escape the protected area. This can happen in non-obvious ways, e.g. when returning `Map.keySet` or `Map.values`. If views or values are requried to pass around, make a copy of the data.
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

Prefer Atomic varibles over explicit synchronization when: (1) all critical updates for an object are confined to a *single* variable and contention is expected. Atomic variables are lock-free and permit more efficient contention. Or (2) synchronization is clearly expressed as a `getAndSet` operation. For example:
  ```scala
  // good: clearly and efficiently express only-once execution of concurrent code
  val initialized = new AtomicBoolean(false)
  if (!initialized.getAndSet(true)) {
    ...
  }
  
  // poor: less clear what is guarded by synchronization, may unnecessarily synchronize
  val initialized = false
  synchronized {
    val wasInitialized = initialized
    initialized = true
  }
  if (!wasInitialized) {
    ...
  }
  ```

### <a name='concurrency-private-this'>Private Fields</a>

Note that `private` fields are still accessible by other instances of the same class, so protecting it with `this.synchronized` (or just `synchronized`) is not sufficient. Make the field `private[this]` instead.
```scala
// The following is still unsafe.
class Foo {
  @volatile private var count: Int = 0
  def inc(): Unit = synchronized { count + 1 }
}

// The following is safe.
class Foo {
  @volatile private[this] var count: Int = 0
  def inc(): Unit = synchronized { count + 1 }
}
```


### <a name='concurrency-futures'>Futures</a>

Work in progress section

- If you are creating a new future yourself, make sure the block of code can timeout. Otherwise, the task may never complete and leak a thread, or even worse, never release a lock.
- onComplete => andThen to guarantee ordering
- thread pool creation


## <a name='perf'>Performance</a>

For the vast majority of the code you write, performance should not be a concern. However, for performance sensitive code, here are some tips:

### <a name='perf-microbenchmarks'>Microbenchmarks</a>

It is ridiculously hard to write a good microbenchmark because the Scala compiler and the JVM JIT compiler does a lot of magic to the code. More often than not your microbenchmark code is not measuring the thing you want to measure.

Use [jmh](http://openjdk.java.net/projects/code-tools/jmh/) if you are writing microbenchmark code. Make sure you read through [all the sample microbenchmarks](http://hg.openjdk.java.net/code-tools/jmh/file/tip/jmh-samples/src/main/java/org/openjdk/jmh/samples/) so you understand the effect of deadcode elimination, constant folding, and loop unrolling on microbenchmarks.


### <a name='perf-whileloops'>Traversal and zipWithIndex</a>

Use `while` loops intead of `for` loops or functional transformations (e.g. `map`, `foreach`). For loops and functional transformations are very slow (due to virtual function calls and boxing).
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
while (i < newArr.length) {
  newArr(i) = if (i % 2 == 0) 0 else arr(i) 
  i += 1
}
```

### <a name='perf-option'>Option and null</a>

For performance sensitive code, prefer `null` over `Option`, in order to avoid virtual function calls and boxing. Label the nullable fields clearly with Nullable.
```scala
class Foo {
  @javax.annotation.Nullable
  private[this] var nullableField: Bar = _
}
```

### <a name='perf-collection'>Scala Collection Library</a>

For performance sensitive code, prefer Java collection library over Scala ones. The Scala collection library often is slower than Java's, and some operations can have surpringly bad asympotic performance (i.e. seemingly O(1) operations are turned into O(n)). An especially bad offender of this is `Seq#size()` being O(n).

### <a name='perf-private'>private[this]</a>

For performance sensitive code, prefer `private[this]` over `private`. `private[this]` generates a field, rather than creating an accessor method. In our experience, the JVM JIT compiler cannot always inline `private` field accessor methods, and thus it is safer to use `private[this]` to ensure no virtual function call for accessing a field.
```scala
class MyClass {
  private val field1 = ...
  private[this] val field2 = ...

  def perfSensitiveMethod(): Unit = {
    var i = 0
    while (i < 1000000) {
      field1  // This might invoke a virtual function call
      field2  // This is just a field access
      i += 1
    }
  }
}
```


## <a name='java'>Java Interoperability</a>

This section covers guidelines for building Java compatible APIs. These do not apply if the component you are building do not require interoperability with Java. It is mostly drawn from our experience in developing the Java APIs for Spark.


### <a name='java-missing-features'>Java Features Missing from Scala</a>

The following Java features are missing from Scala. If you need the following, define them in Java instead. However, be reminded that ScalaDocs are not generated for files defined in Java.

- Static fields
- Static inner classes
- Java enums
- Annotations


### <a name='java-traits'>Traits and Abstract Classes</a>

For interfaces that can be implemented externally, keep in mind the following:

- Traits with default method implementations are not usable in Java. Use abstract classes instead.
- In general, avoid using traits unless you know for sure the interface will not have any default implementation even in its future evolution.
```scala
// The default implementation doesn't work in Java
trait Listener {
  def onTermination(): Unit = { ... }
}

// Works in Java
abstract class Listener {
  def onTermination(): Unit = { ... }
}
```


### <a name='java-type-alias'>Type Aliases</a>

Do NOT use type aliases. They are not visible in bytecode (and Java).


### <a name='java-default-param-values'>Default Parameter Values</a>

Do NOT use default parameter values. Overload the method instead.
```scala
// Breaks Java interoperability
def sample(ratio: Double, withReplacement: Boolean = false): RDD[T] = { ... }

// The following two work
def sample(ratio: Double, withReplacement: Boolean): RDD[T] = { ... }
def sample(ratio: Double): RDD[T] = sample(ratio, withReplacement = false)
```

### <a name='java-multi-param-list'>Multiple Parameter Lists</a>

Do NOT use multi-parameter lists.

### <a name='java-varargs'>Varargs</a>

- Apply `@scala.annotation.varargs` annotation for a vararg method to be usable in Java. The Scala compiler creates two methods, one for Scala (bytecode parameter is a Seq) and one for Java (bytecode parameter array).
  ```scala
  @scala.annotation.varargs
  def select(exprs: Expression*): DataFrame = { ... }
  ```

- Note that abstract vararg methods does NOT work for Java, due to a Scala compiler bug ([SI-1459](https://issues.scala-lang.org/browse/SI-1459), [SI-9013](https://issues.scala-lang.org/browse/SI-9013)).

- Be careful with overloading varargs methods. Overloading a vararg method with another vararg type can break source compatibility.
  ```scala
  class Database {
    @scala.annotation.varargs
    def remove(elems: String*): Unit = ...
    
    // Adding this will break source compatibility for no-arg remove() call.
    @scala.annotation.varargs
    def remove(elems: People*): Unit = ...
  }
  
  // This won't compile anymore because it is ambiguous
  new Database().remove()  
  ```
  Instead, define an explicit first parameter followed by vararg:
  ```scala
  class Database {
    @scala.annotation.varargs
    def remove(elems: String*): Unit = ...
    
    // The following is OK.
    @scala.annotation.varargs
    def remove(elem: People, elems: People*): Unit = ...
  }
  ```


### <a name='java-implicits'>Implicits</a>

Do NOT use implicits, for a class or function. This includes `ClassTag`, `TypeTag`.

### <a name='java-companion-object'>Companion Objects, Static Methods and Fields</a>

There are a few things to watch out for when it comes to companion objects and static methods/fields.

- Companion objects are awkward to use in Java (a companion object `Foo` is a static field `MODULE$` of type `Foo$` in class `Foo$`).
  ```scala
  object Foo
  
  // equivalent to the following Java code
  public class Foo$ {
    Foo$ MODULE$ = // instantiation of the object
  }
  ```
  If the companion object is important to use, create a Java static field in a separate class.

- Unfortunately, there is no way to define a JVM static field in Scala. Create a Java file to define that.
- Methods in companion objects are automatically turned into static methods in the companion class, unless there is a method name conflict. The best (and future-proof) way to guarantee the generation of static methods is to add a test file written in Java that calls the static method.
  ```scala
  class Foo {
    def method2(): Unit = { ... }
  }
  
  object Foo {
    def method1(): Unit = { ... }  // a static method Foo.method1 is created in bytecode
    def method2(): Unit = { ... }  // a static method Foo.method1 is NOT created in bytecode
  }
  
  // FooJavaTest.java (in test/scala/com/databricks/...)
  public class FooJavaTest {
    public static compileTest() {
      Foo.method1();  // This one should compile fine
      Foo.method2();  // This one should fail because method2 is not generated.
    }
  }
  ```

- A case object (or even just plain companion object) MyClass is actually not of type MyClass
  ```scala
  case object MyClass
  
  // Test.java
  if (MyClass$.MODULE instanceof MyClass) {
    // The above condition is always false
  }
  ```
  To implement the proper type hierarchy, define a companion class, and then extend that in case object:
  ```scala
  class MyClass
  case object MyClass extends MyClass
  ```


## <a name='misc'>Miscellaneous</a>

### <a name='misc_currentTimeInMillis_vs_nanoTime'>Prefer nanoTime over currentTimeInMillis</a>

When computing a *duration* or checking for a *timeout*, avoid using `System.currentTimeInMillis()`. Use `System.nanoTime()` instead, even if you are not interested in sub-millisecond precision.

`System.currentTimeInMillis()` returns current wallclock time and will follow changes to the system clock. Thus, negative wallclock adjustments can cause timeouts to "hang" for a long time (until wallclock time has caught up to its previous value again).  This can happen when ntpd does a "step" after the network has been disconncted for some time. The most canonoical example is during system bootup when DHCP takes longer than usual. This can lead to failures that are really hard to understand/reproduce. `System.nanoTime()` it is guaranteed to be monotonically increasing irrespective of wallclock changes.

Caveats:
- Never serialize an absolute `nanoTime()` value or pass it to another system. The absolute value is meaningless, system-specific and reset when the system reboots.
- The absolute `nanoTime()` value is not guranteed to be positive (but `t2 - t1` is guaranteed to yield the right result)
- `nanoTime()` rolls over every 292 years. So if your spark Job is going to take a really long time, you may need something else :)


### <a name='misc_uri_url'>Prefer URI over URL</a>

When storing the URL of a service, you should use the `URI` representation.

The [equality check](http://docs.oracle.com/javase/7/docs/api/java/net/URI.html#equals(java.lang.Object)) of `URL` actually performs a (blocking) network call to resolve the IP address. The `URI` class performs field equality and is a superset of `URL` as to what it can represent.
