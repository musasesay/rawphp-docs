---
title: System Error Handler
---

Things go wrong. You can't predict errors, but you can anticipate them. Each RawPHP Framework application has an error handler that receives all uncaught PHP exceptions. This error handler also receives the current HTTP request and response objects, too. The error handler must prepare and return an appropriate Response object to be returned to the HTTP client.

## Default error handler

The default error handler is very basic. It sets the Response status code to `500`, it sets the Response content type to `text/html`, and appends a generic error message into the Response body.

This is _probably_ not appropriate for production applications. You are strongly encouraged to implement your own RawPHP application error handler.

The default error handler can also include detailed error diagnostic information. To enable this you need to set the `displayErrorDetails` setting to true:

## How to turn error display on or off

You can do this by specifying it in your `config/DatabaseConfig.php`
```
$configuration = [
    'settings' => [
        'displayErrorDetails' => true, //add this
    ],
];
```
Make sure this is set to false when deploying to production.

## Custom error handler

A RawPHP Framework application's error handler is a Pimple service. You can substitute your own error handler by defining a custom Pimple factory method with the application container.

There are two ways to inject handlers:

### Pre App

Specify this in your `config/ContainerConfig.php
```
$container['errorHandler'] = function ($container) {
    return function ($request, $response, $exception) use ($container) {
        return $container['response']->withStatus(500)
                             ->withHeader('Content-Type', 'text/html')
                             ->write('Something went wrong!');
    };
};
```

### Post App
```
$container['errorHandler'] = function ($container) {
    return function ($request, $response, $exception) use ($container) {
        return $container['response']->withStatus(500)
                             ->withHeader('Content-Type', 'text/html')
                             ->write('Something went wrong!');
    };
};
```

In this example, we define a new `errorHandler` factory that returns a callable. The returned callable accepts three arguments:

1. A `\Psr\Http\Message\ServerRequestInterface` instance
2. A `\Psr\Http\Message\ResponseInterface` instance
3. A `\Exception` instance

The callable **MUST** return a new `\Psr\Http\Message\ResponseInterface` instance as is appropriate for the given exception.

### Class-based error handler

Error handlers may also be defined as an invokable class as a middleware.

```
class CustomHandlerMiddleware extends Middleware{
   public function __invoke($request, $response, $exception) {
        return $response
            ->withStatus(500)
            ->withHeader('Content-Type', 'text/html')
            ->write('Something went wrong!');
   }
}
```

Remember to define a shorthand with the name `errorHandler` for it in `config/ContainerSettings.php`


### Handling other errors

**Please note**: The following four types of exceptions will not be handled by a custom `errorHandler`:

- `Slim\Exception\MethodNotAllowedException`: This can be handled via a custom [`notAllowedHandler`](/docs/handlers/not-allowed.html).
- `Slim\Exception\NotFoundException`: This can be handled via a custom [`notFoundHandler`](/docs/handlers/not-found.html).
- Runtime PHP errors (PHP 7+ only): This can be handled via a custom [`phpErrorHandler`](/docs/handlers/php-error.html).
- `Slim\Exception\SlimException`: This type of exception is internal to Slim, and its handling cannot be overridden.

### Disabling

To completely disable RawPHP's error handling, simply remove the error handler from the container in `config/ContainerConfig.php`:

```
unset($app->getContainer()['errorHandler']);
unset($app->getContainer()['phpErrorHandler']);
```

You are now responsible for handling any exceptions that occur in your application as they will not be handled by RawPHP.
