# Data Access With Dapper

[Dapper](https://github.com/DapperLib/Dapper) is a small, targeted data access library that keeps the concepts simple and focuses on speed. You can write SQL by hand and let Dapper do the rest, or you can install extensions on top of Dapper that do rudimentary mapping and more. Created by the StackOverflow team, Dapper is loved by thousands of developers.

Installing Dapper is straightforward, however you need to be sure you also add the database library you're working with:

```sh
dotnet add package Dapper
dotnet add package Npgsql
```

Dapper is a thin layer of abstraction that extends `IDbConnection`. This code is from the documentation on GitHub:

```csharp
public class Dog
{
    public int? Age { get; set; }
    public Guid Id { get; set; }
    public string Name { get; set; }
    public float? Weight { get; set; }

    public int IgnoredProperty { get { return 1; } }
}

var guid = Guid.NewGuid();
var dog = connection.Query<Dog>("select Age = @Age, Id = @Id", new { Age = (int?)null, Id = guid });

Assert.Equal(1,dog.Count());
Assert.Null(dog.First().Age);
Assert.Equal(guid, dog.First().Id);
```

As you can see, mapping is done in the simplest possible way by matching on property names. Parameters are passed using an anonymous object and you don't need data attributes, as with Entity Framework, to facilitate mapping.

Inserting and updating data works the same way, again from the documentation:

```csharp
var count = connection.Execute(@"
  set nocount on
  create table #t(i int)
  set nocount off
  insert #t
  select @a a union all select @b
  set nocount on
  drop table #t", new {a=1, b=2 });

Assert.Equal(2, count);
```

The `Execute` command is able to run multiple statements and will return rows affected for updates and deletes or the new ID for inserts.

To learn more about Dapper, [consult the documentation](https://github.com/DapperLib/Dapper). Here's how we use it with ASP.NET Minimal API.

## Using Dapper Directly

Working with Dapper means creating a connection to your database and then extending it. Because Dapper extends `IDbConnection`, you're free to use whatever that interface supports.

*Note: drivers and database platforms behave differently. Accessing SQLite using `Microsoft.Data.Sqlite`, for instance, will have different methods than working with PostgreSQL and `Npgsql`. You should be aware of the differences when working with Dapper, and some forms of SQL is supported while others are not. PostgreSQL is case sensitive, MySQL and SQLite are not, for example.*

At Tailwind Traders, we prefer explicit naming when it comes to accessing our database. Inspecting the connection string for a platform name is a bit too magical for us, so we prefer to use a factory:

```csharp
namespace Tailwind.Data;

public class DB{
  private DB(){}
  public static Postgres(){
    //get the connection string from the environment
    var connectionString = Environment.GetEnvironmentVariable("DATABASE_URL");
    if(String.IsNullOrEmpty(connectionString)){
      throw new InvalidProgramException("No DATABASE_URL found in environment");
    }
    var conn = new NpgsqlConnection(connectionString);
    conn.Open();
    return conn;
  }
}
```

We can now access this throughout our codebase:

```csharp
//using Dapper
app.MapPost("/signup", async ([FromBody] SignUpRequest req) => {
  var contact = new Contact{
    Email = req.Email,
    Name = req.Name
  };
  //open a connection directly
  using var conn = DB.Postgres();
  var result = await conn.ExecuteAsync("insert into contacts (email, name) values (@Email, @Name)", contact);
  return result;
});
```

If your codebase is small enough and you enjoy clarity, this approach can work well for you. You might be wondering if a method name such as `DB.Open()` might be more flexible, and indeed it is, however your system might have multiple databases you need to access; so name your connection as you need.

## Using Dependency Injection

It is far more common in ASP.NET to use services and inject them as you need. ASP.NET Minimal API allows for this using a simple key name strategy.

The first thing we need to do is to create an interface for our connection class and then implement it:

```csharp
public interface IDb{
  IDbConnection Connect();
}
public class DB: IDb
{
  public IDbConnection Connect()
  {
    //get the connection string from the environment
    var connectionString = Environment.GetEnvironmentVariable("DATABASE_URL");
    if(String.IsNullOrEmpty(connectionString)){
      throw new InvalidProgramException("No DATABASE_URL found in environment");
    }
    var conn = new NpgsqlConnection(connectionString);
    conn.Open();
    return conn;
  }
}
```

Next, we wire up the service using the `builder`:

```csharp
var builder = WebApplication.CreateBuilder(args);
//...
builder.Services.AddScoped<IDb, DB>();
```

Here, I'm adding a scoped instance of our connection service as I want just one instance per request. Now I can open the connection using the injected service:

```csharp
//using Dapper
app.MapPost("/signup", async ([FromBody] SignUpRequest req, [FromServices] IDb db) => {
  var contact = new Contact{
    Email = req.Email,
    Name = req.Name
  };
  using var conn = db.Connect();
  var result = await conn.ExecuteAsync("insert into contacts (email, name) values (@Email, @Name)", contact);
  return result;
});
```

How you go about accessing the `IDbConnection` in your code is up to you, which is one of the nice things about working with ASP.NET Minimal API!

## Using Effective Patterns For More Complex Operations

You can use Dapper with any data access pattern, including none at all, which is what we've been doing so far. If you have multiple commands you need to execute, however, it's typically preferable to use a pattern such as the Repository or Command/Query Separation. Here at Tailwind, we prefer the latter for many of the reasons [outlined here](https://lostechies.com/jimmybogard/2012/10/08/favor-query-objects-over-repositories/).

That's just like, our opinion, so use whatever works for you.

Here is one pattern we quite like, when working with multiple SQL commands in a transaction. We start out with an explicit `CommandResult`:

```csharp
namespace Tailwind.Data;
public class CommandResult{
  public bool Success { get; set; } = false;
  public string Message { get; set; } = "The command didn't run";
  public int Inserted { get; set; } = 0;
  public int Updated { get; set; } = 0;
  public int Deleted { get; set; } = 0;
}
```

Next, we declare a class that exists just to run our command:

```csharp
using Tailwind.Data;
using Dapper;
public class RegisterCommandResult: CommandResult{
  public User User { get; set; }
}
public class RegisterNewUser{
  public string Email { get; set; }
  public string? Name { get; set; }
  public RegisterNewUser(string email, string? name){
    Email = email;
    Name = name;
  }
  public CommandResult Execute(IDb db){
    using var conn = db.Connect();
    var tx = conn.BeginTransaction();
    try{
      //validate their info
      //do they exist?
      //add them
      var user = conn.Insert(...)
      tx.Commit();
      return new RegisterCommandResult{
        Success: true,
        Message: "User Registered",
        Inserted: 1,
        User: user
      }
    }catch(InvalidDataException ex){ //catch explicit exceptions here
      tx.Rollback();
      //rethrow or return
      return new RegisterCommandResult{
        Message: ex.Message,
        Inserted: 0
      }
    }
  }
}
```

Here, everything is nicely wrapped in a transaction and the requirements are clear: we need an `Email` that's valid and optionally a name.

## Using Dapper Extensions

There are multiple libraries out there that you can use to extend Dapper. One of our favorites, here at Tailwind, is [SimpleCRUD](https://github.com/ericdc1/Dapper.SimpleCRUD), which allows you to insert a record very simply:

```csharp
//using Dapper;
var user = new User{Email="test@test.com", Name="Jill Test"};
var conn = DB.Postgres();
conn.Insert(user); //conn.Insert<T>()
```

SimpleCRUD will match on property names for the insert statement, which means you don't have to write out all the SQL. We like SQL here at Tailwind, but it is very helpful to have a library do it for you.

One other nice thing about SimpleCRUD is that it allows you to override its naming and mapping, using custom resolvers. This is useful when working with a database such as Postgres, which has strict naming conventions (which you can override at your own peril) and is also case-sensitive.

Here's a resolver that we like to use:

```csharp

public class CustomResolver : SimpleCRUD.IColumnNameResolver
{
  public string ResolveColumnName(PropertyInfo propertyInfo)
  {
    return propertyInfo.Name.ToSnakeCase();
  }
}
```

You can find `ToSnakeCase()` extension methods online, [including this gist](https://gist.github.com/vkobel/d7302c0076c64c95ef4b).


## Creating Your Own Extensions

One of the wonderful thing about Dapper is that it's just a collection of extension methods on top of `IDbConnection`. You can add to it, if you like, by creating your own extensions:

```csharp
namespace Dapper;
public static class TailwindDapperExtensions{
  public static int Upsert<T>(T item) where T: new(){
    //see if T has an ID
    //if so, see if it exists
    //or use ON CONFLICT with Postgres
  }
}
```

You can be as creative, expressive, or direct as you like, with full control over your data access. That's the point of using Dapper!