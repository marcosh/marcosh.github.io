---
layout: post
title:  "The Mondrian introduction to functional optics"
author: Marco Perone
date: 2025-10-07 08:06:42 +0200
categories: post
tags: haskell lens prism optics functional
comments: true
pageUrl: '"http://marcosh.github.io/post/2025/10/07/the-mondrian-introduction-to-functional-optics.html"'
pageIdentifier: '"The Mondrian introduction to functional optics"'
description: "The Mondrian introduction to functional optics"
image: "/img/mondrian.jpg"
---

In this post I’d like to try to discuss _what_ functional optics are, without going too much into _why_ they are so cool, and you should use them, or _how_ they are implemented[^1] and should be used with a specific language and library.

<br>

I personally think that functional optics should be a really easy concept to grasp, but currently learning them is harder than it should be mostly due to library implementation details, quite obscure documentation and an exotic usage of weird symbols.

<br>

Since a picture is worth a thousand words, I will introduce and use a graphical notation to illustrate the concepts we will discuss.

## Types and values

Let's start introducing our graphical notation from its basic building blocks.

<br>

We can represent a type with a simple coloured rectangle

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="121px" height="121px" viewBox="-0.5 -0.5 121 121" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.0.3&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;KFOvKJ2CKSVV_86AEirZ&quot;&gt;
    &lt;mxGraphModel dx=&quot;1426&quot; dy=&quot;827&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;2&quot; value=&quot;&quot; style=&quot;whiteSpace=wrap;html=1;aspect=fixed;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;200&quot; width=&quot;120&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="lRH7b3wcrNEJ4w8In_7t-1"><g><rect x="0" y="0" width="120" height="120" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g></g></g></g></svg>

A value for a given type will be represented as a horizontal line spanning the width of the rectangle

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="122px" height="121px" viewBox="-0.5 -0.5 122 121" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.0.3&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;CzpSF7XIA-RuL_gY3PdR&quot;&gt;
    &lt;mxGraphModel dx=&quot;1426&quot; dy=&quot;827&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;lRH7b3wcrNEJ4w8In_7t-1&quot; value=&quot;&quot; style=&quot;whiteSpace=wrap;html=1;aspect=fixed;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;200&quot; width=&quot;120&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;lRH7b3wcrNEJ4w8In_7t-4&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0;exitY=0.75;exitDx=0;exitDy=0;entryX=1;entryY=0.75;entryDx=0;entryDy=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;lRH7b3wcrNEJ4w8In_7t-1&quot; target=&quot;lRH7b3wcrNEJ4w8In_7t-1&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;400&quot; y=&quot;460&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;450&quot; y=&quot;410&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="lRH7b3wcrNEJ4w8In_7t-1"><g><rect x="0" y="0" width="120" height="120" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="lRH7b3wcrNEJ4w8In_7t-4"><g><path d="M 0 90 L 120 90" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g></g></g></g></svg>

## Sums and products

When considering algebraic data types, we have two ways of combining types, using products and sums.

<br>

The product of two types `A` and `B` is a new type, which we will denote by `A*B`, whose values are composed of a value of type `A` and a value of type `B`. An example of a product type is a tuple like `(Int, String)` where each value is pair composed of an integer and a string.

<br>

Graphically we can represent a product type as two side by side rectangles

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="241px" height="121px" viewBox="-0.5 -0.5 241 121" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.0.7&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;CzpSF7XIA-RuL_gY3PdR&quot;&gt;
    &lt;mxGraphModel dx=&quot;1426&quot; dy=&quot;827&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;lRH7b3wcrNEJ4w8In_7t-1&quot; value=&quot;&quot; style=&quot;whiteSpace=wrap;html=1;aspect=fixed;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;200&quot; width=&quot;120&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;m6WJKWTpqjJhG-RztG1W-1&quot; value=&quot;&quot; style=&quot;whiteSpace=wrap;html=1;aspect=fixed;fillColor=#f8cecc;strokeColor=#b85450;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;200&quot; width=&quot;120&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="lRH7b3wcrNEJ4w8In_7t-1"><g><rect x="0" y="0" width="120" height="120" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="m6WJKWTpqjJhG-RztG1W-1"><g><rect x="120" y="0" width="120" height="120" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g></g></g></g></svg>

When it comes to values, we need to upgrade a little bit our graphical interpretation. Since a value in a product type is composed of values of its components, we will just represent it as piecewise horizontal line, composed by horizontal lines (possible at different heights) spanning its horizontal sub-rectangles.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="242px" height="121px" viewBox="-0.5 -0.5 242 121" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.0.7&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;CzpSF7XIA-RuL_gY3PdR&quot;&gt;
    &lt;mxGraphModel dx=&quot;1426&quot; dy=&quot;827&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;lRH7b3wcrNEJ4w8In_7t-1&quot; value=&quot;&quot; style=&quot;whiteSpace=wrap;html=1;aspect=fixed;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;200&quot; width=&quot;120&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;m6WJKWTpqjJhG-RztG1W-1&quot; value=&quot;&quot; style=&quot;whiteSpace=wrap;html=1;aspect=fixed;fillColor=#f8cecc;strokeColor=#b85450;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;200&quot; width=&quot;120&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;m6WJKWTpqjJhG-RztG1W-3&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0;exitY=0.75;exitDx=0;exitDy=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;lRH7b3wcrNEJ4w8In_7t-1&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;400&quot; y=&quot;400&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;400&quot; y=&quot;290&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;m6WJKWTpqjJhG-RztG1W-4&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=1;exitY=0.25;exitDx=0;exitDy=0;entryX=1;entryY=0.25;entryDx=0;entryDy=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;lRH7b3wcrNEJ4w8In_7t-1&quot; target=&quot;m6WJKWTpqjJhG-RztG1W-1&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;370&quot; y=&quot;300&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;420&quot; y=&quot;250&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="lRH7b3wcrNEJ4w8In_7t-1"><g><rect x="0" y="0" width="120" height="120" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="m6WJKWTpqjJhG-RztG1W-1"><g><rect x="120" y="0" width="120" height="120" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="m6WJKWTpqjJhG-RztG1W-3"><g><path d="M 0 90 L 120 90" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g><g data-cell-id="m6WJKWTpqjJhG-RztG1W-4"><g><path d="M 120 30 L 240 30" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g></g></g></g></svg>

On the other hand, the sum of two types is represented by two rectangles one on top of the other

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="121px" height="241px" viewBox="-0.5 -0.5 121 241" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.1&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;CzpSF7XIA-RuL_gY3PdR&quot;&gt;
    &lt;mxGraphModel dx=&quot;1426&quot; dy=&quot;827&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;lRH7b3wcrNEJ4w8In_7t-1&quot; value=&quot;&quot; style=&quot;whiteSpace=wrap;html=1;aspect=fixed;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;200&quot; width=&quot;120&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;m6WJKWTpqjJhG-RztG1W-1&quot; value=&quot;&quot; style=&quot;whiteSpace=wrap;html=1;aspect=fixed;fillColor=#f8cecc;strokeColor=#b85450;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;320&quot; width=&quot;120&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="lRH7b3wcrNEJ4w8In_7t-1"><g><rect x="0" y="0" width="120" height="120" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="m6WJKWTpqjJhG-RztG1W-1"><g><rect x="0" y="120" width="120" height="120" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g></g></g></g></svg>

A value of a sum type is a horizontal line spanning the width of the whole rectangle. If it is a horizontal line in the top rectangle, it means that we are selecting the first type, and we're using one of its values.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="122px" height="241px" viewBox="-0.5 -0.5 122 241" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.1&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;CzpSF7XIA-RuL_gY3PdR&quot;&gt;
    &lt;mxGraphModel dx=&quot;570&quot; dy=&quot;331&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;lRH7b3wcrNEJ4w8In_7t-1&quot; value=&quot;&quot; style=&quot;whiteSpace=wrap;html=1;aspect=fixed;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;200&quot; width=&quot;120&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;m6WJKWTpqjJhG-RztG1W-1&quot; value=&quot;&quot; style=&quot;whiteSpace=wrap;html=1;aspect=fixed;fillColor=#f8cecc;strokeColor=#b85450;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;320&quot; width=&quot;120&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-1&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0;exitY=0.5;exitDx=0;exitDy=0;entryX=1;entryY=0.5;entryDx=0;entryDy=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;lRH7b3wcrNEJ4w8In_7t-1&quot; target=&quot;lRH7b3wcrNEJ4w8In_7t-1&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;370&quot; y=&quot;320&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;420&quot; y=&quot;270&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="lRH7b3wcrNEJ4w8In_7t-1"><g><rect x="0" y="0" width="120" height="120" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="m6WJKWTpqjJhG-RztG1W-1"><g><rect x="0" y="120" width="120" height="120" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-1"><g><path d="M 0 60 L 120 60" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g></g></g></g></svg>

If it is a horizontal line in the bottom rectangle, it means that we are selecting the second type and one of its values.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="122px" height="241px" viewBox="-0.5 -0.5 122 241" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.1&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;CzpSF7XIA-RuL_gY3PdR&quot;&gt;
    &lt;mxGraphModel dx=&quot;570&quot; dy=&quot;331&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;lRH7b3wcrNEJ4w8In_7t-1&quot; value=&quot;&quot; style=&quot;whiteSpace=wrap;html=1;aspect=fixed;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;200&quot; width=&quot;120&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;m6WJKWTpqjJhG-RztG1W-1&quot; value=&quot;&quot; style=&quot;whiteSpace=wrap;html=1;aspect=fixed;fillColor=#f8cecc;strokeColor=#b85450;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;320&quot; width=&quot;120&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-1&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0;exitY=0.5;exitDx=0;exitDy=0;entryX=1;entryY=0.5;entryDx=0;entryDy=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;280&quot; y=&quot;379.8&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;400&quot; y=&quot;379.8&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="lRH7b3wcrNEJ4w8In_7t-1"><g><rect x="0" y="0" width="120" height="120" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="m6WJKWTpqjJhG-RztG1W-1"><g><rect x="0" y="120" width="120" height="120" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-1"><g><path d="M 0 179.8 L 120 179.8" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g></g></g></g></svg>

More generally, for any algebraic data type, we can represent it as a sum of products by stacking a series of rectangles one on top of the other, each one potentially divided horizontally in multiple sub-rectangles.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="161px" height="181px" viewBox="-0.5 -0.5 161 181" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.1&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;CzpSF7XIA-RuL_gY3PdR&quot;&gt;
    &lt;mxGraphModel dx=&quot;570&quot; dy=&quot;331&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;300&quot; width=&quot;160&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-7&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;340&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-8&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fad7ac;strokeColor=#b46504;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;360&quot; width=&quot;40&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="hVsSfDapXN-nuvVv81QE-3"><g><rect x="0" y="0" width="80" height="60" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-4"><g><rect x="0" y="60" width="160" height="60" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-5"><g><rect x="80" y="0" width="80" height="60" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-6"><g><rect x="0" y="120" width="60" height="60" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-7"><g><rect x="60" y="120" width="60" height="60" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-8"><g><rect x="120" y="120" width="40" height="60" fill="#fad7ac" style="fill: rgb(250, 215, 172); stroke: rgb(180, 101, 4);" stroke="#b46504" pointer-events="all"></rect></g></g></g></g></g></svg>

In general, we can continue to split any sub-rectangle horizontally or vertically (if you prefer a top-down point of view) or you can place two rectangles side by side or top to bottom.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="161px" height="181px" viewBox="-0.5 -0.5 161 181" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.1&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;CzpSF7XIA-RuL_gY3PdR&quot;&gt;
    &lt;mxGraphModel dx=&quot;570&quot; dy=&quot;331&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;300&quot; width=&quot;70&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-7&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;340&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-8&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fad7ac;strokeColor=#b46504;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;360&quot; width=&quot;40&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-9&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;260&quot; width=&quot;80&quot; height=&quot;10&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-10&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f5f5f5;fontColor=#333333;strokeColor=#666666;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-11&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;390&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-12&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;380&quot; width=&quot;40&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-13&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;gradientColor=#7ea6e0;strokeColor=#6c8ebf;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;350&quot; y=&quot;300&quot; width=&quot;90&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-14&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e6d0de;gradientColor=#d5739d;strokeColor=#996185;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;270&quot; width=&quot;80&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs><linearGradient x1="0%" y1="0%" x2="0%" y2="100%" id="drawio-svg-VwDdBPCmGy-U6pHX-kjV-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0"><stop offset="0%" stop-color="#dae8fc" style="stop-color: rgb(218, 232, 252); stop-opacity: 1;" stop-opacity="1"></stop><stop offset="100%" stop-color="#7ea6e0" style="stop-color: rgb(126, 166, 224); stop-opacity: 1;" stop-opacity="1"></stop></linearGradient><linearGradient x1="0%" y1="0%" x2="0%" y2="100%" id="drawio-svg-VwDdBPCmGy-U6pHX-kjV-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0"><stop offset="0%" stop-color="#e6d0de" style="stop-color: rgb(230, 208, 222); stop-opacity: 1;" stop-opacity="1"></stop><stop offset="100%" stop-color="#d5739d" style="stop-color: rgb(213, 115, 157); stop-opacity: 1;" stop-opacity="1"></stop></linearGradient></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="hVsSfDapXN-nuvVv81QE-3"><g><rect x="0" y="0" width="80" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-4"><g><rect x="0" y="60" width="70" height="60" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-5"><g><rect x="80" y="0" width="80" height="20" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-6"><g><rect x="0" y="120" width="60" height="30" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-7"><g><rect x="60" y="120" width="60" height="60" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-8"><g><rect x="120" y="120" width="40" height="20" fill="#fad7ac" style="fill: rgb(250, 215, 172); stroke: rgb(180, 101, 4);" stroke="#b46504" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-9"><g><rect x="80" y="20" width="80" height="10" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-10"><g><rect x="0" y="40" width="80" height="20" fill="#f5f5f5" style="fill: rgb(245, 245, 245); stroke: rgb(102, 102, 102);" stroke="#666666" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-11"><g><rect x="0" y="150" width="60" height="30" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-12"><g><rect x="120" y="140" width="40" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-13"><g><rect x="70" y="60" width="90" height="60" fill="url(#drawio-svg-VwDdBPCmGy-U6pHX-kjV-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0)" style="fill: url(&quot;#drawio-svg-VwDdBPCmGy-U6pHX-kjV-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0&quot;); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-14"><g><rect x="80" y="30" width="80" height="30" fill="url(#drawio-svg-VwDdBPCmGy-U6pHX-kjV-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0)" style="fill: url(&quot;#drawio-svg-VwDdBPCmGy-U6pHX-kjV-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0&quot;); stroke: rgb(153, 97, 133);" stroke="#996185" pointer-events="all"></rect></g></g></g></g></g></svg>

A value of such a type is a piecewise horizontal line which can not cross a horizontal division.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="162px" height="181px" viewBox="-0.5 -0.5 162 181" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.1&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;CzpSF7XIA-RuL_gY3PdR&quot;&gt;
    &lt;mxGraphModel dx=&quot;475&quot; dy=&quot;276&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;300&quot; width=&quot;70&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-7&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;340&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-8&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fad7ac;strokeColor=#b46504;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;360&quot; width=&quot;40&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-9&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;260&quot; width=&quot;80&quot; height=&quot;10&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-10&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f5f5f5;fontColor=#333333;strokeColor=#666666;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-11&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;390&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-12&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;380&quot; width=&quot;40&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-13&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;gradientColor=#7ea6e0;strokeColor=#6c8ebf;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;350&quot; y=&quot;300&quot; width=&quot;90&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-14&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e6d0de;gradientColor=#d5739d;strokeColor=#996185;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;270&quot; width=&quot;80&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-18&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0;exitY=0.25;exitDx=0;exitDy=0;entryX=1;entryY=0.25;entryDx=0;entryDy=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;hVsSfDapXN-nuvVv81QE-6&quot; target=&quot;hVsSfDapXN-nuvVv81QE-6&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;290&quot; y=&quot;380&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;340&quot; y=&quot;330&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-19&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0;exitY=0.5;exitDx=0;exitDy=0;entryX=1;entryY=0.5;entryDx=0;entryDy=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;hVsSfDapXN-nuvVv81QE-7&quot; target=&quot;hVsSfDapXN-nuvVv81QE-7&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;340&quot; y=&quot;410&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;390&quot; y=&quot;360&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-20&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0;exitY=0.75;exitDx=0;exitDy=0;entryX=1;entryY=0.75;entryDx=0;entryDy=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;hVsSfDapXN-nuvVv81QE-12&quot; target=&quot;hVsSfDapXN-nuvVv81QE-12&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;340&quot; y=&quot;410&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;390&quot; y=&quot;360&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs><linearGradient x1="0%" y1="0%" x2="0%" y2="100%" id="drawio-svg-TmCbqGQn5Sf7Y4LRaOB3-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0"><stop offset="0%" stop-color="#dae8fc" style="stop-color: rgb(218, 232, 252); stop-opacity: 1;" stop-opacity="1"></stop><stop offset="100%" stop-color="#7ea6e0" style="stop-color: rgb(126, 166, 224); stop-opacity: 1;" stop-opacity="1"></stop></linearGradient><linearGradient x1="0%" y1="0%" x2="0%" y2="100%" id="drawio-svg-TmCbqGQn5Sf7Y4LRaOB3-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0"><stop offset="0%" stop-color="#e6d0de" style="stop-color: rgb(230, 208, 222); stop-opacity: 1;" stop-opacity="1"></stop><stop offset="100%" stop-color="#d5739d" style="stop-color: rgb(213, 115, 157); stop-opacity: 1;" stop-opacity="1"></stop></linearGradient></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="hVsSfDapXN-nuvVv81QE-3"><g><rect x="0" y="0" width="80" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-4"><g><rect x="0" y="60" width="70" height="60" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-5"><g><rect x="80" y="0" width="80" height="20" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-6"><g><rect x="0" y="120" width="60" height="30" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-7"><g><rect x="60" y="120" width="60" height="60" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-8"><g><rect x="120" y="120" width="40" height="20" fill="#fad7ac" style="fill: rgb(250, 215, 172); stroke: rgb(180, 101, 4);" stroke="#b46504" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-9"><g><rect x="80" y="20" width="80" height="10" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-10"><g><rect x="0" y="40" width="80" height="20" fill="#f5f5f5" style="fill: rgb(245, 245, 245); stroke: rgb(102, 102, 102);" stroke="#666666" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-11"><g><rect x="0" y="150" width="60" height="30" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-12"><g><rect x="120" y="140" width="40" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-13"><g><rect x="70" y="60" width="90" height="60" fill="url(#drawio-svg-TmCbqGQn5Sf7Y4LRaOB3-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0)" style="fill: url(&quot;#drawio-svg-TmCbqGQn5Sf7Y4LRaOB3-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0&quot;); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-14"><g><rect x="80" y="30" width="80" height="30" fill="url(#drawio-svg-TmCbqGQn5Sf7Y4LRaOB3-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0)" style="fill: url(&quot;#drawio-svg-TmCbqGQn5Sf7Y4LRaOB3-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0&quot;); stroke: rgb(153, 97, 133);" stroke="#996185" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-18"><g><path d="M 0 127.5 L 60 127.5" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-19"><g><path d="M 60 150 L 120 150" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-20"><g><path d="M 120 170 L 160 170" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g></g></g></g></svg>

## Optics

Now that we have this graphical representation to represent data types, we can use it to discuss various kinds of optics.

<br>

In general, we can think of an optic as a way to select, given our graphical representation of a type, one (or more) rectangle inside a given rectangle representing a type. For example in the following picture we are selecting the rectangle with the red boundary inside the main rectangle representing a complex type.

<br>

If call the main type `A` and the selected type `B`, we will denote the optic selecting `B` inside `A` with `Optic A B`.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="163px" height="181px" viewBox="-0.5 -0.5 163 181" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.1&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;CzpSF7XIA-RuL_gY3PdR&quot;&gt;
    &lt;mxGraphModel dx=&quot;396&quot; dy=&quot;230&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;300&quot; width=&quot;70&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-7&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;340&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-8&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fad7ac;strokeColor=#b46504;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;360&quot; width=&quot;40&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-9&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;260&quot; width=&quot;80&quot; height=&quot;10&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-10&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f5f5f5;fontColor=#333333;strokeColor=#666666;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-11&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;390&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-12&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;380&quot; width=&quot;40&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-14&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e6d0de;gradientColor=#d5739d;strokeColor=#996185;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;270&quot; width=&quot;80&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-13&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;gradientColor=#7ea6e0;strokeColor=#FF3333;strokeWidth=3;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;350&quot; y=&quot;300&quot; width=&quot;90&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs><linearGradient x1="0%" y1="0%" x2="0%" y2="100%" id="drawio-svg-NIXjyHQY85LdUeDxcsZw-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0"><stop offset="0%" stop-color="#e6d0de" style="stop-color: rgb(230, 208, 222); stop-opacity: 1;" stop-opacity="1"></stop><stop offset="100%" stop-color="#d5739d" style="stop-color: rgb(213, 115, 157); stop-opacity: 1;" stop-opacity="1"></stop></linearGradient><linearGradient x1="0%" y1="0%" x2="0%" y2="100%" id="drawio-svg-NIXjyHQY85LdUeDxcsZw-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0"><stop offset="0%" stop-color="#dae8fc" style="stop-color: rgb(218, 232, 252); stop-opacity: 1;" stop-opacity="1"></stop><stop offset="100%" stop-color="#7ea6e0" style="stop-color: rgb(126, 166, 224); stop-opacity: 1;" stop-opacity="1"></stop></linearGradient></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="hVsSfDapXN-nuvVv81QE-3"><g><rect x="0" y="0" width="80" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-4"><g><rect x="0" y="60" width="70" height="60" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-5"><g><rect x="80" y="0" width="80" height="20" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-6"><g><rect x="0" y="120" width="60" height="30" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-7"><g><rect x="60" y="120" width="60" height="60" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-8"><g><rect x="120" y="120" width="40" height="20" fill="#fad7ac" style="fill: rgb(250, 215, 172); stroke: rgb(180, 101, 4);" stroke="#b46504" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-9"><g><rect x="80" y="20" width="80" height="10" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-10"><g><rect x="0" y="40" width="80" height="20" fill="#f5f5f5" style="fill: rgb(245, 245, 245); stroke: rgb(102, 102, 102);" stroke="#666666" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-11"><g><rect x="0" y="150" width="60" height="30" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-12"><g><rect x="120" y="140" width="40" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-14"><g><rect x="80" y="30" width="80" height="30" fill="url(#drawio-svg-NIXjyHQY85LdUeDxcsZw-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0)" style="fill: url(&quot;#drawio-svg-NIXjyHQY85LdUeDxcsZw-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0&quot;); stroke: rgb(153, 97, 133);" stroke="#996185" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-13"><g><rect x="70" y="60" width="90" height="60" fill="url(#drawio-svg-NIXjyHQY85LdUeDxcsZw-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0)" style="fill: url(&quot;#drawio-svg-NIXjyHQY85LdUeDxcsZw-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0&quot;); stroke: rgb(255, 51, 51);" stroke="#ff3333" stroke-width="3" pointer-events="all"></rect></g></g></g></g></g></svg>

Before going into inspecting the various kinds of optics, let's try to see if can can already derive some properties of optics just by looking at their graphical representation.

## Compositionality

One thing that we can notice is that optics compose really well. Suppose we have a type `A` represented by the following diagram

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="161px" height="181px" viewBox="-0.5 -0.5 161 181" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.1&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;CzpSF7XIA-RuL_gY3PdR&quot;&gt;
    &lt;mxGraphModel dx=&quot;570&quot; dy=&quot;331&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;300&quot; width=&quot;70&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-7&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;340&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-8&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fad7ac;strokeColor=#b46504;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;360&quot; width=&quot;40&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-9&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;260&quot; width=&quot;80&quot; height=&quot;10&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-10&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f5f5f5;fontColor=#333333;strokeColor=#666666;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-11&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;390&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-12&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;380&quot; width=&quot;40&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-13&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;gradientColor=#7ea6e0;strokeColor=#6c8ebf;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;350&quot; y=&quot;300&quot; width=&quot;90&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-14&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e6d0de;gradientColor=#d5739d;strokeColor=#996185;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;270&quot; width=&quot;80&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs><linearGradient x1="0%" y1="0%" x2="0%" y2="100%" id="drawio-svg-VwDdBPCmGy-U6pHX-kjV-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0"><stop offset="0%" stop-color="#dae8fc" style="stop-color: rgb(218, 232, 252); stop-opacity: 1;" stop-opacity="1"></stop><stop offset="100%" stop-color="#7ea6e0" style="stop-color: rgb(126, 166, 224); stop-opacity: 1;" stop-opacity="1"></stop></linearGradient><linearGradient x1="0%" y1="0%" x2="0%" y2="100%" id="drawio-svg-VwDdBPCmGy-U6pHX-kjV-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0"><stop offset="0%" stop-color="#e6d0de" style="stop-color: rgb(230, 208, 222); stop-opacity: 1;" stop-opacity="1"></stop><stop offset="100%" stop-color="#d5739d" style="stop-color: rgb(213, 115, 157); stop-opacity: 1;" stop-opacity="1"></stop></linearGradient></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="hVsSfDapXN-nuvVv81QE-3"><g><rect x="0" y="0" width="80" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-4"><g><rect x="0" y="60" width="70" height="60" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-5"><g><rect x="80" y="0" width="80" height="20" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-6"><g><rect x="0" y="120" width="60" height="30" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-7"><g><rect x="60" y="120" width="60" height="60" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-8"><g><rect x="120" y="120" width="40" height="20" fill="#fad7ac" style="fill: rgb(250, 215, 172); stroke: rgb(180, 101, 4);" stroke="#b46504" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-9"><g><rect x="80" y="20" width="80" height="10" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-10"><g><rect x="0" y="40" width="80" height="20" fill="#f5f5f5" style="fill: rgb(245, 245, 245); stroke: rgb(102, 102, 102);" stroke="#666666" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-11"><g><rect x="0" y="150" width="60" height="30" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-12"><g><rect x="120" y="140" width="40" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-13"><g><rect x="70" y="60" width="90" height="60" fill="url(#drawio-svg-VwDdBPCmGy-U6pHX-kjV-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0)" style="fill: url(&quot;#drawio-svg-VwDdBPCmGy-U6pHX-kjV-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0&quot;); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-14"><g><rect x="80" y="30" width="80" height="30" fill="url(#drawio-svg-VwDdBPCmGy-U6pHX-kjV-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0)" style="fill: url(&quot;#drawio-svg-VwDdBPCmGy-U6pHX-kjV-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0&quot;); stroke: rgb(153, 97, 133);" stroke="#996185" pointer-events="all"></rect></g></g></g></g></g></svg>

We can first select a sub-rectangle identifying a type `B` with an `Optic A B`.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="164px" height="183px" viewBox="-0.5 -0.5 164 183" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.1&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;CzpSF7XIA-RuL_gY3PdR&quot;&gt;
    &lt;mxGraphModel dx=&quot;396&quot; dy=&quot;230&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;300&quot; width=&quot;70&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-7&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;340&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-8&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fad7ac;strokeColor=#b46504;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;360&quot; width=&quot;40&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-9&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;260&quot; width=&quot;80&quot; height=&quot;10&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-10&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f5f5f5;fontColor=#333333;strokeColor=#666666;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-11&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;390&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-12&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;380&quot; width=&quot;40&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-13&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;gradientColor=#7ea6e0;strokeColor=#6c8ebf;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;350&quot; y=&quot;300&quot; width=&quot;90&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-14&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e6d0de;gradientColor=#d5739d;strokeColor=#996185;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;270&quot; width=&quot;80&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;nXzzlKf4BNkx2kQisM7c-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeWidth=3;strokeColor=#ed333b;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;360&quot; width=&quot;160&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs><linearGradient x1="0%" y1="0%" x2="0%" y2="100%" id="drawio-svg-yM4yL1Ateo9qNM92byQG-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0"><stop offset="0%" stop-color="#dae8fc" style="stop-color: rgb(218, 232, 252); stop-opacity: 1;" stop-opacity="1"></stop><stop offset="100%" stop-color="#7ea6e0" style="stop-color: rgb(126, 166, 224); stop-opacity: 1;" stop-opacity="1"></stop></linearGradient><linearGradient x1="0%" y1="0%" x2="0%" y2="100%" id="drawio-svg-yM4yL1Ateo9qNM92byQG-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0"><stop offset="0%" stop-color="#e6d0de" style="stop-color: rgb(230, 208, 222); stop-opacity: 1;" stop-opacity="1"></stop><stop offset="100%" stop-color="#d5739d" style="stop-color: rgb(213, 115, 157); stop-opacity: 1;" stop-opacity="1"></stop></linearGradient></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="hVsSfDapXN-nuvVv81QE-3"><g><rect x="1" y="0" width="80" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-4"><g><rect x="1" y="60" width="70" height="60" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-5"><g><rect x="81" y="0" width="80" height="20" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-6"><g><rect x="1" y="120" width="60" height="30" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-7"><g><rect x="61" y="120" width="60" height="60" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-8"><g><rect x="121" y="120" width="40" height="20" fill="#fad7ac" style="fill: rgb(250, 215, 172); stroke: rgb(180, 101, 4);" stroke="#b46504" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-9"><g><rect x="81" y="20" width="80" height="10" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-10"><g><rect x="1" y="40" width="80" height="20" fill="#f5f5f5" style="fill: rgb(245, 245, 245); stroke: rgb(102, 102, 102);" stroke="#666666" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-11"><g><rect x="1" y="150" width="60" height="30" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-12"><g><rect x="121" y="140" width="40" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-13"><g><rect x="71" y="60" width="90" height="60" fill="url(#drawio-svg-yM4yL1Ateo9qNM92byQG-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0)" style="fill: url(&quot;#drawio-svg-yM4yL1Ateo9qNM92byQG-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0&quot;); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-14"><g><rect x="81" y="30" width="80" height="30" fill="url(#drawio-svg-yM4yL1Ateo9qNM92byQG-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0)" style="fill: url(&quot;#drawio-svg-yM4yL1Ateo9qNM92byQG-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0&quot;); stroke: rgb(153, 97, 133);" stroke="#996185" pointer-events="all"></rect></g></g><g data-cell-id="nXzzlKf4BNkx2kQisM7c-4"><g><rect x="1" y="120" width="160" height="60" fill="none" stroke="#ed333b" style="stroke: rgb(237, 51, 59);" stroke-width="3" pointer-events="all"></rect></g></g></g></g></g></svg>

Starting now with the type `B` we can use an `Optic B C` to select a type `C` inside `B`.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="163px" height="64px" viewBox="-0.5 -0.5 163 64" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.1&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;CzpSF7XIA-RuL_gY3PdR&quot;&gt;
    &lt;mxGraphModel dx=&quot;396&quot; dy=&quot;230&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-7&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;340&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-8&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fad7ac;strokeColor=#b46504;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;360&quot; width=&quot;40&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-11&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;390&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-12&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;380&quot; width=&quot;40&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;nXzzlKf4BNkx2kQisM7c-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeWidth=3;strokeColor=#FF3333;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;360&quot; width=&quot;40&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="hVsSfDapXN-nuvVv81QE-6"><g><rect x="0" y="1" width="60" height="30" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-7"><g><rect x="60" y="1" width="60" height="60" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-8"><g><rect x="120" y="1" width="40" height="20" fill="#fad7ac" style="fill: rgb(250, 215, 172); stroke: rgb(180, 101, 4);" stroke="#b46504" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-11"><g><rect x="0" y="31" width="60" height="30" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-12"><g><rect x="120" y="21" width="40" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="nXzzlKf4BNkx2kQisM7c-5"><g><rect x="120" y="1" width="40" height="60" fill="none" stroke="#ff3333" style="stroke: rgb(255, 51, 51);" stroke-width="3" pointer-events="all"></rect></g></g></g></g></g></svg>

Using now the `Optic A B` and the `Optic B C` we just chose, we can compose them to obtain an `Optic A C` which directly selects `C` inside `A`.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="163px" height="183px" viewBox="-0.5 -0.5 163 183" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.1&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;CzpSF7XIA-RuL_gY3PdR&quot;&gt;
    &lt;mxGraphModel dx=&quot;396&quot; dy=&quot;230&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;300&quot; width=&quot;70&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-7&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;340&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-8&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fad7ac;strokeColor=#b46504;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;360&quot; width=&quot;40&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-9&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;260&quot; width=&quot;80&quot; height=&quot;10&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-10&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f5f5f5;fontColor=#333333;strokeColor=#666666;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-11&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;390&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-12&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;380&quot; width=&quot;40&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-13&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;gradientColor=#7ea6e0;strokeColor=#6c8ebf;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;350&quot; y=&quot;300&quot; width=&quot;90&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-14&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e6d0de;gradientColor=#d5739d;strokeColor=#996185;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;270&quot; width=&quot;80&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;nXzzlKf4BNkx2kQisM7c-8&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeColor=#ed333b;strokeWidth=3;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;360&quot; width=&quot;40&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs><linearGradient x1="0%" y1="0%" x2="0%" y2="100%" id="drawio-svg-puV3vYn8efrCt4M2mSXX-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0"><stop offset="0%" stop-color="#dae8fc" style="stop-color: rgb(218, 232, 252); stop-opacity: 1;" stop-opacity="1"></stop><stop offset="100%" stop-color="#7ea6e0" style="stop-color: rgb(126, 166, 224); stop-opacity: 1;" stop-opacity="1"></stop></linearGradient><linearGradient x1="0%" y1="0%" x2="0%" y2="100%" id="drawio-svg-puV3vYn8efrCt4M2mSXX-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0"><stop offset="0%" stop-color="#e6d0de" style="stop-color: rgb(230, 208, 222); stop-opacity: 1;" stop-opacity="1"></stop><stop offset="100%" stop-color="#d5739d" style="stop-color: rgb(213, 115, 157); stop-opacity: 1;" stop-opacity="1"></stop></linearGradient></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="hVsSfDapXN-nuvVv81QE-3"><g><rect x="0" y="0" width="80" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-4"><g><rect x="0" y="60" width="70" height="60" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-5"><g><rect x="80" y="0" width="80" height="20" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-6"><g><rect x="0" y="120" width="60" height="30" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-7"><g><rect x="60" y="120" width="60" height="60" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-8"><g><rect x="120" y="120" width="40" height="20" fill="#fad7ac" style="fill: rgb(250, 215, 172); stroke: rgb(180, 101, 4);" stroke="#b46504" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-9"><g><rect x="80" y="20" width="80" height="10" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-10"><g><rect x="0" y="40" width="80" height="20" fill="#f5f5f5" style="fill: rgb(245, 245, 245); stroke: rgb(102, 102, 102);" stroke="#666666" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-11"><g><rect x="0" y="150" width="60" height="30" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-12"><g><rect x="120" y="140" width="40" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-13"><g><rect x="70" y="60" width="90" height="60" fill="url(#drawio-svg-puV3vYn8efrCt4M2mSXX-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0)" style="fill: url(&quot;#drawio-svg-puV3vYn8efrCt4M2mSXX-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0&quot;); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-14"><g><rect x="80" y="30" width="80" height="30" fill="url(#drawio-svg-puV3vYn8efrCt4M2mSXX-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0)" style="fill: url(&quot;#drawio-svg-puV3vYn8efrCt4M2mSXX-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0&quot;); stroke: rgb(153, 97, 133);" stroke="#996185" pointer-events="all"></rect></g></g><g data-cell-id="nXzzlKf4BNkx2kQisM7c-8"><g><rect x="120" y="120" width="40" height="60" fill="none" stroke="#ed333b" style="stroke: rgb(237, 51, 59);" stroke-width="3" pointer-events="all"></rect></g></g></g></g></g></svg>

This optic composition operation is actually associative and has an identity element, turning optics into a [Category](https://en.wikipedia.org/wiki/Category_(mathematics)).

<br>

Let's now start to have a look at some specific families of optics.

## Iso

The simplest optic we can define for any type `A` is the one that we can obtain by selecting the whole rectangle.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="163px" height="183px" viewBox="-0.5 -0.5 163 183" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.1&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;CzpSF7XIA-RuL_gY3PdR&quot;&gt;
    &lt;mxGraphModel dx=&quot;570&quot; dy=&quot;331&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;300&quot; width=&quot;70&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-7&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;340&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-8&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fad7ac;strokeColor=#b46504;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;360&quot; width=&quot;40&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-9&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;260&quot; width=&quot;80&quot; height=&quot;10&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-10&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f5f5f5;fontColor=#333333;strokeColor=#666666;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-11&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;390&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-12&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;380&quot; width=&quot;40&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-13&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;gradientColor=#7ea6e0;strokeColor=#6c8ebf;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;350&quot; y=&quot;300&quot; width=&quot;90&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-14&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e6d0de;gradientColor=#d5739d;strokeColor=#996185;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;270&quot; width=&quot;80&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;nXzzlKf4BNkx2kQisM7c-8&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeColor=#ed333b;strokeWidth=3;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;240&quot; width=&quot;160&quot; height=&quot;180&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs><linearGradient x1="0%" y1="0%" x2="0%" y2="100%" id="drawio-svg-sZxeAOTvKrQW_k22FRUr-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0"><stop offset="0%" stop-color="#dae8fc" style="stop-color: rgb(218, 232, 252); stop-opacity: 1;" stop-opacity="1"></stop><stop offset="100%" stop-color="#7ea6e0" style="stop-color: rgb(126, 166, 224); stop-opacity: 1;" stop-opacity="1"></stop></linearGradient><linearGradient x1="0%" y1="0%" x2="0%" y2="100%" id="drawio-svg-sZxeAOTvKrQW_k22FRUr-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0"><stop offset="0%" stop-color="#e6d0de" style="stop-color: rgb(230, 208, 222); stop-opacity: 1;" stop-opacity="1"></stop><stop offset="100%" stop-color="#d5739d" style="stop-color: rgb(213, 115, 157); stop-opacity: 1;" stop-opacity="1"></stop></linearGradient></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="hVsSfDapXN-nuvVv81QE-3"><g><rect x="1" y="1" width="80" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-4"><g><rect x="1" y="61" width="70" height="60" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-5"><g><rect x="81" y="1" width="80" height="20" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-6"><g><rect x="1" y="121" width="60" height="30" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-7"><g><rect x="61" y="121" width="60" height="60" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-8"><g><rect x="121" y="121" width="40" height="20" fill="#fad7ac" style="fill: rgb(250, 215, 172); stroke: rgb(180, 101, 4);" stroke="#b46504" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-9"><g><rect x="81" y="21" width="80" height="10" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-10"><g><rect x="1" y="41" width="80" height="20" fill="#f5f5f5" style="fill: rgb(245, 245, 245); stroke: rgb(102, 102, 102);" stroke="#666666" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-11"><g><rect x="1" y="151" width="60" height="30" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-12"><g><rect x="121" y="141" width="40" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-13"><g><rect x="71" y="61" width="90" height="60" fill="url(#drawio-svg-sZxeAOTvKrQW_k22FRUr-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0)" style="fill: url(&quot;#drawio-svg-sZxeAOTvKrQW_k22FRUr-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0&quot;); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-14"><g><rect x="81" y="31" width="80" height="30" fill="url(#drawio-svg-sZxeAOTvKrQW_k22FRUr-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0)" style="fill: url(&quot;#drawio-svg-sZxeAOTvKrQW_k22FRUr-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0&quot;); stroke: rgb(153, 97, 133);" stroke="#996185" pointer-events="all"></rect></g></g><g data-cell-id="nXzzlKf4BNkx2kQisM7c-8"><g><rect x="1" y="1" width="160" height="180" fill="none" stroke="#ed333b" style="stroke: rgb(237, 51, 59);" stroke-width="3" pointer-events="all"></rect></g></g></g></g></g></svg>

With such a selection we can see that for any value of the outer type `A`, we actually have a value of the type identified by the red rectangle, which we will call `B`.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="163px" height="183px" viewBox="-0.5 -0.5 163 183" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.1&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;CzpSF7XIA-RuL_gY3PdR&quot;&gt;
    &lt;mxGraphModel dx=&quot;570&quot; dy=&quot;331&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;300&quot; width=&quot;70&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;240&quot; width=&quot;80&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-7&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;340&quot; y=&quot;360&quot; width=&quot;60&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-8&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fad7ac;strokeColor=#b46504;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;360&quot; width=&quot;40&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-9&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;260&quot; width=&quot;80&quot; height=&quot;10&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-10&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f5f5f5;fontColor=#333333;strokeColor=#666666;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;20&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-11&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;390&quot; width=&quot;60&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-12&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;380&quot; width=&quot;40&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-13&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;gradientColor=#7ea6e0;strokeColor=#6c8ebf;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;350&quot; y=&quot;300&quot; width=&quot;90&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;hVsSfDapXN-nuvVv81QE-14&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e6d0de;gradientColor=#d5739d;strokeColor=#996185;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;360&quot; y=&quot;270&quot; width=&quot;80&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;WsW2iOhSiS_W2Il1xUH2-2&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0;exitY=0.75;exitDx=0;exitDy=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;nXzzlKf4BNkx2kQisM7c-8&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;550&quot; y=&quot;430&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;340&quot; y=&quot;375&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;WsW2iOhSiS_W2Il1xUH2-3&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0;exitY=0.75;exitDx=0;exitDy=0;entryX=1;entryY=0.75;entryDx=0;entryDy=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;hVsSfDapXN-nuvVv81QE-7&quot; target=&quot;hVsSfDapXN-nuvVv81QE-7&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;350&quot; y=&quot;385&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;390&quot; y=&quot;395&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;WsW2iOhSiS_W2Il1xUH2-4&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0;exitY=0.25;exitDx=0;exitDy=0;entryX=1;entryY=0.25;entryDx=0;entryDy=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;hVsSfDapXN-nuvVv81QE-12&quot; target=&quot;hVsSfDapXN-nuvVv81QE-12&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;390&quot; y=&quot;380&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;440&quot; y=&quot;330&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;nXzzlKf4BNkx2kQisM7c-8&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeColor=#ed333b;strokeWidth=3;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;280&quot; y=&quot;240&quot; width=&quot;160&quot; height=&quot;180&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs><linearGradient x1="0%" y1="0%" x2="0%" y2="100%" id="drawio-svg-Xaikthn4TMC6FW_89gF1-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0"><stop offset="0%" stop-color="#dae8fc" style="stop-color: rgb(218, 232, 252); stop-opacity: 1;" stop-opacity="1"></stop><stop offset="100%" stop-color="#7ea6e0" style="stop-color: rgb(126, 166, 224); stop-opacity: 1;" stop-opacity="1"></stop></linearGradient><linearGradient x1="0%" y1="0%" x2="0%" y2="100%" id="drawio-svg-Xaikthn4TMC6FW_89gF1-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0"><stop offset="0%" stop-color="#e6d0de" style="stop-color: rgb(230, 208, 222); stop-opacity: 1;" stop-opacity="1"></stop><stop offset="100%" stop-color="#d5739d" style="stop-color: rgb(213, 115, 157); stop-opacity: 1;" stop-opacity="1"></stop></linearGradient></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="hVsSfDapXN-nuvVv81QE-3"><g><rect x="1" y="1" width="80" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-4"><g><rect x="1" y="61" width="70" height="60" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-5"><g><rect x="81" y="1" width="80" height="20" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-6"><g><rect x="1" y="121" width="60" height="30" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-7"><g><rect x="61" y="121" width="60" height="60" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-8"><g><rect x="121" y="121" width="40" height="20" fill="#fad7ac" style="fill: rgb(250, 215, 172); stroke: rgb(180, 101, 4);" stroke="#b46504" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-9"><g><rect x="81" y="21" width="80" height="10" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-10"><g><rect x="1" y="41" width="80" height="20" fill="#f5f5f5" style="fill: rgb(245, 245, 245); stroke: rgb(102, 102, 102);" stroke="#666666" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-11"><g><rect x="1" y="151" width="60" height="30" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-12"><g><rect x="121" y="141" width="40" height="40" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-13"><g><rect x="71" y="61" width="90" height="60" fill="url(#drawio-svg-Xaikthn4TMC6FW_89gF1-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0)" style="fill: url(&quot;#drawio-svg-Xaikthn4TMC6FW_89gF1-gradient-light-dark_dae8fc_1d293b_-1-light-dark_7ea6e0_436697_-1-s-0&quot;); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="hVsSfDapXN-nuvVv81QE-14"><g><rect x="81" y="31" width="80" height="30" fill="url(#drawio-svg-Xaikthn4TMC6FW_89gF1-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0)" style="fill: url(&quot;#drawio-svg-Xaikthn4TMC6FW_89gF1-gradient-light-dark_e6d0de_43303c_-1-light-dark_d5739d_b56185_-1-s-0&quot;); stroke: rgb(153, 97, 133);" stroke="#996185" pointer-events="all"></rect></g></g><g data-cell-id="WsW2iOhSiS_W2Il1xUH2-2"><g><path d="M 1 136 L 61 136" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g><g data-cell-id="WsW2iOhSiS_W2Il1xUH2-3"><g><path d="M 61 166 L 121 166" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g><g data-cell-id="WsW2iOhSiS_W2Il1xUH2-4"><g><path d="M 121 151 L 161 151" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g><g data-cell-id="nXzzlKf4BNkx2kQisM7c-8"><g><rect x="1" y="1" width="160" height="180" fill="none" stroke="#ed333b" style="stroke: rgb(237, 51, 59);" stroke-width="3" pointer-events="all"></rect></g></g></g></g></g></svg>

This means that, given an `Iso A B`, we can actually define a function `view :: A -> B` that for any value of `A` gives us a value of `B`.

<br>

But in this special case also the converse holds! For any value of `B`, since `B` is actually `A` itself, we have in fact a value of `A`. This gives rise to a function `review :: B -> A`.

<br>

In fact `review . view = id_A` and `view . review = id_B` giving rise to a proper [isomorphism](https://en.wikipedia.org/wiki/Isomorphism).

## Lens

In our graphical representation, a `Lens` is a vertical slice of the main rectangle.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="242px" height="163px" viewBox="-0.5 -0.5 242 163" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.2&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;z8OuRh85pukrXDgO7kjt&quot;&gt;
    &lt;mxGraphModel dx=&quot;1426&quot; dy=&quot;827&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-1&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;240&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-2&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;240&quot; y=&quot;360&quot; width=&quot;80&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;320&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;160&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;400&quot; width=&quot;80&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeWidth=3;strokeColor=#ed333b;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;160&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-1"><g><rect x="0" y="1" width="80" height="80" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-2"><g><rect x="0" y="81" width="80" height="80" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-3"><g><rect x="80" y="1" width="80" height="160" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-4"><g><rect x="160" y="1" width="80" height="120" fill="#fff2cc" style="fill: rgb(255, 242, 204); stroke: rgb(214, 182, 86);" stroke="#d6b656" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-5"><g><rect x="160" y="121" width="80" height="40" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-6"><g><rect x="160" y="1" width="80" height="160" fill="none" stroke="#ed333b" style="stroke: rgb(237, 51, 59);" stroke-width="3" pointer-events="all"></rect></g></g></g></g></g></svg>

Any vertical slice cuts out a piece out of any horizontal line. In other terms, given a value of the type `A` represented by the main rectangle, we have a way to obtain a value of type `B` represented by our vertical slice. This means that also in this case we are able to define a function `view :: A -> B` which allows us to focus from the main type to one of its component.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="242px" height="163px" viewBox="-0.5 -0.5 242 163" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.2&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;z8OuRh85pukrXDgO7kjt&quot;&gt;
    &lt;mxGraphModel dx=&quot;1426&quot; dy=&quot;827&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-1&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;240&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-2&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;240&quot; y=&quot;360&quot; width=&quot;80&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;320&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;160&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;400&quot; width=&quot;80&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeWidth=3;strokeColor=#ed333b;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;400&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;160&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-7&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0;exitY=0.5;exitDx=0;exitDy=0;entryX=1;entryY=0.25;entryDx=0;entryDy=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;bTHEDbTsUVzmCIvmqzU7-1&quot; target=&quot;bTHEDbTsUVzmCIvmqzU7-6&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;400&quot; y=&quot;460&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;450&quot; y=&quot;410&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-1"><g><rect x="0" y="1" width="80" height="80" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-2"><g><rect x="0" y="81" width="80" height="80" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-3"><g><rect x="80" y="1" width="80" height="160" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-4"><g><rect x="160" y="1" width="80" height="120" fill="#fff2cc" style="fill: rgb(255, 242, 204); stroke: rgb(214, 182, 86);" stroke="#d6b656" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-5"><g><rect x="160" y="121" width="80" height="40" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-6"><g><rect x="160" y="1" width="80" height="160" fill="none" stroke="#ed333b" style="stroke: rgb(237, 51, 59);" stroke-width="3" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-7"><g><path d="M 0 41 L 240 41" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g></g></g></g></svg>

On the other hand, it's not possible with `Lens`es as it was with `Iso`s to build back a value of type `A` from a value of type `B`, since a value of type `B` is only a part of value of type `A`. What is actually possible, though, is to update only the part included in the red rectangle of a value of type `A`. In other terms, given a `Lens A B`, we can define a function `set :: B -> A -> A` which takes a value of type `B` and a value of type `A` and updates the section of the latter identified by the `Lens`.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="243px" height="564px" viewBox="-0.5 -0.5 243 564" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.2&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;z8OuRh85pukrXDgO7kjt&quot;&gt;
    &lt;mxGraphModel dx=&quot;1240&quot; dy=&quot;719&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-1&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;80&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-2&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;80&quot; y=&quot;360&quot; width=&quot;80&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;160&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;160&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;240&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;240&quot; y=&quot;400&quot; width=&quot;80&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeWidth=3;strokeColor=#ed333b;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;240&quot; y=&quot;280&quot; width=&quot;80&quot; height=&quot;160&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-7&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0;exitY=0.5;exitDx=0;exitDy=0;entryX=1;entryY=0.25;entryDx=0;entryDy=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;bTHEDbTsUVzmCIvmqzU7-1&quot; target=&quot;bTHEDbTsUVzmCIvmqzU7-6&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;240&quot; y=&quot;460&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;290&quot; y=&quot;410&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-8&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;240&quot; y=&quot;480&quot; width=&quot;80&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-9&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;240&quot; y=&quot;600&quot; width=&quot;80&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-10&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0;exitY=0.5;exitDx=0;exitDy=0;entryX=1;entryY=0.5;entryDx=0;entryDy=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;bTHEDbTsUVzmCIvmqzU7-9&quot; target=&quot;bTHEDbTsUVzmCIvmqzU7-9&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;400&quot; y=&quot;460&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;450&quot; y=&quot;410&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-11&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;80&quot; y=&quot;680&quot; width=&quot;80&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-12&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;80&quot; y=&quot;760&quot; width=&quot;80&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-13&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;160&quot; y=&quot;680&quot; width=&quot;80&quot; height=&quot;160&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-14&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;240&quot; y=&quot;680&quot; width=&quot;80&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-15&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;240&quot; y=&quot;800&quot; width=&quot;80&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-19&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0;exitY=0.5;exitDx=0;exitDy=0;entryX=1.008;entryY=0.872;entryDx=0;entryDy=0;entryPerimeter=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;bTHEDbTsUVzmCIvmqzU7-15&quot; target=&quot;bTHEDbTsUVzmCIvmqzU7-16&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;370&quot; y=&quot;760&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;390&quot; y=&quot;740&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-16&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeWidth=3;strokeColor=#ed333b;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;240&quot; y=&quot;680&quot; width=&quot;80&quot; height=&quot;160&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-17&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0;exitY=0.5;exitDx=0;exitDy=0;entryX=0;entryY=0.25;entryDx=0;entryDy=0;&quot; edge=&quot;1&quot; source=&quot;bTHEDbTsUVzmCIvmqzU7-11&quot; target=&quot;bTHEDbTsUVzmCIvmqzU7-16&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;240&quot; y=&quot;860&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;290&quot; y=&quot;810&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-1"><g><rect x="0" y="1" width="80" height="80" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-2"><g><rect x="0" y="81" width="80" height="80" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-3"><g><rect x="80" y="1" width="80" height="160" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-4"><g><rect x="160" y="1" width="80" height="120" fill="#fff2cc" style="fill: rgb(255, 242, 204); stroke: rgb(214, 182, 86);" stroke="#d6b656" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-5"><g><rect x="160" y="121" width="80" height="40" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-6"><g><rect x="160" y="1" width="80" height="160" fill="none" stroke="#ed333b" style="stroke: rgb(237, 51, 59);" stroke-width="3" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-7"><g><path d="M 0 41 L 240 41" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-8"><g><rect x="160" y="201" width="80" height="120" fill="#fff2cc" style="fill: rgb(255, 242, 204); stroke: rgb(214, 182, 86);" stroke="#d6b656" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-9"><g><rect x="160" y="321" width="80" height="40" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-10"><g><path d="M 160 341 L 240 341" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-11"><g><rect x="0" y="401" width="80" height="80" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-12"><g><rect x="0" y="481" width="80" height="80" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-13"><g><rect x="80" y="401" width="80" height="160" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-14"><g><rect x="160" y="401" width="80" height="120" fill="#fff2cc" style="fill: rgb(255, 242, 204); stroke: rgb(214, 182, 86);" stroke="#d6b656" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-15"><g><rect x="160" y="521" width="80" height="40" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-19"><g><path d="M 160 541 L 240.64 540.52" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-16"><g><rect x="160" y="401" width="80" height="160" fill="none" stroke="#ed333b" style="stroke: rgb(237, 51, 59);" stroke-width="3" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-17"><g><path d="M 0 441 L 160 441" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g></g></g></g></svg>

Having a look at the graphical representations of the `view` and `set` functions, we can convince ourselves that the following properties hold:

- If we `set` a value and then we `view` it, we must get back what we put in: `view (set b a) == b`.
- If we `set` what we get out of a `view`, nothing changes: `set (view a) a == a`.
- Setting a value twice is the same thing as setting it once: `set b (set b a) == set b a`.

<br>

Moreover, we can notice that composing two lenses with the operation described in the [Compositionality](#compositionality) section gives us back another lens. A vertical slice of a vertical slice is in fact still a vertical slice of the original rectangle. In other terms this means that `Lens`es form a subcategory of the bigger category of `Optic`s.

<br>

Composing adequately `set` and `view` we can also define a function `over :: (B -> B) -> A -> A` as `over f a = set (f $ view a) a`. This means that if we have a function `f :: B -> B` which can transform values of type `B`, we can use our lens to extract a `B` from an `A` via `view`, use `f` to transform the result, and eventually use `set` to update the `B` part inside the original `A`.

## Prism

If vertical slices are `Lens`es, it is only natural to wonder what are horizontal slices. They correspond to `Prism`s, and they are the dual concept of `Lens`es. Where a `Lens` represents a component in a product type, a `Prism` represents a component in a sum type.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="164px" height="183px" viewBox="-0.5 -0.5 164 183" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.2&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;z8OuRh85pukrXDgO7kjt&quot;&gt;
    &lt;mxGraphModel dx=&quot;1240&quot; dy=&quot;719&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-1&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;rotation=90;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;210&quot; y=&quot;230&quot; width=&quot;60&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-2&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;rotation=90;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;130&quot; y=&quot;230&quot; width=&quot;60&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;rotation=90;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;170&quot; y=&quot;250&quot; width=&quot;60&quot; height=&quot;160&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;rotation=90;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;190&quot; y=&quot;330&quot; width=&quot;60&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;rotation=90;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;110&quot; y=&quot;370&quot; width=&quot;60&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeWidth=3;strokeColor=#ed333b;rotation=90;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;170&quot; y=&quot;310&quot; width=&quot;60&quot; height=&quot;160&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-1"><g><rect x="91" y="-10" width="60" height="80" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" transform="rotate(90,121,30)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-2"><g><rect x="11" y="-10" width="60" height="80" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" transform="rotate(90,41,30)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-3"><g><rect x="51" y="10" width="60" height="160" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" transform="rotate(90,81,90)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-4"><g><rect x="71" y="90" width="60" height="120" fill="#fff2cc" style="fill: rgb(255, 242, 204); stroke: rgb(214, 182, 86);" stroke="#d6b656" transform="rotate(90,101,150)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-5"><g><rect x="-9" y="130" width="60" height="40" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" transform="rotate(90,21,150)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-6"><g><rect x="51" y="70" width="60" height="160" fill="none" stroke="#ed333b" style="stroke: rgb(237, 51, 59);" stroke-width="3" transform="rotate(90,81,150)" pointer-events="all"></rect></g></g></g></g></g></svg>

Looking at values, we can notice that a value in the main type could either be a value of the inner type or it could be completely outside of it. This implies that, given a `Prism A B`, we can define a function `preview :: A -> Maybe B` which, given a value `a :: A` returns a `Just b` if `a` was inside the sub-rectangle identified by `B` and `Nothing` otherwise.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="364px" height="183px" viewBox="-0.5 -0.5 364 183" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.2&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;z8OuRh85pukrXDgO7kjt&quot;&gt;
    &lt;mxGraphModel dx=&quot;434&quot; dy=&quot;277&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-1&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;210&quot; y=&quot;230&quot; width=&quot;60&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-2&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;130&quot; y=&quot;230&quot; width=&quot;60&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;170&quot; y=&quot;250&quot; width=&quot;60&quot; height=&quot;160&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;190&quot; y=&quot;330&quot; width=&quot;60&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;110&quot; y=&quot;370&quot; width=&quot;60&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeWidth=3;strokeColor=#ed333b;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;170&quot; y=&quot;310&quot; width=&quot;60&quot; height=&quot;160&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;wvFn83hAZ2eeNdT3Gh0k-1&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0.5;exitY=1;exitDx=0;exitDy=0;entryX=0.5;entryY=0;entryDx=0;entryDy=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot; source=&quot;bTHEDbTsUVzmCIvmqzU7-6&quot; target=&quot;bTHEDbTsUVzmCIvmqzU7-6&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;210&quot; y=&quot;390&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;260&quot; y=&quot;340&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;wvFn83hAZ2eeNdT3Gh0k-2&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;rotation=90;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;390&quot; y=&quot;330&quot; width=&quot;60&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;wvFn83hAZ2eeNdT3Gh0k-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;rotation=90;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;310&quot; y=&quot;370&quot; width=&quot;60&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;wvFn83hAZ2eeNdT3Gh0k-5&quot; value=&quot;&quot; style=&quot;endArrow=none;html=1;rounded=0;exitX=0.5;exitY=1;exitDx=0;exitDy=0;entryX=0.5;entryY=0;entryDx=0;entryDy=0;&quot; edge=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry width=&quot;50&quot; height=&quot;50&quot; relative=&quot;1&quot; as=&quot;geometry&quot;&gt;
            &lt;mxPoint x=&quot;320&quot; y=&quot;390&quot; as=&quot;sourcePoint&quot; /&gt;
            &lt;mxPoint x=&quot;480&quot; y=&quot;390&quot; as=&quot;targetPoint&quot; /&gt;
          &lt;/mxGeometry&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-1"><g><rect x="91" y="-10" width="60" height="80" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" transform="rotate(90,121,30)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-2"><g><rect x="11" y="-10" width="60" height="80" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" transform="rotate(90,41,30)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-3"><g><rect x="51" y="10" width="60" height="160" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" transform="rotate(90,81,90)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-4"><g><rect x="71" y="90" width="60" height="120" fill="#fff2cc" style="fill: rgb(255, 242, 204); stroke: rgb(214, 182, 86);" stroke="#d6b656" transform="rotate(90,101,150)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-5"><g><rect x="-9" y="130" width="60" height="40" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" transform="rotate(90,21,150)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-6"><g><rect x="51" y="70" width="60" height="160" fill="none" stroke="#ed333b" style="stroke: rgb(237, 51, 59);" stroke-width="3" transform="rotate(90,81,150)" pointer-events="all"></rect></g></g><g data-cell-id="wvFn83hAZ2eeNdT3Gh0k-1"><g><path d="M 1 150 L 161 150" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g><g data-cell-id="wvFn83hAZ2eeNdT3Gh0k-2"><g><rect x="271" y="90" width="60" height="120" fill="#fff2cc" style="fill: rgb(255, 242, 204); stroke: rgb(214, 182, 86);" stroke="#d6b656" transform="rotate(90,301,150)" pointer-events="all"></rect></g></g><g data-cell-id="wvFn83hAZ2eeNdT3Gh0k-3"><g><rect x="191" y="130" width="60" height="40" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" transform="rotate(90,221,150)" pointer-events="all"></rect></g></g><g data-cell-id="wvFn83hAZ2eeNdT3Gh0k-5"><g><path d="M 201 150 L 361 150" fill="none" stroke="#000000" style="stroke: rgb(0, 0, 0);" stroke-miterlimit="10" pointer-events="stroke"></path></g></g></g></g></g></svg>

On the other hard, since a `Prism` constitutes a horizontal slice of the main rectangle, if we have a value of the sub-rectangle, we can always interpret it as a value of the main rectangle. In other words, this means that for a `Prism A B` we can always define a function `review :: B -> A` constructing a value of type `A` from a value of type `B`.

<br>

Again, having a look at the graphical representation we can convince ourselves that the following properties hold for `Prism`s:

- If we preview through a `Prism` what we just built using the same `Prism`, we will get a value back: `preview (review b) == Just b`.
- If when we preview we get a `Just`, then reviewing the result through the same `Prism` will get us to the initial value: `preview s == Just a => review a == s`

<br>

For a `Prism A B` it is also possible to define a function `set :: B -> A -> A` as `set = flip $ const review`. This means that, being able to construct an `A` from a `B`, we are able to substitute a `B` inside an `A` just by discarding the initial `A` and building a new one from `B`. Graphically, we can interpret this as using the `B` value in the inner rectangle to build an `A` value, forgetting about the initial `A` value.

<br>

At this point we can also define another function `over :: (B -> B) -> A -> A` which allows us to update the `B` part inside an `A`. We can define it as `over f a = maybe a review (f <$> preview a)`. In words, we use `preview` to get a `Maybe B` and we map `f` over it to get another `Maybe B`; if we have a value `Just b`, then we can use it to construct an `A` using `review`; on the other hand, if we ended up with a `Nothing`, we just keep the initial `A`. Graphically, we can interpret this as follows: if the `A` value is inside the `B` sub-rectangle, we apply `f` and then we use the result to build a new `A` value; if the value is not in `B`, we just leave it alone.

<br>

Looking at the graphical interpretation, it's easy to convince ourselves that the composition of two `Prism`s is still a `Prism`, given that a horizontal slice of a horizontal slice is still a horizontal slice of the main rectangle. In other terms, also `Prism`s form a subcategory of the category of `Optic`s.

## Affine traversals

Now that we discussed `Lens`es and `Prism`s, one natural question which might arise is what happens when we try to compose a `Lens` and a `Prism`.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="163px" height="182px" viewBox="-0.5 -0.5 163 182" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.1.2&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;z8OuRh85pukrXDgO7kjt&quot;&gt;
    &lt;mxGraphModel dx=&quot;1426&quot; dy=&quot;827&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-1&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;210&quot; y=&quot;230&quot; width=&quot;60&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-2&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;130&quot; y=&quot;230&quot; width=&quot;60&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;170&quot; y=&quot;250&quot; width=&quot;60&quot; height=&quot;160&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;190&quot; y=&quot;330&quot; width=&quot;60&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;110&quot; y=&quot;370&quot; width=&quot;60&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeWidth=3;strokeColor=#ed333b;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;170&quot; y=&quot;310&quot; width=&quot;60&quot; height=&quot;160&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;zOTTJYkB1OkXbjc25o-X-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeColor=#0000FF;strokeWidth=2;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;160&quot; y=&quot;360&quot; width=&quot;120&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-1"><g><rect x="91" y="-10" width="60" height="80" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" transform="rotate(90,121,30)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-2"><g><rect x="11" y="-10" width="60" height="80" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" transform="rotate(90,41,30)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-3"><g><rect x="51" y="10" width="60" height="160" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" transform="rotate(90,81,90)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-4"><g><rect x="71" y="90" width="60" height="120" fill="#fff2cc" style="fill: rgb(255, 242, 204); stroke: rgb(214, 182, 86);" stroke="#d6b656" transform="rotate(90,101,150)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-5"><g><rect x="-9" y="130" width="60" height="40" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" transform="rotate(90,21,150)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-6"><g><rect x="51" y="70" width="60" height="160" fill="none" stroke="#ed333b" style="stroke: rgb(237, 51, 59);" stroke-width="3" transform="rotate(90,81,150)" pointer-events="all"></rect></g></g><g data-cell-id="zOTTJYkB1OkXbjc25o-X-5"><g><rect x="41" y="120" width="120" height="60" fill="none" stroke="#0000ff" style="stroke: rgb(0, 0, 255);" stroke-width="2" pointer-events="all"></rect></g></g></g></g></g></svg>

In the picture above we see a `Lens` (the blue rectangle) composed with a `Prism` (the red rectangle). What we get out of the composition is the lower right rectangle, which is neither a `Lens`, nor a `Prism`, with respect to the main rectangle. It's just a single inner rectangle.

<br>

On the other hand, if you think about it, every inner rectangle of the main rectangle could be obtained by composing `Lens`es and `Prism`s.

<br>

An `Optic` identifying an inner rectangle is called an `AffineTraversal`.

<br>

Combining the intuitions we had for `Lens`es and `Prism`s, it's actually possible to define functions `set :: B -> A -> A` and `over :: (B -> B) -> A -> A` also for `AffineTraversal`s.

<br>

Moreover, the graphical representation suggests us that also `AffineTraversal`s for a subcategory of `Optic`s, since a sub-rectangle of a sub-rectangle is actually a sub-rectangle of the initial one.

## Why stop at one?

All the `Optic`s that we discussed so far focus on a single sub-rectangle. But, if we want, we can consider also `Optic`s which focus on multiple sub-rectangles at the same time.

<br>

We will denote by `Traversal A B` the `Optic`s which focus on multiple sub-rectangles of type `B` inside a main rectangle of type `A`.

<svg xmlns="http://www.w3.org/2000/svg" style="background: #ffffff; background-color: #ffffff; color-scheme: light dark;" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="163px" height="182px" viewBox="-0.5 -0.5 163 182" content="&lt;mxfile host=&quot;app.diagrams.net&quot; agent=&quot;Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0&quot; version=&quot;28.2.5&quot; scale=&quot;1&quot; border=&quot;0&quot;&gt;
  &lt;diagram name=&quot;Page-1&quot; id=&quot;z8OuRh85pukrXDgO7kjt&quot;&gt;
    &lt;mxGraphModel dx=&quot;595&quot; dy=&quot;379&quot; grid=&quot;1&quot; gridSize=&quot;10&quot; guides=&quot;1&quot; tooltips=&quot;1&quot; connect=&quot;1&quot; arrows=&quot;1&quot; fold=&quot;1&quot; page=&quot;1&quot; pageScale=&quot;1&quot; pageWidth=&quot;850&quot; pageHeight=&quot;1100&quot; math=&quot;0&quot; shadow=&quot;0&quot;&gt;
      &lt;root&gt;
        &lt;mxCell id=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;1&quot; parent=&quot;0&quot; /&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-1&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;225&quot; y=&quot;215&quot; width=&quot;30&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-2&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;130&quot; y=&quot;230&quot; width=&quot;60&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;130&quot; y=&quot;290&quot; width=&quot;60&quot; height=&quot;80&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;190&quot; y=&quot;330&quot; width=&quot;60&quot; height=&quot;120&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;bTHEDbTsUVzmCIvmqzU7-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;rotation=90;&quot; parent=&quot;1&quot; vertex=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;110&quot; y=&quot;370&quot; width=&quot;60&quot; height=&quot;40&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;ASNb_kra0QyNIT7Ww5Bv-1&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;200&quot; y=&quot;270&quot; width=&quot;40&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;ASNb_kra0QyNIT7Ww5Bv-2&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#e1d5e7;strokeColor=#9673a6;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;240&quot; y=&quot;270&quot; width=&quot;40&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;ASNb_kra0QyNIT7Ww5Bv-3&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;200&quot; y=&quot;300&quot; width=&quot;80&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;ASNb_kra0QyNIT7Ww5Bv-4&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;200&quot; y=&quot;330&quot; width=&quot;40&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;ASNb_kra0QyNIT7Ww5Bv-5&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;240&quot; y=&quot;330&quot; width=&quot;40&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;ASNb_kra0QyNIT7Ww5Bv-6&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeColor=#FF3333;strokeWidth=3;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;200&quot; y=&quot;270&quot; width=&quot;40&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;ASNb_kra0QyNIT7Ww5Bv-7&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeColor=#FF3333;strokeWidth=3;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;240&quot; y=&quot;330&quot; width=&quot;40&quot; height=&quot;30&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
        &lt;mxCell id=&quot;ASNb_kra0QyNIT7Ww5Bv-8&quot; value=&quot;&quot; style=&quot;rounded=0;whiteSpace=wrap;html=1;fillColor=none;strokeColor=#FF3333;strokeWidth=3;&quot; vertex=&quot;1&quot; parent=&quot;1&quot;&gt;
          &lt;mxGeometry x=&quot;120&quot; y=&quot;360&quot; width=&quot;40&quot; height=&quot;60&quot; as=&quot;geometry&quot; /&gt;
        &lt;/mxCell&gt;
      &lt;/root&gt;
    &lt;/mxGraphModel&gt;
  &lt;/diagram&gt;
&lt;/mxfile&gt;
"><defs></defs><rect fill="#ffffff" style="fill: #ffffff;" width="100%" height="100%" x="0" y="0"></rect><g><g data-cell-id="0"><g data-cell-id="1"><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-1"><g><rect x="106" y="-25" width="30" height="80" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" transform="rotate(90,121,15)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-2"><g><rect x="11" y="-10" width="60" height="80" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" transform="rotate(90,41,30)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-3"><g><rect x="11" y="50" width="60" height="80" fill="#ffe6cc" style="fill: rgb(255, 230, 204); stroke: rgb(215, 155, 0);" stroke="#d79b00" transform="rotate(90,41,90)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-4"><g><rect x="71" y="90" width="60" height="120" fill="#fff2cc" style="fill: rgb(255, 242, 204); stroke: rgb(214, 182, 86);" stroke="#d6b656" transform="rotate(90,101,150)" pointer-events="all"></rect></g></g><g data-cell-id="bTHEDbTsUVzmCIvmqzU7-5"><g><rect x="-9" y="130" width="60" height="40" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" transform="rotate(90,21,150)" pointer-events="all"></rect></g></g><g data-cell-id="ASNb_kra0QyNIT7Ww5Bv-1"><g><rect x="81" y="30" width="40" height="30" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="ASNb_kra0QyNIT7Ww5Bv-2"><g><rect x="121" y="30" width="40" height="30" fill="#e1d5e7" style="fill: rgb(225, 213, 231); stroke: rgb(150, 115, 166);" stroke="#9673a6" pointer-events="all"></rect></g></g><g data-cell-id="ASNb_kra0QyNIT7Ww5Bv-3"><g><rect x="81" y="60" width="80" height="30" fill="#dae8fc" style="fill: rgb(218, 232, 252); stroke: rgb(108, 142, 191);" stroke="#6c8ebf" pointer-events="all"></rect></g></g><g data-cell-id="ASNb_kra0QyNIT7Ww5Bv-4"><g><rect x="81" y="90" width="40" height="30" fill="#d5e8d4" style="fill: rgb(213, 232, 212); stroke: rgb(130, 179, 102);" stroke="#82b366" pointer-events="all"></rect></g></g><g data-cell-id="ASNb_kra0QyNIT7Ww5Bv-5"><g><rect x="121" y="90" width="40" height="30" fill="#f8cecc" style="fill: rgb(248, 206, 204); stroke: rgb(184, 84, 80);" stroke="#b85450" pointer-events="all"></rect></g></g><g data-cell-id="ASNb_kra0QyNIT7Ww5Bv-6"><g><rect x="81" y="30" width="40" height="30" fill="none" stroke="#ff3333" style="stroke: rgb(255, 51, 51);" stroke-width="3" pointer-events="all"></rect></g></g><g data-cell-id="ASNb_kra0QyNIT7Ww5Bv-7"><g><rect x="121" y="90" width="40" height="30" fill="none" stroke="#ff3333" style="stroke: rgb(255, 51, 51);" stroke-width="3" pointer-events="all"></rect></g></g><g data-cell-id="ASNb_kra0QyNIT7Ww5Bv-8"><g><rect x="1" y="120" width="40" height="60" fill="none" stroke="#ff3333" style="stroke: rgb(255, 51, 51);" stroke-width="3" pointer-events="all"></rect></g></g></g></g></g></svg>

For `Traversal`s we can still define `set :: B -> A -> A` which replaces all the selected sub-rectangles of type `B`, inside the main rectangle of type `A`, with the same vale `b` of type `B`, to produce a new `A`.

<br>

Similarly, we can define `over :: (B -> B) -> A -> A` which applies a function to all the selected sub-rectangles of type `B`, inside the main rectangle of type `A`, to produce a new `A`.

<br>

Another relevant function which makes sense to consider for `Traversal`s is `toListOf :: A -> [B]`, which extracts all the values of the selected sub-rectangles of type `B` from the main rectangle of type `A`.

<br>

As usual, we can notice that `Traversal A B` form a subcategory of `Optic A B`, since a selection of sub-rectangles inside a selection 0f sub-rectangles is still a selection of sub-rectangles of the main rectangle.

## Conclusion

The graphical representation we just introduced in this post provides us with a tool to navigate various kinds of `Optic`s and their operations. I hope it can provide a concrete way to understand the basic ideas behind `Lens`es, `Prism`s and other `Optic`s and make it easier to use them.

<br>

Such a representation could also help to explore and shed some light on the mysterious world of `Optic`s. One could try to search for other sub-categories in a graphical fashion and then ask what do they correspond to in other `Optic` representation. For example, what is the sub-category of `Optics` made by multiple horizontal slices? Or the one made by multiple vertical slices?

<br>

I need also to mention that such a representation is not able, as far as I can see, to fully represent the whole universe of `Optic`s. For example, it's hard to distinguish a `Traversal` from a `Fold`, or describe what `Grate`s are.

<br>

All in all, I'm confident that describing and explaining optics in this graphical fashion could help people understand their beauty and usefulness! Thanks for reading up to here!

---

[^1]: If you’re interested, I previously discussed various ways of implementing optics [here](https://www.tweag.io/blog/2022-05-05-existential-optics/).