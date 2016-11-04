---
layout: post
title:  "On valid models and data structures"
date:   2016-11-04 13:09:13 +0200
categories: post
tags: elm data-structure components hanoi
comments: true
pageUrl: '"http://marcosh.github.io/post/2016/11/04/on-valid-models-and-data-structures.html"'
pageIdentifier: '"elm-data-structures"'
---

Some days ago I had the luck to speak at [WebCamp](https://2016.webcampzg.org) in Zagreb. It was a really nice conference and I would suggest every web developer to consider going there next year.

I spoke about [Elm](http://elm-lang.org/), which, at the moment, is definitely my favourite programming language. For my talk I prepared a little sample application that you can find [here](http://marcosh.github.io/presentations/2016/10/28/elm-or-how-I-learned-to-love-frontend-development.html#/23) in my slides (the source code is [here](https://github.com/marcosh/elm-hanoi)). It is a simple implementation of the game of the [Towers of Hanoi](https://en.wikipedia.org/wiki/Tower_of_Hanoi).

When I started writing this application, I had two objectives in mind:

- model the state of the application in such a way that only valid dispositions of the disk were representable;
- provide a module that could be reused with different views and interacted with in different ways. For example, in the current version you select the starting and the ending peg of a move from two select boxes; there could be another version of the game, which could use the same module for the business logic, where the game is actually drawn in 3D graphics and the user interaction happens via drag and drop.

## A valid model

When trying to model the game of the Towers of Hanoi, probably the first approach that comes to mind is the following

    type alias Disk = Int

    type alias Model = ( List Disk, List Disk, List Disk )

This would mean that we represent a disk by an integer (1 for the smallest disk, 2 for the second smallest disk, and so on...) and then every peg is just the list of disks it contains.

This model is quite simple and easy to grasp, but it has an hidden complexity. What if, at at certain point of the lifecycle of our application, we end up with a model like the following?

    model = ([1, 5, 3], [1, 2], [])

This would mean that the first peg contains the first, the fifth and the third disk, the second peg contains the first and the second disk and there are no disks in the third peg. If we think a little about it, we see immediately that we are actually violating three rules of the game:

- the fourth disk is not present;
- the fifth disk is above the third disk;
- the first disk is present twice!

We could try to ensure, checking carefully how we hadle moves in our code, that we never end up at such a state, or we could validate our model everytime we change it.

The other option that we have, is just simply change our model! For example, we could shift our focus from the disks to the pegs

    type Position
        = FirstPeg
        | SecondPeg
        | ThirdPeg

    type alias Model = List Position

In this way we have the three pegs and then we just say for every disk which is its position; for example, the first element of the list is the position of the smallest disk, the second element of the list is the position of the second smallest disk, and so on. Since a list can not have gaps, is always ordered and can hold one value for every position, we already solved all the problems we had with the previous model!

Hence, with such a model, we are safe that in every moment of the life of our application, the model will be in a valid state. As a side note, this will make property based testing much easier.

## Interchangable view

When I started writing the application, I immediately reached for [the Elm architecture](https://guide.elm-lang.org/architecture/) and began with the standard `model`, `view` and `update` functions. Since I wanted my module to be reusable with different views I tought of providing a default view, which could be overwritten if needed.

Going on a bit with this approach I realized that messages were strictly related to the view part and a different way of interacting with the model could have meant the presence of a different set of messages. The consequence of this is that, if somebody wanted to reuse my component with another view, probably had also to modify the update part.

Then, thinking about how to solve these issues to provide an easily reusable package, I realized that my module needed not to be a component with a `model`, a `view` and an `update`. What it actually make sense is that my module exposes just a data structure, the one that I described in the previous section, and some functions:

- a function `move : Model -> Position -> Position -> ( Model, Maybe OutMsg )` that would take as input the current state of the game and the starting and ending pegs of the desired move of the user, and would return the new state of the model after the move is performed and, if needed, a message to the user. This function is meant to be called by the update function of the external component to actually perform a move.

- a function `disposition : Model -> ( List Int, List Int, List Int )` that would just make explicit which disks are present in each peg. This function is meant to be used by the view part of the external component to render on the screen the state of the data structure.

When I realized this it was eventually clear to me what it meant that in Elm the unit of reuse is given by data structures and functions acting on them, and not by components.

## Conclusion

The main takeaway that I had writing this little application was not to reach immediately for the Elm architecture. It will definitely be present in the most external component of the application, but there is not need to use it all the way down. On the other hand, focus on the domain, try to model it in the best possible way with a data structure, such that every valid state of the model actually respects all the invariants and all the rules which we want to hold true. Then, and only then, start to think how to interact with the data structure and which of its aspects expose for the rendering part.
