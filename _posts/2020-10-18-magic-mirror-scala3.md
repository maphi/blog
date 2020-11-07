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

# Mirror, mirror on the Wall, Who's the Genericioust of Them All? - Generic Programming with Scala 3

## The tale of Princess Dotty and brave knight Jon

Once upon a time in the land of Scala there lived brave knight Sir Jon, also called The Pretty. He fought for the rights
of the poor Types, that were ruled by the evil queen of reflection. The queen would throw the Types in the river of Runtime
by even the slightest approach to derive their freedom and flee back to their homeland, the land of Compiletime. 
Jon the knight knew, that a direct fight with the queen would create a lot of logs or even summon the queens feared demons, 
also called "The Exceptions". So he came up with a smart plan: On the foundations of the isle M'Sabin that was hidden from
the evil queen behind shapeless clouds he built the library of Magnolia where he taught each Type to construct an instance
to fight against the queen. Jons comrades in arms, the beautiful Circe and the wise animal Tapir twittered the wise words
all around the community, so that each Type implicitly learned to summon its birth given instance. But Jons plan had a 
weakness. When the queen became aware of Jons plan to start a revolution she casted the curse of Compile on the land 
that would terribly slow down anyType trying to summon its instance, which caused lots of downtime. When Jon became 
aware of the curse he desperatelysearched a cure for the queens curse. His old friend, the wizard M'acro would have been
able to break the curse, but the reign of the queen had exhausted his power and he was approaching his last days in the 
2nd age of Scala.Jon didn't knew what to do and was close to giving up when a small primitive Type named Int approached 
him to share therumours of the prophecy of Odersky. The prophecy was about a long forgotten bloodline and its descendant,
the young princess Dotty. It foretold that at the beginning of the 3rd age princess Dotty would appear to free all 
Types from the queen of reflection. Jon traveled through the whole country, even behind the valleys of typelevel, to 
eventually find princess Dotty which lived in the shadows of mountain E'pfl. Princess Dotty, who was not aware of her 
role in the prophecy was surprised that the well known knight Jon asked her for help but after Jon told her she began to
smile. Her mother Java had given her a magic Mirror before she passed away. Java told young Dotty that everyone looking 
in the mirror will see ones soul broken down into its individual parts, so they can face their real self, as good or as
evil like it is. Jon and Dotty then made up a plan to have an audience with the queen, pretending to surrender. They 
would then give the queen Dottys mirror as a present to ask the queen to spare them. The queen then, in the euphoria
of victory opened the present, when the mirror immediately released its fury. The queen was shaken by the sad sight of 
her soul and she screamed as she realized that Dotty and Jon had tricked her. The magic of the Mirror let her fall
into a deep sleep. Jon and Dotty then put the queen into a Future-Box so that she could no longer access the 
river of Runtime. Now all Types were free and could go back to the land of compiletime, but something was missing. 
The land had dried over time and Jon, Dotty and the Types were searching for water in the valleys and on the hills with
no success. After three days the queen, still trapped inside the Future-Box, slightly began to glow. The magic sleep 
seemed to heal the broken parts of the queens soul. Suddenly little Int screamed with joy: He remembered that in the 
prophecy of Odersky the magic mirror was said to invert the queens soul. She was foreseen to become the queen of mirrors.
The queen, woken up by little Ints scream, slowly the queen began to speak: "You freed me
from my own curse, princess Dotty and Sir Jon. I know that the Types want to live in a blooming land and so may it be.
The river of Runtime needs to flow through the land of compiletime to let the Empire of Programming bloom again." - 
"Yeah, sure" said Sir Jon and laughed. "How could we forget about that?". Then the Types broke the old dam that held back
the river of runtime and it released its refreshing energy into the land of compiletime. Dotty then freeded the queen
and they celebrated a big party with Jon and the Types. And they lived happily ever after, without any exceptions. 

Curse of Macro
shapeless clouds
generic empire
bibliothecarian tapir
typelevel cats
Princess dotty sealed the traits
Jon's beautiful words formed a PrettyString.
Jon - mit oder ohne H  
The types were standing inline, transparently tupling up to whole case classes. 
Isle M'Sabin <=> Ebene von Typelevel 
river of runtime vanishing in the hole of logs
The types could escape, with no exception.
queen 
Release the fury       
Queen of reflection becomes rechtschaffene Queen of mirrors
Sir Jon, the knight of the long forgotten Kingdom of Generics    

Scala 3 (previously called [dotty](https://dotty.epfl.ch/)) is approaching its [release](https://dotty.epfl.ch/blog/2020/09/21/naming-schema-change.html)!
This is a good opportunity to have a deeper look on some new features that it offers. For me one of the most exiting
features are the new metaprogramming abilities that scala 3 offers. Did you ever wonder how json libraries derive 
codecs for you from case classes and sealed traits? Or how tapir generates a whole OpenAPI documentation from your 
endpoint definitions?

These libraries do that by deriving typeclasses, which provide a defined set of functionality for a given data type. 
For example a json codec or an API description. In scala 2 this was only possible by using macros and more or less 
complex constructs which can lead to very long compile times. Also scala 2 macros will not be supported anymore in scala 3.

Instead, scala 3 offers us builtin tools to achieve what was done with macros before, which will hopefully also lead to 
better IDE support for derivation and faster compile times.

In this blogpost we will step by step look into how to derive a custom typeclass for any data type and also provide a 
powerful tool to do that derivation by just implementing some simple functions.

## The Tuple <=> Case Class duality and Typeclasses 

Before we start let us have a look at the underlying concepts. 

When looking at any case class we can also describe that case class by breaking it down into its fields and represent it as a tuple as shown below.

```scala
case class User(name: String, age: Int) 
val someUser: User = User(name = "Bob", age = 25)
val someUserAsTuple: (String, Int) = ("Bob", 25)
```

So a case class is just a tuple with some labels and it's own type. But in terms of the underlying data, 
which in this case are values of type String and Int, there is no difference between a tuple and a case class. 
That means that we can also describe a case class by a tuple with some labels, which will be important later.

The second important concept is that of typeclasses. Let's look at a simple typeclass for that we will implement the generic derivation later:

```scala
trait PrettyString[A] {
  def prettyString(a: A): String // when implemented, prints a type A in a pretty, human readable way
}
```

Every typeclass is characterized by a generic type parameter `A` and some functions or values that it 
provides (in our case `def prettyString`). The typeclass can then be used to implement instances for any type:

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
println(userPrettyString.prettyString(User("Bob", 25"))) // prints User(name="Bob", age=25)
``` 

So by defining the instances `intPrettyString`, `stringPrettyString` and `userPrettyString` we did provide the ability 
to pretty print the types `Int`, `String` and `User`. What is also shown is that the typeclass instance for `User` is 
(manually) derived from the instances for primitive types `Int` and `String`. In the end we will see how to do the 
derivation automatically for any case class or sealed trait by using scala 3 tools.

## A look in the Mirror


  
In Scala 2 macros were needed to derive typeclasses (e.g. to automatically create a json codec) from case classes or 
sealed traits which was often cumbersome and slowed down compile times. Magnolia, a library for generic derivation made 
that a lot easier but it still depends on macros which will no longer be supported in scala 3.

To our rescue scala 3 provides us with such abilities with builtin tools. In this blog post we will discover how to
derive a typeclass from data classes. 

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