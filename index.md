---
layout: home
title: Home
permalink: /
---

CrowPHP is a fast async web framework built on-top of SwoolePHP. You can now build web services with just the CLI version of PHP on the platform of your choice.

## Installation
#### Requirements

Before we begin with the installation make sure you have following installed on your system.

* PHP-CLI version 8 or above.
* composer package manager for PHP
* PHP pear package manager `pecl`
* SwoolePHP latest version `pecl install swoole`

Installing Crow is very easy through `composer`. Create a new directory if you are starting a new project and initialize composer as follows `composer init`.

Once your project is initialized you can now install the Crow framework as follows:

```bash
$ composer install crowphp/crow
```
Now that we have installed Crow, we can create our first `hello world` microservice, create a new file `index.php` with the following code snippet.

{% highlight php startinline=true %}
<?php

require 'vendor/autoload.php';

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface as RequestInterface;
use Crow\Http\Server\Factory as CrowServer;

$app = CrowServer::create(CrowServer::SWOOLE_SERVER);
$router = Crow\Router\Factory::make();

$router->get('/', function (RequestInterface $request, ResponseInterface $response) {
    $response->getBody()->write('Hello World');
    return $response;
});

$app->withRouter($router);

$app->listen(5005);

{% endhighlight %}

You may quickly test your newly built service as follows:
```bash
$ php index.php
```
Going to http://localhost:5005 will now display "Hello World".
