---
layout: post
title:  "Maybe in PHP 2 - The revenge of sum types"
author: Marco Perone
date:   2017-10-27 10:28:42 +0200
categories: post
tags: functional-programming php maybe types
comments: true
pageUrl: '"http://marcosh.github.io/post/2017/10/27/maybe-in-php-2.html"'
pageIdentifier: '"maybe-in-php-2"'
description: "How to model sum types in PHP"
image: "/img/maybe.jpg"
---

In [Maybe in PHP]({% post_url 2017-06-15-maybe-in-php %}) I already talked about several
libraries which implement a `Maybe` data structure in PHP, discussing their pros and cons,
and I concluded the article proposing an alternative implementation.

None of those solutions (including mine) were completely satisfying so I started to believe
that in an object oriented language as PHP it is non possible to flawlessly model a sum
type as `Maybe`.

In this post I would like to review the problem, consider its requirements a little more
deeply and eventually show what I believe to be an elegant solution to the matter.

## REVIEW THE PROBLEM

In strongly typed languages as [Haskell](https://www.haskell.org/) or [Elm](http://elm-lang.org/),
the possibility of a value not to be there is addressed at the type level using `Maybe`, which is
defined as follows:

```
Maybe a = Just a | Nothing
```

This means that a value of type `Maybe a` could be either `Just a`, that is a wrapper around a value
of type `a`, or `Nothing`, which denotes the absence of a value.

Then you can use a value of type `Maybe a` with pattern matching, as follows

```
case foo of
    Just value -> // here you have access to the value
    Nothing -> // handle the case where you don't have a value
```

As I stated in my previous post, this approach has some nice characteristics:

- a value of type `Maybe a` can be either `Just x`, for some `x` of type `a`, or `Nothing`.
There is no other possibility.
- when you need to process something of type `Maybe a`, you must consider both the `Just`
and the `Nothing` cases. The compiler will not forgive you if you are lazy and forget to
handle one of them.
- when you pattern match on a value of type `Maybe a` you have access to the wrapped value
only in the `Just` case; in the `Nothing` case you just don't have any value to consider.

The problem now is to model this behaviour in an object oriented language as PHP, where sum
types are not a natural idiom.

## A CLOSER LOOK

Now that we have defined what we would like to achieve, let's investigate a little bit deeper
what `Maybe a = Just a | Nothing` means.

Starting from the easy pieces, `a` is a type, any type, and `Maybe a` is another type which
depends on `a`. If you like, you can think at `Maybe` as function with signature `Type -> Type`,
that is, you give it a type and you get back another type.

Another way to look at `Maybe a` is to think about it as a generic type, depending on a parameter
of type `a`. Unfortunately we don't have generics in PHP, even if there was a
[proposal to add them](https://wiki.php.net/rfc/generics), somebody proposed a
[proof of concept](https://github.com/ircmaxell/PhpGenerics) for them, and actually
[Hack has them](https://docs.hhvm.com/hack/generics/introduction). Until we get them as a
language feature, we will just be able to consider a generic `Maybe` type, which can contain
anything, or otherwise we would need a specific implementation for each specific type we would
like to be contained in our `Maybe` type.

The less easy pieces are instead `Just a` and `Nothing`. What are they exactly? Are they subtypes
of `Maybe`? Or what else? If you want, try to think about it for a minute before going on reading.

Looking back at all the implementations listed in my [previous post]({% post_url 2017-06-15-maybe-in-php %}),
we can see that they are always (except in the [monad-php](https://github.com/ircmaxell/monad-php) case)
implemented as subclasses of `Maybe`. In my opinion, this is because inheritance is the standard
mechanism in object oriented languages to express that something is a specification of something else.
But is it correct to implement `Just` and `Nothing` as subtypes of `Maybe`? Or is our object oriented
mindset leading us on the wrong path?

Since we're trying to implement a concept from the functional programming world, we should
probably first try to understand what `Just` and `Nothing` are in that environment.

First off, we immediately exclude the possibility that `Just` and `Nothing` are subtypes of `Maybe`.
This is due to the fact that simply `Just a` and `Nothing` are not types! If we try to annotate a variable
with such a type, the compiler will yell at us!

So what are they? The answer is not difficult and, as almost anything in functional programming, they are
just functions! `Just` is a function that takes a parameter of type `a` and returns a value of type `Maybe a`,
while `Nothing` is a constant function, which has no input parameters and always returns the value `Nothing`
of type `Maybe a`. In this light it becomes clear that `Just` and `Nothing` are nothing else but type
constructors.

## PHP IMPLEMENTATION

Let's now think about how we could translate the previous paragraph in PHP code. First off, `Maybe` is a type,
so in PHP we will have an interface or a class. For the sake of simplicity we will build a concrete class

```
final class Maybe
{
    ...
}
```

As we saw, `Just` and `Nothing` are type constructors for `Maybe`. Concretely, this means that they are just
functions which return value is of type Maybe. We could implement them in PHP as any form of callable; we
choose static methods of the `Maybe` class to use them as
[named constructors](http://verraes.net/2014/06/named-constructors-in-php/).

```
final class Maybe
{
    private function __construct(...)
    {
        ...
    }

    /**
     * @param mixed $value
     */
    public static function just($value): self
    {
        ...
    }

    public static function nothing(): self
    {
        ...
    }
}
```

I think it's worth underlining again that there is nothing wrong for `just` and `nothing` not being classes
or interfaces. On the contrary, I think it is better if they are not classes, so that a client can not be
tempted to use them directly instead of using `Maybe`.

Let's now come to the most complicated part, which is pattern matching. This is what you do when you have
something of type `Maybe` and you split the two cases, `Just` and `Nothing`, to handle them separately, as I
mentioned in the first section. We want to oblige the client of our class to handle both cases and to be
able to access the wrapped value only in the `Just` case. We could do this as follows, with a simple `match`
method

```
final class Maybe
{
    /**
     * @var bool
     */
    private $isJust;

    /**
     * @var mixed|null
     */
    private $value;

    ...

    /**
     * @param callable $justHandler handles the Just case, with access to the wrapped value
     * @param callable $nothingHandler handles the Nothing case
     * @return mixed
     */
    public function match(
        callable $justHandler,
        callable $nothingHandler
    ) {
        return $this->isJust ? $justHandler($this->value) : $nothingHandler();
    }
}
```

where `$isJust` and `$value` are implementation details which are completely invisible externally.

You can find the complete implementation of my attempt to a solution
[here](https://github.com/marcosh/php-sum-types/blob/master/src/Maybe.php).

## CONCLUSION

To conclude, let's review if this implementation satisfies the three properties I stated at the beginning:

- **a value of type `Maybe a` can be either `Just x`, for some `x` of type `a`, or `Nothing`**. In fact, since
our `Maybe` class is final and `just` and `nothing` are its only constructors, there is no other way to
create an instance of `Maybe`. Check.

- **when you need to process something of type `Maybe a`, you must consider both the `Just`
and the `Nothing` cases**. Since our `match` function has two required arguments, the client is obliged to
provide the handlers for both cases. Check.

- **when you pattern match on a value of type `Maybe a` you have access to the wrapped value
only in the `Just` case**. The value wrapped by `Maybe` is stored in a private variable `$value` which is
passed exclusively to the `$justHandler` callable. Check.

There is a fourth property, suggested by [Mathias Verraes](https://twitter.com/mathiasverraes) in [his
take](https://gist.github.com/mathiasverraes/b54c2c32fb66f4c6f739e8dff128a4f0) of this exact problem:

- **a `Maybe` is never both `Just` and `Nothing` at the same time**. Depending on the named constructor the client
uses to build a `Maybe`, it will produce either a `Just` or a `Nothing`, never both. Check.

If I'm not missing some details, this implementation satisfies the properties which describe a sum type.
The main difficulty I had in reaching this implementation was understanding that I didn't need
inheritance; I am becoming more and more convinced that inheritance as a modelling tool is
more limited than a type system as the one provided for example by Haskell (Elm's one is still very
nice, but doesn't have an abstraction mechanism as interfaces or type classes).

The downside of this implementation is that it is, as far as I can see, hardly generalizable. Having
a `match` method with the same number of arguments as the number of type constructors makes it
impossible to define a `SumType` interface comprising the `match` method. Moreover, it becomes
very verbose to define a new custom sum types, defining all the name constructors and the `match`
method.

To address this last point, there is a way, but I want to leave it for a future post...
