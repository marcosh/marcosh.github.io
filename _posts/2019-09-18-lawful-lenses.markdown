---
layout: post
title:  "Ramblings on lawful lenses"
author: Marco Perone
date: 2019-09-18 08:06:42 +0200
categories: post
tags: functional-programming lens optic
comments: true
pageUrl: '"http://marcosh.github.io/post/2019/09/18/lawful-lenses.html"'
pageIdentifier: '"lawful-lenses"'
description: "Lawful lenses"
image: ""
---

<style>
* { box-sizing: border-box; }
*:focus {
  outline: none;
}
html, body {
  font-family: sans-serif;
  background: #eee;
  color: #444;
  margin: 0;
  padding: 0;
}

.app {
  display: flex;
  flex-direction: column;
  height: 100vh;
  padding: 20px;
}
.main {
  flex: 1;
  display: flex;
}
aside {
  display: flex;
  flex-direction: column;
  width: 25vw;
  margin: -10px 0 0 20px;
}
h2 {
  font-size: inherit;
  margin-top: 20px;
  margin-bottom: 10px;
}


.bricks {
  position: relative;
  justify-self: stretch;
  display: grid;
  grid-gap: 2px;
  flex: 1;
  user-select: none;
}
.bricks svg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
.box rect:first-child {
  fill: none;
  transition: fill 0.3s;
}
.bricks:focus .box.selected rect:first-child {
  fill: rgba(0, 112, 204, 0.2);
}
.bricks.show-wires:not(:focus) .box rect:first-child {
  fill: rgba(221, 221, 221, 0) !important;
}
.inner-box {
  fill: rgba(255, 255, 255, 0);
  stroke: rgba(68, 68, 68, 0);
  stroke-width: 0.01;
  transition: fill 0.3s, stroke 0.3s;
}
.inner-box-text {
  font-size: .4px;
  font-weight: bold;
  transition: fill 0.3s;
}
.bricks.show-wires .inner-box {
  fill: white;
  stroke: #444;
}
.bricks.show-wires .inner-box-text {
  fill: #444;
}
.selection {
  fill: none;
  stroke: rgba(112, 112, 112, 0);
  stroke-width: 0.02;
  pointer-events: none;
  transition: stroke 0.3s;
}
.bricks:focus .selection {
  stroke: rgba(0, 112, 204, 0.7);
}
.object {
  stroke-width: 0;
  transition: opacity 0.3s;
  font-family: 'Times New Roman', Times, serif;
  font-style: italic;
  font-size: 0.13px;
}
.object.invalid {
  fill: rgba(204, 0, 0, 1);
}
.show-wires path {
  opacity: 0;
}
.object.equal.valid span.output {
  display: none;
}
.show-wires .object.valid::before {
  opacity: 0;
}
.line {
  stroke: #444;
  stroke-width: 0.01;
  fill: none;
  transition: opacity 0.3s, stroke 0.3s, stroke-width 0.3s;
}
.object.invalid .line {
  stroke: rgba(204, 0, 0, 1);
}
.line:hover {
  stroke-width: 0.02;
  stroke: #888;
}
.show-wires .line {
  opacity: 1;
}
.show-bricks .box .line {
  opacity: 0;
}
.bricks:focus::after {
  content: 'Press alt to toggle between bricks and wires.';
  position: absolute;
  right: 0;
  top: 100%;
  font-family: 'Times New Roman', Times, serif;
  font-style: italic;
  padding-top: 5px;
}
.wires rect:first-child {
  fill: none;
}
.bricks:focus .wires.selected rect:first-child {
  fill: rgba(0, 112, 204, 0.1);
}

.fieldset {
  display: grid;
  grid-template-columns: auto 1fr;
  grid-gap: 10px;
  align-items: center;
}
.inputs {
  display: grid;
  grid-gap: 0 10px;
}
textarea, input {
  border: none;
  border-radius: 3px;
  width: 100%;
  font-family: 'Fira Code', 'Courier New', Courier, monospace;
  font-size: 14px;
  font-weight: 500;
  color: inherit;
  padding: 8px;
}
textarea {
  flex: 1;
}


.term {
  line-height: 20px;
  margin-top: -10px;
}
.tc, .tt {
  display: flex;
  border-top: 20px solid rgba(0, 0, 0, .1);
  border-radius: 5px;
  transition: background-color 0.3s;
}
.selected.tc, .selected.tt, .selected .tc, .selected .tt {
  border-top-color: rgba(0, 112, 204, 0.3);
}
.term > .tc, .term > .tt {
  border-top: none;
}
.term span, .term i {
  display: inline-block;
  background: rgba(0, 0, 0, .2);
  color: white;
  width: 20px;
  border-radius: 5px;
  text-align: center;
  font-family: 'Times New Roman', Times, serif;
  transition: background-color 0.3s;
}
.term .selected span, .term .selected i { 
  background: rgba(0, 112, 204, 0.7);
  color: white;
}
.term i {
  background: rgba(0, 0, 0, .1);
  color: inherit;
}
.term div {
  margin: 5px 1px 0 1px;
}

.flex-container {
  display: flex;
  flex-wrap: nowrap;
  align-items: center;
}

.equals {
  width: 100%;
  text-align: center;
}
</style>

Lately there has been a lot of hype around lenses and optics in general, both in the functional programming community and in the category theory community, especially because they are popping up in several different places. This suggest they have an important role that is yet to be completely understood.

One thing that struck me while reading about lenses is the fact that functional programmers tend to consider only lawful lenses, while academics are interested more broadly in generic lenses. Moreover, optics laws, altough very meaningful in practice, appear as something God-given and it is hard to understand where they do come from. Additionally, laws are usually defined only for simple lenses, while it seems reasonable to expect that generic lenses also obey some coherence conditions.

This is why at the [Statebox summit](https://summit.statebox.org/) I was very curious to talk to other people more skilled than me about lawful generic lenses. It turned out this was a really interesting thing to do, since it seems we now have a good idea on how to solve the above issues. I really need to thank [Jules Hedges](https://twitter.com/_julesh_), [David Spivak](https://math.mit.edu/~dspivak/), [Brendan Fong](http://brendanfong.com/), [Christina Vasilakopoulou](https://mathdept.ucr.edu/faculty/cvasil.html), [Eliana Lorch](https://twitter.com/elianalorch), [Andre Knispel](https://whatisrt.github.io/) and everyone else who was there for contributing to what follows.

## Lens recap

Let's review briefly what a `Lens` is. It can be seen as a way to access and update a part of an immutable data structure `S`, through two functions

```
data Lens S T A B = Lens
  { g : S -> A
  , p : S -> B -> T
  }
```

For example we could define a lens `cc : Lens (A x M) (B x M) A B` defined by

```
cc : Lens (A x M) (B x M) A B
cc = lens get put
  where
    get : A x M -> A
    get (a, m) = a

    put : (A x M) -> B -> (B x M)
    put (a, m) b = (b, m)
```

We will call every lens of this form a `constant complement` lens.

We will call a lens `Simple` if `t = s` and `b = a`, that is if `put` does not have the ability to change types, but only to update the value of type `a` inside `s`.

#### Lens laws

Traditionally the lens laws are stated for simple lenses as follows:

```
getput
```

<div class="flex-container" style="height: 200px;">
  <div>
    <svg viewBox="-0.01 -0.01 3.02 2.02" style="height: 200px;"><g class="wires selected"><rect x="0.005" y="0.005" width="0.99" height="1.99" rx="0.07" class=""></rect><g class="object valid"><path class="line" d="M0.0,1.0 C0.25,1.0,0.25,1.0,0.5,1.0"></path><text x="0.05" y="0.95" text-anchor="start" class="object valid">S</text><path d="M-0.001,0.96 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g><g class="object valid"><path class="line" d="M1.0,0.5 C0.75,0.5,0.75,0.9875,0.5,0.9875"></path><text x="0.95" y="0.45" text-anchor="end" class="object valid"></text></g><g class="object valid"><path class="line" d="M1.0,1.5 C0.75,1.5,0.75,1.0125,0.5,1.0125"></path><text x="0.95" y="1.45" text-anchor="end" class="object valid"></text></g><circle cx="0.5" cy="1.0" r="0.05" class="node black"></circle></g><g class="wires "><rect x="1.005" y="0.005" width="0.99" height="0.99" rx="0.07" class=""></rect><path d="M1.0,0.5 C1.5,0.5,1.5,0.5,2.0,0.5" class="line"></path><text x="1.95" y="0.45" text-anchor="end" class="object valid"></text></g><g class="box "><rect x="1.005" y="1.005" width="0.99" height="0.99" rx="0.07" class=""></rect><rect x="1.32" y="1.25" width="0.36" height="0.5" class="inner-box"></rect><text x="1.5" y="1.62" text-anchor="middle" class="inner-box-text">g</text><g class="object valid"><path class="line" d="M1.0,1.5 C1.1600000000000001,1.5,1.1600000000000001,1.5,1.32,1.5"></path></g><g class="object valid"><path class="line" d="M2.0,1.5 C1.8399999999999999,1.5,1.8399999999999999,1.5,1.68,1.5"></path><text x="1.95" y="1.45" text-anchor="end" class="object valid"></text><path d="M2.001,1.46 a0.04,0.04,180.0,1,0,0.0,0.08"></path></g></g><g class="box "><rect x="2.005" y="0.005" width="0.99" height="1.99" rx="0.07" class=""></rect><rect x="2.32" y="0.75" width="0.36" height="0.5" class="inner-box"></rect><text x="2.5" y="1.12" text-anchor="middle" class="inner-box-text">p</text><g class="object valid"><path class="line" d="M2.0,0.5 C2.16,0.5,2.16,0.925,2.32,0.925"></path></g><g class="object valid"><path class="line" d="M2.0,1.5 C2.16,1.5,2.16,1.075,2.32,1.075"></path><text x="2.05" text-anchor="start" class="object valid" y="1.65">A</text><path d="M1.999,1.46 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g><g class="object valid"><path class="line" d="M3.0,1.0 C2.84,1.0,2.84,1.0,2.68,1.0"></path><text x="2.95" y="0.95" text-anchor="end" class="object valid">S</text><path d="M3.001,0.96 a0.04,0.04,180.0,1,0,0.0,0.08"></path></g></g><rect x="0.005" y="0.005" width="0.99" height="0.99" rx="0.07" class="selection"></rect></svg>
  </div>
  <div class="equals"> = </div>
  <div>
    <svg style="height: 140px;" viewBox="-0.01 -0.01 1.02 1.02"><g class="box selected"><rect x="0.005" y="0.005" width="0.99" height="0.99" rx="0.07" class=""></rect><rect x="0.32" y="0.25" width="0.36" height="0.5" class="inner-box"></rect><text x="0.5" y="0.62" text-anchor="middle" class="inner-box-text">i</text><g class="object valid"><path class="line" d="M0.0,0.5 C0.18,0.5,0.18,0.5,0.32,0.5"></path><text x="0.05" y="0.45" text-anchor="start" class="object valid">s</text><path d="M-0.001,0.46 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g><g class="object valid"><path class="line" d="M1.0,0.5 C0.8200000000000001,0.5,0.8200000000000001,0.5,0.6799999999999999,0.5"></path><text x="0.95" y="0.45" text-anchor="end" class="object valid">s</text><path d="M1.001,0.46 a0.04,0.04,180.0,1,0,0.0,0.08"></path></g></g><rect x="0.005" y="0.005" width="0.99" height="0.99" rx="0.07" class="selection"></rect></svg>
  </div>
</div>

Here `• : s -> (s, s)` is the function that produces two copies of the input, and `i : s -> s` is the identity on `s`.

Alternatively, we could rewrite this as `p s (g s) = s`.

```
putget
```
<div class="flex-container" style="height: 200px;">
  <div>
    <svg style="height: 200px;" viewBox="-0.01 -0.01 2.02 2.02"><g class="box selected"><rect x="0.005" y="0.005" width="0.99" height="1.99" rx="0.07" class=""></rect><rect x="0.32" y="0.75" width="0.36" height="0.5" class="inner-box"></rect><text x="0.5" y="1.12" text-anchor="middle" class="inner-box-text">p</text><g class="object valid"><path class="line" d="M0.0,0.5 C0.16,0.5,0.16,0.925,0.32,0.925"></path><text x="0.05" y="0.45" text-anchor="start" class="object valid">S</text><path d="M-0.001,0.46 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g><g class="object valid"><path class="line" d="M0.0,1.5 C0.16,1.5,0.16,1.075,0.32,1.075"></path><text x="0.05" text-anchor="start" class="object valid" y="1.65">A</text><path d="M-0.001,1.46 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g><g class="object valid"><path class="line" d="M1.0,1.5 C0.84,1.5,0.84,1.075,0.6799999999999999,1.075"></path><text x="0.95" y="1.45" text-anchor="end" class="object valid"></text><path d="M1.001,1.46 a0.04,0.04,180.0,1,0,0.0,0.08"></path></g></g><g class="box "><rect x="1.005" y="1.005" width="0.99" height="0.99" rx="0.07" class=""></rect><rect x="1.32" y="1.25" width="0.36" height="0.5" class="inner-box"></rect><text x="1.5" y="1.62" text-anchor="middle" class="inner-box-text">g</text><g class="object valid"><path class="line" d="M1.0,1.5 C1.1600000000000001,1.5,1.1600000000000001,1.5,1.32,1.5"></path><text x="1.05" y="1.45" text-anchor="start" class="object valid">S</text><path d="M0.999,1.46 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g><g class="object valid"><path class="line" d="M2.0,1.5 C1.8399999999999999,1.5,1.8399999999999999,1.5,1.68,1.5"></path><text x="1.95" y="1.45" text-anchor="end" class="object valid">A</text><path d="M2.001,1.46 a0.04,0.04,180.0,1,0,0.0,0.08"></path></g></g><rect x="0.005" y="0.005" width="0.99" height="0.99" rx="0.07" class="selection"></rect></svg>
  </div>
  <div class="equals"> = </div>
  <div>
    <svg style="height: 200px" viewBox="-0.01 -0.01 1.02 2.02"><g class="box selected"><rect x="0.005" y="0.005" width="0.99" height="0.99" rx="0.07" class=""></rect><rect x="0.32" y="0.25" width="0.36" height="0.5" class="inner-box"></rect><text x="0.5" y="0.62" text-anchor="middle" class="inner-box-text">◦</text><g class="object valid"><path class="line" d="M0.0,0.5 C0.16,0.5,0.16,0.5,0.32,0.5"></path><text x="0.05" y="0.45" text-anchor="start" class="object valid">S</text><path d="M-0.001,0.46 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g></g><g class="box "><rect x="0.005" y="1.005" width="0.99" height="0.99" rx="0.07" class=""></rect><rect x="0.32" y="1.25" width="0.36" height="0.5" class="inner-box"></rect><text x="0.5" y="1.62" text-anchor="middle" class="inner-box-text">i</text><g class="object valid"><path class="line" d="M0.0,1.5 C0.16,1.5,0.16,1.5,0.32,1.5"></path><text x="0.05" y="1.45" text-anchor="start" class="object valid">A</text><path d="M-0.001,1.46 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g><g class="object valid"><path class="line" d="M1.0,1.5 C0.84,1.5,0.84,1.5,0.6799999999999999,1.5"></path><text x="0.95" y="1.45" text-anchor="end" class="object valid">A</text><path d="M1.001,1.46 a0.04,0.04,180.0,1,0,0.0,0.08"></path></g></g><rect x="0.005" y="0.005" width="0.99" height="0.99" rx="0.07" class="selection"></rect></svg>
  </div>
</div>

Here, `◦` means that we are not using its argument `s` and `i : a -> a` is the identity on `a`.

This could be rewritten as `g (p s a) = a`

```
putput
```
<div class="flex-container" style="height: 200px;">
  <div>
    <svg style="height: 200px" viewBox="-0.01 -0.01 2.02 2.02"><g class="box selected"><rect x="0.005" y="0.005" width="0.99" height="0.99" rx="0.07" class=""></rect><rect x="0.32" y="0.25" width="0.36" height="0.5" class="inner-box"></rect><text x="0.5" y="0.62" text-anchor="middle" class="inner-box-text">p</text><g class="object valid"><path class="line" d="M0.0,0.25 C0.16,0.25,0.16,0.425,0.32,0.425"></path><text x="0.05" y="0.2" text-anchor="start" class="object valid">S</text><path d="M-0.001,0.21 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g><g class="object valid"><path class="line" d="M0.0,0.75 C0.16,0.75,0.16,0.575,0.32,0.575"></path><text x="0.05" text-anchor="start" class="object valid" y="0.85">A</text><path d="M-0.001,0.71 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g><g class="object valid"><path class="line" d="M1.0,0.5 C0.84,0.5,0.84,0.5,0.6799999999999999,0.5"></path><text x="0.95" y="0.45" text-anchor="end" class="object valid"></text><path d="M1.001,0.46 a0.04,0.04,180.0,1,0,0.0,0.08"></path></g></g><g class="box "><rect x="0.005" y="1.005" width="0.99" height="0.99" rx="0.07" class=""></rect><rect x="0.32" y="1.25" width="0.36" height="0.5" class="inner-box"></rect><text x="0.5" y="1.62" text-anchor="middle" class="inner-box-text">i</text><g class="object valid"><path class="line" d="M0.0,1.5 C0.16,1.5,0.16,1.5,0.32,1.5"></path><text x="0.05" text-anchor="start" class="object valid" y="1.65">A</text><path d="M-0.001,1.46 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g><g class="object valid"><path class="line" d="M1.0,1.5 C0.84,1.5,0.84,1.5,0.6799999999999999,1.5"></path><text x="0.95" y="1.45" text-anchor="end" class="object valid"></text><path d="M1.001,1.46 a0.04,0.04,180.0,1,0,0.0,0.08"></path></g></g><g class="box "><text y="1.12" text-anchor="middle" class="inner-box-text" x="1.5">p</text><g class="object valid"><path class="line" d="M1.0,0.5 C1.16,0.5,1.16,0.925,1.32,0.925"></path><text y="0.45" text-anchor="start" class="object valid" x="1.05">S</text><path d="M0.999,0.46 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g><g class="object valid"><path class="line" d="M1.0,1.5 C1.16,1.5,1.16,1.075,1.32,1.075"></path><text text-anchor="start" class="object valid" x="1.05" y="1.65">A</text><path d="M0.999,1.46 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g><g class="object valid"><path class="line" d="M2.0,1.0 C1.84,1.0,1.84,1.0,1.68,1.0"></path><text y="0.95" text-anchor="end" class="object valid" x="1.95">S</text><path d="M2.001,0.96 a0.04,0.04,180.0,1,0,0.0,0.08"></path></g></g></svg>
  </div>
  <div class="equals"> = </div>
  <div>
    <svg style="height: 200px;" viewBox="-0.01 -0.01 2.02 3.02"><g class="wires "><rect x="0.005" y="0.005" width="0.99" height="0.99" rx="0.07" class=""></rect><path d="M0.0,0.5 C0.5,0.5,0.5,0.5,1.0,0.5" class="line"></path><text x="0.05" y="0.45" text-anchor="start" class="object valid">S</text><path d="M-0.001,0.46 a0.04,0.04,180.0,1,1,0.0,0.08"></path><text x="0.95" y="0.45" text-anchor="end" class="object valid"></text></g><g class="box selected"><rect x="0.005" y="1.005" width="0.99" height="0.99" rx="0.07" class=""></rect><rect x="0.32" y="1.25" width="0.36" height="0.5" class="inner-box"></rect><text x="0.5" y="1.62" text-anchor="middle" class="inner-box-text">◦</text><g class="object valid"><path class="line" d="M0.0,1.5 C0.16,1.5,0.16,1.5,0.32,1.5"></path><text x="0.05" y="1.45" text-anchor="start" class="object valid">A</text><path d="M-0.001,1.46 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g></g><g class="wires "><rect x="0.005" y="2.005" width="0.99" height="0.99" rx="0.07" class=""></rect><path d="M0.0,2.5 C0.5,2.5,0.5,2.5,1.0,2.5" class="line"></path><text x="0.05" y="2.45" text-anchor="start" class="object valid">A</text><path d="M-0.001,2.46 a0.04,0.04,180.0,1,1,0.0,0.08"></path><text x="0.95" y="2.45" text-anchor="end" class="object valid"></text></g><g class="box "><rect x="1.005" y="0.005" width="0.99" height="2.99" rx="0.07" class=""></rect><rect x="1.32" y="1.25" width="0.36" height="0.5" class="inner-box"></rect><text x="1.5" y="1.62" text-anchor="middle" class="inner-box-text">p</text><g class="object valid"><path class="line" d="M1.0,0.5 C1.1600000000000001,0.5,1.1600000000000001,1.4,1.32,1.4"></path></g><g class="object valid"><path class="line" d="M1.0,2.5 C1.1600000000000001,2.5,1.1600000000000001,1.6,1.32,1.6"></path></g><g class="object valid"><path class="line" d="M2.0,1.5 C1.8399999999999999,1.5,1.8399999999999999,1.5,1.68,1.5"></path><text x="1.95" y="1.45" text-anchor="end" class="object valid">S</text><path d="M2.001,1.46 a0.04,0.04,180.0,1,0,0.0,0.08"></path></g></g><rect x="0.005" y="1.005" width="0.99" height="0.99" rx="0.07" class="selection"></rect></svg>
  </div>
</div>

Here `◦` means that we are not using its argument `a`.

This could be rewritten as `p (p s a) = p s`.

We say that a simple lens is `lawful` or `very well-behaved` if it satisfies the three laws above.

#### Ramblings on lens laws

I've always felt a bit bothered by the three lens laws because, though they make a lot of sense, they still feel a lot like God-given rules without a clear mathematical explanation.

Moreover, as presented above, we are able to state them only for simple lenses, and it feels pretty weird that general lenses do not need to satisfy any rule at all!

Also, lawful lenses seem to be of interest only for programmers, while in academic papers (with some exceptions such as [this one](https://arxiv.org/abs/1809.00738)) lawful lenses don't seem to be considered to be very interesting, and generic lenses are usually investigated instead.

Before trying to understand something more about generic lawful lenses, let's have a look at an easier example of optics, namely `adapters`.

## Generic lawful adapters

An `adapter` is the simplest of all optics and it encodes the fact that we can adapt data between two different formats. It is defined as

```
data Adapter S T A B = Adapter
  { f : S -> A
  , t : B -> T
  }
```

An adapter is said to be `simple` if `S = T` and `A = B`. A simple adapter is `lawful` if `f` and `t` are each other's inverses, i.e. `f (t a) = a` and `t (f s) = s` for any `a : A` and `s : S`.

From this point of view, it is clear that we cannot generalise this definition for generic adapters, because we have no way of composing `f` and `t` in the generic case.

On the other hand we can easily interpret a simple adapter as follows: it enables us to adapt functions `A -> A` to functions `S -> S`, and vice versa, using the isomorphism provided by `f` and `t`.

But now we can try to generalize this new point of view to generic adapters: we want a generic adapter to be lawful if it gives us a way to translate functions `A -> B` to functions `S -> T` in an isomorphic way, meaning that we can adapt functions `A -> B` to functions `S -> T` and vice versa. Hence, we could give the following definition:

```
A generic adapter is lawful when both f and t are isomorphisms
```

It is now easy to observe that this definition is equivalent to the previous one when we restict to the simple case, so this new definition is a generalization of the traditional one.

## Generalizing to lenses

Now that we were able to state a definition for a generic lawful adapter, let's see if we can apply this insight to other optics.

If we look at the paper [Categories of Optics](https://arxiv.org/abs/1809.00738) by Mitchell Riley, we see that a generic optic is defined as

```
data Optic S T A B = exists M . Optic
  { g : S -> M ⊗ A
  , p : M ⊗ B -> T
  }
```

where `⊗` is a tensor in a symmetric monoidal category. If you don't know what that is, don't worry and interpret it as follows in our relevant examples.

- For lenses `⊗` is the product, so think to `M ⊗ A` as `(M, A)`.
- For prisms `⊗` is the coproduct, or sum. In this case we think to `M ⊗ A` as `Either M A`.
- For adapters `⊗` is just the constant functor, so `M ⊗ A` is just `A`.

We can observe that for adapters this definition coincides exactly with the one we gave above. So, we restate our new interpretation of generic lawful adapters in this new more generic setting: an adapter is lawful if both `g` and `p` are isomorphisms.

However, with this new interpretation our definition can now be expressed for any possible `Optic`, not only for adapters! To state it again in its full generality, we say that

```
An optic o : Optic S T A B is lawful if g and p are isomorphisms
```

If we now interpret this definition for lenses, where `⊗` is the product, we see that we obtain that lawful lenses are all lenses `Lens S T A B` where `S` is isomorphic to `(M x A)` and `T` is isomorphic to `(M x B)`. But, up to isomorphism, this is exactly the definition of `constant complement` lenses we introduced above!

So, to check if our interpretation makes sense, we want to prove that a simple lens is lawful (in the traditional sense, i.e. it respects the three lens laws) if and only if it is constant complement.

## Lawful lenses are constanct complement

It is quite easy to check that a simple constant complement lens respects the three lens laws, so we are just going to prove that a lawful lens is constant complement.

The idea of this proof came to us thanks to this hint from [Edward Kmett](https://twitter.com/kmett):

<blockquote class="twitter-tweet" data-lang="en-gb"><p lang="en" dir="ltr">It is easiest to see (in tweet form) by looking at s -&gt; (a -&gt; s, a) as extracting the complement as an s with an a-sized hole, and the a. The fact that the former is injective (or s is uninhabited) is key to realizing this is actually just a proxy for the unnamed complement.</p>&mdash; Edward Kmett (@kmett) <a href="https://twitter.com/kmett/status/1174313326509092864?ref_src=twsrc%5Etfw">18 September 2019</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


Let's consider a `Lens S S A A`, defined by functions `g : S -> A` and `p : S -> A -> S`, which satisfies `getput`, `putget` and `putput`.

To prove our claim we need to define an `M` such that `S` is isomorphic to `A x M`.

First off, we observe that we can collapse our functions `g` and `p` into a single function

```
g&p : S -> (A, A -> S)
g&p s = (g s, p s)
```

This actually is really interesting, because it is already sending `S` into the product of `A` with something else, where every element of this something else is of the form `p s`. Hence, we could try to define

```
M = { m : A -> S | ∃ s : S . m = p s }
```

At this point let's try to see if we can prove that `S` is isomorphic to `A x M`. Let's start by defining functions

```
f : S -> A x M
f s = (g s, p s)
```

and

```
h : A x M -> S
h (a, m) = m a
```

Next, we would like to prove that these two functions `f` and `h` are each other's inverses.

Let's check first that `f;h` is the identity on `S`. We have

```
(f;h) s = p s g(s)
```

which is equal to `s` by `getput`.

Finally, let's check that `h;f` is the identity on `A x M`.

```
(h;f) (a, m) = (g(m a), p(m a))
```

By the definition of the set `M`, we know that `m = p s` for some `s` and hence

```
(g(m a), p(m a)) = (g(p s a), p(p s a))
```

which is actually equal to `(a, p s) = (a, m)` by `putget` and `putput` respectively.

QED!

## Conclusion

The fact that simple lawful lenses are constant complement lenses is folklore in the functional programming community (e.g. [Lenses embody Products, Prisms embody Sums](https://blog.jle.im/entry/lenses-products-prisms-sums.html)), but a proof of the fact was not easy to find, so I think that writing it down could be useful for future reference.

On the other hand, a definition for a generic lawful optic seems to be not present in the literature, and I think the approach proposed above works really nicely and can help us understand the origin of lens laws.

One interesting question to tackle now would be to understand how this generic notion of lawfulness translates to the profunctor optics encoding.
