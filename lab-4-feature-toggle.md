# Avanade DevOps HOL - Lab 4 - Feature Toggle

In this lab, we add a Feature Toggle mechanism to our web application. The toggle will be enabled by json config and tested locally.

Based on [this](https://microsoft.github.io/PartsUnlimited/apm/200.6x-APM-FeatureFlagforWebapps.html) article and [this](https://github.com/jason-roberts/FeatureToggle/tree/master/src/Examples/AspDotNetCoreExample) example.

## Prerequisites

- Complete [Lab 3 - Analysis with SonarQube](lab-3-analysis-with-sonarqube.md).

## Tasks

1. Add the following NuGet package to your Web Application project:
    - FeatureToggle (by Jason Roberts)

1. Apply the following code changes to your Web Application:

    - <details><summary>Add the feature class "CheckPhoneNumber" in a folder named "Feature"</summary>

        ```csharp
        using FeatureToggle;

        namespace MyDevOpsApp.Features
        {
            public class CheckPhoneNumber : SimpleFeatureToggle { }
        }
        ```
    </details>

    - <details><summary>Add a model class "ContactViewModel", which will be used to bring the Feature Toggle setting to the Contact page</summary>

        ```csharp
        using FeatureToggle;

        namespace MyDevOpsApp.Models
        {
            public class ContactViewModel
            {
                public IFeatureToggle CheckPhoneNumber { get; set; }

                public string Name { get; set; }

                public int PhoneNumber { get; set; }
            }
        }
        ```
    </details>

    - <details><summary>Modify "Controllers/HomeController" to bind the new viewmodel to the Contact page</summary>

        ```csharp
        ...

        private readonly CheckPhoneNumber _checkPhoneNumber;

        public HomeController(CheckPhoneNumber checkPhoneNumber)
        {
            _checkPhoneNumber = checkPhoneNumber;
        }

        ...

        public IActionResult Contact()
        {
            ViewData["Message"] = "Your contact page.";

            return View(new ContactViewModel { CheckPhoneNumber = _checkPhoneNumber });
        }

        ...
        ```
    </details>

    - <details><summary>Modify "Views/Home/Contact"</summary>

        ```csharp
        @model MyDevOpsApp.Models.ContactViewModel

        ...

        <form asp-action="Contact">
            <div asp-validation-summary="ModelOnly" class="text-danger"></div>
            <div class="form-group">
                <label asp-for="Name" class="control-label"></label>
                <input asp-for="Name" class="form-control" />
                <span asp-validation-for="Name" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="PhoneNumber" class="control-label"></label>

                @if (Model.CheckPhoneNumber.FeatureEnabled)
                {
                    @Html.TextBoxFor(m => m.PhoneNumber, new { @class = "form-control", placeholder = "555-555-5555", type = "tel", pattern = "\\d{3}[\\-]\\d{3}[\\-]\\d{4}", id = "phone" })
                }
                else
                {
                    @Html.TextBoxFor(m => m.PhoneNumber, new { @class = "form-control", placeholder = "Phone Number", id = "phone" })
                }

                <span asp-validation-for="PhoneNumber" class="text-danger"></span>
            </div>
            <div class="form-group">
                <input id="submit" type="submit" value="Create" class="btn btn-default" />
            </div>
        </form>
        ```
    </details>

    - <details><summary>Modify "Startup"</summary>

        ```csharp
        public class Startup
        {
            public Startup(IHostingEnvironment env)
            {
                var builder = new ConfigurationBuilder()
                    .SetBasePath(env.ContentRootPath)
                    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
                    .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                    .AddEnvironmentVariables();
                Configuration = builder.Build();
            }

            public IConfigurationRoot Configuration { get; }

            // This method gets called by the runtime. Use this method to add services to the container.
            public void ConfigureServices(IServiceCollection services)
            {
                // Set provider config so file is read from content root path
                var provider = new AppSettingsProvider { Configuration = Configuration };

                // Add your feature here
                services.AddSingleton(new CheckPhoneNumber { ToggleValueProvider = provider });

                services.AddMvc();
            }

            ...
        }

        ```
    </details>

    - <details><summary>Modify Unit Tests</summary>

        Change every occurrence of:
        ```csharp
        HomeController controller = new HomeController();
        ```

        to:
        ```csharp
        HomeController controller = new HomeController(new CheckPhoneNumber());
        ```
    </details>


1. Use a json config file to store the feature toggle setting

    - <details><summary>Add config file "appsettings.json" with the feature toggle activated</summary>

        ```json
        {
            "FeatureToggle": {
                "CheckPhoneNumber": true
            },
            "Logging": {
                "IncludeScopes": false,
                "LogLevel": {
                "Default": "Warning"
                }
            }
        }
        ```
    </details>

1. Run the web application locally and test the new Contact form:
    1. Disable the feature by editing the config, reload the page:
        1. Enter any phone number. Notice how no validation error is given
    1. Enable the feature, reload the page:
        1. Enter phone number 0123456789, hit submit, and notice the validation error
        1. Enter phone number 123-123-5678, submit and notice the page refreshes without error

1. Commit your code changes and let your pipeline do it's job

## Next steps

Continue with [Lab 5 - UI Testing](lab-5-ui-testing.md).