# ??? Advanced Scala Day 5

## Concurrency


## Type Classes

Type classes are about *ad-hoc polymorphism*. (YAFWIFP)

Get a bunch of types with no particular relationship and say they implement the same operations or interface.

- Algebraic data types are closed and cannot be extended. Type classes can add functionality to any type at any point in time.

- OO world we must declare the interfaces a type implements at the point where we define the type. See above for type classes.

Type classes:
- implementation of the interface is decoupled from the definition of the type that implements the interface
- can add an implementation of the interface to any type for which it is possible to define the operations of interest

Example: converting stuff to JSON
- Java / Scala have `toString`
- What about `toJson`?
- Type classes solve this problem

Components of a type class are:
- the type class: declares an interface / the functionality we want
- type class instances: provide the functionality for specific types
- type class usage: requires the functionality and uses it

Type class: trait w/ at least one type parameter
Type class instance: implicit value
Type class usage: implicit parameter

Type class example:

```scala
// Type class
trait JsonWriter[A] {
  def write(a: A): Json
}
```

Implicit values:

1. `implicit val`
2. `implicit object`
3. `implicit def` with *only implicit parameters* <--- good stuff here

`implicit` is just a marker telling the compiler it can use the value as an implicit value. An `implicit val`, for example, is otherwise a normal `val`.

```scala
// Type class instances
implicit val stringWriter: JsonWriter[String] =
  new JsonWriter[String] {
    def write(a: String): Json = JString(a)
  }
  
implicit object intWriter extends JsonWriter[String] {
  def write(a: Int): Json = JNumber(a)
}
```

Implicit parameters:

- Last parameter list of a method
- Starts with the keyword `implicit`
- Applies to all the parameters in the parameter list

Implicit parameters function like normal parameters if the programmer passes arguments in the normal way. If the programmer doesn't pass arguments the compiler tries to find *implicit values* in the *implicit scope* that match the type of the parameter. If it find such values it provides them as arguments.

Implicit scope:

- the normal "lexical" scope. Enclosing {}s and imports
- companion objects of any type "involved" in the implicit parameter we're looking for.

E.g. If we're looking for a `JsonWriter[List[Int]]` the implicit scope includes the companion objects for `JsonWriter`, `List`, and `Int`.

Therefore, *the idiomatic place to put implicit values (type class instances) is in an appropriate companion object*.


### Ambiguous Values

If there are multiple implicit values with the same type in the implicit scope the compiler with signal an error. Ambiguous implicit values are not allowed!

The implication is that you should usually have only one implementation of a type class (i.e. only one type class instance == implicit value) for a given type.


### Type Class Composition

So far implicits look like a shortcut but don't really bring much power. The good stuff is in type class composition---getting the compiler to create type class instances for us based on rules of composition.

This works using `implicit def` (**with implicit arguments!!!!!**) This specifies rules to create type class instances from existing instances. This means the compiler will create type class instances for us if it can find the right base / atomic instances and the right combinators.

```scala
// This says to the compiler: "Give me a Writer for A and I will give you a writer for Option[A]"
implicit def optionWriter[A](implicit w: JsonWriter[A]): JsonWriter[Option[A]] =
  new JsonWriter[A] {
    def write(a: Option[A]): Json =
      a match {
        case None    => JNull
        case Some(v) => w.write(a) 
      }
  }
```


### The Other Implicits

Implicit conversions are `implicit def` without implicit arguments. They are evil and we never ever ever talk about them.

Implicit classes are a way to create "extension methods"---adding methods to an existing class. They are sometimes used with type classes, to add methods from a type class to a type that has a type class instance. E.g. `parMapN`.



## Monoids

A type `A` has a monoid if we can define:
- combine(A, A): A
- empty: A

Where:
- combine is associative
  `combine(x, combine(y, z)) == combine(combine(x, y), z)`
- empty is a commutative identity
  `combine(x, empty) == x == combine(empty, x)`

(Commutivity in general means that `a + b == b + a`)

Examples:
- Integers, Reals: 0, +;   1, *
- Sets: empty set, union
- Booleans: so many! true, and;  false, or
- Vectors: natural extension of monoid for reals

Associativity rules tells us that parallel processing is ok so long as order is maintained.

`x + y + z == (x + y) + z == x + (y + z)` (associativity)

Assume each pair of brackets is a separate node in our cluster (or thread on our machine) this tells us we get the correct answer so long as we keep the order.

For a commutative monoid order doesn't matter, and then parallelism is really easy.

CRDTs: commutative data structure for eventually consistent distributed systems.

Big data applications are fairly straightforward (see Chronos)

Identity means if a machine doesn't have any data to operate on (because, for example, it has all been filtered out) it can still return a value to the rest of the computation (the identity)


## Semigroup

Semigroup is a monoid without an identity. So just the combine operation, with the same condition (it is associative).

Can turn a semigroup into a monoid by wrapping it in `Option` where `None` is the identity.


## Functor

A type constructor `F[_]` has a `Functor` if we can define map:

`def map[A, B](fa: F[A])(f: A => B): F[B]`
