---
layout: post
title:  "Real world events are commands"
author: Marco Perone
date:   2017-03-23 09:21:42 +0200
categories: post
tags: cqrs eventsourcing domain event command
comments: true
pageUrl: '"http://marcosh.github.io/post/2017/03/23/real-world-events-are-commands.html"'
pageIdentifier: '"real-world-events-are-commands"'
description: "Real world events are actually commands"
image: "/img/messages.jpg"
---

These days (maybe it's better to say, months...) I am working on a ES/CQRS application where
command and event messages play a really important role. Usually it is easy to distinguish
if a message is a command or an event, but I would like to discuss here a case where deciding
which one was more appropriate was not so trivial.

What follows is probably not the best solution, but I'm presenting it with the hope of receiving
constructive comments and deepening my understanding of the matter. Just a plain use of Cunningham's Law:

> The best way to get the right answer on the Internet is not to ask a question, it's to post the wrong answer

## The problem

The application I am currently working on needs to track the state of some IoT devices. Every device,
with limited capabilities, will just notify the main server application of the interactions that the
users are having with it. With every different type of interaction different metadata could be sent to
the server to describe the action.

The server needs to keep track of the state of the devices to provide information about them to the users
of the application. Moreover, the server application can also receive commands aimed at modifying the
state of the devices and will need to validate them before applying them.

For example you could think of a lamp which could be turned on and off both by a physical switch and
by a mobile application communicating through the server.

Every time a real world event happens (e.g. a user clicks the switch of the lamp), a message is sent
from our device to the server.

Now, our problem is whether the server application should treat this message as an event or as a command.

## Is it an event?

As per definition, an event represents a change in the state of the system which occurred in the past. A domain
event, to be more specific, is an event which is relevant for our domain.

It looks pretty easy to conclude then that the message about the real world event that our server
application receives must be treated as an event.

Moreover, in [this thread](https://groups.google.com/forum/#!msg/dddcqrs/V3Ikh1V3V_c/hRheJkXqttgJ),
[Greg Young](https://twitter.com/gregyoung) himself suggests that when we are handling external
systems there are actually no commands.

Still, how does an event enter a CQRS architecture, where the only messages we could send to our
system are commands and queries?  We can exclude right away that our message is a query, since it is
not requiring a return value.

So, after all, maybe it is a command...

## Is it a command?

As per definition, a command is a message representing the request of performing an operation in a
system. Then the system, in particular the aggregate, will perform some logic and decide what to do
with the received request, including refusing to do anything if something is not quite right in the
message.

Coming back to our problem, if we look at our message containing the data of the real world event
from the point of view of the server application, it looks like a request to change the internal state
of our system. In other words, our system is not yet up to date with the actual state of our device
and it receives a request to keep up.

From this point of view, we should treat our incoming message as an event happening in an external
system, another bounded context, which will issue a command to our server application.

Still, commands could be rejected by applications, that's the role of the aggregates, but if the
messages we receive come from the real world, how could we say they are wrong?

## In the end, it is just a matter of trust

If we are sure that what arrives from the device is always correct, there is actually no point in
making it pass through the aggregate and have it check if the domain invariants are preserved.
If it happened in the real world, we just need to store it as it is. Maybe sometime we will receive
messages that do not really make sense, for example a message telling us that a user turned on a lamp
which was already on, but we will trust our device, because maybe the network lost our
previous message.

If this is the case, we could just bypass command handlers and aggregates, and write directly the
message information into our event store.

On the other hand, sometimes we should not completely trust our devices. Maybe they could
send us some wrong metadata; for example, their clock is broken and we receive every message with
timestamp `0`. In these cases maybe we want to make some decision at the domain level, distinguish
the type of event we will publish in the event store and notify someone that something is
not working correctly.

In the end, it is a matter of what we actually need to do with the messages we receive. If we need
to build an audit log and simply store all the messages received from the devices, there probably is
no need for aggregates, we could treat all the messages as events, store them directly into
the event store and react to them as we see fit. On the other hand, if we need to perform a check
on the integrity of the message and decide what to save in the event store accordingly, we better
treat our (external) event messages as (internal) commands which will produce (internal) events.

## Internal and external events

Coming back to our original question: is the message received from the device a command or and event?

I would say that it is both! More accurately, it is a command describing an event.

I think a lot of care needs to be taken here to distinguish events that are internal to the system
from external ones. Since we cannot ask our devices which state they are in, or even better which
are all the events that happened to them until now, the source of truth of our system cannot coincide
with the real world, but must be our server side system.

Hence, something which really happened in the real world did not actually happen in our system
until it is not persisted in the event store.

## Conclusion

The most important takeaway I took reasoning about the above problem, is to always consider things
in their appropriate context. The same message could be interpreted as an event in one context and
as a command in another.

I hope my reasoning is correct and that it will be helpful to other people with my same issues. Still,
remembering Cunningham's law, I also hope I wrote something wrong, so that someone more expert
than me could correct it and help me and others learn something new.
