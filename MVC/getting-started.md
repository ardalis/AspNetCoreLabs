# Getting Started with MVC

## Install the MVC package
1. Use the application you created in the first lab (or create a new Empty Web app)
1. Open the `project.json` file
1. In the `dependencies` section, add an entry for the "Microsoft.AspNetCore.Mvc": "1.0.0-rc2-final" package:

    ```JSON
"dependencies": {
    ...,
    "Microsoft.AspNetCore.Mvc": "1.0.0-rc2-final",
}
    ```

1. Add a "Controllers" folder to your app.

1. Create a new class in the Controllers folder called "HomeController":

    ```c#
public class HomeController
{
    [HttpGet("/")]
    public string Index() => "Hello from MVC!";
}    
    ```

1. Open the `Startup.cs` file

1. Add the MVC services to the configuration in the `Startup.cs`:

    ```c#
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
}
    ```
This step will require ``using Microsoft.Extensions.DependencyInjection;``

1. In the `Configure` method, configure the app to use MVC:
  
    ```c#
public void Configure(IApplicationBuilder app)
{
    app.UseMvc();
}
    ```

1. Run the site and verify the message is returned from your MVC controller

## Extra Credit

1. Change the controller to render a view instead of returning a string directly

1. Use the ``[HttpGet("/")]`` attribute to change the route the action method(s) match
