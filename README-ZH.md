# Databricks Scala 编程风格指南

## 声明 (Disclaimer)

The Chinese version of the [Databricks Scala Guide](https://github.com/databricks/scala-style-guide) is contributed and maintained by community member [Hawstein](https://github.com/Hawstein). We do not guarantee that it will always be kept up-to-date.

本文档翻译自 [Databricks Scala Guide](https://github.com/databricks/scala-style-guide)，目前由 [Hawstein](https://github.com/Hawstein) 进行维护。由于是利用业余时间进行翻译并维护，因此该中文文档并不保证总是与[原文档](https://github.com/databricks/scala-style-guide)一样处于最新版本，不过我会尽可能及时地去更新它。

## 前言

Spark 有超过 1000 位贡献者，就我们所知，应该是目前大数据领域里最大的开源项目且是最活跃的 Scala 项目。这份指南是在我们指导，或是与 Spark 贡献者及 [Databricks](http://databricks.com/) 工程团队一起工作时总结出来的。

代码由作者 __一次编写__ ，然后由大量工程师 __多次阅读并修改__ 。事实上，大部分的 bug 来源于后人对代码的修改，因此我们需要长期去优化我们的代码，提升代码的可读性和可维护性。达到这个目标最好的方式就是编写简单易懂的代码。

Scala 是一种强大到令人难以置信的多范式编程语言。我们总结出了以下指南，它可以很好地应用在一个高速发展的项目。当然，这个指南并非绝对，根据团队需求的不同，可以有不同的标准。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.

## <a name='TOC'>目录</a>

1. [文档历史](#history)

1. [语法风格](#syntactic)
    - [命名约定](#naming)
    - [变量命名约定](#variable-naming)
    - [一行长度](#linelength)
    - [30 法则](#rule_of_30)
    - [空格与缩进](#indent)
    - [空行](#blanklines)
    - [括号](#parentheses)
    - [大括号](#curly)
    - [长整型字面量](#long_literal)
    - [文档风格](#doc)
    - [类内秩序](#ordering_class)
    - [Imports](#imports)
    - [模式匹配](#pattern-matching)
    - [中缀方法](#infix)
    - [匿名方法](#anonymous)

1. [Scala 语言特性](#lang)
    - [样例类与不可变性](#case_class_immutability)
    - [apply 方法](#apply_method)
    - [override 修饰符](#override_modifier)
    - [解构绑定](#destruct_bind)
    - [按名称传参](#call_by_name)
    - [多参数列表](#multi-param-list)
    - [符号方法 (运算符重载)](#symbolic_methods)
    - [类型推导](#type_inference)
    - [Return 语句](#return)
    - [递归及尾递归](#recursion)
    - [Implicits](#implicits)
    - [异常处理 (Try 还是 try)](#exception)
    - [Options](#option)
    - [单子链接](#chaining)

1. [并发](#concurrency)
    - [Scala concurrent.Map](#concurrency-scala-collection)
    - [显式同步 vs 并发集合](#concurrency-sync-vs-map)
    - [显式同步 vs 原子变量 vs @volatile](#concurrency-sync-vs-atomic)
    - [私有字段](#concurrency-private-this)
    - [隔离](#concurrency-isolation)

1. [性能](#perf)
    - [Microbenchmarks](#perf-microbenchmarks)
    - [Traversal 与 zipWithIndex](#perf-whileloops)
    - [Option 与 null](#perf-option)
    - [Scala 集合库](#perf-collection)
    - [private[this]](#perf-private)

1. [与 Java 的互操作性](#java)
    - [Scala 中缺失的 Java 特性](#java-missing-features)
    - [Traits 与抽象类](#java-traits)
    - [类型别名](#java-type-alias)
    - [默认参数值](#java-default-param-values)
    - [多参数列表](#java-multi-param-list)
    - [可变参数](#java-varargs)
    - [Implicits](#java-implicits)
    - [伴生对象, 静态方法与字段](#java-companion-object)

1. [测试](#testing)
    - [异常拦截](#testing-intercepting)
    
1. [其它](#misc)
    - [优先使用 nanoTime 而非 currentTimeMillis](#misc_currentTimeMillis_vs_nanoTime)
    - [优先使用 URI 而非 URL](#misc_uri_url)
    - [优先使用现存的经过良好测试的方法而非重新发明轮子](#misc_well_tested_method)

## <a name='history'>文档历史</a>
- 2015-03-16: 最初版本。
- 2015-05-25: 增加 [override 修饰符](#override_modifier) 一节。
- 2015-08-23: 把一些规则的严重程度从「不要」降级到「避免」。
- 2015-11-17: 更新 [apply 方法](#apply_method) 一节：伴生对象中的 apply 方法应该返回其伴生类。
- 2015-11-17: 该指南被翻译成[中文](README-ZH.md)，由 [Hawstein](https://github.com/Hawstein) 进行维护，中文文档并不保证总是与原文档一样处于最新版本。
- 2015-12-14: 该指南被翻译成[韩文](README-KO.md), 韩文版本由 [Hyukjin Kwon](https://github.com/HyukjinKwon) 进行翻译并且由 [Yun Park](https://github.com/yunpark93), [Kevin (Sangwoo) Kim](https://github.com/swkimme), [Hyunje Jo](https://github.com/RetrieverJo) 和 [Woochel Choi](https://github.com/socialpercon) 进行校对。韩文版本并不保证总是与原文档一样处于最新版本。
- 2016-06-15: 增加 [匿名方法](#anonymous) 一节。
- 2016-06-21: 增加 [变量命名约定](#variable-naming) 一节。
- 2016-12-24: 增加 [样例类与不可变性](#case_class_immutability) 一节。
- 2017-02-23: 增加 [测试](#testing) 一节。
- 2017-04-18: 增加 [优先使用现存的经过良好测试的方法而非重新发明轮子](#misc_well_tested_method) 一节。

## <a name='syntactic'>语法风格</a>

### <a name='naming'>命名约定</a>

我们主要遵循 Java 和 Scala 的标准命名约定。

- 类，trait, 对象应该遵循 Java 中类的命名约定，即 PascalCase 风格。

  ```scala
  class ClusterManager

  trait Expression
  ```

- 包名应该遵循 Java 中包名的命名约定，即使用全小写的 ASCII 字母。

  ```scala
  package com.databricks.resourcemanager
  ```

- 方法/函数应当使用驼峰式风格命名。

- 常量命名使用全大写字母，并将它们放在伴生对象中。

  ```scala
  object Configuration {
    val DEFAULT_PORT = 10000
  }
  ```

- 枚举命名与类命名一致，使用 PascalCase 风格。

- 注解也应遵循 Java 中的约定，即使用 PascalCase 风格。注意，这一点与 Scala 的官方指南不同。

  ```scala
  final class MyAnnotation extends StaticAnnotation
  ```

### <a name='variable-naming'>变量命名约定</a>

- 变量命名应当遵循驼峰式命名方法，并且变量名应当是不言而喻的，即变量名可以直观地反应它的涵义。
  ```scala
  val serverPort = 1000
  val clientPort = 2000
  ```

- 可以在小段的局部代码中使用单字符的变量名，比如在小段的循环体中（例如 10 行以内的代码），“i” 常常被用作循环索引。然而，即使在小段的代码中，也不要使用 “l” （Larry 中的 l）作为标识符，因为它看起来和 “1”，“|”，“I” 很像，难以区分，容易搞错。

### <a name='linelength'>一行长度</a>

- 一行长度的上限是 100 个字符。
- 唯一的例外是 import 语句和 URL (即便如此，也尽量将它们保持在 100 个字符以下)。


### <a name='rule_of_30'>30 法则</a>

「如果一个元素包含的子元素超过 30 个，那么极有可能出现了严重的问题」 - [Refactoring in Large Software Projects](http://www.amazon.com/Refactoring-Large-Software-Projects-Restructurings/dp/0470858923)。

一般来说:

- 一个方法包含的代码行数不宜超过 30 行。
- 一个类包含的方法数量不宜超过 30 个。


### <a name='indent'>空格与缩进</a>

- 运算符前后保留一个空格，包括赋值运算符。
  ```scala
  def add(int1: Int, int2: Int): Int = int1 + int2
  ```

- 逗号后保留一个空格。
  ```scala
  Seq("a", "b", "c") // 使用这种方式

  Seq("a","b","c") // 不要忽略逗号后的空格
  ```

- 冒号后保留一个空格。
  ```scala
  // 使用这种方式
  def getConf(key: String, defaultValue: String): String = {
    // some code
  }

  // 冒号前不需要使用空格
  def calculateHeaderPortionInBytes(count: Int) : Int = {
    // some code
  }

  // 不要忽略冒号后的空格
  def multiply(int1:Int, int2:Int): Int = int1 * int2
  ```

- 一般情况下，使用两个空格的缩进。

  ```scala
  if (true) {
    println("Wow!")
  }
  ```

- 对于方法声明，如果两行无法容纳下所有的参数，那么将每个参数单独放在一行，并使用 4 个空格进行缩进。返回类型可以与最后一个参数在同一行，也可以放在新的一行，使用两个空格缩进。

  ```scala
  def newAPIHadoopFile[K, V, F <: NewInputFormat[K, V]](
      path: String,
      fClass: Class[F],
      kClass: Class[K],
      vClass: Class[V],
      conf: Configuration = hadoopConfiguration): RDD[(K, V)] = {
    // 方法体
  }

  def newAPIHadoopFile[K, V, F <: NewInputFormat[K, V]](
      path: String,
      fClass: Class[F],
      kClass: Class[K],
      vClass: Class[V],
      conf: Configuration = hadoopConfiguration)
    : RDD[(K, V)] = {
    // 方法体
  }
  ```

- 如果两行无法容纳下类头（即 { 前面的部分），那么将每个类参数单独放在一行，并使用 4 个空格进行缩进；将 extends 关键字放在（最后一个参数的）下一行，并使用 2 个空格进行缩进。在类头定义结束后空一行，再开始类内函数或变量的定义。

  ```scala
  class Foo(
      val param1: String,  // 对参数使用 4 个空格进行缩进
      val param2: String,
      val param3: Array[Byte])
    extends FooInterface  // 这里使用 2 个空格进行缩进
    with Logging {

    def firstMethod(): Unit = { ... }  // 上面空一行
  }
  ```

- 对于方法和类的构造函数调用，如果两行无法容纳下所有的参数，那么将每个参数单独放在一行，并使用 2 个空格进行缩进。

  ```scala
  foo(
    someVeryLongFieldName,  // 这里使用 2 个空格进行缩进
    andAnotherVeryLongFieldName,
    "this is a string",
    3.1415)

  new Bar(
    someVeryLongFieldName,  // 这里使用 2 个空格进行缩进
    andAnotherVeryLongFieldName,
    "this is a string",
    3.1415)
  ```

- 不要使用垂直对齐。它使你的注意力放在代码的错误部分并增大了后人修改代码的难度。

  ```scala
  // 不要（对等于号）使用垂直对齐
  val plus     = "+"
  val minus    = "-"
  val multiply = "*"

  // 使用下面的写法
  val plus = "+"
  val minus = "-"
  val multiply = "*"
  ```


### <a name='blanklines'>空行</a>

- 一个空行可以出现在：
  - 连续的类成员或初始化器（initializers）之间：字段，构造函数，方法，嵌套类，静态初始化器及实例初始化器。
    - 例外：连续的两个字段之间的空行是可选的（前提是它们之间没有其它代码），这一类空行主要为这些字段做逻辑上的分组。
  - 在方法体内，根据需要，使用空行来为语句创建逻辑上的分组。
  - 在类的第一个成员之前或最后一个成员之后，空行都是可选的（既不鼓励也不阻止）。
- 使用一个或两个空行来分隔不同类的定义。
- 不鼓励使用过多的空行。


### <a name='parentheses'>括号</a>

- 方法声明应该加括号（即使没有参数列表），除非它们是没有副作用（状态改变，IO 操作都认为是有副作用的）的访问器（accessor）。

  ```scala
  class Job {
    // 错误：killJob 会改变状态，应该加上括号。
    def killJob: Unit

    // 正确：
    def killJob(): Unit
  }
  ```

- 函数调用应该与函数声明在形式上保持一致，也就是说，如果一个方法声明时带了括号，那调用时也要把括号带上。注意这不仅仅是语法层面的人为约定，当返回对象中定义了 `apply` 方法时，这一点还会影响正确性。

  ```scala
  class Foo {
    def apply(args: String*): Int
  }

  class Bar {
    def foo: Foo
  }

  new Bar().foo  // 这里返回一个 Foo 对象
  new Bar().foo()  // 这里返回一个 Int 值！
  ```


### <a name='curly'>大括号</a>

即使条件语句或循环语句只有一行时，也请使用大括号。唯一的例外是，当你把 if/else 作为一个单行的三元操作符来使用并且没有副作用时，这时你可以不加大括号。

```scala
// 正确：
if (true) {
  println("Wow!")
}

// 正确：
if (true) statement1 else statement2

// 正确：
try {
  foo()
} catch {
  ...
}

// 错误：
if (true)
  println("Wow!")

// 错误：
try foo() catch {
  ...
}
```


### <a name='long_literal'>长整型字面量</a>

长整型字面量使用大写的 `L` 作为后缀，不要使用小写，因为它和数字 `1` 长得很像，常常难以区分。

```scala
val longValue = 5432L  // 这样写

val longValue = 5432l  // 不要这样写
```


### <a name='doc'>文档风格</a>

使用 Java Doc 风格，而非 Scala Doc 风格。

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


### <a name='ordering_class'>类内秩序</a>

如果一个类很长，包含许多的方法，那么在逻辑上把它们分成不同的部分并加上注释头，以此组织它们。

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

当然，强烈不建议把一个类写得这么长，一般只有在构建某些公共 API 时才允许这么做。


### <a name='imports'>Imports</a>

- __导入时避免使用通配符__, 除非你需要导入超过 6 个实体或者隐式方法。通配符导入会使代码在面对外部变化时不够健壮。
- 始终使用绝对路径来导入包 (如：`scala.util.Random`) ，而不是相对路径 (如：`util.Random`)。
- 此外，导入语句按照以下顺序排序：
  * `java.*` 和 `javax.*`
  * `scala.*`
  * 第三方库 (`org.*`, `com.*`, 等)
  * 项目中的类 (对于 Spark 项目，即 `com.databricks.*` 或 `org.apache.spark`)
- 在每一组导入语句内，按照字母序进行排序。
- 你可以使用 IntelliJ 的「import organizer」来自动处理，请使用以下配置：

  ```
  java
  javax
  _______ blank line _______
  scala
  _______ blank line _______
  all other imports
  _______ blank line _______
  com.databricks  // or org.apache.spark if you are working on spark
  ```


### <a name='pattern-matching'>模式匹配</a>

- 如果整个方法就是一个模式匹配表达式，可能的话，可以把 match 关键词与方法声明放在同一行，以此减少一级缩进。

  ```scala
  def test(msg: Message): Unit = msg match {
    case ...
  }
  ```

- 当以闭包形式调用一个函数时，如果只有一个 case 语句，那么把 case 语句与函数调用放在同一行。

  ```scala
  list.zipWithIndex.map { case (elem, i) =>
    // ...
  }
  ```
  如果有多个 case 语句，把它们缩进并且包起来。

  ```scala
  list.map {
    case a: Foo =>  ...
    case b: Bar =>  ...
  }
  ```

- 如果唯一的目的就是想匹配某个对象的类型，那么不要展开所有的参数来做模式匹配，这样会使得重构变得更加困难，代码更容易出错。

```scala
case class Pokemon(name: String, weight: Int, hp: Int, attack: Int, defense: Int)
case class Human(name: String, hp: Int)

// 不要像下面那样做，因为
// 1. 当 pokemon 加入一个新的字段，我们需要改变下面的模式匹配代码
// 2. 非常容易发生误匹配，尤其是当所有字段的类型都一样的时候
targets.foreach {
  case target @ Pokemon(_, _, hp, _, defense) =>
    val loss = sys.min(0, myAttack - defense)
    target.copy(hp = hp - loss)
  case target @ Human(_, hp) =>
    target.copy(hp = hp - myAttack)
}

// 像下面这样做就好多了:
targets.foreach {
  case target: Pokemon =>
    val loss = sys.min(0, myAttack - target.defense)
    target.copy(hp = target.hp - loss)
  case target: Human =>
    target.copy(hp = target.hp - myAttack)
}
```


### <a name='infix'>中缀方法</a>

__避免中缀表示法__，除非是符号方法（即运算符重载）。

```scala
// 正确
list.map(func)
string.contains("foo")

// 错误
list map (func)
string contains "foo"

// 重载的运算符应该以中缀形式调用
arrayBuffer += elem
```

### <a name='anonymous'>匿名方法</a>

对于匿名方法，__避免使用过多的小括号和花括号__。

```scala
// 正确
list.map { item =>
  ...
}

// 正确
list.map(item => ...)

// 错误
list.map(item => {
  ...
})

// 错误
list.map { item => {
  ...
}}

// 错误
list.map({ item => ... })
```


## <a name='lang'>Scala 语言特性</a>

### <a name='case_class_immutability'>样例类与不可变性</a>

样例类（case class）本质也是普通的类，编译器会自动地为它加上以下支持：
- 构造器参数的公有 getter 方法
- 拷贝构造函数
- 构造器参数的模式匹配
- 默认的 toString/hash/equals 实现

对于样例类来说，构造器参数不应设为可变的，可以使用拷贝构造函数达到同样的效果。使用可变的样例类容易出错，例如，哈希表中，对象根据旧的哈希值被放在错误的位置上。

```scala
// 这是 OK 的
case class Person(name: String, age: Int)

// 这是不 OK 的
case class Person(name: String, var age: Int)

// 通过拷贝构造函数创建一个新的实例来改变其中的值
val p1 = Person("Peter", 15)
val p2 = p1.copy(age = 16)
```


### <a name='apply_method'>apply 方法</a>

避免在类里定义 apply 方法。这些方法往往会使代码的可读性变差，尤其是对于不熟悉 Scala 的人。它也难以被 IDE（或 grep）所跟踪。在最坏的情况下，它还可能影响代码的正确性，正如你在[括号](#parentheses)一节中看到的。

然而，将 apply 方法作为工厂方法定义在伴生对象中是可以接受的。在这种情况下，apply 方法应该返回其伴生类的类型。

```scala
object TreeNode {
  // 下面这种定义是 OK 的
  def apply(name: String): TreeNode = ...

  // 不要像下面那样定义，因为它没有返回其伴生类的类型：TreeNode
  def apply(name: String): String = ...
}
```


### <A name='override_modifier'>override 修饰符</a>

无论是覆盖具体的方法还是实现抽象的方法，始终都为方法加上 override 修饰符。实现抽象方法时，不加 override 修饰符，Scala 编译器也不会报错。即便如此，我们也应该始终把 override 修饰符加上，以此显式地表示覆盖行为。以此避免由于方法签名不同（而你也难以发现）而导致没有覆盖到本应覆盖的方法。

```scala
trait Parent {
  def hello(data: Map[String, String]): Unit = {
    print(data)
  }
}

class Child extends Parent {
  import scala.collection.Map

  // 下面的方法没有覆盖 Parent.hello,
  // 因为两个 Map 的类型是不同的。
  // 如果我们加上 override 修饰符，编译器就会帮你找出问题并报错。
  def hello(data: Map[String, String]): Unit = {
    print("This is supposed to override the parent method, but it is actually not!")
  }
}
```



### <a name='destruct_bind'>解构绑定</a>

解构绑定（有时也叫元组提取）是一种在一个表达式中为两个变量赋值的便捷方式。

```scala
val (a, b) = (1, 2)
```

然而，请不要在构造函数中使用它们，尤其是当 `a` 和 `b` 需要被标记为 `transient` 的时候。Scala 编译器会产生一个额外的 Tuple2 字段，而它并不是暂态的（transient）。

```scala
class MyClass {
  // 以下代码无法 work，因为编译器会产生一个非暂态的 Tuple2 指向 a 和 b
  @transient private val (a, b) = someFuncThatReturnsTuple2()
}
```


### <a name='call_by_name'>按名称传参</a>

__避免使用按名传参__. 显式地使用 `() => T` 。

背景：Scala 允许按名称来定义方法参数，例如：以下例子是可以成功执行的：

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

在上面的代码中，`inc()` 以闭包的形式传递给 `print` 函数，并且在 `print` 函数中被执行了两次，而不是以数值 `1` 传入。按名传参的一个主要问题是在方法调用处，我们无法区分是按名传参还是按值传参。因此无法确切地知道这个表达式是否会被执行（更糟糕的是它可能会被执行多次）。对于带有副作用的表达式来说，这一点是非常危险的。


### <A name='multi-param-list'>多参数列表</a>

__避免使用多参数列表__。它们使运算符重载变得复杂，并且会使不熟悉 Scala 的程序员感到困惑。例如：

```scala
// 避免出现下面的写法！
case class Person(name: String, age: Int)(secret: String)
```

一个值得注意的例外是，当在定义底层库时，可以使用第二个参数列表来存放隐式（implicit）参数。尽管如此，[我们应该避免使用 implicits](#implicits)！


### <a name='symbolic_methods'>符号方法（运算符重载）</a>

__不要使用符号作为方法名__，除非你是在定义算术运算的方法（如：`+`, `-`, `*`, `/`），否则在任何其它情况下，都不要使用。符号化的方法名让人难以理解方法的意图是什么，来看下面两个例子：

```scala
// 符号化的方法名难以理解
channel ! msg
stream1 >>= stream2

// 下面的方法意图则不言而喻
channel.send(msg)
stream1.join(stream2)
```


### <a name='type_inference'>类型推导</a>

Scala 的类型推导，尤其是左侧类型推导以及闭包推导，可以使代码变得更加简洁。尽管如此，也有一些情况我们是需要显式地声明类型的：

- __公有方法应该显式地声明类型__，编译器推导出来的类型往往会使你大吃一惊。
- __隐式方法应该显式地声明类型__，否则在增量编译时，它会使 Scala 编译器崩溃。
- __如果变量或闭包的类型并非显而易见，请显式声明类型__。一个不错的判断准则是，如果评审代码的人无法在 3 秒内确定相应实体的类型，那么你就应该显式地声明类型。


### <a name='return'>Return 语句</a>

__闭包中避免使用 return__。`return` 会被编译器转成 ``scala.runtime.NonLocalReturnControl`` 异常的 ``try/catch`` 语句，这可能会导致意外行为。请看下面的例子：

  ```scala
  def receive(rpc: WebSocketRPC): Option[Response] = {
    tableFut.onComplete { table =>
      if (table.isFailure) {
        return None // 不要这样做！
      } else { ... }
    }
  }
  ```

`.onComplete` 方法接收一个匿名闭包并把它传递到一个不同的线程中。这个闭包最终会抛出一个 `NonLocalReturnControl` 异常，并在 __一个不同的线程中__被捕获，而这里执行的方法却没有任何影响。

然而，也有少数情况我们是推荐使用 `return` 的。

- 使用 `return` 来简化控制流，避免增加一级缩进。

  ```scala
  def doSomething(obj: Any): Any = {
    if (obj eq null) {
      return null
    }
    // do something ...
  }
  ```

- 使用 `return` 来提前终止循环，这样就不用额外构造状态标志。

  ```scala
  while (true) {
    if (cond) {
      return
    }
  }
  ```

### <a name='recursion'>递归及尾递归</a>

__避免使用递归__，除非问题可以非常自然地用递归来描述（比如，图和树的遍历）。

对于那些你意欲使之成为尾递归的方法，请加上 `@tailrec` 注解以确保编译器去检查它是否真的是尾递归（你会非常惊讶地看到，由于使用了闭包和函数变换，许多看似尾递归的代码事实并非尾递归）。

大多数的代码使用简单的循环和状态机会更容易推理，使用尾递归反而可能会使它更加繁琐且难以理解。例如，下面的例子中，命令式的代码比尾递归版本的代码要更加易读：

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

__避免使用 implicit__，除非：

- 你在构建领域特定的语言（DSL）
- 你在隐式类型参数中使用它（如：`ClassTag`，`TypeTag`）
- 你在你自己的类中使用它（意指不要污染外部空间），以此减少类型转换的冗余度（如：Scala 闭包到 Java 闭包的转换）。

当使用 implicit 时，我们应该确保另一个工程师可以直接理解使用语义，而无需去阅读隐式定义本身。Implicit 有着非常复杂的解析规则，这会使代码变得极其难以理解。Twitter 的 Effective Scala 指南中写道：「如果你发现你在使用 implicit，始终停下来问一下你自己，是否可以在不使用 implicit 的条件下达到相同的效果」。

如果你必需使用它们（比如：丰富 DSL），那么不要重载隐式方法，即确保每个隐式方法有着不同的名字，这样使用者就可以选择性地导入它们。

```scala
// 别这么做，这样使用者无法选择性地只导入其中一个方法。
object ImplicitHolder {
  def toRdd(seq: Seq[Int]): RDD[Int] = ...
  def toRdd(seq: Seq[Long]): RDD[Long] = ...
}

// 应该将它们定义为不同的名字：
object ImplicitHolder {
  def intSeqToRdd(seq: Seq[Int]): RDD[Int] = ...
  def longSeqToRdd(seq: Seq[Long]): RDD[Long] = ...
}
```


## <a name='exception'>异常处理 (Try 还是 try)</a>

- 不要捕获 Throwable 或 Exception 类型的异常。请使用 `scala.util.control.NonFatal`：

  ```scala
  try {
    ...
  } catch {
    case NonFatal(e) =>
      // 异常处理；注意 NonFatal 无法匹配 InterruptedException 类型的异常
    case e: InterruptedException =>
      // 处理 InterruptedException
  }
  ```
  这能保证我们不会去捕获 `NonLocalReturnControl` 异常（正如在[Return 语句](#return)中所解释的）。

- 不要在 API 中使用 `Try`，即，不要在任何方法中返回 Try。对于异常执行，请显式地抛出异常，并使用 Java 风格的 try/catch 做异常处理。

  背景资料：Scala 提供了单子（monadic）错误处理（通过 `Try`，`Success` 和 `Failure`），这样便于做链式处理。然而，根据我们的经验，发现使用它通常会带来更多的嵌套层级，使得代码难以阅读。此外，对于预期错误还是异常，在语义上常常是不明晰的。因此，我们不鼓励使用 `Try` 来做错误处理，尤其是以下情况：

  一个人为的例子：

  ```scala
  class UserService {
    /** 在用户数据库中查找用户信息。 */
    def get(userId: Int): Try[User]
  }
  ```
  以下的写法会更好：

  ```scala
  class UserService {
    /**
     * 在用户数据库中查找用户信息。
     * @return None 如果查找不到用户
     * @throws DatabaseConnectionException 当连接数据库发生异常时
     */
    @throws(DatabaseConnectionException)
    def get(userId: Int): Option[User]
  }
  ```
  第二种写法非常明显地能让调用者知道需要处理哪些错误情况。


### <a name='option'>Options</a>

- 如果一个值可能为空，那么请使用 `Option`。相对于 `null`，`Option` 显式地表明了一个 API 的返回值可能为空。
- 构造 `Option` 值时，请使用 `Option` 而非 `Some`，以防那个值为 `null`。

  ```scala
  def myMethod1(input: String): Option[String] = Option(transform(input))

  // This is not as robust because transform can return null, and then
  // myMethod2 will return Some(null).
  def myMethod2(input: String): Option[String] = Some(transform(input))
  ```
- 不要使用 None 来表示异常，有异常时请显式抛出。
- 不要在一个 `Option` 值上直接调用 `get` 方法，除非你百分百确定那个 `Option` 值不是 `None`。


### <a name='chaining'>单子链接</a>

单子链接是 Scala 的一个强大特性。Scala 中几乎一切都是单子（如：集合，Option，Future，Try 等），对它们的操作可以链接在一起。这是一个非常强大的概念，但你应该谨慎使用，尤其是：

- 避免链接（或嵌套）超过 3 个操作。
- 如果需要花超过 5 秒钟来理解其中的逻辑，那么你应该尽量去想想有没什么办法在不使用单子链接的条件下来达到相同的效果。一般来说，你需要注意的是：不要滥用 `flatMap` 和 `fold`。
- 链接应该在 flatMap 之后断开（因为类型发生了变化）。

通过给中间结果显式地赋予一个变量名，将链接断开变成一种更加过程化的风格，能让单子链接更加易于理解。来看下面的例子：

```scala
class Person(val data: Map[String, String])
val database = Map[String, Person]
// 有时客户端会给 address 赋予一个 null 值，因此下面的代码用了 Option.apply 来处理这种情况

// A monadic chaining approach
def getAddress(name: String): Option[String] = {
  database.get(name).flatMap { elem =>
    elem.data.get("address")
      .flatMap(Option.apply)  // 处理 null 值
  }
}

// 尽管代码会长一些，但以下方法可读性更高
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


## <a name='concurrency'>并发</a>

### <a name='concurrency-scala-collection'>Scala concurrent.Map</a>

__优先考虑使用 `java.util.concurrent.ConcurrentHashMap` 而非 `scala.collection.concurrent.Map`__。尤其是 `scala.collection.concurrent.Map` 中的 `getOrElseUpdate` 方法要慎用，它并非原子操作（这个问题在 Scala 2.11.16 中 fix 了：[SI-7943](https://issues.scala-lang.org/browse/SI-7943)）。由于我们做的所有项目都需要在 Scala 2.10 和 Scala 2.11 上使用，因此要避免使用 `scala.collection.concurrent.Map`。


### <a name='concurrency-sync-vs-map'>显式同步 vs 并发集合</a>

有 3 种推荐的方法来安全地并发访问共享状态。__不要混用它们__，因为这会使程序变得难以推理，并且可能导致死锁。

- `java.util.concurrent.ConcurrentHashMap`：当所有的状态都存储在一个 map 中，并且有高程度的竞争时使用。

  ```scala
  private[this] val map = new java.util.concurrent.ConcurrentHashMap[String, String]
  ```

- `java.util.Collections.synchronizedMap`：使用情景：当所有状态都存储在一个 map 中，并且预期不存在竞争情况，但你仍想确保代码在并发下是安全的。如果没有竞争出现，JVM 的 JIT 编译器能够通过偏置锁（biased locking）移除同步开销。

  ```scala
  private[this] val map = java.util.Collections.synchronizedMap(new java.util.HashMap[String, String])
  ```

- 通过同步所有临界区进行显式同步，可用于监视多个变量。与 2 相似，JVM 的 JIT 编译器能够通过偏置锁（biased locking）移除同步开销。

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

注意，对于 case 1 和 case 2，不要让集合的视图或迭代器从保护区域逃逸。这可能会以一种不明显的方式发生，比如：返回了 `Map.keySet` 或 `Map.values`。如果需要传递集合的视图或值，生成一份数据拷贝再传递。

  ```scala
  val map = java.util.Collections.synchronizedMap(new java.util.HashMap[String, String])

  // 这是有问题的！
  def values: Iterable[String] = map.values

  // 应用使用下面的写法，把元素拷贝一份。
  def values: Iterable[String] = map.synchronized { Seq(map.values: _*) }
  ```

### <a name='concurrency-sync-vs-atomic'>显式同步 vs 原子变量 vs @volatile</a>

`java.util.concurrent.atomic` 包提供了对基本类型的无锁访问，比如：`AtomicBoolean`, `AtomicInteger` 和 `AtomicReference`。

始终优先考虑使用原子变量而非 `@volatile`，它们是相关功能的严格超集并且从代码上看更加明显。原子变量的底层实现使用了 `@volatile`。

优先考虑使用原子变量而非显式同步的情况：（1）一个对象的所有临界区更新都被限制在单个变量里并且预期会有竞争情况出现。原子变量是无锁的并且允许更为有效的竞争。（2）同步被明确地表示为 `getAndSet` 操作。例如：

  ```scala
  // good: 明确又有效地表达了下面的并发代码只执行一次
  val initialized = new AtomicBoolean(false)
  ...
  if (!initialized.getAndSet(true)) {
    ...
  }

  // poor: 下面的同步就没那么明晰，而且会出现不必要的同步
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

### <a name='concurrency-private-this'>私有字段</a>

注意，`private` 字段仍然可以被相同类的其它实例所访问，所以仅仅通过 `this.synchronized`（或 `synchronized`）来保护它从技术上来说是不够的，不过你可以通过 `private[this]` 修饰私有字段来达到目的。

```scala
// 以下代码仍然是不安全的。
class Foo {
  private var count: Int = 0
  def inc(): Unit = synchronized { count += 1 }
}

// 以下代码是安全的。
class Foo {
  private[this] var count: Int = 0
  def inc(): Unit = synchronized { count += 1 }
}
```


### <a name='concurrency-isolation'>隔离</a>

一般来说，并发和同步逻辑应该尽可能地被隔离和包含起来。这实际上意味着：

- 避免在 API 层面、面向用户的方法以及回调中暴露同步原语。
- 对于复杂模块，创建一个小的内部模块来包含并发原语。


## <a name='perf'>性能</a>

对于你写的绝大多数代码，性能都不应该成为一个问题。然而，对于一些性能敏感的代码，以下有一些小建议：

### <a name='perf-microbenchmarks'>Microbenchmarks</a>

由于 Scala 编译器和 JVM JIT 编译器会对你的代码做许多神奇的事情，因此要写出一个好的微基准程序（microbenchmark）是极其困难的。更多的情况往往是你的微基准程序并没有测量你想要测量的东西。

如果你要写一个微基准程序，请使用 [jmh](http://openjdk.java.net/projects/code-tools/jmh/)。请确保你阅读了[所有的样例](http://hg.openjdk.java.net/code-tools/jmh/file/tip/jmh-samples/src/main/java/org/openjdk/jmh/samples/)，这样你才理解微基准程序中「死代码」移除、常量折叠以及循环展开的效果。


### <a name='perf-whileloops'>Traversal 与 zipWithIndex</a>

使用 `while` 循环而非 `for` 循环或函数变换（如：`map`、`foreach`），for 循环和函数变换非常慢（由于虚函数调用和装箱的缘故）。

```scala

val arr = // array of ints
// 偶数位置的数置零
val newArr = list.zipWithIndex.map { case (elem, i) =>
  if (i % 2 == 0) 0 else elem
}

// 这是上面代码的高性能版本
val newArr = new Array[Int](arr.length)
var i = 0
val len = newArr.length
while (i < len) {
  newArr(i) = if (i % 2 == 0) 0 else arr(i)
  i += 1
}
```

### <a name='perf-option'>Option 与 null</a>

对于性能有要求的代码，优先考虑使用 `null` 而不是 `Option`，以此避免虚函数调用以及装箱操作。用 Nullable 注解明确标示出可能为 `null` 的值。

```scala
class Foo {
  @javax.annotation.Nullable
  private[this] var nullableField: Bar = _
}
```

### <a name='perf-collection'>Scala 集合库</a>

对于性能有要求的代码，优先考虑使用 Java 集合库而非 Scala 集合库，因为一般来说，Scala 集合库要比 Java 的集合库慢。

### <a name='perf-private'>private[this]</a>

对于性能有要求的代码，优先考虑使用 `private[this]` 而非 `private`。`private[this]` 生成一个字段而非生成一个访问方法。根据我们的经验，JVM JIT 编译器并不总是会内联 `private` 字段的访问方法，因此通过使用
`private[this]` 来确保没有虚函数调用会更保险。

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


## <a name='java'>与 Java 的互操作性</a>

本节内容介绍的是构建 Java 兼容 API 的准则。如果你构建的组件并不需要与 Java 有交互，那么请无视这一节。这一节的内容主要是从我们开发 Spark 的 Java API 的经历中得出的。


### <a name='java-missing-features'>Scala 中缺失的 Java 特性</a>

以下的 Java 特性在 Scala 中是没有的，如果你需要使用以下特性，请在 Java 中定义它们。然而，需要提醒一点的是，你无法为 Java 源文件生成 ScalaDoc。

- 静态字段
- 静态内部类
- Java 枚举
- 注解


### <a name='java-traits'>Traits 与抽象类</a>

对于允许从外部实现的接口，请记住以下几点：

- 包含了默认方法实现的 trait 是无法在 Java 中使用的，请使用抽象类来代替。
- 一般情况下，请避免使用 trait，除非你百分百确定这个接口即使在未来也不会有默认的方法实现。

```scala
// 以下默认实现无法在 Java 中使用
trait Listener {
  def onTermination(): Unit = { ... }
}

// 可以在 Java 中使用
abstract class Listener {
  def onTermination(): Unit = { ... }
}
```


### <a name='java-type-alias'>类型别名</a>

不要使用类型别名，它们在字节码和 Java 中是不可见的。


### <a name='java-default-param-values'>默认参数值</a>

不要使用默认参数值，通过重载方法来代替。

```scala
// 打破了与 Java 的互操作性
def sample(ratio: Double, withReplacement: Boolean = false): RDD[T] = { ... }

// 以下方法是 work 的
def sample(ratio: Double, withReplacement: Boolean): RDD[T] = { ... }
def sample(ratio: Double): RDD[T] = sample(ratio, withReplacement = false)
```

### <a name='java-multi-param-list'>多参数列表</a>

不要使用多参数列表。

### <a name='java-varargs'>可变参数</a>

- 为可变参数方法添加 `@scala.annotation.varargs` 注解，以确保它能在 Java 中使用。Scala 编译器会生成两个方法，一个给 Scala 使用（字节码参数是一个 Seq），另一个给 Java 使用（字节码参数是一个数组）。

  ```scala
  @scala.annotation.varargs
  def select(exprs: Expression*): DataFrame = { ... }
  ```

- 需要注意的一点是，由于 Scala 编译器的一个 bug（[SI-1459](https://issues.scala-lang.org/browse/SI-1459)，[SI-9013](https://issues.scala-lang.org/browse/SI-9013)），抽象的变参方法是无法在 Java 中使用的。

- 重载变参方法时要小心，用另一个类型去重载变参方法会破坏源码的兼容性。

  ```scala
  class Database {
    @scala.annotation.varargs
    def remove(elems: String*): Unit = ...

    // 当调用无参的 remove 方法时会出问题。
    @scala.annotation.varargs
    def remove(elems: People*): Unit = ...
  }

  // remove 方法有歧义，因此编译不过。
  new Database().remove()
  ```
  一种解决方法是，在可变参数前显式地定义第一个参数：

  ```scala
  class Database {
    @scala.annotation.varargs
    def remove(elems: String*): Unit = ...

    // 以下重载是 OK 的。
    @scala.annotation.varargs
    def remove(elem: People, elems: People*): Unit = ...
  }
  ```


### <a name='java-implicits'>Implicits</a>

不要为类或方法使用 implicit，包括了不要使用 `ClassTag` 和 `TypeTag`。

```scala
class JavaFriendlyAPI {
  // 以下定义对 Java 是不友好的，因为方法中包含了一个隐式参数（ClassTag）。
  def convertTo[T: ClassTag](): T
}
```

### <a name='java-companion-object'>伴生对象，静态方法与字段</a>

当涉及到伴生对象和静态方法/字段时，有几件事情是需要注意的：

- 伴生对象在 Java 中的使用是非常别扭的（伴生对象 `Foo` 会被定义为 `Foo$` 类内的一个类型为 `Foo$` 的静态字段 `MODULE$`）。

  ```scala
  object Foo

  // 等价于以下的 Java 代码
  public class Foo$ {
    Foo$ MODULE$ = // 对象的实例化
  }
  ```
  如果非要使用伴生对象，可以在一个单独的类中创建一个 Java 静态字段。

- 不幸的是，没有办法在 Scala 中定义一个 JVM 静态字段。请创建一个 Java 文件来定义它。
- 伴生对象里的方法会被自动转成伴生类里的静态方法，除非方法名有冲突。确保静态方法正确生成的最好方式是用 Java 写一个测试文件，然后调用生成的静态方法。

  ```scala
  class Foo {
    def method2(): Unit = { ... }
  }

  object Foo {
    def method1(): Unit = { ... }  // 静态方法 Foo.method1 会被创建（字节码）
    def method2(): Unit = { ... }  // 静态方法 Foo.method2 不会被创建
  }

  // FooJavaTest.java (in test/scala/com/databricks/...)
  public class FooJavaTest {
    public static void compileTest() {
      Foo.method1();  // 正常编译
      Foo.method2();  // 编译失败，因为 method2 并没有生成
    }
  }
  ```

- 样例对象（case object） MyClass 的类型并不是 MyClass。

  ```scala
  case object MyClass

  // Test.java
  if (MyClass$.MODULE instanceof MyClass) {
    // 上述条件始终为 false
  }
  ```
  要实现正确的类型层级结构，请定义一个伴生类，然后用一个样例对象去继承它：

  ```scala
  class MyClass
  case object MyClass extends MyClass
  ```

## <a name='testing'>测试</a>

### <a name='testing-intercepting'>异常拦截</a>

当测试某个操作（比如用无效的参数调用一个函数）是否会抛出异常时，对于抛出的异常类型指定得越具体越好。你不应该简单地使用 `intercept[Exception]` 或 `intercept[Throwable]`（ScalaTest 语法），这能拦截任意异常，只能断言有异常抛出，而不能确定是什么异常。这样做在测试中能捕获到代码中的异常并且通过测试，然而却没真正检验你想验证的行为。


  ```scala
  // 不要使用下面这种方式
  intercept[Exception] {
    thingThatThrowsException()
  }

  // 这才是推荐的做法
  intercept[MySpecificTypeOfException] {
    thingThatThrowsException()
  }
  ```

如果你无法指定代码会抛出的异常的具体类型，说明你这段代码可能写得不好，需要重构。这种情况下，你要么测试更底层的代码，要么改写代码令其抛出类型更加具体的异常。


## <a name='misc'>其它</a>

### <a name='misc_currentTimeMillis_vs_nanoTime'>优先使用 nanoTime 而非 currentTimeMillis</a>

当要计算*持续时间*或者检查*超时*的时候，避免使用 `System.currentTimeMillis()`。请使用 `System.nanoTime()`，即使你对亚毫秒级的精度并不感兴趣。

`System.currentTimeMillis()` 返回的是当前的时钟时间，并且会跟进系统时钟的改变。因此，负的时钟调整可能会导致超时而挂起很长一段时间（直到时钟时间赶上先前的值）。这种情况可能发生在网络已经中断一段时间，ntpd 走过了一步之后。最典型的例子是，在系统启动的过程中，DHCP 花费的时间要比平常的长。这可能会导致非常难以理解且难以重现的问题。而 `System.nanoTime()` 则可以保证是单调递增的，与时钟变化无关。

注意事项：

- 永远不要序列化一个绝对的 `nanoTime()` 值或是把它传递给另一个系统。绝对的 `nanoTime()` 值是无意义的、与系统相关的，并且在系统重启时会重置。
- 绝对的 `nanoTime()` 值并不保证总是正数（但 `t2 - t1` 能确保总是产生正确的值）。
- `nanoTime()` 每 292 年就会重新计算起。所以，如果你的 Spark 任务需要花非常非常非常长的时间，你可能需要别的东西来处理了：）


### <a name='misc_uri_url'>优先使用 URI 而非 URL</a>

当存储服务的 URL 时，你应当使用 `URI` 来表示。

`URL` 的[相等性检查](http://docs.oracle.com/javase/7/docs/api/java/net/URL.html#equals(java.lang.Object))实际上执行了一次网络调用（这是阻塞的）来解析 IP 地址。`URI` 类在表示能力上是 `URL` 的超集，并且它执行的是字段的相等性检查。

### <a name='misc_well_tested_method'>优先使用现存的经过良好测试的方法而非重新发明轮子</a>

当存在一个已经经过良好测试的方法，并且不会存在性能问题，那么优先使用这个方法。重新实现它可能会引入Bug，同时也需要花费时间来进行测试（也可能我们甚至忘记去测试这个方法！）。

  ```scala
  val beginNs = System.nanoTime()
  // Do something
  Thread.sleep(1000)
  val elapsedNs = System.nanoTime() - beginNs

  // 不要使用下面这种方式。这种方法容易出错
  val elapsedMs = elapsedNs / 1000 / 1000

  // 推荐方法：使用Java TimeUnit API
  import java.util.concurrent.TimeUnit
  val elapsedMs2 = TimeUnit.NANOSECONDS.toMillis(elapsedNs)

  // 推荐方法：使用Scala Duration API
  import scala.concurrent.duration._
  val elapsedMs3 = elapsedNs.nanos.toMillis
  ```

例外：
- 使用现存的方法需要引入新的依赖。如果一个方法特别简单，比起引入一个新依赖，重新实现它通常更好。但是记得进行测试。
- 现存的方法没有针对我们的用法进行优化，性能达不到要求。但是首先做一下benchmark, 避免过早优化。
