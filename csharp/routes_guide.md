## Request Response Cycle Review

1. A **_client_** makes a **_request_** by visiting a URL
   - From clicking on an `<a>` anchor tag (GET request), typing the URL into the address bar (GET), submitting a form (POST)
2. The **_server_** receives the **_request_** and runs the method/action with the matching URL
3. The **_server_** responds by **_rendering/serving_** an HTML page OR redirects to another method

---
## Routing Review
---
- URLs can have route parameters, which is a placeholder for when controller methods need a specific value to run logic on
- These are provided either in the HTML `<a>` or `<form>` tags, or when redirecting from another controller method
---
## Routing with `<a>`
- For the controller method:
   ```csharp
   [HttpGet("Quotes/{quoteId}")]
   public IActionResult Details(int quoteId){}
   ```
- The view can have an `<a>` tag like this:   
   ```html
   <a asp-controller="Quotes" asp-action="Details" asp-route-quoteId="@Model.QuoteId">
      View Quote
   </a>
   ```
   which is converted into:
   ```html
   <a href="/Quotes/@Model.QuoteId">
      View Quote
   </a>
   ```
---

### Routing with a `<form>`
- Controller method to display the `<form>`:
  ```csharp
   [HttpGet("/Quotes/New")]
   public IActionResult New()
   {
      return View();
   }
   ```
- View has:
   ```html
   <form asp-controller="Quotes" asp-action="Create" method="POST" asp-route-newQuote="@Input"
   ```
   - `asp-route-XXX` must match the controller method parameter name
- Controller method to create:
  ```csharp
  [HttpPost("/Quotes/Create")]
  public IActionResult Create(Quote newQuote)
  {
    if (ModelState.IsValid == false)
    {
        // to display validation error messages
        return View("New");
    }
    return RedirectToAction("OtherControllerMethod");
  }
  ```

---

### Display AND create on same page

- Given a view that displays `EntityA` (or a list of `EntityA`), AND creating `EntityB`, the problem is that the view has 2 models coexisting. This can be solved multiple ways:
  1. Send `EntityA` to be displayed normally, then render a partial view with `EntityB` as its view model (need to pass `EntityB` into partial somehow)
  2. Create a new combined model for this view
      - For returning the view with error messages, this combined model would still need everything for rendering the page in the first place again

##### 1. Partial View Approach - View Sample
  ```csharp
  @model EntityA

  <div>
    @* display EntityA info *@
  </div>

  @{
    EntityB newB = new EntityB();
    newB.EntityAId = Model.EntityAId;
  }

  <partial name="_newModelB" model="@newB"></partial>
  ```
- If `EntityB` was a many to one `EntityA`, it would need a foreign key corresponding to the primary key of `EntityA`, that foreign key could be set in the razor code block before being passed to the partial

---
