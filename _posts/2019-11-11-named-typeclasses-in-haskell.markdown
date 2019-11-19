---
layout: post
title:  "Named typeclasses in Haskell"
author: Marco Perone
date: 2019-11-11 08:06:42 +0200
categories: post
tags: functional-programming haskell named-constructor
comments: true
pageUrl: '"http://marcosh.github.io/post/2019/11/11/named-typeclasses-in-haskell.html"'
pageIdentifier: '"named-typeclasses-in-haskell"'
description: "Named typeclasses in Haskell"
image: "/img/names.jpg"
---

One of the best features of Haskell are typeclasses. Still, it is not easily possible to have multiple implementation of the same typeclass for the same data type. For example, to define an additive and a multiplicative instance of `Monoid` on `Int`, one needs to use specific newtype wrappers such as [`Sum`](https://hackage.haskell.org/package/base-4.12.0.0/docs/Data-Monoid.html#t:Sum) and [`Product`](https://hackage.haskell.org/package/base-4.12.0.0/docs/Data-Monoid.html#t:Product).

On the other hand, in [Idris](https://www.idris-lang.org/) it is possible to define [multiple named implementations](http://docs.idris-lang.org/en/latest/tutorial/interfaces.html#named-implementations) of the same typeclass for the same data type.

A question now naturally arises:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">is there a way in <a href="https://twitter.com/hashtag/haskell?src=hash&amp;ref_src=twsrc%5Etfw">#haskell</a> to have named typeclasses implementations (i.e. different implementations of a typeclass on the same data structure, without newtypes)? Or is there a reason why they are not there?</p>&mdash; Marcosh (@marcoshuttle) <a href="https://twitter.com/marcoshuttle/status/1193884736852246530?ref_src=twsrc%5Etfw">November 11, 2019</a></blockquote>

## Some extensions later...

Almost immediately [Sjoerd](https://twitter.com/sjoerd_visscher) came to the rescue and proposed the following

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Maybe you can add an extra parameter and then use phatom types, AllowAmbiguousInstances and TypeApplications to pick one?</p>&mdash; Sjoerd 슕 Visscher (@sjoerd_visscher) <a href="https://twitter.com/sjoerd_visscher/status/1193888232011894790?ref_src=twsrc%5Etfw">November 11, 2019</a></blockquote>

Let's briefly review what these mentioned things are.

A [phantom type](https://wiki.haskell.org/Phantom_type) is a parametrised type whose parameters do not all appear on the right-hand side of its definition.

For example `newtype Const a b = Const { getConst :: a }` is a phantom type because the parameter `b` does not appear after the `=` sign.

Phantom types are useful because they allow to add at the type level information which is not present at the value level.

A type parameter is said to be `ambiguous` if it can not be correctly inferred by the type checker. This happens with phantom types because the compiler, having no way to infer the actual type of the phantom parameter, considers it ambiguous. The extension `AllowAmbiguousTypes` allows us to ask the compiler to be a little more patient and not error immediately when it detects an ambiguous type.

Still, since the compiler is not able to infer it, we need a way to manually specify the value of an ambiguous type. This is possible with the `@` syntax using the [`TypeApplications`](https://downloads.haskell.org/~ghc/8.2.1/docs/html/users_guide/glasgow_exts.html#ghc-flag--XTypeApplications) extension.

Let's see these techniques in action while we try to solve our initial question.

## Named typeclasses

We will start from the

```haskell
class Monoid a where
  (<>)   :: a -> a -> a
  mempty :: a
```

typeclass, which lives in the Haskell [`Prelude`](https://hackage.haskell.org/package/base-4.12.0.0/docs/Prelude.html#t:Monoid).

As Sjoerd suggested, we can start by adding a new type parameter to it, which will be used later to do some type-level magic.

```haskell
{-# LANGUAGE AllowAmbiguousTypes    #-}
{-# LANGUAGE MultiParamTypeClasses  #-}

class Monoid b a where
  (<>)   :: a -> a -> a
  mempty :: a
```

Then we can define some instances on the `Int` type.

```haskell
import Prelude hiding (Monoid, mempty, (<>))

data Sum
data Product

instance Monoid Sum Int where
  x <> y = x + y
  mempty = 0

instance Monoid Product Int where
  x <> y = x * y
  mempty = 1
```

We need to define the ancillary data types `Sum` and `Product`, whithout any constructor, which we use as tags to distinguish the two different implementations of `Monoid` on `Int`[^1].

Next we want to define a function which uses our `Monoid` class. One typical operation which is performed on `Monoid`s is a `fold` which recursively combines the elements of a list into a single element.

```haskell
fold :: Monoid b a => [a] -> a
```

To implement it we actually need  to use `TypeApplications` but also the `ScopedTypeVariables` extension. This is due to the fact that generally type variables have no notion of scope; this means that anytime we would use `(<>)` or `mempty`, the compiler would introduce a new `b` type variable. This leads to errors as `Could not deduce (NamedTypeclass.Monoid b0 a) from the context: NamedTypeclass.Monoid b a`, leaving us to wonder where that `b0` comes from. `ScopedTypevariables` allows us to use the `forall` keyword to define explicitly the scope of a type variable.

Our definition of `fold` then becomes

```haskell
{-# LANGUAGE ScopedTypeVariables #-}
{-# LANGUAGE TypeApplications    #-}

fold :: forall b a . Monoid b a => [a] -> a
fold []       = mempty @b
fold (x : xs) = (<>) @b x (fold @b xs)
```

Notice also how we had to use the `@` syntax of `TypeApplications` to explicitly state that we are using the same `b` everywhere.

At this point it becomes just a matter of using `TypeApplications` to define specific folds on both our instances of `Monoid`.

For example we could define

```haskell
sum :: [Int] -> Int
sum = fold @Sum

product :: [Int] -> Int
product = fold @Product
```

This allows us to use two different instances of the `Monoid` class on the same `Int` data type, without having to resort to newtype wrappers.

## Single instances

Suppose now that for a datatype we want to define a unique instance of `Monoid`, as we did in the good old days when the phantom parameter `b` was not there.

For example, suppose that for lists we always want to define the same instance of `Monoid`, where the operation is given by concatenation.

```haskell
{-# LANGUAGE FlexibleInstances #-}

instance Monoid b [a] where
  x <> y = x ++ y
  mempty = []
```

Notice that we needed to enable the [`FlexibleInstances`](https://downloads.haskell.org/~ghc/8.2.1/docs/html/users_guide/glasgow_exts.html#ghc-flag--XFlexibleInstances) extension, which allows us to use `[a]` as a type parameter.

Our instance definition amounts to say that for any possible `b` the definition of our instance is the same. The compiler is then smart enough to use this information and actually not require us to write any `TypeApplication` when we use the `fold` function on the `[a]` data type.

```haskell
concat :: [[a]] -> [a]
concat = fold
```

## Conclusion

We just went through a possible way to implement named typeclasses in Haskell. It could be a useful technique for newly defined typeclasses which you control, while it still remains hard to make such an approach interact nicely with already defined typeclasses.

There are other approaches to this same problem, as mentioned by [@Iceland_jack](https://twitter.com/Iceland_jack)

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">There are several approaches to it, there is <a href="https://t.co/aCGEABQUtP">https://t.co/aCGEABQUtP</a> which works at the level of dictionaries and my own wip <a href="https://t.co/UdIgyR4G34">https://t.co/UdIgyR4G34</a> that works at the level of types</p>&mdash; _j (@Iceland_jack) <a href="https://twitter.com/Iceland_jack/status/1193905912802693123?ref_src=twsrc%5Etfw">November 11, 2019</a></blockquote>

If you want to take a look at complete code, take a look at [this gist](https://gist.github.com/marcosh/8b697252a9e8020729f46e040ebe3248).

---

[^1]: Again from Sjoerd: if you want, you could use `GHC.Types (Symbol)` and write `instance Monoid "sum" Int where ...` and avoid defining the ancillary types.

<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
