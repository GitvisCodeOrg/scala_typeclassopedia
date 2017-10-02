# Scala typeclassopedia

* Resources covering all topis about FP in greater details in way better and more approachable form:
  * [Functional Programming in Scala - Paul Chiusano and Runar Bjarnason]((https://www.manning.com/books/functional-programming-in-scala)) Best book about FP in Scala. I have bought it for myself and higly recommend it. Worth reading, doing exercises and re-reading.
  * [Functional Structures in Scala - Michael Pilquist](https://www.youtube.com/watch?v=Dsd4pc99FSY&list=PLFrwDVdSrYE6dy14XCmUtRAJuhCxuzJp0): videos explaining [implementation of Monad, Traversable etc](https://github.com/mpilquist/Structures) with usage information
  * eed3si9n series of blog posts with clear explanations and examples: [herding cats](http://eed3si9n.com/herding-cats/) [learning Scalaz](http://eed3si9n.com/learning-scalaz/)
  * [scalaz usage examples](https://github.com/scalaz/scalaz/tree/series/7.3.x/example/src/main/scala/scalaz/example)
  * [docs for Cats](https://typelevel.org/cats/) (runnable online version on [ScalaExercises](https://www.scala-exercises.org/cats/semigroup))

## Abstract Algebra

### Monoid
* Brent Yorgey: Monoids - theme and variations 
[(video)](https://www.youtube.com/watch?v=X-8NCkD2vOw)
[(paper)](http://repository.upenn.edu/cgi/viewcontent.cgi?article=1773&context=cis_papers)
* On Monoids - Runar: https://apocalisp.wordpress.com/2010/06/14/on-monoids/
* * scalaz [src](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Monoid.scala)

### Semigroup
* Resources
  * scalaz [src](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Semigroup.scala)

## Cathegory Theory

### (Covariant) Functor

Abstraction for type constructor (type with "hole", type parameter) that can be mapped over.

Containers (List, Tree, Option) can apply given function to every element in the collection.
Computation effects (Option - may not have value, List - may have multiple values,
 Either/Validated - may contain value or error) can apply function to a value inside this effect without changing the effect.
```scala
trait Functor[F[_]] {
  def map[A,B](a: F[A])(f: A => B): F[B]
}
```
* Functor Laws:
1. identify: `xs.map(identity) == xs` map identify function does nothing
2. composition: `xs.map(f).map(g) == xs.map(x => g(f(x))` you can map using f and then using g or map once using composition
If Functor satisfy fist law then it also satisfy second law: [(Haskell) The second Functor law is redundant - David Luposchainsky](https://github.com/quchen/articles/blob/master/second_functor_law.md) if we don't include bottom values [(Haskell) contexample using undefined](https://stackoverflow.com/questions/8305949/haskell-functor-implied-law/8323243#8323243)
* Instances can be [implemented](https://github.com/lemastero/learn_scala_cats/blob/master/src/main/scala/functor/InstancesForBuiltInTypes.scala) for: List, Option, Either, Validated, Tuple1, Tuple2, [Function varying return type](https://www.youtube.com/watch?v=Dsd4pc99FSY&t=1075)
* Functor must preserve structure, so Set is not a Functor (map constant function).
* Functors can be composed
* Derived methods of Functor: [(examples)](https://github.com/lemastero/learn_scala_cats/blob/master/src/test/scala/functor/FunctorExamplesSpec.scala)
```scala
def lift[A, B](f: A => B): F[A] => F[B] // lift regular function to function inside container
def fproduct[A, B](fa: F[A])(f: A => B): F[(A, B)] // zip elements with result after applying f
def as[A, B](fa: F[A], b: B): F[B] // replace every element with b
def void[A](fa: F[A]): F[Unit] // clear preserving structure
def tupleLeft[A, B](fa: F[A], b: B): F[(B, A)]
def tupleRight[A, B](fa: F[A], b: B): F[(A, B)]
def widen[A, B >: A](fa: F[A]): F[B]
```
* Resources:
  * herding cats - Functor: [blog post](http://eed3si9n.com/herding-cats/Functor.html)
  * FSiS 1, Type Constructors, Functors, and Kind Projector - Michael Pilquist [video](https://www.youtube.com/watch?v=Dsd4pc99FSY)
  * (Haskell) The Extended Functor Family - George Wilson [video](https://www.youtube.com/watch?v=JUVMiRRq6wU)
  * Cats [docs](https://typelevel.org/cats/typeclasses/functor.html) [src](https://github.com/typelevel/cats/blob/master/core/src/main/scala/cats/Functor.scala)
  * scalaz [src](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Functor.scala)

### Contravariant (Functor)
```scala
trait Contravariant[F[_]] {
  def contramap[A, B](f: B => A): F[A] => F[B]
}
```
* Resources
  * scalaz [(src)](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Contravariant.scala)
  * (Haskell) The Extended Functor Family - George Wilson [video](https://www.youtube.com/watch?v=JUVMiRRq6wU&t=450)

### Exponential Functor (Invariant Functor)
```scala
trait Exponential[F[_]] {
  def xmap[A, B](f: (A => B, B => A)): F[A] => F[B]
}
```

### Bifunctor
Abstracts over type constructor with 2 "holes". Represents two independent functors:
```scala
trait Bifunctor[F[_, _]] {
  def bimap[A, B, C, D](fab: F[A, B])(f: A => C, g: B => D): F[C, D]
}
```
* Bifunctor Laws
1. identity `xs.bimap(identity, identity) == xs` bimap with two identify function does nothing
2. composition `xs.bimap(f, h).bimap(g,i) == xs.bimap(x => g(f(x), x => h(i(x))`  you can bimap using f and h and then bimap using g and i or bimap once using composition
Second law is automatically fulfilled if the first law holds.
* Alternatively can be specified by providing
```scala
def leftMap[A, B, C](fab: F[A, B])(f: A => C): F[C, B]
def rightMap[A, B, D](fab: F[A, B])(g: B => D): F[A, D]
```
In that case identity law must hold for both functions:
3. identity `xs.leftMap(identity) == xs` leftMap with identify function does nothing
4. identity `xs.rightMap(identity) == xs` rightMap with identify function does nothing
If leftMap and rightMap and bimap are specified then additional lwa must be fullfilled:
5. `xs.bimap(f, g) == xs.leftMap(f).rightMap(g)`
* Derived methods
```scala
def leftMap[A, B, C](fab: F[A, B])(f: A => C): F[C, B]
def rightMap[A, B, D](fab: F[A, B])(g: B => D): F[A, D]
def leftFunctor[X]: Functor[F[?, X]]
def rightFunctor[X]: Functor[F[X, ?]]
def umap[A, B](faa: F[A, A])(f: A => B): F[B, B]
def widen[A, B, C >: A, D >: B](fab: F[A, B]): F[C, D]
```
* Instances [can be defined](https://github.com/lemastero/learn_scala_cats/blob/master/src/main/scala/functor/bifunctor/InstancesForForBuildInTypes.scala) for: Tuple2, Either, Validated. For Function1 not - functions are contravariant for input type.
* Resources
  * Funky Scala Bifunctor - Tony Morris [(blog post)](http://blog.tmorris.net/posts/funky-scala-bifunctor/index.html)
  * (Haskell) The Extended Functor Family - George Wilson: [video](https://www.youtube.com/watch?v=JUVMiRRq6wU&t=303)
  * [examples of usage in scalaz](https://github.com/scalaz/scalaz/blob/series/7.3.x/example/src/main/scala/scalaz/example/BifunctorUsage.scala)
  * scalaz [(src)](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Bifunctor.scala)
  * cats [(src)](https://github.com/typelevel/cats/blob/master/core/src/main/scala/cats/functor/Bifunctor.scala)

### Profunctor
* (Haskell) Fun with Profunctors - Phil Freeman: https://www.youtube.com/watch?v=OJtGECfksds
  
### Divide (Contravariant Apply)

* Resources
  * scalaz [(src)](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Divide.scala)

### Applicative (Functor)

Applicative Functor is a Functor that can:
- put value into container (lift into effect)
- apply function already inside container to container
```scala
trait Applicative[F[_]] extends Functor[F] {
  def pure[A](value: A): F[A]  
  def apply[A, B](fa: F[A])(ff: F[A => B]): F[B]
}
```
* Applicative Laws:
1. identify: `xs.apply(pure(identity)) == xs` apply identify function lifted inside effect does nothing
2. homomorphism: `pure(a).apply(pure(f)) == pure(f(a))` lifting value a and applying lifted function f is the same as apply function to this value and then lift result
3. interchange: `pure(a).apply(ff) == ff.apply(pure(f => f(a)))` where `ff: F[A => B]`
4. map: `fa.map(f) == fa.apply(pure(f))`
* Derived methods:
```scala
def apply [A, B]   (fa: F[A])          (ff: F[A => B]):     F[B]
def apply2[A, B, Z](fa: F[A], fb: F[B])(ff: F[(A,B) => Z]): F[Z]
// ...
def map2[A , B, Z]  (fa: F[A], fb: F[B])          (f: (A, B) => Z):    F[Z]
def map3[A, B, C, Z](fa: F[A], fb: F[B], fc: F[C])(f: (A, B, C) => Z): F[Z]
// ...
def tuple2[A, B]   (fa: F[A], fb: F[B]):           F[(A, B)]
def tuple3[A, B, C](fa: F[A], fb: F[B], fc: F[C]): F[(A, B, C)]
// ...
def flip[A, B](ff: F[A => B]): F[A] => F[B]
```
* Applicatives can be composed
* Minimal set of methods to implement Applicative (other methods can be derived from them):
  * map2, pure
  * apply, pure
* Resources:
  * herding cats - Applicative: [blog post](http://eed3si9n.com/herding-cats/Applicative.html)
  * FSiS 2 - Applicative type class - Michael Pilquist: [video](https://www.youtube.com/watch?v=tD_EyIKqqCk)
  * Cats: [docs](https://typelevel.org/cats/typeclasses/applicative.html) [src](https://github.com/typelevel/cats/blob/master/core/src/main/scala/cats/Applicative.scala)
  * scalaz: [src](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Applicative.scala)
  * Applicative programming with effects - Conor McBride, Ross Paterson [(shorter)](http://strictlypositive.org/IdiomLite.pdf) [longer](http://strictlypositive.org/Idiom.pdf)
  * The Essence of the Iterator Pattern - Jeremy Gibbons, Bruno C. d. S. Oliveira: [(paper)](https://www.cs.ox.ac.uk/jeremy.gibbons/publications/iterator.pdf)
    * The Essence of the Iterator Pattern - Eric Torreborre [blog post](http://etorreborre.blogspot.com/2011/06/essence-of-iterator-pattern.html)
    * Lifting - Tony Morris: [blog post](http://blog.tmorris.net/posts/lifting/index.html)
  * (Haskell) Abstracting with Applicatives - Gershom Bazerman [(blog post)](http://comonad.com/reader/2012/abstracting-with-applicatives/)
  * (Haskell) Algebras of Applicatives - Gershom Bazerman [(blog post)](http://comonad.com/reader/2013/algebras-of-applicatives/)
  * (Haskell) Representing Applicatives - Gershom Bazerman [(blog post)](http://comonad.com/reader/2013/representing-applicatives/)

### Divisible (Contravariant Applicative)

### Apply

* Resources
  * scalaz [(src)](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Apply.scala)

### Monad
We add to Apply ability `flatMap` that can joint two computation and use the output from previous computations to decide what computations to run next.
* Minimal set of methods to implement Monad (others can be derived):
  * pure, flatMap
  * pure, compose
  * pure, flatten, map
  * pure, flatten, apply
  * pure, flatten, map2


```scala
trait Monad[F[_]] extends Apply[F] { self =>
  def pure[A](value: A): F[A]
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
}
```
* [Monads do not compose -  Tony Morris](http://blog.tmorris.net/posts/monads-do-not-compose/index.html)
* Monad Laws
1.
2.
  
* Derived methods:
```scala
def flatten[A](ffa: F[F[A]]): F[A]
def sequence[G[_], A](as: G[F[A]])(implicit G: Traverse[G]): F[G[A]]
def traverse[A, G[_], B](value: G[A])(f: A => F[B])(implicit G: Traverse[G]): F[G[B]]
def replicateA[A](n: Int, fa: F[A]): F[List[A]]
def unit: F[Unit] // put under effect ()
def factor[A, B](ma: F[A], mb: F[B]): F[(A, B)]
```

* Resources
  * scalaz [(src)](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Monad.scala)

### State monad
* Resources
  * Memoisation with State using Scala - Tony Morris [(blog post)](http://blog.tmorris.net/posts/memoisation-with-state-using-scala/index.html)
  * Scalaz State Monad - Michael Pilquist: https://www.youtube.com/watch?v=Jg3Uv_YWJqI


### Reader monad
* Resources
  * The Reader Monad for Dependency Injection - Json Arhart: https://www.youtube.com/watch?v=xPlsVVaMoB0

### Writer Monad
* The Writer Monad using Scala (example) - Tony Morris: [blog post](http://blog.tmorris.net/posts/the-writer-monad-using-scala-example/index.html)

### IO monad

### MonadPlus

### Free Monoid, Free Monad, Free Applicative
* Resources
  * Free Monads in Cats docs: http://typelevel.org/cats/datatypes/freemonad.html
  * Free Monoids and Free Monads - Runar: http://blog.higher-order.com/blog/2013/08/20/free-monads-and-free-monoids/
  * Why the free Monad isn’t free - Kelley Robinson: https://www.youtube.com/watch?v=wvNgoeZza2g
  * Beyond Free Monads - John DeGoes: https://www.youtube.com/watch?v=A-lmrvsUi2Y
  * Free as in Monads - Daniel Spiewak: https://www.youtube.com/watch?v=aKUQUIHRGec
  * Move Over Free Monads: Make Way for Free Applicatives! - John deGoes: https://www.youtube.com/watch?v=H28QqxO7Ihc
  * (Haskell) Free Monoids in Haskell - Dan Doel [(blog post)](http://comonad.com/reader/2015/free-monoids-in-haskell/)
  * (Haskell) Many Roads to Free Monads - Dan Doel [(blog post)](https://www.schoolofhaskell.com/user/dolio/many-roads-to-free-monads)

### Foldable

### Traverse
* Finding all permutations of list: [(blog post haskell)](https://byorgey.wordpress.com/2007/06/26/deducing-code-from-types-filterm/) [(translation to Scala using Cats)](https://github.com/lemastero/learn_scala_cats/blob/master/src/main/scala/traverse/ListPermutationsUsingFilterM.scala)

### Arrow
* Learning Scalaz - Arrow - eed3si9n: http://eed3si9n.com/learning-scalaz/Arrow.html
* FixxBuzz using arrows [(blog post) (Haskell)](http://logicaltypes.blogspot.com/2014/02/arrow-is-spelt-fizz-buzz.html)

### Kleisli
* Resources
  * cats [docs](http://typelevel.org/cats/datatypes/kleisli.html) [src](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Kleisli.scala)
  * scalaz [docs](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Kleisli.scala)

### Yoneda, Coyoneda
* Resources
  * (Scala & Haskell) How Haskell is Changing my Brain, Yay Yoneda - Alissa Pajer: https://vimeo.com/96639840
  * (Haskell) Reverse Engineering Machines with the Yoneda Lemma - Dan Piponi: [(blog post)](http://blog.sigfpe.com/2006/11/yoneda-lemma.html)
  * scalaz [(Yoneda src)](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Yoneda.scala) [(Coyoneda src)](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Coyoneda.scala)

### Comonad, Cofree
```scala
trait Comonad[F[_]] {
  def extract[A]: F[A] => A
  def coflatMap[A, B](f: F[A] => B): F[A] => F[B]
}
```
* Resources
  * (Haskell) Getting a Quick Fix on Comonads - Kenneth Foner: https://www.youtube.com/watch?v=F7F-BzOB670
  * Streams for (Co)Free! - John DeGoes: https://www.youtube.com/watch?v=R_nYc4FItcI 
  * scalaz [(src Cofree)](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Cofree.scala) [(src Comonad)](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Comonad.scala)

### Adjunctions
* Resources:
  * Adjunctions in Everyday Life - Rúnar Bjarnason [(video Scala)](https://www.youtube.com/watch?v=BLk4DlNZkL8) [(vide Haskell)](https://www.youtube.com/watch?v=f-kdpR0BPqo)
  * scalaz [src](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Adjunction.scala)
  * (Haskell) Representing Adjunctions - Edward Kmett [(blog post)](http://comonad.com/reader/2008/representing-adjunctions/)

### Kan extensions
* Resources
  * (Haskell) Kan Extensions - Edward Kmett [blog post](http://comonad.com/reader/2008/kan-extensions/)
  * (Haskell) Kan Extensions II: Adjunctions, Composition, Lifting - Edward Kmett [blog post](http://comonad.com/reader/2008/kan-extensions-ii/)
  * (Haskell) Kan Extensions III: As Ends and Coends - Edward Kmett [blog post](http://comonad.com/reader/2008/kan-extension-iii/)
  * scalaz [src](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Kan.scala)

### ... and beyond
* Resources
  * Scala Type-class Hierarchy - Tony Morris [(blog post)](http://blog.tmorris.net/posts/scala-type-class-hierarchy/index.html) (traits for all cathegory theory constructions with exotic ones like `ComonadHoist`)
  * (Haskell) Free Monads for Less (Part 1 of 3): Codensity - Edward Kmett [(blog post)](http://comonad.com/reader/2011/free-monads-for-less/)
  * (Haskell) Free Monads for Less (Part 2 of 3): Yoneda - Edward Kmett [(blog post)](http://comonad.com/reader/2011/free-monads-for-less-2/)
  * (Haskell) Free Monads for Less (Part 3 of 3): Yielding IO - Edward Kmett [(blog post)](http://comonad.com/reader/2011/free-monads-for-less-3/)
  * (Haskell) Monads from Comonads - Edward Kmett [(blog post)](http://comonad.com/reader/2011/monads-from-comonads/)
  * (Haskell) Monad Transformers from Comonads - Edward Kmett [(blog post)](http://comonad.com/reader/2011/monad-transformers-from-comonads/)
  * (Haskell) More on Comonads as Monad Transformers - Edward Kmett [(blog post)](http://comonad.com/reader/2011/more-on-comonads-as-monad-transformers/)
  * (Haskell) The Cofree Comonad and the Expression Problem - Edward Kmett [(blog post)](http://comonad.com/reader/2008/the-cofree-comonad-and-the-expression-problem/)
  * (Haskell) Zapping Adjunctions - Edward Kmett [(blog post)](http://comonad.com/reader/2008/zapping-strong-adjunctions/)
  * (Haskell) Adjoint Triples - Dan Doel [(blog post)](http://comonad.com/reader/2016/adjoint-triples/) 

## Recursive schemas

* Resources:
  * (Haskell) Functional Programming with Bananas, Lenses, Envelopes and Barbed Wire - Erik Meijer, Maarten Fokkinga, Ross Paterson: [(paper Haskell)](https://maartenfokkinga.github.io/utwente/mmf91m.pdf)
  * (Haskell) Recursion Schemes: A Field Guide (Redux) - Edward Kmett: [(blog Haskell)](http://comonad.com/reader/2009/recursion-schemes/)
  * (Haskell) Rotten Bananas - Edward Kmett: http://comonad.com/reader/2008/rotten-bananas/
  * (Haskell) Generalized Hylomorphisms - Edward Kmett [(blog post)](http://comonad.com/reader/2008/generalized-hylomorphisms/)
  * (Haskell) Time for Chronomorphisms - Edward Kmett [(blog post)](http://comonad.com/reader/2008/time-for-chronomorphisms/)
  * (Haskell) Unnatural Transformations - Edward Kmett [(blog post)](http://comonad.com/reader/2008/unnatural-transformations/)
  * (Haskell) Dynamorphisms as Chronomorphisms - Edward Kmett [(blog post)](http://comonad.com/reader/2008/dynamorphisms-as-chronomorphisms/)
  * (Haskell) Elgot (Co)Algebras - Edward Kmett [(blog post)](http://comonad.com/reader/2008/elgot-coalgebras/) 
  
### Cathamorphism
* (Haskell) - Catamorphisms - Edward Kmett: [(article)](https://www.schoolofhaskell.com/user/edwardk/recursion-schemes/catamorphisms)

## Optics

### Iso
* Beyond Scala Lenses - Julien Truffaut: [(video)](https://www.youtube.com/watch?v=6nyGVgGEKdA)

### Prism
* Beyond Scala Lenses - Julien Truffaut: [(video)](https://www.youtube.com/watch?v=6nyGVgGEKdA&t=960)

### Lens
* Resources:
  * Beyond Scala Lenses - Julien Truffaut: [(video)](https://www.youtube.com/watch?v=6nyGVgGEKdA&t=1694)
  * scalaz [Lens src](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Lens.scala) [(PLens src)](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/PLens.scala)

## Effects
* The Eff monad, one monad to rule them all - Eric Torreborre: https://www.youtube.com/watch?v=KGJLeHhsZBo

## Functional data structures

### Zipper
* Resources:
  * learning Scalaz - Zipper [(blog post)](http://eed3si9n.com/learning-scalaz/Zipper.html)
  * Zippers by Example - George Wilson: https://www.youtube.com/watch?v=woK7ntZRwXQ
  * Zippers, Comonads & Data Structures in Scala - Mark Hibberd & Tony Morris: https://www.youtube.com/watch?v=WpA8VPekcK4
  * scalaz [(src)]()

### FingerTree
* Resources:
  * Extreme Cleverness: Functional Data Structures in Scala - Daniel Spiewak [(video)](https://www.youtube.com/watch?v=pNhBQJN44YQ)
  * scalaz [(src)](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/FingerTree.scala)
  * What's new in purely functional data structures since Okasaki? [theory](https://cstheory.stackexchange.com/questions/1539/whats-new-in-purely-functional-data-structures-since-okasaki)

## WIP Collections of interesting papers mostly in (Haskell)
* [Functional pearls](https://wiki.haskell.org/Research_papers/Functional_pearls) (Haskell some in OCaml)
* [School of Haskell article](https://www.schoolofhaskell.com/) 
* [The Comonad.Reader](http://comonad.com/reader/)
* [Monads and all that](https://www.cs.uoregon.edu/research/summerschool/summer12/curriculum.html) - John Hughes
* [The Monad.Reader](https://themonadreader.wordpress.com/)