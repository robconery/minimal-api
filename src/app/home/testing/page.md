# Testing Your API

There are two approaches to testing an ASP.NET application:

 - Using a traditional testing framework, such as Xunit, to test your application logic and
 - Using an integration testing framework, such as Playwright, to do end-to-end testing of both the frontend and the backend.

For the scope of this document, we'll focus on traditional testing with Xunit.

## Adding Testing to Your Project

It's a common practice in the .NET world to create a separate test .NET Project. Developing the `mail` application, for instance, we might create a project called `Tailwind.Mail.Tests`, adding a reference to the `Tailwind.Mail` project to it.

While useful, our blocks are small enough that it makes more sense to have the tests sit right next to the code to keep things simple and lean. We can do this by adding a few lines to our API project file:

```xml
<ItemGroup Condition="'$(Configuration)' == 'Release'">
  <Compile Remove="Tests\*Tests.cs" />
</ItemGroup>
<ItemGroup Condition="'$(Configuration)' != 'Release'">
  <PackageReference Include="Microsoft.AspNetCore.Mvc.Testing" Version="7.0.2" />
  <PackageReference Include="xunit" Version="2.4.1" />
  <PackageReference Include="xunit.runner.visualstudio" Version="2.4.3">
    <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    <PrivateAssets>all</PrivateAssets>
  </PackageReference>
  <PackageReference Include="coverlet.collector" Version="3.1.2">
    <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    <PrivateAssets>all</PrivateAssets>
  </PackageReference>
</ItemGroup>
```

This directive tells MS Build to ignore all test files when configuring for Release. For development and test environments, however, all the testing tools should be present and ready to go.

## Creating a cast of users

Every project has its own testing standards which reflect the project goals as well as your companies practices. We're sharing what we do here at Tailwind in case it might be helpful for you. This approach has worked well for us over the years, and keeps our tests from becoming too "brittle" and breaking whenever we change our code.

Each test suite tells a story, based on a user persona that we develop with our programming team. To create these people, we think about (in any order):

 - What they're doing when they show up at our application; in other words how much of their attention do we demand?
 - Their goal when coming to our app.
 - How much time we'll have with them so we can help them reach that goal.
 - Their experience with our app and our brand.
 - Their biases (we all have them). Maybe they're a lifelong user, brand new, a skeptic, someone needed econimic parity, etc.
 - Their skill level and level on the team they work on.

Using these guidelines, we come up with 3-5 personas, give them names and, if the project is large enough, some AI-generated headshots as well.

### The user stories 

For testing the Mail application, we might have a user we've created named Scott who is well-known online with hundreds of thousands of followers on his social feeds as well as in his mailing list.

Scott has heard about our project and is interested in what we offer because he's currently paying $699 USD/mo to maintain his list, and has, many times, thought about creating his own service because he's a skilled .NET developer.

This story is what drives our testing process:

```cs
public class Scott_wants_to_broadcast_to_his_list: TestBase
{
  [Fact]
  public async Task All_the_emails_are_queued()
  {
    //...
  }
  [Fact]
  public async Task Sending_250K_emails_takes_less_than_4_hours()
  {
    //...
  }
  [Fact]
  public async Task Scott_sees_the_send_process()
  {
    //...
  }
  [Fact]
  public async Task Scott_can_kill_the_process()
  {
    //...
  }
  [Fact]
  public async Task Scott_can_empty_the_queue()
  {
    //...
  }
  //etc
}
```

The `class` name is the _scenario_ and each test illustrates the events in that scenario, hopefully touching on all the possibilities and concerns Scott might have. 

### The `TestBase` class

You might have noticed that we use a `TestBase` class here. It's traditional with ASP.NET to use dependency injection, composing your services rather than using inheritence, but our needs are specific and satisfied in the simplest possible way using a base class:

```cs
public abstract class TestBase : IDisposable
{
    protected TestBase()
    {
      Environment.SetEnvironmentVariable("ASPNETCORE_ENVIRONMENT", "Test");
      DotEnv.Load();
    }

    public void Dispose()
    {
        // Do "global" teardown here
    }
}
```

Here we're making sure the environment variables are set as needed, ensuring that `ASPNETCORE_ENVIRONMENT` is set to `Test`.

## Loading Environment Variables

As you can see, we favor environmment variables using a `.env` file as opposed to `appsettings.json`. We do this because of our team's experience and background, and we'll also get a warning from GitHub if we check this file in.

If you're interested in using our `DotEnv` class, here it is:

```cs
using System;
using System.IO;

//Based on the code from here, but I added the ability to just read the .ENV from the project root
//https://dusted.codes/dotenv-in-dotnet
public static class DotEnv
{
    public static void Load()
    {
        var execDirectory = Directory.GetCurrentDirectory();
        string projectDirectory = Directory.GetParent(execDirectory).Parent.Parent.FullName;
        var filePath = Path.Combine(projectDirectory, ".env");
        if (!File.Exists(filePath)) return;
        
        foreach (var line in File.ReadAllLines(filePath))
        {
          var parts = line.Split('=',StringSplitOptions.RemoveEmptyEntries);
          if (parts.Length != 2) continue;
          Environment.SetEnvironmentVariable(parts[0], parts[1].Replace("\"", ""));
        }
    }
}
```

Drop this in the root of your project and you will have access to the environment variables set therein.