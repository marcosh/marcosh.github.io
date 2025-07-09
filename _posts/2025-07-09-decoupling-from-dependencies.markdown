---
layout: post
title:  "Decoupling from dependencies"
author: Marco Perone
date: 2025-07-09 08:06:42 +0200
categories: post
tags: haskell monad abstraction concretion decoupling dependency
comments: true
pageUrl: '"http://marcosh.github.io/post/2025/07/09/decoupling-from-dependencies.html"'
pageIdentifier: '"Decoupling from dependencies"'
description: "Decoupling from dependencies"
image: "/img/from-abstraction-to-realism-070.jpg"
---

When we are writing our beautiful code we often need to deal with dependencies, might they be tools, libraries, or simply other code we wrote some time ago.

<br>

Choosing to use a dependency usually comes with its own set of tradeoffs and constraints. For example using a relational or a non-relational database might guide us to design our persistence-level data types in different ways. Or using [time](https://hackage.haskell.org/package/time) or [thyme](https://hackage.haskell.org/package/thyme) provide similar but different APIs on the same concepts.

<br>

Most of the time, we accept the tradeoffs and the constraints coming with the choice of a specific dependency, and include those into the mental model we have of our own application.

<br>

Some other times instead we want to protect our code from certain constraints coming from the choice of a specific dependency, or simply we don't want to commit to the usage of a specific dependency and include its tradeoffs into our own model of our application.

<br>

To do this we need a way to protect our application from being corrupted by the choices made by the concrete dependency and hide it from our beautiful and clean code.

<br>

But how can we use a dependency without letting our code know about it?

## A concrete dependency

To work with a concrete example, let's suppose that our dependency is a software component which exposes a clearly defined API.
For the sake of the example, let's just say that our dependency is a single function

```haskell
module Foo where

data Foo = Foo {..}

bar :: Foo -> String
bar = ...
```

The easiest way to use this code is simply by importing the `Foo` module and call the `bar` function in our own code

```haskell
module Domain where

ourOwnCode :: ...
ourOwnCode = do
  ...
  let foo = Foo {..}
      fooResult = bar foo
  ...
```

As we mentioned in the previous section, there might be situations where we don't want to call `bar` directly. This could be for multiple reasons:

- `bar` is too low level, and we want to keep things more abstract.
- We don't like the name `bar` and we want to use a name more in line with the language used in our code.
- We want to commit to use `bar` only later on in the flow of our application and for the moment we want to avoid using a concrete implementation.

## Dependency inversion

At this point we need to find a way to allow our code to use the functionality provided by `bar` without using `bar` directly.

<br>

While this may seem impossible, there's actually a way around it. What we can do is use the [Dependency inversion principle](https://en.wikipedia.org/wiki/Dependency_inversion_principle) (the letter D of the [SOLID](https://en.wikipedia.org/wiki/SOLID) acronym) which states:

> High-level modules should not import anything from low-level modules. Both should depend on abstractions (e.g., interfaces).
> Abstractions should not depend on details. Details (concrete implementations) should depend on abstractions.

The quoted low-level module is our `Foo` dependency. Looks like the trick is to define an abstraction around the `Foo` module.

## Our own `bar`

To abstract ourselves away from the concretions defined in the `Foo` module, we create our own abstract interface around it. For example, we could define[^1]

```haskell
-- in Domain

data OurOwnFoo = OurOwnFoo {..}

data OurOwnBar = OurOwnBar {ourOwnBar :: OurOwnFoo -> String}
```

In this way the rest of our code can use this newly defined API and not depend directly on `bar`. In fact, even though this is really similar to the `Foo` module, it is concretely not related at all with it.

<br>

This might look like a silly step with such a simple and artificial example, but it makes a lot more sense when you have more complex API you depend on.

<br>

For example, a [Repository](https://martinfowler.com/eaaCatalog/repository.html) uses a collection-like interface to provide access to domain objects while hiding the persistence details. As a concrete example we could have a relational database with a `user` table and several functions querying that table, like

```
selectAllUsers :: DB [User]
selectUser :: UserId -> DB (Maybe User)
insertNewUser :: User -> DB UserId
updateUser :: UserId -> User -> DB ()
deleteUser :: UserId -> DB ()
```

We might want to hide these functions away from our domain code behind a nice looking interface, where we can use a more precise language, and we can abstract from the concrete `DB` context. A potential interface could be

```
data UserRepository m = UserRepository
  { retrieveAllUsers :: m [User]
  , retrieveUserById :: UserId -> m (Maybe User)
  , addUser :: User -> DB UserId
  , editUserName :: UserId -> UserName -> m ()
  , editUserBirthday :: UserId -> Birthday -> m ()
  , removeUser :: UserId -> m ()
  }
```

Notice how this allows us to do several things. We can:

- Use a language which conforms better with our domain concepts and is not linked to technical details.
- Redesign our API by changing the number and the scope of the functions.
- Abstract the context `m` in which the operations are actually happening, delaying the choice to a later moment and allowing the possibility to reuse the same abstraction with multiple concrete implementations. This could be useful if we want to write unit tests for effectful code using a mocked version of the dependency.

## Implementing the abstraction

Now that we defined an abstraction, we can follow the dependency inversion principle and adapt things so that "details (concrete implementations) depend on abstractions". This means that we should create a thin layer between our own code and the one of our dependency to adapt the dependency API to the one defined by our own API.

<br>

In our example, this means creating a concrete value of type `OurOwnBar` using the `bar` function provided by our dependency. To do so it is enough to do the following:

```haskell
module Implementation where

encodeFoo :: OurOwnFoo -> Foo
encodeFoo = ...

ourOwnBarImplementation :: OurOwnBar
ourOwnBarImplementation = OurOwnBar $ bar . encodeFoo
```

We use the world `encode` because we are converting from a higher-level representation (`OurOwnFoo`) to a lower-level representation (`Foo`). This is usually a pure operation and could possibly require further arguments. If we had to convert a return type, we would need a `decode` operation instead. Decoding is often an operation which may fail and therefore requires some error handling mechanism.

## Connecting it all

Now we have our abstraction with some code depending on it and a concrete implementation implementing that abstraction using our chosen dependency. But still we are not connecting the two, meaning that we are not telling our domain code which implementation to use to implement our abstraction.

<br>

Notice that we can not connect the two inside our domain layer, because that has no knowledge at all about the concrete implementation (that's exactly the reason why we're doing all of this). The solution is to have a third layer used precisely to connect the two lower layers. Concretely we would have

```haskell
module Application where

import Domain
import Implementation

applicationBar :: OurOwnFoo -> String
applicationBar = ourOwnBar ourOwnBarImplementation
```

As you can see, `Application` imports both `Domain` and `Implementation` and calls the `ourOwnBar` function from our `OurOwnBar` abstraction on the `ourOwnBarImplementation` concrete implementation.

<br>

In the example with the repository, this would also be the perfect place the select a concrete context `m` where to execute operations.

<br>

To sum up, we created a layering like the following, where an arrow means that the layer where the arrow starts can be used by the layer the arrow points to:

<br>

[![](https://mermaid.ink/img/pako:eNptjctqwzAQRX_FzNoxsmLZqhaBNtl031XRRliyLdALRaIP43-vYiilaXf3zpkzs8LopQIGk_Fv4yJiqp5euKuqxxCMHkXS3t3qxVuh9_Rsg1FWuXTHqsPh9L-1k7_a78m9DjXMUUtgKWZVg1WxnCoV1pvLIS3F5cBKlGoS2SQO3G1FC8K9em-_zejzvACbhLmWloMUSV20mKP4WVFOqnj22SVgZL8AbIV3YEfSNwPFGPekx4V8FD40mCDUtl3XoYeWbjV87u9QQ4cCEKJth46Ykn77ArpncH8?type=png)](https://mermaid.live/edit#pako:eNptjctqwzAQRX_FzNoxsmLZqhaBNtl031XRRliyLdALRaIP43-vYiilaXf3zpkzs8LopQIGk_Fv4yJiqp5euKuqxxCMHkXS3t3qxVuh9_Rsg1FWuXTHqsPh9L-1k7_a78m9DjXMUUtgKWZVg1WxnCoV1pvLIS3F5cBKlGoS2SQO3G1FC8K9em-_zejzvACbhLmWloMUSV20mKP4WVFOqnj22SVgZL8AbIV3YEfSNwPFGPekx4V8FD40mCDUtl3XoYeWbjV87u9QQ4cCEKJth46Ykn77ArpncH8)

<br>

- `Domain` contains the important code we really care about, the code which is adding our own logic around the functionality of our dependency. As a layer, it does not depend on anything and can not use code from any other layer.
- `Implementation` contains the concrete implementation and the necessary to adapt it to the abstraction defined in the `Domain`. As a layer it depends only on the domain, since it needs to use its abstractions.
- `Application` connects concretely the `Implementation` layer to the `Domain` layer and depends on both.

## Conclusion

Decoupling allows us to develop components in isolation, focusing on their specificities. This allows us to write code which is easier to read and to manage because it has less moving components. On the other hand, simpler components often mean that the complexity moves to connecting the components themselves.

<br>

What we described in this post is a standard pattern to decouple our domain code from its dependencies, which too often lead us to cripple our code with decisions we didn't make. Using this pattern allows us to keep our domain code cleaner and easier to maintain. The pattern has a price in terms of number of modules and layers you have to use, but it certainly worth it, especially for more complex projects.

---

[^1]: There are several says we could use to declare an interface. I plan to write my next post exactly on this topic.
