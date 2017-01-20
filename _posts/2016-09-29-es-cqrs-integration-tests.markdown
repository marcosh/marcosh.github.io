---
layout: post
title:  "ES/CQRS integration tests"
author: Marco Perone
date:   2016-09-29 10:11:42 +0200
categories: post
tags: event-sourcing cqrs integration-tests
comments: true
pageUrl: '"http://marcosh.github.io/post/2016/09/29/es-cqrs-integration-tests.html"'
pageIdentifier: '"es-cqrs-integration-tests"'
description: "How to set up integration tests with Prooph in an ES/CQRS setting"
image: "/img/integration.jpg"
---

Recently here in [MVLabs](http://www.mvlabs.it) we started working on a new project using event sourcing and CQRS. It was quite some time that we wanted to do it
but we never had the occasion. Now, with a new project in hand, it seemed the best choice to try our luck with this new
approach.

Since the beginning, we started testing carefully our domain, checking both the happy and the less
happy paths. But we were not satisfied... there were still too many places where we could do a small mistake
and create a bug. Every time we completed a new functionality (think of a new command and everything that comes into play
with its handling), we had to create a controller to dispatch the command and check in the database if everything worked
correctly.

Our desire was to avoid to use a browser, or anyway to send a request to the framework, to check whether the system was working appropriately; moreover, we wanted to be able to determine if
the code we just wrote was working or not, independently of it being part of the domain itself (and hence carefully unit tested) or of the surrounding infrastructure. Hence we
decided to start writing some integration tests to see if all the pieces were playing nicely together.

The most important decision that we took structuring these tests was to avoid interactions with the database, using a mocked connection instead.
The main benefit of this approach is that there is no need to create an ad hoc infrastructure for running the tests;
everything is just code and we do not need, for example, to manage an in memory database to handle persistence.
This approach removes from these test the responsibility of checking the actual integration with  the database, which is delegated to end to end testing, still maintaining a good control of the queries that would be executed. 

In this post I would like to share how we created integration tests for our ES/CQRS system and how they helped at controlling the correctness of our application.



## Our setting

As I said, for this project we are using an ES/CQRS approach. We decided to use [Prooph](https://github.com/prooph) as a
set of components for managing the ES/CQRS infrastructure and [Doctrine](http://www.doctrine-project.org/) with [Postgres](https://www.postgresql.org/) for the persistence of both the write and the read model. Around everything we are using [Zend Expressive](http://zendframework.github.io/zend-expressive/) to handle Http related concerns.

## An integration test

Suppose we want to create an integration test to check the command `AddItem`, which is handled by the `Order` aggregate, and, if completed correctly, produces an `ItemWasAdded` event.

If you prefer, you can follow along the whole code [here](https://gist.github.com/marcosh/bcf36f8fd485d19152a141481c20192d).

Let's start by creating a new test

    <?php

    namespace MyTestNamespace;

    final class AddItemIntegrationTest extends \PHPUnit_Framework_TestCase
    {
        public function testAddItem()
        {
            // the test content goes here
        }
    }

The first thing that we need to do now is to create an instance of the command and handle it with the command bus

    $orderId = OrderId::new() // generates a new OrderId value object
    $item = ItemId::new(); // generates a new ItemId value object

    $command = AddItem::fromOrderItemAndQuantity(
        $orderId,
        $item,
        7
    );

    $commandBus = (new CommandBusFactory())($container);

    $commandBus($command);

To construct the command bus we just instantiate its factory and then invoke it with an instance of `ContainerInterface`, here called `$container`, which will be used in the factory to retrieve the needed dependencies. To be as near as possible to the real behaviour of our application, and also to test the container configuration, we will use the real container configuration

    // retrieve configuration
    $config = [];

    {% raw %}
    $files = Glob::glob(
        'config/autoload/{{,*.}global,{,*.}local}.php',
        Glob::GLOB_BRACE
    );
    {% endraw %}
    foreach ($files as $file) {
        $config = ArrayUtils::merge($config, include $file);
    }

    $container = new ServiceManager();
    (new Config($config['service_manager']))->configureServiceManager($container);

    // Inject config
    $container->setService('config', $config);

Here we are using Zend `ServiceManager` to implement `ContainerInterface`. Any other container satisfying the interface will work.

At this point we are actually using the real configuration in its entirety. This means also the real database connection, and this is something we should avoid while testing. To take care of this, we need to override the Doctrine connection in the configuration of our container

    $container->setFactory(
        Connection::class,
        function (ContainerInterface $container) use ($orderId, $item) {
            $connection = \Mockery::mock(Connection::class);

            return $connection;
        }
    );

We are using [Mockery](https://github.com/padraic/mockery) to return a mocked connection, on which we could easily make assertions. Mocking the database connection allows us not to worry about setting up a testing infrastructure with, maybe, an in-memory database which would need to be constructed and destroyed every time we run our integration tests. Still, we have a good level of control on all the queries which will be executed by the application.

The first methods that we need to mock are the following

    $connection->shouldReceive('getTransactionNestingLevel')->andReturn(0);
    $connection->shouldReceive('beginTransaction');

which are calls made by the framework to check that a transaction is not in place and, if that is the case, start a new one.

Then we need to mock the retrieval of events from the event store

    $queryBuilder = Mockery::mock(QueryBuilder::class.['execute'], [$connection]);

    $statement = Mockery::mock(Statement::class);
    $statement->shouldReceive('setFetchMode');

    $customer = CustomerId::new(); // generates a new CustomerId value object

    $statement->shouldReceive('fetch')->once()->andReturn([
        'event_name' => OrderWasCreated::class,
        'event_id' => Uuid::uuid4(),
        'aggregate_id' => (string) $orderId,
        'payload' => json_encode([
            'customer' => (string) $customer
        ]),
        'created_at' => date_create()->format('Y-m-d\\TH:i:s.u'),
        'version' => 1
    ]);
    $statement->shouldReceive('fetch')->andReturn(false);

    $queryBuilder->shouldReceive('execute')->andReturn($statement);
    $connection->shouldReceive('createQueryBuilder')->andReturn($queryBuilder);

For every event we want to be in the history of our aggregate, we need a specific `shouldReceive('fetch')->once()` assertion, returning the event with its payload. Eventually we need to return `false` to the `fetch` call to say that we are done retrieving events.

At this point we have our aggregate rebuilt from its history; now it needs to decide if the provided command is acceptable or not. To do this it may need to access the read model. For example, if the `Order` aggregate needs to check if the required item is in stock, we may need to retrieve this information from the read model

    $connection->shouldReceive('fetchColumn')->with(
        'SELECT availability FROM item_availability WHERE id = :id',
        ['id' => (string) $item]
    )->andReturn(10);

In this case the quantity available in stock is bigger than the requested one, so the command can be processed and the item can be added to the order.

The result of the command handling by the aggregate is a set of events that need to be immediately persisted in the event store.

    $connection->shouldReceive('insert')->with(
        'event_stream',
        \Mockery::on(function (array $record) use ($orderId, $item) {
            return $record['event_name'] === ItemWasAdded::class &&
                $record['aggregate_id'] === (string) $orderId &&
                $record['aggregate_name'] === Order::class &&
                $record['payload'] === json_encode([
                    'item' => (string) $item,
                    'quantity' => 7
                ]) &&
                $record['causation_name'] === AddItem::class &&
                $record['version'] === 2;
        })
    );

Using a callback we are able to check the details of the event we are storing, controlling for example that our event was generated by the expected aggregate while handling the expected command. Another thing we should be careful about is the version associated to the event we are persisting; it should always be consecutive to the previous version.

Once the event is safely stored in the store, we need to manage projections. Suppose we now want to update the table in our read model where we keep all the order items data. First thing we will probably need to retrieve some other data from our read model, like the customer data or the item data

    // RETRIEVE THE CUSTOMER DATA
    $connection->shouldReceive('fetchAssoc')->with(
        'SELECT customer_id, customer_name, customer_email '.
        'FROM orders WHERE id = :id',
        ['id' => (string) $orderId]
    )->andReturn([
        'customer_id' => (string) $customer,
        'customer_name' => 'marco perone',
        'customer_email' => 'marco@perone.com'
    ]);

    // RETRIEVE THE ITEM DATA
    $connection->shouldReceive('fetchAssoc')->with(
        'SELECT name, price, currency FROM items WHERE id = :id',
        ['id' => (string) $item]
    )->andReturn([
        'name' => 'item name',
        'price' => 156,
        'currency' => 'EUR'
    ]);

The last thing that remains to be done now is to update the read model. In our case, this means inserting a new line in the `order_items` table

    $connection->shouldReceive('insert')->with('order_items', [
        'order_id' => (string) $orderId,
        'customer_id' => (string) $customer,
        'customer_name' => 'marco perone',
        'customer_email' => 'email@domain.com',
        'item_id' => (string) $item,
        'item_name' => 'item name',
        'item_price' => 156,
        'item_currency' => 'EUR',
        'quantity' => 7
    ]);

As you see we are duplicating information across the database. This is really helpful to improve read speed from our database. In fact, our read model could be thought as a cache layer where we query our data, and it could be easily reconstructed just by replaying the history of our application through our projections.

## Conclusion

Our test is now complete. We are checking that all the components, except the database, are working nicely together. With these integration tests we are both checking that our domain logic works, even if that is more thoroughly checked with our unit tests, and that our infrastructure connects all the pieces in the correct way.

If we would like to test our system on a even higher level, we could start writing some end to end functional test, but that is a story for another post.