---
layout: post
title:  "Introducing Haskell in Soisy"
author: Marco Perone
date: 2021-06-04 08:06:42 +0200
categories: post
tags: haskell dhall
comments: true
pageUrl: '"http://marcosh.github.io/post/2021/06/04/introducing-haskell-in-soisy.html"'
pageIdentifier: '"Introducing Haskell in Soisy"'
description: "Introducing-Haskell-in-Soisy"
image: "/img/haskell.png"
---

[Soisy](https://www.soisy.it/) is an Italian startup working in the fintech sector, providing a payment by instalments system to affiliated e-shops.

Some months ago the need to release a new version of our risk engine emerged, so we started considering how to actually build it.

The decision we took was to use [Haskell](https://www.haskell.org/). In this post we will try to summarise our path, from the initial situation and the perplexities around the usage of a new technology, to the actual release in production.

If you prefer, you can read the italian version of this post on [Soisy's blog](https://www.soisy.it/introduzione-haskell-stack-tecnologico/).

## A bit of context, what is our risk engine?

As we mentioned above, Soisy provides a payment by instalments system. As any service in the buy-now-pay-later market, we need to protect ourselves from possible insolvencies from our customers.

The main tool we have to do this is our risk engine, which basically processes several data provided by our customers to decide whether to grant or deny a loan.

In extreme synthesis, our risk engine is a stateless service which receives a bunch of data, performs some apparently mysterious computations and then says `Yes` or `No`, which means loan approved or not.

## Our default tech stack

Soisy was born and evolved until now as a monolithic `PHP` application. We use an architecture based on `Event Sourcing` and `CQRS` and we try to keep the quality of our code as high as we can, thanks also to the great experience of our whole tech team.

Still, as time passes by, a monolithic PHP application is starting to feel a bit too tight for our needs and our growth rate.

Also, as our team grows, we are bringing inside the company experiences and skills which goes beyond our current main stack, going from functional programming to distributed systems and infrastructure as code.

## The proposal

Within the context described above in mind, when the time to actually start developing the new version of the risk engine came, one of our engineers came out with the following thought on our internal Slack:

> we should do this in Haskell

The main motivations he brought forward were:

- Haskell would allow us to model the domain in an easier and clearer way. Using algebraic data types and functions instead of classes, objects and methods allows to create a more robust domain model of the application;
- Haskell allows to almost forget about some less relevant issues as serialization and deserialization thanks to generic programming;
- Haskell type system allows extreme agility and confidence in performing complicated refactorings. Changing the design of the application is possible thanks to the help of the compiler;
- the Haskell ecosystem, at least for what we were concerned about, provides high quality libraries and the community is generally very welcoming and helpful;
- performances and speed of execution are on another level with respect to an interpreted language such as `PHP`.

## The doubts

The proposal was received with mixed feelings by the team, meaning that someone was enthusiastic while someone else was a bit more perplexed.

The most relevant questions and observations which were raised were:

- how independent is the risk engine from the rest of the application? Can we easily separate it from the rest?
- if we want to try Haskell, we should treat it as a test, being aware that it could be a failure;
- we should consider well the impact of such a decision with respect to the timing of the realization;
- anything new which goes into production is a possible cause for new bugs and emergencies. How will we keep this factor under control in this case?
- a small part of the team knows the language and would be able to evolve the new application. Wouldn't this create a bottleneck?

The team which formed around the new implementation of the risk engine considered all the above possible perplexities and decided to accept the risks which come with introducing a new technology, judging that the benefits would be more important than the drawbacks. In particular some relevant considerations were:

- regarding the time of delivery, the whole team decided to give priority to the reliability and the quality of the solution with respect to the speed of release;
- regarding bugs and emergencies, we decided to run the new version of the risk engine in parallel to the old version before the actual release, to prevent bad surprises when going live;
- we decided to do as much pair programming as possible to share the knowledge on Haskell and on the domain of the risk engine;
- moreover, after the release we decided to invest some of the tech team's time in education on Haskell.

The company accepted and trusted the considerations of the team and so the adventure could actually start!

## Some technical decisions

Facing the task to write a greenfield Haskell project, we had the chance to evaluate our own tech stack for the project. We mostly went with what we knew best, so that we could benefit from our own previous experience.

Some of the most relevant choices are described here below.

### (Almost) boring Haskell

One member of the team was at his first experience with Haskell, so we decided not to burden him with too many abstract concepts.

Most of the time we preferred to avoid abstractions which were not helping the readability of the code. We preferred to stick with concrete types instead of going with constrained type variables which would have made certain functions more general.

The whole service is basically stateless and pure in nature, so we could avoid even using general monads, monad transformers or any other effect-managing mechanism, except for some minimal `IO` at the borders.

The only concession we decided to grant for more advanced concepts was for lenses, which makes management of nested data structures way easier. Still, we preferred to avoid infix operators and we stuck with explicit functions as `view` and `set` instead.

### Stack

We choose [`Stack`](https://docs.haskellstack.org/en/stable/README/) to build our project and to manage our dependencies. The choice was mainly due to our previous experiences, and we are really happy by the choice we made since we never had any issue with regard to dependencies and builds.

### Servant

With the need to create a web service with a few simple endpoints on a private network, where authentication, for the moment, is not an issue, we decided to go with [`Servant`](https://haskell-servant.github.io/), to take advantage of its extreme type safety and expressivity. We particularly love the ability to easily generate an always up to date documentation of the API, which turns out to be really useful in a team where not everyone can fluently read the actual Haskell code.

### Dhall

The risk engine has a lot of rules which are tuned by a big number of parameters. Being able to manage them in a configuration file makes the whole application much more maintainable and evolvable.

Using [`Dhall`](https://dhall-lang.org/) as a configuration language allowed us to increase our ability to refactor and restructure the configuration, keeping it easily in sync with the code.

### QuickCheck

Also when using PHP we try to write some of our tests using a property based approach. Doing this in Haskell is much more pleasant since the language bends itself towards it very nicely.

We tested our application thoroughly, both unitary and functionally. One great thing is that Haskell is so much faster than PHP at running tests. The same tests that in the previous version of the engine took around 10 minutes, with Haskell it lasts something like 2 seconds.

## The outcome

It's now three weeks since we released the new rating engine into production and, up to now, everything is working smoothly.

The release of the project took a bit longer than what we expected in the beginning, but most of the unexpected delay came from integrating the new project with the PHP monolith and not from issues with Haskell itself.

For the moment, even if the project is running in production, our Haskell adoption remains an experiment. In due time, evaluating how the application is performing and how the maintenance of the project is affecting the whole team, we will close the experiment and decide whether to permanently include  Haskell in our tool stack or go back to our beloved PHP.
