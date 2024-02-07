# Setting Up OpenAPI and Swagger

[Swagger](https://swagger.io) is a toolset for helping you document your API using a JSON or XML manifest. The project was donated to OpenAPI back in 2015, but work on the supporting tools is still maintained by [SmartBear](https://smartbear.com).

By creating a Swagger manifest, you can offer your API developers a great experience when learning how to use your API:

![](/images/swagger_1.jpg)

You can click through to each endpoint to read a description of the endpoint, what parameters it expects (if any), and what types of responses you might get:

![](/images/swagger_2.jpg)

In this example, we're told we have to supply a string value to the `POST`, and we're going to get back a chunk of data as JSON. We're also told that we can expect a 200, but not a 404 or 500 when calling this endpoint.

This is very useful, and as you can imagine the manifest is quite detailed. You could opt to set this up by hand, but thankfully there are a few tools at our disposal that will do this for us.

## Setting Up The Swagger Manifest

There are two main tools for setting up a Swagger manifest using code: [NSwag](https://github.com/RicoSuter/NSwag) and [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle.AspNetCore). We use Swashbuckle here at Tailwind Traders because that's what we know, and we like the way it works.

### Installation

The first step is to install the Nuget package:

```sh
dotnet add package Swashbuckle.AspNetCore
```

Once installed, you need to add the swagger services to your builder:

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Version = "0.0.1",
        Title = "Tailwind Traders Mail Services API",
        Description = "Transactional and bulk email sending services for Tailwind Traders.",
        Contact = new OpenApiContact
        {
            Name = "Rob Conery, Aaron Wislang, and the Tailwind Traders Team",
            Url = new Uri("https://tailwindtraders.dev")
        },
        License = new OpenApiLicense
        {
            Name = "MIT",
            Url = new Uri("https://opensource.org/license/mit/")
        }
    });
});
```
Here, we're specifying that we want to use the API explorer (what you see in the pictures above) as well as the generator. We're almost done!

The final step is to tell our application to use Swagger and to also mount the Swagger UI:

```csharp
app.UseSwagger();
app.UseSwaggerUI(options =>
{
  options.SwaggerEndpoint("/swagger/v1/swagger.json", "v1");
  options.RoutePrefix = string.Empty;
});
```

This will show the Swagger endpoint to the public, which may or may not be your goal. Here, I'm setting the default route of my application to the Swagger UI by setting the `RoutePrefix` to `string.Empty`. Set this to whatever endpoint works best for you.

### Exposing the API Documentation

If your application comes with a developer API, Swagger can offer the kind of detail that developers need. If, however, your API is only for internal use, you might want to wrap the above code in an `if` statement that checks for the runtime environment.


## Creating Endpoint Documentation

Swashbuckle creates your API manifest at application runtime in one of two ways:

 - Looking for method attributes
 - Directly using a fluent interface

Typically, with an ASP.NET web application, you would decorate your `IActionResult` methods with attributes in order to build your Swagger manifest:

```csharp
[HttpPost("{id}")]
[ProducesResponseType(typeof(Product), 200)]
[ProducesResponseType(typeof(IDictionary<string, string>), 400)]
[ProducesResponseType(500)]
public IActionResult GetById(int id)
```

The first attribute is the route, which is an ASP.NET thing, the last three are Swashbuckle specific, and detail what the response types are for each status code.

ASP.NET Minimal API works a bit differently than other ASP.NET frameworks, as you've seen. Instead of having well-defined classes that you can stack attributes on, you have a fluent style of declaring your endpoints:

```csharp
app.MapGet("/", () => "Hello World!");
```

This means we can't use the attribute approach (well, we *can* but it requires .NET 8 and a few editor gymnastics) so, instead, we need to use the fluent style.

Here's the code for the `POST` method above:

```csharp
app.MapPost("/admin/validate", ([FromBody] ValidationRequest req) => {
  //...
  var response = new ValidationResponse{
    Valid = true,
    Data = doc,
    Contacts = contacts
  };
  return response;
  
}).WithOpenApi(op => {
  op.Summary = "Validate the markdown for an email";
  op.Description = "Before you send a broadcast, ping this endpoint to ensure that the markdown is valid";
  op.RequestBody.Description = "The markdown for the email";
  return op;
});
```

As you can see, if we use the `WithOpenApi` extension method, we're given an OpenAPI object that we can then manipulate for this endpoint. Here, I'm giving it a summary, description, and a description for the request body. That's all I need to do!

There is an alternate way to do this as well, which is to use explicit extension methods:

```csharp
.WithSummary("Validate the markdown for an email")
.WithDescription("Before you send a broadcast, ping this endpoint to ensure that the markdown is valid")
```

While useful, this style is a bit verbose and not as capable as `WithOpenApi`.

### Request and Response Types

Our endpoint is designed to validate a markdown document and all it expects is a string value posted to it. We *could* set things up this way, but a well-documented API should have typed values for both request and response.

Here are the ones we're using for this endpoint:

```csharp
public class ValidationResponse{
  public bool Valid { get; set; }
  public string Message { get; set; }
  public long Contacts { get; set; }
  public MarkdownEmail? Data { get; set; }
  public ValidationResponse()
  {
    Message = "The markdown is valid";
  }
}

public class ValidationRequest{
  public string? Markdown { get; set; }
}
```

By providing a typed request and response, the Swagger UI is able to show our API developers schemas that provide much better information:

![](/images/swagger_4.jpg)

### Adding Response Types

Right now, our API is declaring that it will always return a 200 response (OK), which indicates that some type of status message will be returned no matter what. This is rarely the case - problems always happen!

For instance: the user might do something they're not allowed to, or we might lose connectivity to the database. We can handle these things in code, but we should also let our API developers know what will happen if these errors occur.

To do this, we can add the `Produces` methods to our endpoint, along with the status codes:

```csharp
.WithOpenApi(op => {
  op.Summary = "Validate the markdown for an email";
  op.Description = "Before you send a broadcast, ping this endpoint to ensure that the markdown is valid";
  op.RequestBody.Description = "The markdown for the email";
  return op;
})
.Produces<ValidationResponse>(StatusCodes.Status200OK)
.Produces(StatusCodes.Status400BadRequest)
.Produces(StatusCodes.Status403Forbidden)
.Produces(StatusCodes.Status500InternalServerError);
```

We now have an updated API response list:

![](/images/swagger_5.jpg)

As you can see, our `ValidationResponse` will be returned each time, which tells our developers that if an error occurs, we're going to catch it and provide some detail.
