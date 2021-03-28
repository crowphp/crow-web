---
layout: documentation
title: Middlewares - CrowPHP an expressive framework for PHP
permalink: /docs/0.x/middlewares
toc: true
---

Middleware provide a convenient mechanism for inspecting and filtering HTTP requests entering your application. For example, Laravel includes a middleware that verifies the user of your application is authenticated. If the user is not authenticated, the middleware will redirect the user to your application's login screen. However, if the user is authenticated, the middleware will allow the request to proceed further into the application.

For example, CrowPHP by default uses a routing middleware and an exception handling middleware, CrowPHP follows `PSR-15` standard for middlewares so you can also use many readily available middlewares that implement `Psr\Http\Server\MiddlewareInterface` and its also very easy to write your custom middleware for example, a logging middleware might log all incoming requests to your application.

## Defining a middleware

To create a new middleware, create a new `PHP` file and for this example we will call it `EnsureTokenIsValid.php`. We will create a new class `EnsureTokenIsValid` that implements 

```php
<?php
namespace Middlewares;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;
use RingCentral\Psr7\Response;

class EnsureTokenIsValid implements MiddlewareInterface
{

    public function process(
        ServerRequestInterface $request, 
        RequestHandlerInterface $handler): ResponseInterface
    {
        if ($request->getHeader('token') !== 'my-secret-token') {
            return new Response(403);
        }
        return $handler->handle($request);
    }
}
```

Now we can apply our new middleware to any route by simply calling `middleware` method after the route.

```php

$router->post('/posts', function (RequestInterface $request, ResponseInterface $response) {
    $response->getBody()->write('valid token');
    return $response;
})->middleware(new EnsureTokenIsValid());

```
In the example above we are applying the middleware that is following `PSR-15` standards you can also pass `callables` instead.

```php
$router->post('/post', function (RequestInterface $request, ResponseInterface $response) {
    $response->getBody()->write('My Post');
    return $response;
})->middleware(function (RequestInterface $request, RequestHandlerInterface $next) {
    echo "Logging here in the /post request\n";
    return $next->handle($request);
}));

```

## Types of middlewares

CrowPHP has different types of middlewares and its important to understand them since they can be used for different purposes.

* Global Middlewares
* Group-level Middlewares
* Route-level Middlewares

In the examples above we have learned about the `Route-level` middlewares, what if you want to apply the same middleware to multiple routes and that's where `Group-level` middlewares comes in to the rescue.

We can create a route group and after the first two `required` arguments we can pass an `n` number of arguments on `route-group` for middlewares which can be again a `PSR-15` standard middleware or a `callable`. All of the middlewares passed will be executed in the sequence that they are passed to the `route-group` before finally passing the request to the request handler function of each route inside the group and ofcourse it can be nested with more middlewares passed to the sub routes or sub-groups with in.

```php

$router->addGroup('/admin', function (RouterInterface $router) {
    $router->get('/posts/id/{id}', function (
        RequestInterface $request, 
        ResponseInterface $response, $id): ResponseInterface {

        $response->getBody()->write(json_encode(["post_id" => $id]));
        return $response->withHeader('Content-Type', 'application/json')->withStatus(403);
    })->middleware(function (RequestInterface $request, RequestHandlerInterface $next) {
        echo "This is a local middleware 1 for sunny\n";
        return $next->handle($request);
    });

    $router->get('/comments/id/{id}', function (
        RequestInterface $request, 
        ResponseInterface $response, $id): ResponseInterface {
        $response->getBody()->write('comments ' . $id);
        throw new Exception('Hey i am an exception');
    });

}, function (RequestInterface $request, RequestHandlerInterface $next) {
    echo "This is a group middleware 1\n";
    return $next->handle($request);
}, function (RequestInterface $request, RequestHandlerInterface $next) {
    echo "This is a group middleware 2\n";
    return $next->handle($request);
});

```

Last but not the least are the `global` middlewares and as the name suggests these middlewares are applied to all the requests that enter the server.
To define the `global` middleware we use the `use` method on the `app` instance of `CrowServer`.

```php
$app->use(function (RequestInterface $request, RequestHandlerInterface $next) {
    echo "This is a global middleware 1\n";
    return $next->handle($request);
});

$app->use(function (RequestInterface $request, RequestHandlerInterface $next) {
    echo "This is a global middleware 2\n";
    return $next->handle($request);
});
```

Like other middlewares you can pass `PSR-15` compliant middlewares or `callables` to the `use` method and the execution will be in the sequence in which they are passed.


