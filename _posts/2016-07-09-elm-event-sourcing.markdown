---
layout: post
title:  "Elm and Event Sourcing"
date:   2016-07-09 16:09:34 +0200
categories: post
tags: elm eventsourcing ddd functional
comments: true
pageUrl: '"http://marcosh.github.io/post/2016/07/08/elm-event-sourcing.html"'
pageIdentifier: '"elm-event-sourcing"'
---

## Elm

In the last time I've been attracted to study mainly two things, Elm and Domain Driven Design. They are not necessarily two strictly related subjects, but, in my own opinion, they have quite a lot in common. In this article I'll try to make one of these aspects explicit.

Elm is a lovely functional programming language that runs in the browser. One of its aim is to grant the developer a nice working experience, providing him with a friendly compiler and a nice, clean and simple architecture.

The Elm Architecture works more or less as follows. It consists of three main components:

- `model`
- `view`
- `update`

The `model` is generally a data structure that keeps track of the state of the application. For example, if you have a counter, your model could be

    model : Int
    model = 0

where we are just saying that the data that our application needs to keep track of is an integer which has initial value equal to 0.

The `view` is a function that takes care of rendering the model to the screen. The signature of this function is

    view : Model -> Html Msg

meaning that `view` receives a model as input (in our example just an integer) an returns some `Html` which will be rendered to the screen.

The `update` function is the component of the architecture that allows us to evolve our `model`. Its signature is

    update : Msg -> Model -> Model

This means that the `update` function receives in input a `Msg` emitted by our `Html` (think of it as a notification of some action performed by the user, i.e. the click of a button or the typing of some characters in an input field) and the current version of the `model`. It then returns a new version of the `model`, updated to reflect the changes requested by the `Msg`.

The architecture then works as follows. Everything starts with the initial version of the `model`, which is then rendered to the screen using the `view`. The `Html` returned by the `view` produces, responding to user inputs, some `Msg`es that are given to the `update` function, which creates a new version of the `model`. At this point the cycle starts again and everything continues in this unidirectional data flow.

Quite enough about Elm for now.

## Domain driven design

I'm not going to try to define what Domain Driven Design is. I will just say that it is an approach to creating quality software that has its focus in understanding the domain for which the software is created and try to reflect its dynamics into the code itself. It does so providing some patterns, both stategical and tactical and promoting a strong focus on the linguistic of the domain.

One of the concepts that appear often in Domain Driven Design is the one of Events. From Event Storming to Event Sourcing, understanding the events that happen in a domain and their relations is a way to understand, and to make then explicit in the code, how the domain works and evolves over time.

An event is usually a value object, and it could often be implemented just as a container of data which describes something that happened in the lifetime of the application.

Event sourcing consists in keeping track of the evolution of an application by storing all the events that happened to it and, whenever necessary, reconstructing its state by going through all the events in the history.

For example, if we have a counter, with two buttons that allow to increment and decrement by one the value of the counter, we could store one event for every click of each button. Then, when we need to know the actual value of the counter, we could, starting from the initial value of the counter, just add 1 for every event that represents the click of the increment button, and subtract 1 for every event that reprepresents the click of the decrement button.

The main benefit of Event Sourcing is that it allows us to keep track of all the history of our application, without losing data as our application evolves over time. This is really helpful when we are working with server side applications that handle a lot of data, but, with the increase usage of Client Side technologies, it could prove itself helpful also in frontend applications.

Let's now see how we could easily use an Event Sourcing approach with Elm

## Elm and Event Sourcing

First thing to say here, is that actually, beyond the surface, Elm is already using an Event Sourcing approach to keep track of what happens to the application. This is the reason that allows the existance of a Time Travel Debugger, which grants us the possibility of rewinding back time and replaing the events that already happened, maybe changing some parameters and seeing how these changes will impact the evolution of the application.

What I'd like to approach now is how to make Event Sourcing explicit in our Elm applications. In fact, even if beyond the curtains Elm is using an Event Sourcing approach, the `model` we usually work with keeps track just of the current state of the application, not of how the application actually got there.

There could be cases where we'd like to keep track of every step that led to obtain the current state. Think for example to an application where we'd like to provide the user with a `Back` button, which gives him the possibility to revert his actions. Or an application with a really complex domain, where the state itself would need to keep track of a lot of variables regarding the history of the application, so that it becomes easier to just store directly all the happened events. Another use case could be implementing directly in Elm some kind of analytics that needs to keep track of all the events that happened in the application.

Let's see now how we could actually easily implement Event Sourcing with Elm using the Elm architecture.


## Implementing Event Sourcing in ELM

First thing we need to do is define the possible events that could happen in our application. For example, in the example of the counter with the increment and decremente buttons, the events could be

    type Event
        = Increment
        | Decrement

If we want to keep track of all the events happening to our application, we better define a `History` type

    type alias History = List Event

Once we did that, the model of the application is a `History`, which is just a list of `Event`s

The `update` function now becomes really simple. If our model is a list of events, when a new `Event` happens, we just add it to the list

    update : Event -> History -> History
    update event history =
        event :: history

If the `update` function is now trivial, the `view` function becomes a little more complicated, because the model is now much more rich.

    view : History -> Html Event

If we want to retrieve the current state  of our application, we need to go through all the events that happened already and apply them one by one to the initial state.

If we want to reuse the functions that we wrote for our standard Elm Architecture, we could factor our new `view` function like

    view = viewState << buildState

where `<<` is just function composition,

    viewState : State -> Html Msg

corresponds to the standard `view` function in the usual Elm Architecture, and

    buildState : History -> State
    buildState history = List.foldl applyEvent initialState history

applies recursively every event, starting from `initialState`, using at every step the function

    applyEvent : Msg -> State -> State

that does exactly what the `update` function in the standard Elm Architecture does.

## Conclusion

Event sourcing is somehow already present in the Elm Architecture, but not at a userland level. Anyway, as we saw, it is quite straightforward to bring it at the user level.
This allows us not to lose any data in the interaction between our application and the user, and this could turn out to be really useful for application with a pretty complicated domain or even for analytics purposes.