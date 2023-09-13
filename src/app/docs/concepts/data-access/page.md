## Working with Data

OK, so if you use `MapPost` to send data to the code, how does that work?

### Model Binding

The most common way of accessing data in ASP.NET Core is to create classes and let ASP.NET Core fill them in for you:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/hello", () => "Hello World!");
app.MapGet("/todos", () => new { TodoItem = "Learn about routing", Complete = false });

app.MapPost("/todos", (Todo todo) => todo.Name);

app.Run();

class Todo
{
    public string Name {get;set;}
    public bool IsComplete {get;set;}
}
```

This code will send the name part of aIf I then use [Thunder Client]() in VS Code to send a request to the `todos` URL then it would look like this:

![image](https://user-images.githubusercontent.com/234688/129420178-3bbeefd0-1f62-4b24-9163-ec8e3c6a3cba.png)

ASP.NET will try to create an instance of the class you say you need from any JSON in the `body` of a request. It will also automatically convert any objects you return to JSON, like in our `GET` todos URL above.

### HttpContext

Another more basic way of accessing information, and data, from a request is using the `HttpContext` object. ASP.NET Core creates a `HttpContext` for each request, and you can access it in your code like this:

```csharp
app.MapGet("/hello/{name}", (HttpContext ctx) => $"Hello {ctx.Request.RouteValues["name"]}");
```

In this code, you are accepting an `HttpContext` and using it to manually access the route value rather than letting ASP.NET Core automatically match it like you did in the previous example. As well as all route values the `HttpContext` has access to all request information, like the body and cookies. You can read from the request property of HttpContext and write to the Response property, which ASP.NET Core does for you in all of the examples before this one.

## Returning HTML

If you want to return some HTML rather than processing JSON like you've been doing so far, ASP.NET Core uses a language called [Razor](), which is a mix of C# and HTML to make authoring UI easier. So let's add Razor to your app:

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddRazorPages();
var app = builder.Build();

app.MapGet("/todos", () => new { TodoItem = "Learn about routing", Complete = false });
app.MapPost("/todos", (Todo todo) => todo.Name);

app.MapRazorPages();

app.Run();

class Todo
{
    public string Name {get;set;}
    public bool IsComplete {get;set;}
}
```

Then create a folder called `Pages` and a file in that folder called `Index.razor` with this content:

```html
@page "/page/route"

<html>
    <body>
        <div>
            This is some content.
        </div>
    </body>
</html>
```

Then if you run your application and navigate to `/` you will see your content:

![image](https://user-images.githubusercontent.com/234688/129422595-ad395a02-662f-46a3-beed-5f87cff6c774.png)

This is because `Index` is the name of the default URL that a browser will try to load. If you called your `cshtml` file `SomethingElse.cshtml` then you would navigate to `/SomethingElse` to see the content.