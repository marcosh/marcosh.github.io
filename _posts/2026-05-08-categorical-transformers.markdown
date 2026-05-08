---
layout: post
title:  "Categorical transformers"
author: Marco Perone
date: 2026-05-08 08:06:42 +0200
categories: post
tags: haskell category-theory transformers
comments: false
pageUrl: '"http://marcosh.github.io/post/2026/05/08/categorical-transformers.html"'
pageIdentifier: '"Categorical transformers"'
description: "Categorical transformers"
image: "/img/27390-8-transformers-autobot-photos.jpeg"
---

Haskell is a programming language well known for its usage of [monads](https://en.wikipedia.org/wiki/Monad_(functional_programming)) and for the [over-abundance of tutorials trying to explain them](https://wiki.haskell.org/Monad_tutorials_timeline). This is not one of those posts. On the other hand, in this post I'll try to convince you that Haskell without monads (as a main concept) is not only possible, but maybe also a good idea.

<br>

This post is heavily inspired by the post [Exploring Arrows for sequencing effects](https://chrispenner.ca/posts/arrow-effects) by [Chris Penner](https://chrispenner.ca/). If you haven't read that post, I recommend you to do it, not only because it will help you read what follows, but also because it's nice in itself.

## Haskell, monads and categories

When you first learn Haskell, you meet fairly soon data structures like [`Maybe`](https://hackage.haskell.org/package/base/docs/Data-Maybe.html#t:Maybe), [`Either e`](https://hackage.haskell.org/package/base/docs/Data-Either.html#t:Either), [`Reader r`](https://hackage.haskell.org/package/transformers/docs/Control-Monad-Trans-Reader.html#t:Reader), [`State s`](https://hackage.haskell.org/package/transformers/docs/Control-Monad-Trans-State-Lazy.html#t:State) which admit an instance of the [`Monad`](https://hackage.haskell.org/package/base/docs/Control-Monad.html#t:Monad) typeclass:

```haskell
class Applicative m => Monad m where
  (>>=) :: forall a b. m a -> (a -> m b) -> m b
```

The essence of a `Monad` is that it allows sequential composition of actions and, together with [`do` notation](https://en.wikibooks.org/wiki/Haskell/do_notation), it allows writing purely functional code in an imperative style.

<br>

Certainly Haskell and functional programming allow writing imperative code in a much safer way with respect to traditional imperative languages, but didn't everyone came to Haskell for their interest and love of category theory? Why are we back to imperative code? Can't we just work with categories, morphisms, functors and natural transformations?

<br>

Luckily we have a way to transform monads into categories! We can define a data structure

```haskell
newtype Kleisli m a b = Kleisli { runKleisli :: a -> m b }
```

It's not hard to prove that `m` is a monad if and only if `Kleisli m` is a category. This way we have a straight connection between the monadic world and the categorical one.

<br>

Unfortunately the whole Haskell ecosystem was historically built around monads and not categories and hence foundational libraries like `transformers` and `mtl` are built around monads and do not really have a categorical counterpart.

<br>

This blog post is the beginning of an exploration of what Haskell could be if we used categories instead of monads.

## Monads, categories and effects

The last sentence of the previous section was actually a bit imprecise. I forgot to say for what we could want to use categories instead of monads. The answer to that is effects.

<br>

Haskell typically uses monads to describe effects, where an effect is any behaviour which would make a function impure. For example [`Maybe`](https://hackage.haskell.org/package/base/docs/Data-Maybe.html#t:Maybe) is used to model functions where a return value is not definable for some inputs; or [`Reader`](https://hackage.haskell.org/package/transformers/docs/Control-Monad-Trans-Reader.html#t:Reader) is used to model functions which return value does not depend only on the input but also on some additional configuration.

<br>

In this setting an effectful function could be modelled as a function of type `a -> m b` where `m` is a `Monad`. But this is exactly a morphism in the `Kleisli` category associated to `m`. In other terms, we could use the category `Kleisli m` instead of just `m` to talk about effectful functions.

## One monad is fine, two maybe not

Up to now, everything is bright and shiny, instead of talking about a monad `m` we can talk about its `Kleisli` category and work with categories instead of monads. Things start to be hairy when we want to consider multiple effects at the same time.

<br>

The issue here is that monads do not compose, meaning that if `m1` and `m2` are type constructors which allow `Monad` instances, it's not automatically given that [`Compose m1 m2`](https://hackage.haskell.org/package/base/docs/Data-Functor-Compose.html#t:Compose) admits a `Monad` instance.

<br>

The classical solution to this problem is using [monad transformers](https://hackage.haskell.org/package/transformers). The key observation here is that, while it is not true that `Compose m1 m2` admits a monad instance for every `m1` and `m2` with a `Monad` instance, it is true for some specific `m1` and `m2`.

<br>

For example `Compose m Maybe` admits a `Monad` instance anytime `m` does so. Or similarly, for any `Monad` `m`, `Compose ((->) r) m` admits a `Monad` instance. This way we can define the so-called monad transformers `MaybeT m`, which is isomorphic to `Compose m Maybe`, and `ReaderT r m`, which is isomorphic to `Compose ((->) r) m`.

<br>

At this point a question arises quite naturally: can we define a categorical equivalent to monad transformers? Glad you asked... this is exactly what we'll try to explore in the rest of the post.

## Categorical Kleisli categories

As we saw in the previous section, monads are types of kind `Type -> Type` and as such they can be composed easily.

<br>

[![](https://mermaid.ink/img/pako:eNptjT9vwyAUxL8KerNjYbDBMGRJx0xRp5YOxBA7agCLYLWp5e9eEvWPGnV7d_e7ezN0wViQcDiFt27QMaHtTnn9rODxMloFL2i1QpvgxnC2yBHkqmysUfcLKI_QHf8F7f9C-zuI_LcEBfTxaECmONkCnI1OXyXM11RBGqzLrMyn0fFVgfJL7ozaP4XgvmsxTP0A8qBP56ym0ehkH466j9r9uNF6Y-MmTD6BJDW-jYCc4T1LLkrCqkYwxgVreVPABSStcEmJoJwRgjmm1VLAx-0rLpuaUsxyKOq2FXWzfALvRGaP?type=png)](https://mermaid.live/edit#pako:eNptjT9vwyAUxL8KerNjYbDBMGRJx0xRp5YOxBA7agCLYLWp5e9eEvWPGnV7d_e7ezN0wViQcDiFt27QMaHtTnn9rODxMloFL2i1QpvgxnC2yBHkqmysUfcLKI_QHf8F7f9C-zuI_LcEBfTxaECmONkCnI1OXyXM11RBGqzLrMyn0fFVgfJL7ozaP4XgvmsxTP0A8qBP56ym0ehkH466j9r9uNF6Y-MmTD6BJDW-jYCc4T1LLkrCqkYwxgVreVPABSStcEmJoJwRgjmm1VLAx-0rLpuaUsxyKOq2FXWzfALvRGaP)

<br>

`Compose` takes two types of kind `Type -> Type`, which could be monads, and return another type of kind `Type -> Type`. Hence, we can see `Compose` having the signature `(Type -> Type) -> (Type -> Type) -> (Type -> Type)`. If we fix one of the two inputs of `Compose` we end up with types of kind `(Type -> Type) -> (Type -> Type)`, which is the kind signature of monad transformers. Types with this kind are easily composable, and this allows us to stack monad transformers one on top of the other to combine effects.

<br>

We can now observe that `Kleisli m`, for any monad `m`, maps the `(->)` category to the `Kleisli m` category. So in some sense it is transforming from one category to another. What if we now generalise the starting `(->)` category to a generic one and define

```haskell
type ProKleisli :: (Type -> Type) -> (Type -> Type -> Type) -> (Type -> Type -> Type)
data ProKleisli m p a b = ProKleisli {runProKleisli :: p a (m b)}
```

This way we obtain that `ProKleisli m` has kind `(Type -> Type -> Type) -> (Type -> Type -> Type)`. Similarly to what we had for monad transformers, we can now easily compose `ProKleisli m1` with `ProKleisly m2` and we can stack them one on top of the other to combine their effects.

## Is the Kleisli category of a Kleisli category still a category?

Now the natural question to ask is whether composing `ProKleisli m` categories form a category, of whether there are specific conditions when that always happens.

<br>

First, we can try to understand what conditions we need to impose on `p` so that `ProKleisli m p` is still a category. In other terms, we would like to implement this instance:

```haskell
instance (Monad m) => Category (ProKleisli m p) where
  id :: ProKleisli m p a a
  id = _

  (.) :: ProKleisli m p b c -> ProKleisli m p a b -> ProKleisli m p a c
  (.) = _
```

To implement `id`, we need a value of type `p a (m a)`. Given that `m` has a monad instance, we have a function `pure :: a -> m a`. If we are able to lift `pure` from `(->)` to `p` then we are done. One way to obtain this is to require `p` to have an instance of

```haskell
class Arr (p :: Type -> Type -> Type) where
  arr :: (a -> b) -> p a b
```

which allows us to lift functions to the `p` category[^1]. Hence, if we add a constraint `Arr p`, we can implement `id = ProKleisli $ arr pure`.

<br>

To implement `(.)`, we need to construct a value of type `p a (m c)` out of values of type `p a (m b)` and `p b (m c)`. Following the standard implementation of the `Category` instance for the `Kleisli` data type, we can first lift the latter value to `p (m b) (m (m c))`, then compose this with `p a (m b)` to obtain `p a (m (m c))` and then compose again with `arr join :: p (m (m c)) (m c)` to end up with `p a (m c)`. The constraint we need to add should hence allow us to lift a value `p a b` to `p (m a) (m b)`; if you have a closer look, this is a categorical generalisation of a `Functor`. We will define it as

```haskell
class CatFunctor f p q where
  catMap :: p a b -> q (f a) (f b)
```

meaning that `f` is a functor between the categories `p` and `q`.

<br>

At this point we can complete our `Category` instance as follows

```haskell
instance (Monad m, Category p, Arr p, CatFunctor m p p) => Category (ProKleisli m p) where
  id :: ProKleisli m p a a
  id = ProKleisli $ arr pure

  (.) :: ProKleisli m p b c -> ProKleisli m p a b -> ProKleisli m p a c
  (.) (ProKleisli g) (ProKleisli f) = ProKleisli $ arr join . catMap g . f
```

## Categorical functors

The `Arr p` constraint is not particularly stringent, since every `p` which admits a `Profunctor` or `Arrow` constraint then have one. The more interesting constraint we are imposing is the `CatFunctor m p p` one. Let's try to see what could be instances for it.

<br>

We can immediately observe that if `f` admits an instance `Functor f`, then we can also implement a `CatFunctor f (->) (->)`.

```haskell
instance (Functor f) => CatFunctor f (->) (->) where
  catMap :: (a -> b) -> f a -> f b
  catMap = fmap
```

<br>

For some of the mostly used monads like `Either`, `(,) w` and `(->) r`[^2] we just need to ask that the category respects sums, products and functions, respectively. This means just adding constraint for `Choice p`, `Strong p` or `Closed p`, respectively.

```haskell
instance (Choice p, Category p) => CatFunctor (Either e) p p where
  catMap :: p a b -> p (Either e a) (Either e b)
  catMap f = id +++ f

instance (Closed p) => CatFunctor ((->) r) p p where
  catMap :: p a b -> p (r -> a) (r -> b)
  catMap = closed

instance (Strong p) => CatFunctor ((,) w) p p where
  catMap :: p a b -> p (w, a) (w, b)
  catMap = second'
```

## ProKleisli is `Strong`, `Choice` and `Closed` if `p` is.

In the previous section we noticed that in many cases we can reduce the `CatFunctor` constraint to a constraint directly on `p` like `Choice`, `Strong` or `Closed`. Since we would like to use this machinery to compose multiple `ProKleisli`, let's take a look in what cases `ProKleisli m p` satisfies those constraint.

```haskell
instance (Choice p, Applicative m) => Choice (ProKleisli m p) where
  right' :: ProKleisli m p a b -> ProKleisli m p (Either c a) (Either c b)
  right' (ProKleisli p) = ProKleisli $ rmap sequenceA $ right' p

instance (Strong p, Applicative m) => Strong (ProKleisli m p) where
  second' :: ProKleisli m p a b -> ProKleisli m p (c, a) (c, b)
  second' (ProKleisli p) = ProKleisli $ rmap sequenceA $ second' p

instance (Closed p, Distributive m) => Closed (ProKleisli m p) where
  closed :: ProKleisli m p a b -> ProKleisli m p (x -> a) (x -> b)
  closed (ProKleisli p) = ProKleisli $ rmap distribute $ closed p
```

We can notice that for `Strong` and `Choice` we can use the `Traversable` instances of `Either` and `(,)`, while for `Closed` we need to use the `Distributive` instance of `m`.

Putting together the information of the last two sections, we can deduce that if we start with a `Strong` and `Choice` category like `(->)`, we can stack as many layers of `ProKleisli (Either e)` or `ProKleisli ((,) w)` and still end up with a category which is still `Strong` and `Choice`.

## Category transformers

At this point, let's see if we can adapt the [definition of monad transformers](https://hackage-content.haskell.org/package/transformers-0.6.3.0/docs/Control-Monad-Trans-Class.html#t:MonadTrans) to a categorical setting and define what a category transformer is.

```haskell
class CatTrans (t :: (Type -> Type -> Type) -> (Type -> Type -> Type)) where
  catLift :: (Category c, Arr c) => c a b -> (t c) a b
```

With such a definition we can define an instance for the `ProKleisli` data type

```haskell
instance (Applicative m) => CatTrans (ProKleisli m) where
  catLift :: (Category p, Arr p) => p a b -> ProKleisli m p a b
  catLift f = ProKleisli $ arr pure . f
```

## An example

To show that all this machinery actually works in practice, let's take a look at a concrete example. Consider this monad code written using monad transformers to combine effects

```haskell
isEnvGreaterThanInput :: MaybeT (ReaderT Int IO) Bool
isEnvGreaterThanInput = do
  -- read from environment
  env <- lift $ ask
  -- read from input
  input <- lift . lift $ getLine
  -- try to parse input to Int (might fail)
  n <- hoistMaybe $ readMaybe input
  -- compare environment with input
  pure $ env > n
```

It reads an `Int` from the environment, it reads a value from user input and tries to parse it to an `Int` and then compares the two integers. Nothing extremely fancy, but just a fairly standard script combining multiple effects.

<br>

The category transformer version of the same code is as follows

```haskell
isEnvGreaterThanInput' :: (ProKleisli Maybe (ProKleisli IO (ProKleisli ((->) Int) (->)))) () Bool
isEnvGreaterThanInput' =
  -- compare environment with input
  (arr $ uncurry (>))
    . (readFromEnvironment &&& readFromInput)
  where
    readFromEnvironment :: (ProKleisli Maybe (ProKleisli IO (ProKleisli ((->) Int) (->)))) a Int
    readFromEnvironment = catLift . catLift $ ProKleisli $ const ask -- we use const because we actually ignore the `a` input of the computation

    readFromInput :: (ProKleisli Maybe (ProKleisli IO (ProKleisli ((->) Int) (->)))) a Int
    readFromInput =
      -- try to parse input to Int
      (ProKleisli $ arr readMaybe)
        .
        -- read from input
        (catLift $ ProKleisli $ arr $ const getLine)
```

I personally find this version quite nice because it highlights the fact that the two reading operations can happen in parallel, while the comparing operation needs to come after in a sequential fashion.

## Pros and cons

To wrap up, I'd like to briefly discuss what could be the pros and cons of such an approach.

<br>

For sure the first thing which come to mind when looking at this is that the Haskell ecosystem is mostly based on monads, and working with categories instead creates friction with the usage of well established patterns and libraries. I guess there's no arguing against this, but we can still recognise that a category-based approach has it advantages.

<br>

The first one is that a category-based approach is much more symmetrical than a monad-based one. In fact, we could, for example, define a `ProCoKleisli` data type which produces categories out of `Comonad`s, allowing us to have a shared setup where we can actually naturally compose monads and comonads[^3].

<br>

The second one is, to me, the best selling point. As Chris Penner explained in his post [Exploring Arrows for sequencing effects](https://chrispenner.ca/posts/arrow-effects), categories allow for static analysis, while monads don't. This mean that using a category-based approach we could be able to inspect, describe, even print as a diagram the structure of our programs!

<br>

If you want to have a look at the details, I'm working on these ideas in https://codeberg.org/marcosh/category-transformers.

---

[^1]: I could have used [`Arrow`](https://hackage.haskell.org/package/base/docs/Control-Arrow.html#t:Arrow) or [`Profunctor`](https://hackage.haskell.org/package/profunctors/docs/Data-Profunctor.html#t:Profunctor), but I decided to introduce this new class to keep the requirements minimal and not lock myself in one of the two class hierarchies.

[^2]: `(,) w` is isomorphic to `Writer w`, while `(->) r` is isomorphic to `Reader r`.

[^3]: Open question: is there actually any other example of category transformer other than `ProKleisli` and `ProCoKleisli`?
