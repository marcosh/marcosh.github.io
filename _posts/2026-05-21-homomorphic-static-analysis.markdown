---
layout: post
title:  "Homomorphic static analysis"
author: Marco Perone
date: 2026-05-21 08:06:42 +0200
categories: post
tags: haskell category-theory transformers
comments: false
pageUrl: '"http://marcosh.github.io/post/2026/05/21/homomorphic-static-analysis.html"'
pageIdentifier: '"Homomorphic static analysis"'
description: "Homomorphic static analysis"
image: "/img/homomorphic.jpeg"
---

It is somehow shared knowledge among Haskellers that applicatives allow performing static analysis, while monads do not. But what does it mean in practice? And what kind of static analysis is this referring to?

<br>

The best answer I could find online is [this answer](https://stackoverflow.com/a/20591008/2718064) on StackOverflow. In this post I'd like to expand on it a bit and take the concept a bit further[^1].

## Contextual interface

Consider an example interface dependent on a generic context `m`, implemented as a record of functions:

```haskell
data Interface m = Interface
  { foo :: FooInput -> m FooOutput
  , bar :: BarInput -> m BarOutput
  , baz :: BazInput -> m BazOutput
  }
```

This is an approach for defining interfaces in a pure language which has been discussed and proposed multiple times[^2] and works pretty well. It defines a set of operations which work in the same context and which can be used to define more complex programs[^2].

```haskell
myProgram :: Interface m -> Input -> m Output
myProgram interface input = ...
```

`myProgram` will use the basic operations defined by `Interface m`, but how such operations can be combined is defined by `m`. If `m` admits an instance of `Functor`, we are able to execute one of its operations and then map the result with a pure function. In case `m` admits an instance of `Applicative`, we are able to independently execute operations from `Interface m` and then combine their results. If `m` admits an instance of `Monad`, we will be able to combine operations from `Interface m` sequentially, deciding whether to execute an operation based on the result of a previous one.

## Interpreting the code without executing it

In our production code `myProgram` will be interpreted in some effectful context `m` where we will execute all the required effects. For example `m` could be a concrete monad, a stack of monad transformers or an `Eff` monad from your favourite effect system.

<br>

Still, we might want to collect some information about our code without performing any of the concrete effects; examples of this could be analysing some code metric, computing a list of dependencies or creating a graphical representation of our code flow.

<br>

One way to achieve this is by interpreting `myProgram` in another context `m'` which happen not to perform any of the effects, but it's there just to collect information. Once we have this we can write another executable which interprets `myProgram` in `m'` and retrieves the information we are interested in without executing any effect. This could be done without running our application and that's why it is called static analysis, even if it requires running some code.

## Interpreting in a data-agnostic context

One way of defining a context `m'` like the one described in the last paragraph of the previous section is to make it data-agnostic. With this I mean a data structure like

```haskell
newtype StaticInterpreter a = StaticInterpreter StaticDatum
```

The relevant thing of `StaticInterpreter` is that `a` is a phantom type, meaning that it appears in the type, but there is nothing of that type at the value level.

<br>

The benefit of this is that we can now combine values of `StaticInterpreter` even if they are tagged with different types. For example, suppose we have `StaticDatum = Int`, then we can combine `StaticInterpreter i1 :: StaticInterpreter a1` and `StaticInterpreter i2 :: StaticInterpreter a2` and obtain `StaticInterpreter (i1 + i2) :: StaticInterpreter a`.

## Constraining context constrains static datum

Let's now try to see what happens to `StaticDatum` as soon as we start imposing constraints on the `m` from `myProgram`.

<br>

If we need `m` to be a `Functor`, then also `StaticInterpreter` should admit a `Functor` instance

```haskell
instance Functor StaticInterpreter where
  fmap :: (a -> b) -> StaticInterpreter a -> StaticInterpreter b
  fmap f (StaticInterpreter datum) = StaticInterpreter datum
```

To implement it, we have no constraint to impose on `StaticDatum`. This means that if we just need `m` to be a `Functor`, we can statically analyse `myProgram` and interpret it with any possible type.

<br>

Usually though our programs will require something more than `Functor`. Our next step is requiring `m` to have an `Applicative` instance. Let's see what happens to `StaticDatum` as soon as we do that.

```haskell
instance Applicative StaticInterpreter where
  pure :: a -> StaticInterpreter a
  pure a = StaticInterpreter _

  (<*>) :: StaticInterpreter (a -> b) -> StaticInterpreter a -> StaticInterpreter b
  (<*>) (StaticInterpreter sd1) (StaticInterpreter sd2) = StaticInterpreter _
```

We basically need a way to create a `StaticDatum` out of nothing, and a way to combine two `StaticDatum` values into a new `StaticDatum` value. In other terms, we need

```haskell
_outOfNothing :: StatidDatum

_combineTwo :: StaticDatum -> StaticDatum -> StaticDatum
```

But this is exactly what a `Monoid` instance on `StaticDatum` gives us!

<br>

This means that we can use any type with a `Monoid` instance to statically analyse a program which is constrained by an `Applicative` constraint. For example `StaticDatum` could be a list of values and interpreting `myProgram` in `StaticInterpreter` would collect and concatenate all elements[^4].

<br>

Let's now see what happens if we require `StaticInterpreter` to have a `Monad` instance.

```haskell
instance Monad StaticInterpreter where
  (>>=) :: StaticInterpreter a -> (a -> StaticInterpreter b) -> StaticInterpreter b
  (>>=) (StaticInterpreter sd) f = _
```

If we want a non-trivial implementation of `>>=` (i.e. we don't want to return `StaticInterpreter mempty` every time), we would need to use `f` to obtain a value of type `StaticInterpreter b`. But to call `f` we need a value of type `a`, which we can't extract from `StaticInterpreter a`, because `a` there is actually a phantom type and there's no value of type `a` in there. The fact that we can not implement `>>=` is the reason why we usually say that `Monad`s do not allow static analysis[^5].

## Let's play the same game with categories

In the previous section we had a look at what kind of static analysis we can perform on a program written in a given context `m`. We concluded that if `m` is just required to have an `Applicative` instance, we can statically analyse our program with any monoid, while if we require an instance of `Monad`, then we basically lose our ability to introspect our code statically.

<br>

What could be interesting now (it is for me!) is trying to apply the same reasoning working with categories instead of functors/applicatives/monads. This would mean considering types with kind `Type -> Type -> Type` instead of `Type -> Type`. Examples of this could be simply `(->)` or [`Kleisli m`](https://hackage-content.haskell.org/package/base-4.22.0.0/docs/Control-Arrow.html#t:Kleisli)[^6].

<br>

Let's consider, as above, a data type which is defined with phantom types

```haskell
newtype CatStaticInterpreter a b = CatStaticInterpreter StaticDatum
```

and see what happens when we try to implement instances of several type classes. Let's start with the `Profuctor` typeclass.

```haskell
instance Profunctor CatStaticInterpreter where
  dimap :: (a -> b) -> (c -> d) -> CatStaticInterpreter b c -> CatStaticInterpreter a d
  dimap f g (CatStaticInterpreter sd) = CatStaticInterpreter sd
```

As we did above with `Functor`, we just ignore the `f` and `g` functions, and we keep the same `StaticDatum` value.

<br>

Next we can investigate what happens with the `Category` typeclass

```haskell
instance Category CatStaticInterpreter where
  id :: CatStaticInterpreter a a
  id = _

  (.) :: CatStaticInterpreter b c -> CatStaticInterpreter a b -> CatStaticInterpreter a c
  (.) (CatStaticInterpreter sd1) (CatStaticInterpreter sd2) = _
```

Similarly to what was happening with the `Applicative` typeclass in the previous section, we can notice that we can implement the `Category` instance for `CatSTaticInterpreter` if `StaticDatum` admits a `Monoid` instance. In that case we can define

```
  id = CatStaticInterpreter mempty

  (.) (CatStaticInterpreter sd1) (CatStaticInterpreter sd2) = CatStaticInterpreter (sd1 <> sd2)
```

Please notice here that, while `Monad` and `Category` are equally powerful from a computational point of view, the former does not admit any kind of static analysis, while the latter does.

## An example with `Strong` and `Choice`

Thing start to get even more interesting as soon as we start other constraints like `Strong` or `Choice`. For each one of these typeclasses we need another binary operation on `StaticDatum` which would correspond to [`(***)`](https://github.com/ekmett/profunctors/blob/0cc1d9dd0363c57645621c24e271986ca0920544/src/Data/Profunctor/Strong.hs#L119) and [`(+++)`](https://github.com/ekmett/profunctors/blob/0cc1d9dd0363c57645621c24e271986ca0920544/src/Data/Profunctor/Choice.hs#L111) respectively.

<br>

For this use case, let's define `StaticDatum` as

```haskell
data StaticDatum
  = Unit Text
  | Compose StaticDatum StaticDatum
  | Alternative StaticDatum StaticDatum
  | Parallel StaticDatum StaticDatum
```

With this definition we can define instances for `Category`, `Strong` and `Choice` for `CatStaticInterpreter`.

```haskell
instance Category CatStaticInterpreter where
  id = CatStaticInterpreter Unit
  (.) (CatStaticInterpreter sd1) (CatStaticInterpreter sd2) = CatStaticInterpreter (Compose sd1 sd2)

instance Strong CatStaticInterpreter where
  first' :: CatStaticInterpreter a b -> CatStaticInterpreter (a, c) (b, c)
  first' (CatStaticInterpreter sd) = CatStaticInterpreter (Parallel sd (Unit "id"))

  second' :: CatStaticInterpreter a b -> CatStaticInterpreter (c, a) (c, b)
  second' (CatStaticInterpreter sd) = CatStaticInterpreter (Parallel (Unit "id") sd)

instance Choice CatStaticInterpreter where
  left' :: CatStaticInterpreter a b -> CatStaticInterpreter (Either a c) (Either b c)
  left' (CatStaticInterpreter sd) = CatStaticInterpreter (Alternative sd (Unit "id"))

  right' :: CatStaticInterpreter a b -> CatStaticInterpreter (Either c a) (Either c b)
  right' (CatStaticInterpreter sd) = CatStaticInterpreter (Alternative (Unit "id") sd)
```

At this point let's define an interface depending on a category `p`

```haskell
data Interface p a b c d e f g h = Interface
  { foo1 :: p a b
  , foo2 :: p a c
  , foo3 :: p a (Either a a)
  , foo4 :: p a d
  , foo5 :: p a e
  , foo6 :: p (b, c) f
  , foo7 :: p (Either d e) g
  , foo8 :: p (f, g) h
  }
```

With it, we can define an example function `foo` consuming that interface

```haskell
foo :: (Category p, Strong p, Choice p) => Interface p a b c d e f g h -> p a h
foo Interface {..} =
  foo8 . ((foo6 . (foo1 &&& foo2)) &&& (foo7 . (foo4 +++ foo5) . foo3))
```

Using our `CatStaticInterpreter` we can now define an implementation for `Interface CatStaticInterpreter`

```haskell
implementation :: Interface CatStaticInterpreter a b c d e f g h
implementation =
  Interface
    { foo1 = CatStaticInterpreter (Unit "foo1")
    , foo2 = CatStaticInterpreter (Unit "foo2")
    , foo3 = CatStaticInterpreter (Unit "foo3")
    , foo4 = CatStaticInterpreter (Unit "foo4")
    , foo5 = CatStaticInterpreter (Unit "foo5")
    , foo6 = CatStaticInterpreter (Unit "foo6")
    , foo7 = CatStaticInterpreter (Unit "foo7")
    , foo8 = CatStaticInterpreter (Unit "foo8")
    }
```

Eventually we can statically analyse `foo` with `CatStaticInterpreter` just by evaluating `foo implementation`

```haskell
>>> foo implementation
Compose
  (Unit "foo8")
  (Compose
    (Parallel
      (Compose
        (Unit "foo6")
        (Compose
          (Parallel (Unit "foo1") (Unit "id"))
          (Parallel (Unit "id") (Unit "foo2"))
        )
      )
      (Unit "id")
    )
    (Parallel
      (Unit "id")
      (Compose
        (Unit "foo7")
        (Compose
          (Compose
            (Alternative (Unit "foo4") (Unit "id"))
            (Alternative (Unit "id") (Unit "foo5"))
          )
          (Unit "foo3")
        )
      )
    )
  )
```

For example, with this tree-like representation, it's not hard to traverse it and check whether a specific pattern is used (just a single function, or a more complex combination), or even optimise and restructure the tree. For example any `(Compose (Parallel (Unit "foo1") (Unit "id")) (Parallel (Unit "id") (Unit "foo2")))` could probably be reinterpreted to `(Parallel (Unit "foo1") (Unit "foo2"))`.

<br>

Another possibility is printing out a graphical representation of the resulting tree, which documents the flow of our program. For `foo` it could look like

![Profunctor flow](/img/profunctor_flow.svg)

## Conclusion

To recap what we went through in this post, we saw that we can analyse statically any code working in an `Applicative` context with any monoid; on the other hand monads do not allow non-trivial static analysis.

<br>

Moving to a categorical realm, which is as powerful as the monadic one, we saw that we can actually perform static analysis, and actually adding more constraints allow us to have more powerful understanding of the structure of our code. In our example we saw what we can do using `Strong` and `Choice`. One can use the same trick with `Closed`, which deals with functions, `CoStrong`, which deals with recursion, or `CoChoice`.

<br>

---

<br>

[^1]: See also the two posts from Chris Penner on a similar topic: https://chrispenner.ca/posts/expressiveness-spectrum and https://chrispenner.ca/posts/arrow-effects.

[^2]: See for example [Four ways of declaring interfaces in Haskell](https://marcosh.github.io/post/2025/07/22/four-ways-of-declaring-interfaces-in-haskell.html) by myself, [Scrap your type classes](https://haskellforall.com/2012/05/scrap-your-type-classes) by Gabriella Gonzales or [Embracing Flexibility in Haskell Libraries: The Power of Records of Functions](https://www.iankduncan.com/engineering/2024-01-26-records-of-effects) by Ian Duncan.

[^3]: What we discuss it not strictly related to records of functions. The same concepts can be applied to the usage of type classes, where an instance actually require you to write a default record of function for a specific type.

[^4]: The fact that in this way we respect the algebraic structure is the reason why I decided to use the word ["homomorpic"](https://en.wikipedia.org/wiki/Homomorphism) in the title.

[^5]: In between `Applicative` and `Monad` sit `Selective` functors, which define a `select :: f (Either a b) -> f (a -> b) -> f b ` combinator. If we try to write an instance for `Selective StaticInterpreter`, we could choose to implement `select` with `<>` and just keep our monoids; it would probably be more interesting to add another operation in our target algebraic structure so that we can clearly track when `select` is used. Does anyone have examples of static analysis of `Selective` functors which is not using a `Monoid`?

[^6]: Or even a stack of [`ProKleisli m`](https://marcosh.github.io/post/2026/05/08/categorical-transformers.html).
