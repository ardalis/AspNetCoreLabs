# Building a Simple API

## Setup
1. Start with a new Empty Web app)
1. Open the `project.json` file
1. In the `dependencies` section, add an entry for the "Microsoft.AspNetCore.Mvc": "1.0.0" package:

    ```JSON
"dependencies": {
    ...,
    "Microsoft.AspNetCore.Mvc": "1.0.0",
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

1. Create a folder called `Models` and create a class called `Product` in that folder:

    ```C#
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
}
    ```
  
1. Create a folder called `Controllers` and create a class called `ProductsController` in that folder.

1. Add an attribute route `[Route("/api/products")]` to the `ProductsController` class:

    ```C#
[Route("/api/products")]
public class ProductsController
{
}
    ```
Note that routes can be applied at the action or controller level.
  
1. Add a `Get` method to `ProductsController` that returns a `string` "Hello API World" with an attribute route

    ```C#
[Route("/api/products")]
public class ProductsController
{
public string Get() => "Hello World";
}
    ```

1. Run the application and navigate to `/api/products`, it should return the string "Hello World".

## Returning JSON from the controller

1. Add a static list of projects to the top of `ProductsController`:

    ```C#
  public class ProductsController
  {
      private static List<Product> _products = new List<Product>(new[] {
          new Product() { Id = 1, Name = "Lawn Gnome" },
          new Product() { Id = 2, Name = "Light Saber" },
          new Product() { Id = 3, Name = "Magic Wand" },
      });
      ...
  }
    ```
This code will require ``using System.Collections.Generic;``

1. Change the `Get` method in `ProductsController` to return `IEnumerable<Product>` and return the list of products.

    ```C#
public IEnumerable<Product> Get()
{
    return _products;
}
    ```

1. Run the application and navigate once more to `/api/products`. You should see a JSON payload of with all of the products. Note that JSON is the default format used by ASP.NET Core MVC.

1. Make a POST request (using [Fiddler](https://www.telerik.com/download/fiddler) or [Postman](https://www.getpostman.com/apps) for example) to `/api/products`, you should see a JSON payload with all products.

1. Restrict the `Get` method to respond to GET requests by adding an `[HttpGet]` attribute to the method:

    ```C#
  [HttpGet]
  public IEnumerable<Product> Get()
  {
      return _products;
  }
    ```
Make another POST request; you should get a 404 status code.

## Add a method to Get a single product

1. Add a `Get` method to the `ProductsController` that takes an `int id` parameter and returns `Product`.

    ```C#
public Product Get(int id)
{
    return _products.FirstOrDefault(p => p.Id == id);
}
    ```
This code will require ``using System.Linq;``

1. Add an `HttpGet` route specifying the `id` as a route parameter:

  ```C#
[HttpGet("{id}")]
public Product Get(int id)
{
    return _products.FirstOrDefault(p => p.Id == id);
}
  ```

1. Run the application, and navigate to `/api/products/1`, you should see a JSON response for the first product.

1. Navigate to `/api/products/25`, it should return a 204 status code. (Make the request in Fiddler/Postman, or with Developer Tools turned on. Otherwise, you will probably just see the URL change to `/api/products/1` if that's the last request you made in the browser)

1. Change the `Get` method in the `ProductsController` to return a 404 if the product search returns null, or an ``ObjectResult`` if a product is found.

    ```C#
[HttpGet("{id}")]
public IActionResult Get(int id)
{
    var product = _products.FirstOrDefault(p => p.Id == id);

    if (product == null)
    {
        return new NotFoundResult();
    }

    return new ObjectResult(product);
}
    ```

1. Run the application and navigate to `/api/products/40` and it should return a 404 status code.

## Adding to the list of products

1. Add a `Post` method to `ProductsController` the takes a `Product` as input and adds it to the list of products:

    ```C#
public void Post(Product product)
{
    _products.Add(product);
}
    ```

1. Add an `[HttpPost]` attribute to the method to constrain it to the POST HTTP verb:

    ```C#
[HttpPost]
public void Post(Product product)
{
    _products.Add(product);
}
    ```
  
1. Add a `[FromBody]` attribute to the `product` argument:

    ```C#
  [HttpPost]
  public void Post([FromBody]Product product)
  {
      _products.Add(product);
  }
    ```

1. Run the application, open a tool like [fiddler](http://www.telerik.com/fiddler) or [POSTman](https://www.getpostman.com/), and post a JSON payload with the `Content-Type` header `application/json` to `/api/products`:

    ```JSON
{
    "Id" : "4",
    "Name": "Sonic Screwdriver"
}
    ```

1. Make a GET request to `/api/products` and the new entity should show up in the list.

**Note**: The default JSON format will be camelCase by default in RTM.

## Adding XML support

1. Add the `Microsoft.AspNetCore.Mvc.Formatters.Xml` package to `project.json`:

    ```JSON
"dependencies": {
...
"Microsoft.AspNetCore.Mvc.Formatters.Xml": "1.0.0-rc2-final",
},
    ```

1. In `Startup.cs` add a call to  `AddXmlSerializerFormatters()` chained off the `AddMvc` method in `ConfigureServices`:

    ```C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddXmlSerializerFormatters();
}
    ```

1. Run the application and make a GET request to `/api/products` with the Accept header `application/xml`. The response should be an XML payload of the products.

    ```JSON
// Example fiddler Headers
User-Agent: Fiddler
Host: localhost:5000
Accept: application/xml
    ```

## Restrict the ProductsController to be JSON only

1. Add a `[Produces("application/json")]` attribute to the `ProductsController` class:

    ```C#
[Produces("application/json")]
[Route("/api/products")]
public class ProductsController
    ```

1. Re-attempt a request specifying 'application/xml' in the Accept header. You should receive JSON this time.
