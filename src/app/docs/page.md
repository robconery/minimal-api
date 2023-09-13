# Quickstart

Before getting started here, be sure to install the [.NET 6 SDK](https://dotnet.microsoft.com/download/dotnet/6.0).

Next, please install the following:

- [.NET 6 SDK](https://dotnet.microsoft.com/download/dotnet/6.0)
- An editor of your choice, such as [VS Code](https://code.visualstudio.com/) or [Visual Studio](https://visualstudio.microsoft.com/)

Once you have done that, open a terminal such as PowerShell, Command Prompt, or bash. Run the following command to create your first app:

```bash
dotnet new web --output minimalapp
```

This command will create a new directory with a few files to get you started:

1. _Program.cs_: This is your first code file we will be editing for most of this guide.
1. _MinimalApp.csproj_: This is a project file that helps the `dotnet` command know how to run your code. We will ignore it for this guide as it has everything it needs already.
1. _appsettings.json_: This is a JSON configuration file with some initial settings for your app, we also will not be using these in this guide but you can use them to add your own settings like, for example, a connection string to get data from a database.

## Your app

Getting to know your app.

The app in your _Program.cs_ looks like this:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.Run();
```

## What have we built?

In four lines of code

1. **`var builder = WebApplication.CreateBuilder(args);`**

  You create an app builder, which is used to configure the app. In this default example, you aren't doing any configuration yet, so just build an `app` object. You use the builder to create an app and then you run the app, this is known as the builder pattern.

2. **`var app = builder.Build();`**

  Build the `app` object, the `app` object is what we will use to route URLs to code.

3. **`app.MapGet("/", () => "Hello World!");`**

  You call `MapGet`, which is how you route URLs to code.

4. **`app.Run();`**

  Finally, `app.Run` executes the app you configured in the previous lines. It's not until you call `Run` that your app will start and you can browse to URLs.

## Running your app

To run your app, use the `dotnet run` command on your terminal in the same directory as the _Program.cs_ file.

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

## What if I see something different?

If you haven't changed any code in the _Program.cs_ and your app still fails to run then it is likely a problem with the installation of `dotnet`. But there are a couple of common things you might see:

- If you see an error saying `Couldn't find a project to run` then you are probably in the wrong directory, make sure your terminal is in the right place.
- If you see an error saying `Failed to bind to address https://127.0.0.1:5001: address already in use` then you probably have another `dotnet run` command running on another terminal window. You can stop that app by pressing `CTRL + C`. You can only have one app listening on a given address on a computer. We will talk about how to change the URL your app is listening on a bit later.