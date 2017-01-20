---
layout: post
title:  "Constructing objects with PDO"
author: Marco Perone
date:   2017-01-05 15:03:13 +0200
categories: post
tags: php pdo
comments: true
pageUrl: '"http://marcosh.github.io/post/2017/01/05/pdo-fetch-func.html"'
pageIdentifier: '"pdo-fetch-func"'
description: "How to use PDO::FETCH_FUNC to create custom objects directly from PDO queries"
image: "/img/object-table.jpeg"
---

One of the most common non-trivial issue that needs to be solved in a project is deciding how to build domain objects starting from database data.

Usually the approach I took was to use [Doctrine](http://www.doctrine-project.org/) and delegate to it the whole process of constructing objects when retrieving data from the database. For a really small project I had to work on lately, I had just one table and using Doctrine really seemed too much of a burden. Hence I decided to use directly [PDO](http://php.net/manual/en/book.pdo.php) to interact with the database.

## The database and the domain

As I mentioned earlier, my database consisted just of one table, called `availability`, and actually it was quite a big one. It had more or less 30 columns and it contained more than a million rows.

The only query I had to perform on it was a simple `SELECT * FROM availability WHERE ...` with filtering conditions depending on user input.

On the domain side, I had just one entity, which looked like this

    final class Availability
    {
        /**
         * @var \DateTimeImmutable
         */
        private $from;

        /**
         * @var \DateTimeImmutable
         */
        private $to;

        /**
         * @var Room
         */
        private $room;

        /**
         * @var Beds
         */
        private $beds;

        /**
         * @var Price
         */
        private $price;

        /**
         * @var Structure
         */
        private $structure;

        public function __construct(
            \DateTimeImmutable $from,
            \DateTimeImmutable $to,
            Room $room,
            Beds $beds,
            Price $price,
            Structure $structure
        ) {
            $this->from = $from;
            $this->to = $to;
            $this->room = $room;
            $this->beds = $beds;
            $this->price = $price;
            $this->structure = $structure;
        }

        ...
    }

where `Room`, `Beds`, `Price` and `Structure` were value objects with their specific properties.

The columns in my `availability` table corresponded exactly to the data necessaty to build an `Availability` object.

Now the problem was how to build my `Availability` objects when retrieving data from the database.

## The options

If we have a look at the [`PDOStatemet::fetch` documentation](http://php.net/manual/en/pdostatement.fetch.php) we see that we have various options regarding the format of the results of a query; the two possibilities are retrieving each row of the result as an object or as an array.

Looking closer, we see that both `PDO::FETCH_CLASS`, `PDO::FETCH_INTO` and `PDO::FETCH_OBJ` use the named properties of a class to map the columns to the results. This did not work for me, since my `Availability` class did not have properties corresponding to the table columns, as they actually were inside the value objects.

So the best option to keep my domain intact would have been to retrieve the data from `PDO` as arrays and manually build my value objects and my entity. I did not like too much this approach, so I looked a bit further.

## The hidden option, `PDO::FETCH_FUNC`

Reading more carefully the documentation of `PDO`, I discovered that there is actually another option that we could use instead of the ones listed in the documentation of the `fetch` method. It is called `PDO::FETCH_FUNC` and it is mentioned in the [documentation of the `PDOStatement::fetchAll` method](http://php.net/manual/en/pdostatement.fetchall.php). It allows to pass to `PDO` a callable that is invoked for every row of the result set. It could be used to format the results of a query, as described in the **Example #5** [here](http://php.net/manual/en/pdostatement.fetchall.php).

Another possibility is to pass as callable a constructor of an entity, so that `PDO` itself would return us valid entities.

So I just added a named constructor to my entity, like

    public static function fromNativeData(
        $from,
        $to,
        ...
    ) {
        // validation of the input
        Assert::...

        // contruction of the value objects and of the entity
        $availability = new self(...);

        return $availability;
    }

where each input parameter corresponded to a column in my `availability` table.

This allowed me to keep my domain clean with my value obejcts and validate the data arriving from the database.

## Comparing with `PDO:FETCH_ASSOC`

I was a bit worried about the performance of this approach, so I did a little benchmark and I compared it to using `PDO::FETCH_ASSOC` and then manually building my objects cycling through the query results.

You can find the results [here](https://gist.github.com/marcosh/4177e21ef0e29c7c5e84a57a1e6d9333). For every case I tested I tried with 10, 100, 1000, 10000 and 100000 returned rows, and for each one of this possibilities I run the program 5 times.

For `FETCH_FUNC` I tried with and without validation of the data and for `FETCH_ASSOC` I measured retieving the data as plain arrays, as objects but without validation and as object with validation.

As expected, `FETCH_ASSOC` with results as arrays is the fastest option. On the other hand, what I did not expect was to see a significative difference between `FETCH_FUNC` and `FETCH_ASSOC` with class creation, both whith and without validation. We are talking about seconds here!

## Conclusions

I really liked using `FETCH_FUNC` as it allowed me to keep my domain model clean, with its value objects and its validation. Even the code is really concise, since it's enough to use

    $availabilities = $statement->fetchAll(
        PDO::FETCH_FUNC,
        [Availability::class, 'fromNativeData']
    );

From the performance point everything is fine and there in no bottleneck in using `FETCH_FUNC`.

The only thing that needs to be taken care of is the order of the parameters in the named contructor, since `PDO` will not do any type checking and just fed the values into the variables.

