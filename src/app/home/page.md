# The Basic Tailwind API

Every Tailwind API is built with ASP.NET Minimal API, so it's a good idea to start there so you're familiar with what's going on.

## Installation

ASP.NET works on top of .NET, so let's be sure that's installed. You can [download the latest .NET SDK(7.0) right here.](https://dotnet.microsoft.com/en-us/download/dotnet/7.0).

You can work with .NET in any text editor, however we recommend using [VS Code](https://code.visualstudio.com/) or [Visual Studio](https://visualstudio.microsoft.com/), as they have plugins that work with C# and ASP.NET, which makes life easier.

Once you have done that, open a terminal (PowerShell, Bash, etc.) and run:

```bash
dotnet new web --output minimalapp
```

This command will create a new directory with a few files to get you started:

![The Initial Files](/images/home/demo.jpg)


 - `Program.cs`: This is your first code file we will be editing for most of this guide.
 - `MinimalApp.csproj`: This is a project file that helps the `dotnet` command know how to run your code. We will ignore it for this guide as it has everything it needs already.
 - `appsettings.json`: This is a JSON configuration file with some initial settings for your app, we also will not be using these in this guide but you can use them to add your own settings like, for example, a connection string to get data from a database.
 - `Properties/launchSettings.json` configures your server environment. When you start the application using `dotnet run`, the built-in web server (named Kestrel) will look at this file to figure out how to set things, like port number and application URL. 
 - The `bin` and `obj` directories contain build output and can be safely ignored.

## Taking a Look Around

The core of the application is `Program.cs`, which is the how .NET boots things up. As you can see, it's quite terse:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.Run();
```

Just as with Express and Flask, you can put your application logic right in this file, if you like, but that doesn't scale very well, especially if you're planning on having multiple endpoints.

The next file we care about is `appsettings.json`, which should look something like this:

```js
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

This configures the Minimal API runtime with default settings.

Finally, we have our "project" file, `minimalapp.csproj` (assuming you used the same `dotnet new` command above), which is actually a _build file_, telling .NET how to compile our application.

Initially, it should look something like this:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net7.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

</Project>
```


## Running The Application

To run your app, use the `dotnet run` command on your terminal in the same directory as the `Program.cs` file.

```bash
dotnet run
```

After running you will see log output like the following:

```bash
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5000
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
```

If you navigate to `http://localhost:5000` in a browser you will see the text `Hello World!` as the browser makes a request to your app and displays the output.

### What if I see something different?

If you haven't changed any code in the _Program.cs_ and your app still fails to run then it is likely a problem with the installation of `dotnet`. But there are a couple of common things you might see:

- If you see an error saying `Couldn't find a project to run` then you are probably in the wrong directory, make sure your terminal is in the right place.
- If you see an error saying `Failed to bind to address https://127.0.0.1:5001: address already in use` then you probably have another `dotnet run` command running on another terminal window. You can stop that app by pressing `CTRL + C`. You can only have one app listening on a given address on a computer. We will talk about how to change the URL your app is listening on a bit later.

## Ready For More?

Understanding how Minimal API works is crucial when working with our blocks, so let's spend a little time digging into [Routing](/home/routing) shall we?