# Setup Cheat Sheet for ASP.NET Core 2.2 + Entity Framework
**_NOTE: Double check and change anything with a {{ }}!_**
- [Original Sheet from Coding Dojo](https://github.com/TheCodingDojo/student_md_docs/blob/master/CA-OC/csharp/efSetupCheatSheet.md)

## 1. Create New Project & Install Dependencies
1. open terminal to desired project folder location
2. `dotnet new mvc --no-https -o {{ProjName}}`
3. `cd {{ProjName}}`
4. `dotnet add package Pomelo.EntityFrameworkCore.MySql -v 2.2.0`
   - package will be installed and added to `.csproj` file in project
5. `code .`
---
## 2. Edit `appsettings.json`
- add `"DBInfo"` property
  ```json
   "DBInfo": {
     "Name": "MySqlConnection",
     "ConnectionString": "server=localhost;userid={{root}};password={{root}};port=3306;database={{YOUR_DB_NAME}};SslMode=None"
   },
  ```
---
## 3. Create `DbContext` file inside `Models` folder
- **_only add `DbSet` for models you want in MySQL database_**
  ```csharp
  using Microsoft.EntityFrameworkCore;

  namespace {{ProjName}}.Models
  {
    public class {{ProjNameContext}} : DbContext
    {
      public {{ProjNameContext}}(DbContextOptions options) : base(options) { }

      // add every model you want in the DB
      public DbSet<User> Users { get; set; }
      public DbSet<Sample> Samples { get; set; }
    }
  }
  ```
---
## 4. Create necessary models in `Models` folder
- Remember to include `namespace` line in every file!
  ```csharp
  namespace {{ProjName}}.Models
  {
      public class User
      {
        [Key]
        public int UserId { get; set; }

        [Required(ErrorMessage = "is required")]
        [MinLength(2, ErrorMessage = "must be at least 2 characters")]
        [Display(Name = "Your Name: ")]
        public string FullName { get; set; }
        
        [Required(ErrorMessage = "Required")]
        [EmailAddress]
        [Display(Name = "Email:")]
        public string Email { get; set; }

        [Required(ErrorMessage = "Required")]
        [MinLength(8, ErrorMessage = "must be at least 8 characters")]
        [DataType(DataType.Password)] // Changes form input's type to password
        [Display(Name = "Password:")]
        public string Password { get; set; }

        [NotMapped] // NOT ADDING TO DB
        [DataType(DataType.Password)]
        [Compare("Password", ErrorMessage = "Must match password")]
        [Display(Name = "Confirm Password:")]
        public string PasswordConfirm { get; set; }

        public DateTime CreatedAt { get; set; } = DateTime.Now;
        public DateTime UpdatedAt { get; set; } = DateTime.Now;
        
        // any One-to-Many or Many-to-Many relationships and custom validators could go here
      }
  }
  ```
---
## 5. `Startup.cs`
- Inside `ConfigureServices` method, add:
  ```csharp
  services.AddDbContext<{{ProjNameContext}}>(options => options.UseMySql(Configuration["DBInfo:ConnectionString"]));
  services.AddSession();
  services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
  ``` 
  and use the lightbulb or manually add to the top of the file: `using {{namespace}}.Models;` and `using Microsoft.EntityFrameworkCore;`, replacing `namespace`

- Inside `ConfigureServices` method, remove or comment out:
  ```csharp
    services.Configure<CookiePolicyOptions>(options =>
    {
      // This lambda determines whether user consent for non-essential cookies is needed for a given request.
      options.CheckConsentNeeded = context => true;
      options.MinimumSameSitePolicy = SameSiteMode.None;
    });
  ```
- Inside `Configure` method, add ABOVE `app.UseMvc()`:
  ```csharp
  app.UseStaticFiles();
  app.UseSession();
  ```
- Inside `Configure` method, remove or comment out `app.UseHttpsRedirection();` and `app.UseCookiePolicy();`
---
## 7. `Views/Shared/_Layout.cshtml`
- Remove or comment out `<partial name="_CookieConsentPartial"></partial>` (Around line 56)
---
## 8. To use DbContext in any Controller File
- inside your controller class, at the top
  ```csharp
  private {{ProjContext}} db;
  public HomeController({{ProjContext}} context)
  {
    db = context;
  }
  ```
  which will allow other methods to make calls like `db.Users.Add()`
---

## Create DB (Migrate)
**You must migrate and update database EVERY time you change your models!**
---
### 9. `dotnet ef migrations add {{MigrationName}}`
### 10. `dotnet ef database update` 
### 11. Verify DB & Tables in workbench or mysql Shell
- you should see a table with model attributes as columns
---

## To get started: dotnet run
