---
layout: post
title:  "Combining monads with natural transformations"
author: Marco Perone
date: 2025-03-10 08:06:42 +0200
categories: post
tags: haskell monad natural transformation functional
comments: true
pageUrl: '"http://marcosh.github.io/post/2025/03/10/combining-monads.html"'
pageIdentifier: '"Combining monads with natural transformations"'
description: "Combining monads with natural transformations"
image: "/img/natural-transformation-condition.png"
---

One of the aspects which sets Haskell apart from other programming languages is that we tend to handle our effects with care.
We are used to using monads to encode side effects; for example we use `Either e` to encode failures, `Reader r` to describe configuration and `State s` for stateful operations.

<br>

This introduces the issue of combining multiple effects needed in the same computation.
Many solutions to this problem were proposed through the years, from monad transformers, to `mtl`, to more advanced effect systems.

<br>

In this blog post I'd like to present an alternative approach which appears quite natural to me, but I was not able to find already implemented and used in the wild. As far as I can see, it hits a nice spot in the scale between simplicity and complexity and deserves to be explored further.

<br>

I'm very interested to receive feedback on these ideas and see what other people think of them.

<br>

## Natural transformations

<br>

Let's start from the main ingredient of our recipe and briefly introduce [natural transformations](https://en.wikipedia.org/wiki/Natural_transformation).

<br>

If we want to execute two operations `f :: m1 a` and `g :: m2 b` together in the same context, we first need to find a monad `m` where both `f` and `g` could live and have a meaning.

<br>

To do this, we want a mechanism to interpret operations defined in a context `m` into another context `n`. A natural transformation is nothing else but a way to do that in a uniform (i.e. _natural_) way.

<br>

Let's define a natural transformation with a typeclass[^2]

<br>

```haskell
class Embeds m n where
    embed :: forall (a :: Type) . m a -> n a
```

<br>

This means that for every type `a`, we have a function from `m a` to `n a`. In other terms, we have a way to transform every operation in the `m` context into an operation in the `n` context.

<br>

Now, if we come back to our `f :: m1 a` and `g :: m2 b` operations, we see that we can interpret both `f` and `g` in any context `m` which admits a natural transformation from `m1` and a natural transformation from `m2`.

<br>

```haskell
f&g :: (Monad m, Embeds m1 m, Embeds m2 m) => m b
f&g = do
  embed f
  embed g
```

<br>

This way, we can execute `f` and `g` together in the same operation `f&g`.

<br>

## Natural natural transformations

<br>

Being able to write `f&g` to combine two operations from different contexts is nice, but we're still missing a key ingredient: writing the instances for the `Embeds` class.

<br>

How hard is it? Do we really need to write an instance by hand every time we want to interpret an operation in a more general context?

<br>

Luckily the answer to the second question is often no, and writing just very few instances could take us a very long way[^3].

Let's start to think about what some obvious instances could be.

The most obvious one is probably

<br>

```haskell
-- | (1)
instance Embeds m m where
  embed :: m a -> m a
  embed = id
```

<br>

stating that you can embed any `m` into itself.

<br>

Another easy one is

<br>

```haskell
-- | (2)
instance (Applicative m) => Embeds Identity m where
  embed :: Identity a -> m a
  embed = pure . runIdentity
```

<br>

which states that, for any applicative functor `m`, it's possible to embed `Identity` into `m`.

<br>

The next two instances are a bit more complex.

<br>

```haskell
-- | (3)
instance (Monad m, MonadTrans t, Embeds n m) => Embeds n (t m) where
  embed :: n a -> (t m) a
  embed = lift . embed
```

<br>

This says that whenever we already have that `Embeds n m`, it is also true that `Embeds n (t m)`, if `m` is a monad and `t` is a monad transformer.

<br>

```haskell
-- | (4)
instance {-# OVERLAPPING #-} (Monad n, MFunctor t, Embeds n m) => Embeds (t n) (t m) where
  embed :: t n a -> t m a
  embed = hoist embed
```

<br>

The last one[^4] says that, whenever `Embeds n m`, we can deduce that `Embeds (t n) (t m)`, provided that `n` is a monad and `t` is an [MFunctor](https://hackage.haskell.org/package/mmorph-1.2.0/docs/Control-Monad-Morph.html#t:MFunctor).

<br>

## An explanatory example

<br>

To better understand why the above instances make sense, let's have a look at a concrete example.

<br>

Let's say we have a monad transformer stack `type Foo r s = ReaderT r (StateT s IO)`.

<br>

With the above instances, we can deduce `Embeds (Reader r) (Foo r s)`, `Embeds (State s) (Foo r s)` and `Embeds IO (Foo r s)`.

<br>

To jump to the motivation, this allows you to write code like

<br>

```haskell
foo :: Foo r s c
foo = do
  a <- embed (ioOperation :: IO a)
  b <- embed (readerOperation a :: Reader r b)
  embed (statefulOperation a b :: State s c)
```

<br>

and combine operations from different contexts in the same operation.

<br>

We can deduce `Embeds (Reader r) (Foo r s)` in two steps:

- both `Reader r = ReaderT r Identity` and `Foo r s = ReaderT r (StateT s IO)` are of the shape `t m`, hence we can use our last instance `(4)` and be left to deduce `Embeds Identity (StateT s IO)`;
- the second instance `(2)` we introduced in the previous paragraph is exactly what we need to conclude.

<br>

To deduce `Embeds (State s) (Foo r s)`, we can proceed as follows:

- the only instance that applies is `(3)`. To satisfy it we are left with `Embeds (State s) (StateT s IO)`;
- since both `State s = StateT s Identity` and `StateT s IO` are of the shape `t m`, we can use `(4)` and be left with `Embeds Identity IO`;
- now `(2)` allows us to conclude.

<br>

Lastly, we can deduce `Embeds IO (Foo r s)` as follows:

- the only instance that applies is `(3)`. Applying it, we are left with `Embeds IO (StateT s IO)`;
- again, the only instance that applies is `(3)`. To use it, we need an instance of `Embeds IO IO`;
- that is granted by our first instance `(1)`.

<br>

## Relation with `mtl`

<br>

A natural question which might arise, looking at such an approach, is how it compares with `mtl`.

For example, how does `Embeds (Either e) m` relate to `MonadError e m`? Or `Embeds (Reader r) m` with `MonadReader r m`?

It's easy to show that one implication holds: for example, we could have

<br>

```haskell
instance (MonadError e m) => Embeds (Either e) m where
  embed :: Either e a -> m a
  embed (Left e) = throwError e
  embed (Right a) = pure a
```

<br>

and

<br>

```haskell
instance (MonadReader r m) => Embeds (Reader r) m where
  embed :: Reader r a -> m a
  embed (ReaderT f) = reader (runIdentity . f)
```

<br>

At the moment of writing, I have no clear idea whether the other implication (i.e. whether, for example, `Embeds (Either e) m` implies `MonadError e m`) should hold, making the two approaches equivalent.

Possibly, an equivalence might hold, if we restrict the type of natural transformations allowed in the `Embeds` typeclass. One possibility is to consider only [monomorphic](https://en.wikipedia.org/wiki/Monomorphism) natural transformations.

<br>

In any case, the `Embeds` approach does not suffer from the [`n^2` issue](https://book.realworldhaskell.org/read/monad-transformers.html#id659760), which affects `mtl`.

If you need to introduce a new effect, you need to define a monad `m` and a monad transformer `t` such that `m` is isomorphic to `t Identity` and implement the `MonadTrans t` and `Mfunctor t` instances.

At that point, the above-defined instances `(1)` to `(4)` will do all the hard work.

<br>

## Hiding natural transformations

<br>

Being able to combine functions living in different contexts using `Embeds` could already be a nice cake, but, if we like it, we can add a cherry on top.

<br>

First thing, using `Embeds` we can define a more general version of `>>=`

<br>

```haskell
(>>=) :: (Monad m, Embeds n1 m, Embeds n2 m) => n1 a -> (a -> n2 b) -> m b
(>>=) n1a an2b = embed n1a Prelude.>>= (embed . an2b)
```

<br>

This custom `(>>=)` allows us to sequence computations which live in different monads, using the `Embeds` machinery.

<br>

At this point, we can add just a tiny bit of black magic, in the form of the [RebindableSyntax](https://ghc.gitlab.haskell.org/ghc/doc/users_guide/exts/rebindable_syntax.html#extension-RebindableSyntax) GHC extension.
It allows us, among other things, to interpret `do` notation with the `>>=` operation which is in scope and not the one from `Prelude`.

<br>

This allows the following code to compile correctly, if our custom `>>=`[^5] is in scope:

<br>

```haskell
{-# LANGUAGE RebindableSyntax #-}

ioOperation :: IO ()

statefulOperation :: State Int ()

combineTheTwo :: StateT Int IO ()
combineTheTwo = do
  ioOperation
  statefulOperation
```

<br>

This is probably going a bit too far from usual Haskell, since it hides many things under a lot of magic. But if you like it, feel free to use it!

<br>

## Conclusion

<br>

In Italy, we have a saying that goes like "Not everything that shines is gold". Even if, from the short presentation given in this post, this approach might seem nice, there could be some dark corners.
One aspect is that using an approach like this where the types are less strict, you lose type inference, and hence you need to provide more type annotations to help the compiler.
Then, there's probably other subtle issues I didn't encounter yet. If you notice them, please let me know.

This is not to say that this approach is flawed, but just that it comes with its pros and cons, and it needs to be evaluated carefully.

<br>

The last thing I need to say trying to wrap this up is that all of this is really in alpha state, and more work needs to be done to prove it's actually something useful to have.

<br>

If you want to discuss further or contribute, I'm using [https://github.com/marcosh/embeds](https://github.com/marcosh/embeds) as a repository for all the code related to these ideas.
Feel free to open issues there or to contact me directly.

<br>

---

[^1]: Restrictions apply. I'm not saying that you can combine any two monads. Read further to get a clearer picture.

[^2]: The `forall (a :: Type) . ` part is necessary. Omitting it would lead with issue later on. Check [here](https://discourse.haskell.org/t/issue-with-overlapping-instances/11467/2?u=marcosh) for more details.

[^3]: To make everything work, there's actually the need to have more instances than the one described here. Have a look [here](https://github.com/marcosh/embeds/blob/main/src/Embeds/NatTrans.hs) to check the details.

[^4]: The `{-# OVERLAPPING #-}` bit is necessary to say that this instance takes the precedence with respect to the previous one, during instance resolution.

[^5]: Actually, for this specific example, it should be `(>>)`, not `(>>=)`.
