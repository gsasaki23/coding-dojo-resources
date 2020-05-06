## Store complex objects in ASP.NET Core Session
- The built-in session only stores strings and integers. 
- We can, however, make customizations to allow for storing other data types and complex objects, through serializing the object to JSON and storing it as a string! 

- Add the following anywhere A) within the namespace of the controller and B) outside of other classes
```
using Newtonsoft.Json;
public static class SessionExtensions
{
    public static void SetObjectAsJson(this ISession session, string key, object value)
    {
        session.SetString(key, JsonConvert.SerializeObject(value));
    }
       
    public static T GetObjectFromJson<T>(this ISession session, string key)
    {
        string value = session.GetString(key);
        return value == null ? default(T) : JsonConvert.DeserializeObject<T>(value);
    }
}
```

- Note that T is a generic type, standing in for whatever type we would use on retrieval.

- For complex objects, make sure that the Model has a parameter-less constructor! If you didn't intend to have any parameters in the constructor's signature, add `bool New = true` to the signature, and include `NewGame:true` every time you call the constructor. The JSON de-serizliation calls the constructor, which may give you unwanted results in many situations unless taken care of properly.

- To store: `HttpContext.Session.SetObjectAsJson("objectInSession",object)`
- To retrieve: `HttpContext.Session.GetObjectFromJson<Object>("objectInSession")`

---
## Access Session from Views
- add `@using Microsoft.AspNetCore.Http` in `Views/_ViewImports.cshtml`
- add `services.AddHttpContextAccessor();` in `ConfigureServices` in `Startup.cs`
- Access in a view: `<p>@Context.Session.GetString("UserFullName")</p>`

---
## Access Session from `Views/Shared/_Layout.cshtml`
- add `@using Microsoft.AspNetCore.Http` in `Views/_ViewImports.cshtml`
- Access example:
    ```
    @{
        int? userId = Context.Session.GetInt32("UserId");
    }
    ```
