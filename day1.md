# ??? Advanced Scala Day 1

## Introductions

Name, programming background, kinds of things you work on and are interested in.

Noel: Scala 10 years, ML, PL, teaching
Arun: Nemesis, Java, apply FP to build real world systems
Eric: C++, Objective-C, Python, client side and backend. Video games.
Kaushik: C++, C, Java, Python. Past 5 years moving to Scala (previous job, Cats, IO). Advanced topics, cutting edge stuff.
Nevin: Python, Java, C++. Web development, Javascript, Typescript. Mobile app development, Kotlin, Swift. A bit of Scala in previous job. R & Matlab in small quantities. Explore a new language, learn more about FP.
Akash: Nemesis, NodeJS. Understand how to write production level code.
Arsalan: Web, Python. 1.5 years Scala. Data lake governance. Safety. Types. Make code accessible. Patterns / strategies.
Neelam: Data lake / data engineering team. 3.5 years. Working w/ Scala for 2 years. Previously Java, C / C++. Interested in learning about metaprogramming, Cats.
Karine Mellata: SE. First job out of college. 1 year Scala. Goal is design systems in Scala. Architecture / design. Efficient code for concurrency. Data engineering.
Swapna Marru
Yash Kotresh


## Admin

1. Cameras / mics on please, unless you have a good reason
2. Pets (and small children) are welcome to join us, but you should introduce them to everyone else. Older children, friends, housemates, partners, etc. are also welcome.
3. You are ultimately responsible for your learning. Three hints:
   1. Ask questions at any time. "I don't understand this and I don't know how to form a clearer question" is a great question!
   2. Collaborate with other students. Explain what you are doing to them. Explain what you're thinking. "I don't know what to do right now" is a great thing to say.
   3. Take notes.

Goals
1. Overall goal: learn how to design and build systems in Scala
2. Basic principles of functional programming and their application in system design. The importance of semantics.
3. Interpreters and their implementation techniques
4. Type classes in general, and specific type classes that are useful

Admin
- Our curriculum is not fixed. We have a rough plan but it will change depending on your interests.
- We have 5 3.5 hr sessions. We'll have a 30 min break about half way through


## Streaming

Questions:
- +A on Stream
- >: bounds
- :: method
- Emit could values be iterator of constant of A?
- Map extends Stream[B] not Stream[A]---why is this?
- Nothing type
- parameters to foldLeft: type parameter B, multiple parameter lists
- append / merge / zip are repeated in Ir---why?
- difference between case class and case object
- some methods inside sealed trait and some in the companion object---why?
- what are interpreters and combinators?
- why do we have the compile method?
- is this a type class and if so why?
- why do we have them case classes in the companion object?
- what is implementation strategy for stream?
- what are the semantics for merge & interleave?


## Algebraic Data Types

Algebraic data types are:
- representing data using logical or and logical and to connect data elements
- implemented in Scala using sealed trait and final case classes

```scala
// A is B or C
sealed trait A
final case class B() extends A
final case class C() extends A

// A is B and C
final case class A(b: B, c: C)
```

Wrinkles:
- can use `sealed abstract class` instead of `sealed trait`
- can sometimes use `case object` instead of `final case class`
- often you will see just `case class` instead of `final case class` (not strictly an algebraic data type because a case class can be extended by a class (but not another case class) but this is almost always an error.)

Algebraic data types are a closed world. Cannot be extended without modifying source code.

Terminology:
- sum type is used for logical or
- product type is used for logical and

## Structural Recursion

Any time we want to transform an algebraic data type we can use structural recursion.

There are two techniques for implementing structural recursion in Scala:
- pattern matching
- dynamic dispatch

### Pattern Matching

Classic FP

```scala
// A is a B or C
anA match {
  case B => ???
  case C => ???
}

// A is a B and C
case A(b, c) => ???
```

### Dynamic Dispatch

Classic OO

```scala
sealed trait A {
  def aMethod: D
}
final case class B() extends A {
  def aMethod: D = ???
}
final case class C() extends A {
  def aMethod: D = ???
}
```

### Recursion Rule

When the data is recursive the method is recursive.
