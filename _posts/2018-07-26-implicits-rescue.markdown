---
title: "Scala Implicits Rescued Me!"
date: 2018-07-26
categories: [programming,scala,implicits]
tags: [programming,scala,implicits,reflection]
---
Recently I found myself faced with an interesting problem:

* I have a [Java][java] function (from a 3rd party) that takes a [`Varargs`][varargs]
* I also have a set of [Scala][scala] [case classes][caseclass] (also 3rd party)
* Some of the case class values are [`Option`][option] values.
* I need to pass the case class data to the function, but skip `None` values.

For the sake of the rest of this blog post, let's assume this is one of the case classes:

```scala
case class Person(
  first: String,
  last: String,
  email: Option[String],
  age: Option[Int],
  hourlyWage: Option[Double]
)
```

One option would be to simply brute-force it: create a function per case class that "does the right thing" for it. In theory, each function would look something like this:

```scala
def callWithPerson(p: Person) = {
  performJavaOp(
    "firstName", p.firstName,
    "lastName", p.lastName,
    "email", p.email,
    "age", p.age,
    "hourlyWage", p.hourlyWage
  )
}
```

Of course, this fails to work for multiple reasons:

* The `Option` values aren't handled by the Java function (crash).
* The `None` values should be ignored, but aren't.

Additionally, some of the values are primitives and are not of type `Object` and need to be coerced:

* `Int` to `java.lang.Integer`
* `Double` to `java.lang.Double`

Scala does this already, but can't implicitly convert from `Option[primitive]` to `Option[Object]`.

Worst of all, the thought of coding - and maintaining - a function per case class is untenable. If another programmer were to add or remove a field from the case class, then they would have to _know_ to come here and update this function: the compiler wouldn't inform them that a field in the class wasn't being passed to the Java function. That's unacceptable.

In my perfect, ideal world, it would be possible to just pass the case class to the Java function like so:

```scala
performJavaOp(person: _*)
```

So, is that possible? **YES**.

Every case class is automatically derived from [`Product`][product], which means we can iterate over all their values and - with a little [reflection][reflection] - also get all the field names. All that's needed to do is code up an implicit type conversion from any case class to an sequence of interspersed key/values:

```scala
import scala.language.implicitConversions

implicit def prod2seq[P <: Product](p: P): Seq[Object] = {
  val fields = p.getClass.getDeclaredFields.map(_.getName)
  val values = p.productIterator.to

  /* Zip and intersperse fields and values, removing None values.
   *
   * Note the explicit type signature of `v` as it will force implicit
   * primitive conversion to Object (e.g. Int to java.lang.Integer)!
   */
  fields.zip(values).collect {
    case (k, Some(v: Object))        => Seq(k, v)
    case (k, v: Object) if v != None => Seq(k, v)
  }.flatten.toSeq
}
```

And now, we can just call the Java function with any case class:

```scala
performJavaOp(person: _*)
```

Want to see it in action? [Try it on Scastie!][scastie]

# fin.

[scala]: https://scala-lang.org/
[implicits]: https://docs.scala-lang.org/overviews/core/implicit-classes.html
[java]: https://docs.oracle.com/javase/8/
[varargs]: https://docs.oracle.com/javase/8/docs/technotes/guides/language/varargs.html
[caseclass]: https://docs.scala-lang.org/tour/case-classes.html
[option]: https://www.scala-lang.org/api/2.12.6/scala/Option.html
[product]: https://www.scala-lang.org/api/2.12.6/scala/Product.html
[reflection]: https://docs.scala-lang.org/overviews/reflection/overview.html
[scastie]: https://scastie.scala-lang.org/5JPXsK4DR0CkNByMjoHesw