# InstantAPIs

[![Nuget](https://img.shields.io/nuget/v/Fritz.InstantAPIs)](https://www.nuget.org/packages/Fritz.InstantAPIs/)
[![Instant APIs Documentation](https://img.shields.io/badge/docs-ready!-blue)](https://csharpfritz.github.io/InstantAPIs)
![GitHub last commit](https://img.shields.io/github/last-commit/csharpfritz/InstantAPIs)
![GitHub contributors](https://img.shields.io/github/contributors/csharpfritz/InstantAPIs)

This article contains two different ways to get an instant API:

- An API based on a `DbContext`, it will generate the routes it needs given a database class. There are two implementations of this: Reflection and a source generator.
- An API based on a JSON file.

## DbContext based API

A proof-of-concept library that generates Minimal API endpoints for an Entity Framework context. Right now, there are two implementations of this: one that uses Reflection (this is the one currently in the NuGet package), the other uses a source generator. Let's see how both of them work.

For a given Entity Framework context, `MyContext`

```csharp
public class MyContext : DbContext 
{
    public MyContext(DbContextOptions<MyContext> options) : base(options) {}

    public DbSet<Contact> Contacts => Set<Contact>();
    public DbSet<Address> Addresses => Set<Address>();

}
```

We can generate all of the standard CRUD API endpoints with the Reflection approach using this syntax in `Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddSqlite<MyContext>("Data Source=contacts.db");

var app = builder.Build();

app.MapInstantAPIs<MyContext>();

app.Run();
```

Now we can navigate to `/api/Contacts` and see all of the Contacts in the database.  We can filter for a specific Contact by navigating to `/api/Contacts/1` to get just the first contact returned.  We can also post to `/api/Contacts` and add a new Contact to the database. Since there are multiple `DbSet`, you can make the same calls to `/api/Addresses`.

You can also customize the APIs if you want
```csharp
app.MapInstantAPIs<MyContext>(config =>
{
	config.IncludeTable(db => db.Contacts, ApiMethodsToGenerate.All, "addressBook");
});
```

This specifies that the all of the CRUD methods should be created for the `Contacts` table, and it prepends the routes with `addressBook`.

The source generator approach has an example in the `WorkingApi.Generators` project (at the moment a NuGet package hasn't been created for this implementation). You specify which `DbContext` classes you want to map with the `InstantAPIsForDbContextAttribute`, For each context, an extension method named `Map{DbContextName}ToAPIs` is created. The end result is similar to the Reflection approach:

```csharp
[assembly: InstantAPIsForDbContext(typeof(MyContext))]

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddSqlite<MyContext>("Data Source=contacts.db");

var app = builder.Build();

app.MapMyContextToAPIs();

app.Run();
```

You can also do customization as well
```csharp
app.MapMyContextToAPIs(options =>
	options.Include(MyContext.Contacts, "addressBook", ApisToGenerate.Get));
```

Feel free to try both approaches and let Fritz know if you have any issues with either one of them. The intent is to keep feature parity between the two for the forseable future.

### Demo

Check out Fritz giving a demo, showing the advantage of InstantAPIs on YouTube: https://youtu.be/vCSWXAOEpBo



## A JSON based API

An API will be generated based on JSON file, for now it needs to be named *mock.json*.

A typical content in *mock.json* looks like so:

```json
{
  "products" : [{
    "id": 1,
    "name": "pizza"
  }, {
    "id": 2,
    "name": "pineapple pizza"
  }, {
    "id": 3,
    "name": "meat pizza"
  }],
  "customers" : [{
    "id": 1,
    "name": "customer1"
  }]
  
}
```

The above JSON will create the following routes:

|HTTP Verb  |Endpoint  |
|---------|---------|
|  GET   | /products        |
|  GET   | /products/{id}        |
|  POST   | /products        |
|  DELETE   | /products/{id}        |
|  GET   | /customers        |
|  GET   | /customers/{id}        |
|  POST   | /customers        |
|  DELETE   | /customers/{id}        |

### Demo

To use this, do the following:

1. Create a new minimal API, if you don't already have one:

   ```bash
   dotnet new web -o DemoApi -f net6.0
   cd DemoApi 
   ```

1. Add the NuGet package for [Fritz.InstantAPIs](https://www.nuget.org/packages/Fritz.InstantAPIs/):

   ```bash
   dotnet add package Fritz.InstantAPIs --prerelease
   ```

1. In *Program.cs*, add the following namespace:

   ```csharp
   using Mock;
   ```

1. Create a file *mock.json* and give it for example the following content:

   ```json
   {
      "products" : [{
        "id": 1,
        "name": "pizza"
      }]
    }
   ```

1. Now add the following code for the routes to be created:

   ```csharp
   app.UseJsonRoutes();
   ```

Here's an example program:

```csharp
using Mock;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello World!");
app.UseJsonRoutes();
app.Run();
```

### Coming features

Support for:

- [query parameters](https://github.com/csharpfritz/InstantAPIs/issues/40)
- [PUT](https://github.com/csharpfritz/InstantAPIs/issues/39)

## Community

This project is covered by a [code of conduct](https://github.com/csharpfritz/InstantAPIs/blob/main/CODE-OF-CONDUCT.md) that all contributors must abide by.  [Contributions are welcome and encouraged.](https://github.com/csharpfritz/InstantAPIs/blob/main/CONTRIBUTING.md).
