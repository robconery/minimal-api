## Basic Routing

When building a web application you typically want to try and create meaningful URLs that execute your code. For example, going to `/hello` returns a hello string and going to `/todos` or `/todolist` shows me all the todo items I have. It doesn't matter what URL you use as long as it makes sense to you and you think it will make sense to your users.

In your ASP.NET Core program, you use the `Map` methods to create an endpoint that binds a URL to our code:

```c
app.MapGet("/hello", () => "Hello World!");
app.MapGet("/todos", () => new { TodoItem = "Learn about routing", Complete = false });
```

> If you run the app with the above `/todos` route and view it in a browser, you'll see that the framework has automatically converted the todo item to JSON.

### Route Variables

You can add variables to routes using curly braces like `{id}`:

```c
app.MapGet("/hello/{name}", (string name) => $"Hello {name}");
```

This endpoint will match URLs like `/hello/David` would return `Hello David`. If you add this method to the ones shown earlier, then navigating to `/hello` would return `Hello World!` as the two endpoints have different routes and match differently.

> If you have two routes the same then your application will still run, but when you try to navigate to those routes you will get an error like:
![image](https://user-images.githubusercontent.com/234688/128390787-b3ab9769-a0c4-4a67-9d16-716bc52b4416.png)
In this image, we have two `hello` routes and the framework can't tell which code you want it to run, so it throws an error. Remember that `hello/` and `hello` are the same as far as ASP.NET Core is concerned, the end slash doesn't make them different.

### Constraints

You can specify the type of a variable by doing `/hello/{name:int}`, which would mean that the route would no longer match if you navigated to `/hello/David` but would match if you navigate to `/hello/1` because we said that our name variable should be an `int` in the route. These are called [Route Constraints](https://docs.microsoft.com/aspnet/core/fundamentals/routing?view=aspnetcore-5.0#route-constraint-reference) and allow you to have fine control over what code gets run when users enter different types of URLs.

## HTTP Methods Overview

So far we've shown `MapGet`, which allows you to specify a HTTP Get action, which is what a browser sends when you go to a URL. But there are other HTTP methods you are likely to want and you can use other Map methods to get those, for example `MapPost` or `MapPut`. The other Map methods work the same as `MapGet` that we've already seen but responds to a post or put respectively. You can learn more about [HTTP request methods here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods). For now, think of using `Post` when sending data to the app and `Get` when getting data from the app.

```c
app.MapGet("/hello", () => "Hello World!");
app.MapGet("/todos", () => new { TodoItem = "Learn about routing", Complete = false });
app.MapPost("/todos", () => Results.Ok());
```

Notice that `MapPost` and `MapGet` in this example are using the same URL. That is because the different verbs can all use the same URL and ASP.NET Core will invoke the right code for you. What this allows is for you to create a URL for your todos and use GET to get todos and POST to add a todo.
