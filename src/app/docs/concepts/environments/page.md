## Environments

Consider this screenshot:

![image](https://user-images.githubusercontent.com/234688/129421155-fb148b66-30ee-4156-ad5c-52c84f31ccac.png)

Here, you can see the terminal output saying `Hosting environment: Development`. This is controlled by the environment variable `ASPNETCORE_ENVIRONMENT`. If the `ASPNETCORE_ENVIRONMENT` is not set, then ASP.NET Core assumes the value is `Production`, when using `dotnet run` or editors like VS Code or Visual Studio they will set the value to `Development`.

The error handling page showed above only appears when the environment is `Development`. You can check the environment value yourself like this:

```csharp
if (app.Environment.IsDevelopment())
{
    app.MapGet("/OnlyInDev",
        () => "This can only be accessed when the app is running in development.");
}
```

You can set the environment variable to whatever value you want and add whatever logic you like to your app based on the environment.

```csharp
if (app.Environment.EnvironmentName == "TestEnvironment")
{
    app.MapGet("/OnlyInTestEnvironment", () => "TestEnvironment");
}
```

The pre-defined values for environment are `Development`, `Staging`, and `Production`. `Development` is set by development time tooling like `dotnet run` and we assume that the environment is `Production` if it isn't set to anything. ASP.NET Core will add the error handling page to your app if the environment is `Development`.