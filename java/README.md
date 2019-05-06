
1. [Java Interoperability](#java)
    * [Java Features Missing from Scala](#java-missing-features)
    * [Traits and Abstract Classes](#java-traits)
    * [Type Aliases](#java-type-alias)
    * [Default Parameter Values](#java-default-param-values)
    * [Multiple Parameter Lists](#java-multi-param-list)
    * [Varargs](#java-varargs)
    * [Implicits](#java-implicits)
    * [Companion Objects, Static Methods and Fields](#java-companion-object)


## <a name='java'>Java Interoperability</a>

This section covers guidelines for building Java compatible APIs. These do not apply if the component you are building does not require interoperability with Java. It is mostly drawn from our experience in developing the Java APIs for Spark.


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

Do NOT use implicits, for a class or method. This includes `ClassTag`, `TypeTag`.
```scala
class JavaFriendlyAPI {
  // This is NOT Java friendly, since the method contains an implicit parameter (ClassTag).
  def convertTo[T: ClassTag](): T
}
```

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
    def method2(): Unit = { ... }  // a static method Foo.method2 is NOT created in bytecode
  }

  // FooJavaTest.java (in test/scala/com/databricks/...)
  public class FooJavaTest {
    public static void compileTest() {
      Foo.method1();  // This one should compile fine
      Foo.method2();  // This one should fail because method2 is not generated.
    }
  }
  ```

- A case object (or even just plain companion object) MyClass is actually not of type MyClass.
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

