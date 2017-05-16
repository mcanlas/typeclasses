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


<code>Semigroup</code> is the type class
<pre><code class="scala">def printAgg[A : Semigroup](a: A, b: A): Unit =
  printSum(a, b)

def printSum[A : Semigroup](a: A, b: A): Unit =
  println(sum(a, b))

def sum[A : Semigroup](a: A, b: A): A =
  implicitly[Semigroup[A]].plus(a, b)</code></pre>



# Why type classes?


## The ability to extend behavior<br>on arbitrary types
Including ones you can't control


## The ability to easily provide<br>default drivers


## Consume drivers in a lightweight syntax
Either yours or ones given to you



# Extending<br>behavior
The old school ways


<pre><code class="scala">class Widget

class Sprocket</code></pre>


<pre><code class="scala">trait JsonWritable {
  def toJson: String
}

class Widget extends JsonWritable

class Sprocket extends JsonWritable</code></pre>


## Extension via inheritance
* Pro: easy to understand
* Con: requires access to modify target types


<pre><code class="scala">final class Quark

// Int, String, Double</code></pre>


<pre><code class="scala">class WritableQuark(q: Quark) extends JsonWritable

class WritableInt(i: Int) extends JsonWritable</code></pre>


## Extension via wrapping
* Pro: easy to understand
* Con: cognitive/mechnical overhead

<code>JsonWritable</code> is the type class
<pre><code class="scala">trait JsonWritable[A] {
  def toJson(a: A): String
}

object QuarkWriter extends JsonWritable[Quark]

object IntWriter extends JsonWritable[Int]</code></pre>


## Extension via static methods
* Pro: Fundamentally a type class
* Pro: Stateless (easy to test)
* Pro: Extension on sealed types
* Con: Boilerplate delivery



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


<pre><code class="scala">JsonWritable[Int] // framework

JsonWritable[Double] // framework

JsonWritable[Widget] // custom</code></pre>


## Implicit delivery
Run away!


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


## Criteria:
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

calculate(456)(global)

// go away, compiler!
import scala.concurrent.ExecutionContext.Implicits.global

calculate(123)</code></pre>


## Nested implicits
<pre><code class="scala">def inner(implicit ai: Int): Unit =
  println(ai)

def outer(implicit ao: Int): Unit =
  inner

{
  implicit val a = 3
  outer
}

outer(4)
</code></pre>


Good for hiding context that is necessary but not essential for a function



# Type classes
Finally.


# Haskell
f() that.


## A pattern as implemented in Scala
* Traits
* Type parameters
* Implicits


## Clunky name


## A class for types


## What is a class?


<pre><code class="scala">// a template for values
final class Actor(name: String)</code></pre>


<pre><code class="scala">// Actor itself is not a value
type A = Actor</code></pre>


<pre><code class="scala">// Template + Value(s) = Value</code></pre>


# Type class
## A template for types


<pre><code class="scala">// F[_] + A = F[A]</code></pre>
Note: Not every trait with a type parameter is a type class


<pre><code class="scala">// Actor("Mark") is one value

val x = new Actor("Mark")

// F[A] ("F of A") is one type

type X = F[A]</code></pre>


## Encoding behavior
<pre><code class="scala">trait Dancer[A]

trait Comedian[A]

// an actor, that can dance
type A = Dancer[Actor]

// an actor, that can do comedy
type B = Comedian[Actor]
</code></pre>


## Drivers
Data classes, behavior on the outside.
<pre><code class="scala">// an int that can be tripled
type A = Triple[Int]

// an int that can be written as json
type B = JsonWriter[Int]
</code></pre>


* Static (stateless)
* Canonical


<pre><code class="scala">trait Triple[A] {
  def triple(a: A): A
}

object TripleInt extends Triple[Int] {
  def triple(a: Int): Int = a * 3
}

// old way, not stateful
class TripleWrapper(a: Int) {
  def triple: Int = a * 3
}
</code></pre>


<pre><code class="scala">trait JsonWriter[A] {
  def write(a: A): String
}

object JsonWriterInt extends JsonWriter[Int] {
  def write(a: Int): String = a.toString
}

// old way, not stateful
class JsonWrapper(a: Int) {
  def write: String = a.toString
}
</code></pre>


<pre><code class="scala">// a B that can do A
type X = A[B]

type Y = JsonWriter[Dancer]

type Z = Triple[Comedian]
</code></pre>



# Context bounds
A tool for writing generic code


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
// that A can B, given A and B[_]

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



# Resolution
Where do the implicits come from?


Members of the companion objects<br>for the types in play (A and B)
<pre><code class="scala">def foo[A : B]</code></pre>


<pre><code class="scala">// typical of framework authors
object JsonWriter {
  implicit object JsonWriterInt extends JsonWriter[Int]
}

// typical of framework consumers, domain classes
object Dancer {
  implicit object JsonWriterDancer extends JsonWriter[Dancer]
}

// one or the other, not both
</code></pre>



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


Typical of frameworks
* Int
* Double
* String
* Float
* Char
* Boolean
* List
* Tuple



# The Summit


### Scalding uses context bounds on parameterized types to implicitly find a semigroup type class provided by Algebird for safe map reduce.



# HOMEWORK.MD



## github.com / mcanlas / typeclasses
## @mark canlas nyc
## @tapad eng
We are hiring.