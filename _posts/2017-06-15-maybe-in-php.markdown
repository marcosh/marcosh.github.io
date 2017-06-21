---
layout: post
title:  "Maybe in PHP"
author: Marco Perone
date:   2017-06-16 17:53:42 +0200
categories: post
tags: functional-programming php maybe types
comments: true
pageUrl: '"http://marcosh.github.io/post/2017/06/16/maybe-in-php.html"'
pageIdentifier: '"maybe-in-php"'
description: "How to implement a Maybe type in PHP despite object orientation"
image: "/img/maybe.jpg"
---

Doing functional programming in a language as PHP, which is almost completely used as
an imperative or object oriented way, is not always easy. Good progresses have
been made thanks to the introduction of [callable type hints](http://php.net/manual/en/language.types.callable.php)
in PHP 5.4 and the diffusion of functional interfaces like the ones present in
[PSR-7](http://www.php-fig.org/psr/psr-7/).

Still, all "good" PHP code is still written using objects and classes and the object
oriented perspective on the world strongly influences even the most functional oriented
libraries.

In this post I would like to propose as an example how we could implement the `Maybe` type
in PHP. We will see how some open source libraries do this, we will see an alternative solution
and we will raise concerns about some modelling issues.

## SOMETIMES WE HAVE A VALUE, SOMETIMES WE DON'T

Sometimes in our application we need to treat values that may or may not exist at that
precise instant. You could think about an optional parameter, or about a value that needs
to be retrieved remotely and we don't know if it has arrived yet. Usually in PHP we
instantiate such a variable with a default value, generally `null`, which is used to
identity the case in which the value is not there yet. This approach is problematic,
because, before doing anything with the value, we need to check if it is not `null`, and,
if it is, handle the `null` case separately.

In strongly type languages like [Haskell](https://www.haskell.org/) or [Elm](http://elm-lang.org/),
the possibility of the non-existence of a value is approached differently. It is
addressed at the type level with the `Maybe` type, which is defined as follows:

```
Maybe a = Just a | Nothing
```

This means that a variable of type `Maybe a` could be either of type `Just a`, which is
just a wrapper for a value of type `a`, or of type `Nothing`, which means that no value
is there.

Now, if you have a variable `foo` of type `Maybe a`, you could use it in the following way

```
case foo of
    Just value -> // here you have access to the value
    Nothing -> // handle the case where you don't have a value
```

There are several things I really like about this approach and I'll try to list them here briefly:

- when you have a `Maybe a` you are sure it is or a `Just a` or a `Nothing`, there are no other
possibilities;
- when you have a `Maybe a` you are obliged to consider both cases (the compiler will check that
in Haskell and Elm), you can not be lazy and forget to treat the `Nothing` case;
- when you have a `Nothing` you don't have any operation that allows you to retrieve something of
type `a`.

## MAYBE IN PHP

`Maybe` is quite a common structure and there are several libraries in PHP implementing it. I would
like now to analyze the implementation of some of them to check if they comply with the points I
listed above. I would like to point out that I am not in any way suggesting that these libraries
are poorly written; I would just like to point out some limitations which, in my opinion, are
inherently present in an object oriented approach.

Let's start with [monad-php](https://github.com/ircmaxell/monad-php) by [Anthony Ferrara](https://twitter.com/ircmaxell),
where the author tries to show that the concept of monad could be used also in PHP.
He defines the `Maybe` class as follows (I'm stripping away non-relevant code):

```
class Maybe
{
    protected $value;

    public function __construct($value) {
        $this->value = $value;
    }

    public function extract() {
        return $this->value;
    }

    ...
}
```

As you see, only one class is defined, and therefore there is no clear distinction between `Just`
and `Nothing` and `null` is used as the discriminating factor. Moreover, we can call the `extract`
method also on `Nothing` objects, receiving `null` as return value.

Let's continue with [Phunkie](https://github.com/phunkie/phunkie) by [Marcello Duarte](https://twitter.com/_md),
a library with functional structures for PHP. In there we can find three classes that constitutes
an implementation of `Maybe`:

```
abstract class Option
{
    abstract public function get();

    ...
}

final class Some extends Option
{
    private $t;

    public function get() {
        return $this->t;
    }

    ...
}

final class None extends Option
{
    public function get() {
        throw new \RuntimeException("Illegal get() call on None");
    }

    ...
}
```

Here it is not possible to instantiate directly an Option and there is a clear distinction between
`Just` (here called `Some`) and `Nothing` (here `None`). Still, there is no way to ensure that `Option`
is either `Some` or `None`, since a new class could be defined to extend `Option`. Moreover, since `None`
extends `Option`, it must define a method `get` which should not be there; to denote that the method
should not be used, an exception is thrown introducing side effects in a generically pure codebase.

UPDATE 21/06/2017: Phunkie was updated to solve the issues I mention here; now it does not define `get` on
`Option` and therefore it doesn't need to define it throwing an exception on `None`. Moreover, using a
`final` constructor on `Option`, it ensures that an `Option` could be only `Just` or `None`. The only small
pitfall of this is that it ensures it throwing an exception.

These two libraries we just saw are not limited to define a `Maybe` type and address functional programming
in PHP more broadly. Let's have a look now at two libraries which are focused on the definition of `Maybe`
in PHP.

Let's start with [php-maybe-monad](https://github.com/haskellcamargo/php-maybe-monad) by
[Marcelo Camargo](http://haskellcamargo.github.io/about/), where we could find the following implementation
of `Maybe`:

```
interface MaybeInterface
{
    public function fromJust();

    ...
}

class Just implements MaybeInterface
{
    private $value;

    public function fromJust() {
        return $this->value;
    }

    ...
}

class Nothing implements MaybeInterface
{
    public function fromJust() {
        throw new \RuntimeException('Cannot call fromJust() on Nothing');
    }

    ...
}
```

As in the `Phunkie` implementation, inheritance is used to describe the fact that `Just` and `Nothing`
are subtypes of `Maybe` and an exception is used to prevent the client to use the `fromJust` method. The
main difference is given by the fact that `Just` and `Nothing` are not `final` here and therefore
subclasses of them could be defined.

Let's conclude our tour with [php-fp-maybe](https://github.com/php-fp/php-fp-maybe) by
[Tom Harding](http://www.tomharding.me/), where we can find the
following definition of `Maybe`:

```
abstract class Maybe
{
    abstract public function fork($default);

    ...
}

final class Just extends Maybe
{
    private $value = null;

    public function fork($_) {
        return $this->value;
    }

    ...
}

final class Nothing extends Maybe
{
    public function fork($default) {
        return $default;
    }
}
```

Here the approach is similar to the previous two libraries that we saw (especially to `Phunkie`), with
an abstract class `Maybe` extended by two final classes `Just` and `Nothing`. The main difference is that
the author decided not to expose a method `get()` to extract the value from `Just` because, as we saw
earlier, this would have caused him to implementing it also on `Nothing` where the only solution then is
throwing an exception. Instead a method `fork($default)` is defined, which requires a default value that
is used in the `Nothing` case; this saves us from using the exception, but still it forces us to have a
useless `$default` value even in the `Just` implementation of `fork`.

Rounding up, I think that all these libraries which we analyzed have some little design flaw, because in one
way or the other, they allow you to do more than what you should be supposed to do (for example, calling `get`
on `Nothing`) or they just force you to do something you should not be supposed to do (requiring a `$default`
value for the `Just` case). As I said before, I think that these flaws are inherently caused by an object
oriented approach; maybe I'll dive deeper on this idea in a future post.

I concluded the introduction saying that we would also look to a possible alternative to solve some of the
issues I raised until this point. Let's have a look!

## ANOTHER MAYBE IN PHP

I will first provide my proposal and then discuss it a bit with regarding to the issues I raised above.

```
interface Just
{
    public function get();
}

interface Nothing {}

class Maybe
{
    public static function nothing()
    {
        return new class() extends Maybe implements Nothing {};
    }

    public static function just($value)
    {
        return new class($value) extends Maybe implements Just
        {
            private $value;

            public function __construct($value)
            {
                $this->value = $value;
            }

            public function get()
            {
                return $this->value;
            }
        };
    }
}
```

You can notice that only `Just` has the `get` method to access its wrapped value. `Nothing` instead has no methods
and the only thing we could make of it is using it as type hinting (which is exactly the
only thing we would like to be possible). This saves us from using `null` as discriminating
value, throwing exceptions to notify the usage of a method which should not be called, or
requiring the usage of a default value in the signature of the method.

Still, we are not preventing the fact that a `Maybe` could be defined that is not `Just` or `Nothing`. This is
caused by the fact that we can not make `Maybe` as `final` since we want the objects returned by `Maybe::just`
and `Maybe::nothing` to type hint against `Maybe`.

I would be really curious if it would be possible to overcome these limitations and obtain a data structure
that better resembles the functional `Maybe` type.

## CONCLUSION

We saw several implementations in PHP of a `Maybe` data structure to see if we could replicate the following
properties of the `Maybe` type:

- when you have a `Maybe a` you are sure it is or a `Just a` or a `Nothing`, there are no other
possibilities;
- when you have a `Maybe a` you are obliged to consider both cases;
- when you have a `Nothing` you don't have any operation that allows you to retrieve something of
type `a`.

We saw that in an object oriented world like PHP it is pretty hard (if not impossible) to design a data
structure that satisfies these three properties. The main problem to overcome derives directly from the
usage of inheritance.

Inheritance naturally guides us or to expose methods where they shouldn't be and leaves the possibility
to provide other implementations of `Maybe` which are not `Just` or `Nothing`.

Maybe trying to replicate functional programming data structures in an object oriented world is not a
very smart thing to do, but it still helps us at identifying the limitations of the mainstream paradigm.
