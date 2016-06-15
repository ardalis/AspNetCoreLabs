# Introduction to Routing

## Install the routing package
1. Use the application you created in the first lab (or create a new Empty Web app)
1. Open the `project.json` file
1. In the `dependencies` section, add an entry for the "Microsoft.AspNetCore.Routing" package:

    ```JSON
"dependencies": {
    ...,
    "Microsoft.AspNetCore.Routing": "1.0.0-rc2-final",
}
    ```

1. Open the `Startup.cs` file

1. Add the routing services to the configuration in the `Startup.cs`:

    ```c#
public void ConfigureServices(IServiceCollection services)
{
    services.AddRouting();
}
    ```
This step will require ``using Microsoft.Extensions.DependencyInjection;``

1. In the `Configure` method, create a `RouteBuilder` with a handler for the root of the site and add it to the middleware pipeline:
  
    ```c#
public void Configure(IApplicationBuilder app)
{
    var routeBuilder = new RouteBuilder(app);

    routeBuilder.MapGet("", context => context.Response.WriteAsync("Hello from Routing!"));
        
    app.UseRouter(routeBuilder.Build());

    ... // any other middleware you have configured
}
    ```
This step will require ``using Microsoft.AspNetCore.Routing;``

1. Run the site and verify your middleware is hit via routing

1. Add another route that matches a sub-path (between MapGet and UseRouter):
  
    ``` c#
routeBuilder.MapGet("sub", context => context.Response.WriteAsync("Hello from sub!"));
    ```

1. Run the site and verify that your routes are hit when the matching URL is requested.

1. If you don't still have it, add a default middleware after your call to UseRouter:

    ```c#
app.Run(async context =>
{
    await context.Response.WriteAsync("Hello World");
});
    ```

1. Investigate what happens when you navigate to paths "/", "/sub", and "/foo".

## Capture and use route data

1. Add another route that captures the name of an item from the URL, e.g. "item/{itemName}", and displays it in the response:
  
    ``` c#
routeBuilder.MapGet("item/{itemName}", context => context.Response.WriteAsync($"Item: {context.GetRouteValue("itemName")}"));
    ```

1. Run the site and verify that your new route works

1. Modify the route to include a route constraint on the captured segment, enforcing it to be a number:
  
    ``` c#
routeBuilder.MapGet("item/{id:int}", context => context.Response.WriteAsync($"Item ID: {context.GetRouteValue("id")}"));
    ```

1. Run the site again and see that the route is only matched when the captured segment is a valid number. What happens when text is used?

1. Modify the router to include both versions of the route above (with and without the route constraint)

1. Experiment with changing the order the routes are added and observe what effect this has on which route is matched for a given URL

1. Experiment with other [route constraints](https://docs.asp.net/en/latest/fundamentals/routing.html#id5) and routes.
