---
layout: post
title:  "Type equality in object oriented programming"
author: Marco Perone
date: 2020-08-21 08:06:42 +0200
categories: post
tags: monoid types object-oriented
comments: true
pageUrl: '"http://marcosh.github.io/post/2020/08/21/type-equality-in-object-oriented-programming.html"'
pageIdentifier: '"Type equality in object oriented programming"'
description: "Type equality in object oriented programming"
image: "/img/monoid.png"
---

Object orientation is a programming paradigm which I consider to be good enough for most things. There are some case, though, were it makes things harder than what they should actually be. One of the issues which always bothered me is the lack of an abstract way of expressing being the _same thing_ at the type level. There are cases where generics save the day, but sometimes they are not enough. I'd like to illustrate this providing a concrete use case with several examples.

I'll be using PHP syntax, because that's what I'm most comfortable with, but the ideas presented here are applicable to any object oriented language.

## Abstracting addition, multiplication and concatenation

Let's consider a class

```php
class Int
{
    public function add(Int $that): Int
}
```

which represents integer numbers (the same applies to any other set of numbers) and provides a method to add two integers to return a new integer. Similarly, we could do the same for multiplication:

```php
class Int
{
    public function multiply(Int $that): Int
}
```

Consider now a `List` class which allows you to concatenate two `List`s to obtain a new `List`

```php
/**
 * @template A
 */
class List
{
    /**
     * @param List<A> $that
     * @return List<A>
     */
    public function append(List $that): List
}
```

This starts to be a recurring pattern. Let's provide a last, a bit less obvious, example, before drawing a conclusion.

Consider a class which describes an operation which can transform a value of a certain type in another value of the same type

```php
/**
 * @template A
 */
class Endo
{
    /**
     * @param Endo<A> $that
     * @return Endo<A>
     */
    public function compose(Endo $that): Endo
}
```

We can endow it with a `compose` method which receives as input another `Endo` and returns an `Endo` which performs the two operations in sequence; first the transformation described by the `$this` instance, and then applies to the result the transformation described by `$that`.

Now we have four examples, and we could present more, of methods which, on a class of type `T`, require a parameter of type `T` and return a result of type `T`. The question now emerges naturally: can we abstract such a behaviour?

## Attempts at writing an interface

In object oriented programming, as soon as we'd like to abstract something, we immediately reach out to an `interface`. So let's try to write one to describe the behaviour we presented in the previous section

```php
interface Semigroup
{
    public function append($that);
}
```

This could work but we can't be satisfied with it. That's because we are not typing either the parameter or the return value; this means that what we have at the moment is a leaky abstraction, because it allows instances which are outside our desired scope.

Hence, we need to provide types in the `Semigroup::append` method both to the parameter and the return value, to express that they should be of the _same type_ of the instance of `Semigroup` which is considered. Let's try some options.

First thing we could try is using the `Semigroup` interface itself

```php
interface Semigroup
{
    public function append(Semigroup $that): Semigroup;
}
```

This is better than having no types at all, but still not good enough. With this typing, using `append` on an `Int`, passing a `List` as parameter and returning an `Endo` would be valid, but it doesn't make too much sense considering how we are seeing our abstraction (combining two things of the _same type_ to produce another thing of the _same type_).

What other options do we have? We could try using `self`

```php
interface Semigroup
{
    public function append(self $that): self;
}
```

but that won't change much, since `self` references the class or interface where it appears, which is `Semigroup` in this case. Therefore this signature is absolutely equivalent to our previous one.

Similar to `self`, one could try to use `static`, which [refers the class which was initially called at runtime](https://www.php.net/manual/en/language.oop5.late-static-bindings.php)

```php
interface Semigroup
{
    public function append(static $that): static;
}
```

but that doesn't even compile. `static` will work as a return type from [PHP8](https://wiki.php.net/rfc/static_return_type), but it is not allowed as a parameter type because it would break the Liskov substitution principle.

At this point, I must say, I am out of options... is there really no way to abstract the concept of being the _same type_? That seem like something you'd like to be able to do, but it looks like object orientation does not play in our favour here.

## Maybe I have an idea...

Maybe not all hope is lost and we should not give up just yet. Maybe we could perform some type level trickery to obtain what we want. We are in luck that we know already about [brands and their usage to implement higher kinded types]({% post_url 2020-04-15-higher-kinded-types-php-solution %}). Maybe we could try to use something of the like to solve our issue. Let's see.

The best solution we have up to now is to use `Semigroup` as type hint. We could try to improve on it. What if we introduce a type parameter to the `Semigroup` interface to denote on which type it should work?

```php
/**
 * @template B
 */
interface Semigroup
{
    /**
     * @param Semigroup<B> $that
     * @return Semigroup<B>
     */
    public function append(Semigroup $that): Semigroup;
}
```

This still encodes the fact that we are combining two things of the _same type_ to produce another thing of the _same type_, but allows us to distinguish between different `Semigroup`s using the type variable.

At this point, we can implement the fact that `List<A>` is a `Semigruop` as follows

```php
/**
 * @template A
 */
class ListBrand
{}

/**
 * @template A
 * @implements Semigroup<ListBrand<A>>
 */
class List implements Semigroup
{
    /**
     * @param Semigroup<ListBrand<A>> $that
     * @return List<A>
     */
    public function append(Semigroup $that): List
}
```

There is a little unpleasantness in the type signature of `append` because we would have liked to have `List<A>` as a parameter type instead of `Semigroup<ListBrand<A>>`. But this is actually not a problem since we can pass a `List<A>` as a parameter without issues and actually, as soon as `List` is the only class using `ListBrand` as brand, nothing else. The implementation of `List::append` will just need to take care about a type conversion between `Semigroup<ListBrand<A>>` to `List<A>`, but that's something the client should not be concerned with.

In this way we are able to type correctly the idea of having the _same type_ we wanted to abstract. Introducing brands complicates things since they require the creation of two classes instead of one for each new interface implementation, but it is a small price we could be very well willing to pay to obtain a great improvement in type safety.

## Conclusion

Once again, adding more information at the type level saved the day and allowed us to introduce in our code a concept which was otherwise not representable. I must point out that having to deal with an object oriented API created more issues than the benefits it provided. Using a simple function like

```php
/**
 * @template A
 * @param A $this
 * @param A $that
 * @return A
 */
function append($this, $that)
```

would have not created any issue whatsoever.

Moreover, the solution I propose does not play nicely with inheritance as you can see [here](https://psalm.dev/r/ae441ac9f8). You can forget such issues by avoiding inheritance altogether and using such techniques only on `final` classes.

As always, this turns out to be a trade-off. Object oriented languages are not designed to achieve the best type safety one could desire, so if one wants to achieve it, she needs to renounce to other object oriented features.
