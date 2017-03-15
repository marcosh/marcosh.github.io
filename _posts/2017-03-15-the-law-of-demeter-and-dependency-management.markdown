---
layout: post
title:  "The Law of Demeter and dependency management"
author: Marco Perone
date:   2017-03-15 07:40:42 +0200
categories: post
tags: demeter dependency-management
comments: true
pageUrl: '"http://marcosh.github.io/post/2017/03/15/demeter-dependency-management.html"'
pageIdentifier: '"demeter-dependency-management"'
description: "Applying the Law of Demeter to dependency management to avoid tricky issues"
image: "/img/demeter.jpg"
---

The Law of Demeter is a heuristic for writing good object oriented code which was
introduced by K. Lieberherr, I. Holland and A. Riel in their paper
[Object-Oriented Programming: An Objective Sense of Style](http://www.ccs.neu.edu/research/demeter/papers/law-of-demeter/oopsla88-law-of-demeter.pdf).

It is widely known among developers with the following concise formulation:

> Talk only to your friends

but actually it has several different variants, each one with a precise definition.
The most commoly used is probably the following:

> A method of an object may only call methods of:<br>
> - The object itself.<br>
> - An argument of the method.<br>
> - Any object created within the method.<br>
> - Any direct properties of the object.

In such a way it becomes explicit which methods are `friends` with the method
we are working on.

## Benefits of the Law of Demeter

If we pay attention to obeying the Law of Demeter while writing our code, we will
probably end up with a well organized codebase, since it promotes encapsulation
and modularity of code.

In particular it promotes coupling control and information hiding by restricting
the dependency relations among objects.

Moreover, as explicitly underlined in
[The Paperboy, The Wallet, and The Law of Demeter](http://www.ccs.neu.edu/research/demeter/demeter-method/LawOfDemeter/paper-boy/demeter.pdf),
it often leads to code that models more correctly the actual domain, respecting
the `Tell, don't ask` principle.

## Let's generalise

With the definition I gave above, the Law of Demeter clearly concerns object oriented
programming, and that was actually the field it was always applied to.

But, if we look closely to its definition, we see that there is room for generalisation.

The vague formulation I provided at the beginning speaks about `friends`, so we
need an abstraction where we can define a concept of `friend`. Facebook and Twitter
come to the rescue suggesting that the right abstraction here is a
[directed graph](https://en.wikipedia.org/wiki/Directed_graph).

In fact, if `G` is a directed graph, we can say that a vertex `W` is friend with
a vertex `V` if there is an arrow pointing from `V` to `W`.

With this definition, we could generalise the Law of Dementer as:

> A vertex V can use only vertices which are its friends.

To use this practically, we first need to define a directed graph and we also need
to specify what `use` means in that context.

In the standard object oriented setting, the directed graph is built as follows;
the vertices are all the methods of our objects and there is an arrow between a
method `M` and a method `N` in the following cases:

- `M` and `N` are methods of the same object;
- `N` is a method of an argument of `M`;
- `N` is a method of an object created inside `M`;
- `N` is a method of a property of the class which contains `M`.

In this context `M uses N` means that the method `N` is called inside of `M`.

If we look back at the first definition I gave, it's easy to see that they are
equivalent.

With our new definition at a graph theoretical level, we can now try to apply the
Law of Demeter to other contexts, just by defining a directed graph and stating
what `use` means.

## Application to dependency management

The first application that comes to mind for our new graph definition is dependency
management among software libraries.

Nowadays every modern programming language has its own package manager that allows
the developer to easily reuse public libraries. You could for example think of [npm](https://www.npmjs.com/),
[bower](https://bower.io/) or [yarn](https://yarnpkg.com/en/) for Javascript,
[Composer](https://getcomposer.org/) for PHP, [pip](https://pip.pypa.io) for Python,
and so on. Each one of these defines a format that the developer can
use to define which are the dependencies of his project.

The big job of the package managers is then retrieving recursively all the dependencies
and their own dependencies, with the correct version.

Having a concept of `dependency`, it is now easy to define a directed graph. The
vertices of the graph are all the packages and we draw an arrow from a package
`P` to a package `R` if `R` is an explicit dependency of `P`. By explicit we mean that
`R` is listed as a dependency of `P` in the specific section of the dependency
manager configuration.

On the other hand, we say that a package `P` uses a package `R` if in the code of
`P` there is any kind of reference to `R`. For example, `P` is using `R` if a 
function or a class defined in `R` appears in the code of `P`.

At this point, just by applying the graph version of the Law of Demeter to this new
context, we obtain the Law of Demeter for dependency management:

> A package `P` can contain references only to packages explicitly listed among
> its dependencies

For example, if a package `P` depends on a package `R` which depends on a package
`Q`, this version of the Law of Demeter suggests that, even if the code of `Q` is
available, we should not refer to it inside of `P`.

## Benefits of the Law of Demeter for dependency management

Similarly to what happens in the object oriented case, the Law of Demeter for
dependencies helps us prevent coupling between libraries, or at least makes us
aware of the coupling.

In practice this means that we need to care about changes only in our direct
dependencies. Suppose on the contrary that we are not following the Law of Demeter
for dependencies and we are using, in our package `P`, some component `C` defined in
a package `Q`, which is not a direct dependency of `P` but is a dependency of `R`,
which is itself a direct dependency of `P`.

What happens now if we update our dependencies and `C` changes inside of `Q`? Even
if we are following [SemVer](http://semver.org/) for our dependencies, this could
happen with a patch release of `R`, and our code will probably break unexpectedly.

Worse, it could happen that `R` abandons altogether `Q` as a dependency, even in
a patch release, and we will still be trying to use `C`, which is not available
anymore since `Q` is gone.

## Prevent the problems

If we want to prevent such issues, the solution is pretty easy; we should pay
attention to include among our dependencies all the packages that we are actually
using, even if they are already available because they are a dependency of a dependency.

Still, doing this manually could be pretty boring and error prone. At the moment
I am aware of two solutions to prevent the above described issues. In [Elm](http://elm-lang.org/)
the compliler itself will check if all the libraries that are imported in a file
are explicitely declared as dependencies in the package manager, making it impossible
to violate the Law of Demeter for dependencies and notifying every violation with
a compile time error. Less optimal than the solution in Elm, in Php you could use
the library [PhpDependencyChecker](https://github.com/maglnet/ComposerRequireChecker)
by [@Ocramius](https://twitter.com/Ocramius) and [@MaGlNet](https://twitter.com/MaGlNet)
to statically chack if all the symbols used in a library come from friendly dependencies.

## Conclusion

The Law of Demeter was introduced explicitly for object oriented code, but, as we
saw, it is easy to generalise to any setting where a concept of `dependency`
is present.

We saw how it could be immediately applied to the management of package dependencies
to give us a simple rule to prevent subtle bugs.

I would be really interested to know if the graph version of the Law of Demeter
could be applied to other settings and provide some other useful insights.
