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

Scala 3 (previously called [dotty](https://dotty.epfl.ch/)) is approaching its [release](https://dotty.epfl.ch/blog/2020/09/21/naming-schema-change.html)!
This is a good opportunity to have a deeper look at some new features it offers. For me one of the most exiting
features are the metaprogramming abilities. Did you ever wonder how json libraries derive 
codecs for you from case classes and sealed traits? Or how tapir generates a whole OpenAPI documentation from your 
endpoint definitions? In this blog post we will discover how to do things like that in Scala 3 and discover some new 
language features like given, singleton types, reworked tuples, the exiting new enum definitions and finally the Mirror 
trait that allows us to derive typeclass instances automatically for our data types. If you haven't heard of these things
yet, don't be afraid. We will dive into them step by step. Even if the concepts of this post are between intermediate
to advanced level, i will try to explain it so that it is also understandable for people that already got some
experience with Scala but haven't explored these concepts yet. If there is something you do not understand please don't
hesitate to ask in the comment section below or DM me on [twitter](https://twitter.com/maphiFP).    

But before we start let me tell you the tale of princess Dotty. For the impatient reader: You can skip the tale by
clicking [here](#concepts).

## Princess Dotty, Sir Jon and the Queen of Reflection

Once upon a time in the land of Scala there lived a brave knight. His name was Sir Jon, but most people just called him [The Pretty](https://twitter.com/propensive). 
He was the defender of a poor folk which called itself "The Types". Many years ago they were abducted from the kingdom of
[Compiletime](https://dotty.epfl.ch/docs/reference/metaprogramming/inline.html#the-scalacompiletime-package)
by the evil Queen of Reflection. The queen ruled with terrible harshness. Everyone who tried to flee 
from her frightening reign would be thrown into the river of Runtime, that flowed into the great log pool, to never be
seen again. 
 
Sir Jon's plan was to build an army in secret, but he knew that a direct fight with the queen would not be possible. The
queen was guarded by an army of dreaded demons which were called "The Exceptions" by the people, who did not dare to
pronounce their names. That's why he worked out a smart plan: On the foundations of the [isle M'Sabin](https://twitter.com/milessabin), 
that was hidden from the evil queen's henchmen behind the mystic [Shapeless](https://github.com/milessabin/shapeless) Clouds,
he built the library [Magnolia](https://github.com/propensive/magnolia). From there he twittered his wise words all 
around the community of types, that came to learn at his [zone](https://scala.zone/). He would teach each Type how to 
`summon` their birth-`given` instance to defend themselves from the queen's army.

But the queen became aware of Jons plan to start a revolution and she casted a curse on the land that would 
terribly slow down any Type trying to summon its instance, which caused lots of compiletime. When Jon became aware of 
the curse he desperately began to search for a cure. He asked his old friend the wizard M'acro for help, that was a 
skilled witcher and wise man, but the reign of the queen had exhausted his power, and he was approaching his
[last days in the 2nd age]((http://dotty.epfl.ch/docs/reference/dropped-features/macros.html)) of Scala. 
Jon didn't knew what to do and was close to giving up when a small primitive Type named Boo Lean approached 
him to share the rumours of the prophecy of [Odersky](https://twitter.com/odersky). 

The prophecy was about a long forgotten bloodline and its descendant, young princess [Dotty](https://dotty.epfl.ch/).
It foretold that at the beginning of the 3rd age princess Dotty would appear to free all Types from the queen. 
Jon traveled the whole country, even behind the valleys of [typelevel](https://typelevel.org/), to 
eventually find princess Dotty which lived in the shadows of mountain [E'pfl](https://scala.epfl.ch/) in a small 
village with the strange name Rele Asecan Didate. Princess Dotty, who was not aware of her role in the prophecy, 
was surprised that the well-known knight asked her for help as she was still a young girl and hadn't been [released](https://dotty.epfl.ch/blog/2020/09/21/naming-schema-change.html)
outside of her village yet, but after Jon told her the story she began to smile. 

Her mother Java had given her a magic mirror before she passed away. Java told young Dotty that those who look 
into the mirror will see their soul breaking into its elementary parts so they have to face their real self, as good or
as evil as it is. 

The following day Jon and Dotty came up with a plan to speak up at the next audience of the queen, pretending to
surrender. At the audience they gave the queen Dotty's mirror as a gift. The queen then in her arrogance
and with a toxic smile in her face began to unwrap the gift. But at the first blink the mirror immediately released its
[fury](https://github.com/propensive/fury). The queen was shaken by her own reflection and screamed in agony as she 
realized that Dotty and Jon had tricked her. The magic of the mirror quickly made her fall asleep. Jon and Dotty 
imprisoned the queen in a [trifold IO-Monad](https://zio.dev/) guarded by the mystic
[Cats](https://typelevel.org/cats-effect) of [Monix](https://monix.io/) so that she no longer could cause any 
side effects. 

When they heard of Dotty's and Jon's victory over the queen the Types started dancing and went on a march back to the
kingdom of Compiletime. But as they approached the border, they realized that the land had dried over time. 
They started searching for the [stream](https://fs2.io/) that had once fed them and the 
[Alp](https://doc.akka.io/docs/alpakka/current/index.html) [akkas](https://akka.io/) nearby. 

After three days of search, thirsty and exhausted, Dotty all of a sudden noticed that the queen, still trapped inside 
the IO-Monad, slightly started to glow. The magic sleep seemed to heal the broken parts of the queen's soul. 
Little Boo Lean then remembered that in the prophecy of Odersky the magic mirror was told to be able to invert 
the queens soul. She was foreseen to become the Queen of 
[Mirrors](http://dotty.epfl.ch/docs/reference/contextual/derivation.html), a righteous and kindly ruler!
The queen, woken up by little Ints scream, slowly began to speak in a deep voice: "You freed me
from my own curse, princess Dotty. I know that the Types want to live in a flourishing land and so may it be.
You need to release the river of Runtime to flow through the kingdom to enter the era of generic programming". - 
"Yeah, sure!" said Sir Jon and laughed. "Why didn't we think of this?". 

The Types then went to the old dam that held
back the river of runtime and it released its refreshing energy into the kingdom of Compiletime. After that Dotty freed
the queen and they celebrated a big party with Jon and the Types where they ate many
[burritos](https://emorehouse.wescreates.wesleyan.edu/silliness/burrito_monads.pdf). Dotty had finally 
[unioned](http://dotty.epfl.ch/docs/reference/new-types/union-types.html) all Types under the Queen of Mirrors
like the prophecy of Odersky foresaw. And they lived in 
[equality](http://dotty.epfl.ch/docs/reference/contextual/multiversal-equality.html) happily ever after,
without any exceptions.

Even if it is only a tale that kids have been listening to for centuries there lies some truth within that you may run 
across while reading this post. But before we get our hands dirty let's dive into some new and old concepts first.

## Concepts

### Typeclasses

The first important concept to understand is that of typeclasses. You might already know about it and if not don't worry. 
I will show you how to use it and explain the new Scala 3 syntax around typeclasses. Instead of talking about the theory
behind typeclasses let's look at an example:  

```scala
trait PrettyString[A] {
  def prettyString(a: A): String // will print a type A in a pretty, human readable way
}
```

Pretty simple, right? Just an interface with a type parameter `A` and a method that takes a parameter and returns
`String`. So this typeclass provides us with the ability to convert any type `A` to a string. Why is that useful? -  
Imagine having a type that cannot be extended e.g. `Int` or `String` but you want to add some extra functionality in a 
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

You might have heard that in Scala 3 implicits have been reworked to make their different meanings and use cases
clearer. One of these use cases is typeclasses. Instead of using implicit here two new keywords were added:
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

The given syntax looks slightly strange at first sight, but you will get *used* to it pretty quickly. In Scala 2 `given`
and `using` would both have been `implicit` but in Scala 3  it was reworked to clarify intent:
`given` the instance `intPrettyString` we can call `prettyPrintln` `using` the `PrettyString` instance for the type
`Int`. The compiler will then take care of passing the `intPrettyString` parameter to `prettyPrintln`.

We need one last thing before we can start. Let's check out literal types.

## Singleton Types and The Literal Next Door

Since Scala 2.13 literals do not only exist in the value space but also on type-level. Each literal has it's
corresponding counterpart at type-level which is written exactly in the same way. So instead of writing 
`val myInt: Int = 1` you can be more precise by writing `val myInt: 1 = 1`. Of course if you would try 
`val myInt: 2 = 1` that would not compile. It also works with strings: 
`val helloWorld: "Hello world!" = "Hello world!"`.

These type-level representations of literals tend to become really helpful for things that are done at compiletime. E.g.
they would allow us to create matrices of a known size and check at compiletime that they are multipliable. Illegal
operations would simply not compile at all. 

Literal types also allow us to bringt literals from type-level to value-level which will be important in the next step.

So let's quickly recap what we've discovered so far: We've learned that using typeclasses we can implement additional 
functionality for each type. We can then use the typeclass instances and implicitly pass them to methods via `given` 
and `using`. And finally we know that literals can be presented in value and type space. Having that let's ultimately 
check how to automatically derive typeclasses for case classes and sealed traits.

## A look in the Mirror

Before we learn how to derive typeclasses automatically we will once do it by hand. Taking the 
`case class User(name: String, age: Int)` let's define a `PrettyString` instance which contrary to the case classes 
`toString` method also prints the labels of the case classes fields:

```scala
given userPrettyString as PrettyString[User] {
  def prettyString(a: User): String = 
    s"User(name=${stringPrettyString.prettyString(a.name)}, age=${intPrettyString.prettyString(a.age)})"
}
```

So to create the `userPrettyString` instance we used 3 kinds of information: 
- The label of the case class itself: "User"
- The labels of the fields: "name" and "age"
- The typeclass instances for the fields: `stringPrettyString` and `intPrettyString`

Please also note that we built this instance by composing the typeclasses for our primitive types `Int` and `String`. 
This is where typeclasses shine. You can derive typeclasse instances for complex types from simpler ones.

To automate this process we will need a tool that provides us with that label and type information.
This is where the Scala 3 trait `Mirror` comes into place:

```scala
sealed trait Mirror {
  type MirroredType // the type that was mirrored itself
  type MirroredLabel <: String // the label of the mirrored type
  type MirroredElemLabels <: Tuple // a tuple of the labels for each individual element
  type MirroredElemTypes <: Tuple // a tuple of types for each individual element
}
```

Ok, that's a lot of types. The first thing to not here is that the `Mirror` trait is providing *type-level* information
only. Which makes sense, as the derivation of typeclasses happens at compiletime.

So what do the different types mean and how do they look for our `User` case class?

 - `MirroredType` - is just the type we are deriving from, so our case class `User`
 - `MirroredLabel` - type-level representation of the label of the type we are mirroring, so the literal type `"User"`
 - `MirroredElemLabels` - A Tuple that contains the field names at type-level (literal types again!), so of type `("name", "age")`
 - `MirroredElemTypes` - A tuple of all elements that `User` can be broken into: `(String, Int)` (for fields name and age)
 
Now we've got the labels and types that our `User` consist of. But we still need some extra information: 
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
 
Ok, now we can also distinguish between case classes and sealed traits! 
Side note: Case classes are also called product types and sealed traits sum types
(hence the naming of the traits). Also we've got some methods that we can call which will be important later.
For `Product` a method to assemble the type from its elements and for `Sum` one that gives us the 
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

Ok, doesn't look too complicated! We have all the information in place that we would need to automatically 
derive the `PrettyString` instance for `User`. But the information is still at the type-level. We need to bring it to 
the value-level to work with it at runtime. Let's start with the label of the type itself:

```scala
import scala.compiletime.constValue

// get the type label from a mirror
inline def labelFromMirror[A](using m: Mirror.Of[A]): String = constValue[m.MirroredLabel]

println(labelFromMirror[User]) // prints User
```

This was pretty easy. We just pass our `labelFromMirror` the mirror as an argument and it will use the `constValue` 
method that we discussed before to summon the value from the type-level. The keyword `inline` is needed here because
the compiler needs to resolve `constValue` statically and inline it at compiletime which the keyword makes
possible. 

Summoning the element labels is a little bit more tricky. As it is a tuple we need to deconstruct it step by step
and create a simple `List[String]` out of that by using a recursive method:

```scala
import scala.compiletime.erasedValue

inline def getElemLabels[A <: Tuple]: List[String] = inline erasedValue[A] match {
  case _: EmptyTuple => Nil // stop condition - the tuple is empty
  case _: (head *: tail) =>  // yes, in scala 3 we can match on tuples head and tail to deconstruct them step by step
    val headElementLabel = constValue[head].toString // bring the head label to value space
    val tailElementLabels = getElemLabels[tail] // recursive call to get the labels from the tail
    headElementLabel :: tailElementLabels // concat head + tail
}

// helper method to get the mirror from compiler
inline def getElemLabelsHelper[A](using m: Mirror.Of[A]) = 
  getElemLabels[m.MirroredElemLabels] // and call getElemLabels with the elemlabels type

val userElementLabels = getElemLabelsHelper[User] // List("name", "age")
```

Ah ... yes. Let me say it like this: To make the compiler happy we need to feed it a lot of `inline` keywords here.
In terms of automatic derivation the Scala compiler is just a hungry alpaca in the category of keywords ...
ehm ... whatever. Let's not get distracted. 

The most interesting thing here is the trick with the `erasedValue` method. It allows us to create a **virtual** 
instance of the type `A` and match on it. Virtual means that this is done at compiletime and there is no actual value 
to be matched on at runtime. It is just resolved statically by the compiler looking only at the types. This allows us to
deconstruct the tuple step by step during compilation.

Great, we've got all the labels in place. What's still missing are the typeclass instances for the individual elements.
Gathering them can be done like this:
 
```scala
import scala.compiletime.summonInline

inline def getTypeclassInstances[A <: Tuple]: List[PrettyString[Any]] = inline erasedValue[A] match {
  case _: EmptyTuple => Nil
  case _: (head *: tail) => 
    val headTypeClass = summonInline[PrettyString[head]] // summon was known as implicitly in scala 2
    val tailTypeClasses = getTypeclassInstances[tail] // recursive call to resolve also the tail
    headTypeClass.asInstanceOf[PrettyString[Any]] :: getTypeclassInstances[tail]
}

// helper method like before
inline def summonInstancesHelper[A](using m: Mirror.Of[A]) = 
   getTypeclassInstances[m.MirroredElemTypes]

val userInstances: List[PrettyString[Any]] = 
  summonInstancesHelper[User] // List(stringPrettyString, intPrettyString)
```

This is very similar to the `getElemLabels` method. It recurses through the elements types tuple and summons the 
respective typeclass instances for the individual elements. The method `summon` was introduced in dotty to clean up with
the different meanings of implicits in Scala 2. In Scala 3 typeclass instances no longer use `implicit` as keyword but 
`given` instead. And the method to fetch the instance is called `summon` and does the exact same things as `implicitly`
did in Scala 2. And of couse it needs to be inlined again, so we need to use not `summon` but `summonInline`. 
Right now we got all the information in place that we need for the generic derivation. Let's implement it:

```scala
inline def derivePrettyStringCaseClass[A](using m: Mirror.ProductOf[A]) =
     new PrettyString[A] {
       def prettyString(a: A): String = { 
         val label = labelFromMirror[m.MirroredType]
         val elemLabels = getElemLabels[m.MirroredElemLabels]
         val elemInstances = getTypeclassInstances[m.MirroredElemTypes]
         val elems = a.asInstanceOf[Product].productIterator // every case class implements scala.Product, we can safely cast here
         val elemStrings = elems.zip(elemLabels).zip(elemInstances).map{
           case ((elem, label), instance) => s"$label=${instance.prettyString(elem)}"
         }     
         s"$label(${elemStrings.mkString(", ")})"
       }
     }

val userPrettyString = derivePrettyStringCaseClass[User]
```

We are just gathering the information and then iterating over the elements of the case class with the `productIterator`
method from `scala.Product` that returns an iterator of all elements of that case class. After zipping it with the 
labels and instances we can assemble our super pretty string typeclass:

```scala
val userPrettyString = derivePrettyStringCaseClass[User]

println(userPrettyString.prettyString(User("Bob", 25))) // prints User(name="Bob", age=25)
```

Awesome! In the snippet above we simply let the compiler assemble the typeclass for our `User` type. This would also
work for any other case class as long as all individual elements have the respective `given` instances defined (like
the ones we defined [above](#implicits-30---give-it-use-it-quick-enjoy-it) for `String` and `Int` to create the typeclass instance for `User`). 

It will also work for nested case classes as soon as we make the derivation itself a `given`. But let's first take a
look at how to handle sealed traits. We can use the same methods as before to extract information but we would need to 
use them in a different way. Look at this example:

```scala
sealed trait Visitor
case class User(name: String, age: Int) extends Visitor // User like before, but extending Visitor
case object AnonymousVisitor extends Visitor // A user that is not registered visiting our website

// Btw. in Scala 3 you can write sealed traits also with the new enum syntax which is super lean:
enum Visitor {
  case User(name: String, age: Int)
  case AnonymousVisitor
}
// For the enum: Don't forget the import to have User and AnonymousVisitor in scope:
import Visitor._
```

Now we can implement the derivation for the sealed trait:

```scala
inline def derivePrettyStringSealedTrait[A](using m: Mirror.SumOf[A]) =  
  new PrettyString[A] {
    def prettyString(a: A): String = { 
      // val label = labelFromMirror[m.MirroredType] - not needed
      // val elemLabels = getElemLabels[m.MirroredElemLabels] - not needed
      val elemInstances = getTypeclassInstances[m.MirroredElemTypes] // same as for the case class
      val elemOrdinal = m.ordinal(a) // Checks the ordinal of the type, e.g. 0 for User or 1 for AnonymousVisitor
  
      // just return the result of prettyString from the right element instance
      elemInstances(elemOrdinal).prettyString(a)
    }
  }
```

Using that on our sealed trait will not work yet because it also contains case classes. we need to combine the 
derivation of both, case classes and sealed traits first in a separate `given` method:

```scala
// in Scala 2 this would have been: implicit def derived[A](implicit m: Mirror.Of[A]): PrettyString[A]
inline given derived[A](using m: Mirror.Of[A]) as PrettyString[A] =
  inline m match {
    case s: Mirror.SumOf[A]     => derivePrettyStringSealedTrait(using s)
    case p: Mirror.ProductOf[A] => derivePrettyStringCaseClass(using p)
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
// prints: 
// User("Bob", 25)
// AnonymousVisitor()  
```

Ok fine, but wait! Where are these parantheses behind "AnonymousVisitor" coming from? This is because `AnonymousVisitor`
is a case object which the compiler treates as a case class with zero fields. So in the derivation process
for `AnonymousVisitor` it will create an instance for a case class, that as we defined it above prints also the 
parantheses. Let's quickly fix that:

```scala
inline def derivePrettyStringCaseClass[A](using m: Mirror.ProductOf[A]) =
  new PrettyString[A] {
    def prettyString(a: A): String = { 
      val label = labelFromMirror[m.MirroredType]
      val elemLabels = getElemLabels[m.MirroredElemLabels]
      val elemInstances = getTypeclassInstances[m.MirroredElemTypes]
      val elems = a.asInstanceOf[Product].productIterator
      val elemStrings = elems.zip(elemLabels).zip(elemInstances).map{
        case ((elem, label), instance) => s"$label=${instance.prettyString(elem)}"
      }
  
      if (elemLabels.isEmpty) { // check if this is a case object (or parameterless case class)
        label
      } else {
        s"$label(${elemStrings.mkString(", ")})"
      }
    }
  }
```

Now our derived instance will print "AnonymousVisitor" without the parantheses! If you want to have the best experience
when using automated derivation you should put the `derived` method in the companion object of your typeclass. After 
doing that you can easily have an instance created automatically by writing:

```scala
enum Visitor derives PrettyString {
  case User(name: String, age: Int)
  case AnonymousVisitor
}
```

What we still didn't use is the `fromProduct` method from the `Product` trait. Using that you can create the case class
which the `Product` trait mirrors. This must be used when creating instances for things like a JSON codec where you need
to assemble a case class from a collection of fields. As this blog post is already super long i will not show it here
but instead show you some code that i wrote to make the derivation much easier by abstracting away all the methods we 
discussed above. It was heavily inspired by the [magnolia] library and allows you to add automatic derivation to your
typeclass by implementing two simple methods: `deriveCaseClass` and `deriveSealed`

```scala
import scala.deriving.Mirror
import scala.compiletime.{constValue, erasedValue, summonInline}

/**************************** trait to be used for simple derivation below **************************/

trait EasyDerive[TC[_]] {
  final def apply[A](using tc: TC[A]): TC[A] = tc

  case class CaseClassElement[A, B](label: String, typeclass: TC[B], getValue: A => B, idx: Int)
  case class CaseClassType[A](label: String, elements: List[CaseClassElement[A, _]], fromElements: List[Any] => A)

  case class SealedElement[A, B](label: String, typeclass: TC[B], idx: Int, cast: A => B)
  case class SealedType[A](label: String, elements: List[SealedElement[A, _]], getElement: A => SealedElement[A, _])

  inline def getInstances[A <: Tuple]: List[TC[Any]] = inline erasedValue[A] match {
    case _: EmptyTuple => Nil
    case _: (t *: ts) => summonInline[TC[t]].asInstanceOf[TC[Any]] :: getInstances[ts]
  }

  inline def getElemLabels[A <: Tuple]: List[String] = inline erasedValue[A] match {
    case _: EmptyTuple => Nil
    case _: (t *: ts) => constValue[t].toString :: getElemLabels[ts]
  }

  def deriveCaseClass[A](caseClassType: CaseClassType[A]): TC[A]

  def deriveSealed[A](sealedType: SealedType[A]): TC[A]

  inline given derived[A](using m: Mirror.Of[A]) as TC[A] = {
    val label = constValue[m.MirroredLabel]
    val elemInstances = getInstances[m.MirroredElemTypes]
    val elemLabels = getElemLabels[m.MirroredElemLabels]

    inline m match {
      case s: Mirror.SumOf[A] =>
        val elements = elemInstances.zip(elemLabels).zipWithIndex.map{ case ((inst, lbl), idx) =>
          SealedElement[A, Any](lbl, inst.asInstanceOf[TC[Any]], idx, identity)
        }
        val getElement = (a: A) => elements(s.ordinal(a))
        deriveSealed(SealedType[A](label, elements, getElement))

      case p: Mirror.ProductOf[A] =>
        val caseClassElements =
          elemInstances
            .zip(elemLabels)
            .zipWithIndex.map{ case ((inst, lbl), idx) =>
              CaseClassElement[A, Any](lbl, inst.asInstanceOf[TC[Any]],
                (x: Any) => x.asInstanceOf[Product].productElement(idx), idx)
            }
        val fromElements: List[Any] => A = { elements =>
          val product: Product = new Product {
            override def productArity: Int = caseClassElements.size

            override def productElement(n: Int): Any = elements(n)

            override def canEqual(that: Any): Boolean = false
          }
          p.fromProduct(product)
        }
        deriveCaseClass(CaseClassType[A](label, caseClassElements, fromElements))
    }
  }
}

/**************************** define typeclass and derivation **************************/

trait PrettyString[A] {
  def prettyString(a: A): String
}

object PrettyString extends EasyDerive[PrettyString] {
  override def deriveCaseClass[A](productType: CaseClassType[A]): PrettyString[A] = new PrettyString[A] {
    override def prettyString(a: A): String = {
      if (productType.elements.isEmpty) productType.label
      else productType.elements.map(p => s"${p.label}=${p.typeclass.prettyString(p.getValue(a))}").mkString(
        s"${productType.label}(",
        ", ",
        ")"
      )
    }
  }

  override def deriveSealed[A](sumType: SealedType[A]): PrettyString[A] = new PrettyString[A] {
    override def prettyString(a: A): String = {
      val elem = sumType.getElement(a)
      elem.typeclass.prettyString(elem.cast(a))
    }
  }

  // some instances for primitive types

  given stringPrettyString as PrettyString[String] {
    def prettyString(x: String): String = s""""x""""
  }

  given intPrettyString as PrettyString[Int] {
    def prettyString(x: Int): String = x.toString
  }

  given longPrettyString as PrettyString[Long] {
    def prettyString(x: Long): String = x.toString
  }

  given doublePrettyString as PrettyString[Double] {
    def prettyString(x: Double): String = x.toString
  }

  given booleanPrettyString as PrettyString[Boolean] {
    def prettyString(x: Boolean): String = x.toString
  }
}

/************************************** Use it *****************************************/

enum Visitor derives PrettyString {
  case User(name: String, age: Int)
  case AnonymousVisitor
}

import Visitor._

val someVisitors = List(
  User("bob", 25),
  AnonymousVisitor
)

someVisitors.foreach(prettyPrintln)
```

So, i hope you you enjoyed the post. Feel free to ask questions in the comment section or reach out to me 
on [Twitter](https://twitter.com/maphiFP). For additional information please consult the [dotty documentation](https://dotty.epfl.ch/docs/index.html)
and [blog](https://dotty.epfl.ch/blog/index.html). 

I'd want to thank my friends and collegues, that helped me reviewing this post. Also shout-out
to the Scala community for creating such a great ecosystem! 

Stay healthy you all!

<!--
- check code compiles
- fix blog search
- github pages https
-->
