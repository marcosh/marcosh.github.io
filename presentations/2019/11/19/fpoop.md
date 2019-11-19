# FUNCTIONAL PILLS FOR OO DEVELOPERS

---

## Marco Perone

[twitter.com/marcoshuttle](https://twitter.com/marcoshuttle)

[github.com/marcosh](https://github.com/marcosh)

[marcosh.github.com](https://marcosh.github.com)

[medium.com/@marcosh](https://medium.com/@marcosh)

---

![red-blue-pill](img/MatrixBluePillRedPill.jpg)

---

## What are FP and OOP?

---

## Values <!-- .element: class="values" -->

Basic principles, guiding precepts

---

## Ideas <!-- .element: class="ideas" -->

Technical ideas to realise values

---

## Style

Coherent unions of values and ideas

---

## OOP style

Encapsulation <!-- .element: class="values fragment" data-fragment-index="1" -->

Cohesiveness <!-- .element: class="values fragment" data-fragment-index="2" -->

Classes as data structures <!-- .element: class="ideas fragment" data-fragment-index="3" -->

Interfaces for abstraction <!-- .element: class="ideas fragment" data-fragment-index="4" -->

---

## FP style

Correctness <!-- .element: class="values fragment" data-fragment-index="1" -->

Ability to reason about the code <!-- .element: class="values fragment" data-fragment-index="2" -->

Referential transparency <!-- .element: class="ideas fragment" data-fragment-index="3" -->

Compositionality <!-- .element: class="ideas fragment" data-fragment-index="4" -->

Everything is a value <!-- .element: class="ideas fragment" data-fragment-index="5" -->

---

## Can we use some ideas of FP in OOP?

---

## #1 Equational Reasoning <!-- .element: class="ideas" -->

---

## What does `=` mean?

```haskell
a = f x
```

---

## Can we interchange equal things?

```haskell
a = f x

# is the program

g a

# equivalent to

g (f x)

```

---

## Code as equations

Executing the code means solving a set of equations

Easy to reason about the code

---

### How?

The only job of functions is returning a value

Same input => same output

---

## #2 Immutability <!-- .element: class="ideas" -->

---

## Immutability

Never mutate the state of an object

---

## Why?

What is the value of `$name`?

```php
$person = new Person('Gigi', 'Zucon');

doSomethingToPerson($person);

$name = $person->name();
```

---

## How?

Never mutate the state of an object

Always return a new copy

```php
public function changeName($newName) : self
{
  $newPerson = clone $this;

  $newPerson->name = $newName;

  return $newPerson;
}
```

---

## #3 Compositionality <!-- .element: class="ideas" -->

---

## Compositionality

Ensure composition is zero-cost operation

---

## Benefit

Easy way to decompose and re-compose

Modularity

---

## How?

Composition over inheritance? Not enough

Fluent interfaces? [Meh...](https://ocramius.github.io/blog/fluent-interfaces-are-evil/)

---

### Again, how?

Model your domain with compositional structures

Monoids, categories, ...

---

### And again, how?

Pure functions create naturally a category

Function composition as glue

---

## #4 Everything is a value <!-- .element: class="ideas" -->

---

## Functions are values

```php
$f = function (int $x) : int {return $x + 5;}
```

---

## Programs are values

Programs as inputs/outputs of other programs

---

## Abstraction

We can now abstract over inner computation

```haskell
decorator : (a -> b) -> a -> b

continuation : a -> (a -> b) -> b
```

---

## How?

Every modern language has lambdas

Be careful with the typing

---

## #5 Make invalid states impossible <!-- .element: class="ideas" -->

---

## Avoid impossible states

Better model design

Reduce need for error handling

---

## How?

Use the correct data structure

Value objects

---

## How? (where you can)

Algebraic data types

```haskell
data Player = First | Second

data Point = Love | Fiftween | Thirty

data Score
  = Points Point Point
  | Forty Player Point
  | Deuce
  | Advantage Player
  | Game Player

---

## How? (where you can't)

[Visitor pattern](https://blog.ploeh.dk/2018/06/25/visitor-as-a-sum-type/)

Named constructors

---

## #6 Good ad-hoc polymorphism <!-- .element: class="ideas" -->

---

## Aren't interfaces enough?

Mmmm... [no](https://diogocastro.com/blog/2018/06/17/typeclasses-in-perspective/)

---

## Extensibility

Ability to implement interfaces for classes out of our control

---

## Conditional implementation

```haskell
instance Eq a => Eq [a]

instance (Eq a, Eq b) => Eq (a, b)
```

---

## Return type polymorphism

```java
Optional<String>  fromJson(Json s) { ... }
Optional<Integer> fromJson(Json s) { ... }
```

---

## Relation between types

```haskell
class Cast a b where
  cast :: a -> b

class Mult a b c where
  mult :: a -> b -> c
```

---

## So, how?

Sorry... ¯\\\_(ツ)\_/¯

Take a look at typeclasses

---

## #7 Abstract on abstractions <!-- .element: class="ideas" -->

---

## An example

```haskell
sum : [Int] -> Int
sum []        = 0
sum (x :: xs) = x + sum xs
```

---

## Generalise on the type

```haskell
fold : (Monoid a) => [a] -> a
fold []        = 0
fold (x :: xs) = x + fold xs
```

---

## Generalise on the data structure

```haskell
fold : (Monoid a, Foldable t) => t a -> a
```

---

## Higher-kinded types

Abstract on data structures

---

## How?

Parametric polymorphism is not enough

Sorry... ¯\\\_(ツ)\_/¯

---

## #8 Effects as data <!-- .element: class="ideas" -->

---

## Effects as data

Control what your program can do, so that you can reason about it

---

## What can this function do?

```haskell
f : a -> b
```

---

## What can this function do?

```haskell
f : a -> IO b
```

---

## What can this function do?

```haskell
f : a -> State s b
```

---

## What can this function do?

```haskell
f : a -> Maybe (State s b)
```

---

## What can this function do?

```haskell
f : Member (MyEff a) r => a -> Eff r b
```

---

## How?

Describe your effects as data structures

---

## Failure

```php
final class Maybe {
  private $isJust;
  private $value;

  static function just($value) : self
  static function nothing() : self

  function match(callable $onJ, callable $onN)
}
```

---

## Asynchronicity

```typescript
export interface Task<A> {
  (): Promise<A>
}
```

---

<!-- .slide: class="no-margin" -->

## Ideas

Equational Reasoning <!-- .element: class="ideas fragment" data-fragment-index="1" -->

Immutability <!-- .element: class="ideas fragment" data-fragment-index="2" -->

Compositionality <!-- .element: class="ideas fragment" data-fragment-index="3" -->

Everything is a value <!-- .element: class="ideas fragment" data-fragment-index="4" -->

Make invalid states impossible <!-- .element: class="ideas fragment" data-fragment-index="5" -->

Good ad-hoc polymorphism <!-- .element: class="ideas fragment" data-fragment-index="6" -->

Abstract on abstractions <!-- .element: class="ideas fragment" data-fragment-index="7" -->

Effects as data <!-- .element: class="ideas fragment" data-fragment-index="8" -->

---

![solong](img/solong.jpg)

---

## Marco Perone

[twitter.com/marcoshuttle](https://twitter.com/marcoshuttle)

[github.com/marcosh](https://github.com/marcosh)

[marcosh.github.com](https://marcosh.github.com)

[medium.com/@marcosh](https://medium.com/@marcosh)
