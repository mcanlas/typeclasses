# Implicits,<br>Type Classes,<br>Algebird

Mark Canlas (@mark canlas nyc) - May 16, 2017



# Spoiler alert
Type classes are drivers


# Don't be scared
Implicits, type classes, algebra, semigroups


# Be confident
New tools, old problems



# Extending behavior
The traditional ways


<pre><code class="scala">class Actor(name: String)</code></pre>


<pre><code class="scala">class Actor(name: String)

// new behaviors via extension

class Dancer extends Actor {
  def dance: Unit
}

class Comedian extends Actor {
  def makeJokes: Unit
}</code></pre>


<pre><code class="scala">final class FinalActor(name: String)

Int, String, Double</code></pre>


<pre><code class="scala">// wrapper classes

class Dancer(a: FinalActor)

class Comedian(a: FinalActor)</code></pre>



# Implicits
Run away!


## Implicits<br>are not magic
Note: Tool


## Compile safe
<code>Ctrl + Q</code>


## "implicits"
* Implicit conversions
* Implicit parameters



# Implicit conversions


<pre><code class="scala">val shape: RoundHole = ??? : SquarePeg</code></pre>



# Implicit parameters


<pre><code class="scala">def sum(a: Int)(implicit b: Int)</code></pre>


## Implicit parameters
## are parameters


<pre><code class="scala">def sum(a: Int)(b: Int)

def sum(a: Int)(implicit b: Int)</code></pre>


<pre><code class="scala">// satisfied explicitly

sum(2)(3)</code></pre>


<pre><code class="scala">// satisfied implicitly

implicit val b: Int = 3

sum(2)</code></pre>


## Criteria:
* In scope
* Type fits
* Marked as implicit


<pre><code class="scala">def format(n: Int)(implicit loc: Locale)</code></pre>


<pre><code class="scala">def format(n: Int)(implicit loc: Locale)

// a required parameter, and a common value

format(12345)(Locale.US)

format(6789)
</code></pre>


<pre><code class="scala">def calculate(a: Int)(implicit ec: ExecutionContext)

// go away, compiler!
import scala.concurrent.ExecutionContext.Implicits.global

calculate(123)

calculate(456)(ec)</code></pre>


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


* Useful for hiding boilerplate
* Season to taste



# Type classes
Finally.


# Haskell
f() that.


## As implemented in Scala
* Traits
* Type parameters
* Implicits


# Clunky name


# A class for types


# What is a class?


<pre><code class="scala">// a template for values
final class Actor(name: String)</code></pre>


<pre><code class="scala">// Actor itself is not a value
type A = Actor</code></pre>


<pre><code class="scala">// Template + Value(s) = Value</code></pre>


# Type classes
## A class (a template) for types


<pre><code class="scala">// F[_] + A = F[A]</code></pre>


<pre><code class="scala">// Actor("Mark") is one value

val b = new Actor("Mark")

// F[A] ("F of A") is one type

type B = F[A]</code></pre>


<pre><code class="scala">trait Dancer[A]

trait Comedian[A]

// an actor, that can dance
type A = Dancer[Actor]

// an actor, that can do comedy
type B = Comedian[Actor]
</code></pre>


<pre><code class="scala">// an int that can be tripled
type A = Triple[Int]

// an int that can be written as json
type B = JsonWriter[Int]

// a driver, maybe?
</code></pre>


* Static
* Canonical


<pre><code class="scala">trait Triple[A] {
  def triple(a: A): A
}

object TripleInt extends Triple[Int] {
  def triple(a: Int): Int = a * 3
}
</code></pre>


<pre><code class="scala">trait JsonWriter[A] {
  def write(a: A): String
}

object JsonWriterInt extends JsonWriter[Int] {
  def write(a: Int): String = a.toString
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

def write(a: Int)(implicit j: JsonWriter[Int]) =
  j.write(a)</code></pre>


<pre><code class="scala">// where am i?

def outerWrite[A](a: A)(implicit j: JsonWriter[A]) =
  innerWrite

def innerWrite[A](a: A)(implicit j: JsonWriter[A]) =
  j.write(a)
</code></pre>


## Context bound
On a parameterized type
<pre><code class="scala">def outerWrite[A : JsonWriter](a: A) =
  innerWrite

def innerWrite[A](a: A)(implicit j: JsonWriter[A]) =
  j.write(a)
</code></pre>


<pre><code class="scala">// prove to me statically
// that A can B
// given A and B[_]

def foo[A : B] = ???</code></pre>


<pre><code class="scala">def outerWrite[A : JsonWriter](a: A) =
  innerWrite

// anonymous implicit
def innerWrite[A : JsonWriter](a: A) = ???
</code></pre>


<pre><code class="scala">// where Z is JsonWriter[A]
def findImplicit[Z](implicit z: Z) = z

// anonymous implicit
def innerWrite[A : JsonWriter](a: A) =
  findImplicit[JsonWriter[A]].write(a)
</code></pre>


<pre><code class="scala">// anonymous implicit via predef
def innerWrite[A : JsonWriter](a: A) =
  implicitly[JsonWriter[A]].write(a)
</code></pre>



# Algebird
Algebra for birds


## Framework
By Twitter


## Common scenarios, solved
<code>Monoid</code>, <code>Semigroup</code>


## Reliability
When's the last time you jumped to definition on <code>Seq</code>?


<pre><code class="scala">// proceed with caution

abstract class MyLinkedList[A]</code></pre>


<pre><code class="scala">type A = Zero[Double] // 0d

type B = Identity[Int] // + 0

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
// that i can reduce over A

def sum[A : Semigroup]: Pipe = ???</code></pre>


<pre><code class="scala">case class MyCounter()

object MyCounter extends Semigroup[MyCounter]</code></pre>


## Algebird provides solutions
To commonly occurring problems


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


### Scalding uses context bounds on parameterized types to implicitly find a semigroup type class provided by Algebird.



# HOMEWORK.MD



## github.com / mcanlas / typeclasses
# @mark canlas nyc
# @tapad eng
