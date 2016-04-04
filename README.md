# UnivEq

Safer universal equivalence for Scala & Scala.JS.
*(zero-dependency)*


## Motivation

In Scala, all values and objects have the following methods:
* `equals(Any): Boolean`
* `==(Any): Boolean`
* `!=(Any): Boolean`

This means that you can perform nonsensical comparisons that, at compile-time, you know will fail.

You're likely to quickly detect this kind of errors when you're writing them for the first time, but the larger problems are:
* valid comparisons becoming invalid after refactoring your data.
* calling a method that expects universal equality to hold with a data type in which it doesn't (eg. a method that uses `Set` under the hood).

It's a breeding ground for bugs.


## Provided Here
This library contains:

* A typeclass `UnivEq[A]`.
* A macro to derive instances for your types.
* Compilation error if a future change to your data types' args or their types, lose universal equality.
* Proofs for most built-in Scala & Java types.
* Ops `==*`/`!=*` to be used instead of `==`/`!=` so that incorrect type comparison yields compilation error.
* A few helper methods that provide safety during construction of maps and sets.
* Optional modules for Scalaz and Cats.


## Example

```scala
import japgolly.univeq._

case class Foo[A](name: String, value: Option[A])

// This will fail at compile-time.
// It doesn't hold for all A...
//implicit def fooUnivEq[A]: UnivEq[Foo[A]] = UnivEq.derive

// ...It only holds when A has universal equivalence.
implicit def fooUnivEq[A: UnivEq]: UnivEq[Foo[A]] = UnivEq.derive

// Let's create data with & without universal equivalence
trait Whatever
val nope = Foo("nope", Some(new Whatever{}))
val good = Foo("yay", Some(123))

nope ==* nope // This will fail at compile-time.
nope ==* good // This will fail at compile-time.
good ==* good // This is ok.

// Similarly, if you made a function like:
def countUnique[A: UnivEq](as: A*): Int =
  as.toSet.size

countUnique(nope, nope) // This will fail at compile-time.
countUnique(good, good) // This is ok.
```


## Installation

##### No dependencies:
```scala
// Your SBT
libraryDependencies += "com.github.japgolly.univeq" %%% "univeq" % "1.0.0"
// Your code
import japgolly.univeq._
```

##### Scalaz:
```scala
// Your SBT
libraryDependencies += "com.github.japgolly.univeq" %%% "univeq-scalaz" % "1.0.0"
// Your code
import japgolly.univeq.UnivEqScalaz._
```

##### Cats:
```scala
// Your SBT
libraryDependencies += "com.github.japgolly.univeq" %%% "univeq-cats" % "1.0.0"
// Your code
import japgolly.univeq.UnivEqCats._
```


## Usage

* Create instances for your own types like this:
  ```scala
  implicit def xxxxxxUnivEq[A: UnivEq]: UnivEq[Xxxxxx[A]] = UnivEq.derive
  ```
* Change `UnivEq.derive` to `UnivEq.deriveDebug` to display derivation details.
* If needed, you can create instances with `UnivEq.force` to tell the compiler to take your word.
* Use `==*`/`!=*` in place of `==`/`!=`.
* Add `: UnivEq` to type params that need it.


## Future Work

* Get rid of the `==*`/`!=*`; write a compiler plugin that checks for `UnivEq` at each `==`/`!=`.
* Add a separate `HashCode` typeclass instead of just using `UnivEq` for maps, sets and similar.

Note: I'm not working on these at the moment, but they'd be fantastic contributions.