# Powered By ASP.NET Minimal API

Minimal APIs are a simplified approach for building fast HTTP APIs with ASP.NET Core. You can create fully functioning REST endpoints with minimal code and configuration.

ASP.NET Minimal APIs are built using C#, an open-source, modern, object-oriented, and type-safe programming language you'll love. 

You can build an API in C# with just **3 lines of code**:

```csharp
var app = WebApplication.Create(args);

app.MapGet("/", () => "Hello World!");

app.Run();
```

Minimal code patterns have been popular in modern web frameworks for a while. If you're familiar with these frameworks (Express, Flask, Sinatra) then you'll feel right at home with ASP.NET Minimal API.

For example, here is how routing works with Express, the NodeJS web framework:

```js
// GET method route
app.get('/', function (req, res) {
  res.send('GET request to the homepage')
})

// POST method route
app.post('/', function (req, res) {
  res.send('POST request to the homepage')
})
```

Here are those same routes with MinimalAPI:

```csharp
app.MapGet("/", () => "GET request to the homepage");
app.MapPost("/", () => "POST request to the homepage");
```

## Ready To Learn More?

Getting started with ASP.NET Minimal API is pretty straightforward! Let's start out by [getting to know what a basic application looks like](/home).

