---
layout: post
title:  "Applicative and monadic validation in PHP"
author: Marco Perone
date: 2019-04-10 08:06:42 +0200
categories: post
tags: functional-programming php applicative monadic validation
comments: true
pageUrl: '"http://marcosh.github.io/post/2019/04/10/applicative-monadic-validation-php.html"'
pageIdentifier: '"applicative-monadic-validation-php"'
description: "Applicative and monadic validation in PHP"
image: "/img/valid1.jpg"
---

In my [last post]({% post_url 2018-12-01-php-validation-dsl %}) I described a way to structure data validation inspired by the principles of functional programming, in particular immutability and compositionality. Such an approach allows to validate any kind of data in a safe and stateless way, building more and more complex validators starting from basic ones.

I believe it is a really nice approach, more general and elastic than many other PHP libraries which do the same thing. The only drawback I see is that, for achieving explicitness and precision, it is quite verbose. Even if data validation is a very important component of your application, it will probably never be the core domain of your business and therefore many people will prefer a slightly less general library which instead saves them learning and coding time.

In this post I'm going to present a way, actually two, to define complex validators in term of simpler ones, which is really easy and terse. I'm going to do this adapting two validation techniques which are widespread in the functional programming world; they are called applicative and monadic validation, but you need not to be scared by the names! As we will see, there's nothing exoteric beyond them. Still, if you're curios where these names come from, you could have a look [here](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)

## THE VALIDATION INTERFACE

Let's start by recalling how we defined our `Validation`s. They are identified by the following interface

```php
interface Validation
{
    public function validate($data): ValidationResult;
}
```

In other terms, a validator receives some `$data`, which could be anything, not necessarily an array or an object, and returns a `ValidationResult` which, in its essence, is defined as follows

```php
final class ValidationResult
{
    public static function valid($validContent): self

    public static function errors(array $messages): self

    /**
     * @param callable $processValid : $validContent -> mixed
     * @param callable $processErrors : $messages -> mixed
     * @return mixed
     */
    public function process(
        callable $processValid,
        callable $processErrors
    )
```

In practice a `ValidationResult` could be either valid, and in that case it contains the result of the validation, or invalid, and contain some error messages. There is only one way to access the result or the error messages, using the `process` method which forces you to consider both cases at the same time.

This setting leads to a safe working environment, where you never have access to terms when you shouldn't and you are guided to handle correctly the validation of your data.

## APPLICATIVE AND MONADIC VALIDATION

Suppose we have in our domain code a class which represents a customer; in particular, for the sake of this example, it will have an id and an email address.

```
class CustomerInfo
{
    /** @var CustomerId */
    private $id;

    /** @var EmailAddress */
    private $emailAddress;

    public function __construct(
        CustomerId $id,
        EmailAddress $emailAddress
    )
}
```

Our domain requires the id to be a positive integer and the email a string containing an `@` symbol. These domain rules are enforced in the named constructors of the `CustomerId` and `EmailAddress` value objects, respectively.

```
class CustomerId
{
    /** @var int should be positive */
    private $id;

    /**
     * @param int $id
     * @return ValidationResult containing a CustomerId
     */
    public static function buildValid(int $id): ValidationResult
}

class EmailAddress
{
    /** @var string should contain "@" */
    private $email;

    /**
     * @param string $email
     * @return ValidationResult containing an EmailAddress
     */
    public static function buildValid(string $email): ValidationResult
}
```

The question now is: how can we define a validator for `CustomerInfo`? We need a function which takes as arguments a integer and a string, and returns a `ValidationResult` containing a `CustomerInfo`, if both values are valid, and some error messages if at least one of the two parameters is invalid.

We want to do this using just the named constructors for `CustomerId` and `EmailAddress` and the named constructor for `CustomerInfo`. The first two ingredients should allow us to pass from a integer and a string to a `CustomerId` and an `EmailAddress` and the latter combines the two to obtain a `CustomerInfo`. It looks doable! The trick is to find the right function which allows us to actually do it.

There is actually one thing we have not defined properly, yet. We still did not say how we want to treat the error messages; in particular, what should happen if both the validations of `CustomerId` and `EmailAddress` fail? Do we want to return to our end user both the error messages or just the first one? This is actually the key difference between applicative and monadic validations! Let's see them both more closely and try to understand which are the pros and cons of both approaches.

### Applicative validation

Applicative validation allows you to validate separately several elements and then compose them using a function which acts on their results. In our examples we have

```
CustomerId::buildValid(int $id): ValidationResult // containing a CustomerId

EmailAddress::buildValid(string $email): ValidResult // containing an EmailAddress
```

which validates raw data and wraps it in a `ValidationResult`. Moreover, we also have the `CustomerInfo` constructor

```
CustomerInfo::__construct(CustomerId $id, EmailAddress $address)
```

which takes a `CustomerId` and an `EmailAddress` and returns a `CustomerInfo`.

If we try to compose these functions, we soon realize we have a little problem. We have a `CustomerId` and an `EmailAddress` wrapped in a `ValidationResult`, while the `CustomerInfo` constructor acts on the unwrapped values. We want to avoid unwrapping the `ValidationResult`s, because that would force us to manage manually the error cases and the merge of the error messages, which is exactly what we would like to hide.

Instead what we could do is lift the `CustomerInfo` costructor to the `ValidationResult` level and obtain a function which receives as arguments a `ValidationResult` containing a `CustomerId` and a `ValidationResult` containing an `EmailAddress` and returns a `ValidationResult` containing a `CustomerInfo`.

It is in fact possible to define a [`lift` function](https://github.com/marcosh/php-validation-dsl/blob/e55f0d651fb035f73ec3ef48d657efc7694874ce/src/Result/functions.php#L18) which does exactly what we need! Hence we can obtain

```
/**
 * @param ValidationResult containing a CustomerId
 * @param ValidationResult containing an EmailAddress
 * @return ValidationResult containing a CustomerInfo
 */
lift(function (CustomerId $id, EmailAddress $emailAddress) {
    return new self($id, $emailAddress);
})
```

At this point it becomes easy to compose everything and obtain a function which takes an integer and a string and returns a `ValidationResult` containing a `CustomerInfo` as follows

```
public static function buildValidApplicativeWithLift(int $id, string $email): ValidationResult
{
    return lift(function (CustomerId $id, EmailAddress $emailAddress) {
        return new self($id, $emailAddress);
    }) (
        CustomerId::buildValid($id),
        EmailAddress::buildValid($email)
    );
}
```

In case of errors, `lift` will automatically collect all the error messages and return them together.

### Monadic validation

Applicative validation is sweet, but it does not allow sequential validation, since all pieces are processed independently. But that's exactly what the monadic style comes to solve!

Monadic validation means specifying a series of steps, each one returning a `ValidationResult`, and processing them one by one having the possibility to reuse in the following steps the results of the previous ones.

In our example this would look as follows

```
public static function buildValidMonadicWithDo(int $id, string $email): ValidationResult
{
    return mdo(
        function () use ($id) {
            return CustomerId::buildValid($id);
        },
        function (CustomerId $id) use ($email) {
            return EmailAddress::buildValid($email);
        },
        function (CustomerId $id, EmailAddress $email) {
            return ValidationResult::valid(new self ($id, $email));
        }
    );
}
```

In the first step we validate the `CustomerId`. If it fails, we return the error messages and we are done. If it succeeds, we pass its unwrapped result to the next step. In the second step we do the same, we compute the validation and, if it fails, we return the error messages; if it succeeds, we pass both the results of both steps to the third one and we compute its validation.

As you can understand, the flow here is quite different from the applicative one, since steps are processed sequentially and at the first failure we short circuit all the computation. What we gain instead is the possibility of reusing results from previous steps.

### Monadic validation, take 2

If you have a closer look in the [library](https://github.com/marcosh/php-validation-dsl/blob/master/src/Result/functions.php), you will see that there are actually two different functions for monadic validation. We took a look at `mdo`; its peculiarity is that at each step it requires to receive all the results of the previous steps. If you have several steps, this could become quite awkward, with all those parameters. If instead you are happy to receive at each step only the result of the previous step, you could use `sdo` which looks as follows:

```
function validateZipCodeWithCountryAndProvince(
    string $country,
    string $province,
    string $zipCode
): ValidationResult {
    return sdo(
        function () use ($country) {
            return Country::validate($country);
        },
        function (Country $country) use ($province) {
            return Province::validateForCountry($country, $province);
        }, function (Province $province) use ($zipCode) {
            return ZipCode::validateForProvince($province, $zipCode);
        }
    );
}
```

In this case we really need each time only the previous result to continue our validation steps, so this is the perfect use case for `sdo`.

## Conclusion

I must say that I am pretty happy about how this validation library came out and I am fairly confident to say that its approach is kind of unique in the PHP world. With the introduction of applicative and monadic validation now we have the possibility to write validators in a really easy and expressive way.

Still, PHP is not the best language for this kind of thing especially because writing anonymous functions is still quite verbose. I guess for that we will all need to wait for the approval of the [arrow_function](https://wiki.php.net/rfc/arrow_functions_v2) proposal which could land already in PHP 7.4. It would be just some syntactic sugar, but for doing functional programming related stuff, it would be really really helpful.

Anyway, writing nice, expressive, safe, and composable validators is already possible! Have fun with doing it!