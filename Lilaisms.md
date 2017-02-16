# Lilaisms

Code concepts and functions that are specific to, and idiomatic to, the lila codebase.

## How lila uses scala

Scala can be used as a better Java, or as a worse Haskell. lila definitely goes for the latter.

lila makes little use of object orientation and particularly inheritance, but:

- Immutability is required everywhere. Notable exceptions can be made:
  - in akka actors (because race conditions are no longer a concern)
  - in performance sensitive functions. In this case, the mutability
    must be contained in the function scope. The function must expose
    an immutable signature.
- Strong typing is preferred. For instance, `FiniteDuration` is better than `Int`,
  and `case class Name(value: String)` is better than `String`.
  The more the value is used in the code, and the more useful it is to type it correctly.
  This rule is actually loosely respected in lila, but that's the direction we're going to.

## Weird shit you will only see in lila

If the function you're looking for is not here,
try the [scala doc](http://www.scala-lang.org/files/archive/api/2.12.1/)
or the [scalaz doc](https://oss.sonatype.org/service/local/repositories/releases/archive/org/scalaz/scalaz_2.11/7.1.11/scalaz_2.11-7.1.11-javadoc.jar/!/index.html).

## Disclaimer

> Most of these lilaisms are driven by a compulsive tendency to play [code golf](https://en.wikipedia.org/wiki/Code_golf).
> However, they may also present benefits in terms of type safety.
> In any case, contributors are never expected to use lilaisms. You can make use of them,
> but a pull request will likely not be rejected because it isn't idiomatic to lila.
> However, immutability and type safety remain requirements (with discussable exceptions).

### Debugging (actually useful)

#### `pp`

`pp` means `print & pass`, it's an inline `println`.
It prints the value and returns it.

```scala
"foo".pp                   // prints "foo" to stdout, and returns "foo"

player.pp.make(move.pp).pp // behaves like player.make(move), but prints
                           // player, move, and the result of player.make(move)

"foo".pp("context")        // prints "context: foo" to stdout, and returns "foo"
```

#### `thenPp`

Like `pp` but applies to futures, and prints their result.

```scala
fetchUser(id).thenPp       // returns Future[Option[User]] and prints the Option[User] when available
```

### Type aliases

For code golf purposes, common types have been aliased.

```scala
Fu[A]         // Future[A]
Funit         // Future[Unit]
```

### Future shortcuts

```scala
def fuccess[A](a: A) = Future successful a
def fufail[A <: Throwable, B](a: A): Fu[B] = Future failed a
def fufail[A](a: String): Fu[A] = fufail(common.LilaException(a))
val funit = fuccess(())
```

### Int functions

```scala
// I think it reads better
a atMost b                  // a min b
a atLeast b                 // a max b
```

### Option functions

Some of these (like `|`) actually come from [scalaz](https://github.com/scalaz/scalaz)

> Reminder: In scala, `a.b(c)` == `a b c`. For instance, `1.+(2)` == `1 + 2`.

```scala
42.some                     // Option(42) // or Some(42): Option[Int]
none[Int]                   // None: Option[Int]

maybeInt | 0                // maybeInt.getOrElse(0) // but with extra type safety (and code golfing)
maybeInt ifTrue boolean     // maybeInt.filter(_ => boolean)
maybeInt ifFalse boolean    // maybeInt.filter(_ => !boolean)
maybeInt has value          // maybeInt contains value // but with extra type safety

~maybeInt                   // maybeInt | 0 // where 0 is provided by the Zero[Int] typeclass instance
maybeInt ?? f               // maybeInt.fold(0)(f)
                            // e.g. 42.some ?? (_ + 1) == 43
                            //      none[Int] ?? (_ + 1) == 0
```

### Boolean functions

```scala
boolean option 42           // if (boolean) Some(42) else None // or Some(42) ifTrue boolean

boolean ?? 42               // if (boolean) 42 else 0
boolean ?? "foo"            // if (boolean) "foo" else ""
boolean ?? List(1, 2)       // if (boolean) List(1, 2) else List.empty[Int]
boolean ?? 42.some          // if (boolean) 42.some else none
boolean ?? fuccess(42)      // if (boolean) fuccess(42) else fuccess(0)
boolean ?? asyncComputation // if (boolean) asyncComputation else funit

// etc, with every type having a Zero instance.
```

### Future functions

```scala
val fu: Future[Int]
val f: Int => Boolean

fu dmap f                   // fu map f // but runs on the same thread (perf tweak)
fu dforeach f               // fu foreach f // but runs on the same thread (perf tweak)

fu.void                     // fu.map(_ => ()) // discards the result, returns Funit
fu inject "foo"             // fu.map(_ => "foo") // replaces the result
```

#### Sequencing futures and effects

```scala
fu >> otherFu               // fu.flatMap(_ => otherFu) // sequence without using first result
fu >>- effect               // fu andThen { case _ => effect } // run a side effect after completion

fu1 >> fu2 >>- effect1 >>- effect2 // sequences f1 and f2, then runs effect1 then effect2
```

```scala
val futures: List[Future[Int]]

futures.sequenceFu: Future[List[Int]] // executes all futures and return one with a list of values
```
