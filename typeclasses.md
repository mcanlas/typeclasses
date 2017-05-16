# Implicits,<br>Type Classes,<br>Algebird

Mark Canlas (@markcanlasnyc) - May 16, 2017


# Spoiler alert
Type classes are drivers


# Don't be scared


# Be confident



# Extending behavior
The problem statement


<pre><code class="scala">class Actor(name: String)</code></pre>


<pre><code class="scala">class Actor(name: String)

// new behaviors

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


# Implicits<br>are not magic
Note: Tool


# Compile safe


# "implicits"


* Implicit conversions
* Implicit parameters



# Implicit conversions


<pre><code class="scala">val shape: RoundHole = ??? : SquarePeg</code></pre>



# Implicit parameters


<pre><code class="scala">def sum(a: Int)(implicit b: Int)</code></pre>


# Implicit parameters
# are parameters


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


<pre><code class="scala">def execute[A, B](f: A => B)(implicit ec: ExecutionContext)

// go away, compiler!
import scala.concurrent.ExecutionContext.Implicits.global

execute()

execute(ec)</code></pre>


## Nested implicits
<pre><code class="scala">def inner(implicit ai: Int): Unit = println(ai)

def outer(implicit ao: Int): Unit = inner

{
  implicit val a = 3

  outer
}

outer(4)
</code></pre>



# Type classes
Finally.


# Haskell


## Implemented with multiple mechanisms in Scala
* Traits
* Implicits


# Clunky name


# A class for types


# What is a class?


<pre><code class="scala">final class Actor(name: String)</code></pre>


<pre><code class="scala">type A = Actor</code></pre>


<pre><code class="scala">// Template + Value(s) = Value</code></pre>


# Type classes
# A class for types


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


<pre><code class="scala">// an A that can do B
type X = A[B]

type Y = JsonWriter[Dancer]

type Z = Triple[Comedian]
</code></pre>



# Context bounds
A tool for writing generic code


<pre><code class="scala">// providing driver implicitly, easy peasy

def write(a: Int)(implicit j: JsonWriter[Int]) =
  j.write(a)</code></pre>


<pre><code class="scala">// where am i?

def outerWrite[A](a: A)(implicit j: JsonWriter[A]) =
  innerWrite

def innerWrite[A](a: A)(implicit j: JsonWriter[A]) =
  j.write(a)
</code></pre>


<pre><code class="scala">def outerWrite[A : JsonWriter](a: A) =
  innerWrite

def innerWrite[A](a: A)(implicit j: JsonWriter[A]) =
  j.write(a)
</code></pre>


<pre><code class="scala">// prove to me statically
// that A can B

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


# Framework
By Twitter


# Common scenarios, solved
Monoid, Semigroup


# Reliability
When's the last time you jumped to definition on <code>Seq</code>?


<pre><code class="scala">// proceed with caution

abstract class MyLinkedList[A]</code></pre>


<pre><code class="scala">type A = Zero[Double] // 0d

type B = Identity[Int] // + 0

type C = Sum[String] // _ + _</code></pre>


<pre><code class="scala">reduce(_ + _)

reduce(_ * _)

reduce(_ ++ _)</code></pre>



# Big data



# Scalding


<pre><code class="scala">def reduce</code></pre>


# Big ideas


# Implicit parameters
## A tool to pass around default values


# Type classes
## Are drivers


# Algebird
## A framework for common solutions



# HOMEWORK.MD


# github.com / mcanlas / typeclasses
# @mark canlas nyc
# @tapad eng
