---
layout: post
title:  "Thoughts on code quality"
author: Marco Perone
date: 2020-06-09 08:06:42 +0200
categories: post
tags: code quality programming
comments: true
pageUrl: '"http://marcosh.github.io/post/2020/06/09/thoughts-on-code-quality.html"'
pageIdentifier: '"Thoughts on code quality"'
description: "Thoughts on code quality"
image: "/img/quality.jpg"
---

Lately, while I was looking for a new job, I went through quite a lot of job interviews and I had the chance to talk about code and what I like about it with several other developers coming from different backgrounds. Unsurprisingly, the concept of quality emerged several times and this got me thinking about what I actually mean by code quality. It is something I care about a lot, so I spent some time trying to clear my mind on the subject and now I decided to share my thoughts, hopefully to get some feedback.

## Code quality for code quality's sake

To try to judge the quality of a codebase we need some kind of metric or heuristic which can help us understand if the code is actually good or bad. There are definitely many heuristics every developer knows and tries to follow while writing software. Some are more concrete, some more abstract, some are easily measurable with a tool, some others really depend on the perception of the single person.

I'd like to distinguish between intrinsic and extrinsic metrics. Extrinsic ones depend on input external to the codebase, such as customer expectations, user satisfaction while using the product and similar heuristics which we are not able to judge just by looking at the code. Intrinsic ones on the other hand are metrics we could obtain just by looking at and analysing the codebase itself. Some well-known examples are test coverage and cyclomatic complexity.

For the sake of this post, I'm interested only in intrinsic metrics and I'll try to explain what I consider to be good heuristics to judge the quality of a codebase just by looking at the code. With this choice, I definitely don't want to convey the idea that intrinsic metrics are more relevant than extrinsic ones, especially when considering software products. Nonetheless, I feel that it still make sense to talk about good quality code even without knowing if the software conformes to some requirements of if it provides real customer value. In other words, useless software can truly be beautiful!

## Why do we care about code quality

To understand which quality metrics are more relevant, we first need to understand what is the aim of writing code of good quality. I won't spend too many words on this, since it has already been well covered [elsewhere](https://martinfowler.com/articles/is-quality-worth-cost.html). Just to briefly sum up the topic, the main takeaway is that high quality software is actually cheaper to produce.

This is due to the fact that high quality code aims to be maintainable, making refactorings and evolution easy. Reading and understanding a high quality codebase should be easy, so that a new developer working on it should be able to add new functionalities in a short time span.

I'd like to add one more thing to this consideration: producing good quality code makes the developer motivated and proud of what she is doing. This leads to happy coders, who are likely to be more productive, in a continuous virtuous cycle.

Assuming now that the reason we should write quality code —to achieve reliable and maintainable projects— is clear, a big question remains open: how do we achieve that? Which are the traits that characterise good code? What can we do to introduce them in our codebase?

In my humble opinion, there are mainly two attributes of good quality code: being explicit and offering guarantees.

## Quality code is explicit

How do we achieve readability of a codebase? How do we make sure that we don't hide bad surprises for those who come after us in the project's evolution?

My answer would be just by trying to make things as explicit as possible. If a behaviour of our code is explicitly described and explained, whoever comes after us will have an easier job understanding what our code is actually doing.

There are several ways to improve how explicit a codebase  is. The main ones are documentation, tests and types, each ones having their pros and cons. Documentation is certainly the most human readable of the three, being mainly prose describing what the code does and how to use it. Unfortunately it is the one which is harder to maintain since it is very time consuming to keep it up to date. Tests are a great way to express explicitly how a codebase behaves and how it should actually be used. On the other hand, tests are often separated from the actual code and they are not always written in a way which improves the understanding of the codebase. Types allows us to describe what a piece of code does in a very detailed way in a place which is extremely close to the code itself. Still, many programming languages lack a type system sufficiently expressive to describe precisely what our code does and doesn't do.

To make things a bit more concrete, let me try to provide a specific example. Consider a function `getUser` which, given an `Id`, retrieves a `User`. Such a function can fail if the required user is not actually on the list.

```java
function getUser(id) {...}
```

If we write just the above, it is not explicit at all that the function itself can fail. The reader of the code needs to examine all the implementation of the function and reason about it to deduce that it could fail. This process could be very involved, complicated and error prone.

We could try adding some documentation to the function, stating explicitly that it could fail, possibly with an exception.

```java
// this function could fail throwing a UserNotFound exception
function getUser(id) {...}
```

This is definitely nicer, since now it's not needed to read the whole implementation to find out it could fail. The reader is already provided all the information she needs and doesn't need to parse and process all the implementation.

We could provide the same information using the type system, stating explicitly in the return type that we could fail to retrieve a `User`.

```java
function getUser(Id id): Maybe User {...}
```

where `Maybe` is a data structure that makes it explicit that its wrapped value may in fact be missing.

## Quality code offers guarantees

How do we make sure that a codebase is easy to refactor and maintain?

We need to provide the reader of our codebase guarantees that could be leveraged while evolving the code. There are two kinds of guarantees we can provide: we can guarantee that what our codebase says it does is actually true, and we can also guarantee that our codebase doesn't do anything else than what it claims to be doing. How might we achieve that?

Coming back to the three ways of making things explicit — documentation, tests and types— let's see which guarantees they actually offer. Documentation can be very explicit and can describe in details what the code does and doesn't do, but you still need to trust who wrote the docs and hope that they are still up to date. Tests can certainly give much more confidence, since we can execute them and get an immediate feeling that the codebase satisfies the behaviours which are under test. Still, even with a 100% code coverage, we could have missed a special edge case, or forgot to test a particular combination of function calls. Types on the other hand cannot lie, they can describe precisely what a piece of code can and can't do; they are without any doubt up to date and, in a strongly typed language, we cannot forget to provide types for data and functions (or we could, and rely on type inference to figure out the types for us).

Coming back to the example from the previous section, let's see how we can make some guarantees on the `getUser` function  explicit.

We can use tests to guarantee that we will get back a `User` if he is found on the list, or we will receive a `UserNotFound` exception otherwise.

```java
describe "getUser"
  it "returns a User if it is found" ...
  it "throws a UserNotFound exception it the user is not found" ...
```

This can guarantee us those specific behaviours, but we can't ascertain whether some other behaviour is also possible. For example, if we want to retrieve data from a database, we might get a connection error instead.

Using types, we can guarantee what the function does and also what the function doesn't do. When we return `Maybe User` we know that there are only two possible outcomes: success, which brings with it a `User`, and failure. Moreover, if we know that our function is pure, we also know that nothing else can happen in that function other that returning one of those two values; no I/O, no stateful computations, nothing which could lead to unexpected behaviour.

## Explicitness and guarantees are team-dependent

Being explicit and providing guarantees are two heuristics which can help to create and maintain a high quality codebase. Still, I believe it is not universally possible to agree on what explicit and readable code means. Something is explicit if it highlights and conveys an idea or a concept in a straightforward way; therefore it depends on the perception of the addressee of the information whether the idea comes through easily. Some code could appear very explicit to one developer and very abstruse to another one. One developer could be very comfortable with very abstract code, while another might better understand some very concrete and down to earth code.

For example these types

```haskell
type Optic p s t a b = p a b -> p s t
type Grate s t a b = forall p. Closed p => Optic p s t a b
grate :: forall s t a b. (((s -> a) -> b) -> t) -> Grate s t a b
```

are extremely precise, but I wish you good luck if you assign a developer unexperienced with profunctor optics to work with them! Precision and abstraction as above have an inner cost in previous knowledge and comprehension. So if your team is not comfortable with such concepts, I'd say that such code is not readable and hence not high quality code.

To obtain a shared good level of quality, we have to consider what we actually want to make explicit based on the experience of the team. Experienced developers, which are more at ease with more advanced code, should continuously try to raise the bar teaching others so that the whole team level could improve over time.

## Conclusion

It is my opinion that code quality should be considered and possibly measured independently from external factors such as adherence to specifications or user satisfaction. It is then a choice of the development team or of the business to prioritise code quality, performance, speed of development or any other metric one could think of. It should be a shared and clear choice so that everyone knows what is expected.

Writing explicit code which states clearly what it will and won't do and which provides clear guarantees could be a productive way to create high quality code.

Aiming at code quality certainly has a cost, that can't be denied, but the return on investment, especially on big projects on complex domains, can be so much worth it!

I would really like to thank a lot my friends [Ubaldo Pescatore](https://twitter.com/maubalpes), [Erik Post](https://twitter.com/erikagain) and [Matteo Baglini](https://twitter.com/matteobaglini) for the review and the useful and interesting comments.
