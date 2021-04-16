---
layout: post
title:  "Exercises in style - 2"
author: Marco Perone
date: 2021-04-16 08:06:42 +0200
categories: post
tags: php object-oriented
comments: true
pageUrl: '"http://marcosh.github.io/post/2021/04/16/exercises-in-style-2.html"'
pageIdentifier: '"Exercises in style - 2"'
description: "Exercises-in-style-2"
image: "/img/style.png"
---

I would like to briefly present another exercise in design style. You can check also the previous one [here]({% post_url 2021-04-16-exercises-in-style-1 %}).

This time we're going to discuss how to build an object. For the sake of the example we'll want to contruct an object `Foo` which composes two `callable`s `$bar` and `$baz`.

```php
final class Foo
{
  /** @var callable */
  private $bar;

  /** @var callable */
  private $baz;
}
```

The example is intentionally simple and the solutions discussed below could make sense only in more complicated cases. We are only interested in exploring the techniques.

## Solution 1: constructor

The most obvious solution is just to pass them in the constructor

```php
final class Foo
{
  ...

  public function __construct(
    callable $bar,
    callable $baz
  ) {
    $this->bar = $bar;
    $this->baz = $baz;
  }
}
```

You would have to instanciate `Foo` as in

```php
$foo = new Foo(
  fn() => 42,
  fn() => 'hello'
);
```

## Solution 2: named constructor

Reading the last snippet of code could be hard, since the type system does not help you to understand which callable is which (this could be bypassed using [named parameters](https://wiki.php.net/rfc/named_params) even if I won't fully encourage [their usage](https://externals.io/message/110004#110005)).

You could sweeten the usage of `Foo` by providing a named constructor which is more readable

```php
final class Foo
{
  ...

  private function __construct(
    callable $bar,
    callable $baz
  ) {
    $this->bar = $bar;
    $this->baz = $baz;
  }

  public static function withBarAndBaz(
    callable $bar,
    callable $baz
  ): self {
      return new self($bar, $baz);
  }
}

$foo = Foo::withBarandBaz(
  fn() => 3,
  fn() => ""
);
```

This is a bit more explicit and could really help the reader of the code.

## Solution 3: newtypes

The previous solution improves the readibility but not the type safety. To achieve the latter we could define some wrappers around our callables

```php
final class Bar
{
    /** @var callable */
    private $f;

    private function __construct(callable $f)
    {
        $this->f = $f;
    }

    public static function new(callable $f): self
    {
        return new self($f);
    }
}

final class Baz
{
    /** @var callable */
    private $f;

    private function __construct(callable $f)
    {
        $this->f = $f;
    }

    public static function new(callable $f): self
    {
        return new self($f);
    }
}


final class Foo
{
  /** @var Bar */
  private $bar;

  /** @var Baz */
  private $baz;

  private function __construct(
    Bar $bar,
    Baz $baz
  ) {
    $this->bar = $bar;
    $this->baz = $baz;
  }

  public static function withBarAndBaz(
    Bar $bar,
    Baz $baz
  ): self {
      return new self($bar, $baz);
  }
}

$foo = Foo::withBarAndBaz(
  Bar::new(fn() => 3),
  Baz::new(fn() => "")
);
```

In this way, the reader of the code will immediately understand that the first argument of `Foo` is of type `Bar` and the second of type `Baz` (here using reasonable names and not `Bar` and `Baz` could help 😀).

## Solution 4: type safe builder

The previous solution is quite nice but extremely verbose in a language such as PHP.

Another road we could try to cover is building the object one step at the time. I'm thinking about an API which looks like

```php
$foo = Foo::empty()
  ->withBar(fn() => 3)
  ->withBaz(fn() => "");
```

This is readable and explicit, but in principle it allows you to build instances of `Foo` in an invalid state. To prevent this we could try to use [Psalm](https://psalm.dev/) and add some type level informations. This could look as follows

```php
<?php

/**
 * @psalm-immutable
 * @template Bar of callable|null
 * @template Baz of callable|null
 */
final class Foo
{
  /** @var Bar */
  public $bar;

  /** @var Baz */
  public $baz;

  private function __construct()
  {
  	$this->bar = null;
    $this->baz = null;
  }

  /**
   * @return self<null, null>
   */
  public static function new(): self
  {
    /** @var self<null, null> */
    $a = new self();

    return $a;
  }

  /**
   * @return (Baz is null ? self<callable, null> : self<callable, callable>)
   */
  public function withBar(callable $bar): self
  {
    $that = new self();
    $that->bar = $bar;
    $that->baz = $this->baz;

    /** @var self<callable, null>|self<callable, callable> */
    return $that;
  }

  /**
   * @return (Bar is null ? self<null, callable> : self<callable, callable>)
   */
  public function withBaz(callable $baz): self
  {
    $that = new self();
    $that->bar = $this->bar;
    $that->baz = $baz;

    /** @var self<null, callable>|self<callable, callable> */
    return $that;
  }
}

/**
 * @param Foo<callable, callable> $foo
 */
function useFoo(Foo $foo): array {
  return [($foo->bar)(), ($foo->baz)()];
}

$foo = Foo::new()
  ->withBar(fn() => 42);

useFoo($foo); // InvalidArgument - Argument 1 of useFoo expects Foo<callable, callable>, Foo<callable, null> provided
useFoo($foo->withBaz(fn() => "")); // works
```

Lifting enough information at the type level we are able to enforce that an object is not consumed before it is actually fully constructed.
