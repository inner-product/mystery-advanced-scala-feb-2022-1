# ??? Advanced Scala Day 4

## Effect Types

- What is a monad?
- What is an effect monad?
- How does this differ from `Future`?

### Monads

Read "Scala with Cats", but for our purposes:

- a type constructor `F` with a `flatMap` method with type

  `def flatMap[B](f: A => F[B]): F[B]`

What does `flatMap` mean?

On containers you can think of `flatMap` as `map` followed by `flatten`.

`List(1,2,3).map(x => List(x, -x)).flatten`
`List(1,2,3).flatMap(x => List(x, -x))`


### Effect Monads

1. We want to reason about code (means controlling side effects)
2. We want composition (building code from smaller components)

However we also want concurrency, resource management, error handling.

Solve this with effect monads.

Separation between description and action
- provide a way of describing concurrency etc.
- then run the description

`flatMap` describes sequential computation. Do this and then do that.

`F[A].flatMap(A => F[B])`
 ^------ Do this
              ^----- Using the result
                   ^----- Do that

Sequential composition of actions or programs or computations.

Fail-fast error handling. Stops when there is an error. Handle errors with different methods (`handleError` etc.)

`IO` is a description. Nothing happens until you run it using one of the `unsafe` methods (or your main program entry point can extend `IOApp` and you return an `IO` from `run`.)



### Future

`Future` has `flatMap` but it starts executing immediately (and it caches results). We don't have the separation between description and action.

It's similar to `Promise` in JS.

```scala
val a = Future(...)
val b = Future(...)

for {
  theA <- a
  theB <- b
} yield doSomething(theA, theB)
```

should be equivalent to the below, according to substitution

```scala
for {
  theA <- Future(...)
  theB <- Future(...)
} yield doSomething(theA, theB)
```

but it is not because the `Futures` are created at different times and hence start running at different times.


## Concurrency in IO

The easiest way to get concurrency is with `parMapN`, `parSequence` and related methods. These come from Cats.

`import cats.syntax.all._`

`(F[A], F[B], ..., F[D]).parMapN((A, B, ..., D) => E): F[E]`

Available on `IO` and runs all the `IO` in parallel.

`(IO[A], IO[B], ..., IO[D]).parMapN((A, B, ..., D) => E): IO[E]`

For `List[IO[A]]` you can use `parSequence` to get `IO[List[A]]` which is computed in parallel.

`parTraverse` for more exciting use cases.


## Error Handling

Two classes of errors:
- recoverable: generally end user errors
- unrecoverable: everything else

Two classes of people interested in errors:
- end user: what did I do wrong and how can I fix it?
- programmer: where in the code did this occur?

Recoverable errors intended for the end user: carefully structure and report
Unrecoverable errors just crash (maybe with some shutdown logic)

Programmer: wants stack traces
End user: shouldn't see stack traces (security hole) but does want reports in their language / terminology

Unrecoverable error: just throw an exception
- in an effect monad, the effect monad will handle releasing resources etc.

Recoverable errors: consider using `Either[Error, Result]`
- `Error` is usually an algebraic data type (compiler shouts at you if you don't handle a case)

`IO[Either[Error, Result]]` for recoverable errors

Can avoid nested flatMap with `EitherT`

https://typelevel.org/cats/datatypes/eithert.html

EitherT is syntactic sugar for F[Either[A,B]]

(See "Scala with Cats")

`F[Either[A, B]].flatMap(e: Either[A, B] => e.flatMap(b: B => ...))`
`EitherT[F, A, B].flatMap(b: B => ...)`


## IO Runtime

Fibers are lightweight threads, or green threads, or cooperative threads. 

OS threads have high context switch overhead.
Lightweight threads do not, but are not preemptive.

IO runs on fibers. You can access fibers directly but that is unnecessary in most cases.

M:N threading / green threads
- M "green" threads assigned to N OS threads
- Usually M >> N


## Essential Effects

Book by my colleague Adam Rosien.


## Datastructures

Concurrent data structures like `Queue` etc.

`IO[Queue[...]]` is a description to create a queue

```scala
val q: IO[Queue[...]] = ...


for {
  q1 <- q // These are different queues
  q2 <- q // Not the same as q1!
} yield ...
```
