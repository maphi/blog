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
This is a good opportunity to have a deeper look at some new features it offers. For me one of the most exiting
features are the new metaprogramming abilities that scala 3 offers. Did you ever wonder how json libraries derive 
codecs for you from case classes and sealed traits? Or how tapir generates a whole OpenAPI documentation from your 
endpoint definitions? In this blog post we will discover how to do things like that in scala 3 and discover some new 
language features like given, singleton types, reworked tuples, the exiting new enum definitions and finally the Mirror 
trait that allows us to derive typeclass instances automatically for our data types. If you haven't heard of this things
yet, don't be afraid. We will dive into them step by step. Even if the concepts of this post are between intermediate
to advanced level, i will try to explain it so that it is also understandable for people that already got some
experience with scala but haven't explored these concepts yet. If there is something you do not understand please don't
hesitate to [ask](TODO).    

But before we start let me tell you the tale of princess Dotty. For the impatient reader: You can skip the tale by
clicking [here](TODO). 

<section style="font-family: 'Lora', serif;">
## Princess Dotty, Sir Jon and the Queen of Reflection

Once upon a time in the land of Scala there lived a brave knight. His name was Sir Jon but most people just called him [The Pretty](https://twitter.com/propensive). 
He was the defender of a poor folk which called itself "The Types". Many year ago they were abducted from the kingdom of
[Compiletime](https://dotty.epfl.ch/docs/reference/metaprogramming/inline.html#the-scalacompiletime-package)
by the evil Queen of Reflection. The queen ruled with terrible harshness. Everyone who tried to flee 
from her frightening reign would be thrown into the river of Runtime, that flowed into the great log pool, to never be
seen again. 
 
Sir Jons plan was to build an army in secret, but he knew that a direct fight with the queen would not be possible. The
queen was guarded by an army of dreaded demons which were called "The Exceptions" by the people, who did not dare to
pronounce their names. That why he thought of a smart plan: On the foundations of the [isle M'Sabin](https://twitter.com/milessabin), 
that was hidden from the evil queens henchmen behind the [shapeless](https://github.com/milessabin/shapeless) clouds,
he built the library [Magnolia](https://github.com/propensive/magnolia). From there he twittered his wise words all 
around the community of types, that came to learn at his [zone](https://scala.zone/). He would teach each Type how to 
`summon` their birth-`given` instance to defend themselves from the queens army.

But the queen became aware of Jons plan to start a revolution and she casted a curse on the land that would 
terribly slow down any Type trying to summon its instance, which caused lots of compiletime. When Jon became aware of 
the curse he desperately began to search for a cure. His old friend, the wizard M'acrowas a skilled witcher and wise 
man but the reign of the queen had exhausted his power, and he was approaching his
[last days in the 2nd age]((http://dotty.epfl.ch/docs/reference/dropped-features/macros.html)) of Scala. 
Jon didn't knew what to do and was close to giving up when a small primitive Type named Boo Lean approached 
him to share the rumours of the prophecy of [Odersky](https://twitter.com/odersky). 

The prophecy was about a long forgotten bloodline and its descendant, young princess [Dotty](https://dotty.epfl.ch/).
It foretold that at the beginning of the 3rd age princess Dotty would appear to free all Types from the queen. 
Jon traveled the whole country, even behind the valleys of [typelevel](https://typelevel.org/), to 
eventually find princess Dotty which lived in the shadows of mountain [E'pfl](https://scala.epfl.ch/) in a small 
village with the strange name named Rele Asecan Didate. Princess Dotty, who was not aware of her role in the prophecy, 
was surprised that the well-known knight asked her for help as she was still a young girl and hadn't been [released](https://dotty.epfl.ch/blog/2020/09/21/naming-schema-change.html)
outside of her village yet, but after Jon told her the story she began to smile. 

Her mother Java had given her a magic mirror before she passed away. Java told young Dotty that everyone looking 
in the mirror will see his soul breaking into its elementary parts andface their real self, as good or as
bad like it is. 

The following day Jon and Dotty made up a plan to speak up at the next audience of the queen under the excuse to
surrender. At the audience they gave the queen Dottys mirror as a gift to soothe her. The queen then in her arrogance
and with a toxic smile in her face began to unwrap the gift. But at the first blink the mirror immediately released its
[fury](https://github.com/propensive/fury). The queen was shaken by her own reflection and screamed in agony as she 
realized that Dotty and Jon had tricked her. The magic of the mirror quickly made her fall asleep. Jon and Dotty 
imprisoned the queen in a trifold [IO-Monad](https://zio.dev/) guarded by the mystic
[Cats](https://typelevel.org/cats-effect) of [Monix](https://monix.io/) so that she no longer could cause any 
side effects. 

When they heard of Dottys and Jons victory over the queen the Types started dancing and went on a march back to their
kingdom of Compiletime. But as they approached the border, they realized that the land had dried over time. 
They started searching for the [stream](https://fs2.io/) that had once fed them and the 
[Alp](https://doc.akka.io/docs/alpakka/current/index.html) [akkas](https://akka.io/) nearby. 

After three days of search, thirsty and exhausted, Dotty all of a sudden noticed that the queen, still trapped inside 
the IO-Monad, slightly started to glow. The magic sleep seemed to heal the broken parts of the queens soul. 
Little Boo Lean then remembered that in the prophecy of Odersky the magic mirror was told to be able to invert 
the queens soul. She was foreseen to become the Queen of 
[Mirror](http://dotty.epfl.ch/docs/reference/contextual/derivation.html), a righteous and kindly ruler!
The queen, woken up by little Ints scream, slowly began to speak in a deep voice: "You freed me
from my own curse, princess Dotty. I know that the Types want to live in a flourishing land and so may it be.
You need to release the river of Runtime to flow through the kingdom to enter the era of generic programming". - 
"Yeah, sure" said Sir Jon and laughed. "How could we forget about that?". 

The Types then went to the old dam that held
back the river of runtime and it released its refreshing energy into the kingdom of Compiletime. After that Dotty freed
the queen and they celebrated a big party with Jon and the Types where they ate many
[burritos](https://emorehouse.wescreates.wesleyan.edu/silliness/burrito_monads.pdf). Dotty had finally 
[unioned](http://dotty.epfl.ch/docs/reference/new-types/union-types.html) all Types under the Queen of Mirrors
like the prophecy of Odersky foresaw. And they lived in 
[equality](http://dotty.epfl.ch/docs/reference/contextual/multiversal-equality.html) happily ever after,
without any exceptions.

</section>

Even if it is only a tale that kids have been listening to for centuries there lies some truth within that you may run 
across while reading this post. Before we get our hands dirty let's dive into some new and old concepts first.

## Concepts

### Typeclasses

The first important concept to understand is that of typeclasses. You might already know about it and if not don't worry. 
I will show you how to use it and explain the new scala 3 syntax around typeclasses. Instead of talking about the theory
behind typeclasses let's look at an example:  

```scala
trait PrettyString[A] {
  def prettyString(a: A): String // will print a type A in a pretty, human readable way
}
```

Pretty simple, right? Just a simple interface with a type parameter `A` and a method that takes a parameter and returns
`String`. So this provides us with the ability to convert any type `A` to a string. So why is that useful? 
Imagine having a type that you can not extend e.g. `Int` or `String` but you want to add some extra functionality in a 
way that also works for other types.

That can be done by implementing the `PrettyString` trait from above for that types:

```scala
val intPrettyString = 
  new PrettyString[Int] {
    def prettyString(a: Int): String  = a.toString
  }

val stringPrettyString = new PrettyString[String] {
  def prettyString(a: String): String = "\"" + a + "\""
}

println(intPrettyString.prettyString(123)) // prints 123
println(stringPrettyString.prettyString("hello world")) // prints "hello world"
``` 

You may now think "Great, this is like the `.toString` method, but much more difficult to use", 
but calm down young padawan. First our `PrettyString` instance for `String` wraps the result in quotes which `toString`
does not (yeah, awesome - i know) and second you can do super cool stuff with it which we'll see later.

But in the code above converting something to a string included a lot of boilerplate. Let's get rid of that.

### Implicits 3.0 - Give it, use it, quick enjoy it

You might have heard that in scala 3 implicits have been reworked to make their different meanings and use cases
clearer. One of this use cases is typeclasses. Instead of using implicit here two new keywords were added:
`given` and `using`.   

To make printing the result of `PrettyString.prettyString` more pleasing let's introduce the method `prettyPrintln`. We
will also **give** the compiler our `PrettyString` instances from above so that we can **use** them without explicitly
handing them over to the `prettyPrintln` method. 

```scala
given intPrettyString as PrettyString[Int] {
    def prettyString(a: Int): String = a.toString
  }

given stringPrettyString as PrettyString[String] {
  def prettyString(a: String): String = "\"" + a + "\""
}

def prettyPrintln[A](a: A)(using prettyStringInstance: PrettyString[A]) = 
  println(prettyStringInstance.prettyString(a))

prettyPrintln(123) // prints 123
prettyPrintln("hello world") // prints "hello world"

// we can also pass the PrettyString instance explicitly
prettyPrintln(123)(using intPrettyString)
``` 

The given syntax looks slightly strange at first sight, but you will get *used* to it pretty quickly. In scala 2 `given`
and `using` would both have been `implicit` but in scala 3  it was reworked to clarify intent:
`given` the instance `intPrettyString` we can call `prettyPrintln` `using` the `PrettyString` instance for the type
`Int`. The compiler will then take care of passing the `intPrettyString` parameter to `prettyPrintln`.

The next thing we will check out is the relation between case classes and tuples.

## Singleton Types and The Literal Next Door

Since Scala 2.13 literals do not only exist in the value space but also on type level. Each literal has it's
corresponding counterpart in typelevel which is written exactly the same. So instead of writing `val myInt: Int = 1` you
can be more precise by writing `val myInt: 1 = 1`. Of course if you would try `val myInt: 2 = 1` that would not compile.
It also works with strings: `val helloWorld: "Hello world!" = "Hello world!"`.

These typelevel representations of literals tend to get really helpful for things done at compiletime. E.g. they would
allow us to create matrices of a known size and check at compiletime that they are multipliable, so that impossible 
operations don't compile. 

Literal types also allow us to bringt literals from typelevel to value level which will be important for the next step.

So let's quickly recap what we've discovered so far: We learned that using typeclasses we can implement some 
functionality for any type. We can then use these instances and implicitly pass them to methods via `given` and `using`. 
And finally we know that literals can be presented in value and type space. Having that let's check how to automatically
derive typeclasses for case classes and sealed traits.

## A look in the Mirror

Before we learn how to derive typeclasses automatically we will do it by hand once. Taking the 
`case class User(name: String, age: Int)` let's define a `PrettyString` instance which also prints the labels
of the case classes fields:

```scala
given userPrettyString as PrettyString[User] {
  def prettyString(a: User): String = 
    s"User(name=${stringPrettyString.prettyString(a.name)}, age=${intPrettyString.prettyString(a.age)})"
}
```

So to create the `userPrettyString` instance we used 3 kinds of informations: 
- The label of the case class itself: "User"
- The labels of the fields: "name" and "age"
- The typeclass instances for the fields types: `stringPrettyString` and `intPrettyString`

Please also note that we built this instance by composing the typeclasses for our primitive types `Int` and `String. 
This is where the power of typeclasses comes into place, that you can compose them to generate new instances for complex
types.

To automate this process we will need a tool that provides us with that label and type information.
This is were the scala 3 trait `Mirror` comes into place:

```scala
sealed trait Mirror {
  type MirroredType // the type that was mirrored itself
  type MirroredLabel <: String // the label of the mirrored type
  type MirroredElemLabels <: Tuple // a tuple of the labels for each individual element
  type MirroredElemTypes <: Tuple // a tuple of types for each individual element
}
```

Ok, that's a lot of types. The first thing to notice is that `Mirror` is providing us with *typelevel* information only. 
Which makes sense, as the derivation of typeclasses happens at compiletime.

So what do the different types mean and how do they look for our `User` case class?

 - `MirroredType` - is just the type we are deriving from, so our case class `User`
 - `MirroredLabel` - typelevel representation of the label of the type we mirror, so the literal type `"User"`
 - `MirroredElemLabels` - A Tuple that contains the field names at typelevel (literal types again!), so of type `("name", "age")`
 - `MirroredElemTypes` - A tuple of all elements that `User` can be broken into: `(String, Int)` (for fields name and age)
 
Now we've got some the labels and types that our `User` consist of. But we still need some extra information: 
Is our type a case class or a sealed trait? Thankfully two traits extend `Mirror` which are:
 
 ```scala
// represents a case class
trait Product extends Mirror {
 def fromProduct(p: scala.Product): MirroredType
}

// represents a sealed trait
trait Sum extends Mirror { self =>
 def ordinal(x: MirroredType): Int
}
 ```
 
Ok, now we can distinguish also between case classes and sealed traits! 
Small side note: Case classes are also called product types and sealed traits sum types
(hence the naming of the traits). Also we've got some methods that we can call which will be important later.
: For `Product` a method to assemble the type from its elements and for `Sum` one that gives us the 
ordinal of a member of the respective sealed trait.

Let's now recheck how the Mirror type would look like for our `User` case class leaving out the `fromProduct` 
method which is not relevant right now.

```scala
import scala.compiletime.Mirror

// this is how the mirror for User would look like:
trait UserMirror extends Mirror.Product {
  type MirroredType = User
  type MirroredLabel = "User"
  type MirroredElemLabels = ("name", "age")
  type MirroredElemTypes = (String, Int)
}
```

Ok that doesn't look too complicated. We have all the information in place that we would need to automatically 
derive the `PrettyString` instance for `User`. But the information is still at the type level. We need to bring it to 
the value level to work with it at runtime. Let's start with the label of the type itself:

```scala
import scala.compiletime.constValue

// get the type label from a mirror
inline def labelFromMirror[A](using m: Mirror.Of[A]): String = constValue[m.MirroredLabel]

println(labelFromMirror[User]) // prints User
```

This was pretty easy. We just pass our `labelFromMirror` the mirror as an argument and it will use the `constValue` 
function that we discussed before to summon the value from the type level. The keyword `inline` is needed here because
the compiler needs to resolve `constValue` statically and inline it at compiletime which the `inline` keyword makes
possible. 

Summoning the element labels is a little bit more tricky. As it is a tuple we need to deconstruct it step by step
and create a simple `List[String]` out of that by using a recursive function:

```scala
import scala.compiletime.erasedValue

inline def summonElemLabels[A <: Tuple]: List[String] = inline erasedValue[A] match {
    case _: EmptyTuple => Nil // stop condition - the tuple is empty
    case _: (t *: ts) => 
      val headElementLabel = constValue[t].toString // bring the head label to value space
      val tailElementLabels = summonElemLabels[ts] // recursive call to get the labels from the tail
      headElementLabel :: tailElementLabels // concat head + tail
  }

inline def summonElemLabelsHelper[A](using m: Mirror.Of[A]) = // helper to summon the mirror
   summonElemLabels[m.MirroredElemLabels] // and call summonElemLabels with the elemlabels type

val userElementLabels = summonElemLabelsHelper[User] // List("name", "age")
```

Ah ... yes. Let me say it like this: To make the compiler happy we need to feed it a lot of `inline` keywords here.
In terms of automatic derivation the scala compiler is just a hungry alpaca in the category of keywords ...
ehm ... whatever, let's not get distracted. The most interesting thing here is the trick with the `erasedValue`
function. It allows us to create a **virtual** instance of the type `A` and match on it. Virtual means that this is done
at compiletime and there is no actual value at runtime. It is just resolved statically by the compiler. This allows us 
to deconstruct the tuple step by step during compilation.

Great, we've got all the labels in place. What's still missing are the typeclass instances for the individual elements.
Gathering them can be done like this:
 
```scala
import scala.compiletime.summonInline

inline def summonInstances[A <: Tuple]: List[PrettyString[Any]] = inline erasedValue[A] match {
  case _: EmptyTuple => Nil
  case _: (t *: ts) => 
    val headTypeClass = summonInline[PrettyString[t]] // summon was known as implicitly in scala 2
    val tailTypeClasses = summonInstances[ts] // recursive call to resolve also the tail
    headTypeClass.asInstanceOf[PrettyString[Any]] :: summonInstances[ts]
}
inline def summonInstancesHelper[A](using m: Mirror.Of[A]) = // helper again
   summonInstances[m.MirroredElemTypes]

val userInstances: List[PrettyString[Any]] = 
  summonInstancesHelper[User] // List(stringPrettyString, intPrettyString)
```

This is very similar to the `summonElemLabels` method. It recurses through the element types tuple and summons the 
respective typeclass instances for the individual elements. The method `summon` was introduced in dotty to clean up with
the different meanings of implicits in scala 2. In scala 3 typeclass instances no longer use `implicit` as keyword but 
`given` instead. And the method to fetch the instance is called `summon` and does the exact same things is `implicitly`
did in scala 2. Right now we got all the information in place that we need for the generic derivation. 
With that we are able to implement a method to do the derivation:

```scala
inline def derivePrettyString[A](using m: Mirror.ProductOf[A]) =
     new PrettyString[A] {
       def prettyString(a: A): String = { 
         val label = labelFromMirror[m.MirroredType]
         val elemLabels = summonElemLabels[m.MirroredElemLabels]
         val elemInstances = summonInstances[m.MirroredElemTypes]
         val elems = a.asInstanceOf[Product].productIterator // every case class extends scala.Product, we can safely cast here
         val elemStrings = elems.zip(elemLabels).zip(elemInstances).map{
           case ((elem, label), instance) => s"$label=${instance.prettyString(elem)}"
         }     
         s"$label(${elemStrings.mkString(", ")})"
       }
     }

val userPrettyString = derivePrettyString[User]
```

We are just gathering the information and then iterating over the elements of the case class with the `productIterator`
method from `scala.Product` that returns an iterator of all elements of that case class. After zipping it with the 
labels and instances we can assemble our pretty string.

Before we can use the `derivePrettyString` method we also need to provide the instances for the element types String 
and Int with the new `given` syntax. `given` replaces the implicits from scala 2 that were used for typeclass instances
before:

```scala
given intPrettyString as PrettyString[Int] {
  def prettyString(a: Int): String  = a.toString
}

given stringPrettyString as PrettyString[String] {
  def prettyString(a: String): String = s""""$a""""
}

val userPrettyString = derivePrettyString[User]

println(userPrettyString.prettyString(User("Bob", 25))) // prints User(name="Bob", age=25)
```

Awesome! In the snippet above we let the compiler assemble the typeclass for our `User` type. This would also work for 
any other case class as long as all individual elements have a respective given instance defined. 

It will also work for nested case classes as soon as we make the derivation itself a `given`. But let's first take a
look at how to handle sealed traits. We can use the same methods as before to extract information but we would need to 
use them in a different way. Here is a concret example that we can work with:

```scala
sealed trait Visitor
case class User(name: String, age: Int) extends Visitor // User like before, but extending Visitor
case object AnonymousVisitor extends Visitor // A user that is not registered visiting our website

// Btw in scala 3 you can write sealed traits also with the new enum syntax which is super nice:
enum Visitor {
  case User(name: String, age: Int)
  case AnonymousVisitor
}
// dont forget the import to have User and AnonymousVisitor in scope:
import Visitor._
```

Now we can implement the derivation for the sealed trait:

```scala
inline def derivePrettyStringSealedTrait[A](using m: Mirror.SumOf[A]) =  
  new PrettyString[A] {
    def prettyString(a: A): String = { 
      // val label = labelFromMirror[m.MirroredType] - not needed for our PrettyString
      // val elemLabels = summonElemLabels[m.MirroredElemLabels] - not needed for our PrettyString
      val elemInstances = summonInstances[m.MirroredElemTypes]
      val elemOrdinal = m.ordinal(a) // Checks the ordinal of the type, e.g. 0 for User or 1 for AnonymousVisitor
      val elemPrettyString = elemInstances(elemOrdinal).prettyString(a) 
      
      s"$elemPrettyString"
    }
  }
```

Using that on our sealed trait will not work yet because it also contains case classes. we need to combine the 
derivation of both, case classes and sealed traits first in a common `given` method:

```scala
// in scala 2 this would have been: implicit def derived[A](implicit m: Mirror.Of[A]): PrettyString[A]
inline given derived[A](using m: Mirror.Of[A]) as PrettyString[A] =
  inline m match {
    case s: Mirror.SumOf[A]     => derivePrettyStringSealedTrait(using s)
    case p: Mirror.ProductOf[A] => derivePrettyString(using p)
  }
```

Using the derived method we can now create `PrettyString` instances for our `Visitor` type:

```scala
val visitorPrettyString = derived[Visitor]
val visitors = List(
  User("Bob", 25),
  AnonymousVisitor
)

visitors.foreach(visitor =>
  println(visitorPrettyString.prettyString(visitor))
)
// prints: User("Bob", 25)
           AnonymousVisitor()  
```

Ok fine, but wait! Where are these brackets behind "AnonymousVisitor" coming from? This is because `AnonymousVisitor`
is a case object which is from the compiler treated like a case class with zero elements. So in the derivation process
for `AnonymousVisitor` it will create an instance for a product type, that as we defined it above prints also the 
brackets. Let's quickly fix that:

```scala
inline def derivePrettyString[A](using m: Mirror.ProductOf[A]) =
     new PrettyString[A] {
       def prettyString(a: A): String = { 
         val label = labelFromMirror[m.MirroredType]
         val elemLabels = summonElemLabels[m.MirroredElemLabels]
         val elemInstances = summonInstances[m.MirroredElemTypes]
         val elems = a.asInstanceOf[Product].productIterator
         val elemStrings = elems.zip(elemLabels).zip(elemInstances).map{
           case ((elem, label), instance) => s"$label=${instance.prettyString(elem)}"
         }     
         // TODO: highlight
         if (elemLabels.isEmpty) { // check if this is a case object (or parameterless case class)
           label
         } else {
           s"$label(${elemStrings.mkString(", ")})"
         }
       }
     }
```

Now our derived instance will print "AnonymousVisitor" without the brackets! If you wan't to have the best experience
when using automated derivation you should put the `derived` method in the companion object of your typeclass. After 
doing that you can easily have an instance created automatically by writing:

```scala
enum Visitor derives PrettyString {
  case User(name: String, age: Int)
  case AnonymousVisitor
}
```

TODO: Schlusssatz

TODO: shoutout to miles for shapeless, jon for magnolia, library maintainers for scala3 migration, hope overcome split in scala like usa 

TODO: reference dotty docs
TODO: function => method
TODO: rename summonElemLabels => getElemLabels
TODO: rename summonInstances => getInstances
TODO: scala groß
TODO: derivePrettyString => derivePrettyStringCaseClass
TODO: explain scala product
TODO: remove Product subtyping constraint text
TODO: Hero image - queen with reflection crazy alpaca

 
******************************************************************************************************
******************************************************************************************************
******************************************************************************************************
******************************************************************************************************
******************************************************************************************************

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
