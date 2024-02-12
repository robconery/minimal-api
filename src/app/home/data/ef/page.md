# Data Access With Entity Framework

You have quite a few choices when it comes to data access, but the most common scenario when working in .NET is to use [Entity Framework](https://learn.microsoft.com/en-us/ef/), commonly called "EF", which is what we'll do here as that's how we're accessing data in our block apps.

## Installing Entity Framework

The first thing we need to do is to install EF using the command line:

```sh
dotnet tool install --global dotnet-ef
dotnet ef database update
```

Now that we've done that, we need to install the packages we'll be working with. At Tailwind Traders, we use three main databases:

 - PostgreSQL for just about everything
 - SQLServer for just about everything
 - SQLite for smaller apps and testing

That means we'll need to install the following packages:

```sh
dotnet add package Microsoft.EntityFrameworkCore.SQLite
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL  
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL.Design
```

Great, now let's set things up.

## Setting Up the Data Context

EF uses the "[Unit of Work](https://martinfowler.com/eaaCatalog/unitOfWork.html)" pattern, which is quite useful when using C# classes to model your database tables. The idea is straightforward: you "check out" instances, which correspond to records in your database, do something with them, and then record those changes back to the database in a transaction.

First, let's create a `Data` directory, which is where we'll put our database code:

```sh
mkdir Data
touch Data/Db.cs
```

_Note: if you're coming to ASP.NET from another framework, such as Ruby on Rails, you might wonder about the casing of the file names? This is a .NET convention which is what we're going to do here for our directories, files, and code_.

The `Db.cs` file is where we'll set up our "Data Context", which will track changes to our models. For this example, we'll use `Npgsql` to connect to PostgreSQL:

```cs
using Microsoft.EntityFrameworkCore;

namespace Tailwind.Data;

public class Db: DbContext
{

  protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
  {
    var connectionString = Environment.GetEnvironmentVariable("DATABASE_URL");
    optionsBuilder.UseNpgsql(connectionString);
  }
}
```

We'll add to this file when we create a model, but for now, let's consider the connection string.

## Handling Connection Strings

It's common in ASP.NET to put application configuration settings inside the `appsettings.json` file:

```js
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "ConnectionStrings": {
    "App": "Host=localhost;Database=tailwind;Username=rob;"
  },
  "AllowedHosts": "*"
}
```

This works, but we have encountered a few issues with this file being checked into source control by mistake. We want to be sure our database credentials are kept safe, so here at Tailwind Traders we have opted to use Environment Variables.

### Creating a DotEnv File

In most other web frameworks, you store sensitive information (aka "secrets") in the shell's environment. That's what we're doing to do here, and we're going to access that information only when we need it using a commonly named file called `DotEnv`, which we'll pop right into the root of our project (you can [read more about this file here](https://dusted.codes/dotenv-in-dotnet)):

```sh
touch DotEnv.cs
```

Here's our code:

```cs
using System;
using System.IO;

//based on code from https://dusted.codes/dotenv-in-dotnet
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

This code looks for a file called `.env`, which is a convention in most application frameworks, that store our environment variables. It's such a convention, as a matter of fact, that if you try to check this into GitHub you'll get a warning!

Let's create the `.env` file and, while we're at it, let's be sure it's in our `.gitignore` file, however:

```sh
touch .env
echo '\n.env' >> .gitignore
```

Great. Now let's add our connection string to the `.env` file along with the development environment setting:

```sh
ASPNETCORE_ENVIRONMENT="Development"
DATABASE_URL="postgres://localhost/tailwind"
```

This will work on Windows machines as well as Unix-based!

## Creating Our First Model

Let's create a simple `User` model, just for demonstration. Before we do, however, we'll need to create a table for it in our database. 

_Note: we could use [EF migrations](https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/?tabs=dotnet-core-cli) for this, but what we're doing is simple and straightforward, so we're just going to use SQL_.

We'll use PostgreSQL for our `User` data, so open up your favorite database tool (ours is `psql`) and add the following code:

```sql
create table users(
    id serial primary key,
    name text,
    email text not null unique
);
```

Run that SQL and you should see yourself a `users` table.

### A Note About Casing

You'll notice here that **we are not** following the .NET casing guidelines with PostgreSQL? That's because PostgreSQL has different casing rules, and working in PascalCasing isn't fun if you're not working directly in C#.

We can get around PostgreSQL's naming preference by delimiting our table and column names with double quotes:

```sql
create table "Users"(
    "Id" serial primary key,
    "Name" text,
    "Email" text not null unique
);
```

At Tailwind, we have decided to let PostgreSQL be PostgreSQL as setting up mapping rules with Entity Framework is pretty straightforward.

### The User Model

Now let's create our model file:

```cs
touch Data/User.cs
```

Next, we drop in our code:

```cs
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace Tailwind.Data.Models;

public class User
{
    [Key]
    [Column("id")]
    public int ID { get; set; }
    [Column("name")]
    public string? Name { get; set; }
    [Column("email")]
    public string? Email { get; set; }
}
```

Here we're adding _annotations_ to the properties of the class so that EF knows how to map things.

Now that we've done this, let's update our data context:

```cs
using Tailwind.Mail.Data.Models;
using Microsoft.EntityFrameworkCore;

namespace Tailwind.Mail.Data;

public class Db: DbContext
{
  public DbSet<User>? Users { get; set; }

  protected override void OnModelCreating(ModelBuilder modelBuilder)
  {
      modelBuilder.Entity<User>().ToTable("users");
  }
  protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
  {
    var connectionString = Environment.GetEnvironmentVariable("DATABASE_URL");
    optionsBuilder.UseNpgsql(connectionString);
  }
}
```

Perfect. We can now do all the CRUD (Create, Read, Update, Delete) things we need to do with our `users` table.

_Note: one nice thing about mapping in this way is that we can specify a schema name for our table by passing in the `schema: "[name]"` option to the `modelBuilder`. If you have a large database, dividing things out by schema names is extremely useful. See our mailer block if you want to see this in action_.


## Binding To a Model In Our Route

One of the nice things about working with ASP.NET is that it will do a lot for you, under the hood. Here, we'll handle a `POST` from a form and bind the values directly to a `User` instance:

```csharp
using Tailwind.Data.Models;
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/hello", () => "Hello World!");
app.MapGet("/users", () => new { Email="test@tailwindtraders.dev", Name="Rob" });

app.MapPost("/users", (User user) => {user.Name, user.Email});

app.Run();

```

ASP.NET will do it's best to work with posted JSON data, binding it only to the values you specify. Correspondingly, we can also send objects directly back from our routes and ASP.NET will serialize them for us.

