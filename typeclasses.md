# Implicits,<br>Type Classes,<br>Algebird

Mark Canlas (@mark canlas nyc) - May 16, 2017



<pre><code class="scala">def printAgg[A : Semigroup](a: A, b: A): Unit =
  printSum(a, b)

def printSum[A : Semigroup](a: A, b: A): Unit =
  println(sum(a, b))

def sum[A : Semigroup](a: A, b: A): A =
  implicitly[Semigroup[A]].plus(a, b)</code></pre>



# Type classes
A design pattern for drivers


# Haskell
f() that.


## A pattern as implemented in Scala
* Traits
* Type parameters
* Implicits


<code>Semigroup</code> is the type class
<pre><code class="scala">def printAgg[A : Semigroup](a: A, b: A): Unit =
  printSum(a, b)

def printSum[A : Semigroup](a: A, b: A): Unit =
  println(sum(a, b))

def sum[A : Semigroup](a: A, b: A): A =
  implicitly[Semigroup[A]].plus(a, b)</code></pre>



# Why type classes?


## To extend behavior<br>on arbitrary types
Including ones you can't control


## To easily provide<br>default drivers
And create new driver implementations


## Consume drivers<br>in a lightweight syntax
Either yours or ones given to you



# Extending<br>behavior
The old school ways


<pre><code class="scala">class Widget

class Sprocket</code></pre>


<pre><code class="scala">trait JsonWritable {
  def toJson: String
}

class Widget extends JsonWritable { ... }

class Sprocket extends JsonWritable { ... }
</code></pre>


## Extension via inheritance
* Pro: easy to understand
* Con: requires access to modify target types
* Con: uses a class (implies state)


<pre><code class="scala">final class Quark

// Int, String, Double</code></pre>


<pre><code class="scala">class WritableQuark(q: Quark) extends JsonWritable  { ... }

class WritableInt(i: Int) extends JsonWritable { ... }</code></pre>


## Extension via wrapping
* Pro: easy to understand
* Pro: Extension on sealed types
* Con: uses a class (implies state)
* Con: overhead
  * Payload must always be wrapped
  * Raw value must always be extracted


How about static functions?


<code>JsonWritable</code> is the type class
<pre><code class="scala">trait JsonWritable[A] {
  def toJson(a: A): String
}

object QuarkWriter extends JsonWritable[Quark] { ... }

object IntWriter extends JsonWritable[Int] { ... }</code></pre>


What would have been methods on a class instance are now in a static companion.
<pre><code class="scala">val a = new WritableQuark(quark)

type A = JsonWritable[Quark]
</code></pre>


## Type class
A class/template for types
<pre><code class="scala">val a = new WritableQuark(quark)

type A = JsonWritable[Quark]
</code></pre>


<pre><code class="scala">// type A that can do behavior B
// an A that can do B

type X = B[A]
</code></pre>


## Extension via static methods
* Pro: Fundamentally a type class
* Pro: Stateless (easy to test)
* Pro: Extension on sealed types
* Con: Verbose delivery...



# Easy delivery
Of default implementations and yours.


| Framework | You    |
|-----------|--------|
| Int       | Widget
| Double    | Sprocket
| String
| ...
| Tuple
| List


<pre><code class="scala">// framework
object IntWriter extends JsonWritable[Int]

// framework
object DoubleWriter extends JsonWritable[Double]

// custom
object WidgetWriter extends JsonWritable[Widget]</code></pre>


### Explicit dependency injection
<pre><code class="scala">def inner(i: Int)(w: Writer[Int]): Unit = w(i)

def outer1(i: Int)(w1: Writer[Int]): Unit = inner(i)(w1)

def outer2(i: Int)(w2: Writer[Int]): Unit = outer1(i)(w2)

def outer3(i: Int)(w3: Writer[Int]): Unit = outer2(i)(w3)

outer3(42)(IntWriter)
</code></pre>



# Easy delivery
Part I, simpler call sites


# Implicits
## are not magic
Note: Tool


## Compile safe


## "implicits"
* Implicit conversions
* Implicit parameters



# Implicit<br>conversions


<pre><code class="scala">val shape: RoundHole = ??? : SquarePeg</code></pre>



# Implicit<br>parameters


## Contrived examples warning
Use sparingly.


<pre><code class="scala">def sum(a: Int)(implicit b: Int)</code></pre>


## Implicit parameters
## are parameters


<pre><code class="scala">def sum(a: Int)(b: Int): Int = a + b

def sum(a: Int)(implicit b: Int): Int = a + b</code></pre>


<pre><code class="scala">// satisfied explicitly

sum(2)(3) // 5</code></pre>


<pre><code class="scala">// satisfied implicitly

implicit val b: Int = 3

sum(2) // 5</code></pre>


## Implicit resolution criteria:
* In scope
* Type fits
* Marked as implicit


<pre><code class="scala">def format(n: Int)(implicit loc: Locale)</code></pre>


<pre><code class="scala">def format(n: Int)(implicit loc: Locale)

format(12345)(Locale.US)

implicit defaultLocale: Locale = Locale.US

format(6789)

format(90210)

format(8675)
</code></pre>


<pre><code class="scala">def calculate(a: Int)(implicit ec: ExecutionContext)

// go away, compiler!
import scala.concurrent.ExecutionContext.Implicits.global

calculate(456)(global)

calculate(123)</code></pre>


## Nested implicits
Implicit parameters of a function are implicit values for its body


<pre><code class="scala">def inner(implicit i: Int): Unit = println(i)

def outer1(implicit o1: Int): Unit = inner

def outer2(implicit o2: Int): Unit = outer1

def outer3(implicit o3: Int): Unit = outer2

implicit val a = 42
outer3 // prints 42
</code></pre>


<pre><code class="scala">def inner(implicit i: Int): Unit = println(i)

def outer1(implicit o1: Int): Unit = inner

def outer2(implicit o2: Int): Unit = outer1

def outer3(implicit o3: Int): Unit = outer2

outer3(42) // prints 42
</code></pre>


Good for hiding context that is necessary<br>but not essential for a function


### Implicit dependency injection
<pre><code class="scala">def inner(i: Int)(implicit w: Writer[Int]): Unit = w.write(i)

def outer1(i: Int)(implicit w1: Writer[Int]): Unit = inner(i)

def outer2(i: Int)(implicit w2: Writer[Int]): Unit = outer1(i)

def outer3(i: Int)(implicit w3: Writer[Int]): Unit = outer2(i)

implicit val w = IntWriter
outer3(42)
</code></pre>


## Implicit dependency injection
* Pro: Slims down call sites
* Con: Bulkier function definitions



# Easy delivery
Part II, describing type classes


# Context bounds
An annotation on type parameters


<pre><code class="scala">// providing a driver implicitly, easy peasy

def toJson(a: Int)(implicit j: JsonWriter[Int]) =
  j.write(a)</code></pre>


<pre><code class="scala">// where am i?

def toJson[A](a: A)(implicit j: JsonWriter[A]) =
  inner

def inner[A](a: A)(implicit j: JsonWriter[A]) =
  j.write(a)
</code></pre>


## Context bound
On a parameterized type
<pre><code class="scala">// syntactic sugar
def toJson[A : JsonWriter](a: A) =
  inner

def inner[A](a: A)(implicit j: JsonWriter[A]) =
  j.write(a)
</code></pre>


<pre><code class="scala">// prove to me statically
// that A can B, given type A and type class B[_]

def foo[A : B] = ???

// referred to as "evidence" in the scala spec</code></pre>


<pre><code class="scala">def toJson[A : JsonWriter](a: A) =
  inner

// anonymous implicit, formerly named `j`
def inner[A : JsonWriter](a: A) = ???
</code></pre>


<pre><code class="scala">// where Z is JsonWriter[A]
def findImplicit[Z](implicit z: Z) = z

// anonymous implicit
def inner[A : JsonWriter](a: A) =
  findImplicit[JsonWriter[A]].write(a)
</code></pre>


<pre><code class="scala">// anonymous implicit via predef
def inner[A : JsonWriter](a: A) =
  implicitly[JsonWriter[A]].write(a)
</code></pre>


### Context bound dependency injection
<pre><code class="scala">def inner[A : Writer](a: A): Unit = implicitly[Writer[A]].write(i)

def outer1[A : Writer](a: A): Unit = inner(a)

def outer2[A : Writer](a: A): Unit = outer1(a)

def outer3[A : Writer](a: A): Unit = outer2(a)

implicit val w = IntWriter
outer3(42)
</code></pre>


## Context bounds
* Pro: lighter syntax, especially for deep nesting
* Con: needs to be summoned at final use site



# Easy delivery
Part III, companion resolution


Where do the implicits come from?


## More implicit resolution criteria:
Includes members of the companion objects<br>for the types in play (A and B)
<pre><code class="scala">def foo[A : B]</code></pre>


<pre><code class="scala">// typical of framework authors
object JsonWriter {
  implicit object JsonWriterInt extends JsonWriter[Int]
}

// typical of framework consumers, domain classes
object Widget {
  implicit object JsonWriterDancer extends JsonWriter[Widget]
}

// one or the other, not both
</code></pre>


|           | JsonWriter[A] |
|-----------|--------|
| Int       | JsonWriter[Int]
| Double    | JsonWriter[Double]
| String    | JsonWriter[String]
| ...       | JsonWriter[...]
| Tuple     | JsonWriter[Tuple]
| List      | JsonWriter[List]
| Widget       | JsonWriter[Widget]
| Sprocket    | JsonWriter[Sprocket]


### Companion dependency injection
<pre><code class="scala">object Writer {
  implicit val intWriter = ???
}

def inner[A : Writer](a: A): Unit = implicitly[Writer[A]].write(i)

def outer1[A : Writer](a: A): Unit = inner(a)

def outer2[A : Writer](a: A): Unit = outer1(a)

def outer3[A : Writer](a: A): Unit = outer2(a)

outer3(42)
</code></pre>


## Companion dependency injection
* Pro: Declutters use sites
* Pro: Feels cool
* Con: Not as transparent/easily understood
* Con: Difficult to track down if companion hierarchy is complex



# Algebird
Algebra for birds
Note: The study of structures/properties


## Framework
By Twitter


## Common scenarios, solved
With type classes like <code>Monoid</code>, <code>Semigroup</code>


## Reliability
When's the last time you jumped to definition on <code>Seq</code>?


<pre><code class="scala">// proceed with caution

abstract class MyLinkedList[A]</code></pre>


<pre><code class="scala">type A = Zero[Double] // 0d

type B = Identity[Int] // _ + 0

type C = Sum[String] // _ + _</code></pre>


<pre><code class="scala">// also known as "sum"

reduce(_ + _)

reduce(_ * _)

reduce(_ ++ _)</code></pre>


## Big data
Aggregation without ordering


## Scalding

Scala DSL for MapReduce

<pre><code class="scala">// can you prove to me statically
// that i can reduce over A reliably?

def sum[A : Semigroup]: Pipe = ???</code></pre>


<pre><code class="scala">case class MyCounter()

object MyCounter extends Semigroup[MyCounter]</code></pre>



# HOMEWORK.MD



## github.com / mcanlas / typeclasses
## @mark canlas nyc
## @tapad eng
We are hiring.