---
layout: post
title:  "Higher kinded types in PHP - the issue"
author: Marco Perone
date: 2020-04-15 08:06:42 +0200
categories: post
tags: functional-programming object-oriented php types
comments: true
pageUrl: '"http://marcosh.github.io/post/2020/04/15/higher-kinded-types-php-issue.html"'
pageIdentifier: '"Higher kinded types in PHP - the issue"'
description: "Higher kinded types in PHP - the issue"
image: "/img/higher.jpg"
---

Higher kinded types are one of those features which allows to write code in a very different way, to create and manage abstractions which are not possible otherwise. Few languages, such as Haskell, Scala and Purescript, feature them natively, but luckily it is possible to emulate them also in other languages. Still, to do that you need to have at hand a type system which comprises type variables (usually called generics in the object oriented realm).

For example, it is not possible to emulate higher kinded types in Javascript, because the type system is too basic; it is possible though to implement them in Typescript, as it is done in the library [fp-ts](https://github.com/gcanti/fp-ts) by [Giulio Canti](https://twitter.com/GiulioCanti). Similarly it is not possible to implement them with the type system provided by PHP itself; it is possible though to do it using a library which works as a type checker. We're going to do it using [Psalm](https://psalm.dev/), a static analysis tools which works using annotations.

Adding higher kinded types to a type system which does not provide them natively could be achieved following the strategy described in the paper [Lightweight higher kinded polymorphism](https://www.cl.cam.ac.uk/~jdy22/papers/lightweight-higher-kinded-polymorphism.pdf) by Jeremy Yallop and Leo White. That's exactly what we will do. But first, it's probably better to get a better understanding of what higher kinded types actually are.

## VALUES AND TYPES

Before defining exactly what higher kinded types are, let's review some other preliminary concepts and fix some terminology.

> A **type** is an attribute of data which tells the compiler or interpreter how the programmer intends to use the data
>
> [(Wikipedia](https://en.wikipedia.org/wiki/Data_type))

So we have data, which could be any possible value like `42`, `-273.15`, `"foobar"`, `new DateTimeImmutable()`, ... and we have types, like `int`, `float`, `string` and `DateTimeImmutable` which classify the values. Being PHP a dynamic language, the interpreter does not necessarily complains if we pass an integer where a string is expected, and it tries to cast it to the desired type whenever need. This though does not conflict with the fact that every value has its own type.

In a language like PHP we could use native types like `int`, `bool` and `string` or we could define our own types by creating `classes` and `interfaces` which represent more carefully the concepts we are working with in our applications and libraries. These will all be treated as types by the PHP interpreter and by Psalm, which is doing the work of a type checker.

## KINDS

Psalm allows us to define [generic parameters](https://psalm.dev/docs/annotating_code/templated_annotations/) for values inside our compound types. This allows us to consider and to reason, at the type level, about the content of our container types (think for example to homogeneous arrays and collection classes).

For example we could define a `Stack` interface which contains only elements of a generic type `A`

```
/**
 * @template A
 */
interface Stack
{
  /**
   * @param A $a
   */
  public function push($a): void;

  /**
   * @return A
   */
  public function pop();
}
```

In this way we can distinguish at the type level between a `Stack<int>` which contains integers and a `Stack<string>` which contains strings.

Now we notice that `Stack` itself is not exactly a type, since it needs a type parameter to be a fully specified type. We can refer to `Stack` as a type constructor, in the same fashion as we have constructors for our values.

`Higher kinded types` are simply those types which have type variables which are left generic. So, in our example, `Stack` is an higher kinded type while `Stack<int>` and `Stack<string>` are not.

We can think about higher kinded types as functions at the type level, which receive some types as inputs and produce some other type as output. From this point of view, `Stack` can be seen as a function at the type level which receives a type `A` a produces a type `Stack<A>`.

More formally, we can define the [kind](https://en.wikipedia.org/wiki/Kind_(type_theory)) for any possible type of our language. Types which do not accept type parameters will have kind `*`. Types which accept a single type parameter have kind `* -> *` (the notation suggests that they actually are functions from types of kind `*` to types of kind `*`). Types which have two type parameters will have kind `* -> * -> *`, and so on.

In our example, `int` and `Stack<int>` have kind `*`, while `Stack` has kind `* -> *`.

It seems then that generics allows us to define higher kinded types. So what's all the fuss about this? Actually, when we say that a language has higher kinded types, we mean that it is possible to abstract on them!

## HIGHER KINDED TYPES

What do we mean by abstracting on higher kinded types? We just mean that we want to be able to treat them as type parameters as any other type.

The issue with Psalm type system (or Java, C# and many other type systems with type variables) is that a type variable represents any possible concrete type, i.e. a type of kind `*`. So in `Stack<A>`, the type variable `A` represents any possible type available in the types system. What if on the other hand we wanted to abstract the type constructor and consider something like `F<int>`, where the parameter is specified to `int` but the type constructor `F` is abstract? That is not possible because `F` would be treated as a generic type variable, where any possible type could go; for example `string<int>` is not something which makes sense, because `string` is not a type constructor. In other terms, `F` would be treated by the type checker as something of kind `*`, while it must be something of kind `* -> *` since it has a single type parameter.

So what we need is a way to abstract just type constructors and not all types. We need to be able to tell to our type checker that we are dealing with a type of a specific kind. We basically need to teach to our type checker the concept of kind, so that it could use it to distinguish types of different kind.

## THE SOLUTION

It seems odd that we could actually teach a type checker a concept it doesn't know about. But, as we will see in our [next post]({% post_url 2020-04-15-higher-kinded-types-php-solution %}) this is actually possible and allows us to introduce in our applications a whole new level of abstractions which weren't possible before.
