# Powered By ASP.NET Minimal API

Build powerful, simple APIs with C# and .NET

{% quick-links %}

{% quick-link title="Getting Started" icon="plugins" href="/home" description="Minimal API is all about keeping it simple - here are the basics you should understand" /%}

{% quick-link title="Building With Our Blocks" icon="presets" href="/docs/architecture" description="How do you structure your API? Here are some tips." /%}

{% quick-link title="Resources and Reference" icon="installation" href="/docs/advanced" description="Moving beyond Todo list stuff, let's dig in to the details." /%}

{% quick-link title="Templates" icon="theming" href="/docs/starters" description="Get off the ground quickly with our prebuilt starter templates." /%}

{% /quick-links %}

---

## What's a Minimal API?

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

## Get Started With Our Blocks

Tailwind Traders has created a set of APIs built with ASP.NET Minimal API, and we have open sourced these "blocks", as we call them, so that you can rapidly build a business of your own using our goodies.

Each one has deployment options for Microsoft Azure, so you can **move rapidly from idea to startup running on Azure in very little time**.

### Simple, Markdown CMS

We believe in the power of simplicity and it doesn't get much simpler than a bunch of markdown documents that are loaded into memory, queryable using LINQ.

### Flexible Email List Service

### eCommerce Store Powered by Stripe

### A Simple Job Queue

### Digital Fulfillment

### A Blog, Powered by Markdown
