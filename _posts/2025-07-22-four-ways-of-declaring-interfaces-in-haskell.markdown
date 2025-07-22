---
layout: post
title:  "Four ways of declaring interfaces in Haskell"
author: Marco Perone
date: 2025-07-22 08:06:42 +0200
categories: post
tags: haskell monad abstraction concretion decoupling interfaces record typeclass gadt free
comments: true
pageUrl: '"http://marcosh.github.io/post/2025/07/22/four-ways-of-declaring-interfaces-in-haskell.html"'
pageIdentifier: '"Four ways of declaring interfaces in Haskell"'
description: "Four ways of declaring interfaces in Haskell"
image: "/img/monitor-ports.jpg"
---

When we think about _interfaces_ in software development, it's easy that our thoughts go the `interface` keyword used in many Object-Oriented languages like Java and C#.

<br>

But interfaces are a much more general concept, and they could be defined as:

> an interface is a shared boundary across which two or more separate components of a computer system exchange information
>
> - Wikipedia

Even restricting to the realm of software architecture, an interface is an abstract specification which allows declaring a shared boundary between one (or more) consumer and one (or more) producer. This allows consumers to not know any details about the producers and, likewise, producers not to know details about the consumers. Hence, interfaces can help us decoupling software components that need to collaborate, keeping our application modular.

<br>

In particular, there's nothing specific about Object-Oriented programming when it comes to interfaces. In fact there's even purely functional languages which are using an [`interface` keyword](https://docs.idris-lang.org/en/latest/tutorial/interfaces.html).

<br>

In this post, I'd like to show and discuss four different ways for declaring interfaces in Haskell.

## The consequences of an interface

To compare multiple ways of declaring an interface, we should consider all the aspects involved with the declaration and the usage of an interface.

<br>

For the sake of the example, we will abstract the following operations through an interface in four different ways.

```haskell
retrieveAllUsers :: DB [User]

retrieveUserById :: UserId -> DB (Maybe User)

addUser :: User -> DB UserId

editUserName :: UserId -> UserName -> DB ()

editUserBirthday :: UserId -> Birthday -> DB ()

removeUser :: UserId -> DB ()
```

where `DB` has a monad instance.

<br>

Please remember that an interface is used to decouple two systems, and it should be the one defining the desired API, not indulging in technical details, while the specific implementations are the ones which should adapt to the interface definition.

<br>

The first aspect we should consider is the definition of the interface itself. How do we say that such an interface exists? And how do we declare its API?

<br>

The second aspect is how we can define implementations for a given interface. How easy is it to do so? How clear is it that what we're defining is an instance for a given interface?

<br>

Then we need to consider how we can consume an interface. How easy is it to work with an abstraction defined in such a way?

<br>

The last aspect to consider is how the wiring happens. How can we say that a specific implementation should be used?

<br>

In the next section we're going through four different ways of defining an interface in Haskell, trying to describe how each one of these aspects work.

## Interface as a typeclass

The most common way to declare an interface in Haskell is probably by using a typeclass. The declaration of the interface looks like

```haskell
class UserRepositoryClass m where
  retrieveAllUsers :: m [User]
  retrieveUserById :: UserId -> m (Maybe User)
  addUser :: User -> m UserId
  editUserName :: UserId -> UserName -> m ()
  editUserBirthday :: UserId -> Birthday -> m ()
  removeUser :: UserId -> m ()
```

An implementation of this interface is defined by providing an instance for the typeclass for a concrete type

```haskell
instance UserRepositoryClass DB where
  retrieveAllUsers = retrieveAllUsers
  retrieveUserById = retrieveUserById
  addUser = addUser
  editUserName = editUserName
  editUserBirthday = editUserBirthday
  removeUser = removeUser
```

where we are using the functions defined in the previous section.

<br>

Using the interface is a matter of writing code which has a `UserRepositoryClass` constraint.
For example, we can define a function to update a `User` like

```haskell
classUpdateUser :: (Monad m, UserRepositoryClass m) => UserId -> Maybe User -> m ()
classUpdateUser userId = \case
  -- we want to remove the user
  Nothing -> removeUser userId
  Just user -> do
    -- retrieve the user with the given id
    dbUser <- retrieveUserById userId
    case dbUser of
      -- if such a user does not exist, we create one
      Nothing -> do
        _ <- addUser user
        pure ()
      -- if the user exist in the database, update their data with the ones provided by the `user`
      Just _ -> do
        editUserName userId (name user)
        editUserBirthday userId (birthday user)
```

Connecting the interface with a concrete implementation is a matter of specifying the type which we want to use

```haskell
classUsage :: UserId -> Maybe User -> DB ()
classUsage = classUpdateUser
```

I think this style is very well known and widely used. Let's now proceed to other, probably less used, approaches.

## Interface as a record

Typeclasses are very powerful and allow hiding a lot of boilerplate by resolving constraints behind the scenes. There could be cases, though, where one desires to have more control, or simply a more explicit management of the application components.

<br>

To achieve this one could decide to renounce to the magic given by typeclasses and use a basic record of functions as an interface.

```haskell
data UserRepositoryRecord m = UserRepositoryRecord
  { retrieveAllUsers :: m [User]
  , retrieveUserById :: UserId -> m (Maybe User)
  , addUser :: User -> m UserId
  , editUserName :: UserId -> UserName -> m ()
  , editUserBirthday :: UserId -> Birthday -> m ()
  , removeUser :: UserId -> m ()
  }
```

This is really similar to the definition using a typeclass, but it uses `data` instead of `class`. It still abstracts on the context `m`, allowing the same level of decoupling one could have with the typeclass style.

<br>

Defining a concrete instance is a matter of providing a concrete value for the above defined type

```haskell
userRepositoryRecord :: UserRepositoryRecord DB
userRepositoryRecord = UserRepositoryRecord
  { retrieveAllUsers = retrieveAllUsers
  , retrieveUserById = retrieveUserById
  , addUser = addUser
  , editUserName = editUserName
  , editUserBirthday = editUserBirthday
  , removeUser = removeUser
  }
```

Using the interface declaration means accepting as an argument a value of the above defined type

```haskell
recordUpdateUser :: Monad m => UserRepositoryRecord m -> UserId -> Maybe User -> m ()
recordUpdateUser UserRepositoryRecord{..} userId = \case
  Nothing -> removeUser userId
  Just user -> do
    dbUser <- retrieveUserById userId
    case dbUser of
      Nothing -> do
        _ <- addUser user
        pure ()
      Just _ -> do
        editUserName userId (name user)
        editUserBirthday userId (birthday user)
```

And selecting a concrete implementation means passing the desired value as an argument

```haskell
recordUsage :: UserId -> Maybe User -> DB ()
recordUsage = recordUpdateUser userRepositoryRecord
```

As mentioned earlier, this style allows for a more explicit management of the concrete implementations. This could be especially useful if you need to access and interact with such values at runtime.[^1]

## Interface as a free monad

Another fairly common approach is using a [free monad](https://serokell.io/blog/introduction-to-free-monads). This means expressing our interface as a sum type instead of a product type

```haskell
data UserRepositoryFree a
  = RetrieveAllUsers ([User] -> a)
  | RetrieveUserById UserId (Maybe User -> a)
  | AddUser User (UserId -> a)
  | EditUserName UserId UserName a
  | EditUserBirthday UserId Birthday a
  | RemoveUser UserId a
  deriving stock Functor
```

Every constructor represents a potential action that we might want to do. What was the return type of the various actions in the previous approaches, here becomes the argument of a continuation.

<br>

Defining a concrete implementation means interpreting our constructors in a concrete monad

```haskell
freeInterpreter :: UserRepositoryFree a -> DB a
freeInterpreter = \case
    RetrieveAllUsers next -> next <$> retrieveAllUsers
    RetrieveUserById userId next -> next <$> retrieveUserById userId
    AddUser user next -> next <$> addUser user
    EditUserName userId userName next -> next <$ editUserName userId userName
    EditUserBirthday userId userBirthday next -> next <$ editUserBirthday userId userBirthday
    RemoveUser userId next -> next <$ removeUser userId
```

Using an interface defined through such an approach means building a value combining the above defined constructors. The implementation of the same functionality defined for the other two approaches would look like this

```haskell
freeUpdateUser :: UserId -> Maybe User -> Free UserRepositoryFree ()
freeUpdateUser userId = \case
  Nothing -> liftF $ RemoveUser userId ()
  Just user -> do
    dbUser <- liftF $ RetrieveUserById userId id
    case dbUser of
      Nothing -> do
        _ <- liftF $ AddUser user id
        pure ()
      Just _ -> do
        liftF $ EditUserName userId (name user) ()
        liftF $ EditUserBirthday userId (birthday user) ()
```

Connection the usage of the interface with a concrete implementation then looks like

```haskell
freeUsage :: UserId -> Maybe User -> DB ()
freeUsage = (foldFree freeInterpreter .) . freeUpdateUser
```

The benefit of using a free monad approach is that you're just building values with could be later interpreted. In the meanwhile, you can manipulate this values and potentially perform some static analysis on them.

<br>

On the other hand, for example, using two interfaces defined with free monads in the same functionality, proves to be more complex than what it is using two interfaces defined with one of the other approaches described above.

## Interface as a GADT

A variation of the free monad approach could be defined using a [GADT](https://wiki.haskell.org/GADTs_for_dummies) instead of a simple sum type using continuations

```haskell
data UserRepositoryGADT a where
  RetrieveAllUsers :: UserRepositoryGADT [User]
  RetrieveUserById :: UserId -> UserRepositoryGADT (Maybe User)
  AddUser :: User -> UserRepositoryGADT UserId
  EditUserName :: UserId -> UserName -> UserRepositoryGADT ()
  EditUserBirthday :: UserId -> Birthday -> UserRepositoryGADT ()
  RemoveUser :: UserId -> UserRepositoryGADT ()
```

As you can see, with respect to free monads, here constructors hold functions and not product types, and we don't have to use continuations.

<br>

Similarly to the free monads approach, a concrete implementation is an interpreter in a concrete monad

```haskell
userReporitoryInterpreter :: UserRepositoryGADT a -> DB a
userReporitoryInterpreter = \case
  RetrieveAllUsers -> retrieveAllUsers
  RetrieveUserById userId -> retrieveUserById userId
  AddUser user -> addUser user
  EditUserName userId userName -> editUserName userId userName
  EditUserBirthday userId userBirthday -> editUserBirthday userId userBirthday
  RemoveUser userId -> removeUser userId
```

To use the interface we need to pass as an argument a potential interpreter

```haskell
gadtUpdateUser :: Monad m => (forall a. UserRepositoryGADT a -> m a) -> UserId -> Maybe User -> m ()
gadtUpdateUser f userId = \case
  Nothing -> f $ RemoveUser userId
  Just user -> do
    dbUser <- f $ RetrieveUserById userId
    case dbUser of
      Nothing -> do
        _ <- f $ AddUser user
        pure ()
      Just _ -> do
        f $ EditUserName userId (name user)
        f $ EditUserBirthday userId (birthday user)
```

Connecting the usage to a concrete implementation becomes a matter of passing a function argument

```haskell
gadtUsage :: UserId -> Maybe User -> DB ()
gadtUsage = gadtUpdateUser userReporitoryInterpreter
```

With respect to free monads this approach is easier to use and does not require understanding `Free`, but it forces our domain code to depend on an interpreter (the `f` in the previous snippet. It's not a concrete interpreter, so the decoupling is still happening).

## Conclusion

This post presents four techniques for declaring interfaces in Haskell and briefly discusses and compares them. I'm fairly sure that there are other way people use to declare abstractions and decouple them from their implementations. I'd be quite curious to learn about them, so please feel free to contact me to share with me (and possibly the rest of the community) other ways to accomplish that.

<br>

---

[^1]: A more detailed discussion on using typeclasses instead of record of functions can be found [here](https://www.haskellforall.com/2012/05/scrap-your-type-classes.html)
