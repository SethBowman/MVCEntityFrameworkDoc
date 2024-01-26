## Using Entity Framework with an MVC Application

### Prerequisites:

Before getting started with Entity Framework, make sure you have the following installed:

1. **Microsoft SQL Server:**

   - Install the free Developer Edition from [Microsoft SQL Server Downloads](https://www.microsoft.com/en-us/sql-server/sql-server-downloads).

2. **SQL Server Management Studio (SSMS):**
   - Download and install SSMS from [SQL Server Management Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms).
   - Open SSMS and connect to the server by typing "localhost" as the server.

### Step-by-Step Guide to Building an ASP.NET Core MVC Project:

#### Step 1: Create a new ASP.NET Core MVC Project

Create a new ASP.NET Core MVC project using Visual Studio.

#### Step 2: Install Required NuGet Packages

Install the necessary NuGet packages via Package Manager Console:

```bash
Install-Package Microsoft.EntityFrameworkCore.Design
Install-Package Microsoft.EntityFrameworkCore.Tools
Install-Package Microsoft.EntityFrameworkCore.SqlServer
Install-Package Microsoft.Extensions.Configuration.Json
```

#### Step 3: Create `appsettings.json`

Add the `appsettings.json` file to the project root and include the provided connection string:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=localhost;Initial Catalog=Testdb;Integrated Security=True;Encrypt=False"
  }
}
```

#### Step 4: Configure Database in `Program.cs`

Open `Program.cs` and configure services for MVC and Entity Framework. Set up the database connection:

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;

// Create a new web application builder
var builder = WebApplication.CreateBuilder(args);

// Add MVC services to the container
builder.Services.AddControllersWithViews();

// Load configuration from appsettings.json
var config = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json")
    .Build();

// Configure DbContext to use SQL Server with connection string from appsettings.json
builder.Services.AddDbContext<Testdb>(options =>
    options.UseSqlServer(config.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// Configure the HTTP request pipeline
if (!app.Environment.IsDevelopment())
{
    // Use exception handling for non-development environments
    app.UseExceptionHandler("/Home/Error");

    // Enable HTTP Strict Transport Security (HSTS)
    app.UseHsts();
}

// Redirect HTTP to HTTPS
app.UseHttpsRedirection();

// Serve static files
app.UseStaticFiles();

// Enable routing
app.UseRouting();

// Enable authorization
app.UseAuthorization();

// Map default controller route
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// Run the application
app.Run();
```

#### Step 5: Create the `Testdb` Class

Add a new class named `Testdb.cs`:

```csharp
using EntityFrameworkMVC.Models;
using Microsoft.EntityFrameworkCore;

// DbContext for the application
namespace EntityFrameworkMVC.Data
{
    public class Testdb : DbContext
    {
        // Constructor to receive DbContext options
        public Testdb(DbContextOptions<Testdb> options) : base(options)
        {
        }

        // DbSet for the User model
        public DbSet<User> Users { get; set; }

        // Configure database connection in the absence of options
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            var config = new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile("appsettings.json")
                .Build();

            optionsBuilder.UseSqlServer(config.GetConnectionString("DefaultConnection"));
        }
    }
}
```

#### Step 6: Create the `User` Model

Add a new class named `User.cs`:

```csharp
namespace EntityFrameworkMVC.Models
{
    public class User
    {
        // User properties
        public int Id { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
    }
}
```

#### Step 7: Create the `HomeController` Class

Add a new controller named `HomeController.cs`:

```csharp
using EntityFrameworkMVC.Data;
using EntityFrameworkMVC.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using System.Diagnostics;

namespace EntityFrameworkMVC.Controllers
{
    public class HomeController : Controller
    {
        private readonly Testdb _data;

        // Constructor to inject Testdb context
        public HomeController(Testdb data)
        {
            _data = data;
        }

        // Action to display a list of users
        public IActionResult Index()
        {
            // Retrieve all users from the database
            var users = _data.Users.ToList();

            // Pass the list of users to the view
            return View(users);
        }

        // ... (Other actions like Privacy, Error)
    }
}
```

#### Step 8: Create the `Views/Home/Index.cshtml` View

Create a new view file named `Index.cshtml` in the `Views/Home` folder and include the provided HTML code:

```html
@model List<User>
  <!-- Display a header -->
  <h2>Users</h2>

  <!-- Create a table to display users -->
  <table class="table">
    <tr>
      <th>ID</th>
      <th>First Name</th>
      <th>Last Name</th>
    </tr>

    <!-- Loop through each user in the model and display their information in a row -->
    @foreach (var user in @Model) {
    <tr>
      <td>@user.Id</td>
      <td>@user.FirstName</td>
      <td>@user.LastName</td>
    </tr>
    }
  </table></User
>
```

#### Step 9: Run the Application

Build and run the application to see if it works. Navigate to the `Index` page to view the list of users.

#### Step 10: Code-First Approach with Migrations

##### Step 10.1: Install Entity Framework Tools

Ensure that you have the Entity Framework tools installed. You can install them using the following command in the Package Manager Console:

```bash
Install-Package Microsoft.EntityFrameworkCore.Tools
```

##### Step 10.2: Create Initial Migration

Run the following command in the Package Manager Console to create an initial migration:

```bash
Add-Migration InitialCreate
```

This command generates a migration file in the "Migrations" folder with code to create the initial database schema based on your model.

##### Step 10.3: Apply the Migration

Run the following command to apply the migration and create the database:

```bash
Update-Database
```

This command executes the code in the migration file and creates the database with the specified schema.

##### Step 10.4: Verify in SQL Server Management Studio (SSMS)

Open SSMS, connect to the server, and verify that the database (`Testdb` in this case) has been created with the expected tables.

By following these steps, you've taken a code-first approach using Entity Framework migrations to generate and apply the database schema based on your model classes. This approach allows you to evolve the database schema as your application evolves, making it a powerful and flexible method.
