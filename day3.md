# ??? Advanced Scala Day 3

## Covariance, Contravariance, and Invariance

Variance is about how subtyping relationships between complex types relate to subtyping relationships between their components.

Imagine we have some type `A` that is a subtype of `B`. `A <: B`. Then how does some type `F[A]` relate to `F[B]`? This is what variance addresses.

For example, imagine we have some event type `Event` and a type `PurchaseEvent` which is a subtype of `Event`. So `PurchaseEvent <: Event`. Then, should a `List[PurchaseEvent]` be a subtype of `List[Event]`? What about a function `PurchaseEvent => Array[Byte]`? Is that a subtype of `Event => Array[Byte]`?


### Liskov Substitution Principle

If `A` is a subtype of `B`, then anywhere we use objects of type `B` we should be able to substitute objects of type `A` without changing the behaviour of our program. We write `A <: B` to indicate `A` is a subtype of `B`.

In Scala, if `A extends B` then `A` is a subtype of `B` as far as the compiler is concerned. It's up to us to make sure the property above holds.


### Subtyping and Type Constructors

We often have types like `List`, `Option`, and `Function1` that are not strictly types, but rather type constructors. For example, `List[Int]` is a type, but `List` on its own is not---we need to specify the type of the elements. We can consider `List` to a be a function *at the type level* that, when supplied with a type (the type of the elements) produces a type (the type of the list containing those elements). We call these type constructors.

Given some type constructor `F` and some subtyping relationship between `A` and `B` we can have the following relationships between `F[A]` and `F[B]`:

|----------------|----------------------------|----------------|
| A & B          | F[A] & F[B]                | Name           |
|----------------|----------------------------|----------------|
| A <: B         | F[A] <: F[B]               | Covariance     |
| A >: B         | F[A] <: F[B]               | Contravariance |
| Doesn't matter | No subtypiing relationship | Invariance     |
|----------------|----------------------------|----------------|

Covariance means the subtyping relationship between the type constructors varies in the same way as the component types.

Contravariance means the subtyping relationship between the type constructors varies in the opposite way as the component types.

Invariance means there is no relationship between them. This is the default.


### Examples

CreditCardPurchaseEvent <: PurchaseEvent <: Event

List[PurchaseEvent]  <:  List[Event] Covariant?
List[PurchaseEvent]  no relationship to  List[Event] Invariant?

It seems to be covariant.

It is covariant. The + in front of the type variable A indicates covariance.

When we declare a type variable (generic type) we can optionally specify variance.
- default is invariant
- + means covariant
- - means contravariant


Serializer. Event => Array[Byte]
If I'm working with a PurchaseEvent and I want a serializer (PurchaseEvent => Array[Byte]), which of the below can I use (i.e. which is a subtype of PurchaseEvent => Array[Byte]):

- PurchaseEvent => Array[Byte] (what I want; this is fine by definition)
- Event => Array[Byte] <--- this one?
- CreditCardPurchaseEvent => Array[Byte]
- all of them?

Event => Array[Byte] because functions are contravariant in their input.

Inputs: we want something that accepts our type specifically or a larger set of types (a super type)

Outputs: we want something that gives us our type specifically, or a more specific set of types (subtypes)


Vet: Animal => HealthyAnimal
SmallAnimalVet: SmallAnimal => HealthyAnimal
LargeAnimalVet: LargeAnimal => HealthyAnimal

Siamese <: Cat <: SmallAnimal <: Animal
Horse <: LargeAnimal <: Animal

Cat is ill :-(
- Vet? Yes
- LargeAnimalVet? Nope
- SmallAnimalVet? Yes
- CatVet? Yes
- SiameseCatVet? Nope

```scala
trait Vet[-A <: Animal] {
  def treat(patient: A): HealthyAnimal
}
final case class CatVet() extends Vet[Cat] {
  def treat(patient: Cat): HealthyAnimal =
    patient.fix
}
final case class IntVet() extends Vet[Int] // Int is not <: Animal so this won't compile
```

```scala
trait Hospital {
  def treat(patient: Mammal): Healthy =
    patient.bill
}
trait Vet {
  def treat(patient: Mammal): Healthy = 
    patient.fix
}
object CatVet extends Hospital with Vet {
}
```
Trait linearization specifies with implementation you get.

***NEVER EVER EVER WRITE CODE LIKE THIS***

```scala
def treatEmAll(animals: List[SmallAnimal]): Unit =
  animals.foreach(a => a.fix)
```

type variable `[A]` means for all types, or any type
type variable `[A <: B]` means for all types that are a subtype of `B`

You never need to use variance. 


Invariance: mutable containers.

Array[Cats]

def addAnAnimal(animals: Array[Animal]) =
  animals(0) = Dog


## Thoughts

Effect types & type classes
Distributed tracing
Audit trails
Error handling
