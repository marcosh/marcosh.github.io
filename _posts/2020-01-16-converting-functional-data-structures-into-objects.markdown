---
layout: post
title:  "Converting functional data structures into objects"
author: Marco Perone
date: 2020-01-16 08:06:42 +0200
categories: post
tags: functional-programming object-oriented
comments: true
pageUrl: '"http://marcosh.github.io/post/2020/01/16/converting-functional-data-structures-into-objects.html"'
pageIdentifier: '"Converting functional data structures into objects"'
description: "Converting functional data structures into objects"
image: "/img/small-objects.png"
---

Since I discovered functional programming, I have been interested in how to port some of its ideas to object oriented programming.
In particular my attention deals with a safe implementation of tipical functional data structures. I experimented with it several times,
particularly with [`Maybe`](https://hackage.haskell.org/package/base-4.12.0.0/docs/Prelude.html#t:Maybe) and [`Validation`](https://hackage.haskell.org/package/either-5.0.1.1/docs/Data-Either-Validation.html#t:Validation)
and I think that in the end what came out reflects quite well the interface available in the functional world. If you want you can check my implementation of [`Maybe`](https://github.com/marcosh/maybe-php/blob/master/src/Maybe.php) and of [`Validation`](https://github.com/marcosh/php-validation-dsl/blob/master/src/Result/ValidationResult.php) in PHP.

It took me quite a while to arrive to that approach (you can check how it evolved in my posts [Maybe in PHP]({% post_url 2017-06-15-maybe-in-php %}) and [Maybe in PHP 2 - The revenge of sum types]({% post_url 2017-10-17-maybe-in-php-2 %})), but lately I just realised that there is actually a completely mechanical way to translate a functional data structure into a class in an object oriented language.

## Constructors, combinators and eliminators

When you define a class in an object oriented language there are some methods which play a special role; I'm referring to constructors, which are necessarily called when you need to create a new instance of the class.

In functional programming there is no such distinction, no function has a special role, since every function `a -> b` can be seen as a transformation from a data type `a` to a data type `b`, in a completely symmetrical fashion.
Still, if we focus our attention on a specific data type `a`, we can try to classify functions involving `a` discriminating on the position where `a` appears. We can identify the three following families:

- **constructors**: these are functions where `a` appears only as a result, so any function of the form `b -> a`.
- **eliminators**: these are the duals on constructors and are functions where `a` appears only as an argument and not as a result.
- **combinators**: these are all other functions, i.e. all the functions where `a` appears both as an argument and as a result.

Let's try to make some examples using [`Bool`](https://hackage.haskell.org/package/base-4.12.0.0/docs/Prelude.html#t:Bool), which is a very simple data type.
If we look at the documentation and search for constructors, i.e. functions where `Bool` appears only as a result and not as an argument, we find only `False` and `True`, which have type `Bool` and can be thought as functions with no arguments returning a `Bool`.

Eliminators of the `Bool` data type are functions where `Bool` does not appear in the return type; for example we have

```haskell
fromEnum :: Bool -> Int
compare  :: Bool -> Bool -> Ordering
show     :: Bool -> String
```

We need to remember also that the most common way to eliminate a data structure is by pattern matching. For `Bool` this corresponds to a function

```haskell
evalBool :: a -> a -> Bool -> a
```

where the first `a` is returned in the `True` case and the second in the `False` case.

Combinators are what is left, that is functions where `Bool` appears both as an argument and as a result. For example

```haskell
(&&) :: Bool -> Bool -> Bool
(||) :: Bool -> Bool -> Bool
not  :: Bool -> Bool
```

## Converting to a class

Now we would like to create a class `Bool` in our favourite object oriented programming language which behaves exactly as the `Bool` data structure above.

So we start declaring our new class (I'm using pseudo-code, so don't try to parse this code in a specific language).

```java
final class Bool
{
  ...
}
```

We need to declare it as `final` because we don't want anyone to add new constructors to `Bool`.

Then we can add our two constructors `True` and `False`.

```java
public static True() : self

public static False() : self
```

These are just named constructors, i.e. static methods which return an instance of the class itself. These two methods would probably call to a private constructor of the class which, being private, is not exposed for public usage.

Combinators and eliminators are just a little bit more tricky. This is due to the fact that when defining a method in object oriented programming we already have an instance of the class at our hands, usually called `this`. This means that when we translate a combinator or an eliminator for a type `a`, we need to choose one of the parameters of type `a` to be treated as `this`. If there is only one `a` parameter, there is no choice to be done, but if there is more than one, one of them receives a special treatment.

Let's start by converting the `evalBool` eliminator.

```java
public evalBool(a ifTrue, a ifFalse) : a
```

We receive two input parameters of the same type `a` and we return a value of type `a`, which will actually be either `ifTrue` or `ifFalse` depending on the internal state of `this`.

Let's see now how we could translate `compare`, which has two input parameters of type `Bool`.

```java
public compare(self that) : Ordering
{
  return this.evalBool(
    that.evalBool(Ordering::EQ(), Ordering::GT()),
    that.evalBool(Ordering::LT(), Ordering::EQ())
  )
}
```

We see how the two arguments of the functional version of `compare` are now treated differently; one becomes `this` and the other instead is the input parameter `that`.

To provide an example of a combinator, let's try to implement `not`.

```java
public not() : self
{
  return this.evalBool(self::False(), self::True())
}
```

The distinguishing trait between an eliminator and a combinator is that the latter returns an instance of the same class.

As you probably noticed, we used `evalBool` to implement both `compare` and `not`. It is actually true that we can use `evalBool` to implement any other combinator and eliminator on `Bool`.

If you want to check a working version of this process of porting `Bool` into an object oriented language, in a interface-preserving way, you can take a look at my [PHP implementation](https://gist.github.com/marcosh/b43063e31bb5d8dbd6019be36ea4b3c1).

## Conclusion

Porting functional data structures as defined in Haskell, Purescript or Elm to an object oriented language, preserving the same guarantees we have in the functional world is actually easy and could be done in a mechanical process.

Such an approach should allow to port functional data structures into the object oriented world in a straightforward way. This could help many developers which are familiar just with an object oriented approach to get in touch with the functional way of doing things, without renouncing to their favourite programming language and development tools.
