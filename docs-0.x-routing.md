---
layout: documentation
title: Routing - CrowPHP an expressive framework for PHP
permalink: /docs/0.x/routing
toc: true
---

# Routing
Routing refers to determining how an application responds to a client request to a particular endpoint, 
which is a URI (or path) and a specific HTTP request method (GET, POST, and so on).

Each route can have one or more handler functions, which are executed when the route is matched.

Route definition takes the following structure:
```php
$router->{method}({path}, {handler});
```

* $router is an instance of `Crow\Router\Factory::make()`.
* {method} is an HTTP request method, in lowercase.
* {path} is a path on the server.
* {handler} is the function executed when the route is matched.

This tutorial assumes that an instance of CrowServer named `$app` and `$router` is created and the server is running.
If you are not familiar with creating an app and starting it, see the Hello world example on the home page.

The most basic CrowPHP routes accept a URI and a closure, providing a very simple and expressive method of defining routes and behavior without complicated routing configurations.

## Route methods
A route method is derived from one of the HTTP methods, and is attached to an instance of the `router` class.

The following code is an example of routes that are defined for the GET and the POST methods to the root of the app.
```php
// GET method
$router->get('/', function (RequestInterface $request, ResponseInterface $response) {
    $response->getBody()->write('Hello World get request');
    return $response;
});

//POST method
$router->post('/', function (RequestInterface $request, ResponseInterface $response) {
    $response->getBody()->write('Hello World post request');
    return $response;
});
```
CrowPHP supports methods that correspond to all HTTP request methods: get, post, and so on. For a full list, see $router->{method}.

## Route paths
By default the route path pattern uses a syntax where {foo} specifies a placeholder with name foo and matching the regex [^/]+. 
To adjust the pattern the placeholder matches, you can specify a custom pattern by writing {bar:[0-9]+}. Some examples:

```php
// Matches /user/42, but not /user/xyz
$router->get('/user/{id:\d+}', 'handler');

// Matches /user/foobar, but not /user/foo/bar
$router->get('/user/{name}', 'handler');

// Matches /user/foo/bar as well
$router->get('/user/{name:.+}', 'handler');
```

Custom patterns for route placeholders cannot use capturing groups. 
For example {lang:(en|de)} is not a valid placeholder, because () is a capturing group. 
Instead you can use either {lang:en|de} or {lang:(?:en|de)}.

Furthermore parts of the route enclosed in [...] are considered optional, so that /foo[bar] will match both /foo and /foobar.
Optional parts are only supported in a trailing position, not in the middle of a route.

```php
// This route
$router->get('/user/{id:\d+}[/{name}]', 'handler');
// Is equivalent to these two routes
$router->get('/user/{id:\d+}', 'handler');
$router->get('/user/{id:\d+}/{name}', 'handler');

// Multiple nested optional parts are possible as well
$router->get('/user[/{id:\d+}[/{name}]]', 'handler');

// This route is NOT valid, because optional parts can only occur at the end
$router->get('/user[/{id:\d+}]/{name}', 'handler');

```

The $handler parameter does not necessarily have to be a callback, it could also be a controller class name or any other kind of data you wish to associate with the route. FastRoute only tells you which handler corresponds to your URI, how you interpret it is up to you.
Shortcut methods for common request methods

For the GET, POST, PUT, PATCH, DELETE and HEAD request methods shortcut methods are available. For example

```php
$router->get('/get-route', 'get_handler');
$router->post('/post-route', 'post_handler');
```

Is equivalent to:

```php
$router->addRoute('GET', '/get-route', 'get_handler');
$router->addRoute('POST', '/post-route', 'post_handler');
```

## Route parameters

Sometimes you will need to capture segments of the URI within your route. For example, you may need to capture a user's ID from the URL. You may do so by defining route parameters:

```php
$router->get('/user/{id}', function (RequestInterface $request, ResponseInterface $response, $id) {
    $response->getBody()->write('my name is='.$name);
    return $response;
});
```

You may define as many route parameters as required by your route:

```php
$router->get('/posts/{post}/comments/{comment}', function (
    RequestInterface $request, 
    ResponseInterface $response, $post, $comment) 
{
    //
});
```

Route parameters are always encased within {} braces and should consist of alphabetic characters. Underscores (_) are also acceptable within route parameter names. Route parameters are injected into route callbacks / controllers based on their order - the names of the route callback / controller arguments do not matter.

## Route groups

Additionally, you can specify routes inside of a group. All routes defined inside a group will have a common prefix.

For example, defining your routes as:
```php
$router->addGroup('/admin', function (RouterInterfave $router) {
    $router->get('/do-something', 'handler');
    $router->get('/do-another-thing', 'handler');
    $router->get('/do-something-else', 'handler');
});
```

Will have the same result as:

```php
$router->get('/admin/do-something', 'handler');
$router->get('/admin/do-another-thing', 'handler');
$router->get( '/admin/do-something-else', 'handler');
```

Nested groups are also supported, in which case the prefixes of all the nested groups are combined.
