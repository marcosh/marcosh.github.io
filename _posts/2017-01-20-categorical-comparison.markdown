---
layout: post
title:  "A categorical comparison between FP and OOP"
date:   2017-01-20 09:31:13 +0200
categories: post
tags: functional-programming category-theory object-oriented-programming
comments: true
pageUrl: '"http://marcosh.github.io/post/2017/01/20/categorical-comparison.html"'
pageIdentifier: '"categorical-comparison"'
---

[Category theory](https://en.wikipedia.org/wiki/Category_theory) is the branch of mathematics abstracting on the concept of [composition](https://en.wikipedia.org/wiki/Function_composition). As [Bartosz Milewski](https://twitter.com/BartoszMilewski) explains in [this post](https://bartoszmilewski.com/2014/11/04/category-the-essence-of-composition/), composition (and decomposition) is essential in computer programming, since it allows us to solve small issues one by one and then compose the pieces to tackle complex problems.

There is a lot of literature regarding how category theory and functional programming go really well together. In this post I would like to explore how object oriented programming relates with category theory, and hence with the concept of composition.

## The relation between Category Theory and FP

To being able to draw a comparison, let us consider briefly a strongly typed purely functional language. Here we have a concept of `Type` and a concept of `Function`.
A function `f : A -> B` is just a procedure that is able to compute a value of type `B` starting from a value of type `A`.

In this setting it is really easy to define the concept of composition of two functions: if we have a function `f : A -> B` and a function `g : B -> C`, we can define the function `g.f : A -> C` as the function that computes a value of type `C` passing to `g` the value computed by `f`.

With this definition of function composition, it is not hard to verify that the collection of types and the collection of morphisms between them actually form a category.

Now that we have a category, we have at hand all the theory developed by mathematician regarding categories and we could start to use it and profit from it in our programs.

## Category theory with OOP

Can we do the same with object oriented programming? Let's start by defining what objects (I mean [objects]() from a categorical point of view, not from an OOP one) and morphisms could be.

It seems natural to consider classes as the collection of objects and methods defined on those classes as morphisms. This means that a method `f` defined on a class `A` that returns instances of a class `B` defines a morphism `f : A -> B` in our category.

Let's try to see now if with this definition  we are actually able to obtain a category of if we fall short.

First thing, we need to define how function composition works. Let `f : A -> B` and `g : B -> C` be methods defined respectively on classes `A` and `B`

    class A
    {
        method f (params)
        {
            // do something on params

            return something; // this is an instance of class B
        }
    }

    class B
    {
        method g (otherParams)
        {
            // do something on otherParams

            return somethingElse; // this is an instance of class C
        }
    }

We can define a morphisms `g.f : A -> C` as a method on the class `A` in the following way

    class A
    {
        method g.f (params, otherParams)
        {
            // do something on params and otherParams

            return (this->f(params))->g(otherParams);
        }
    }

It is easy to define an identity morphism for every class `A` as

    class A
    {
        method id ()
        {
            return this;
        }
    }

that satisfies the equalities `id.f = f` and `g.id = g` for every `f : B -> A` and `g : A -> C`.

For associativity, let's now consider three morphisms `f : A -> B`, `g : B -> C` and `h : C -> D` and verify that `h.(g.f) = (h.g).f`. Making it explicit we need to consider the two methods `h.(g.f)` and `(h.g).f` on the class `A`

    class A
    {
        method h.(g.f) (fParams, gParams, hParams)
        {
            return (this->(g.f)(fParams, gParams))->h(hParams);
        }

        method (h.g).f (fParams, gParams, hParams)
        {
            return (this->f(fParams))->(h.g)(hParams, gParams);
        }
    }

By definition of `g.f` and `h.g` they are both equal to

    class A
    {
        method h.(g.f) (fParams, gParams, hParams)
        {
            return ((this->f(fParams))->g(gParams))->h(hParams);
        }
    }

hence the composition is actually associative. Therefore we can conclude that we are actually looking at a true category!

## The categorical comparison

Let's invest a minute now to see if the category that we just defined is actually worth considering and handy to use.

The first thing that we can observe is that composition in the `OOP` category requires us to define a new method every time. Hence composing morphisms in such a category is possible but not so handy and not that much used as a design tool when architecting and implementing a system. This issue is not present in functional programming, where you have function composition for free; therefore in a `FP` setting composition of functions becomes the main tool to put together several components of an application. I wonder if having a specific syntax (something like `g.f`) in an `OOP` language to define composite methods would turn out useful and promote more method composition, while helping avoid violations of the [Law of Demeter](https://en.wikipedia.org/wiki/Law_of_Demeter).

The second thing is that the concept of equality of morphisms in the `OOP` category is a little trickier than the one in the `FP` category. Just consider the following methods

    class A
    {
        private i;

        method f (param)
        {
            this.i = this.i + 1;

            return param;
        }

        method g (param)
        {
            return param;
        }
    }

It is clear that `f (arg) = g (arg)` for every possible `arg`, but still I would argue that `f` and `g` are not equal because their [effects](https://en.wikipedia.org/wiki/Side_effect_(computer_science)) are different. This can not happen in a functional setting where every function is pure and can't have hidden side effects, and therefore two functions are equal if and only if their output is equal for any possible input. Probably, if `this` was immutable, we wouldn't have such an issue, but that would probably cause all the `OOP` world to lose its meaning (or maybe not?).

## Conclusion

Both `FP` and `OOP` let us work with a category and use function and method composition to build complex solutions from smaller pieces. By its simple nature, with a composition easily achievable in the language, `FP` seems to be more suitable to create composable and reusable applications. This is still possible and commonly done in an `OOP` world, where you have also other concepts of [composition](https://en.wikipedia.org/wiki/Object_composition), but it requires more complicated machineries as design patterns and the like.