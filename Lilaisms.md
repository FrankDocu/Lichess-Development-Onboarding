# Lilaisms

> Code concepts and functions that are specific to, and idiomatic to, the lila codebase.

## Disclaimer

> Most of these lilaisms are driven by a compulsive tendency to play [code golf](https://en.wikipedia.org/wiki/Code_golf).
> However, they may also present benefits in terms of type safety.
> In any case, contributors are never expected to use lilaisms. You can make use of them,
> but a pull request will likely not be rejected because it isn't idiomatic to lila.
> However, immutability and type safety remain requirements (with discussable exceptions).

## How lila uses scala

Scala can be used as a better Java, or as a worse Haskell. lila definitely goes for the latter.

lila makes little use of object orientation and particularly inheritance, but:

- Immutability is required everywhere. Notable exceptions can be made:
  - in akka actors (because race conditions are no longer a concern)
  - in performance sensitive functions. In this case, the mutability
    must be contained in the function scope. The function must expose
    an immutable signature.
- Strong typing is preferred. For instance, `FiniteDuration` is better than `Int`,
  and `case class Name(value: String)` is better than String.
  This rule is actually loosely respected in lila, but that's the direction we're going to.

## Weird shit you will only see in lila

Here comes the codegolfing madness. You don't have to use it, but you will have to read it.

### Debugging (actually useful)

#### `pp`

`pp` means `print & pass`, it's an inline `println`.
It prints the value and returns it.

```scala
"foo".pp                   // prints "foo" to stdout

player.pp.make(move.pp).pp // behaves like player.make(move), but prints
                              the string representations of player, move, and the function result

"foo".pp("context")        // prints "context: foo" to stdout
```

#### `thenPp`

Like `pp` but applies to futures, and prints their result.

```scala
fetchUser(id).thenPp       // returns Future[Option[User]] and prints the Option[User] when available
```
