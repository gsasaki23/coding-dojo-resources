# EF Setup Cheat Sheet

## 1. Create New Project & Install Dependencies

1. open terminal
2. `dotnet new mvc --no-https -o ProjName`
3. `cd ProjName`
4. `dotnet add package Pomelo.EntityFrameworkCore.MySql -v 2.2.0`
   - this will add an installed package line to your `.csproj` file
   - [Learn platform version number reference](http://learn.codingdojo.com/m/25/5675/40112)
5. `code .`

---

## 2. `appsettings.json` (add your own db name and change password if needed)

- add `"DBInfo"` property, don't forget the `,` that is needed between each property

  - ```json
      "DBInfo": {
        "Name": "MySqlConnection",
        "ConnectionString": "server=localhost;userid=root;password=root;port=3306;database=YOUR_DB_NAME;SslMode=None"
      }
    ```

---

## 3. Create basic `User` model in `Models` folder

- right click in file pane -> New C# Class
  - this should create the `namespace` line for you with your own project name
- you can add the rest of the properties to this later or delete it if you don't need it

- ```csharp
  namespace ProjName.Models
  {
      public class User
      {
        [Key] // the below prop is the primary key, [Key] is not needed if named with pattern: ModelNameId
        public int UserId { get; set; }

        [Required(ErrorMessage = "is required")]
        [MinLength(2, ErrorMessage = "must be at least 2 characters")]
        [Display(Name = "First Name")]
        public string FirstName { get; set; }
      }
  }
  ```

## 4. Create `DbContext` model

- **_only add `DbSet` for your models_**

- ```csharp
  using Microsoft.EntityFrameworkCore;

  namespace ProjName.Models
  {
    public class ProjNameContext : DbContext
    {
      public ProjNameContext(DbContextOptions options) : base(options) { }

      // for every model / entity that is going to be part of the db
      // the names of these properties will be the names of the tables in the db
      public DbSet<User> Users { get; set; }
      public DbSet<Widget> Widgets { get; set; }
      public DbSet<Item> Items { get; set; }
    }
  }
  ```

---

## 5. [Delete the following lines from `Startup.cs`](http://learn.codingdojo.com/m/25/5671/39759)

- ```csharp
    services.Configure<CookiePolicyOptions>(options =>
    {
      // This lambda determines whether user consent for non-essential cookies is needed for a given request.
      options.CheckConsentNeeded = context => true;
      options.MinimumSameSitePolicy = SameSiteMode.None;
    });
  ```

- `app.UseHttpsRedirection();`
- `app.UseCookiePolicy();`

- delete `<partial name="_CookieConsentPartial"></partial>` from `_Layout.cs`

---

## 6. `ConfigureServices` method in `Startup.cs`

- add the below lines using your own context name instead of `ProjNameContext`

- ```csharp
  services.AddDbContext<ProjNameContext>(options => options.UseMySql(Configuration["DBInfo:ConnectionString"]));
  services.AddSession();
  services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
  ```

---

## 7. `Configure` method

- add below lines above `app.UseMvc`

- ```csharp
  app.UseStaticFiles();
  app.UseSession();
  ```

---

## 8. Add Controller Constructor to Receive DbContext (IN EVERY CONTROLLER YOU MAKE)

- inside your controller class, at the top

- ```csharp
  private ForumContext db;
  public HomeController(ForumContext context)
  {
    db = context;
  }
  ```

---

## Create DB (Migrate)

---

### 9. `dotnet ef migrations add GiveANameToThisMigration`

---

### 10. `dotnet ef database update`

- **EVERY time you migrate you need to update for the database to update**
- **EVERY time you change your models you must migrate and then update database**

---

### 11. Verify DB & Tables in workbench or mysql Shell

- you should see a `users` table with columns: `UserId` and `FirstName`

---

## 12. Access Session from Views Directly

- this is helpful if you are repeatedly adding the same thing from session into the `ViewBag` for many actions
- add `@using Microsoft.AspNetCore.Http` in `Views/_ViewImports.cshtml`
- add `services.AddHttpContextAccessor();` in `ConfigureServices` in `Startup.cs`
- Access in a view: `<p>@Context.Session.GetString("UserFullName")</p>`

---

## 13. Create Other Models and [Relationships](https://github.com/TheCodingDojo/student_md_docs/blob/master/CA-OC/csharp/relationships.md)

- set up one relationship at a time
  - then migrate and update database again
    - this way, if the migration fails, you know that which relationship you ruined

---

## Request Response Cycle Review (if you can't explain this effortlessly when asked to, keep re-reading it)

1. the **_client_** makes a **_request_** by visiting a URL
   - a URL is usually visited by the **_client_** clicking on an anchor tag (GET request), typing a URL into the address bar (GET), submitting a form (POST)
2. the **_server_** receives the **_request_** and runs the method (action) that has the matching URL
3. the **_server_** **_responds_** to the request by **_serving_** up an HTML page (rendering), or redirecting to another action method which has it's own URL
4. the request response cycle is complete
   - a redirect triggers an additional request response cycle, the **_response_** is a redirect which triggers a **_request_** for the URL that the client was redirected to

---

## Routing Review

---

### Route Parameters

- read the Request Response Cycle Review
- a route parameter is a placeholder in a URL that is given a name when the route is set up in the controller
  - just like a parameter in a function, it is a placeholder name that will be given a value later on (when function is called / route is visited)
  - the controller method (action) needs a corresponding parameter with a matching name for every parameter in the URL
  - when redirecting to a controller method (action), if that action has parameters, they need to be provided when redirecting

---

### Where are the route parameter placeholders given an actual value

- this happens either in the HTML files or when redirecting in the controller
  - `<a>` tags and `<form>` tags need to have actual parameter values added to them using razor
    - the values are inserted either directly into the `<a>` tag's `href` and the `<form>` tag's `action` attributes or using ASP.NET tag helpers

---

#### Route Parameter Example

- ```html
  <a href="/Quotes/@Model.QuoteId">
    View Quote
  </a>
  ```

- OR

  - ```html
    <a
      asp-controller="Quotes"
      asp-action="Details"
      asp-route-quoteId="@Model.QuoteId"
    >
      View Quote
    </a>
    ```

- the above `<a>` tags would create an `href` that looks like this with some id value inserted: `href="/Quotes/5"`
- these URLS would match / connect to a URL pattern in the `QuotesController`

  - ```csharp
    // the parameter in the method has to have the same name as the route parameter in the URL
    // and the same name as the asp-route-paramName tag helper if using that
    [HttpGet("Quotes/{quoteId}")]
    public IActionResult Details(int quoteId)
    ```

---

### Routing with a `<form>`

- a form needs a way for the **_client_** to **_request_** it to be displayed (rendered), therefore a URL / route is needed for displaying / rendering the HTML page that has the `<form>` in it
  - GET **_request_** to GET the page that displays the `<form>`
  - there will be a controller method (action) with matching URL to handle the **_request_** that will **_respond_** with `return View()` to display the HTML page with the `<form>` in it
- the form needs a DIFFERENT URL / route to submit to so the submission can be processed, this URL is specified in the `<form>` `action` attribute or the ASP.NET tag helpers on the `<form>` tag
  - the URL / route that the `<form>` submits to (POST **_request_**) needs a corresponding controller method (action) with a matching URL
    - this method (action) will **_respond_** with either a `return RedirectToAction` if successful or a `return View("NameOfView")` if there was a validation error to display the `<form>` again with error messages

---

#### `<form>` routing example

- handle the _request_ for the `<form>` to be displayed

  - ```csharp
    [HttpGet("/Quotes/New")]
    public IActionResult New()
    {
      return View();
    }
    ```

- ```html
  <form asp-controller="Quotes" asp-action="Create" method="POST"></form>
  ```

- OR

  - ```html
    <form action="/Quotes/Create" method="POST"></form>
    ```

- form submits to the route that matches the `action` attribute or the `asp-controller` and `asp-action`, MAY NEED TO ADD A ROUTE PARAMETER TO THE FORM if the controller method (action) that the `<form>` submits to needs more information

- ```csharp
  [HttpPost("/Quotes/Create")]
  public IActionResult Create(Quote newQuote)
  {
    if (ModelState.IsValid == false)
    {
        // to display validation error messages
        return View("New");
    }

    // no validation errors: save to database then redirect
    // if action redirecting to has parameters, must provide them here as well
    return RedirectToAction("NameOfControllerMethod");
  }
  ```

---

## ViewModel Review

- if you have a view that needs to be sent more than one model, create a new model specifically for that view which contains all of the models it needs as properties
- OR use one of the models as the View Model and put the other(s) into `ViewBag`

---

### Display AND create on same page

- if you have a view whose model (for displaying) is `EntityA` (or a list of `EntityA`)
- and the same view is used to create `EntityB`
- your problem is that this one view has two view models that it needs
- this can be solved multiple ways
  1. you can send `EntityA` to be displayed, then render a partial for creating `EntityB`, the partial would `EntityB` as it's view model, but the partial would still need to be passed `EntityB`
  2. you can create a new view model for this page, a combined model that has a prop for both of the models that the page needs
     - if returning the view to display error messages, this combined model would need to be returned in the same way it was for rendering the page in the first place

#### Examples

##### 1. partial example

- ```csharp
  @model EntityA

  <div>
    @* display EntityA which is in the Model *@
  </div>

  @{
    EntityB newB = new EntityB();
  }

  <partial name="_newModelB" model="@newB"></partial>
  ```

  - if `newB` was related to `EntityA` where `EntityB` is the many, so it needs a foreign key corresponding to the primary key of `EntityA`, that foreign key could be set in the razor code block before being passed to the partial
    - `newB.EntityAId = Model.EntityAId;`

---

## Validation review

- when you `return View` to display error validations, if this page was originally passed a model, then you must pass that same model (data) to it again

---

## [Troubleshooting](https://github.com/TheCodingDojo/student_md_docs/blob/master/CA-OC/csharp/troubleshooting.md)

---

## [Relationship Advice](https://github.com/TheCodingDojo/student_md_docs/blob/master/CA-OC/csharp/relationships.md)

---

## Old stuff

- `dotnet add package MySql.Data.EntityFrameworkCore -v 8.0.11`
