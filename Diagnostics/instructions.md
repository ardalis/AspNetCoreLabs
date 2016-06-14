
# Adding Diagnostics

## Add Diagnostics Pages to Your App

### Developer Exception Page

1. Start with the "Hello World" application you already created, or just create a new empty ASP.NET Core application
1. Open `project.json`
1. Ensure you have added "Microsoft.AspNetCore.Diagnostics" to "dependencies"
1. Open `Startup.cs`
1. Modify your middleware that returns "Hello World" so that it throws an exception.
1. Test the app - note that you don't get much information from the exception.
1. In ``Configure`` add the DeveloperExceptionPage if the app is in ``Development`` environment
  
    ``` C#
public void Configure(IApplicationBuilder app)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
}
    ```

1. Run the app again and test the page. You should see a detailed error page. Review the information it provides.

### Status Codes
  
1. Now modify your middleware so it only handles some requests:

    ```c#
    // in Configure, replace app.Run with this:
    app.Map("/hello", HelloWorld);

    // create new method in Startup
    private void HelloWorld(IApplicationBuilder applicationBuilder)
    {
        applicationBuilder.Run(async (context) =>
        {
            await context.Response.WriteAsync($"Hello World!");
        });
    }
    ```

1. Test that navigating to /hello still gives the expected response.

1. Navigate to any other path; you should get a blank page and a 404 if you check the status in Developer Tools or Fiddler

1. In Configure, add

    ```c#
    app.UseStatusCodePages();
    ```

1. Navigate to a path that gave a 404. It should now show you the 404 in the browser (as text).

### Welcome Page

You can add a welcome page to your app, which can be useful to test that it's working when you're first setting it up. To do so, just add to ``Configure``:

```c#
app.UseWelcomePage();
```

Once the app is created, you typically won't want this page to appear in most cases. In that case, if you still want it available, you can specify a particular path to which it should respond:

```c#
app.UseWelcomePage("/welcome");
```

1. Modify your app to include support for the welcome page on a particular path.

1. Test that the welcome page is displayed properly for that path.
