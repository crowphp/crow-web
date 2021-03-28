---
layout: documentation
title: Getting Started - CrowPHP an expressive framework for PHP
permalink: /docs/0.x/
toc: true
---

# Why CrowPHP?

PHP has grown a lot as a language it has a mature ecosystem and a vibrant community, for PHP to become a goto language for modern-microservices and container oriented ecosystem we lacked a way to deploy our web services in a stand-alone PHP, well not any more, with the advancements in asynchronous and concurrent PHP environment provided by platforms like SwoolePHP and ReactPHP it’s not a dream any more to run your web services on stand-alone PHP and here comes CrowPHP to provide an elegant and un-opinionated framework on top of SwoolePHP and ReactPHP HTTP servers.

# Your First CrowPHP Microservice

Before we start with the crowPHP lets make sure you have the right system requirements.

* PHP-CLI version 8 or above.
* composer package manager for PHP
* PHP pear package manager `pecl`
* SwoolePHP latest version `pecl install swoole`

## Docker Example
To follow this example I am assuming that you are familiar with `docker` and `docker-compose` and the latest versions are installed on your system.

Let's start with a new `composer.json` file in your project.

```json
{
    "name": "helloworld",
    "authors": [
      {
        "name": "Your Name",
        "email": "your@email.com"
      }
    ],
    "require": {
      "crowphp/crow": "0.3.1"
    }
}
```
Now let's create a new PHP file in the root of your project `index.php`. In this file, we will be created a new CrowPHP server and a route that will serve our request.

```php
<?php

require 'vendor/autoload.php';

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface as RequestInterface;
use Crow\Http\Server\Factory as CrowServer;

$app = CrowServer::create(CrowServer::SWOOLE_SERVER);
$router = Crow\Router\Factory::make();

$router->get('/', function (RequestInterface $request, ResponseInterface $response) {
    $response->getBody()->write('Hello World!');
    return $response;
});

$app->withRouter($router);
$app->on('start', function ($server) {
    echo "CrowPHP server is listening on port $server->host:$server->port " . PHP_EOL;
});

$app->listen(5000, "0.0.0.0");

```

First, we have imported the generic `vendor/autoload.php` that will provide us with the resolutions of all the necessary `namespaces` and classes. We are using standard `PSR` request and response interfaces, which are supported by CrowPHP.

We are then creating an `$app` instance from the `CrowServer::create` method by passing a single integer constant `CrowServer::SWOOLE_SERVER` this is important because CrowPHP also supports ReactPHP server but for this example; we are going to stick with `SwoolePHP` as its more production-ready at the moment.

After that we are making a `$router` instance from the router factory i.e. `Crow\Router\Factory::make`, with router instance in the next step we call a `get` method to create a new route which accepts to parameters i.e. `{path}` and a `{handler}`. Handler is a `callback` function and it provides `$request` and `$response` instances that expect the instance of `ResponseInterface` to be returned.

Now the instance of the router is attached to the main app instance with the `$app->withRouter` method and we have attached a listener for the `start` event just to log in when the server is ready to accept new requests.

At last, we have called the `listen` method with `5000` port and `0.0.0.0` as host since we are running in the Docker container.


### Docker File

Now let's create a new `Dockerfile` in the root your project with following contents, and ofcourse you can modify it to your liking :)

```bash
FROM php:8.0-cli-buster

RUN apt update && apt upgrade -y
RUN apt install zip unzip libzip-dev git -y
# Install PHP extensions
RUN pecl install swoole \
    && pecl install zip \
    && docker-php-ext-enable swoole zip

RUN mv /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini
# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

WORKDIR /code

ADD entry.sh /usr/bin/entry.sh
EXPOSE 5000/tcp
ENTRYPOINT [ "/usr/bin/entry.sh" ]
```

If you notice we are adding a file `entry.sh` in our docker file lets create that file in the root of our project.

```bash
#!/bin/bash
composer install
php index.php
```

It can't be more simpler, we are installing composer when the container starts for the first time followed by execution of our PHP server.

### Docker Compose File

Finally to make our project easier to manage for the next time we will be using docker-compose and this step is (optional). Let's create a new `docker-composer.yaml` file in the root of the project.

```yaml
version: "3"
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
```

Now, we can finally run our project with `docker-compose up` and our service should be ready to recieve requests on `http://localhost:5000`