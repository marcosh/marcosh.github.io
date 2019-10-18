---
layout: post
title:  "More (categorical) ramblings on lawful lenses"
author: Marco Perone
date: 2019-09-24 08:06:42 +0200
categories: post
tags: functional-programming lens optic
comments: true
pageUrl: '"http://marcosh.github.io/post/2019/09/24/more-lawful-lenses.html"'
pageIdentifier: '"more-lawful-lenses"'
description: "Categorical lawful lenses"
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

h2 code {
  font-size: inherit;
}

.equation {
  margin-bottom : 15px;
}
</style>

In [Lawful lenses]({% post_url 2019-09-18-lawful-lenses %}) we discussed how we could state a definition for generic lawful optics and we proved that lawful lenses are exactly constant complement lenses.

In this post we will revisit the ideas presented there in a more categorical fashion, but focusing on lenses in particular.

What we will present in this post is a series of results which were obtained at the third [Statebox Summit](https://summit.statebox.org/) with the collaboration of the partecipants, out of which I ought to mention [Jules Hedges](https://twitter.com/_julesh_), [David Spivak](https://math.mit.edu/~dspivak/), [Brendan Fong](http://brendanfong.com/), [Christina Vasilakopoulou](https://mathdept.ucr.edu/faculty/cvasil.html), [Eliana Lorch](https://twitter.com/elianalorch) and [Andre Knispel](https://whatisrt.github.io/) for taking active part in this discussion.

## The category of lenses

Let's start by defining the category of lenses. The objects of our category will be pairs of sets $$(S, T)$$,  morphisms between $$(S, T)$$ and $$(A, B)$$ will be lenses of type $$Lens \, S \, T \, A \, B $$, which are described by pairs of functions

$$
g : S \to A \\
p : S \to B \to T
$$


The composition of two morphisms, i.e. two lenses, $$(g, p) : (S, T) \to (A, B)$$ and $$(h, q) : (A, B) \to (C, D)$$ is a new lens $$Lens \, S \, T \, C \, D$$ defined by functions

$$
g_c : S \to C \\
g_c = g\, ; h \\
$$

$$
p_c : S \to D \to T \\
p_c \, s \, d = p \, s \, (q \, (g \, s) \, d)
$$

where $$p_c$$ can be represented by the following diagram

<svg viewBox="-0.01 -0.01 4.02 3.02"><g class="wires "><rect x="0.005" y="0.005" width="0.99" height="1.99" rx="0.07" class=""></rect><g class="object valid"><path class="line" d="M0.0,1.0 C0.25,1.0,0.25,1.0,0.5,1.0"></path><text x="0.05" y="0.95" text-anchor="start" class="object valid">S</text><path d="M-0.001,0.96 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g><g class="object valid"><path class="line" d="M1.0,0.5 C0.75,0.5,0.75,0.9875,0.5,0.9875"></path><text x="0.95" y="0.45" text-anchor="end" class="object valid"></text></g><g class="object valid"><path class="line" d="M1.0,1.5 C0.75,1.5,0.75,1.0125,0.5,1.0125"></path><text x="0.95" y="1.45" text-anchor="end" class="object valid"></text></g><circle cx="0.5" cy="1.0" r="0.05" class="node black"></circle></g><g class="wires "><rect x="0.005" y="2.005" width="0.99" height="0.99" rx="0.07" class=""></rect><path d="M0.0,2.5 C0.5,2.5,0.5,2.5,1.0,2.5" class="line"></path><text x="0.05" text-anchor="start" class="object valid" y="2.65">D</text><path d="M-0.001,2.46 a0.04,0.04,180.0,1,1,0.0,0.08"></path><text x="0.95" y="2.45" text-anchor="end" class="object valid"></text></g><g class="wires "><rect x="1.005" y="0.005" width="0.99" height="0.99" rx="0.07" class=""></rect><path d="M1.0,0.5 C1.5,0.5,1.5,0.5,2.0,0.5" class="line"></path><text x="1.95" y="0.45" text-anchor="end" class="object valid"></text></g><g class="box "><rect x="1.005" y="1.005" width="0.99" height="0.99" rx="0.07" class=""></rect><rect x="1.32" y="1.25" width="0.36" height="0.5" class="inner-box"></rect><text x="1.5" y="1.62" text-anchor="middle" class="inner-box-text">g</text><g class="object valid"><path class="line" d="M1.0,1.5 C1.1600000000000001,1.5,1.1600000000000001,1.5,1.32,1.5"></path></g><g class="object valid"><path class="line" d="M2.0,1.5 C1.8399999999999999,1.5,1.8399999999999999,1.5,1.68,1.5"></path><text x="1.95" y="1.45" text-anchor="end" class="object valid"></text><path d="M2.001,1.46 a0.04,0.04,180.0,1,0,0.0,0.08"></path></g></g><g class="wires "><rect x="1.005" y="2.005" width="0.99" height="0.99" rx="0.07" class=""></rect><path d="M1.0,2.5 C1.5,2.5,1.5,2.5,2.0,2.5" class="line"></path><text x="1.95" y="2.45" text-anchor="end" class="object valid"></text></g><g class="wires "><rect x="2.005" y="0.005" width="0.99" height="0.99" rx="0.07" class=""></rect><path d="M2.0,0.5 C2.5,0.5,2.5,0.5,3.0,0.5" class="line"></path><text x="2.95" y="0.45" text-anchor="end" class="object valid"></text></g><g class="box selected"><rect x="2.005" y="1.005" width="0.99" height="1.99" rx="0.07" class=""></rect><rect x="2.32" y="1.75" width="0.36" height="0.5" class="inner-box"></rect><text x="2.5" y="2.12" text-anchor="middle" class="inner-box-text">q</text><g class="object valid"><path class="line" d="M2.0,1.5 C2.16,1.5,2.16,1.925,2.32,1.925"></path><text x="2.05" y="1.45" text-anchor="start" class="object valid">A</text><path d="M1.999,1.46 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g><g class="object valid"><path class="line" d="M2.0,2.5 C2.16,2.5,2.16,2.075,2.32,2.075"></path></g><g class="object valid"><path class="line" d="M3.0,1.5 C2.84,1.5,2.84,1.925,2.68,1.925"></path><text x="2.95" y="1.45" text-anchor="end" class="object valid"></text><path d="M3.001,1.46 a0.04,0.04,180.0,1,0,0.0,0.08"></path></g></g><g class="box "><rect x="3.005" y="0.005" width="0.99" height="1.99" rx="0.07" class=""></rect><rect x="3.32" y="0.75" width="0.36" height="0.5" class="inner-box"></rect><text x="3.5" y="1.12" text-anchor="middle" class="inner-box-text">p</text><g class="object valid"><path class="line" d="M3.0,0.5 C3.16,0.5,3.16,0.925,3.32,0.925"></path></g><g class="object valid"><path class="line" d="M3.0,1.5 C3.16,1.5,3.16,1.075,3.32,1.075"></path><text x="3.05" text-anchor="start" class="object valid" y="1.65">B</text><path d="M2.999,1.46 a0.04,0.04,180.0,1,1,0.0,0.08"></path></g><g class="object valid"><path class="line" d="M4.0,1.0 C3.84,1.0,3.84,1.0,3.68,1.0"></path><text x="3.95" y="0.95" text-anchor="end" class="object valid">T</text><path d="M4.001,0.96 a0.04,0.04,180.0,1,0,0.0,0.08"></path></g></g><rect x="2.005" y="1.005" width="0.99" height="1.99" rx="0.07" class="selection"></rect></svg>

where $$\bullet : S \to S \times S$$ is the $$copy$$ operation.

Given any object of our category, which is a pair $$(S, T)$$, the identity morphism on it is the lens $$(g_{id}, p_{id}) : (S, T) \to (S, T)$$ defined by

$$
g_{id} : S \to S \\
g_{id} \, s = s
$$

$$
p_{id} : S \to T \to T \\
p_{id} \, s \, t = t
$$

It can be checked that the above definitions satisfy the axioms for a category.

## Associated lenses

Given a generic lens $$L := (g, p) : (S, T) \to (A, B)$$, we saw in [Lawful lenses]({% post_url 2019-09-18-lawful-lenses %}) that we can define a set

$$
M = \{ m \in B \to T \, | \, \exists \, s : S . \; m = p \, s \}
$$

which turns out to be really helpful in proving that lawful lenses are exactly the constant complement ones. In fact, this set allows us to define two new lenses associated to the lens $$L$$.

First off, we can define $$\tilde{L} : (S, T) \to (A \times M, B \times M)$$ by

$$
\tilde{g} : S \to A \times M \\
\tilde{g}\, s = (g \, s, p \, s)
$$

$$
\tilde{p} : S \to B \times M \to T \\
\tilde{p} \, s \, (b, m) = m \, b
$$

Moreover we can also define $$\hat{L} : (B \times M, A \times M) \to (T, S)$$ by

$$
\hat{g} : B \times M \to T \\
\hat{g} \, (b, m) = m \, b
$$

$$
\hat{p} : B \times M \to S \to A \times M \\
\hat{p} \, (b, m) s = (g \, s, p \, s)
$$

Notice how ^ (pronounced "hat") produces a swap between $$A$$ and $$B$$ and between $$S$$ and $$T$$. In their generic form, $$\tilde{L}$$ and $$\hat{L}$$ are not composable. However, if we restrict them to simple lenses, they are!

So, with these two new lenses at hand, some questions arise naturally:

- In the simple lens case, when are $$\tilde{L}$$ and $$\hat{L}$$ inverse of one another?
- In the simple lens case, when are $$\tilde{L}$$ and $$\hat{L}$$ lawful?
- In the generic case, if we consider $$\pi : (A \times M, B \times M) \to (A, B)$$ to be the lens defined by

  $$
  g_{\pi} : A \times M \to A \\
  g_{\pi} \, a \, m = a
  $$

  $$
  p_{\pi} : A \times M \to B \to B \times M \\
  p_{\pi} \, (a, m) \, b = (b, m)
  $$

  is it always true that $$L = \tilde{L} \,  ; \pi$$?

## Are $$\tilde{L}$$ and $$\hat{L}$$ inverses?

Let us first consider the composition $$\tilde{L} \, ; \hat{L}$$. In this case we have

$$
g_{\hat{\sim}} : S \to S \\
g_{\hat{\sim}} \, s = p \, s \, g(s) = s \textrm{ --- by getput}
$$

$$
p_{\hat{\sim}} : S \to S \to S \\
p_{\hat{\sim}} \, s_1 \, s_2 = p \, s_2 \, (g \, s_2) = s_2 \textrm{ --- by getput}
$$

Therefore $$\tilde{L} \, ; \hat{L} = id_{(S, S)}$$ if and only if $$getput$$ holds for $$L$$.

On the other hand, consider $$\hat{L} \, ; \tilde{L}$$. We obtain

$$
g_{\tilde{\wedge}} : A \times M \to A \times M \\
g_{\tilde{\wedge}} \, (a, m) = (g \, (m \, a), p \, (m \, a))
$$

$$
p_{\tilde{\wedge}} : A \times M \to A \times M \to A \times M \\
p_{\tilde{\wedge}} \, (a_1, m_1) \, (a_2, m_2) = (g \, (m_2 \, a_2), p \, (m_2 \, a_2))
$$

Remember that by definition of $$M$$ we have that any $$m \in M$$ is of the form $$p \, s$$ for some $$s \in S$$. Hence we get

$$
g_{\tilde{\wedge}} : A \times M \to A \times M \\
\begin{aligned}
g_{\tilde{\wedge}} (a, p \, s) &= (g \, (p \, s \, a), p \, (p \, s \, a)) \\
                               &= (a, p \, s) \textrm{ --- by putget and putput, respectively}
\end{aligned}
$$

$$
p_{\tilde{\wedge}} : A \times M \to A \times M \to A \times M \\
\begin{aligned}
p_{\tilde{\wedge}} (a_1, p \, s_1) (a_2, p \, s_2) &= (g \, (p \, s_2 \, a_2), p \, (p \, s_2 \, a_2)) \\
                                                   &= (a_2, p \, s_2) \textrm{ --- by putget and putput, respectively}
\end{aligned}
$$

We then obtain that $$\hat{L} \, ; \tilde{L} = id_{(M \times A, M \times A)} $$ whenever $$L$$ satisfies $$putget$$ and $$putput$$.

We conclude by saying that $$\tilde{L}$$ and $$\hat{L}$$ are each other's inverses if and only if $$L$$ is a lawful lens.

## Are $$\tilde{L}$$ and $$\hat{L}$$ lawful?

In this section we will check whether $$\tilde{L}$$ and $$\hat{L}$$ are lawful lenses in the simple case, and which conditions we do need to assume on $$L$$ for them to satisfy the lens laws.

Let's start with $$\tilde{L}$$

#### getput:

$$
\tilde{p} \, s \, (\tilde{g} \, s) = p \, s \, (g \, s) = s \textrm{ --- by getput}
$$

#### putget:

$$
\begin{aligned}
\tilde{g} \, (\tilde{p} \, s_1 \, (a, p \, s_2)) &= \tilde{g} \, (p\, s_2 \, a) \\
                                                 &= (g \, (p \, s_2 \, a), p \, (p \, s_2 \, a)) \\
                                                 &= (a, p \, s_2) \textrm{ --- by putget and putput}
\end{aligned}
$$

#### putput:

$$
\begin{aligned}
\tilde{p}\, (\tilde{p} \, s \, (a_1, m_1)) \, (a_2, m_2) &= \tilde{p} \, (m_1 \, a_1) \, (a_2, m_2) \\
                                                         &= m_2 \, a_2 \\
                                                         &= \tilde{p} \, s \, (a_2, m_2) \textrm{ --- always holds}
\end{aligned}
$$

Hence $$\tilde{L}$$ always satisfies $$putput$$, satisfies $$getput$$ if $$L$$ satisfies $$getput$$ and satisfies $$putget$$ if $$L$$ satisfies $$putget$$ and $$putput$$. Therefore $$\tilde{L}$$ is lawful if and only if $$L$$ is.

Let's now have a look at $$\hat{L}$$

#### getput:

$$
\begin{aligned}
\hat{p} \, (a, p \, s) \, (\hat{g}(a, p \, s)) &= \hat{p} \, (a, p \, s)(p \, s \, a) \\
                                               &= (g \, (p \, s \, a), p \, (p \, s \, a)) \\
                                               &= (a, p \, s) \textrm { --- by putget and putput, respectively}
\end{aligned}
$$

#### putget:

$$
\begin{aligned}
\hat{g} \, (\hat{p} \, (a, m) \, s) &= \hat{g} \, (g \, s, p \, s) \\
                                    &= p \, s (g \, s) \\
                                    &= s \textrm{ --- by getput}
\end{aligned}    
$$

#### putput:

$$
\begin{aligned}
\hat{p} \, (\hat{p} \, (a, m) \, s_1) \, s_2 &= \hat{p} \, (g \, s_1, p \, s_1) \, s_2 \\
                                             &= (g \, s_2, p \, s_2) \\
                                             &= \hat{p} \, (a, m) \, s_2 \textrm{ --- always}
\end{aligned}                     
$$

Therefore also $$\hat{L}$$ always satisfies $$putput$$, and it satisfies $$getput$$ if $$L$$ satisfies $$putget$$ and $$putput$$ and satisfies $$putget$$ if $$L$$ satisfies $$getput$$. So also $$\hat{L}$$ is lawful if and only if $$L$$ is.

## Decomposing $$L$$

One question remains: whether $$L = \tilde{L} \, ; \pi$$ holds. It is not hard to check that it actually always does; in fact if we compute $$g^{\circ}$$ and $$p^{\circ}$$ for the composed lens, we get

$$
g^{\circ} : S \to A \\
g^{\circ} s = g_{\pi} \, (\tilde{g} \, s) = g_{\pi} \, (g \, s, p \, s) = g \, s
$$

$$
p^{\circ} : S \to B \to T \\
\begin{aligned}
p^{\circ} \, s \, b &= \tilde{p} \, (s, p_{\pi} \, (\tilde{g} \, s, b)) \\
                    &= \tilde{p} \, (s, p_{\pi} \, ((g \, s, p \, s), b)) \\
                    &= \tilde{p} \, (s, (b, p \, s)) = p \, s \, b
\end{aligned}      
$$

In conclusion, we can always decompose $$L$$ into $$\tilde{L} \, ; \pi$$, where $$\pi$$ is always lawful.

## Conclusion

In this post we investigated the role of $$M$$ more closely, as well as the lenses $$\tilde{L}$$ and $$\hat{L}$$ which are generated from it. We saw that $$\tilde{L}$$ and $$\hat{L}$$ are strictly related to $$L$$, and that they satisfy the lens laws only if $$L$$ satisfies other lens laws.

Putting together the three results presented above, we can also recover a more formal proof that a simple lens is lawful if and only if it is isomorphic to a constant complement lens.

Some further questions we might consider:
- what is $$M$$ for $$\tilde{L}$$ and $$\hat{L}$$?
- how are $$\tilde{\tilde{L}}$$, $$\hat{\tilde{L}}$$, $$\tilde{\hat{L}}$$ and $$\hat{\hat{L}}$$ related to $$L$$, $$\tilde{L}$$ and $$\hat{L}$$?
- is there a relation between $$M$$ being a singleton set and isomorphisms in the category of lenses (it is easy to prove that $$M = \{*\}$$ implies that $$L$$ satisfies $$putput$$)?
- are the properties stated for the simple case generalizable to the general case?
