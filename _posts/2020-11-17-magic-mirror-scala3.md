---
title: "Mirror, Mirror on the Wall, Who's the Genericioust of Them All? - Generic Programming with Scala 3" 
date: 2020-11-17T21:00:00+01:00
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
This is a good opportunity to have a deeper look at some new features it offers. For me one of the most exciting
features are the metaprogramming abilities. 

Did you ever wonder how JSON libraries like [circe](https://circe.github.io/circe/) derive 
codecs for you from case classes and sealed traits? Or how [tapir](https://tapir.softwaremill.com/en/latest/) generates a whole OpenAPI documentation from your 
endpoint definitions? 

In this blog post we will discover how to do things like that in Scala 3 and discover some new 
language features like the given keyword, literal types, reworked tuples, the exciting new enum definitions and finally the Mirror 
trait that allows us to derive typeclass instances automatically for our data types. 

If you haven't heard of these things
yet, don't worry. We will dive into them step by step. Even if the concepts of this post are between intermediate
to advanced level, I will try to explain it so that it is also understandable for people that already got some
experience with Scala but haven't explored these concepts yet. If there is something you do not understand please don't
hesitate to DM me on [Twitter](https://twitter.com/maphiFP). 

The language version I used in this post is `0.27.0-RC1`. 
You can find a guide how to install dotty here: [dotty page](https://dotty.epfl.ch/)

But before we get our hands dirty let's dive into some new and old concepts first.

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
but calm down young Padawan. First our `PrettyString` instance for `String` wraps the result in quotes which `toString`
does not (yeah, awesome - I know) and second you can do super cool stuff with it which we'll see later.

But in the code above converting something to a string included a lot of boilerplate. Let's get rid of that.

### Implicits 3.0 - Give it, Use it, Quick Enjoy it

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

## Singleton Types and the Literal Next Door

Since Scala 2.13 literals do not only exist in the value space but also on type-level. Each literal has it's
corresponding counterpart at type-level which is written exactly in the same way. So instead of writing 
`val myInt: Int = 1` you can be more precise by writing `val myInt: 1 = 1`. Of course if you would try 
`val myInt: 2 = 1` that would not compile. It also works with strings: 
`val helloWorld: "Hello world!" = "Hello world!"`.

These type-level representations of literals tend to become really helpful for things that are done at compile time. E.g.
they would allow us to create matrices of a known size and check at compile time that they are multipliable. Illegal
operations would simply not compile at all. 

Literal types also allow us to bring literals from type-level to value-level which will be important in the next step.

So let's quickly recap what we've discovered so far: We've learned that using typeclasses we can implement additional 
functionality for each type. We can then use the typeclass instances and implicitly pass them to methods via `given` 
and `using`. And finally we know that literals can be presented in value and type space. Having that let's ultimately 
check how to automatically derive typeclasses for case classes and sealed traits.

## A Look in the Mirror

Before we learn how to derive typeclasses automatically we will once do it by hand. Taking the 
`case class User(name: String, age: Int)` let's define a `PrettyString` instance which contrary to the case classes 
`toString` method also prints the labels of the case classes fields:

```scala
given userPrettyString as PrettyString[User] {
  def prettyString(a: User): String = 
    s"User(name=${stringPrettyString.prettyString(a.name)}, age=${intPrettyString.prettyString(a.age)})"
}

prettyPrintln(User("bob", 25)) // prints User(name="bob", age=25)
```

So to create the `userPrettyString` instance we used 3 kinds of information: 
- The label of the case class itself: "User"
- The labels of the fields: "name" and "age"
- The typeclass instances for the fields: `stringPrettyString` and `intPrettyString`

Please also note that we built this instance by composing the typeclasses for our primitive types `Int` and `String`. 
This is where typeclasses shine. You can derive typeclasse instances for complex types from simpler ones.

To automate this process we will need a tool that provides us with that label and type information.
This is where the Scala 3 trait [`Mirror`](https://dotty.epfl.ch/docs/reference/contextual/derivation.html) comes into place:

```scala
sealed trait Mirror {
  type MirroredType // the type that was mirrored itself
  type MirroredLabel <: String // the label of the mirrored type
  type MirroredElemLabels <: Tuple // a tuple of the labels for each individual element
  type MirroredElemTypes <: Tuple // a tuple of types for each individual element
}
```

Ok, that's a lot of types. The first thing to note here is that the `Mirror` trait is providing *type-level* information
only. Which makes sense, as the derivation of typeclasses happens at compile time.

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
 
Ok, now we can distinguish between case classes and sealed traits! 
Side note: Case classes are also called product types and sealed traits sum types
(hence the naming of the traits). In addition, we've got some methods that we can call which will be important later.
For `Product` a method to assemble the type from its elements and for `Sum` one that gives us the 
ordinal of a member of the respective sealed trait.

Let's now recheck how the Mirror type would look like for our `User` case class leaving out the `fromProduct` 
method which is not relevant right now.

```scala
import scala.deriving.Mirror

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
the compiler needs to resolve `constValue` statically and inline it at compile time which the keyword makes
possible. 

Summoning the element labels is a bit more tricky. As it is a tuple we need to deconstruct it step by step
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

getElemLabelsHelper[User] // List("name", "age")
```

Ah ... yes. Let me say it like this: To make the compiler happy we need to feed it a lot of `inline` keywords here.
In terms of automatic derivation the Scala compiler is just a hungry alpaca in the category of keywords ...
ehm ... whatever. Let's not get distracted. 

The most interesting thing here is the trick with the `erasedValue` method. It allows us to create a **virtual** 
instance of the type `A` and match on it. Virtual means that this is done at compile time and there is no actual value 
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

summonInstancesHelper[User] // List(stringPrettyString, intPrettyString)
```

This is very similar to the `getElemLabels` method. It recurses through the elements types tuple and summons the 
respective typeclass instances for the individual elements. The method `summon` was introduced in dotty to clean up with
the different meanings of implicits in Scala 2. In Scala 3 typeclass instances no longer use `implicit` as keyword but 
`given` instead. And the method to fetch the instance is called `summon` and does the exact same things as `implicitly`
did in Scala 2. And of course it needs to be inlined again, so we need to use not `summon` but `summonInline`.
 
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
look at how to handle sealed traits. We can use the same methods as before to extract information, but we would need to 
use them differently. Look at this example:

```scala
sealed trait Visitor
case class User(name: String, age: Int) extends Visitor // User like before, but extending Visitor
case object AnonymousVisitor extends Visitor // A user that is not registered visiting our website

// Btw. in Scala 3 you can write sealed traits also with the new enum syntax which is super lean:
enum Visitor {
  case User(name: String, age: Int)
  case AnonymousVisitor
}
// For the Scala 3 enum: Don't forget the import to have User and AnonymousVisitor in scope:
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

Using that on our sealed trait will not work yet because it also contains case classes. We need to combine the 
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
// User(name="Bob", age=25)
// AnonymousVisitor()  
```

Ok fine, but wait! Where are these parentheses behind "AnonymousVisitor" coming from? This is because `AnonymousVisitor`
is a case object which the compiler treats as a case class with zero fields. So in the derivation process
for `AnonymousVisitor` it will create an instance of a case class, that as we defined it above prints also the 
parentheses. Let's quickly fix that:

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

Now our derived instance will print "AnonymousVisitor" without the parentheses! If you want to have the best experience
when using automated derivation you should put the `derived` method in the companion object of your typeclass. After 
doing that you can easily have an instance created automatically by writing:

```scala
enum Visitor derives PrettyString {
  case User(name: String, age: Int)
  case AnonymousVisitor
}
```

What we still didn't use is the `fromProduct` method from the `Product` trait. Using that you can create the case class
which the `Product` trait mirrors from its individual elements. This must be used when creating instances for things like a JSON codec where you need
to assemble a case class from a collection of fields.

As this blog post is already pretty long I will not show it here
but instead show you some code that I wrote to make the derivation much easier by abstracting away all the methods we 
discussed above. It allows you to add automatic derivation to your typeclass by implementing two simple methods: 
`deriveCaseClass` and `deriveSealed`

```scala
import scala.deriving.Mirror
import scala.compiletime.{constValue, erasedValue, summonInline}

/**************************** trait to be used for simple derivation below **************************/

// this traits can just be copy/pasted or reside in a library
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
      else {
        val prettyElements = productType.elements.map(p => s"${p.label}=${p.typeclass.prettyString(p.getValue(a))}")
        prettyElements.mkString(s"${productType.label}(", ", ", ")")
      }
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

  // our helper method to print the PrettyString
  def prettyPrintln[A](a: A)(using prettyStringInstance: PrettyString[A]) = 
    println(prettyStringInstance.prettyString(a))
}

/************************************** Use it *****************************************/
import PrettyString.prettyPrintln

enum Visitor derives PrettyString { // magic happens here via 'derives'
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

So, I hope you enjoyed the post. Feel free to reach out to me 
on [Twitter](https://twitter.com/maphiFP). For additional information please consult the [dotty documentation](https://dotty.epfl.ch/docs/index.html)
and [blog](https://dotty.epfl.ch/blog/index.html). 

I'd want to thank my friends and collegues, that helped me to review this post. Also, shout-out
to the Scala community for creating such a great ecosystem! 

Stay healthy and follow me on [Twitter](https://twitter.com/maphiFP) to stay tuned for more posts.
