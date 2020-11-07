---
title: "Mirror, mirror on the Wall, Who's the Genericioust of Them All? - Generic Programming with Scala 3" 
date: 2020-10-18T14:00:00+02:00
categories:
  - blog
tags:
  - dotty
  - scala3
  - metaprogramming
  - scala
---

<style type="text/css">
  @import url('https://fonts.googleapis.com/css2?family=Lora:ital@1&display=swap');
</style> 

# Mirror, mirror on the Wall, Who's the Genericioust of Them All? - Generic Programming with Scala 3

Scala 3 (previously called [dotty](https://dotty.epfl.ch/)) is approaching its [release](https://dotty.epfl.ch/blog/2020/09/21/naming-schema-change.html)!
This is a good opportunity to have a deeper look on some new features that it offers. For me one of the most exiting
features are the new metaprogramming abilities that scala 3 offers. Did you ever wonder how json libraries derive 
codecs for you from case classes and sealed traits? Or how tapir generates a whole OpenAPI documentation from your 
endpoint definitions?

But before we start let me tell you the tale of princess Dotty. If you aren't into tales you may want to immediately 
get your hands dirty by [jumping into the technical](TODO) part of the post. 

<section style="font-family: 'Lora', serif;">
## The tale of Princess Dotty and Sir Jon

Once upon a time in the land of Scala there lived brave knight Sir Jon, also called [The Pretty](https://twitter.com/propensive). He fought for the rights
of the poor Types, that were ruled by the evil queen of reflection. The queen would throw the Types in the river of Runtime
by even the slightest approach to derive their birth given instances and flee back to their home, the kingdom of [Compiletime](https://dotty.epfl.ch/docs/reference/metaprogramming/inline.html#the-scalacompiletime-package). 
Sir Jon knew, that a direct fight with the queen would create a lot of logs or even summon the queens feared demons, 
also called "The Exceptions". So he came up with a smart plan: On the foundations of the [isle M'Sabin](https://twitter.com/milessabin) that was hidden from
the evil queen behind [shapeless](https://github.com/milessabin/shapeless) clouds he built the library of [Magnolia](https://github.com/propensive/magnolia) where he taught each Type to construct an instance
to fight against the queen. Jons comrades in arms, the beautiful [Circe](https://github.com/circe/circe) and the 
librarian [Tapir](https://tapir.softwaremill.com/) twittered the wise words
all around the community, so that each Type [implicitly learned to summon](http://dotty.epfl.ch/docs/reference/contextual/using-clauses.html) 
its birth [given instance](http://dotty.epfl.ch/docs/reference/contextual/givens.html). But Jons plan had a 
weakness. When the queen became aware of Jons plan to start a revolution she casted the curse of Compile on the land 
that would terribly slow down any Type trying to summon its instance, which caused lots of downtime. When Jon sensed 
the curse he desperately began to search for a cure. His old friend, the wizard [M'acro](http://dotty.epfl.ch/docs/reference/dropped-features/macros.html) would have been
able to break the curse, but the reign of the queen had exhausted his power, and he was approaching his last days in the 
2nd age of Scala. Jon didn't knew what to do and was close to giving up when a small primitive Type named Int approached 
him to share the rumours of the prophecy of [Odersky](https://twitter.com/odersky). The prophecy was about a long forgotten bloodline and its descendant,
young princess [Dotty](https://dotty.epfl.ch/). It foretold that at the beginning of the 3rd age princess Dotty would appear to free all 
Types from the evil queen of reflection. Jon traveled the whole country, even behind the valleys of [typelevel](https://typelevel.org/), to 
eventually find princess Dotty which lived in the shadows of mountain [E'pfl](https://scala.epfl.ch/). Princess Dotty, who was not aware of her 
role in the prophecy, was surprised that the well-known knight Jon asked her for help but after Jon told her about the prophecy she began to
smile. Her mother Java had given her a magic mirror before she passed away. Java told young Dotty that everyone looking 
in the mirror will see ones soul broken down into its individual parts, so they have to face their real self, as good or as
evil like it is. Jon and Dotty then made up a plan to speak up at the next audience of the queen, pretending to surrender. 
They went there and gave the queen Dottys mirror as a gift to soothe her. The queen then, in the euphoria
of victory unwrapped the gift, when at the first blink the mirror immediately released its [fury](https://github.com/propensive/fury). The queen was shaken by 
her own reflection and she screamed in agony as she realized that Dotty and Jon had tricked her. The magic of the mirror quickly let her fall
into a deep sleep. Jon and Dotty imprisoned the queen in a trifold [IO-Monad](https://zio.dev/) guarded by the mystic [Cats](https://typelevel.org/cats-effect) of [Monix](https://monix.io/) so that she could no 
longer access the river of Runtime. Now all Types were free and could go back to the kingdom of Compiletime. 
But as they approached the border, they realized that the land had dried over time. Sir Jon, Dotty and the Types started searching
for the [stream](https://fs2.io/) that had once fed them and the [Alp](https://doc.akka.io/docs/alpakka/current/index.html) [akkas](https://akka.io/) nearby. 
After three days of search, thirsty and exhausted, Dotty all of a sudden 
noticed that the queen, still trapped inside the IO-Monad, slightly started to glow. The magic sleep 
seemed to heal the broken parts of the queens soul. Suddenly little Int screamed with joy: He remembered that in the 
prophecy of Odersky the magic mirror was said to invert the queens soul. She was foreseen to become the queen of [mirrors](http://dotty.epfl.ch/docs/reference/contextual/derivation.html)!
The queen, woken up by little Ints scream, slowly began to speak in a deep voice: "You freed me
from my own curse, princess Dotty and Sir Jon. I know that the Types want to live in a flourishing land and so may it be.
You need to release the river of Runtime to again flow through the kingdom to enter the era of generic programming." - 
"Yeah, sure" said Sir Jon and laughed. "How could we forget about that?". Then the Types broke the old dam that held back
the river of runtime and it released its refreshing energy into the kingdom of Compiletime. Dotty then freed the queen
and they celebrated a big party with Jon and the Types under the beats of [DJ Caliban](https://github.com/ghostdogpr/caliban) 
where they drank a lot of [sangria](https://sangria-graphql.github.io/) 
and ate many [burritos](https://emorehouse.wescreates.wesleyan.edu/silliness/burrito_monads.pdf). Dotty had finally 
[unioned](http://dotty.epfl.ch/docs/reference/new-types/union-types.html) all Types under the righteous queen of mirrors like the prophecy of 
Odersky foresaw. And they lived in [equality](http://dotty.epfl.ch/docs/reference/contextual/multiversal-equality.html) happily ever after, without any exceptions.

</section> 

// TODO: Überleitung

These libraries do that by deriving typeclasses, which provide a defined set of functionality for a given data type. 
For example a json codec or an API description. In scala 2 this was only possible by using macros and more or less 
complex constructs which can lead to very long compile times. Also scala 2 macros will not be supported anymore in scala 3.

Instead, scala 3 offers us builtin tools to achieve what was done with macros before, which will hopefully also lead to 
better IDE support and faster compile times.

In this blogpost we will step by step look into how to derive a custom typeclass for any data type and also provide a 
powerful tool to do that derivation by just implementing some simple functions.

## The Tuple <=> Case Class duality and Typeclasses TODO: typelevel strings

Lets first check the underlying concepts. When looking at any case class we can also describe that case class by 
breaking it down into its fields and represent it as a tuple as shown below.

```scala
case class User(name: String, age: Int) 
val someUser: User = User(name = "Bob", age = 25)
val someUserAsTuple: (String, Int) = ("Bob", 25)
```

So a case class is just a tuple with some labels and it's own type. But in terms of the underlying data, 
which in this case are values of type String and Int, there is no difference between a tuple and a case class. 
That means that we can also describe a case class by a tuple with some labels, which will be important later.

The second important concept is that of typeclasses. Let's look at a simple typeclass:

```scala
trait PrettyString[A] {
  def prettyString(a: A): String // when implemented, prints a type A in a pretty, human readable way
}
```

The `PrettyString` typeclass provides us with the ability to convert any type to a string, which is similar to what
the `.toString` method does, but we will do it in a more pretty way, e.g. also print the fieldnames of case classes, 
wrap strings in quotes etc.

Every typeclass is characterized by a generic type parameter `A` and some functions or values that it 
provides (in our case `def prettyString`). The typeclass can then be used to implement instances for arbitrary types:

```scala
val intPrettyString = 
  new PrettyString[Int] {
    def prettyString(a: Int): String  = a.toString
  }

val stringPrettyString = 
  new PrettyString[String] {
    def prettyString(a: String): String = s"\"$a\"" // notice: string is printed in quotes!
  }

val userPrettyString = 
  new PrettyString[User] {
    def prettyString(a: User): String = 
      s"User(name=${stringPrettyString.prettyString(a.name)}, age=${intPrettyString.prettyString(a.age)})"
  }

println(intPrettyString.prettyString(5)) // prints 5
println(stringPrettyString.prettyString("hello world")) // prints "hello world"
println(userPrettyString.prettyString(User("Bob", 25))) // prints User(name="Bob", age=25)
``` 

So by defining the instances `intPrettyString`, `stringPrettyString` and `userPrettyString` we did provide the ability 
to pretty print the types `Int`, `String` and `User`. What is also shown is that the typeclass instance for `User` is 
(manually) derived from the instances for primitive types `Int` and `String`. In the end we will see how to do the 
derivation automatically for any case class or sealed trait by using scala 3 tools.

# TODO

As shown above we did create a typeclass instance for `User` by hand but in scala 2 the libraries could to that 
automatically by using macros (e.g. circe for json codecs or tapir for api descriptions). 

In Scala 2 the libraries used macros to achieve the automatic derivation from case classes or sealed traits which often 
slowed down compile times and was difficult to implement. Magnolia, a library for generic derivation made that a lot 
easier but it still depends on scala 2 macros which will no longer be supported in scala 3.

To our rescue scala 3 has builtin derivation utilities. In the section below we will discover how to derive a typeclass
instance for our `PrettyString` typeclass for any case class or sealed trait.

## A look in the Mirror

The tool that scala 3 provides us with that will allow us to do the derivation is the Mirror trait:

```scala
// taken from https://dotty.epfl.ch/docs/reference/contextual/derivation.html

// shortened
sealed trait Mirror {
  type MirroredType // the type that was mirrored itself
  type MirroredLabel <: String // the label of the mirrored type
  type MirroredElemTypes <: Tuple // a tuple of types for each individual element
  type MirroredElemLabels <: Tuple // a tuple of the labels for each individual element
}
```

Ok, that's a lot of types. First thing to notice is that `Mirror` is providing us with *typelevel* information only. 
Which makes sense, as the derivation of the typeclass happens at compiletime.

So what do the different types mean:

 - `MirroredType` - is just the type we are deriving from, e.g. our case class `User`
 - `MirroredLabel` - typelevel representation of the label of the type we mirror, so the string `"User"`
 - `MirroredElemTypes` - A tuple of all elements that `User` can be broken into: `(String, Int)` (for fields name and age)
 - `MirroredElemLabels` - A Tuple that contains the member names at typelevel, so of type `("name", "age")`
 
Now we've got some useful things, like the labels and the types of the individual elements that our type consist of. 
But we still need some extra information if our type is a case class or a sealed trait. 
 
Thankfully two traits extend `Mirror` which are:
 
 ```scala
// represents a case class
trait Product extends Mirror {
 def fromProduct(p: scala.Product): MirroredType
}

// represents a sealed trait
trait Sum extends Mirror { self =>
 def ordinal(x: MirroredMonoType): Int
}
 ```
 
Yay, we can distinguish between case classes and sealed traits! A small site note: Case classes are also called product
types and sealed traits sum types (hence the naming of the traits). Also we've got some methods that we can call at
runtime: For `Product` a method to assemble the type from its elements and for `Sum` one that gives us the 
ordinal of a member of the respective sealed trait.

Let's now recheck how the Mirror type would look like for our `User` case class leaving out the `fromProduct` 
method which is not relevant for now.

```scala
val userMirror: Product = new Product with Mirror {
  type MirroredType = User
  type MirroredLabel = "User"
  type MirroredElemTypes = (String, Int)
  type MirroredElemLabels = ("name", "age")
} 
```
 
 **********************************

Before we start let me introduce to you the concept of product-types and sum-types. Most likely you already know them 
by different names. Ever heard of case classes and sealed traits? Case classes represent product types and sealed 
traits are sum types. 

With Scala 3 (or [dotty](https://dotty.epfl.ch/)) approaching its [release](https://dotty.epfl.ch/blog/2020/09/21/naming-schema-change.html)
let's have a look at one of the new features: Mirror

It allows us to break down product types (case class) and sum types (sealed trait) into their individual parts. 
Using that we can automatically derive typeclasses for complex types. In Scala 2 it was necessary to
use macros or libraries like shapeless or magnolia. In Scala 3 this will be part of the standard feature set of 
the compiler. 

Ok, to start let's pick a simple typeclass: `AsString` (also known as `Show`), that is the typeclass equivalent of the `toString` method. 
<!---
TODO: In case you haven't heard of typeclasses see this excellent post about them. 
-->

```scala
trait AsString[A] {
  // "asString" because "toString" is already defined for any object on the JVM
  def asString(a: A): String 
}
```

Now add a simple data type for demonstration: 

```scala
enum User { // enum is the new, elegant scala 3 way to define sealed traits
  case Anonymous
  case RegisteredUser(name: String, age: String)
}
```

Ok so far so clear. Besides enum instead of sealed trait this could have been scala 2 code. So let's take a look in the mirror:

```scala
// taken from https://dotty.epfl.ch/docs/reference/contextual/derivation.html

sealed trait Mirror {

  /** the type being mirrored */
  type MirroredType

  /** the type of the elements of the mirrored type */
  type MirroredElemTypes

  /** The mirrored *-type */
  type MirroredMonoType

  /** The name of the type */
  type MirroredLabel <: String

  /** The names of the elements of the type */
  type MirroredElemLabels <: Tuple
}
```

// TODO: explain typelevel strings

Ok, that's a lot of types. First thing to notice is that Mirror is providing us with *typelevel* information only. 
Which makes sense, as the derivation of the typeclass happens at compiletime. 

So what do the different types mean:

 - `MirroredType` - is just the type we are deriving from, so in our case the enum `User`
 - `MirroredElemTypes` - is the type of all elements that `User` can be broken into: `Anonymous` and `RegisteredUser`
 - `MirroredMonoType` - TODO
 - `MirroredLabel <: String` - typelevel representation of the label of the type we mirror, so `User` (see here to check out about values at the typelvel TODO)
 - `MirroredElemLabels  <: Tuple` - A Tuple that contains the member names at typelevel, so of type `("Anonymous", "User")`
  
Now we've got some useful things, like the labels and the types of the individual elements that our type consist of. 
But what we still need is the information if it is a product or sum type.

Thankfully two traits extend `Mirror` which are:

```scala
object Mirror {
  /** The Mirror for a product type */
  trait Product extends Mirror {

    /** Create a new instance of type `T` with elements taken from product `p`. */
    def fromProduct(p: scala.Product): MirroredMonoType
  }

  trait Sum extends Mirror { self =>
    /** The ordinal number of the case class of `x`. For enums, `ordinal(x) == x.ordinal` */
    def ordinal(x: MirroredMonoType): Int
  }
}
```

Yay, we can distinguish between sum and product types! Also we've got some methods that we can call at
runtime: For `Product` a method to assemble the type from its elements and for `Sum` one that gives us the 
ordinal of a value.

Ok, so far we learned what the Mirror trait provides us with and which subtypes exist that reflect our data types.
Let's start using it. Before we start with that I want to mention that in scala 3 implicits were revised
 to differentiate between their different meanings and use cases. For typeclass instances we will use the keywords 
 `given` and `using` in scala 3.  

```scala
// scala 2
implicit intAsString: AsString[Int] = new AsString[Int] { def asString(a: Int): String = a.toString }
def print[A](value: A)(implicit asStringInstance: AsString[A]) = asStringInstance.asString(value)

// scala 3
given intAsString as AsString[Int] { def asString(a: Int): String = a.toString }
def print[A](value: A)(using asStringInstance: AsString[A]) = asStringInstance.asString(value)
```
I know the syntax looks strange for the scala 2 user at first sight, but you will quickly get used to it. 
As you can see this reflects the intention way better then scala 2's implicit: `given` the instance intAsString 
execute the method `print` by `using` that instance.

Now back to deriving our `AsString` typeclass. Starting with the easiest step we derive the type label first. 
As already said above the label is encoded as a typelevel string in the Mirror trait:

```scala
sealed trait Mirror {
  // ...
  type MirroredLabel <: String
  // ...
}
``` 
 
To print it we need to make an actual value out of this type. This can be done via the new compiletime functions:

```scala
import scala.compiletime.constValue

// type to value
val str = constValue["Hello"] // note that strings on typelevel are also surrounded by quotes
println(str) // prints "Hello"

// get the label using Mirror
def labelFromMirror[A](using m: Mirror.Of[A]): String = constValue[m.MirroredLabel]

labelFromMirror[User] // "User"
labelFromMirror[Anonymous] // "Anonymous"
labelFromMirror[RegisteredUser] // "RegisteredUser"
```

Next step would be to not only get the label but also the labels of the elements.

```scala
inline def summonElemLabels[A <: Tuple]: List[String] = inline erasedValue[A] match {
    case _: EmptyTuple => Nil
    case _: (t *: ts) => constValue[t].toString :: summonElemLabels[ts]
  }

summonElemLabels[User] // List("Anonymous", "RegisteredUser")
summonElemLabels[RegisteredUser] // List("name", "age")
```

Ok, this method looks very frightening. But don't worry we will check on it step by step. First, the `inline` keyword
guarantees that the code is inlined at use site, even for recursive calls. `inline erasedValue` in combination with 
`match` is a trick to summon a value by matching on a type. Looking at the 2 match cases we can the that the first one
will just return Nil in case of an empty tuple and the second is matching on a non-empty tuple by splitting it by the 
type for the head element `t` and. 

Similar to that constValue is used to create values from
singleton types. Singleton types are the representation of primitive values like numbers or strings at type level. 
In our case the summoned string is the element label.