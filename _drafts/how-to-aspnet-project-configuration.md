# How to - ASP.NET core project configuration

## Basic example

### Create and run

Create a webapi project and execute it:

```sh
dotnet new webapi -o aspnet-core-webapi-example --no-https
cd aspnet-core-webapi-example
dotnet run
```

The template uses the port 5056 in my example, change this ro rhe port that you have assigned for your project

### Send a HTTP request

Execute a GET request to the API created with the template:

```sh
curl -v http://localhost:5056/WeatherForecast
```
### See the swagger documentation

Open `http://localhost:5056/swagger` to see the page with the swagger documentation.

Open `http://localhost:5056/swagger/v1/swagger.json` to see the swagger specification in JSON format.

## Project templates

### List all available project templates

```sh
dotnet new --list
```

### Create project using the "ASP.NET Core Web API" template

```sh
dotnet new webapi -o dotnet-webapi-example --no-https
```
Description:

- The `webapi` parameter selects what template to use when creating your app.
- The `-o` parameter creates a directory named `dotnet-webapi-example` where your app is stored.
- The `--no-https` flag specifies not to enable HTTPS.

### Create project using the "ASP.NET Core with Angular" template

```sh
dotnet new angular -o dotnet-angular-example --no-https
```

## Project files description

- `{PROJECT}.csproj` defines some project settings, such as, .NET SDK version to target.
- `Program.cs` contains the app startup code and middleware configuration.
- `appsettings.json` and `appsettings.{ENVIRONMENT}.json` is the common source of settings.
- `Properties/launchSettings.json` and `.vscode/launch.json` are tooling configuration files for the `Development` environment. Defines different profile settings for the local development environment.

## Program.cs and top-level statements

You can see that `Program.cs` is a source code file wuthout namespace, class and method definitions. It is using "top-level statements".

"Top-level statements" allows to write an small source code file without the `Main` method in the project.

NOTE: The compiler generates a method to serve as the program entry point for a project with top-level statements. The name of this method isn't actually `Main`, it's an implementation detail that your code can't reference directly.

NOTE: Namespaces are optional and `global` namespace (the "root" namespace) will be the default. In this case the generated static class and static method used as entry point in Program.cs will be in the global namespace because the generated code does not have an `namespace` wrapper.

You can add `using` statements, these statements must be before any other statements in the file. Otherwise, it's a compiler error.

A project can have only one file with top-level statements, that is only one entry point. Putting top-level statements in more than one file in a project results in a compiler error.

Of course, a project can have any number of source code files that don't have top-level statements.

Top-level statements are implicitly in the global namespace.

The signature of the entry point method depends on whether the top-level statements contain the `await` keyword or the `return` statement. The following table shows what the method signature would look like, using the method name EntyPoint in the table for convenience.

| Top-level code contains    | Implicit Main signature                    |
|----------------------------|--------------------------------------------|
| `await` and `return`       | static async Task<int> Main(string[] args) |
| `await`                    | static async Task Main(string[] args)      |
| `return`                   | static int Main(string[] args)             |
| No `await` and no `return` | static void Main(string[] args)            |

More info: https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/program-structure/top-level-statements

## Host configuration

On startup, an ASP.NET Core app builds a _host_. The host encapsulates all of the app's resources, such as:

- An HTTP server implementation
- Middleware components
- Logging
- Dependency injection (DI) services
- Configuration

There are three different hosts:

- .NET WebApplication Host
- .NET Generic Host
- ASP.NET Core Web Host

The .NET Minimal Host is recommended and used in all the ASP.NET Core templates. The Minimal and Generic hosts share many of the same interfaces and classes. The ASP.NET Core Web Host is available only for backwards compatibility.

NOTE: Web Host remains available only for backward compatibility. The ASP.NET Core templates create a .NET Generic Host, which is recommended for all app types.

### Host vs app configuration

Before the app is configured and started, a host is configured and launched. The host is responsible for app startup and lifetime management. Both the app and the host are configured using the configuration providers described in this topic. Host configuration key-value pairs are also included in the app's configuration.

### .NET WebApplication Host

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
// builder.Services.AddRazorPages();
// builder.Services.AddControllersWithViews();

var app = builder.Build();
```

The WebApplicationBuilder.Build method configures a host with a set of default options, such as:

- Use Kestrel as the web server and enable IIS integration.
- Load configuration from appsettings.json, environment variables, command line arguments, and other configuration sources.
- Send logging output to the console and debug providers.

### .NET Generic Host and non-web scenarios

The Generic Host allows other types of apps to use cross-cutting framework extensions, such as logging, dependency injection (DI), configuration, and app lifetime management. For more information, see .NET Generic Host in ASP.NET Core and Background tasks with hosted services in ASP.NET Core.

More info:
https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host
https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/hosted-services

### Default builder settings

The [CreateDefaultBuilder](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.hosting.host.createdefaultbuilder) method:

- Sets the "content root" to the path returned by GetCurrentDirectory.
- Loads host configuration from:
-- Environment variables prefixed with DOTNET_ (for example, DOTNET_ENVIRONMENT) using the "Environment Variables configuration provider". The prefix (DOTNET_) is stripped when the configuration key-value pairs are loaded.
-- Command-line arguments using the "Command-line configuration provider".
- Loads app configuration from:
-- `appsettings.json`.
-- `appsettings.{ENVIRONMENT}.json`.
-- User secrets when the app runs in the Development environment.
-- Environment variables.
-- Command-line arguments.
- Adds the following logging providers:
-- Console
-- Debug
-- EventSource
-- EventLog (only when running on Windows)
- Enables "scope validation" and "dependency validation" when the environment is Development.

The [ConfigureWebHostDefaults](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.hosting.generichostbuilderextensions.configurewebhostdefaults) method:

- Loads host configuration from environment variables prefixed with ASPNETCORE_.
- Sets Kestrel server as the web server and configures it using the app's hosting configuration providers.
- Adds Host Filtering middleware.
- Adds Forwarded Headers middleware if ASPNETCORE_FORWARDEDHEADERS_ENABLED equals true.
- Enables IIS integration. For the IIS default options, see Host ASP.NET Core on Windows with IIS.

More info:
https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host
https://docs.microsoft.com/en-us/aspnet/core/fundamentals
https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/options
https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/iis/#iis-options

## Configuration in ASP.NET Core

Configuration in ASP.NET Core is performed using one or more "configuration providers". Configuration providers read configuration data from key-value pairs using a variety of configuration sources:

- Settings files, such as `appsettings.json`
- Environment variables
- Azure Key Vault
- Azure App Configuration
- Command-line arguments
- Custom providers, installed or created
- Directory files
- In-memory .NET objects

### Configuration providers

The following list shows the configuration providers available to ASP.NET Core apps:

- Azure Key Vault configuration provider - Provides configuration from: Azure Key Vault
- Azure App configuration provider - Provides configuration from: Azure App Configuration
- Command-line configuration provider - Provides configuration from: Command-line parameters
- Custom configuration provider - Provides configuration from: Custom source
- Environment Variables configuration provider - Provides configuration from: Environment variables
- File configuration provider - Provides configuration from: INI, JSON, and XML files
- Key-per-file configuration provider - Provides configuration from: Directory files
- Memory configuration provider - Provides configuration from: In-memory collections
- User secrets - Provides configuration from: File in the user profile directory

Configuration sources are read in the order that their configuration providers are specified. Order configuration providers in code to suit the priorities for the underlying configuration sources that the app requires.

A typical sequence of configuration providers is:

1. `appsettings.json`
2. `appsettings.{ENVIRONMENT}.json`
3. [User secrets](https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets)
4. Environment variables using the "Environment Variables configuration provider".
5. Command-line arguments using the "Command-line configuration provider".

A common practice is to add the Command-line configuration provider last in a series of providers to allow command-line arguments to override configuration set by the other providers.

The preceding sequence of providers is used in the default configuration.

### Default configuration

When `WebApplication.CreateBuilder` is invoked, it initializes a new instance of the `WebApplicationBuilder` class (builder) with preconfigured defaults. The initialized builder provides default configuration for the app in the following order:

1. `ChainedConfigurationProvider`: Adds an existing `IConfiguration` as a source. In the default configuration case, adds the host configuration (https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration#default-host-configuration) and setting it as the first source for the app configuration.
2. `appsettings.json` using the "JSON configuration provider".
3. `appsettings.{ENVIRONMENT}.json` using the "JSON configuration provider". For example, appsettings.Production.json and appsettings.Development.json.
4. ["App secrets"](https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets) when the app runs in the Development environment.
5. Environment variables using the "Environment Variables configuration provider".
6. Command-line arguments using the "Command-line configuration provider".

Configuration providers that are added later override previous key settings. For example, if MyKey is set in both `appsettings.json` and the environment, the environment value is used. Using the default configuration providers, the Command-line configuration provider overrides all other providers.

For more information on CreateBuilder, see Default builder settings: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host#default-builder-settings

## Environment configuration

More info: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/environments

### DOTNET_ and ASPNETCORE_ environment variables

Host configuration values are loaded from three different sources:
- Environment variables that have the prefix DOTNET_. The environment variables have the prefix removed and are added to the collection.
- Environment variables that have the prefix ASPNETCORE_. For ASP.NET Core apps only, these environment variables are also added.
- Command line arguments.

For example, ASP.NET Core will use the value of the URLS key from the configuration system. You can set the URLS using either of the following environment variables:

- DOTNET_URLS
- ASPNETCORE_URLS

If you set both of these environment variables, the ASPNETCORE_URLS parameter takes precedence.

### Setting environment variables in the system environment

#### Current session

To set for example `ASPNETCORE_ENVIRONMENT` for the current session when the app is started using `dotnet run`, use the following commands.

in cmd:

```batch
set ASPNETCORE_ENVIRONMENT=Staging
dotnet run --no-launch-profile
```

in PowerShell:

```powershell
$Env:ASPNETCORE_ENVIRONMENT = "Staging"
dotnet run --no-launch-profile
```

in linux:

```sh
ASPNETCORE_ENVIRONMENT=Staging dotnet run --no-launch-profile
```

NOTE: `--no-launch-profile` is needed if you dont want to read environment variables from `launchSettings.json` after.

NOTE: When using Visual Studio Code, environment variables can be set in the `.vscode/launch.json` file.

#### Globally

In Windows you can either:
- Open the Control Panel > System > Advanced and add or edit the ASPNETCORE_ENVIRONMENT value in System level.
- Open an administrative command prompt and use the setx command:
```batch
setx ASPNETCORE_ENVIRONMENT Staging /M
```
The /M switch sets the environment variable at the system level. If the /M switch isn't used, the environment variable is set for the user account.
- or open an administrative PowerShell command prompt and use `[Environment]::SetEnvironmentVariable`:
```powershell
[Environment]::SetEnvironmentVariable("ASPNETCORE_ENVIRONMENT", "Staging", "Machine")
```
The Machine option sets the environment variable at the system level. If the option value is changed to User, the environment variable is set for the user account.

For Bash shell in Linux, you have multiple locations for setting the environment variable permanently:
- Bash as login shell will load `/etc/profile`, `~/.bash_profile`, `~/.bash_login`, `~/.profile` in the order.
- Bash as non-login interactive shell will load `~/.bashrc`.
- Bash as non-login non-interactive shell will load the configuration specified in environment variable $BASH_ENV.
Common users will include the line `export ASPNETCORE_ENVIRONMENT=Staging` in `~/.bashrc`.

#### Escape environment variables on Linux

On Linux, the value of URL environment variables must be escaped so systemd can parse it.

```sh
systemd-escape http://localhost:5001
# returns: http:--localhost:5001
```

### Setting environment variables in launchSettings.json

Environment variables set in launchSettings.json override values set in the system environment.

These environment variables are defined in `{PROFILE}.environmentVariables` node.

### Changing host configuration values in command line

Command line arguments override the value of environment variables. Example:

```sh
dotnet run --urls "http://localhost:5100;https://localhost:5101"
```

NOTE: By default, configuration values set on the command-line override configuration values set with all the other configuration providers.

### Execution environment configuration

To determine the runtime environment, ASP.NET Core reads from the following environment variables:

- DOTNET_ENVIRONMENT
- ASPNETCORE_ENVIRONMENT when the `WebApplication.CreateBuilder` method is called. The ASPNETCORE_ENVIRONMENT value overrides DOTNET_ENVIRONMENT.

NOTE: `WebApplication.CreateBuilder` automatically enables the "Developer Exception Page" when ASPNETCORE_ENVIRONMENT is set to `Development`.

Values can be:

- `Development`: You can see how launchSettings.json file sets ASPNETCORE_ENVIRONMENT to `Development`.
- `Staging`
- `Production`: The dafault if DOTNET_ENVIRONMENT and ASPNETCORE_ENVIRONMENT have not been set.

NOTE: `Production` is the default value if DOTNET_ENVIRONMENT and ASPNETCORE_ENVIRONMENT have not been set.

#### Configuring services, middleware, or custom behavior by environment

Use `WebApplicationBuilder.Environment` or `WebApplication.Environment` to conditionally add services or middleware depending on the current environment. Example:

```csharp
var builder = WebApplication.CreateBuilder(args);

// ...

var app = builder.Build();

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
  app.UseExceptionHandler("/Error");
  // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
  app.UseHsts();
}

// ...
```

### Endpoint configuration

This configuration indicates the IP addresses or host addresses with ports and protocols that the server should listen on for requests.

ASP.NET Core projects are configured to bind to a random HTTP port between 5000-5300 and a random HTTPS port between 7000-7300. This default configuration is specified in the generated `Properties/launchSettings.json` file and can be overridden. If no ports are specified, Kestrel binds to http://localhost:5000 and https://localhost:5001 (when a local development certificate is present).

Cross-server endpoint configuration can be specified with the following alternatives:

- Setting the environment variable ASPNETCORE_URLS.
- Configuring either `{PROFILE}.applicationUrl` (for non-IIS profiles) or `iisSettings.iisExpress.applicationUrl` (for IIS profiles) in launchSettings.json. This sets the ASPNETCORE_URLS environment variable and overrides values set in the system environment.
- `--urls` on the command line.
- `UseUrls`.

The value provided using these approaches can be one or more HTTP and HTTPS endpoints (HTTPS if a default cert is available) as a semicolon-separated list (for example, "Urls": "http://localhost:8000;http://localhost:8001").

Use "\*" to indicate that the server should listen for requests on any IP address or hostname using the specified port and protocol (for example, http://*:5000). The protocol (http:// or https://) must be included with each URL.

More info: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/endpoints

#### Kestrel specific endpoint configuration

Kestrel specific endpoint configuration in `appsettings.json` overrides all cross-server endpoint configurations (even --urls option specified in command line):

```json
{
  // ...

  "Kestrel": {
    "Endpoints": {
      "Https": {
        "Url": "https://localhost:9999"
      }
    }
  }

  // ...
}
```

By default, environment variables using the "Environment Variables configuration provider" are read after `appsettings.{ENVIRONMENT}.json`, therefore, the following Kestrel specific endpoint environment variable will override above configuration and it will be used for the Https endpoint.

```batch
set Kestrel__Endpoints__Https__Url=https://localhost:8888
```

### Host filtering

The Host Filtering Middleware is used for binding your app to specific hostnames. It is disabled by default, but can be enabled defining the `AllowedHosts` key in `appsettings.json` or `appsettings.{ENVIRONMENT}.json`.

The value for `AllowedHosts` must be a semicolon-delimited list of host names without port numbers. Setting for example `"AllowedHosts": "example.com"` binds the app only to example.com, and the host Filtering Middleware will not be not allowing the app to bind the app to any other hostname. For example, if you try to access your app using http://localhost/ address (with or without specifying a port) you will get default bad request (400) response.

## Properties/launchSettings.json

NOTE: launchSettings.json shouldn't store secrets. The Secret Manager tool can be used to store secrets for local development.

The launchSettings.json file:

- Is only used on the local development machine.
- Is not deployed.
- Contains profile settings.

### Profiles

Profiles are custom configurations of the environment that can be used when launching the application.

The value of `commandName` can specify the web server to launch:

- `IISExpress`: Launches IIS Express.
- `IIS`: No web server launched. IIS is expected to be available.
- `Project`: Launches Kestrel.

Project templates usually creates two profiles:

- `ProjectExample` (the profile name is the project name): As the first profile listed, this profile is used by default. The `commandName` key has the value `Project`, therefore, the Kestrel web server is launched.
- `IIS Express`: The `commandName` key has the value `IISExpress`, therefore, IISExpress is the web server.

More info: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/environments

### {PROFILE}.environmentVariables

Environment variables set in launchSettings.json override those set in the system environment.

### {PROFILE}.applicationUrl vs iisSettings.iisExpress.applicationUrl

NOTE: In this section `{PROFILE}` refers to all those non-IIS profiles, that is all profiles that launches kestrel and not IIS Express.

launchSettings.json is one alternative for setting application URLs via the `applicationUrl` property - you can see one under the `iisSettings` for IIS express, and one under `{PROFILE}`.

When you run your app from the command line with `dotnet run`, your app will use the `applicationUrl` property under `{PROFILE}`, the profile that launches kestrel. Otherwise when you run the app using an "IIS Express"-like profile, your app will use the `applicationUrl` property under the iisSettings.iisExpress node.

### The iisSettings.iisExpress.sslPort property

IIS Express supports HTTPS for development. You can use the `iisSettings.iisExpress.sslPort` property in launchSettings.json to set the HTTPS port. However this port must be in the range 44300–44399, otherwise your browser will be unable to connect to IIS, and you will not get a helpful error message.

`sslPort` has a default value of 0. The 0 value of `sslPort` means Visual Studio expects to launch an HTTP site, and the URL should be one specified in iisSettings.iisExpress.applicationUrl.

### HTTPS in non-IIS profiles

For non-IIS profiles, you have to include the HTTPS URL in the applicationUrl property.

Example of configuration using HTTPS both in IISExpress an Project type profiles:

```json
{
  // ...
  "iisSettings": {
    // ...
    "iisExpress": {
      "applicationUrl": "http://localhost:5990",
      "sslPort": 44318
    }
  },
  "profiles": {
    "ProjectExample": {
      "commandName": "Project",
      "applicationUrl": "https://localhost:7220;http://localhost:5091",
      // ...
    },
    "IIS Express": {
      "commandName": "IISExpress",
      // ...
    }
  }
}
```

### Launching with a specific profile in command line

Using the dotnet run CLI command with the `--launch-profile` option set to the profile's name. This approach only supports Kestrel profiles, for running "IISExpress" profiles you will need Visual Studio.

```sh
dotnet run --launch-profile "AnotherProfile"
```

## Logging

Logging configuration is usually provided by the Logging section of `appsettings.{ENVIRONMENT}.json` files, where the `{ENVIRONMENT}` placeholder is the environment (Development, Staging or Production).

Logging categories are specified inside `Logging.LogLevel` and applies to all sub categories of the specified category. For example, the "Microsoft.AspNetCore" category applies to all categories that start with "Microsoft.AspNetCore" like the "Microsoft.AspNetCore.Routing.EndpointMiddleware" category.

### Provider specific logging configuration

A provider property can specify a LogLevel property. LogLevel under a provider specifies levels to log for that provider, and overrides the non-provider log settings.

Settings in `Logging.{PROVIDER NAME}.LogLevel` override settings in `Logging.LogLevel`, where the `{PROVIDER NAME}` placeholder is the provider name.

### Log Levels

The value specifies the minimum level to log for selected categories. This specifies the severity of the log and ranges from 0 to 6:

Trace = 0, Debug = 1, Information = 2, Warning = 3, Error = 4, Critical = 5, and None = 6.

Logging is enabled for messages at the specified level and higher. For example, if the `Default` category is `Information`, then it is logged for `Information` and higher (`Information`, `Warning`, `Error`, and `Critical` messages will be logged).

To suppress all logs, specify LogLevel `None`. `None` has a value of 6, which is higher than `Critical` (5).

NOTE: If no LogLevel is specified, `Default` (the default logging) category defaults to the `Information` level.

### Log categories

When an ILogger object is created, a category is specified.

The category string is arbitrary, but the convention is to use the class name. For example, in a controller the name might be `"TodoApi.Controllers.TodoController"`. The ASP.NET Core web apps use `ILogger<T>` to automatically get an ILogger instance that uses the fully qualified type name of `T` as the category.

### Log event ID

Each log can specify an event ID. An event ID associates a set of events. For example, all logs related to displaying a list of items might be 1001.

The logging provider may store the event ID in an ID field, in the logging message, or not at all. For example, the Debug provider doesn't show event IDs, but the console provider shows event IDs in brackets after the category.

### ASP.NET Core and EF Core categories

The following is a list of categories used by ASP.NET Core and Entity Framework Core, with notes about the logs:

- `Microsoft.AspNetCore`:	General ASP.NET Core diagnostics.
- `Microsoft.AspNetCore.DataProtection`:	Which keys were considered, found, and used.
- `Microsoft.AspNetCore.HostFiltering`:	Hosts allowed.
- `Microsoft.AspNetCore.Hosting`:	How long HTTP requests took to complete and what time they started. Which hosting startup assemblies were loaded.
- `Microsoft.AspNetCore.Mvc`:	MVC and Razor diagnostics. Model binding, filter execution, view compilation, action selection.
- `Microsoft.AspNetCore.Routing`:	Route matching information.
- `Microsoft.AspNetCore.Server`:	Connection start, stop, and keep alive responses. HTTPS certificate information.
- `Microsoft.AspNetCore.StaticFiles`:	Files served.
- `Microsoft.EntityFrameworkCore`:	General Entity Framework Core diagnostics. Database activity and configuration, change detection, migrations.

###  Built-in logging providers

ASP.NET Core includes the following logging providers as part of the shared framework:

- Console: logs output to the console.
- Debug: writes log output by using the `System.Diagnostics.Debug` class. Calls to `System.Diagnostics.Debug.WriteLine` writes to the Debug provider.On Linux, the Debug provider log location is distribution-dependent and may be one of the following: `/var/log/message` or `/var/log/syslog`.
- EventSource: writes to a cross-platform event source with the name Microsoft-Extensions-Logging. On Windows, the provider uses ETW (Event Tracing for Windows).
- EventLog: sends log output to the Windows Event Log. Unlike the other providers, the EventLog provider does not inherit the default non-provider settings. If EventLog log settings aren't specified, they default to Warning.

NOTE: ASP.NET Core doesn't include a logging provider for writing logs to files.

### Third-party logging providers

Third-party logging frameworks that work with ASP.NET Core:

- [elmah.io](https://elmah.io/)
- [Gelf](https://docs.graylog.org/en/2.3/pages/gelf.html)
- [JSNLog](https://jsnlog.com/)
- [KissLog.net](https://kisslog.net/)
- [Log4Net](https://logging.apache.org/log4net/)
- [NLog](https://nlog-project.org/)
- [PLogger](https://www.nuget.org/packages/InvertedSoftware.PLogger.Core/)
- [Sentry](https://sentry.io/welcome/)
- [Serilog](https://serilog.net/)
- [Stackdriver](https://cloud.google.com/dotnet/docs/stackdriver#logging)

Some third-party frameworks can perform ["semantic logging", also known as "structured logging"](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging).

Using a third-party framework is similar to using one of the built-in providers:

1. Add a NuGet package to your project.
2. Call an `ILoggerFactory` extension method provided by the logging framework.

## The Kestrel web server

More info: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel

### Kestrel behind a reverse proxy

A reverse proxy is a server that receives HTTP requests from the network and can forwards them to the web server (Kestrel in this case).

Kestrel can be used by itself (as an edge Internet-facing web server) or with a reverse proxy server, such as IIS, Nginx, or Apache.

More info: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/when-to-use-a-reverse-proxy

## HTTPS

For ASP.NET Core web apps is recommended to use:

- HTTPS Redirection Middleware (`UseHttpsRedirection`) - to redirect HTTP requests to HTTPS.
- HSTS Middleware (`UseHsts`) - to send HTTP Strict Transport Security Protocol (HSTS) headers to clients.

NOTE: Apps deployed in a reverse proxy configuration allow the proxy to handle connection security (HTTPS). If the proxy also handles HTTPS redirection, there's no need to use HTTPS Redirection Middleware. If the proxy server also handles writing HSTS headers (for example, native HSTS support in IIS 10.0 (1709) or later), HSTS Middleware isn't required by the app.

More info: https://docs.microsoft.com/en-us/aspnet/core/security/enforcing-ssl

### HTTPS Redirection Middleware (UseHttpsRedirection)

```csharp
var builder = WebApplication.CreateBuilder(args);

// builder.Services.AddRazorPages();

var app = builder.Build();

// if (!app.Environment.IsDevelopment())
// {
//     app.UseExceptionHandler("/Error");
//     app.UseHsts();
// }

app.UseHttpsRedirection();
// app.UseStaticFiles();

// app.UseRouting();

// app.UseAuthorization();

// app.MapRazorPages();

app.Run();
```

`UseHttpsRedirection`:
- Uses the default `HttpsRedirectionOptions.RedirectStatusCode` (`Status307TemporaryRedirect`).
- Uses the default `HttpsRedirectionOptions.HttpsPort` (null) unless overridden by the ASPNETCORE_HTTPS_PORT environment variable or `IServerAddressesFeature`.

Is recommended to use temporary redirects rather than permanent redirects. Link caching can cause unstable behavior in development environments. If you prefer to send a permanent redirect status code when the app is in a non-Development environment.

Is recommended using HSTS to signal to clients that only secure resource requests should be sent to the app (only in production).

A port must be available for the middleware to redirect an insecure request to HTTPS.

Specify the HTTPS port using any of the approaches described in https://docs.microsoft.com/en-us/aspnet/core/security/enforcing-ssl#port-configuration.

The most common approach is to indicate a port with the secure scheme using the ASPNETCORE_URLS environment variable (applicationUrl property if you configure it on launchStettings.json). The environment variable configures the server. The middleware indirectly discovers the HTTPS port via `IServerAddressesFeature`. This approach doesn't work in reverse proxy deployments because when an app is run in a reverse proxy configuration, `IServerAddressesFeature` isn't available, so set the port using one of the other approaches.

If no port is available:

- Redirection to HTTPS doesn't occur.
- The middleware logs the warning "Failed to determine the https port for redirect."

### Edge deployments with HTTPS

A HTTP server could be used:

- By itself a an edge server that process requests (aka Public-facing edge server connections) directly from a network, including the Internet.
- With a reverse proxy server (such as IIS, Nginx, or Apache) that receives HTTP requests (aka reverse proxy connections) from the Internet and forwards them to the HTTP server.

When Kestrel or HTTP.sys is used as a public-facing edge server, Kestrel or HTTP.sys must be configured to listen on both:

- The secure port where the client is redirected (typically, 443 in production and 5001 in development).
- The insecure port (typically, 80 in production and 5000 in development).

The insecure port must be accessible by the client in order for the app to receive an insecure request and redirect the client to the secure port.

### HTTP redirection to HTTPS causes ERR_INVALID_REDIRECT on the CORS preflight request

Requests to an endpoint using HTTP that are redirected to HTTPS by `UseHttpsRedirection` fail with ERR_INVALID_REDIRECT on the CORS preflight request.

API projects can reject HTTP requests rather than use `UseHttpsRedirection` to redirect requests to HTTPS.

### Do not use HTTP redirection on Web API projects

API projects can reject HTTP requests rather than use `UseHttpsRedirection` to redirect requests to HTTPS.

Do not use `RequireHttpsAttribute` on Web APIs that receive sensitive information. `RequireHttpsAttribute` uses HTTP status codes to redirect browsers from HTTP to HTTPS. API clients may not understand or obey redirects from HTTP to HTTPS. Such clients may send information over HTTP. Web APIs should either:

- Not listen on HTTP.
- Close the connection with status code 400 (Bad Request) and not serve the request.

To disable HTTP redirection in an API, set the ASPNETCORE_URLS environment variable or use the --urls command line flag.

### Do not use HSTS on Web API projects

The default API projects don't include HSTS because HSTS is generally a browser only instruction. Other callers, such as phone or desktop apps, do not obey the instruction. Even within browsers, a single authenticated call to an API over HTTP has risks on insecure networks. The secure approach is to configure API projects to only listen to and respond over HTTPS.

## Controller vs ControllerBase

Don't create a web API controller by deriving from the Controller class. Controller derives from ControllerBase and adds support for views, so it's for handling web pages, not web API requests. There's an exception to this rule: if you plan to use the same controller for both views and web APIs, derive it from Controller.

### AddControllers vs AddControllersWithViews

TODO: more info required here.

## Routing

ASP.NET Core controllers use the Routing middleware to match the URLs of incoming requests and map them to actions. Route templates:

- Are defined at startup in Program.cs or in attributes.
- Describe how URL paths are matched to actions.
- Are used to generate URLs for links. The generated links are typically returned in responses.

Actions are either conventionally-routed or attribute-routed. ASP.NET Core apps could also mix the use of conventional routing and attribute routing.

It's typical to use conventional routes for controllers serving HTML pages for browsers, and attribute routing for controllers serving REST APIs.

Routing is configured using the `UseRouting` and `UseEndpoints` middleware. To use controllers:

- Call `MapControllers` to map attribute routed controllers.
- Call `MapControllerRoute` or `MapAreaControllerRoute`, to map both conventionally routed controllers and attribute routed controllers.

More info: https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/routing

### Attribute routing

REST APIs should use attribute routing to model the app's functionality as a set of resources where operations are represented by HTTP verbs.

Attribute routing uses a set of attributes to map actions directly to route templates. Placing a route on the controller or action makes it attribute-routed.

Here we have the following route templates:

- All the HTTP verb templates.
- `[Route]`

#### HTTP verb templates

ASP.NET Core has the following HTTP verb templates:

```
[HttpGet]
[HttpPost]
[HttpPut]
[HttpDelete]
[HttpHead]
[HttpPatch]
```

#### The ApiController attribute

The `[ApiController]` attribute can be applied to a controller class to enable the following opinionated, API-specific behaviors:

- Attribute routing requirement - It means that actions are inaccessible via conventional routes.
- Automatic HTTP 400 responses - Makes model validation errors automatically trigger an HTTP 400 response.
- Binding source parameter inference - Read below.
- Multipart/form-data request inference - Applies an inference rule when an action parameter is annotated with the [FromForm] attribute. The multipart/form-data request content type is inferred.
- Problem details for error status codes - Transforms an error result (a result with status code 400 or higher) to a result with [ProblemDetails](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.problemdetails). The `ProblemDetails` type is based on the [RFC 7807](https://tools.ietf.org/html/rfc7807) specification for providing machine-readable error details in an HTTP response.

**Binding source parameter inference**

A binding source attribute defines the location at which an action parameter's value is found. The following binding source attributes exist:

- `[FromBody]` - Request body
- `[FromForm]` - Form data in the request body
- `[FromHeader]` - Request header
- `[FromQuery]` - Request query string parameter
- `[FromRoute]` - Route data from the current request
- `[FromServices]` - The request service injected as an action parameter

NOTE: Don't use `[FromRoute]` when values might contain %2f (that is /). %2f won't be unescaped to /. Use `[FromQuery]` if the value might contain %2f.

Without the `[ApiController]` attribute or binding source attributes like `[FromQuery]`, the ASP.NET Core runtime attempts to use "the complex object model binder". "The complex object model binder" pulls data from value providers in a defined order.

More info: https://docs.microsoft.com/en-us/aspnet/core/web-api

### Conventional routing

Conventional routing means that it establishes a convention for URL paths.

Conventional routing is used with controllers and views. The default conventional route defined in ASP.NET Core MVC projects is as follows:

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

where:

- {controller=Home}, maps to the controller name.
- {action=Index}, maps to the action name.
- {id?} is used for an optional id. The ? in {id?} makes it optional. id is used to map to a model entity.

The default conventional route:

- Supports a basic and descriptive routing scheme.
- Is a useful starting point for UI-based apps.
- Is the only route template needed for many web UI apps. For larger web UI apps, another route using Areas is frequently all that's needed.

Using this default route allows the following examples:

- `/Products/List` maps to the `ProductsController.List` action.
- `/Blog/Article/17` maps to `BlogController.Article` and typically model binds the id parameter to 17.

This mapping:

- Is based on the controller and action names only.
- Isn't based on namespaces, source file locations, or method parameters.

Using conventional routing with the default route allows creating the app without having to come up with a new URL pattern for each action. For an app with CRUD style actions, having consistency for the URLs across controllers:

- Helps simplify the code.
- Makes the UI more predictable.

## Static files

Static File Middleware (`UseStaticFiles`) returns static files and short-circuits further request processing. These static files, such as HTML, CSS, images, and JavaScript, are assets an ASP.NET Core app will be serving directly to clients by default.

The default web app templates call the `UseStaticFiles` method in `Program.cs`, which enables static files to be served.

Static files are stored within the project's web root directory. The default directory is `{content root}/wwwroot`, but it can be changed with the `UseWebRoot` method.

More info: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/static-files

### Static file authorization

Static assets under `wwwroot` are publicly accessible when the default Static File Middleware (`app.UseStaticFiles();`) is called before `UseAuthentication`. Most apps follow this pattern.

When the Static File Middleware is called before the authorization middleware:

- No authorization checks are performed on the static files.
- Static files served by the Static File Middleware, such as those under `wwwroot`, are publicly accessible.

To serve static files based on authorization:

- Store them outside of `wwwroot`.
- Call `UseStaticFiles`, specifying a path, after calling `UseAuthorization`.
- Set the "fallback authorization policy".

### Setting HTTP response header for static files

A `StaticFileOptions` object can be used to set HTTP response headers. In addition to configuring static file serving from the web root, the following code sets the Cache-Control header:

```csharp
// makes static files publicly available in the local cache for one week (604800 seconds)
var cacheMaxAgeOneWeek = (60 * 60 * 24 * 7).ToString();
app.UseStaticFiles(new StaticFileOptions
{
    OnPrepareResponse = ctx =>
    {
        // makes static files publicly available in the local cache for one week (604800 seconds)
        ctx.Context.Response.Headers.Append(
             "Cache-Control", $"public, max-age={cacheMaxAgeOneWeek}");
    }
});
```

### Compressing and caching response of static files

NOTE: Middleware could have different ordering in different scenarios. For example, caching and compression ordering is scenario specific, and there are multiple valid orderings.

With `app.UseResponseCaching();` before `app.UseResponseCompression();` CPU usage could be reduced by caching the compressed response, but you might end up caching multiple representations of a resource using different compression algorithms such as Gzip or Brotli.

The following ordering combines static files to allow caching compressed static files:

```csharp
app.UseResponseCaching();
app.UseResponseCompression();
app.UseStaticFiles();
```

The following ordering creates a scenario where requests for static files are handled by Static File Middleware before Response Compression Middleware. Static files aren't compressed with this middleware order, but the Razor Pages responses can be compressed.

```csharp
// Static files aren't compressed by Static File Middleware.
app.UseStaticFiles();
app.UseRouting();
app.UseResponseCompression();
app.MapRazorPages();
```

## URL generation

The `IUrlHelper` interface is the underlying element of infrastructure between MVC and routing for URL generation. An instance of `IUrlHelper` is available through the Url property in controllers, views, and view components.

URL generation fails if any required route parameter doesn't have a corresponding value. If URL generation fails for a route, the next route is tried until all routes have been tried or a match is found.

### Generate URLs by action name

In the following example, the IUrlHelper interface is used through the Controller.Url property to generate a URL to another action

```csharp
public class UrlGenerationAttrController : Controller
{
    [HttpGet("custom")]
    public IActionResult Source()
    {
        var url = Url.Action("Destination");
        return ControllerContext.MyDisplayRouteInfo("", $" URL = {url}");
    }

    [HttpGet("custom/url/to/destination")]
    public IActionResult Destination()
    {
       return ControllerContext.MyDisplayRouteInfo();
    }
}
```

In an advanced example, `Url.Action("Buy", "Products", new { id = 17, color = "red" });` generates `/Products/Buy/17?color=red`.

`Url.Action` works both for "conventional routing" and "attribute routing".

### Generate URLs by route

The preceding code demonstrated generating a URL by passing in the controller and action name. `IUrlHelper` also provides the `Url.RouteUrl` family of methods. These methods are similar to `Url.Action`, but they don't copy the current values of action and controller to the route values. The most common usage of `Url.RouteUrl`:

- Specifies a route name to generate the URL.
- Generally doesn't specify a controller or action name.

```csharp
public class UrlGeneration2Controller : Controller
{
    [HttpGet("")]
    public IActionResult Source()
    {
        var url = Url.RouteUrl("Destination_Route");
        return ControllerContext.MyDisplayRouteInfo("", $" URL = {url}");
    }

    [HttpGet("custom/url/to/destination2", Name = "Destination_Route")]
    public IActionResult Destination()
    {
        return ControllerContext.MyDisplayRouteInfo();
    }
}
```

## Swagger documentation

Swagger documentation can be generated using the Swashbuckle.AspNetCore package. With this package you can explore the API via Swagger UI.

Add the Swagger generator to the services collection in Program.cs:

```csharp
builder.Services.AddControllers();

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
```

Enable the middleware for serving the generated JSON document and the Swagger UI, also in Program.cs:

```csharp
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
```

The preceding code adds the Swagger middleware only if the current environment is set to Development.

NOTE: The UseSwaggerUI method call enables the Static File Middleware.

Now you can launch the app and navigate to `https://localhost:<port>/swagger/v1/swagger.json`. The generated document describing the endpoints appears as shown in [OpenAPI specification (openapi.json)](https://docs.microsoft.com/en-us/aspnet/core/tutorials/web-api-help-pages-using-swagger?view=aspnetcore-6.0#openapi-specification-openapijson).

The Swagger UI can be found at `https://localhost:<port>/swagger`.

There are three main components of the Swashbuckle.AspNetCore package are:

- Swashbuckle.AspNetCore.Swagger: a Swagger object model and middleware to expose `SwaggerDocument` objects as JSON endpoints.
- Swashbuckle.AspNetCore.SwaggerGen: a Swagger generator that builds `SwaggerDocument` objects directly from your routes, controllers, and models. It's typically combined with the Swagger endpoint middleware to automatically expose Swagger JSON.
- Swashbuckle.AspNetCore.SwaggerUI: an embedded version of the Swagger UI tool. It interprets Swagger JSON to build a rich, customizable experience for describing the web API functionality. It includes built-in test harnesses for the public methods.

More info: https://docs.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-swashbuckle

## Authorization

Authorization Middleware (`UseAuthorization`) authorizes a user to access secure resources.

TODO: more info required here

## Error handling

TODO: populate this section

More info:
https://docs.microsoft.com/en-us/aspnet/core/web-api/handle-errors
https://docs.microsoft.com/en-us/aspnet/core/fundamentals/error-handling

## Middlewares

More info: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware

### Built-in middlewares

More info: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#built-in-middleware

### Response caching

Provides support for caching responses.	Must be invoked before components that require caching. `UseCORS` must come before `UseResponseCaching`.

More info: https://docs.microsoft.com/en-us/aspnet/core/performance/caching/middleware

### Response compression

Provides support for compressing responses.	Must be invoked before components that require compression.

More info: https://docs.microsoft.com/en-us/aspnet/core/performance/response-compression

## Project management

### Creating a solution

The following commands creates a solution, a console app, and two class libraries. Add the projects to the solution, and use the `--solution-folder` or the `--in-root` option of `dotnet sln` to organize the them into the Visual Studio's Solution Explorer panel.

```sh
# Create solution
dotnet new sln --name mysolution
# Create projects
dotnet new console --output src/myapp
dotnet new classlib --output src/mylib1
dotnet new classlib --output src/mylib2
# Include projects in the solution
dotnet sln add src/myapp/myapp.csproj --in-root
dotnet sln add src/mylib1/mylib1.csproj --solution-folder libs
dotnet sln add src/mylib2/mylib2.csproj --solution-folder libs
```

Reference:

- `--name <OUTPUT_NAME>` - The name for the created output. If no name is specified, the name of the current directory is used.
- `--in-root` - Places the projects in the root of the solution, rather than creating a solution folder.
- `--solution-folder <PATH>` - The destination solution folder path to add the projects to.

A "solution folder" is a virtual folder that is visible only in Solution Explorer in Visual Studio, where you can use it to group projects in a solution.

If the path to the project specified in `dotnet sln add` includes folders that contain the project folder, that portion of the path is used to create solution folders. You can override this default behavior by using the `--in-root` or `--solution-folder <PATH>` option.

### Install listed dependencies (aka package references)

The `dotnet restore` command restores packages listed in the project file. Restore also is done automatically with `dotnet build` and `dotnet run`.

As of NuGet 4.0, this runs the same code as `nuget restore`.

```sh
cd <PROJECT_PATH>
dotnet restore
```

### Add and install a dependency (aka package reference)

`dotnet add package` adds a package reference to the project file.

To install a NuGet package:

```sh
cd <PROJECT_PATH>
dotnet add package <PACKAGE_NAME>
# or
# dotnet add package <PACKAGE_NAME> --version <VERSION>
```

After the command completes, the added reference will be specified in the .csproj file.

### Update a dependency (aka package reference)

Use `dotnet add package` too. NuGet installs the latest version of the package when you use the command unless you specify the package version (-v switch).

### Remove a dependency (aka package reference)

```sh
dotnet remove package <PACKAGE_NAME>
```

### List dependencies (aka package references)

The following command lists all the package dependencies of the project/solution in the current directory.

```sh
dotnet list package
```

### Run

```
dotnet run
```

### Watch - Hot Reload

.NET Hot Reload applies code changes, including changes to stylesheets, to a running app without restarting the app and without losing app state.

NOTE: Hot Reload is supported for all ASP.NET Core projects.

Hot Reload is activated using the `dotnet watch` command. The `dotnet watch` command will build and start the app, and then update the running app whenever you make code changes. You can stop the app at any time by pressing `Ctrl+C`.

```sh
dotnet watch
```

To force the app to rebuild and restart, use the keyboard combination `Ctrl+R` in the command shell.

### Create a release build

`dotnet publish` publishes the application and its dependencies to a folder for deployment to a hosting system.

It's the only officially supported way to prepare the application for deployment. Depending on the type of deployment that the project specifies, the hosting system may or may not have the .NET shared runtime installed on it.

There are three type of deployments:
- Framework-dependent deployment (FDD)
- Framework-dependent executable (FDE)
- Self-contained deployment (SDC)

The `<TargetFramework>` setting of the project file specifies the default target framework when you publish your app. You can change the target framework to any valid Target Framework Moniker (TFM).

Unless otherwise set, the output directory of the dotnet publish command is `./bin/<BUILD-CONFIGURATION>/<TFM>/publish/`. The default BUILD-CONFIGURATION mode is `Debug` unless changed with the `-c` parameter. For example, `dotnet publish -c Release -f netcoreapp2.1` publishes to `./bin/Release/netcoreapp2.1/publish/`.

NOTE: You can reduce the total size of your deployment by enabling **globalization invariant mode**. This mode is useful for applications that are not globally aware and that can use the formatting conventions, casing conventions, and string comparison and sort order of the **invariant culture**. For more information about **globalization invariant mode** and how to enable it, see [.NET Globalization Invariant Mode](https://github.com/dotnet/runtime/blob/main/docs/design/features/globalization-invariant-mode.md).

More info:
https://docs.microsoft.com/en-us/dotnet/core/deploying/deploy-with-cli
https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-publish
https://docs.microsoft.com/en-us/dotnet/core/rid-catalog
https://docs.microsoft.com/en-us/dotnet/core/deploying/runtime-patch-selection

### Framework-dependent executable (FDE)

The app is configured to target a specific version of .NET. That targeted .NET runtime is required to be on any machine where the app runs.

Publishing an FDE creates an app that automatically rolls-forward to the latest .NET security patch available on the system that runs the app.

In this mode, a platform-specific executable host is created to host your cross-platform app. This mode is similar to FDD, as FDD requires a host in the form of the dotnet command. The host executable filename varies per platform and is named something similar to `<PROJECT-FILE>.exe`. You can run this executable directly instead of calling dotnet `<PROJECT-FILE>.dll`, which is still an acceptable way to run the app.

For the .NET 5 (and .NET Core 3.1) SDK CLI, framework-dependent executable (FDE) is the default mode for the basic dotnet publish command. You don't need to specify any other parameters, as long as you want to target the current operating system.

```
dotnet publish -c Release -r <RID> --self-contained false
dotnet publish -c Release
```

Where:
- `-r <RID>` - This switch uses an identifier (RID) to specify the target platform.
- `--self-contained false` - This switch disables the default behavior of the `-r` switch, which is to create a self-contained deployment (SCD). This switch creates an FDE.

Whenever you use the `-r` switch, the output folder path changes to: `./bin/<BUILD-CONFIGURATION>/<TFM>/<RID>/publish/`. For example `dotnet publish -f net5.0 -r win10-x64 --self-contained false` creates the following executable: `./bin/Debug/net5.0/win10-x64/publish/apptest1.exe`.

### Self-contained deployment (SDC)

Creates a platform-specific executable. Publishing an SCD includes all required .NET files to run your app but it doesn't include the [native dependencies of .NET](https://github.com/dotnet/core/blob/main/Documentation/prereqs.md). These dependencies must be present on the system before the app runs.

Publishing an SCD creates an app that doesn't roll forward to the latest available .NET security patch.

This approach bundles the .NET runtime and libraries with your application. SDC's don't have a dependency on runtime environments. Runtime version selection occurs at publishing time, not run time.

The restore event that occurs when publishing selects the latest patch version of the given runtime family. For example, `dotnet publish` will select .NET 5.0.3 if it's the latest patch version in the .NET 5 runtime family. The target framework (including the latest installed security patches) is packaged with the application.

An error occurs if the minimum version specified for an application isn't satisfied. `dotnet publish` binds to the latest runtime patch version (within a given major.minor version family). `dotnet publish` doesn't support the roll-forward semantics of `dotnet run`. For more information about patches and self-contained deployments, see the article on runtime patch selection in deploying .NET applications.

```
dotnet publish -c Release -r <RID> --self-contained true
```

Where:

- `-r <RID>` - This switch uses an identifier (RID) to specify the target platform.
- `--self-contained true` - This switch tells the .NET SDK to create an executable as an SCD.

### Framework-dependent deployment (FDD)

The app is configured to target a specific version of .NET. That targeted .NET runtime is required to be on any machine where the app runs.

Publishing an FDD creates an app that automatically rolls-forward to the latest .NET security patch available on the system that runs the app.

```
dotnet publish -c Release -p:UseAppHost=false
```

When you publish your app as an FDD, a `<PROJECT-NAME>.dll` file is created in the `./bin/<BUILD-CONFIGURATION>/<TFM>/publish/` folder. To run your app, navigate to the output folder and use the `dotnet <PROJECT-NAME>.dll` command.

## Hosting startup assemblies in ASP.NET Core

An 'IHostingStartup' (hosting startup) implementation allows an external assembly adding enhancements to an app at startup. For example, an external library can use a hosting startup implementation to provide additional configuration providers or services to an app.

Hosting startup can be implemented with either of the following project types:

- Class library
- Console app without an entry point

For either a class library- or console app-supplied hosting startup, you have to specify the hosting startup assembly's name in the ASPNETCORE_HOSTINGSTARTUPASSEMBLIES environment variable. The environment variable is a semicolon-delimited list of assemblies.

More info: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/platform-specific-configuration

## Deployment on Linux

TODO: populate this section

More info: https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/linux-nginx

## Tools

The dotnet tool install command provides a way for you to install .NET tools on your machine.

```sh
dotnet tool install <PACKAGE_NAME> --global
# or
dotnet tool install <PACKAGE_NAME> --tool-path <PATH>
# or
dotnet tool install <PACKAGE_NAME> --local
```

To use the command, you specify one of the following installation options:

- To install a global tool in the default location, use the --global option.
- To install a global tool in a custom location, use the --tool-path option.
- To install a local tool, omit the --global and --tool-path options.

Global tools are installed in the following directories by default when you specify the -g or --global option:

| OS	        | Path                        |
|-------------|-----------------------------|
| Linux/macOS	| $HOME/.dotnet/tools         |
| Windows	    | %USERPROFILE%\.dotnet\tools |

To install a tool for local access only (for the current directory and subdirectories), it has to be added to a tool manifest file (dotnet-tools.json under the .config directory). If the tool manifest file doesn't exist yet, create it by running the following command:

```sh
dotnet new tool-manifest
```

### List installed tools

```sh
dotnet tool list
```

### Search tools in Nuget

The `dotnet tool search` command provides a way for you to search NuGet for tools that can be used as .NET global, tool-path, or local tools. The command searches the tool names and metadata such as titles, descriptions, and tags.

The command uses the NuGet Search API. It filters on `packageType=dotnettool` to select only .NET tool packages.

```sh
dotnet tool search [--detail] <SEARCH_TERM>
```

### The "guid" tool

This tool quickly generates one or multiple GUIDs/UUIDs, in whatever format could be needed.

```sh
dotnet tool install -g dotnet-guid
```

### Copy folders when building

Add the following when you need to copy, for example, a directory from `$(BUILD_PATH)\src\Resources` to `$(BUILD_DIR)\Resources`:

```xml
<ItemGroup>
  <ContentWithTargetPath Include="src\Resources\**">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    <TargetPath>Resources\%(RecursiveDir)%(Filename)%(Extension)</TargetPath>
  </ContentWithTargetPath>
</ItemGroup>
```

## Troubleshooting

TODO: populate this section

More info: https://docs.microsoft.com/en-us/troubleshoot/developer/webapps/aspnetcore/practice-troubleshoot-linux

### Trust HTTPS certificate on Linux

https://docs.microsoft.com/en-us/aspnet/core/security/enforcing-ssl#trust-https-certificate-on-linux

### Obtain data from an app

If an app is capable of responding to requests, you can obtain the following data from the app using middleware:

- Request: Method, scheme, host, pathbase, path, query string, headers
- Connection: Remote IP address, remote port, local IP address, local port, client certificate
- Identity: Name, display name
- Configuration settings
- Environment variables

Place the following middleware code at the beginning of the `Startup.Configure` method's request processing pipeline. The environment is checked before the middleware is run to ensure that the code is only executed in the Development environment.

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, 
    IConfiguration config)
{
    if (env.IsDevelopment())
    {
        app.Run(async (context) =>
        {
            var sb = new StringBuilder();
            var nl = System.Environment.NewLine;
            var rule = string.Concat(nl, new string('-', 40), nl);
            var authSchemeProvider = app.ApplicationServices
                .GetRequiredService<IAuthenticationSchemeProvider>();

            sb.Append($"Request{rule}");
            sb.Append($"{DateTimeOffset.Now}{nl}");
            sb.Append($"{context.Request.Method} {context.Request.Path}{nl}");
            sb.Append($"Scheme: {context.Request.Scheme}{nl}");
            sb.Append($"Host: {context.Request.Headers["Host"]}{nl}");
            sb.Append($"PathBase: {context.Request.PathBase.Value}{nl}");
            sb.Append($"Path: {context.Request.Path.Value}{nl}");
            sb.Append($"Query: {context.Request.QueryString.Value}{nl}{nl}");

            sb.Append($"Connection{rule}");
            sb.Append($"RemoteIp: {context.Connection.RemoteIpAddress}{nl}");
            sb.Append($"RemotePort: {context.Connection.RemotePort}{nl}");
            sb.Append($"LocalIp: {context.Connection.LocalIpAddress}{nl}");
            sb.Append($"LocalPort: {context.Connection.LocalPort}{nl}");
            sb.Append($"ClientCert: {context.Connection.ClientCertificate}{nl}{nl}");

            sb.Append($"Identity{rule}");
            sb.Append($"User: {context.User.Identity.Name}{nl}");
            var scheme = await authSchemeProvider
                .GetSchemeAsync(IISDefaults.AuthenticationScheme);
            sb.Append($"DisplayName: {scheme?.DisplayName}{nl}{nl}");

            sb.Append($"Headers{rule}");
            foreach (var header in context.Request.Headers)
            {
                sb.Append($"{header.Key}: {header.Value}{nl}");
            }
            sb.Append(nl);

            sb.Append($"Websockets{rule}");
            if (context.Features.Get<IHttpUpgradeFeature>() != null)
            {
                sb.Append($"Status: Enabled{nl}{nl}");
            }
            else
            {
                sb.Append($"Status: Disabled{nl}{nl}");
            }

            sb.Append($"Configuration{rule}");
            foreach (var pair in config.AsEnumerable())
            {
                sb.Append($"{pair.Path}: {pair.Value}{nl}");
            }
            sb.Append(nl);

            sb.Append($"Environment Variables{rule}");
            var vars = System.Environment.GetEnvironmentVariables();
            foreach (var key in vars.Keys.Cast<string>().OrderBy(key => key, 
                StringComparer.OrdinalIgnoreCase))
            {
                var value = vars[key];
                sb.Append($"{key}: {value}{nl}");
            }

            context.Response.ContentType = "text/plain";
            await context.Response.WriteAsync(sb.ToString());
        });
    }
}
```
