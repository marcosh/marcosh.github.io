---
layout: post
title:  "Higher kinded types in PHP - the solution"
author: Marco Perone
date: 2020-04-15 09:06:42 +0200
categories: post
tags: functional-programming object-oriented php types
comments: true
pageUrl: '"http://marcosh.github.io/post/2020/04/15/higher-kinded-types-php-solution.html"'
pageIdentifier: '"Higher kinded types in PHP - the solution"'
description: "Higher kinded types in PHP - the solution"
image: "/img/higher.jpg"
---

In our [previous post]({% post_url 2020-04-15-higher-kinded-types-php-issue %}) we reviewed the concepts of value, type and kind and we saw how abstracting on types of a specific kind is an issue when working with a type system like the one provided by [Psalm](https://psalm.dev/) (or generally a type system which does not track kinds, like the ones of Typescript, Java, C#, ...). In this post we will see how we can actually bypass this limitation and implement higher kinded types in PHP using Psalm as a type checker.

We will follow the approach presented in the paper [Lightweight higher kinded polymorphism](https://www.cl.cam.ac.uk/~jdy22/papers/lightweight-higher-kinded-polymorphism.pdf) by Jeremy Yallop and Leo White.

## HIGHER KINDED TYPES IN ACTION: FUNCTORS

We will show how to implement higher kinded types using one example, namely [functors](https://en.wikipedia.org/wiki/Functor). You don't need to know precisely what they are or why they are actually really useful. It's enough to know that a type constructor defines a functor if we are able to lift a function between any two types `A` and `B` to the level of our type constructor. More explicitly, let `F<T>` be a type constructor with a type parameter `T` and `$f` be a function between any two types `A` and `B`; then `F` is a functor if we can lift `$f` to a function between `F<A>` and `F<B>`.

If we want to express this in code, it would be something like

```php
/**
 * @template A
 */
interface Functor
{
    /**
     * @template B
     * @psalm-param callable(A): B $f
     * @return Functor
     */
    public function map(callable $f): Functor;
}
```

This implementation could work but is still unsatisfactory. The main issue is that we are not able to specify correctly the return type of `map`, which should be `F<B>` and not a generic `Functor`. The issue is even worse is we had another input of type `F<_>`, because contravariance of inputs would really not allow to to add such a method to the interface.

To solve this we need to express clearly that we are dealing with our type constructor `F` both in the inputs and the outputs of the methods of the interface.

## HK AND BRANDS

The main trick is to define a new interface `HK` which allows us to state explicitly that a class is an higher kinded type. The interface has no methods and is used just to keep track of information at the type level.

```php
/**
 * @template G
 * @template A
 */
interface HK
{
}
```

The type parameter `A` will indicate the type which our higher kinded type will be parametrised on. The news is the type parameter `G`, which will be used to pass around information at the type level about which type constructor we are dealing with. Basically, instead of writing `G<A>`, we will write `HK<G, A>`. This is useful because in the expression `HK<G, A>`, both `G` and `A` have kind `*`, so we can deal with them with the type system we currently have at our disposal.

To be more precise, we need one more ingredient. In the place of the template `G` we can not use directly our type constructor `F`, because the former has kind `*` and the latter has kind `* -> *`. We need something else which we will introduce ad hoc. For each type constructor `G` which we want to treat as an higher kinded type, we define a new class, which we call `Brand`, which is used just to pass around information at the type level.

```php
final class GBrand
{
}
```

So we will write `HK<GBrand, A>` every time we want to actually say `G<A>`.

## MAYBE A FUNCTOR

I guess that a complete example with some code could be helpful here to better grasp what is going on.

First we need to rewrite our `Functor` interface using our newly crafted `HK` interface. What we get is

```php
/**
 * @template G
 * @template A
 * @extends HK<G, A>
 */
interface Functor extends HK
{
    /**
     * @template B
     * @psalm-param callable(A): B $f
     * @psalm-return HK<G, B>
     *
     */
    public function map(
        callable $f
    ): HK;
}
```

We say that `Functor<G, A>` extends `HK<G, A>` to express the fact that the classes implementing the `Functor` interface will need to be higher kinded types. Moreover, the return type of the `map` function is now `HK<G, B>`, which expresses the fact that what we return is using exactly the same type constructor represented by the brand `G`.

Next we can proceed to implement an actual functor. We choose our beloved [`Maybe`]({% post_url 2017-10-17-maybe-in-php-2 %}), which is a functor which allows us to represent in a pure way the absence of a value, solving the many issues arising from the usage of `null`s.

```php
final class MaybeBrand
{
}

/**
 * @template A
 * @implements Functor<MaybeBrand, A>
 */
final class Maybe implements Functor
{
  /**
     * @template B
     * @psalm-param callable(A): B $f
     * @psalm-return HK<MaybeBrand, B>
     */
    public function map(callable $f): HK
    { ... }
}
```

We need to introduce first our brand `MaybeBrand` which will allow us to tag the `HK` interface for the actual `Maybe` class. In fact, in the implementation of `Maybe` we have `Maybe<A> implements Functor<MaybeBrand, A>`. This allows us to express correctly the return type of the `map` function as `HK<MaybeBrand, B>`. This tells us that the actual type parameter changed from `A` to `B`, according to the types `A` and `B` of the callable we actually passed as a parameter of `map`. Moreover, it informs us that we are still considering an higher kinded type with respect to the `MaybeBrand` tag.

Notice that there is still a possibility to get things wrong here. One could define another class `OtherMaybe<A>` which implements `HK<MaybeBrand, A>` and this would allow the implementation of `Maybe::map` to actually return an instance of `OtherMaybe`, which is something we would like to avoid. This issue should be documented carefully and taken care of during code reviews.

Still, this allows the usage of higher kinded types in a language like PHP and this opens up the door to the implementation of a lot of interesting patterns.

## CONCLUSION

Higher kinded types are a great technique which allow to create a whole new level of abstractions. They allow to reduce the duplication of code and exploit even more the advantages provided by parametric polymorphism. The ideas presented in [Lightweight higher kinded polymorphism](https://www.cl.cam.ac.uk/~jdy22/papers/lightweight-higher-kinded-polymorphism.pdf) enable us to mimic higher kinded types in a type system which does not allow them natively. Using [Psalm](https://psalm.dev/) as a type checker we were able to bring them to PHP.

I'm pretty excited to have them at hand now and being able to experiment with them. I hope I'll be able to post pretty soon another article showing how higher kinded types allow to encode useful abstraction and enable us to structure programs in a more abstract and generic way.
