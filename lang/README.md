1. [Scala Language Features](#lang)
    * [Case Classes and Immutability](#case_class_immutability)
    * [apply Method](#apply_method)
    * [override Modifier](#override_modifier)
    * [Destructuring Binds](#destruct_bind)
    * [Call by Name](#call_by_name)
    * [Multiple Parameter Lists](#multi-param-list)
    * [Symbolic Methods (Operator Overloading)](#symbolic_methods)
    * [Type Inference](#type_inference)
    * [Return Statements](#return)
    * [Recursion and Tail Recursion](#recursion)
    * [Implicits](#implicits)
    * [Exception Handling (Try vs try)](#exception)
    * [Options](#option)
    * [Monadic Chaining](#chaining)

## <a name='lang'>Scala Language Features</a>

### <a name='case_class_immutability'>Case Classes and Immutability</a>

Case classes are regular classes but extended by the compiler to automatically support:
- Public getters for constructor parameters
- Copy constructor
- Pattern matching on constructor parameters
- Automatic toString/hash/equals implementation

Constructor parameters should NOT be mutable for case classes. Instead, use copy constructor.
Having mutable case classes can be error prone, e.g. hash maps might place the object in the wrong bucket using the old hash code.
```scala
// This is OK
case class Person(name: String, age: Int)

// This is NOT OK
case class Person(name: String, var age: Int)

// To change values, use the copy constructor to create a new instance
val p1 = Person("Peter", 15)
val p2 = p1.copy(age = 16)
```


### <a name='apply_method'>apply Method</a>

Avoid defining apply methods on classes. These methods tend to make the code less readable, especially for people less familiar with Scala. It is also harder for IDEs (or grep) to trace. In the worst case, it can also affect correctness of the code in surprising ways, as demonstrated in [Parentheses](#parentheses).

It is acceptable to define apply methods on companion objects as factory methods. In these cases, the apply method should return the companion class type.
```scala
object TreeNode {
  // This is OK
  def apply(name: String): TreeNode = ...

  // This is bad because it does not return a TreeNode
  def apply(name: String): String = ...
}
```


### <a name='override_modifier'>override Modifier</a>
Always add override modifier for methods, both for overriding concrete methods and implementing abstract methods. The Scala compiler does not require `override` for implementing abstract methods. However, we should always add `override` to make the override obvious, and to avoid accidental non-overrides due to non-matching signatures.
```scala
trait Parent {
  def hello(data: Map[String, String]): Unit = {
    print(data)
  }
}

class Child extends Parent {
  import scala.collection.Map

  // The following method does NOT override Parent.hello,
  // because the two Maps have different types.
  // If we added "override" modifier, the compiler would've caught it.
  def hello(data: Map[String, String]): Unit = {
    print("This is supposed to override the parent method, but it is actually not!")
  }
}
```



### <a name='destruct_bind'>Destructuring Binds</a>

Destructuring bind (sometimes called tuple extraction) is a convenient way to assign two variables in one expression.
```scala
val (a, b) = (1, 2)
```

However, do NOT use them in constructors, especially when `a` and `b` need to be marked transient. The Scala compiler generates an extra Tuple2 field that will not be transient for the above example.
```scala
class MyClass {
  // This will NOT work because the compiler generates a non-transient Tuple2
  // that points to both a and b.
  @transient private val (a, b) = someFuncThatReturnsTuple2()
}
```


### <a name='call_by_name'>Call by Name</a>

__Avoid using call by name__. Use `() => T` explicitly.

Background: Scala allows method parameters to be defined by-name, e.g. the following would work:
```scala
def print(value: => Int): Unit = {
  println(value)
  println(value + 1)
}

var a = 0
def inc(): Int = {
  a += 1
  a
}

print(inc())
```
in the above code, `inc()` is passed into `print` as a closure and is executed (twice) in the print method, rather than being passed in as a value `1`. The main problem with call-by-name is that the caller cannot differentiate between call-by-name and call-by-value, and thus cannot know for sure whether the expression will be executed or not (or maybe worse, multiple times). This is especially dangerous for expressions that have side-effect.


### <a name='multi-param-list'>Multiple Parameter Lists</a>

__Avoid using multiple parameter lists__. They complicate operator overloading, and can confuse programmers less familiar with Scala. For example:

```scala
// Avoid this!
case class Person(name: String, age: Int)(secret: String)
```

One notable exception is the use of a 2nd parameter list for implicits when defining low-level libraries. That said, [implicits should be avoided](#implicits)!


### <a name='symbolic_methods'>Symbolic Methods (Operator Overloading)</a>

__Do NOT use symbolic method names__, unless you are defining them for natural arithmetic operations (e.g. `+`, `-`, `*`, `/`). Under no other circumstances should they be used. Symbolic method names make it very hard to understand the intent of the methods. Consider the following two examples:
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
- __Variables or closures with non-obvious types should be explicitly typed__. A good litmus test is that explicit types should be used if a code reviewer cannot determine the type in 3 seconds.


### <a name='return'>Return Statements</a>

__Avoid using return in closures__. `return` is turned into ``try/catch`` of ``scala.runtime.NonLocalReturnControl`` by the compiler. This can lead to unexpected behaviors. Consider the following example:
  ```scala
  def receive(rpc: WebSocketRPC): Option[Response] = {
    tableFut.onComplete { table =>
      if (table.isFailure) {
        return None // Do not do that!
      } else { ... }
    }
  }
  ```
  the `.onComplete` method takes the anonymous closure `{ table => ... }` and passes it to a different thread. This closure eventually throws the `NonLocalReturnControl` exception that is captured __in a different thread__ . It has no effect on the poor method being executed here.

However, there are a few cases where `return` is preferred.

- Use `return` as a guard to simplify control flow without adding a level of indentation
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

__Avoid using recursion__, unless the problem can be naturally framed recursively (e.g. graph traversal, tree traversal).

For methods that are meant to be tail recursive, apply `@tailrec` annotation to make sure the compiler can check it is tail recursive. (You will be surprised how often seemingly tail recursive code is actually not tail recursive due to the use of closures and functional transformations.)

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

__Avoid using implicits__, unless:
- you are building a domain-specific language
- you are using it for implicit type parameters (e.g. `ClassTag`, `TypeTag`)
- you are using it private to your own class to reduce verbosity of converting from one type to another (e.g. Scala closure to Java closure)

When implicits are used, we must ensure that another engineer who did not author the code can understand the semantics of the usage without reading the implicit definition itself. Implicits have very complicated resolution rules and make the code base extremely difficult to understand. From Twitter's Effective Scala guide: "If you do find yourself using implicits, always ask yourself if there is a way to achieve the same thing without their help."

If you must use them (e.g. enriching some DSL), do not overload implicit methods, i.e. make sure each implicit method has distinct names, so users can selectively import them.
```scala
// Don't do the following, as users cannot selectively import only one of the methods.
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


## <a name='exception'>Exception Handling (Try vs try)</a>

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

- Do NOT use `Try` in APIs, that is, do NOT return Try in any methods. Instead, prefer explicitly throwing exceptions for abnormal execution and Java style try/catch for exception handling.

  Background information: Scala provides monadic error handling (through `Try`, `Success`, and `Failure`) that facilitates chaining of actions. However, we found from our experience that the use of it often leads to more levels of nesting that are harder to read. In addition, it is often unclear what the semantics are for expected errors vs exceptions because those are not encoded in `Try`. As a result, we discourage the use of `Try` for error handling. In particular:

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
     * @throws DatabaseConnectionException when we have trouble connecting to the database/
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

  // This is not as robust because transform can return null, and then
  // myMethod2 will return Some(null).
  def myMethod2(input: String): Option[String] = Some(transform(input))
  ```
- Do not use None to represent exceptions. Instead, throw exceptions explicitly.
- Do not call `get` directly on an `Option`, unless you know absolutely for sure the `Option` has some value.


### <a name='chaining'>Monadic Chaining</a>

One of Scala's powerful features is monadic chaining. Almost everything (e.g. collections, Option, Future, Try) is a monad and operations on them can be chained together. This is an incredibly powerful concept, but chaining should be used sparingly. In particular:

- Avoid chaining (and/or nesting) more than 3 operations.
- If it takes more than 5 seconds to figure out what the logic is, try hard to think about how you can express the same functionality without using monadic chaining. As a general rule, watch out for flatMaps and folds.
- A chain should almost always be broken after a flatMap (because of the type change).

A chain can often be made more understandable by giving the intermediate result a variable name, by explicitly typing the variable, and by breaking it down into more procedural style. As a contrived example:
```scala
class Person(val data: Map[String, String])
val database = Map[String, Person]
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


