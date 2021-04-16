---
layout: post
title:  "Exercises in style - 1"
author: Marco Perone
date: 2021-04-16 08:06:42 +0200
categories: post
tags: php object-oriented
comments: true
pageUrl: '"http://marcosh.github.io/post/2021/04/16/exercises-in-style-1.html"'
pageIdentifier: '"Exercises in style - 1"'
description: "Exercises-in-style-1"
image: "/img/style.png"
---

I would like to present briefly some interesting solutions to common design problems. I'll go from the most basic to the most elaborate options.

In this exercise I'm studying how to design a couple of checks `$check1 : bool` and `$check2 : bool`, which could evolve indeplendetly.

The example is intentionally simple and the solutions discussed below could make sense only in more complicated cases. We are only interested in exploring the techniques.

## Solution 1: associative array

The most standard solution is use an associative array.

```php
$checks = [
  'check1' => false,
  'check2' => true
];
```

## Solution 2: class

If you want to attach some behaviour to the three checks, it could be a good idea to encapsulate it all together in a class.

```php
final class Checks
{
	/** @var bool */
	private $check1;

	/** @var bool */
	private $check2;

	public function __construct(
		bool $check1,
		bool $check2
	) {
		$this->check1 = $check1;
		$this->check2 = $check2;
	}

	public function setCheck1(bool $check1): void
	{
		$this->check1 = $check1;
    }

	public function getCheck1(): bool
	{
		return $this->check1;
  }

	public function setCheck2(bool $check2): void
	{
		$this->check2 = $check2;
  }

	public function getCheck2(): bool
	{
		return $this->check2;
  }
}
```

## Solution 3: immutable class

If you care about state management, you could be tempted to make this immutable.

```php
final class Checks
{
	/** @var bool */
	private $check1;

	/** @var bool */
	private $check2;

	private function __construct(
		bool $check1,
		bool $check2
	) {
		$this->check1 = $check1;
		$this->check2 = $check2;
	}

	public static function new(
		bool $check1,
		bool $check2
	): self {
		return new self($check1, $check2);
	}

	public function withCheck1(bool $check1): self
	{
		$that = clone $this;
		$that->check1 = $check1;

		return $that;
  }

	public function check1(): bool
	{
		return $this->check1;
  }

	public function withCheck2(bool $check2): self
	{
		$that = clone $this;
		$that->check2 = $check2;

		return $that;
  }

	public function check2(): bool
	{
		return $this->check2;
  }
}
```

## Solution 4: monoids

The previous solution still goes around the idea of creating an instance and then modifying it (even if it does it explicitly as immutability forces you to do) to track its state changes.

Another option is just composing several instances to create the state we want.

```php
final class Checks
{
	/** @var bool */
	private $check1;

	/** @var bool */
	private $check2;

	private function __construct(
		bool $check1,
		bool $check2
	) {
		$this->check1 = $check1;
		$this->check2 = $check2;
	}

	public static function check1Ok(): self
	{
		return new self(true, false);
  }

	public static function check2Ok(): self
	{
		return new self(false, true);
  }

	public function check1(): bool
	{
		return $this->check1;
  }

	public function check2(): bool
	{
		return $this->check2;
  }

	public function and(self $that): self
	{
		return new self(
			$this->check1 && $that->check1,
			$this->check2 && $that->check2
		);
	}
}
```

You could then use this as in

```php
$someChecks = Checks::check1Ok();
$otherChecks = Checks::check2Ok();

$wholeChecks = $someChecks->add($otherChecks);
```

This provides a composable API which allows you to build complex objects starting from simple ones.
