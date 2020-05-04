---
layout: post
title:  "Intervals and their relations"
author: Marco Perone
date: 2020-05-04 09:06:42 +0200
categories: post
tags: functional-programming time intervals ddd
comments: true
pageUrl: '"http://marcosh.github.io/post/2020/05/04/intervals-and-their-realations.html"'
pageIdentifier: '"Intervals and their relations"'
description: "Intervals and their relations"
image: "/img/intervals.png"
---

Two years ago I had the luck to participate to [DDDEurope](https://dddeurope.com) and listen to the nice keynote by Eric Evans on [Modelling Time](https://youtu.be/T29WzvaPNc8). I really liked the idea of modelling time not necessarily as instants but more as intervals, and making explicit the possible relations which two intervals can have. These relations are described by the so called [Allen's interval algebra](https://en.wikipedia.org/wiki/Allen%27s_interval_algebra).

Some time ago I started working on a little Haskell pet project to compute the common time availability of several people (something like [doodle.com](https://doodle.com) but where everyone first declares his/her availability and only then a time is chosen from the intersection of the availabilities) and the key concept of the domain were exactly intervals.

A bit after that [Taylor Fausak](https://twitter.com/taylorfausak) released a Haskell package called [Rampart](https://hackage.haskell.org/package/rampart) which implements exactly the relations of Allen's interval algebra.

Finding myself with a little of free time, I didn't resist the temptation of trying to link my own library, which was dealing with intervals, to Rampart, so that I could consider relations between intervals.

While I was doing that, I realised that interpreting Allen's interval algebra relations without ambiguities was not trivial (see for example [this](https://github.com/tfausak/rampart/issues/2) Rampart issue) and in the end I decided to reimplement it in a more general and unambiguous way.

## INTERVALS

Loosely speaking we can define an interval as the set of values included between two boundaries. Trying to model this idea using types, we could sketch something like

```haskell
data Boundary a = Boundary { _value :: a }

data Interval a = Interval { _start :: Boundary a, _end :: Boundary a }
```

We would like to ensure that `_start` is always smaller than `_end`. To this end we hide the `Interval` data constructor and we expose a function to build an interval from two boundaries

```haskell
instance Ord a => Ord (Boundary a)

boundsInterval :: Ord a => Boundary a -> Boundary a -> Interval
```

At this point we have to make our first design decision. What should `boundsInterval` return if the first boundary if bigger that the second one? One option could be returning `Maybe Interval` to notify the caller that he provided invalid values. Another option is to refine a bit the vague definition of interval we gave above, declaring an interval to be the set of values which are greater than `_start` and smaller than `_end`. Doing so, it becomes clear that, if `_start` is greater than `_end`, the interval delimited by `_start` and `_end` should simply be empty. Currently though we don't have an explicit way to represent the empty interval, so let's add it

```haskell
data Interval a
  = Empty
  | Interval { _start :: Boundary a, _end :: Boundary a }
```

## RELATIONS

Now that we defined what an interval could be, let's try to understand how two intervals could relate to each other. We can list all the 13 possible relations in the following table

<table style="border: 1px solid; border-collapse: collapse; margin: 1em auto">
  <tbody>
    <tr>
      <th style="border: 1px solid; padding: 0.2em 0.4em">Relation</th>
      <th style="border: 1px solid; padding: 0.2em 0.4em">Illustration</th>
      <th style="border: 1px solid; padding: 0.2em 0.4em">Interpretation</th>
    </tr>
    <tr>
      <td style="border: 1px solid; padding: 0.2em 0.4em">
        <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/e39508d9792afbb9e4349a31177feedf8c0739ba" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.505ex; width:6.605ex; height:2.343ex;" alt="X\,{\mathrel  {{\mathbf  {<}}}}\,Y">
        <br>
        <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/f6cd026fa563453cec492707a4480906945c1af7" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.505ex; width:6.605ex; height:2.343ex;" alt="Y\,{\mathrel  {{\mathbf  {>}}}}\,X">
      </td>
      <td style="border: 1px solid; padding: 0.2em 0.4em">
        <a href="/wiki/File:Allen_calculus_before.png" class="image" title="X takes place before Y"><img alt="X takes place before Y" src="//upload.wikimedia.org/wikipedia/commons/8/83/Allen_calculus_before.png" decoding="async" data-file-width="194" data-file-height="36" width="194" height="36"></a>
      </td>
      <td style="border: 1px solid; padding: 0.2em 0.4em">X takes place before Y</td>
    </tr>
    <tr>
      <td style="border: 1px solid; padding: 0.2em 0.4em">
        <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/11a7b4cd52af8f35cd166f49055a562aa599da81" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.338ex; width:6.754ex; height:2.176ex;" alt="X\,{\mathrel  {{\mathbf  {m}}}}\,Y">
        <br>
        <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/e70f9d816a23e5833cfc5048c44fe1dc314c0111" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.338ex; width:7.496ex; height:2.176ex;" alt="Y\,{\mathrel  {{\mathbf  {mi}}}}\,X">
      </td>
      <td style="border: 1px solid; padding: 0.2em 0.4em">
        <a href="/wiki/File:Allen_calculus_meet.png" class="image" title="X meets Y"><img alt="X meets Y" src="//upload.wikimedia.org/wikipedia/commons/b/b8/Allen_calculus_meet.png" decoding="async" data-file-width="194" data-file-height="36" width="194" height="36"></a>
      </td>
      <td style="border: 1px solid; padding: 0.2em 0.4em">X meets Y (<i>i</i> stands for <i><b>i</b>nverse</i>)</td>
    </tr>
    <tr>
      <td style="border: 1px solid; padding: 0.2em 0.4em">
        <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/a5e3f53f470500c00b1799b9c40408e3a56e1f76" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.338ex; width:5.864ex; height:2.176ex;" alt="X\,{\mathrel  {{\mathbf  {o}}}}\,Y">
        <br>
        <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/f850e7d43af3a6349369554dd563d1613f45fc69" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.338ex; width:6.606ex; height:2.176ex;" alt="Y\,{\mathrel  {{\mathbf  {oi}}}}\,X">
      </td>
      <td style="border: 1px solid; padding: 0.2em 0.4em">
        <a href="/wiki/File:Allen_calculus_overlap.png" class="image" title="X overlaps with Y"><img alt="X overlaps with Y" src="//upload.wikimedia.org/wikipedia/commons/1/1f/Allen_calculus_overlap.png" decoding="async" data-file-width="194" data-file-height="36" width="194" height="36"></a>
      </td>
      <td style="border: 1px solid; padding: 0.2em 0.4em">X overlaps with Y</td>
    </tr>
    <tr>
      <td style="border: 1px solid; padding: 0.2em 0.4em">
        <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/0ef88d453109202be464e5a4fce6339d8d334f35" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.338ex; width:5.583ex; height:2.176ex;" alt="X\,{\mathrel  {{\mathbf  {s}}}}\,Y">
        <br>
        <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/dc6b94fb68b04aea5cf276509708de2e8df6cad5" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.338ex; width:6.325ex; height:2.176ex;" alt="Y\,{\mathrel  {{\mathbf  {si}}}}\,X">
      </td>
      <td style="border: 1px solid; padding: 0.2em 0.4em">
        <a href="/wiki/File:Allen_calculus_start.png" class="image" title="X starts with Y"><img alt="X starts with Y" src="//upload.wikimedia.org/wikipedia/commons/4/4a/Allen_calculus_start.png" decoding="async" data-file-width="194" data-file-height="36" width="194" height="36"></a>
      </td>
      <td style="border: 1px solid; padding: 0.2em 0.4em">X starts Y</td>
    </tr>
    <tr>
      <td style="border: 1px solid; padding: 0.2em 0.4em">
        <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/c9a0d1b7600bbae4d7fb352952511e01f0df7ad5" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.338ex; width:6.013ex; height:2.176ex;" alt="X\,{\mathrel  {{\mathbf  {d}}}}\,Y">
        <br>
        <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/367f82e4478a64966e4fdc19029789ad85c61e27" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.338ex; width:6.755ex; height:2.176ex;" alt="Y\,{\mathrel  {{\mathbf  {di}}}}\,X">
      </td>
      <td style="border: 1px solid; padding: 0.2em 0.4em">
        <a href="/wiki/File:Allen_calculus_during.png" class="image" title="X during Y"><img alt="X during Y" src="//upload.wikimedia.org/wikipedia/commons/5/53/Allen_calculus_during.png" decoding="async" data-file-width="193" data-file-height="36" width="193" height="36"></a>
      </td>
      <td style="border: 1px solid; padding: 0.2em 0.4em">X during Y</td>
    </tr>
    <tr>
      <td style="border: 1px solid; padding: 0.2em 0.4em">
        <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/af8dcc36a06ff722e40fabda958b7bb14433d549" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.338ex; width:5.581ex; height:2.176ex;" alt="X\,{\mathrel  {{\mathbf  {f}}}}\,Y">
        <br>
        <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/251dfd4bfb83542ed45e075f3be12b54224c3388" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.338ex; width:6.323ex; height:2.176ex;" alt="Y\,{\mathrel  {{\mathbf  {fi}}}}\,X">
      </td>
      <td style="border: 1px solid; padding: 0.2em 0.4em">
        <a href="/wiki/File:Allen_calculus_finish.png" class="image" title="X finishes with Y"><img alt="X finishes with Y" src="//upload.wikimedia.org/wikipedia/commons/0/06/Allen_calculus_finish.png" decoding="async" data-file-width="193" data-file-height="36" width="193" height="36"></a>
      </td>
      <td style="border: 1px solid; padding: 0.2em 0.4em">X finishes Y</td>
    </tr>
    <tr>
      <td style="border: 1px solid; padding: 0.2em 0.4em">
        <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/fa17ac451505aeb6e18fd71f20f6e41576aa984b" class="mwe-math-fallback-image-inline" aria-hidden="true" style="vertical-align: -0.338ex; width:6.605ex; height:2.176ex;" alt="X\,{\mathrel  {{\mathbf  {=}}}}\,Y">
      </td>
      <td style="border: 1px solid; padding: 0.2em 0.4em">
        <a href="/wiki/File:Allen_calculus_equal.png" class="image" title="X is equal to Y"><img alt="X is equal to Y" src="//upload.wikimedia.org/wikipedia/commons/d/df/Allen_calculus_equal.png" decoding="async" data-file-width="193" data-file-height="36" width="193" height="36"></a>
      </td>
      <td style="border: 1px solid; padding: 0.2em 0.4em">X is equal to Y</td>
    </tr>
  </tbody>
</table>

and define the following Haskell data type to enumerate all the possibilities

```haskell
data Relation
  = Equal
  | Starts
  | Finishes
  | During
  | StartedBy
  | FinishedBy
  | Contains
  | Before
  | After
  | Meets
  | IsMet
  | Overlaps
  | OverlappedBy
```

We would like now to define a function

```haskell
relate :: Ord a => Interval a -> Interval a -> Relation
```

to compute how two given intervals are related.

## SINGLETONS

When we try concretely to implement `relate` we realise that there are some situations which are a bit ambiguous and they all pertain the case where one of the two intervals is a singleton, i.e. the starting bound and the ending bound coincide.

Let's consider for example the relation between two intervals `a = boundsInterval x x` and `b = boundsInterval x y`. Should it be that `a` meets `b` since they have a boundary in common and `b` happens all after `a`? Or should it be that `a` starts `b`? Or maybe `a` overlaps `b`? It is not immediately clear what the correct choice should be.

The Allen's paper notices the issue and dismisses it considering only intervals with a start smaller than the end.

One possible solution comes from a more precise inspection of the domain and using a bit of maths to abstract the relevant information out of our particular case.

## BOUNDARIES

The first ingredient to solve our issue with `relate` is to be more precise in the description of our domain model.

Above, when we gave the definition of interval, we never really specified the semantic of the boundaries. When we write `Interval x y` do we mean that `x` and `y` are actually contained in the interval, or do we mean that they are excluded? There is no correct choice here, so to remain agnostic of this detail, we provide the possibility to specify explicitly in a boundary whether it should be included or excluded

```haskell
data IsIncluded
  = Included
  | Excluded

data Boundary a = Boundary
  { _boundaryValue :: a
  , _isIncluded :: IsIncluded
  }
```

Now we can be very explicit when we define our intervals

```haskell
a = boundsInterval (Boundary 1 Included) (Boundary 3 Excluded)
-- 1 ≤ x < 3

b = boundsInterval (Boundary 2 Included) (Boundary 2 Included)
-- 2 ≤ x ≤ 2 => b = {2}

c = boundsInterval (Boundary 0 Excluded) (Boundary 0 Included)
-- 0 < x ≤ 0 => c is empty
```

## RELATIONS RELOADED

Now that we introduced both included and excluded boundaries, the 13 cases in the `Relation` data type reveal themselves as being not that well defined. For example, if we consider the two intervals

```haskell
a = boundsInterval (Boundary 1 Excluded) (Boundary 2 Included)
b = boundsInterval (Boundary 2 Included) (Boundary 3 Excluded)
```

what should `relate a b` be? Should it be `Meets` or should it be `Overlaps`? We clearly need a more precise definition to disambiguate such cases.

To do this, we need to consider the operations that we have available on intervals. Considering intervals as sets, we can derive for them operations to compute the intersection and the union of two intervals

```haskell
intersection :: Ord a => Interval a -> Interval a -> Interval a

union :: Ord a
  => Interval a
  -> Interval a
  -> Either (Interval a) (Interval a, Interval a)
```

Notice how we are making explicit that the union of two intervals could be either one or two separate intervals.

Intersection and union induce a partial order `≤` on intervals defined by

```haskell
-- a is contained in b
a ≤ b = a `union` b == b
      = a `intersection` b == a
```

Using this partial order, we can identify four cases, when considering two intervals `a` and `b`:

- `a ≤ b` and `b ≤ a`: this, by antisymmetry, means that `a == b`; hence, it is clear that is must be `relate a b = Equal`

- `a ≤ b` and `b ≰ a`: this means that `a` is properly contained in `b`. We can further dissect this possibility checking if the boundaries are equal:
  - `start a == start b`: in this case `relate a b = Starts`
  - `end a == end b`: in this case `relate a b = Finishes`
  - `start a != start b` and `end a != end b`: in this case `relate a b = During`
<br><br>

- `a ≰ b` and `b ≤ a`: this is the symmetric case with respect to the previous one. Dissecting the possibilities as we did above we obtain the `StartedBy`, `FinishedBy` and `Contains` cases

- `a ≰ b` and `b ≰ a`: this means that neither `a` is contained in `b`, nor `b` is contained in `a`

The last case is still too broad, so we have to analyse it more precisely. We can check the intersection of `a` and `b` to split this further:

- ``a `intersection` b != empty``: this means that the two intervals have some value in common, hence they overlap. Depending on the ordering of `start a` and `start b`, we would get either `relate a b = Overlaps` or `relate a b = OverlappedBy`

- ``a `intersection` b == empty``: no value is in both intervals. At this point we inspect better the union of `a` and `b`:
  - ``a `union` b`` is two separate intervals: in this case the two intervals are really disjoint, meaning that there exists a value which is bigger that one and smaller than the other. Depending on the order of `start a` and `start b` we will get either `relate a b = Before` or `relate a b = After`
  - ``a `union` b`` is one single interval: the two intervals are disjoint, but there is not value between them, so they must be one right after the other. Depending on the order of `start a` and `start b` we will get either `relate a b = Meets` or `relate a b = IsMet`

This gives a precise algorithm to determine the relation occurring between any two intervals. We can see now that we have a precise way to distinguish what happens in cases where two intervals have a boundary with a common value

```haskell
relate (boundsInterval (Boundary 0 Included) (Boundary 1 Excluded))
       (boundsInterval (Boundary 1 Excluded) (Boundary 2 Included))
  = Before -- not Meets

relate (boundsInterval (Boundary 0 Included) (Boundary 1 Included))
       (boundsInterval (Boundary 1 Excluded) (Boundary 2 Included))
  = Meets

relate (boundsInterval (Boundary 0 Included) (Boundary 1 Included))
       (boundsInterval (Boundary 1 Included) (Boundary 2 Included))
  = Overlaps -- not Meets
```

It disambiguates also the issues with singleton intervals

```haskell
relate (boundsInterval (Boundary 0 Included) (Boundary 0 Included))
       (boundsInterval (Boundary 0 Included) (Boundary 0 Included))
  = Equals

relate (boundsInterval (Boundary 0 Included) (Boundary 0 Included))
       (boundsInterval (Boundary 0 Included) (Boundary 2 Excluded))
  = Starts

relate (boundsInterval (Boundary 1 Included) (Boundary 1 Included))
       (boundsInterval (Boundary 0 Included) (Boundary 2 Included))
  = During
```

## CONCLUSION

We took a look at intervals and how they can relate with each other. We noticed how a naive model of the domain leads to subtle issues and ambiguities. With a better model, distinguishing between included and excluded interval boundaries, and using their algebraic operations on intervals as much as possible, we were able to disambiguate the problematic cases and recover a clear view of the situation.

In general I believe that trying to model a domain using mathematical abstractions is a very productive choice. It usually leads to more general and elegant solutions because it forces us to understand our domain to isolate the relevant aspects and forget about the noise given by context-over-specific information.

If you want to check out a working implementation of what we discussed above, you can find it in my [availer](https://github.com/marcosh/availer) project.
