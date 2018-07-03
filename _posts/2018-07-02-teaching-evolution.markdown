---
title: "Teaching Evolution With Genetic Algorithms"
date: 2018-07-02
categories: [evolution,teaching,programming,scala]
tags: [evolution,teaching,programming,scala,genetic,algorithm]
---
My 8 year-old daughter this year began asking me some of the important questions of life: who was the first person, was the first person a boy or girl (note: our family is not religious, so she's never heard the story of Adam and Eve to my knowedge), were dogs or cats first, etc.

I work at [The Broad Institute][broad], so questions like these _should_ be right up my alley, right? I took a little bit of time and eventually came up with an idea...

I drew some circles an put some letters in them. The letters were meant to represent [nucleobases][bases], but for simplicity I let them be any letter of the alphabet. I then showed how when they joined to make a "child", half of the letters come from the mom and the other half from the dad. Next, we discussed how sometimes a letter can mutate (or insert/delete), causing a change. Over very long periods of time, the mutations build up and eventually you have a new animal:

![evolution](https://raw.githubusercontent.com/massung/codeninja/master/_posts/images/evolution.png)

Surprisingly, she grokked this pretty quick and started drawing her own trees that created horses and ferrets. 

I wasn't quite happy yet. While this explained (conceptually) [DNA][dna], it didn't really do anything to explain [_evolution_][evolution]: why would some mutations be preffered over another?

I next decided that I would try and code a very simple program that she could interact with and actually _see_ evolution taking place, and perhaps understand a bit more. This ended up better than I could have expected, and so I thought I would share the code here.

---

**Concept:** The program should let the user type anything they want, and see if the computer - through evolution - could eventually reproduce what was typed. The program should be simple to explain. The code - to a non-programmer - may not make sense, but the program as a whole and each individual piece (a function) should.

I decided to stick with the concept I started with: letters, and so the program was broken down thusly:

* A DNA strand is just a string of characters.
* Multiple strands are sorted by [fitness][fitness]: similarity to the target string.
* The next generation is created by [crossing over][crossover] and introducing random [mutations][mutation].

**Step 1:** Code helper functions and a couple implicit classes so we can just extend `String` objects and treat them like populations of DNA sequences. 

```scala
object Evolution {
  import scala.util.Random.{nextBoolean, nextInt, nextDouble}

  /** Generate a random string. */
  def randomString(n: Int) = (1 to n).map(_ => (nextInt(96)+32).toChar).mkString

  /** Generate a random, initial population of DNA sequences. */
  def initialPopulation(n: Int): IndexedSeq[String] = {
    for (_ <- 1 to n) yield randomString(nextInt(20) + 10)
  }

  /** An implicit class to add functionality to a DNA string. */
  implicit class DNA(s: String) {

    /** Calculate how close this string is to a target string. Calculate the
      * delta between each character and when one string is longer than the
      * other assume null (\u0000) for the character.
      *
      * A fitness value of 0.0 is a perfect match. As the strings diverge,
      * make the fitness exponentially larger.
      */
    def fitness(target: String): Double = {
      (0 until target.size.max(s.size)).foldLeft(0.0) { (f, i) =>
        val a = s.lift(i).map(_.toInt).getOrElse(0)
        val b = target.lift(i).map(_.toInt).getOrElse(0)

        // square the delta between two characters
        f + (a - b) * (a - b)
      }
    }

    /** Since a new DNA sequence is a combination of "mom" and "dad", the
      * first step is to crossover two DNA sequences: take a slice of one
      * and a slice of the other and combine them together.
      */
    def cross(c: String): String = {
      val i = nextInt(c.size.min(s.size))

      // take some from one string and some from the other
      if (nextBoolean) {
          c.take(i) + s.drop(i)
      } else {
          s.take(i) + c.drop(i)
      }
    }

    /** Mutations are a random change within a DNA sequence. One or more
      * bases within the original DNA strand are removed and a new sequence
      * replaces it. The new sequence can be shorter or longer.
      */
    def mutate(mutationRate: Double = 0.1): String = {
      if (nextDouble >= mutationRate) {
        return s
      }

      // what character(s) to replace (i), how many (r), and with how many (n)
      val i = nextInt(s.size)
      val r = nextInt(3)+1
      val n = nextInt(3)

      // patch the string with a new sequence
      s.patch(i, randomString(n), r)
    }
  }

  /** An implicit class for manipulating populations of DNA sequences. */
  implicit class Population(xs: IndexedSeq[String]) {

    /** Returns a population sorted by fitness to a particular string. */
    def fitToString(target: String): IndexedSeq[String] = {
      xs.map(it => (it fitness target, it)).sortBy(_._1).map(_._2)
    }

    /** Return the best parent randomly from a few choices. Assuming
      * the population is sorted (fitToString), then the smallest
      * index chosen is considered the "best".
      *
      * This is meant to simulate selective breeding. In the wild,
      * animals have multiple choices for mating, and fight for
      * rights to choose mate that will provide their children the
      * greatest chances of survival.
      */
    def parent(n: Int) = xs((1 to n).map(_ => nextInt(xs.size)).min)

    /** Create a child from two random parents. This is done by first
      * selecting the parents, crossing them, and then (optionally) 
      * introducing a random mutation.
      */
    def breed(selections: Int, mutationRate: Double) = {
      parent(selections).cross(parent(selections)).mutate(mutationRate)
    }

    /** Breed a new population from the current one. */
    def nextGen(selections: Int, mutationRate: Double) = {
      (1 to xs.size).map(_ => breed(selections, mutationRate)).filter(_.size > 0)
    }
  }
}
```

_I recommend using [Ammonite-REPL][ammonite] and copy/pasting the code into the REPL and then `import Evolution._`._

Next, we tested the code together and let her see how each part worked just like the graph that I drew (see above) on paper.

```scala
@ randomString(10)
String = "CRcp(4av~]"

@ "Test".fitness("Test")
Double = 0.0

@ "Test".fitness("foobar")
Double = 261.0

@ "abcde".cross("12345678")
String = "123de"

@ "abcde".mutate(1.0)
String = "a]cd"

@ initialPopulation(5)
IndexedSeq[String] = Vector(
  "$%iJzT<EUF/",
  "m:n]VpeKy1jY_|{QJn|l^M",
  "o|Qo\"P02) d/a%[^8VzMI\u007fO!sQ2zv",
  "Vt)vRi:FgT#<.1ep`ADl`",
  "\u007f<H9JAoJ*ETaL)*0\";@\\{C,y~;S"
)

@ initialPopulation(5).fitToString("Hello").nextGen(2, 0.1)
IndexedSeq[String] = Vector(
  "UN=q]T0W9I~rte$K'biSBmh",
  "2Cu;pre\u007fNz<,fm*+-&iSBmh",
  "tQ)|!o?ei`hjo+&!FUw",
  "tQ)|!o0W9I~rte$K'b9SsKK7Q\"h",
  "0X|jprKvqu"
)
```

The hardest parts to explain were `fitness` and generating the next generation. It took a bit, but she was able to see the similarities in the next generation created; multiple children must have had the same parents.

**Step 2:** Bring it together with a little bit of code that asks for an input string and then runs over many generations using the input as the fitness target.

```scala
import scala.io.StdIn

def run = {
  print("Enter something to evolve: ")

  // read the fitness target
  val target = StdIn.readLine

  // some parameters worth tweaking to see the results
  val generations = 10000
  val populationSize = 200
  val breedingSelections = 3
  val mutationRate = 0.1

  // create an initial population of strings
  var pop = initialPopulation(populationSize).fitToString(target)

  // run for N generations
  for (i <- 1 to generations) {
    pop = pop.nextGen(breedingSelections, mutationRate).fitToString(target)

    // output the "best" child for each generation
    println(s"generation $i: ${pop.head}")
  }
}
```

And some selected output...

```
@ run
Enter something to evolve: Daddy and Isabel programmed evolution using Scala!
generation 1000: D`dbz `md"Iqcaik$npngqaopgg!gvmktshom"uuhoh!Rcao`
generation 2000: Daddy amd Iqcaek!qpngraomge!fvnktshon"urhoh Scala!
generation 3000: Daddy amd Itabek qqngrammee!fvnkttion!urinh Scala!
generation 4000: Daddy and Itabek pqngrammee!fvnkttion urinh Scala!
generation 5000: Daddy and Itabel programmed fvnkttion uring Scala!
generation 6000: Daddy and Itabel programmed fvokttion using Scala!
generation 7000: Daddy and Isabel programmed evolttion using Scala!
generation 8000: Daddy and Isabel programmed evolttion using Scala!
generation 9000: Daddy and Isabel programmed evolution using Scala!
generation 10000: Daddy and Isabel programmed evolution using Scala!
```

---

In the end, she was able to grasp how selective breeding - combined with tiny, random changes - over a long periods of time could transform into something much better. This was true even if what was started with was something small and non-sensical and the target was extremely large (we used [lorem ipsum][lorem] as the target and she noticed that it took a much larger population several hundred thousand generations to evolve to it).

I later extended the program a bit allowing her to play with the other parameters. This allowed her - on her own - to reach conclusions regarding some rather complex topics:

* Inbreeding - small populations were unable to evolve;
* Genetic malformations - very high mutation rates prevented evolution;
* Selection of the fittest - too few mating choices led to unfit children;

But I'll leave those bits as a tiny exercise for the reader...

# fin.

[broad]: https://www.broadinstitute.org
[bases]: https://en.wikipedia.org/wiki/Nucleobase
[dna]: https://en.wikipedia.org/wiki/DNA
[evolution]: https://en.wikipedia.org/wiki/Evolution
[chromosome]: https://en.wikipedia.org/wiki/Chromosome
[inheritance]: https://en.wikipedia.org/wiki/Mendelian_inheritance
[centromere]: https://en.wikipedia.org/wiki/Centromere
[ammonite]: http://ammonite.io/#Ammonite-REPL
[crossover]: https://en.wikipedia.org/wiki/Crossover_(genetic_algorithm)
[mutation]: https://en.wikipedia.org/wiki/Mutation_(genetic_algorithm)
[fitness]: https://en.wikipedia.org/wiki/Fitness_function
[lorem]: https://en.wikipedia.org/wiki/Lorem_ipsum