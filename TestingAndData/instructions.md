# Testing Data-Driven MVC Apps

## Open Visual Studio - Create Solution Structure

1. Create a new Empty Solution called "Brainstormer"

![New Project Image](newproject.png)

1. Right click on the Solution. Add > New Project. Create a new ASP.NET Core Web Application project (don't add any user accounts) called "Brainstormer.Web". Don't add Application Insights.

![New Project Image](add-brainstormer-web.png)

1. Choose Web Application. Click "Change Authentication" and make sure Authentication is set to "No Authentication".

![New Project Image](select-template.png)

1. Once the project is created, test it with CTRL+F5 and confirm it runs.

### Configure XUnit

1. Add another New Project to the solution. Choose a .NET Core Console Application and name it "Brainstormer.Tests".

![New Project Image](add-test-project.png)

1. Open the **test project's** *project.json* and update its entire contents as follows:

    ```JSON
{
  "version": "1.0.0-*",

  "dependencies": {
    "Microsoft.NETCore.App": {
      "type": "platform",
      "version": "1.0.0-rc2-3002702"
    },
    "xunit": "2.1.0",
    "dotnet-test-xunit": "1.0.0-rc2-*",
    "Microsoft.AspNetCore.TestHost": "1.0.0-rc2-final",
    "Brainstormer.Web": "1.0.0-*",
    "Newtonsoft.Json": "8.0.3"
  },
  "testRunner": "xunit",
  "frameworks": {
    "netcoreapp1.0": {
      "imports": [
        "dotnet5.4",
        "portable-net451+win8"
      ]
    }
  }
}
    ```

Note that "Brainstormer.Web" must match the name you gave the web project.

1. Add a new folder to the test project called "IntegrationTests".

1. Add a new class called "TestStartup" and edit it to contain the following:

    ```c#
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;

namespace Brainstormer.Tests.IntegrationTests
{
    public class TestStartup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvc();
        }

        public void Configure(IApplicationBuilder app)
        {
            app.UseMvc(routes =>
            {
                routes.MapRoute(
                    name: "default",
                    template: "{controller=Home}/{action=Index}/{id?}");
            });
        }
    }
}    
    ```

1. Add another class called "TestHostingEnvironment":

    ```c#
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.FileProviders;

namespace Brainstormer.Tests.IntegrationTests
{
    public class TestHostingEnvironment : IHostingEnvironment
    {
        public string EnvironmentName { get; set; }
        public string ApplicationName { get; set; }
        public string WebRootPath { get; set; }
        public IFileProvider WebRootFileProvider { get; set; }
        public string ContentRootPath { get; set; }
        public IFileProvider ContentRootFileProvider { get; set; }
    }
}
    ```

1. Integration tests use instances of ``HttpClient``, which are creatd from a ``TestServer``. You can do this in each test, but that tends to get repetitive, so the last helper class you need is a "TestClientFactory":

    ```c#
using System.IO;
using System.Net.Http;
using System.Net.Http.Headers;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.TestHost;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.FileProviders;
using Microsoft.Extensions.PlatformAbstractions;

namespace Brainstormer.Tests.IntegrationTests
{
    public class TestClientFactory
    {
        public static HttpClient Create()
        {
            var builder = new WebHostBuilder()
                .ConfigureServices(services =>
                {
                    var env = new TestHostingEnvironment();
                    env.ContentRootPath = Path.GetFullPath(Path.Combine(
                        PlatformServices.Default.Application.ApplicationBasePath,
                        "..", "..", "..", "..", "Brainstormer.Web"));
                    env.ContentRootFileProvider = new PhysicalFileProvider(env.ContentRootPath);
                    env.WebRootPath = env.ContentRootPath;
                    env.WebRootFileProvider = new PhysicalFileProvider(env.WebRootPath);
                    env.ApplicationName = "Brainstormer.Web";
                    services.AddSingleton<IHostingEnvironment>(env);
                })
                .UseEnvironment("Testing")
                .UseStartup<TestStartup>();
            var server = new TestServer(builder);
            var client = server.CreateClient();

            // client always expects json results
            client.DefaultRequestHeaders.Clear();
            client.DefaultRequestHeaders.Accept.Add(
                new MediaTypeWithQualityHeaderValue("application/json"));

            return client;
        }
    }
}
    ```
The TestHostingEnvironment and its related path properties are necessary for Views to be accessed correctly. Otherwise, the tests will look for the views in the test project, not the web project.

1. Now you're ready to add your first test. You can verify that a request to "/" returns OK by adding the following class, "HomeControllerShould":

    ```c#
using System.Net.Http;
using Xunit;


namespace Brainstormer.Tests.IntegrationTests
{
    public class HomeControllerShould
    {
        private readonly HttpClient _client;

        public HomeControllerShould()
        {
            _client = TestClientFactory.Create();
        }

        [Fact]
        public async void ReturnOkGivenRootPathRequest()
        {
            var response = await _client.GetAsync("/");
            response.EnsureSuccessStatusCode();
        }
    }
}
    ```

1. Run the tests. You can do this from Visual Studio, or from a command prompt (in the test project folder) via ``dotnet test``. Run from the command line, you should see:

![New Project Image](command-line-tests.png)



