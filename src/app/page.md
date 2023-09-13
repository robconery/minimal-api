---
title: ASP.NET Minimal API
---

Build powerful, simple APIs with C# and .NET {% .lead %}

{% quick-links %}

{% quick-link title="Concepts" icon="plugins" href="/docs/concepts" description="Minimal API is all about keeping it simple - here are the basics you should understand" /%}

{% quick-link title="Architecture guide" icon="presets" href="/docs/architecture" description="How do you structure your API? Here are some tips." /%}

{% quick-link title="Advanced Topics" icon="installation" href="/docs/advanced" description="Moving beyond Todo list stuff, let's dig in to the details." /%}

{% quick-link title="Templates" icon="theming" href="/docs/starters" description="Get off the ground quickly with our prebuilt starter templates." /%}

{% /quick-links %}

---

## What's a Minimal API?

Minimal APIs are a simplified approach for building fast HTTP APIs with ASP.NET Core. You can create fully functioning REST endpoints with minimal code and configuration.

```csharp
var app = WebApplication.Create(args);

app.MapGet("/", () => "Hello World!");

app.Run();
```

## Great for Microservices

Minimal APIs are great for building microservices for your applications fast.

## C# Ecosystem

Minimal APIs is built using C#. An open-source, modern, object-oriented, and type-safe programming language you'll love. Build an API in C# with just **3 lines of code**.

```csharp
var app = WebApplication.Create(args);

app.MapGet("/", () => "Hello World!");

app.Run();
```

## Routing

Minimal routes by design. Create meaningful low ceremony URLs that execute your code.

```csharp
app.MapGet("/", () => "Hello World!");
```

## Feels familiar

Minimal code patterns have been popular in JavaScript and Python web frameworks for a while. Whether you are familiar with Express or Flask, we think you can easily do things you enjoy about those frameworks with minimal APIs as well. 

**For example**

Routes in Express 
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

Routes in minimal APIs:

```csharp
app.MapGet("/", () => "GET request to the homepage");
app.MapPost("/", () => "POST request to the homepage");
```