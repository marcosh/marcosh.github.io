---
layout: post
title:  "Manage effects"
author: Marco Perone
date:   2017-05-16 18:32:42 +0200
categories: post
tags: cqs effect functional-programming
comments: true
pageUrl: '"http://marcosh.github.io/post/2017/05/16/manage-effects.html"'
pageIdentifier: '"manage-effects"'
description: "The path from side effects to managed effects"
image: "/img/water_effect.jpg"
---

Often, when you start learning a new programming language, the first exercises you do require to create some procedure
that will compute a certain output given some inputs. This is helpful since it teaches you the syntax of the language,
but it does not allow you to write meaningful programs. To do that you will need to learn how to interact with external
components, may they be a database, a remote client, or directly a user of the system.

In this post we will talk about how, if we are not careful, this interaction with external systems often makes our
programs harder to read and to reason about, and we will consider a way to handle these concerns more smoothly.

## WHAT IS AN EFFECT

The interactions with the external system I was referring to above fall inside the concept of `effect` (often you will
find them referred as `side effects`, with a slightly negative connotation). But what is exactly an effect? Consider the
following pseudocode:

```
a = someComputation()

... // possibly some other code

b = someComputation()
```

We are evaluating the `someComputation` procedure twice in two different places of our program. Now, an effect is just
anything that will make `a != b`, for some function `someComputation` that we can define in our program.

Let's see some concrete examples:

```
a = readValueFromDatabase()

...

b = readValueFromDatabase()
```

If we read a value from a database in two different sections of our program, nothing assures us that we will obtain the
same result, hence reading from a database must be an effect.

Let us see another example:

```
a = readUserInput()

...

b = readUserInput()
```

If we read the input received from a user twice, we can not assure that the user did not change what she typed in the
meanwhile. Therefore reading the input of a user must be an effect.

Let's see one more example. For the sake of this example, suppose we are working with a generic object oriented
language:

```
obj = new Class()

a = obj->getValue()

...

b = obj->getValue()
```

Not knowing what happens between the two calls to the `getValue` method and what `getValue` actually does, we can not be
sure they will return the same value. This is due to the fact that `getValue` could depend on some mutable local state
encapsulated by `obj`. In other words, modifying local state is an effect.

## ANOTHER POINT OF VIEW: PURE FUNCTIONS

Before going further, let's give a relevant definition: a function is said to be `pure` if no effect can have an impact
on it. In other words, a function is pure if, given a certain input, it will always return the same output.

For example, any constant function, which always return the same value, is obviously a pure function.

Any function that performs some algebraic juggling on its inputs is pure, since there is no way that an addition,
a multiplication or a division would return a different value in two different points in time, even if the external
conditions are completely different.

More generally, a pure function is nothing else than a function as we studied it in school in our math courses, i.e.
a relation between two sets `A` and `B` which associates exactly one element `b` of `B` to every element `a` of `A`

## PROGRAMMING WITHOUT EFFECTS IS EASIER

I guess that it is not a case that when we start learning a new programming language initially we do not take effects
into account. This is because effects make our code harder to read and to manage.

Pure functions, i.e. functions which are not affected by effects, have some nice properties. First thing, if we compute
a pure function for some value, there is no need to do it anymore! In fact, if we know that for an input value `x` the
function `f` will return a value `f(x) = y`, that value will never change since `f` is pure. This means that we can
store somewhere the value `y` and use it directly next time we feed `x` to `f`, without actually needing to compute `f`
again.

Moreover, pure functions can help us reasoning with our code. In fact, if a function is pure, we can substitute the
result of the computation with the function itself everywhere in our program, without changing the meaning of the
program. For example, consider the following code:

```
f = function (string) { return length(string) }

a = 'goodbye effects'

l = f(a)
```

where we define a function `f` and a value `a` and we apply the function to the value. The meaning, and the output of
our program, would not change if we replaced `f(a)` with its result `15`, using directly `l = 15`. Mind you, we could do
this only because `f` is pure! In this simple example this technique does not help very much, but in a program with
a lot of functions composed with one another in contrived ways, such a line of reasoning could turn out to be very
useful. Reasoning about the output of a program, given a certain input, will become just like solving a mathematical
expression, where we simply proceed step by step computing simple operations, until we reach the final result.

Last point I want to make about pure functions is that they are really easy to test. Since all they do is associate
an output to an input, we just need to check that the association is made correctly, without worrying about anything
else. A typical test of a pure function could look like this:

```
input = // a given input
output = // the correct output

assertEquals(output, f(input))
```

## BUT PROGRAMMING WITHOUT EFFECTS IS USELESS

As much as one could be in love with pure functions and code free from effects, writing programs without effects is
pretty much useless. This would mean writing programs which could not interact with a user, read and write to a
database, receive or send any kind of message from another component, and so on ...

If a program is completely free from effects, its result can not depend on external input. In other words, a program
free from effects must be a program that returns just a constant. I must admit that I would like to create programs
a little bit more interactive than programs which just return the same value every time they are run.

Still, there exist some [purely functional programming languages](https://en.wikipedia.org/wiki/List_of_programming_languages_by_type#Pure)
where every function is pure and any effect is banned forever. How could they do something useful if every program
written with them is just a constant?

## DIVIDE ET IMPERA. ISOLATE EFFECTS TO MANAGE THEM

Let's leave our question aside for one moment and let's have a little detour. If I told you that I need to go
somewhere but I don't know the road, you could take me there, or you could just provide me with directions. Both ways
I'll be able to reach my destination. Still, the first option has the effect of changing our position, while the second
doesn't.

Similarly, an effectful program would perform some effect to obtain the desired result, while an effect-free program
would just provide the instructions to obtain the result and delegate the execution to something else. This is how
programs written in [Haskell](https://www.haskell.org/) or in [Elm](http://elm-lang.org/) work, they return a constant
which describes how to obtain the desired result, they provide the directions to our desired output. Then another
component, the runtime in this case, will analyze that description and will perform the required effects.

Still, this approach is not something that could be used only with purely functional languages (in fact, in such a
setting, you have no other choice). It could be used in any programming language, just with a little discipline from the
developer. The trick is just to separate code which is effectful from code which is not (doesn't this sound a lot like
[CQS](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation)?).

For example, consider this snippet of code:

```
void concatenate (string a, string b)
{
    print a + b;
}
```

The method `concatenate` is mixing effects and logic. We could split the two in the following way:

```
function lazyPrint (string a)
{
    return (a -> print a)
}

callable concatenate (string a, string b)
{
    return lazyPrint(a + b);
}
```

We are just separating in two different steps the purely functional operation of concatenating two strings and the
effectful operation of printing the string.

But what are the benefits that we get from this?

## BENEFITS OF ISOLATED EFFECTS

The main benefit that we get if we are able to isolate the effectful code is that we end up with a (almost) purely
functional codebase. This immediately implies all the benefits we talked above regarding code free from effects.
To recap, this means we will obtain code that is easily testable, highly composable, easy to reason about and often
really performant.

Moreover, if we have a purely functional core of our application that gets executed in a second step, in this fashion:

```
app = // purely functional core

app->execute(); // side effects
```

what we can actually do is compute `app` just once, since it is effect free and hence its result is constant. If we take
this line of reasoning further, we could think also to compute `app` locally in our machine and send around the computed
result to be just executed.

More generally, in our application effects will just become data and hence we will be able to manipulate effects as any
other kind of data. Treating effects as data means that we are able to pass them around in our programs, feeding them as
function arguments or returning them as function results; being just data, we will also be able to modify them
inside our applications. For example, if at a certain point we decide that we don't want to retrieve all our
inputs from a database anymore but we want to ask them directly to the user, we could just replace the effect describing
the database interaction with an effect representing interaction with the user. This will allow a great freedom in
controlling the workflow and the interaction with the external world of your application. This way effects are no more
`side effects` but they become `managed effects`.

## CONCLUSIONS

Managing effects correctly is one of the trickiest parts of programming, hence it must be treated with great care. One
approach that really helps is confining effects at the boundary of our application to obtain a purely functional, free
from effects, core. This approach allows us to work enjoying all the benefits of functional programming and grants us a
great control on the workflow and on the interactions with the external world. In purely functional languages this is
the only way of doing things, but such an approach could be used in any programming language with just a little
discipline from the developer.