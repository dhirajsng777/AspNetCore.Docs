---
title: What's new in ASP.NET Core 3.0
author: rick-anderson
description: Learn about the new features in ASP.NET Core 3.0.
ms.author: riande
ms.custom: mvc
ms.date: 09/19/2019
uid: aspnetcore-3.0
---
# What's new in ASP.NET Core 3.0

This article highlights the most significant changes in ASP.NET Core 3.0 with links to relevant documentation.

## Use the ASP.NET Core shared framework

The ASP.NET Core 3.0 shared framework, contained in the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app), no longer requires an explicit `<PackageReference />` element in the project file. The shared framework is automatically referenced when using the `Microsoft.NET.Sdk.Web` SDK in your project file:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

## Assemblies removed from the ASP.NET Core shared framework

The most notable assemblies removed from the ASP.NET Core 3.0 shared framework are:

* [Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json/) (Json.NET). To add Json.NET to ASP.NET Core 3.0, see [Add Newtonsoft.Json-based JSON format support](xref:web-api/advanced/formatting#add-newtonsoftjson-based-json-format-support). ASP.NET Core 3.0 introduces `System.Text.Json` for reading and writing JSON. For more information, see [New JSON serialization](#json) in this document.
* [Entity Framework Core](/ef/core/)

For a complete list of assemblies removed from the shared framework, see [Assemblies being removed from Microsoft.AspNetCore.App 3.0](https://github.com/aspnet/AspNetCore/issues/3755). For more information on the motivation for this change, see [Breaking changes to Microsoft.AspNetCore.App in 3.0](https://github.com/aspnet/Announcements/issues/325) and [A first look at changes coming in ASP.NET Core 3.0](https://devblogs.microsoft.com/aspnet/a-first-look-at-changes-coming-in-asp-net-core-3-0/).

<a name="json"></a>

### New JSON serialization

ASP.NET Core 3.0 now uses `System.Text.Json` by default for JSON serialization:

* Reads and writes JSON asynchronously.
* Is optimized for UTF-8 text.
* Typically higher performance than `Newtonsoft.Json`.

To add Json.NET to ASP.NET Core 3.0, see [Add Newtonsoft.Json-based JSON format support](xref:web-api/advanced/formatting#add-newtonsoftjson-based-json-format-support).

## gRPC

[gRPC](https://grpc.io/):

* Is a popular, high-performance RPC (remote procedure call) framework.
* Offers an opinionated contract-first approach to API development.
* Uses modern technologies such as:

  * HTTP/2 for transport.
  * Protocol Buffers as the interface description language.
  * Binary serialization format.
* Provides features such as:

  * Authentication
  * Bidirectional streaming and flow control.
  * Cancellation and timeouts.

gRPC functionality in ASP.NET Core 3.0 includes:

* [Grpc.AspNetCore](https://www.nuget.org/packages/Grpc.AspNetCore) &ndash; An ASP.NET Core framework for hosting gRPC services. gRPC on ASP.NET Core integrates with standard ASP.NET Core features like logging, dependency injection (DI), authentication and authorization.
* [Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client) &ndash; A gRPC client for .NET Core that builds upon the familiar `HttpClient`.
* [Grpc.Net.ClientFactory](https://www.nuget.org/packages/Grpc.Net.ClientFactory) &ndash; gRPC client integration with `HttpClientFactory`.

For more information, see <xref:grpc/index>.

## New Razor features

The following list contains new Razor features:

* [@attribute](xref:mvc/views/razor#attribute) &ndash; The `@attribute` directive applies the given attribute to the class of the generated page or view. For example, `@attribute [Authorize]`.
* [@implements](xref:mvc/views/razor#implements) &ndash; The `@implements` directive implements an interface for the generated class. For example, `@implements IDisposable`.

## Certificate and Kerberos authentication

Certificate authentication requires:

* Configuring the server to accept certificates.
* Adding the authentication middleware in `Startup.Configure`.
* Adding the certificate authentication service in `Startup.ConfigureServices`.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(
        CertificateAuthenticationDefaults.AuthenticationScheme)
            .AddCertificate();
    // Other service configuration removed.
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseAuthentication();
    // Other app configuration removed.
}
```

Options for certificate authentication include the ability to:

* Accept self-signed certificates.
* Check for certificate revocation.
* Check that the proffered certificate has the right usage flags in it.

A default user principal is constructed from the certificate properties. The user principal contains an event that enables supplementing or replacing the principal. For more information, see <xref:security/authentication/certauth>.

[Windows Authentication](/windows-server/security/windows-authentication/windows-authentication-overview) has been extended onto Linux and macOS. In previous versions, Windows Authentication was limited to [IIS](xref:host-and-deploy/iis/index) and [HttpSys](xref:fundamentals/servers/httpsys). In ASP.NET Core 3.0, [Kestrel](xref:fundamentals/servers/kestrel) has the ability to use Negotiate, [Kerberos](/windows-server/security/kerberos/kerberos-authentication-overview), and [NTLM on Windows](/windows-server/security/kerberos/ntlm-overview), Linux, and macOS for Windows domain-joined hosts. Kestrel support of these authentication schemes is provided by the [Microsoft.AspNetCore.Authentication.Negotiate NuGet](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Negotiate) package. As with the other authentication services, configure authentication app wide, then configure the service:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(NegotiateDefaults.AuthenticationScheme)
        .AddNegotiate();
    // Other service configuration removed.
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseAuthentication();
    // Other app configuration removed.
}
```

Host requirements:

* Windows hosts must have [Service Principal Names](/windows/win32/ad/service-principal-names) (SPNs) added to the user account hosting the app.
* Linux and macOS machines must be joined to the domain.

  * SPNs must be created for the web process.
  * [Keytab files](https://blogs.technet.microsoft.com/pie/2018/01/03/all-you-need-to-know-about-keytab-files/) must be generated and configured on the host machine.

See <xref:security/authentication/windowsauth> for more information.

## Template changes

The web UI templates (Razor Pages, MVC with controller and views) have the following removed:

* The cookie consent UI is no longer included. To enable the cookie consent feature in an ASP.NET Core 3.0 template generated app, see <xref:security/gdpr>.
* Scripts and related static assets are now referenced as local files instead of using CDNs. For more information, see [this GitHub issue](https://github.com/aspnet/AspNetCore.Docs/issues/14350).

The Angular template updated to use Angular 8.

The Razor class library (RCL) template defaults to Razor component development by default. A new template option in Visual Studio provides template support for pages and views. When creating an RCL from the template in a command shell, pass the `-support-pages-and-views` option (`dotnet new razorclasslib -support-pages-and-views`).

## Blazor

Blazor is a new framework in ASP.NET Core for building interactive client-side web UI with .NET:

* Create rich interactive UIs using C# instead of JavaScript.
* Share server-side and client-side app logic written in .NET.
* Render the UI as HTML and CSS for wide browser support, including mobile browsers.

Using .NET for client-side web development offers the following advantages:

* Write code in C# instead of JavaScript.
* Leverage the existing .NET ecosystem of .NET libraries.
* Share app logic across server and client.
* Benefit from .NET's performance, reliability, and security.
* Stay productive with Visual Studio on Windows, Linux, and macOS.
* Build on a common set of languages, frameworks, and tools that are stable, feature-rich, and easy to use.

Blazor framework supported scenarios:

* Reusable UI components (Razor components)
* Client-side routing
* Component layouts
* Support for dependency injection
* Forms and validation
* Build component libraries with Razor class libraries
* JavaScript interop

For more information, see <xref:blazor/index>.

### Blazor Server

Blazor decouples component rendering logic from how UI updates are applied. Blazor Server provides support for hosting Razor components on the server in an ASP.NET Core app. UI updates are handled over a SignalR connection. Blazor Server is supported in the ASP.NET Core 3.0 release.

### Blazor WebAssembly

Blazor WebAssembly is a single-page app framework for building interactive client-side web apps with .NET. Blazor WebAssembly uses open web standards without plugins or code transpilation and works in all modern web browsers, including mobile browsers. Blazor WebAssembly is in preview.

### Razor components

Razor components are self-contained chunks of user interface (UI), such as a page, dialog, or form. Razor components are normal .NET classes that define UI rendering logic and client-side event handlers. You can create rich interactive web apps without JavaScript.

Razor components are typically authored using Razor syntax, a natural blend of HTML and C#. Razor components are similar to Razor Pages and MVC views in that they both use Razor. Unlike pages and views, which are based on a request-response model, components are used specifically for handling UI composition.

## SignalR

See [Update SignalR code](xref:migration/22-to-30#update-signalr-code) for migration instructions. [Switch to Newtonsoft.Json](xref:migration/22-to-30#switch-to-newtonsoftjson) has instructions for migrating to `System.Text.Json`.

In the JavaScript and .NET Clients for SignalR, support was added for automatic reconnection. By default, the client tries to reconnect immediately and retry after 2, 10, and 30 seconds if necessary. If the client successfully reconnects, it receives a new connection ID. Automatic reconnect is opt-in:

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chatHub")
    .withAutomaticReconnect()
    .build();
```

The reconnection intervals can be specified by passing an array of millisecond-based durations:

```javascript
.withAutomaticReconnect([0, 3000, 5000, 10000, 15000, 30000])
//.withAutomaticReconnect([0, 2000, 10000, 30000]) The default intervals.
```

A custom implementation can be passed in for full control of the reconnection intervals.

If the reconnection fails after the last reconnect interval:

* The client considers the connection is offline.
* The client stops trying to reconnect.

During reconnection attempts, update the app UI to notify the user that the reconnection is being attempted.

To provide UI feedback when the connection is interrupted, the SignalR client API has been expanded to include the following event handlers:

* `onreconnecting`:  Gives developers an opportunity to disable UI or to let users know the app is offline.
* `onreconnected`: Gives developers an opportunity to update the UI once the connection is reestablished.

The following code uses `onreconnecting` to update the UI while trying to connect:

```javascript
connection.onreconnecting((error) => {
    const status = `Connection lost due to error "${error}". Reconnecting.`;
    document.getElementById("messageInput").disabled = true;
    document.getElementById("sendButton").disabled = true;
    document.getElementById("connectionStatus").innerText = status;
});
```

The following code uses `onreconnected` to update the UI on connection:

```javascript
connection.onreconnected((connectionId) => {
    const status = `Connection reestablished. Connected.`;
    document.getElementById("messageInput").disabled = false;
    document.getElementById("sendButton").disabled = false;
    document.getElementById("connectionStatus").innerText = status;
});
```

SignalR 3.0 and later provides a custom resource to authorization handlers when a hub method requires authorization. The resource is an instance of `HubInvocationContext`. The `HubInvocationContext` includes the:

* `HubCallerContext`
* Name of the hub method being invoked.
* Arguments to the hub method.

Consider the following example of a chat room app allowing multiple organization sign-in via Azure Active Directory. Anyone with a Microsoft account can sign in to chat, but only members of the owning organization can ban users or view users’ chat histories. The app could restrict certain functionality from specific users.

```csharp
public class DomainRestrictedRequirement :
    AuthorizationHandler<DomainRestrictedRequirement, HubInvocationContext>,
    IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context,
        DomainRestrictedRequirement requirement,
        HubInvocationContext resource)
    {
        if (context.User?.Identity?.Name == null)
        {
            return Task.CompletedTask;
        }

        if (IsUserAllowedToDoThis(resource.HubMethodName, context.User.Identity.Name))
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }

    private bool IsUserAllowedToDoThis(string hubMethodName, string currentUsername)
    {
        if (hubMethodName.Equals("banUser", StringComparison.OrdinalIgnoreCase))
        {
            return currentUsername.Equals("bob42@jabbr.net", StringComparison.OrdinalIgnoreCase);
        }

        return currentUsername.EndsWith("@jabbr.net", StringComparison.OrdinalIgnoreCase));
    }
}
```

In the preceding code, `DomainRestrictedRequirement` serves as a custom `IAuthorizationRequirement`. Because the `HubInvocationContext` resource parameter is being passed in, the internal logic can:

* Inspect the context in which the Hub is being called.
* Make decisions on allowing the user to execute individual Hub methods.

Individual Hub methods can be decorated with the name of the policy the code checks at run-time. As clients attempt to call individual Hub methods, the `DomainRestrictedRequirement` handler runs and controls access to the methods. Based on the way the `DomainRestrictedRequirement` controls access:

* All logged-in users can call the `SendMessage` method.
* Only users who have logged in with a `@jabbr.net` email address can view users’ histories.
* Only `bob42@jabbr.net` can ban users from the chat room.

```csharp
[Authorize]
public class ChatHub : Hub
{
    public void SendMessage(string message)
    {
    }

    [Authorize("DomainRestricted")]
    public void BanUser(string username)
    {
    }

    [Authorize("DomainRestricted")]
    public void ViewUserHistory(string username)
    {
    }
}
```

Creating the `DomainRestricted` policy might involve:

* In *Startup.cs*, adding the new policy.
* Provide the custom `DomainRestrictedRequirement` requirement as a parameter.
* Registering `DomainRestricted` with the authorization middleware.

```csharp
services
    .AddAuthorization(options =>
    {
        options.AddPolicy("DomainRestricted", policy =>
        {
            policy.Requirements.Add(new DomainRestrictedRequirement());
        });
    });
```

SignalR hubs use [Endpoint Routing](xref:fundamentals/routing). SignalR hub connection was previously done explicitly:

```csharp
app.UseSignalR(routes =>
{
    routes.MapHub<ChatHub>("hubs/chat");
});
```

In the previous version, developers needed to wire up controllers, Razor pages, and hubs in a variety of different places. Explicit connection results in a series of nearly-identical routing segments:

```csharp
app.UseSignalR(routes =>
{
    routes.MapHub<ChatHub>("hubs/chat");
});

app.UseRouting(routes =>
{
    routes.MapRazorPages();
});
```

SignalR 3.0 hubs can be routed via endpoint routing. With endpoint routing, typically all routing can be configured in `UseRouting`:

```csharp
app.UseRouting(routes =>
{
    routes.MapRazorPages();
    routes.MapHub<ChatHub>("hubs/chat");
});
```

ASP.NET Core 3.0 SignalR added:

Client-to-server streaming. With client-to-server streaming, server-side methods can take instances of either an `IAsyncEnumerable<T>` or `ChannelReader<T>`. In the following C# sample, the `UploadStream` method on the Hub will receive a stream of strings from the client:

```csharp
public async Task UploadStream(IAsyncEnumerable<string> stream)
{
    await foreach (var item in stream)
    {
        // process content
    }
}
```

.NET client apps can pass either an `IAsyncEnumerable<T>` or `ChannelReader<T>` instance as the `stream` argument of the `UploadStream` Hub method above.

```csharp
async IAsyncEnumerable<string> clientStreamData()
{
    for (var i = 0; i < 5; i++)
    {
        var data = await FetchSomeData();
        yield return data;
    }
    //After the for loop has completed and the local function exits the stream completion is sent.
}

await connection.SendAsync("UploadStream", clientStreamData());
```

JavaScript client apps use the SignalR `Subject` (or an RxJS Subject) for the `stream` argument of the `UploadStream` Hub method above.

```javascript
let subject = new signalR.Subject();
await connection.send("StartStream", "MyAsciiArtStream", subject);
```

The JavaScript code could use the `subject.next` method to handle strings as they are captured and ready to be sent to the server.

```javascript
subject.next("example");
subject.complete();
```

Using code like the two preceding snippets, real-time streaming experiences can be created.

## Generic Host

The ASP.NET Core 3.0 templates use <xref:fundamentals/host/generic-host>. Previous versions used <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder>. Using the .NET Core Generic Host (<xref:Microsoft.Extensions.Hosting.HostBuilder>) provides better integration of ASP.NET Core apps with other server scenarios that are not web specific.

The following two images show the changes made to the template generated *Program.cs* file:

 ![2.2 version](aspnetcore-3.0/_static/2.2host.png)
 ![3.0 version](aspnetcore-3.0/_static/3.0host.png)

In the preceding images:

* The ASP.NET Core 2.2 version is shown first. The red boxes indicate code that has been removed in the 3.0 version.
* The ASP.NET Core 3.0 version is shown second. The green boxes indicate code that has been added. The 3.0 version requires `using Microsoft.Extensions.Hosting;`.

## Host configuration and options

Host builder configuration is provided by <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*>:

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureHostConfiguration(configHost =>
        {
            configHost.SetBasePath(Directory.GetCurrentDirectory());
            configHost.AddJsonFile("hostsettings.json", optional: true);
            configHost.AddEnvironmentVariables(prefix: "PREFIX_");
            configHost.AddCommandLine(args);
    });
```

Prior to the release of ASP.NET Core 3.0, environment variables prefixed with `ASPNETCORE_` were loaded for host configuration of the Web Host. In 3.0, `AddEnvironmentVariables` is used to load environment variables prefixed with `DOTNET_` for host configuration with `CreateDefaultBuilder`. 

To configure host options in code, call `Configure<HostOptions>` on the host builder's services provided by `ConfigureServices`:

```csharp
public static IHostBuilder CreateHostBuilder2(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureServices((hostContext, services) =>
        {
            services.Configure(option =>
            {
                option.ShutdownTimeout = System.TimeSpan.FromSeconds(20);
            });
        });
```

As a result of this change, the Startup class now only supports *constructor injection* of the following types: `IHostEnvironment`, `IWebHostEnvironment`, and `IConfiguration`. All services can still be injected directly as arguments to the `Startup.Configure` method. For more information, see [aspnet/Announcements#353](https://github.com/aspnet/Announcements/issues/353).

For more information, see the following articles:

* <xref:fundamentals/configuration/index>
* <xref:fundamentals/host/generic-host>

## Kestrel

* Kestrel configuration has been updated for the migration to the Generic Host. In 3.0, Kestrel is configured on the web host builder provided by `ConfigureWebHostDefaults`.
* Connection Adapters have been removed from Kestrel and replaced with Connection Middleware, which is similar to HTTP Middleware in the ASP.NET Core pipeline but for lower-level connections.
* The Kestrel transport layer has been exposed as a public interface in `Connections.Abstractions`.
* Ambiguity between headers and trailers has been resolved by moving trailing headers to a new collection.
* Synchronous IO APIs, such as `HttpReqeuest.Body.Read`, are a common source of thread starvation leading to app crashes. In 3.0, `AllowSynchronousIO` is disabled by default.

For more information, see <xref:migration/22-to-30>.

## Request counters

The Hosting EventSource (Microsoft.AspNetCore.Hosting) emits the following EventCounters related to incoming requests:

* `requests-per-second`
* `total-requests`
* `current-requests`
* `failed-requests`

## Endpoint routing

See https://github.com/aspnet/AspNetCore.Docs/issues/14291

## Health Checks

Health Checks use endpoint routing with the Generic Host. In `Startup.Configure`, call `MapHealthChecks` on the endpoint builder with the endpoint URL or relative path:

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health");
});
```

Health Checks endpoints can:

* Specify one or more permitted hosts/ports.
* Require authorization.
* Require CORS.

For more information, see the following articles:

* <xref:migration/22-to-30#health-checks> <!-- ignore warning per @guardrex -->
* <xref:host-and-deploy/health-checks>

## Pipes on HttpContext

It is now possible to read the request body and write the response body using the <xref:System.IO.Pipelines> API. The <!-- <xref:Microsoft.AspNetCore.Http.HttpRequest.BodyReader> --> `HttpRequest.BodyReader` property provides a <xref:System.IO.Pipelines.PipeReader> that can be used to read the request body. The <!-- <xref:Microsoft.AspNetCore.Http.> --> `HttpResponse.BodyWriter` property provides a <xref:System.IO.Pipelines.PipeWriter> that can be used to write the response body.

<!-- indirectly related, https://github.com/dotnet/docs/pull/14414 won't be published by 9/23  -->

## Worker service and SDK

See https://github.com/aspnet/AspNetCore.Docs/issues/14269

## Forwarded Headers Middleware improvements

In previous versions, calling <xref:Microsoft.AspNetCore.Builder.HstsBuilderExtensions.UseHsts*> and  <xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection*> were problematic when deployed to an Azure Linux or behind any reverse proxy other than IIS. The fix for previous versions is documented in [Forward the scheme for Linux and non-IIS reverse proxies](xref:host-and-deploy/proxy-load-balancer#forward-the-scheme-for-linux-and-non-iis-reverse-proxies).

In ASP.NET Core 3.0, this problem has been fixed. The host enables the [Forwarded Headers Middleware](xref:host-and-deploy/proxy-load-balancer#forwarded-headers-middleware-options) when the `ASPNETCORE_FORWARDEDHEADERS_ENABLED` environment variable has been set to `true`.

## HTTP/2 enabled by default

HTTP/2 is enabled by default in Kestrel for HTTPS endpoints. HTTP/2 support when using IIS or HttpSysServer is enabled, when supported by the operating system.

## ASP.NET Core 3.0 only runs on .NET Core 3.0

As of ASP.NET Core 3.0, .NET Framework is no longer a supported target framework. Projects targeting .NET Framework can continue in a fully supported fashion using the [.NET Core 2.1 LTS release](https://www.microsoft.com/net/download/dotnet-core/2.1). Most ASP.NET Core 2.1.x related packages will be supported indefinitely, beyond the 3 year LTS period for .NET Core 2.1.

See [Port your code from .NET Framework to .NET Core](/dotnet/core/porting/) for migration information.


<!-- 
## Additional information
For the complete list of changes, see the [ASP.NET Core 3.0 Release Notes](WHERE IS THIS????).
-->