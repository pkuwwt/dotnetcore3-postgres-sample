
# A sample dotnetcore 3 webapi with PostgreSQL

## Preparation

### Install dotnetcore 3

For MacOs, download the following files.

  * [dotnet-sdk-3.0.100-osx-x64.pkg](https://download.visualstudio.microsoft.com/download/pr/5c281f95-91c4-499d-baa2-31fec919047a/38c6964d72438ac30032bce516b655d9/dotnet-sdk-3.0.100-osx-x64.pkg)
  * [dotnet-runtime-3.0.0-osx-x64.pkg](https://download.visualstudio.microsoft.com/download/pr/1b09851c-1c1a-4aeb-a94a-7065db8741c0/b22a0b5501191fe1a263913d8ed11b2e/dotnet-runtime-3.0.0-osx-x64.pkg)
  * [aspnetcore-runtime-3.0.0-osx-x64.tar.gz](https://download.visualstudio.microsoft.com/download/pr/3ab4125a-c616-4aec-8fdb-763039e99f1c/08a6d2546fbbd4b1b959e6a3da3b9eb4/aspnetcore-runtime-3.0.0-osx-x64.tar.gz)

Install the two pkgs (the default install location is `/usr/local/share/dotnet`), and uncompress the `aspnetcore-runtime-3.0.0-osx-x64.tar.gz` to the directory.

In a word, install all the files to the same directory. This is true for all OSs.

Configure the environmental variables:

```
export DOTNET_ROOT=/usr/local/share/dotnet
export PATH=$PATH:$DOTNET_ROOT:~/.dotnet/tools
```

`DOTNET_ROOT` is required by Linux. `~/.dotnet/tools` is the path for installed dotnet tools, i.e. `dotnet-ef`.

### Install dotnet-ef

`dotnet-ef` is bundled in dotnetcore 2, but not dotnetcore 3, so we should install it manually.

```
dotnet tool install --global dotnet-ef --version 3.0.0
```

### Install PostgreSql

Just download the installer and install.

Create a database called `todolist`. This is can be done by `pgAdmin4` or command line.

```
psql -h 127.0.0.1 -p 5432 -U postgres postgres -c "CREATE DATABASE todolist;"
```

## Create a project

### Init a project

```
dotnet new webapi
```

### Add dependencies

```
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL --version 3.0.1
dotnet add package NpgSql.EntityFrameworkCore.PostgreSQL.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
```

Then the `.csproj` file will add three items.

### Define models

A model is an ORM class mapped to the database table, just like `Models/TodoItem.cs`.

Each model has its corresponding context `Models/TodoContext.cs`, which is used as a bridge between database context and model.

### Define controllers

Controller is used to define routes, just like `Controllers/TodoControllers.cs`.

The database context is passed to controller, so that the controller can access the models.

### Register controllers

Revise `Startup.cs`:

```
// ...
using Microsoft.EntityFrameworkCore;
using TodoApi.Models;
namespace TodoApi3
{
    public class Startup
    {
		// ...
        public void ConfigureServices(IServiceCollection services)
        {
			// ...
            services.AddDbContext<TodoContext>(options =>
					options.UseNpgsql(Configuration.GetConnectionString("DefaultConnection")));
        }
		// ...
```

Then where is the `DefaultConnection`? It is obviously the connection string for database.

### Add connection string

Revise `appsettings.json`, which will looks like
```
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
	  "DefaultConnection": "Host=localhost;Port=5432;Username=postgres;Password=123456;Database=todolist;"
  }
}
```

Make sure the connection string is correct.

### Database initialization

For now, the database `todolist` is empty, so we should init the tables in it.

```
# create a Migrations directory with initialization code
dotnet ef migrations add initial
# create tables in the empty database
dotnet ef database update
```

### Run the application

```
dotnet run
```

If everything is OK, you can browse `https://127.0.0.1:5001/api/todo` now.

## Reference
  
  * [webapi with net core and postgres in visual studio code](https://medium.com/@agavatar/webapi-with-net-core-and-postgres-in-visual-studio-code-8b3587d12823) and its [code](https://github.com/laxmansahni/TodoApi/)

