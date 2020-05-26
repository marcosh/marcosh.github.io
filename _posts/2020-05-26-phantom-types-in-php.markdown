---
layout: post
title:  "Who's scared of phantom types?"
author: Marco Perone
date: 2020-05-26 08:06:42 +0200
categories: post
tags: functional-programming php types phantom
comments: true
pageUrl: '"http://marcosh.github.io/post/2020/05/26/phantom-types-in-php.html"'
pageIdentifier: '"Phantom types in PHP"'
description: "Phantom types in PHP"
image: "/img/phantom.jpg"
---

The type system of PHP is evolving with every new released version of the language and more new shiny features are coming with PHP 8, like [union types](https://wiki.php.net/rfc/union_types_v2) and [static return type](https://wiki.php.net/rfc/static_return_type). Still, if you want to push things further, you'd better use libraries as [PHPStan](https://phpstan.org) and [Psalm](https://psalm.dev/) to get access to features like type variables and purity checks. For this blog post I'm going to use Psalm as a type checker and show how to use a little type level trick with phantom types.

## Identify everything

I'll try to make my point going through a concrete example, which illustrates the issue and how phantom types help us achieve a nice solution.

Let's consider a common entity of your domain, let's call it `Foo`, which is identified by some kind of identifier. Let's say, for the sake of the example, that we are using integers as identifiers, probably coming from an auto-incrementing field on a relational database.

To make sure not to confuse our identifiers with other integer values floating around our application, we decide to wrap them in a class called `Id`.

```php
final class Id
{
  /** @var int */
  private $intId;

  public function __construct(int $intId)
  {
    $this->intId = $intId;
  }

  public function asInt(): int
  {
    return $this->intId;
  }
}
```

In this way we can distinguish at the type level an `Id` from any other integer in our application. For example, in our entity `Foo` we are going to type hint its identifier with `Id` and not just simple with `int`. This has the deficit that we have to wrap and unwrap our integer value, but has the big advantage of preventing passing an age, a day, a year or any other numeric value where an identifier was expected.

We're now happy because we have increased the type safety of our application!

## Until the next entity comes along...

Consider now another entity `Bar` which needs to refer to the same type of identifier `Foo` had. We are immediately tempted to use the same `Id` class we were using for `Foo`, it doesn't make sense to write another class which behaves exactly like `Id` does. Or does it?

If we reuse the same class, we'll still be able to distinguish between `Id`s and other integer values, but we're not going to be able to distinguish between `Id`s for `Foo` and `Id`s for `Bar`. That's something we might actually want to do, distinguish between `FooId` and `BarId`, so that we'll always know at compile time if we're dealing with an identifier for a `Foo` entity or for a `Bar` one. Consider for example repositories for `Foo` and `Bar`; the repository `FooRepository` could have a method `loadFoo(FooId $id): Foo` which accepts just identifiers for `Foo`, making it impossible to pass an identifier for `Bar`. This is definitely a win for type safety, but we're basically duplicating code in two identical classes which differ only for their name.

One could save code duplication defining an abstract class `Id` and concrete classes `FooId` and `BarId` inheriting from it.

```php
abstract class Id
{
  /** @var int */
  private $intId;

  public function __construct(int $intId)
  {
    $this->intId = $intId;
  }

  public function asInt(): int
  {
    return $this->intId;
  }
}

final class FooId extends Id {}

final class BarId extends Id {}
```

In this way we are not duplicating code, but still every time we introduce a new entity `Baz` in out system, we need to define also a new class `BazId extends Id` for its identifiers. In the long run, for big projects, this could become cumbersome.

## Let's move to the type level

If we examine carefully our desire to distinguish between `FooId` and `BarId`, we quickly realise that we are trying to communicate information at the type level. In fact, this is evident from the fact that the implementations are precisely the same. This observation suggests us that maybe we should try to use type level mechanisms to convey such information and not value level tools like inheritance.

One trick we could actually use is add a type variable to our `Id` class and use it to tag the identifier with the entity it is actually referring to. Let's see how that would work.

```php
/**
 * @template A
 */
final class Id
{
  /** @var int */
  private $intId;

  public function __construct(int $intId)
  {
    $this->intId = $intId;
  }

  public function asInt(): int
  {
    return $this->intId;
  }
}
```

The only difference with out first version of `Id` is that we now added the `@template` annotations to introduce the `A` type variable. Now we can simply refer to `Id<Foo>` and `Id<Bar>` to speak about identifiers for `Foo` and `Bar`, respectively.

Reconsidering the repository example from above, the same method `loadFoo` would become

```php
interface FooRepository
{
  /**
   * @psalm-param Id<Foo> $id
   * @return Foo
   */
  public function loadFoo(Id $id): Foo
}
```

This guarantees the same type safety as our intermediate solution using inheritance but avoids completely the need to create specific `BazId` classes for every new `Baz` entity.

Such kind of type variables, which are used only at the type level and do not refer to anything at the value level, are usually called phantom types.

## Conclusion

Now that PHP is getting more and more features to play with types, it is important to learn to use them at their full potential and to explore their practical value. One very relevant skill that developers need to develop in a language with a rich type system is how to discern information which is needed at the value level and information important only at the type level. Distinguishing between the two levels and separating information accordingly allows to design simpler and safer code and to reduce code duplication.

The usage of phantom types is a simple type level trick which can help with this in practical settings, without requiring complicated features or making your code too abstract and less readable.
