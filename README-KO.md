# Databricks Scala Guide

800명이 넘는 contributor를 가진 Spark는 빅데이터 분야에서 가장 큰 오픈소스 프로젝트이며, Scala를 사용하는 프로젝트 중 가장 활성화된 프로젝트 입니다. 이 가이드라인은 [Databricks](http://databricks.com/) 엔지니어 팀 뿐 아니라, Spark에 기여하는 엔지니어들의 경험을 바탕으로 만들어 졌습니다.

코드는 저자에 의해 __한 번 쓰여지지만__, 많은 다른 엔지니어들은 그 같은 코드를 __반복적으로 수정하고 읽습니다__. 대부분의 버그들은 보통 코드의 변경으로부터 나옵니다. 그래서 우리는 코드의 가독성과 유지 보수성을 향상시키기 위해 우리의 코드를 최적화 해야합니다. 이를 위한 최선의 방법은 간단한 코드를 작성하는 것입니다.

Scala는 매우 강력하며 여러가지 페러다임에 적용 가능한 언어입니다. 우리는 아래의 가이드라인을 통해 여러가지 프로젝트를 빠른 속도로 진행하고 있습니다. 팀이나 회사의 요구사항 등에 따라서 일부 다르게 적용 해야 할 수도 있습니다.


## <a name='TOC'>목차</a>

  0. [문서 역사](#history)
  1. [구문 스타일](#syntactic)
    - [명명 규칙](#naming)
    - [라인 길이](#linelength)
    - [30 규칙](#rule_of_30)
    - [공백 및 들여쓰기](#indent)
    - [빈 줄](#blanklines)
    - [괄호](#parentheses)
    - [중괄호](#curly)
    - [Long 정수](#long_literal)
    - [문서 스타일](#doc)
    - [클래스 내의 순서](#ordering_class)
    - [Imports](#imports)
    - [패턴 매칭](#pattern-matching)
    - [중위 표기](#infix)
  2. [Scala 언어의 기능](#lang)
    - [apply 함수](#apply_method)
    - [override 수정자](#override_modifier)
    - [튜플 추출](#destruct_bind)
    - [Call by Name](#call_by_name)
    - [다중 매개 변수 표기](#multi-param-list)
    - [특수 문자 함수 (오퍼레이터 오버로딩)](#symbolic_methods)
    - [타입 추론](#type_inference)
    - [Return 예약어](#return)
    - [재귀 용법과 꼬리 재귀 용법](#recursion)
    - [Implicits](#implicits)
    - [예외 처리, i.e. Try vs try](#exception)
    - [Options](#option)
    - [모나드 채이닝](#chaining)
  3. [동시성 제어](#concurrency)
    - [Scala concurrent.Map](#concurrency-scala-collection)
    - [동기화 (synchronized) 명시 vs Java 제공 동시성 라이브러리](#concurrency-sync-vs-map)
    - [동기화 (synchronized) 명시 vs Atomic 변수 vs @volatile](#concurrency-sync-vs-atomic)
    - [Private 변수](#concurrency-private-this)
    - [동시성 로직 분리](#concurrency-isolation)
  4. [성능](#perf)
    - [Microbenchmarks](#perf-microbenchmarks)
    - [순회와 zipWithIndex](#perf-whileloops)
    - [Option과 null](#perf-option)
    - [Scala Collection 라이브러리](#perf-collection)
    - [private[this]](#perf-private)
  5. [Java 호환성](#java)
    - [Scala에서 사용 할 수 없는 Java 기능](#java-missing-features)
    - [Traits와 Abstract 클래스](#java-traits)
    - [Type 별칭](#java-type-alias)
    - [기본 매개변수 값](#java-default-param-values)
    - [다중 매개변수 표기](#java-multi-param-list)
    - [가변인자](#java-varargs)
    - [Implicits](#java-implicits)
    - [관련 객체, 정적 함수 및 변수](#java-companion-object)
  6. [기타](#misc)
    - [currentTimeMillis 보다는 nanoTime](#misc_currentTimeMillis_vs_nanoTime)
    - [URL 보다는 URI](#misc_uri_url)



## <a name='history'>문서 역사</a>
- 2015-03-16: 초기 버전.
- 2015-05-25: [override 수정자](#override_modifier) 섹션 추가.
- 2015-08-23: "do NOT"에서 "avoid"으로 심각도 낮춤.
- 2015-11-17: [apply 함수](#apply_method) 섹션 추가:  한 객체의 apply 함수는 그 객체와 같은 이름을 가진 클래스를 반환해야 합니다.
- 2015-11-17: 이 가이드라인이 [중국어로 번역되었습니다](README-ZH.md). 중국어 번역은 커뮤니티 맴버인 [Hawstein](https://github.com/Hawstein) 이 했습니다. 이 문서의 최신성을 보장하지 않습니다.
- 2015-12-14: 이 가이드라인이 [중국어로 번역되었습니다](README-KO.md). 한국어 번역은 [HyukjinKwon](https://github.com/HyukjinKwon) 이 했습니다. 이 문서의 최신성을 보장하지 않습니다.

## <a name='syntactic'>구문 스타일</a>

### <a name='naming'>명명 규칙</a>

우리는 주로 Java와 Scala의 표준 명명 규칙을 따릅니다.

- Class, trait, 객체는 명명규칙 즉 낙타등 표기법(PascalCase) 을 따라야 합니다.
  ```scala
  class ClusterManager

  trait Expression
  ```

- Package는 Java의 명명 규칙을 따라야 합니다. 모두 소문자로 ASCII 문자를 사용합니다.
  ```scala
  package com.databricks.resourcemanager
  ```

- 메소드/함수는 낙타등 표기법 (PascalCase)을 사용해야 합니다.

- 모든 상수는 대문자로 표기 하고, 연관된 객체에 배치합니다.
  ```scala
  object Configuration {
    val DEFAULT_PORT = 10000
  }
  ```

- Enum은 낙타등 표기법 (PascalCase)을 따라야 합니다.

- Annotation 또한 낙타등 표기법 (PascalCase)을 따라야 합니다. 이 가이드라인이 Scala의 공식 가이드라인과 다름을 주의하시기 바랍니다.
  ```scala
  final class MyAnnotation extends StaticAnnotation
  ```


### <a name='linelength'>라인 길이</a>

- 라인 길이는 100자를 넘지 않습니다.
- 단, import나 URL의 경우는 예외입니다. (그렇다 하더라도 100자의 제약을 지켜주도록 합니다).


### <a name='rule_of_30'>30 규칙</a>

"한 개의 엘리먼트가 30개 이상의 하위 엘리먼트를 포함 하고 있다면, 심각한 문제가 있을 가능성이 높다." - [Refactoring in Large Software Projects](http://www.amazon.com/Refactoring-Large-Software-Projects-Restructurings/dp/0470858923).

일반적으로:

- 함수는 30줄 이상의 라인을 초과하지 않아야 합니다.
- 하나의 클래스당 30개 이상의 함수를 갖지 않도록 합니다.


### <a name='indent'>공백 및 들여쓰기</a>

- 2칸 공백 들여쓰기를 합니다.
  ```scala
  if (true) {
    println("Wow!")
  }
  ```

- 함수 선언에서 파라메터가 한 줄에 맞지 않아 들여쓰기를 하는 경우, 4칸 공백을 합니다. 반환 타입은 다음 줄에 배치되거나 같은 라인에 배치될 수 있습니다. 같은 라인에 쓰는 경우, 2칸 들여쓰기를 합니다.
  ```scala
  def newAPIHadoopFile[K, V, F <: NewInputFormat[K, V]](
      path: String,
      fClass: Class[F],
      kClass: Class[K],
      vClass: Class[V],
      conf: Configuration = hadoopConfiguration): RDD[(K, V)] = {
    // method body
  }

  def newAPIHadoopFile[K, V, F <: NewInputFormat[K, V]](
      path: String,
      fClass: Class[F],
      kClass: Class[K],
      vClass: Class[V],
      conf: Configuration = hadoopConfiguration)
    : RDD[(K, V)] = {
    // method body
  }
  ```

- Class의 해더가 한 줄에 맞지 않을 때는 2칸 공백 들여쓰기를 합니다. 그리고 그 해더 뒤에 한 개의 빈 줄을 입력 합니다.
  ```scala
  class Foo(
      val param1: String,  // 4 space indent for parameters
      val param2: String,
      val param3: Array[Byte])
    extends FooInterface  // 2 space here
    with Logging {

    def firstMethod(): Unit = { ... }  // blank line above
  }
  ```

- 수직 정렬을 사용하지 않습니다. 이것은 중요치 않은 코드에 집중하게 하고, 차후에 코드 수정을 어렵게 만듭니다.
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


### <a name='blanklines'>빈 줄</a>

- 빈 줄은 아래의 경우에 사용합니다:
    - 연속되는 변수, 생성자, 함수 또는 내부 클래스들 사이 빈 줄이 삽입 될 수 있습니다.
      - 예외: 연속되는 변수 선언 사이 아무런 코드도 없다면 빈 줄은 옵션입니다. 이런 빈 줄들은 논리적인 그룹을 만들 때 사용 될 수 있습니다.
  - 함수 안에서 빈 줄을 삽입하여 논리적인 그룹을 만들 수 있습니다.
  - 첫 번째 맴버 앞이나 마지막 맴버 뒤에 빈 줄이 있을 수 있습니다.
- 한개 또는 두개의 빈 줄을 사용하여 Class 선언들을 분리합니다.
- 과도한 수의 빈 줄은 권장하지 않습니다.


### <a name='parentheses'>괄호</a>

- I/O 접근이나 상태 변형에 대한 접근을 갖고 있거나 side-effect를 줄 수 있는 함수는 괄호와 함께 선언되어야 합니다.
  ```scala
  class Job {
    // Wrong: killJob changes state. Should have ().
    def killJob: Unit

    // Correct:
    def killJob(): Unit
  }
  ```

- 함수 호출자는 반드시 함수의 정의를 따라야 합니다. 예를 들어, 함수가 괄호 없이 선언되었다면 괄호 없이 호출되어야 합니다. 이 것은 단지 문법적인 문제일 뿐만 아니라 `apply`를 호출 할 때에도 문제가 될 수 있습니다.
 
  ```scala
  class Foo {
    def apply(args: String*): Int
  }

  class Bar {
    def foo: Foo
  }

  new Bar().foo  // This returns a Foo
  new Bar().foo()  // This returns an Int!
  ```


### <a name='curly'>중괄호</a>

한 줄 조건부 식이나 순환문에도 중괄호를 넣어야 합니다. 단, if/else문의 경우에는 한 줄로 표기 하거나, side-effect가 없는 3항 연산자로 표기 할 수 있습니다.

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


### <a name='long_literal'>Long 정수</a>

Long 정수의 접미사는 `L`로 사용합니다. 이는 가끔 `l`과 `1`을 구분하기 힘들 때가 있기 때문입니다.

```scala
val longValue = 5432L  // Do this

val longValue = 5432l  // Do NOT do this
```


### <a name='doc'>문서 스타일</a>

Scala 주석 스타일 대신 Java 주석 스타일을 따릅니다.
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


### <a name='ordering_class'>클래스 내의 순서</a>

만약 Class의 정의가 길고 많은 함수들을 포함하고 있다면, 논리적으로 분할 하고, 아래와 같은 주석 헤더를 이용하여 구분 합니다.
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

물론, 이 예와 같은 Class의 길이는 권장하지 않습니다. 일반적으로 내부 구현이 아닌 공개되어있는 API를 만들 때 위와 같은 형식 사용 됩니다.

### <a name='imports'>Imports</a>

- __와일드 카드를 이용한 import는 피하도록 합니다__. 단, 6개 이상 같은 페키지에서 import하는 경우 혹은 implicit 함수들을 import하는 경우는 허용 됩니다. 와일드카드 import는 외부(import 된 페키지)의 변화에 약할 수 있습니다.
- import를 할 때는 상대 경로가 아닌 절대 경로를 사용합니다. 예를 들어 상대경로 `util.Random` 가 아닌`scala.util.Random` 을 사용합니다.
- 또한, import는 아래와 같은 순서로 정렬해야 합니다:
  * `java.*` 와 `javax.*`
  * `scala.*`
  * Third-party 라이브러리 (`org.*`, `com.*`, etc)
  * 프로젝트 페키지 (`com.databricks.*` 혹은 Spark에서 작업하는 경우 `org.apache.spark`)
- 각각의 그룹 안에서, import는 알파벳 순서로 정렬 합니다.
- IntelliJ의 import 최적화 기능을 사용하여 자동으로 할 수 있습니다. 아래와 같은 config를 적용합니다:


  ```
  java
  javax
  _______ blank line _______
  scala
  _______ blank line _______
  all other imports
  _______ blank line _______
  com.databricks  // or org.apache.spark if you are working on Spark
  ```


### <a name='pattern-matching'>패턴 매칭</a>

- 함수 전체가 패턴 매칭을 하는 함수라면 `match` 를 함수의 정의로써 같은 줄에 놓습니다. 이와 같이 들여쓰기의 레벨을 한단계 줄이도록 합니다.
  ```scala
  def test(msg: Message): Unit = msg match {
    case ...
  }
  ```

- 함수를 호출 할 때, 아래와 같은 중괄호 안에 (혹은 partial function 안에) 한 개의 `case` 만 있다면, 같은 줄에 넣어 함수 호출을 합니다.
  ```scala
  list.zipWithIndex.map { case (elem, i) =>
    // ...
  }
  ```
  만약 여러 개의 `case` 문이 존재한다면 아래와 같이 들여쓰기를 합니다.
  ```scala
  list.map {
    case a: Foo =>  ...
    case b: Bar =>  ...
  }
  ```


### <a name='infix'>중위 표기</a>

특수 문자 함수 (symbolic methods)를 제외하고는 __중위 표기를 피합니다__.
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


## <a name='lang'>Scala 언어의 기능</a>


### <a name='apply_method'>apply 함수</a>

Class 안에서의 apply 함수는 코드의 가독성을 저하 시킵니다. 특히, Scala에 익숙하지 않은 사람들에게는 더욱 생소 할수 있습니다. 또한 IDE가 호출을 따라가기 어렵게 만듭니다. 최악의 경우, [괄호](#parentheses) 항목의 예제에서 보이듯이 예상치 못 한 방향으로 코드의 정확성에 영향을 미칠 수 있습니다.

같은 이름을 갖는 객체에 펙토리 패턴으로써 apply 함수를 정의하는 것은 괜찮습니다. 이런 경우, apply 함수는 같은 이름의 class타입의 객체를 리턴해야 합니다.
```scala
object TreeNode {
  // This is OK
  def apply(name: String): TreeNode = ...

  // This is bad because it does not return a TreeNode
  def apply(name: String): String = ...
}
```


### <a name='override_modifier'>override 수정자</a>
항상 함수를 위한 override 수정자는 추상 함수를 오버라이드하는 경우이건 실제 함수를 오버라이드 하는 경우이건 항상 붙여줘야 합니다. Scala 컴파일러는 `override` 추상 함수들에 있어서는 수정자를 요구하지는 않습니다. 하지만 우리는 함수를 위한 override 수정자는 추상 함수를 오버라이드 하든 실제 함수를 오버라이드 하든 항상 붙여줘야 합니다.

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



### <a name='destruct_bind'>튜플 추출</a>

튜플 추출은 (바인딩 제거) 두 개의 변수를 하나의 표현식에서 선언 및 대입 할 수 있는 편한 방법입니다.
```scala
val (a, b) = (1, 2)
```

그러나 생성자에서 이를 사용하지 말아야 합니다 (특히 `a` 와 `b` 가 `transient`의 어노테이션으로 표기되어 있는 경우). Scala 컴파일러는 여분의 Tuple2 필드를 하나 생성하게 되는데 이는 `transient`가 아래 예제에서 적용되지 않습니다.   
```scala
class MyClass {
  // This will NOT work because the compiler generates a non-transient Tuple2
  // that points to both a and b.
  @transient private val (a, b) = someFuncThatReturnsTuple2()
}
```


### <a name='call_by_name'>Call by Name</a>

__Call by name 은 피하도록 합니다__. `() => T`을 명시적으로 사용합니다.

배경: Scala는 함수의 인자가 by-name으로 정의되는 것을 허용합니다. 예를 들어 아래와 같은 코드는 정상적으로 작동 합니다.
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

위의 예제에서는 `inc()`가 `print`에게 값 `1`이 아닌 함수 (closure) 로써 전달됩니다. 그리고 `print`에서 두 번 실행이 됩니다. 여기서 문제점은 호출하는 쪽에서는 call-by-name과 call-by-value를 구분 할 수 없다는 것 입니다. 따라서, 이 표현이 print가 호출 되기 전에 실행되었는지 아닌지(혹은 여러번 실행이 될 것이라는 것 까지도)를 확신 할 수 없게 됩니다. 이 것은 근본적으로 위험하고 side-effect가 있을 수 있게 됩니다.


### <a name='multi-param-list'>다중 매개 변수 표기</a>

__다중의 변수를 리스트로 묶어 표기하는 것을 피하도록 합니다__. 이는 연산자의 오버로딩을 복잡하게 하고, Scala에 익숙치 않은 개발자들을 헷갈리게 할 수 있습니다.

```scala
// Avoid this!
case class Person(name: String, age: Int)(secret: String)
```

하나의 주의할 예외로는 implicit에서 낮은 레벨의 라이브러리를 정의 할 때 (리스트로 묶은) 사용 되는 두번째 인자 입니다. 하지만 되도록이면 [implicit은 피해야 합니다](#implicits).


### <a name='symbolic_methods'>특수 문자 함수 (오퍼레이터 오버라이딩)</a>

__특수 문자(오퍼레이터) 를 함수 이름으로 사용하지 않아야 합니다__. 단, 사칙연산에 있어서, 기호에 알맞게 작용 하는 경우는 허용합니다. (예를 들어 `+`, `-`, `*`, `/`). 그 외에는 어떤 환경에서도 이렇게 사용 되어선 안됩니다. 이런 함수들이 사용 되는 경우에는 가독성이 매우 떨어지고 함수들을 이해하기 힘들게 됩니다. 아래의 두 가지 예제를 참고하시기 바랍니다:
```scala
// symbolic method names are hard to understand
channel ! msg
stream1 >>= stream2

// self-evident what is going on
channel.send(msg)
stream1.join(stream2)
```


### <a name='type_inference'>타입 추론</a>

Scala의 타입 추론 (특히 left-side 타입 추론) 과 함수 (closure) 추론은 코드를 더 간결하게 만들 수 있습니다. 아래와 같은 몇 가지 경우는 명시적인 타입이 주어져야 합니다: 

- __Public 함수는 명시적으로 타입이 주어져야 합니다__. 그렇지 않다면 컴파일러가 잘못된 타입을 추론 할 수 있습니다.
- __Implicit 함수들은 명시적으로 타입이 주어져야 합니다__. 그렇지 않다면 Scala 컴파일러는 증분 컴파일에서 실패 할 수 있습니다.
- __변수 혹은 타입이 생략된 함수(closure)는 명시적으로 타입이 주어져야합니다__. 좋은 리트머스 테스트는 명시적인 타입들이 사용되어야 합니다. 리뷰어들이 3초 내에 타입을 확인 할 수 없는 경우는 권장되지 않습니다.

### <a name='return'>Return 예약어</a>

__Return을 함수(closure)에 사용하지 않도록 합니다__. `return` 은 컴파일러가 ``scala.runtime.NonLocalReturnControl`` 을 위해 ``try/catch`` 를 하도록 만듭니다. 이것은 예상치 못한 컴파일러의 행동으로 이어질 수 있습니다. 아래 예제를 참고해주시길 바랍니다:
  ```scala
  def receive(rpc: WebSocketRPC): Option[Response] = {
    tableFut.onComplete { table =>
      if (table.isFailure) {
        return None // Do not do that!
      } else { ... }
    }
  }
  ```
  이 `.onComplete` 함수는 익명의 함수(closure) `{ table => ... }`를 받고 그 것을 다른 스레드로 보냅니다. 이 함수(closure)는 결국 `NonLocalReturnControl` 을 내뿜게 되고, 이 것은 __다른 스레드__ 에서 잡히게 됩니다. 이 것은 여기서 실행된 함수에게 아무런 영향을 미치지 않게 됩니다.

그러나 몇 가지 경우에는 `return` 키워드의 사용이 권고됩니다.

- `return` 구문을 사용하여, 한 단계의 들여쓰기 레벨을 추가 하지 않고, 흐름 제어를 단순화 시킵니다.
  ```scala
  def doSomething(obj: Any): Any = {
    if (obj eq null) {
      return null
    }
    // do something ...
  }
  ```

- `return` 구문을 사용하여, 플래그 변수를 만들지 않고, 루프를 일찍 종료 합니다.
  ```scala
  while (true) {
    if (cond) {
      return
    }
  }
  ```

### <a name='recursion'>재귀 용법 과 꼬리 재귀 용법</a>

__재귀는 피하도록 합니다__. 단, 이 문제가 자연적으로 재귀로 해결되어야 하는 경우는 사용합니다(예를 들어, 그래프 순회 혹은 트리 순회). 

꼬리 재귀 용법이 적용되어야 하는 함수에 있어서는, `@tailrec` 어노태이션을 사용합니다. 이는 컴파일러가 이 것이 꼬리 재귀 용법이 적용 되어야 한다는 것을 확인 할 수 있도록 합니다 (사실은, 함수(closure)의 사용과 functional transformation등 으로 많은 꼬리 재귀 용법이 사용 되지 않을 수 있습니다)).

대부분의 코드는 간단한 루프를 통해 추론하는 것이 더 쉽습니다. 꼬리 재귀를 통해 만들어진 함수는 길고 이해하기 어렵습니다. 예를 들어서 아래 예제는 꼬리 재귀용법보다는 간단한 루프를 사용해서 쉽게 만들 수 있습니다.
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

__implicit의 사용은 피하도록 합니다__. 단, 아래 경우에 대해서는 예외일 수 있습니다.
- 도메인-특정-언어(DSL)를 빌드 하는 경우
- 암시적 타입의 인자를 사용하는 경우(예를 들어. `ClassTag`, `TypeTag`)
- 특정 클래스 안에서 타입 변환의 코드를 줄이기 위해 사용되는 경우 (예를 들어,Scala 함수(closure) 에서 Java 함수(closure)로의 변환)

우리는 코드를 작성한 사람이 아닌 다른 개발자가 이 코드를 implicit의 정의를 읽지 않고 이해 할 수 있도록 합니다. implicit은 상당히 복잡하고 코드를 이해하기 어렵게 만듭니다. Twitter의 Scala 가이드라인에서는 이와 같이 얘기합니다:"만약 당신이 implicit을 사용 하고 있다면, 이를 사용 하지 않고 같은 목적을 달성 할수 없는지 확인하세요."

만약 꼭 이를 사용해야 한다면 (예를 들어 DSL을 개선하기 위해), implicit 함수를 오버로드 하지 않습니다. 예를 들어 다른 유저가 손쉽게 골라서 import할 수 있도록 implicit 함수가 중복되지 않는 이름을 갖게 합니다.
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


## <a name='exception'>예외 처리, i.e. )Try vs try</a>

- Throwable 또는 Exception 유형을 다루지 않도록 합니다. `scala.util.control.NonFatal` 를 사용합니다:
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
  이것은 우리가 `NonLocalReturnControl`를 에러 처리 하지 않도록 해 줍니다([Return 예약어](#return) 항목에 설명되어 있는 대로). 

- API 안에서 `Try` 를 사용 하지 않습니다. 예를 들어 어떤 함수에서도 Try를 반환값으로 사용하지 않습니다. 정상적으로 실행되지 않는 경우 명시적으로 예외를 던지고, Java의 try/catch 문을 사용하여 핸들링 하는 것이 권장됩니다. 

  배경: Scala는 `Try`, `Success` 그리고 `Failure`를 통해서 모나딕한 에러 핸들리을 지원합니다. 이는 로직의 체이닝을 가능하게 합니다. 그러나, 이 모나딕한 에러 핸들링은 종종 다중 레벨의 복잡성을 가하고, 코드의 가독성을 저하 시킨다는 것을 경험을 통해 알게 됐습니다. 더군다나, 종종 어느 부분에서 에러가 나오고, 예상치 못 한 예외가 나오는지 알기가 힘듭니다. 그 이유는 `Try` 안에서 이 에러와 예외가 인코딩 되지 않기 때문 입니다. 따라서, 우리는 에러 핸들링을 위해 `Try`의 사용을 권고하지 않습니다. 특히:

  이 예제의 경우:
  ```scala
  class UserService {
    /** Look up a user's profile in the user database. */
    def get(userId: Int): Try[User]
  }
  ```
  이렇게 쓰이는 것이 낫습니다:
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
  두번째는 확실히 어떤 에러를 핸들링 하는지 호출하는 쪽에서 알기가 쉽습니다.


### <a name='option'>Options</a>

- 값이 비어 있을 수 있을 때 `Option`을 사용합니다. `null`과 대조되어, `Option`은 API에서 명시된 대로 `None`값을 갖을 수 있습니다.
- `Option`을 생성 할 때 `Some`보다는 `Option`을 사용하도록 합니다. 이는 `null` 값으로 부터 안전하도록 합니다.
  ```scala
  def myMethod1(input: String): Option[String] = Option(transform(input))

  // This is not as robust because transform can return null, and then
  // myMethod2 will return Some(null).
  def myMethod2(input: String): Option[String] = Some(transform(input))
  ```
- None을 사용하여 예외를 표현하지 않습니다. 대신, 명시적으로 예외를 던집니다.
- `Option`에서의 값을 확신 할 수 있지 않는 이상, `Option`에서 `get`을 명시적으로 호출하지 않습니다.  

### <a name='chaining'>모나드 채이닝</a>

Scala의 강력한 특징중 하나는 모나드 채이닝 입니다. 거의 모든 것(예를 들어 collections, Option, Futrue 혹은 Try) 이 모나드 채이닝을 지원하고 같이 맞물려서 동작 할 수 있습니다. 이 것은 놀라울 정도로 강력한 개념입니다. 하지만 이 채이닝은 함부로 남용되어서는 안됩니다. 특히:

- 3개 이상의 (내부를 포함)모나드 채이닝은 피하도록 합니다. 
- 만약 코드의 논리를 이해하는데 5초 이상이 걸린다면, 모나드 체이닝을 사용 하지 않고, 같은 성과를 이룰수 있는 방법을 생각해 볼 필요가 있습니다. 일반적으로 `flatMap` 혹은 `fold`가 이에 해당 됩니다.
- `flatMap` 후에는 거의 항상 모나드 채이닝을 이어가지 않습니다 (왜냐하면 타입이 바뀌기 때문입니다).

모나드 체인은 종종 명시적으로 타입이 주어진 중간 값을 저장하는 식으로 채이닝을 끊어 더 이해하기 쉽도록 만듭니다. 예를 들어:
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


## <a name='concurrency'>동시성 제어</a>

### <a name='concurrency-scala-collection'>Scala concurrent.Map</a>

__`java.util.concurrent.ConcurrentHashMap` 이  `scala.collection.concurrent.Map` 보다 권장됩니다__. 특히, `scala.collection.concurrent.Map`의 `getOrElseUpdate` 함수는 atomic하지 않습니다 (이는 Scala 2.11.6에서 고쳐졌습니다. [SI-7943](https://issues.scala-lang.org/browse/SI-7943)). 우리가 관리하고 있는 모든 프로젝트에서는 Scala 2.10과 Scala 2.11의 크로스 빌딩을 하기 때문에 `scala.collection.concurrent.Map`의 사용은 피해야 합니다.

### <a name='concurrency-sync-vs-map'>동기화(synchronized) 명시 vs Java 제공 동시성 라이브러리</a>

동시성을 제어하기 위해서 3가지의 추천하는 방법이 있습니다. __섞어서 사용하지 않습니다__, 왜냐하면 이는 프로그램을 더욱 복잡하게 하고 데드락을 일으킬 수 있기 때문입니다.

1. `java.util.concurrent.ConcurrentHashMap`: 모든 상태가 map에 저장 되고, 빈번한 접근이 일어 날 때 사용합니다.
  ```scala
  private[this] val map = new java.util.concurrent.ConcurrentHashMap[String, String]
  ```

2. `java.util.Collections.synchronizedMap`: 모든 상태가 map에 저장되고, 빈번한 접근이 일어나지 않지만 코드를 안전하게 만들고 싶을 때 사용합니다. 만약 아무런 동시성 접근이 일어나지 않는다면, JVM JIT 컴파일러는 동기화의 오버헤드를 지울 수 있습니다.
  ```scala
  private[this] val map = java.util.Collections.synchronizedMap(new java.util.HashMap[String, String])
  ```

3. 명시적으로 동기화를 하는 방법: 이 방법은 여러 변수들을 동시성 제어를 할 수 있도록 합니다. 2번과 비슷하게 JVM JIT 컴파일러가 동기화의 오버헤드를 지울 수 있습니다.
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

1번의 경우와 2번의 경우, 값을 읽거나 이터레이터로 해당 콜랙션에 접근시, 이 보호된 영역에서 빠져나오게 되는 것을 주의 합니다. 이는 종종 `Map.keySet`이나 `Map.values`을 사용 하는 경우 벌어지게 됩니다. 만약 값을 읽거나 값들이 루프를 돌아야 하는 경우, 복사본을 만들어 사용하도록 합니다.
  ```scala
  val map = java.util.Collections.synchronizedMap(new java.util.HashMap[String, String])

  // This is broken!
  def values: Iterable[String] = map.values

  // Instead, copy the elements
  def values: Iterable[String] = map.synchronized { Seq(map.values: _*) }
  ```

### <a name='concurrency-sync-vs-atomic'>동기화(synchronized) 명시 vs Atomic 변수 vs @volatile</a>

`java.util.concurrent.atomic` 페키지는 원시타입을 원자적으로 읽고 쓸 수 있는 API를 제공 합니다(예를 들어 `AtomicBoolean`, `AtomicInteger` 와 `AtomicReference`).

항상 `@volatile` 보다는 이를 사용한 변수들을 사용 하는 것이 권고됩니다. 이들은 많은 기능들을 제공하며, 코드의 가독성을 증가시켜 줍니다. 이 변수들은 내부적으로 `@volatile`을 사용하여 구현되어있습니다.

이 명시적인 동기화 보다는 Atomic 변수를 사용하는 것이 권장되는 경우가 몇 가지 있습니다: (1) 어떤 객체의 모든 중요한 갱신이 하나의 *단일* 변수에 존재 할 때 그리고 동시성 접근이 예상 될 때. 이 변수들은 원자적으로 동작하기 때문에 효과적인 동시성 제어를 제공합니다. 혹은 (2) 동기화가 명확하게 `getAndSet` 함수로 표현 될 수 있을 때. 예를 들어:
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

### <a name='concurrency-private-this'>Private 변수</a>

`private` 변수들이 외부 같은 클래스의 다른 객체들로부터 접근이 가능하다는 것을 주의하시기 바랍니다. 따라서, 이를 `this.synchronized` (혹은 `synchronized`) 으로 보호하는 것은 기술적으로 충분하지 않습니다. 그 대신, `private[this]`를 사용하시기 바랍니다.
```scala
// The following is still unsafe.
class Foo {
  private var count: Int = 0
  def inc(): Unit = synchronized { count + 1 }
}

// The following is safe.
class Foo {
  private[this] var count: Int = 0
  def inc(): Unit = synchronized { count + 1 }
}
```


### <a name='concurrency-isolation'>동시성 로직 분리</a>

일반적으로, 동시성과 동기화 로직은 최대한 분리되고 독립적이어야 합니다. 이 것은 다음을 의미합니다:
 
- API레벨에서 유저에게 노출된 함수나 콜백함수에 이 동기화 변수들을 노출 하는 것을 피합니다.
- 복잡한 모듈에서는 작은 내부 모듈을 만들어 동시성을 위한 변수들을 갖고 있도록 합니다.


## <a name='perf'>성능</a>

대부분의 코드는 보통 성능에 대하여 크게 고려되지 않습니다. 성능 향상을 위한 코드를 위해서 몇 가지 팁이 있습니다.

### <a name='perf-microbenchmarks'>Microbenchmarks</a>

좋은 microbenchmark를 작성하는 것은 아주 어려운 일 입니다, 왜냐하면 Scala 컴파일러와 JVM JIT 컴파일러는 많은 마법과 같은 일을 코드에 하기 때문입니다. 왜냐하면 대게의 경우 microbenchmark는 측정하고자 하는 것을 측정하지 않습니다.

microbenchmark를 작성하려면 [jmh](http://openjdk.java.net/projects/code-tools/jmh/)를 사용하시기 바랍니다. "죽은 코드" 제거와 상수값 대체 그리고 루프 풀기를 이해하기 위해 직접 [모든 샘플](http://hg.openjdk.java.net/code-tools/jmh/file/tip/jmh-samples/src/main/java/org/openjdk/jmh/samples/)을 읽도록 합니다.

### <a name='perf-whileloops'>순회와 zipWithIndex</a>

`for`나 혹은 functional transformations (예를 들어, `map` 혹은 `foreach`) 보다는 `while` 루프를 를 사용하기를 권장합니다. For 루프나 functional transformations은 상당히 느립니다(이유는 가상 함수 호출과 boxing 때문입니다).
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

### <a name='perf-option'>Option 과 null</a>

성능을 고려한 코드를 위해, 가상 함수 호출과 boxing을 피하는 `Option`보다는 `null`의 사용이 권장됩니다. Null을 갖을 수 있는 변수에는 Nullable 이라고 label을 확실히 하도록 합니다.

```scala
class Foo {
  @javax.annotation.Nullable
  private[this] var nullableField: Bar = _
}
```

### <a name='perf-collection'>Scala Collection 라이브러리</a>

성능을 고려한 코드를 위해, Scala의 라이브러리 사용 보다는 Java의 collection 라이브러리 사용이 권장됩니다. 이는 Scala의 라이브러리가  종종 Java 라이브러리 보다 느리기 때문입니다.

### <a name='perf-private'>private[this]</a>

성능을 고려한 코드를 위해, `private` 보다는 `private[this]`이 권장됩니다. `private[this]`는 접근자 함수를 생성하지 않고 하나의 변수만 생성합니다. 우리의 경험으로는 JVM JIT 컴파일러는 항상 `private` 변수를 한 번에 (하나의 정의로) 처리하지 못하였습니다. 따라서 해당 변수에 접근 할 가상 함수 호출을 없애기 위해서 `private[this]` 을 사용하는 것이 더 안전합니다.
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


## <a name='java'>Java 호환성</a>

이 항목은 Java 호환 가능한 API를 만들기 위한 가이드라인을 다루고 있습니다. 이 것은 현재 당신이 만들고 있는 컴포넌트가 Java와의 호환성을 필요로 하지 않는다면 적용되지 않습니다. 이 가이드라인은 주로 우리가 Spark의 Java API를 만드는 과정에서 우리가 경험한 것을 바탕으로 작성되었습니다.

### <a name='java-missing-features'>Scala에서 사용 할 수 없는 Java 기능</a>

아래의 Java특징들은 Scala에 없습니다. 만약 아래의 기능이 필요하다면 Java에서 정의하여 사용하시기 바랍니다. 그러나 Scala 문서를 보시면 Java로 정의된 파일에 대한 보장은 하지 않는다고 명시되어 있습니다.

- Static 변수
- Static 내부 클래스
- Java enum
- Annotation


### <a name='java-traits'>Traits 와 Abstract 클래스</a>

외부에서 구현 될 수 있는 인터페이스의 경우 아래의 항목을 명심하시길 바랍니다:

- Trait에 있는 기본으로 정의되어 있는 함수들은 Java에서 사용 할 수 없습니다. 대신 추상 클래스를 사용하시기 바랍니다.
- 일반적으로 trait의 사용을 피하시길 바랍니다. 단, 인터페이스가 미래의 어떤 경우에도 어떠한 정의된 구현을 사용하지 않는 다는 것을 확신 할 수 있다면 사용 할 수 있습니다.
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


### <a name='java-type-alias'>Type 별칭</a>

별칭을 사용하지 않습니다. 이들은 바이트코드와 Java에서 보여지지 않습니다.


### <a name='java-default-param-values'>기본 매개변수 값</a>

인자에 기본값을 주어 사용하지 않습니다. 대신 함수를 오버로드 합니다.
```scala
// Breaks Java interoperability
def sample(ratio: Double, withReplacement: Boolean = false): RDD[T] = { ... }

// The following two work
def sample(ratio: Double, withReplacement: Boolean): RDD[T] = { ... }
def sample(ratio: Double): RDD[T] = sample(ratio, withReplacement = false)
```

### <a name='java-multi-param-list'>다중 매개변수 표기</a>

여러 인자를 리스트로 묶어 사용하지 않습니다.

### <a name='java-varargs'>가변인자</a>

- varargs 함수가 Java에서 사용 될 수 있도록 `@scala.annotation.varargs` 어노테이션을 적용합니다. Scala 컴파일러는 하나는 Scala를 위해(바이트코드 인자는 Seq 입니다) 다른 하나는 Java를 위해 (바이트코드 인자는 배열 입니다) 총 두개의 함수를 만듭니다.
  ```scala
  @scala.annotation.varargs
  def select(exprs: Expression*): DataFrame = { ... }
  ```

- 추상 varargs 함수는 Java에서 작동하지 않습니다. 이는 Scala의 버그 때문입니다([SI-1459](https://issues.scala-lang.org/browse/SI-1459), [SI-9013](https://issues.scala-lang.org/browse/SI-9013)).

- varargs 함수들을 오버로딩할 때 조심하도록 합니다. varargs 함수를 다른 varargs 타입과 오버로딩 하는 것은 소스의 호환성을 보장하지 않습니다. 
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
  대신, 명시적 타입을 갖는 인자를 처음 오게 합니다:
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

클래스나 함수를 위해 implicit을 사용하지 않습니다. 이는 `ClassTag`, `TypeTag` 를 포함합니다.
```scala
class JavaFriendlyAPI {
  // This is NOT Java friendly, since the method contains an implicit parameter (ClassTag).
  def convertTo[T: ClassTag](): T
}
```

### <a name='java-companion-object'>관련 객체, 정적 함수 및 변수</a>

동반하는 객체들과 정적 함수/변수 들을 사용 할 때, 몇 가지 조심해야 할 부분이 있습니다.

- 동반(companion) 객체들은 Java에서 사용하기에는 조금 어색합니다(동반(companion) 객체 `Foo` 는 `Foo$` 클래스의 `Foo$` 타입의 `MODULE$` 정적 변수 입니다).
  ```scala
  object Foo

  // equivalent to the following Java code
  public class Foo$ {
    Foo$ MODULE$ = // instantiation of the object
  }
  ```
  만약 동반(companion) 객체를 사용해야 한다면, Java 정적 변수를 다른 클래스에 만듭니다.

- 불행히도, JVM 정적 변수를 Scala에서 정의하는 방법은 없습니다. Java파일을 만들어 이를 정의하는데 사용하도록 합니다.
- 동반(companion) 객체의 함수들은 자동으로 동반(companion) 클래스의 정적 함수로 변하게 됩니다. 단, 같은 함수가 존재하지 않아야 합니다. 정적 함수의 생성이 보장 되도록 하는 가장 좋은 방법은 Java테스트 파일을 작성하여 이 정적 함수를 호출하는 것 입니다.
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

- 하나의 case 객체 (혹은 심지어 보통 동반(companion) 객체) MyClass는 사실 MyClass 타입이 아닙니다.
  ```scala
  case object MyClass

  // Test.java
  if (MyClass$.MODULE instanceof MyClass) {
    // The above condition is always false
  }
  ```
  이를 적절한 타입 구조를 갖을 수 있도록 구현하기 위해서 동반(companion) 클래스를 정의하고, 이를 case 객체에서 상속 받도록 합니다:
  ```scala
  class MyClass
  case object MyClass extends MyClass
  ```


## <a name='misc'>기타</a>

### <a name='misc_currentTimeMillis_vs_nanoTime'>currentTimeMillis 보다는 nanoTime</a>

*지속 시간*을 계산할 때 혹은 *타임아웃*을 확인 할 때에는, 심지어 millisecond 이하의 숫자들이 필요 없는 경우에도  `System.currentTimeMillis()`의 사용을 피하시고 `System.nanoTime()`을 사용 하시길 바랍니다.   

`System.currentTimeMillis()`는 현재 시간을 반환하고 현재 시스템의 클록을 뒤따라 바꿉니다. 따라서 이러한 네거티브 클록 조정은 긴 시간의 타임아웃을 초래할 수 있습니다(클록 시간 이전 값으로 잡을 때 까지). 이 것은 네트워크가 장 시간 중단 된 후에, ntpd가 다음 "step"으로 진행할 때 발생 될 수 있습니다. 가장 전형적인 예로는 시스템 부팅 동안 DHCP 시간이 평소보다 오래 소요될 때 입니다. 이는, 이는, 이해하거나 재현하기 힘든 에러를 초래 할수 있습니다. `System.nanoTime()`은 wall-clock에 상관 없이, 항상 일정하게 증가 합니다.

주의:
- 절대 `nanoTime()`의 절대값을 절대로 직렬화 하거나 다른 시스템으로 보내지 않습니다. 이 절대값은 의미가 없으며, 시스템 관련 값이고, 시스템이 재부팅 되면 리셋됩니다.
- 절대 `nanoTime()`의 절대값은 양수로 보장되지 않습니다(하지만 `t2 - t1` 은 올바른 값을 계산하도록 보장 됩니다).
- `nanoTime()`은 292년을 주기로 다시 계산합니다. 따라서 만약 Spark 작업(job)이 아주 긴 시간이 걸릴 것으로 예상된다면, 다른 무언가를 찾아야 하겠죠 :)

### <a name='misc_uri_url'>URL 보다는 URI</a>

어떤 서비스의 URL을 정렬 할 때, `URI` 표현을 사용하는 것이 권장됩니다.

`URL`의 [동일성 검사](http://docs.oracle.com/javase/7/docs/api/java/net/URL.html#equals(java.lang.Object)) 는 사실 IP 주소를 알아내기 위해 (블로킹) 네트워크 호출을 합니다.  `URI` 클래스는 필드의 동일성을 확인하고 `URL`의 상위 집합 입니다.


