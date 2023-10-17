# Routing

When building a web application you typically want to try and create meaningful URLs that execute your code. For example, going to `/hello` returns a hello string and going to `/todos` or `/todolist` shows me all the todo items I have. It doesn't matter what URL you use as long as it makes sense to you and you think it will make sense to your users.

In your ASP.NET Core program, you use the `Map` methods to create an endpoint that binds a URL to our code:

```csharp
app.MapGet("/hello", () => "Hello World!");
app.MapGet("/todos", () => new { TodoItem = "Learn about routing", Complete = false });
```

> If you run the app with the above `/todos` route and view it in a browser, you'll see that the framework has automatically converted the todo item to JSON.

## HTTP Verbs

As with most modern frameworks, ASP.NET Routing will match on the following [HTTP request verbs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods):

 - GET: `MapGet()`,
 - POST: `MapPost()`,
 - PUT: `MapPut()`,
 - DELETE: `MapDelete()`
 - PATCH: `MapPatch()`

If you're new to web development and don't know what this means, nor why you should care, just know that it's common to handle form posts using a `POST` verb, everything else is usually a `GET`. 

Each of these methods returns a `RouteHandlerBuilder`, which implements an `IEndpointRouteBuilder`. This will be important to us later on!

## Dynamic Routes

Like any modern web framework, you can create dynamic routes by indicating the dynamic route segment in braces:

```csharp
app.MapGet("/hello/{name}", (string name) => $"Hello {name}");
```

This endpoint will match URLs like `/hello/david` would return `Hello david`. 

## Route Matching Precedence

Routing in ASP.NET is done via _sequential matching_, which means that each route pattern (the first argument in the route declaration) is tested, and if it matches, then the _route handler_ (the second argument to the route declaration) is fired.

To see this, let's add this route to our application:

```csharp
app.MapGet("/hello", () => "Hello World!");
app.MapGet("/todos", () => new { TodoItem = "Learn about routing", Complete = false });
app.MapGet("/hello/{name}", (string name) => $"Hello {name}");
```

If we navigate to `/hello`, we'll match the first route and we'll see `Hello World` as the response. The route `/hello/david` doesn't match the first route, only the last one, so we'll see `Hello david` as described above.

If no routes match, we'll get a `404 Not Found` error. But what happens if more than one route matches? We (thankfully) will get an error!

![image](https://user-images.githubusercontent.com/234688/128390787-b3ab9769-a0c4-4a67-9d16-716bc52b4416.png)

In this image, we have two `hello` routes and the framework can't tell which code you want it to run, so it throws an error. Remember that `hello/` and `hello` are the same as far as ASP.NET Core is concerned, the end slash doesn't make them different.

## Constraints

You can specify the type of a variable by doing `/hello/{name:int}`, which would mean that the route would no longer match if you navigated to `/hello/David` but would match if you navigate to `/hello/1` because we said that our name variable should be an `int` in the route. These are called [Route Constraints](https://docs.microsoft.com/aspnet/core/fundamentals/routing?view=aspnetcore-5.0#route-constraint-reference) and allow you to have fine control over what code gets run when users enter different types of URLs.

## Next Up: Data

Every web application needs to work with data somehow, so [let's dig into that now](/home/data)