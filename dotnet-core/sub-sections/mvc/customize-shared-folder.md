# Customizing Your Shared Views Folder

Lets assume that you are reaching a point in your MVC application whereby the shared folder really needs seperate sections. Well Razor provides a method used to customize this. This is quite simple:

**Step 1: Open the `Startup.cs`  file of your project**

**Step 2: Navigate to the `ConfigureServices` method**

**Step 3: Add the following**

```c#
 public void ConfigureServices(IServiceCollection services)
 {
     services.AddMvc()
                .AddRazorOptions(opt => {
                    opt.ViewLocationFormats.Add("/Views/{1}/Partials/{0}.cshtml");
                    opt.ViewLocationFormats.Add("/Views/Shared/Partials/{0}.cshtml");
                });
 } 
```

 

## Remarks 

The locations of the views returned from controllers that do not belong to an area. Locations are format strings (see [here](https://msdn.microsoft.com/en-us/library/txafckwd.aspx)) which may contain the following format items:

- {0} - Action Name
- {1} - Controller Name

The values for these locations are case-sensitive on case-sensitive file systems. For example, the view for the `Test` action of `HomeController` should be located at `/Views/Home/Test.cshtml`. Locations such as `/views/home/test.cshtml` would not be discovered.



---

## Change Log

- [16-10-2020] - Added Section Customizing Your Shared Views Folder