---
layout: post
title:  "Functional data validation in PHP"
author: Marco Perone
date:   2018-12-01 20:06:42 +0200
categories: post
tags: functional-programming php dsl validation
comments: false
pageUrl: '"http://marcosh.github.io/post/2018/12/01/functional-data-validation.html"'
pageIdentifier: '"functional-data-validation"'
description: "Functional data validation"
image: "/img/valid.jpg"
---

Data validation is one of the things I was never fully satisfied about. Doing it manually generally resulted in a lot of repetitive and ugly code. Even using libraries I was never able to obtain a solution which pleased me.

So this time I decided to write a library on my own and, since I'm pretty satisfied with the result, I'd like to present it here. Mainly I'd like to share the design process and the principles which guided it.

## WHAT IS VALIDATION?

In many software projects it happens to receive untrusted data from a client. Before passing it to our domain services, which need to process them, we need to ensure they satisfy the rules we would like to hold.

Moreover, if the validation fails, we want to return some meaningful and precise error messages to inform the user about what is wrong with the provided data.

Therefore we can sum up data validation as follows:
- receive some data from the client
- check whether the incoming data satisfy the domain rules we want to impose
- if the validation suceeds, return the validated data
- if the validation fails, return some error messages

## THE INTERFACE

We can now try to translate the description above into code. We do it writing down an interface

```php
interface Validation
```

The first point in our description of data validation required us to receive some data from a client. This means that we need to have a method, which we call `validate`, that is going to receive some data as an input parameter.

```php
interface Validation
{
    /**
     * @param mixed $data
     */
    public function validate($data);
}
```

We can not be more precise on our type hinting on the input parameter, because we actually want to receive any possible input, only to return some error message if the received data do not satisfy our requirements.

What about the return type of our `validate` method? The third point of our description above tells us that it should be some validated data, while the fourth point tells us that it should be some error messages. How can we return two such different things from the same method? Maybe we should split our method in two, one for the valid case, one for the invalid one... This is a temptation we should definitely resist, bacause we wouldn't know what to do if the client used the wrong method (e.g. the method for the valid case, but with some invalid data); or we could throw exceptions, but that is something else we should try to avoid, because it makes the code not referentially transparent and hence less composable.

Hence what should we do insted? We could take advantage of the idea present in my blog post on [Maybe in PHP]({% post_url 2017-10-17-maybe-in-php-2 %}) and create a class which will represent explicitely the fact that we could have one of two possible cases. We will call this class `ValidationResult`.

```php
final class ValidationResult
{
    /**
     * @var bool
     */
    private $isValid;
    
    /**
     * @var mixed
     */
    private $validContent;
    
    /**
     * @var array
     */
    private $messages;
    
    private function __construct(
        bool $isValid,
        $validContent,
        array $messages
    ) {
        $this->isValid = $isValid;
        $this->validContent = $validContent;
        $this->messages = $messages;
    }
    
    public static function valid($validContent): self
    {
        return new self(true, $validContent, []);
    }
    
    public static function errors(array $messages): self
    {
        return new self(false, null, $messages);
    }
}
```

As you could see, there are two different ways to build such a class, through the two named constructors, one for the valid case, one for the invalid one. The valid case will contain the valid data, while the invalid case will contain an array of error messages.

Hence we can update our `Validation` interface as follows

```php
interface Validation
{
    /**
     * @param mixed $data
     * @return ValidationResult
     */
    public function validate($data): ValidationResult;
}
```

Our interface is now complete, but at the moment we have no way to process the result of our validation, since it does not expose in any way neither the valid data nor the error messages.

At this point we have a similar problem to the one we described above. If we add to the `ValidationResult` class a `validData` method to retrieve the validated data, we could actually be in the invalid case, so we won't have any meaningful result to return, and a similar reasoning could apply adding an `errors` method. In other terms, this suggests us that we can not access the validated data or the error messages independently. Still, nothing forbids us to access them simultaneously, or better, be ready to process them both at the same time! We could achieve this as follows

```php
final class ValidationResult
{
    ...

    /**
     * @param callable $processValid : validData -> mixed
     * @param callable $processErrors : errorMessages -> mixed
     * @return mixed
     */
    public function process(
        callable $processValid,
        callable $processErrors
    ) {
        if (! $this->isValid) {
            return $processErrors($this->messages);
        }

        return $processValid($this->validContent);
    }
}
```

The `process` method receives as input two callables. The first is a function which receives as input the valid data and process them as needed. The second one instead receives an array of error messages.

This allows us to handle a `ValidationResult` and obtain from it whatever we want. Most importantly, it forces us to handle both the success and the failure case.

I need to highlight here that callables are not typed in PHP, so the language is not going to check for you that the client passed to the `process` methods something that will actually work.

## A SIMPLE EXAMPLE

Now that we defined how we would like things to be, let's just give an example to show how they work in practice. Let's try to define a validator to check if some input data is actually a `string`.

```php
final class IsString implements Validation
{
    public function validate($data): ValidationResult
    {
        if (! is_string($data)) {
            return ValidationResult::errors(['THIS IS NOT A STRING!']);
        }

        return ValidationResult::valid($data);
    }
}

$isString = new IsString();

$isString->validate('a string'); // ValidationResult::valid('a string')

$isString->validate(42); // ValidationResult::errors(['THIS IS NOT A STRING!'])
```

Pretty self-explanatory, isn't it?

## COMBINATORS

Validating one single rule was pretty easy, but what about validating two or more? We need a way to combine simple validators to create more complex ones. We can do exactly this using so called combinators. In our case a combinator will be a new validator built from simpler validators. Let's see some examples.

For example, we could build a `Sequence` combinator which processes an array of validators. `Sequence` will process them sequentially, one after the other. As soon as one validator fails, `Sequence` will fail returning its error messages. If all the composed validator succeed, `Sequence` will suceed too.

```php
final class Sequence implements Validation
{
    /**
     * @var Validation[]
     */
    private $validations;

    public function __construct(array $validations)
    {
        $this->validations = $validations;
    }

    public function validate($data): ValidationResult
    {
        return array_reduce(
            $this->validations,
            function (ValidationResult $carry, Validation $validation) {
                return $carry->process(
                    function ($validData) use ($validation) {
                        return $validation->validate($validData);
                    },
                    function () use ($carry) {
                        return $carry;
                    }
                );
            },
            ValidationResult::valid($data)
        );
    }
```

As another example, we could also build a `All` combinator which, similarly to `Sequence`, processes an array of validators. The diffence lies in the fact that `All` will process all of them, and return the error messages of all failed validators if one of them fails.

Similarly we can build many other interesting combinators, which will allow us to create really interesting combinations and use them for practically every use case we could encounter.

## CONCLUSION

I hope I was able to describe how to implement data validation in a simple but effective way. The ideas exposed in this post are hugely inspired by functional programming concepts and drew their biggest inspiration from the [Data.Validation](https://hackage.haskell.org/package/validation-1/docs/Data-Validation.html) Haskell data structure.

I also wrote a library to implement these ideas in PHP: [https://github.com/marcosh/php-validation-dsl](https://github.com/marcosh/php-validation-dsl). It provides several basic validators and some combinators. Moreover it handles some more complicated cases that the ones explained above. I really encourage you to take a look at it, try it out and let me know if you feel that something is missing or should be improved.
