---
layout: post
title: Integration tests .Net Core
categories: [testing, integration]
tags: [.NET]
comments: true
mermaid: true
---

# Integration tests for .NET applications

## What it is
## Microsoft documentation

[Integration tests in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-6.0)

Source code:  
- [Microsoft.AspNetCore.MVC.Testing](https://github.com/dotnet/aspnetcore/tree/main/src/Mvc/Mvc.Testing/src)
  - [Microsoft.AspNetCore.TestHost](https://github.com/dotnet/aspnetcore/tree/main/src/Hosting/TestHost/src)


_Things that I've found looking into the code_
- **[TestServerOptions](https://github.dev/dotnet/aspnetcore/blob/1852bb78776cbcf0c6f6daaa238e4a0c525366fc/src/Hosting/TestHost/src/TestServerOptions.cs#L28)**
  - This is a configuration class that is used inside the TestHosting to determine some behavior like
    - The default URL
    - AllowSynchronousIO (I don't know what is that)
    - PreserveExecutionContext (I don't know what is that)
- **[WebApplicationFactoryContentRootAttribute](https://github.com/dotnet/aspnetcore/blob/1852bb78776cbcf0c6f6daaa238e4a0c525366fc/src/Mvc/Mvc.Testing/src/WebApplicationFactoryContentRootAttribute.cs)**
  - This is interesting, need more investigation but it seems that using this it is possible to determine with projects the Hosting should treat as the application to load
- **MvcTestingAppManifest.json**
  - Apparently this could be the configurations for the Acceptance tests, not sure how it works because I could not see any reference from it in a simple search
  - I would like to test it to see if it is possible to override the appsettings from the API without mirror it on the acceptance project
  - It also seems could be an option for the .NET 6 issue regarding the multiple appsettings during the publish. [Generate error for duplicate files in publish output](https://docs.microsoft.com/en-us/dotnet/core/compatibility/sdk/6.0/duplicate-files-in-output)
- **Environment**
  - Inside the WebApplicationFactory I could see that in many places the application has been set to `Environment.Development` so it means that this way it is not possible to test the application against the `Production` environment. I don't know yet what the implications of that
    - How about the targeting, this will affect some how maybe this changes from `Debug` to `Release` somewhere that I'm not aware about it yet
- **services.SwapTransient**
  - inside the _builder.ConfigureTestServices_ it is possible to swap services this could be useful once integrating with external dependencies
  - it is an extension method from `Microsoft.AspNetCore.TestHost`

## WebApplicationFactory Flow

How does the WebApplicationFactory works:

<div class="mermaid">
  graph LR;
    A[WebApplicationFactory] --> |EntryPoint| B(Startup or Program)
    B --> C[TestServer]
    C -.Optional.-> E(Inject services)
    C -.Optional.-> F(Configure)
    C -.Optional.-> G(Swap or remove services)
    E & F & G --> H[Run in memory server]
</div>

1) For startup the application it infer the application path based on the EntryPoint with could be the Startup or in minimal API the Program.cs   
2) There's an extension method inside `WebHostBuilderExtensions.UseTestServer` for `IWebHostBuilder`, with after determine with application should be hosted, it say to the host it is a test server 
   1) So I assume here is where everything are going to the memory
   2) I don't know yet the implication of it, regarding the target for `Debug` or `Release` I also assume that it is `Debug` but the question is does it change anything else?
   3) This extension performs an: `services.AddSingleton<IServer, TestServer>();`
3) This bootstrap process also creates an `RedirectHandler` and configure it inside the HttpClient that has been created, this is how the request for `http://localhost` are been redirected to the InMemory server


--- 

Questions that I've:

- How the test host or mvc.testing host the application?
- In my current projects/executions I see that in sometimes the Startup are loaded 2x
  - Is this something expected or it is a bad implementation from my side?
- Does this way influence somehow the API behavior ?
- There are any project using it on a CI/CD environment?
- About the AppSettings.Json, today it is necessary to mirror the API settings in order to have the API working properly
  - Again is this the expected behavior or only a bad implementation from my side?  

Solving the questions:

- _Question:_ In my current projects/executions I see that in sometimes the Startup are loaded 2x
  - Using a TestFixture directly from xUnit without SpecFlow I could not see this behavior, maybe it could be related
  - Still not solved, I could not reproduce the behavior with simple project [IntegrationTestLearning](https://github.com/afborgesDev/testingMyLearns/tree/main/integrationTestLearning/IntegrationTestLearning)
  - Maybe this could be related to IClassFixture

## Scenarios
## Tools

<script src="/assets/mermaid/mermaid.min.js"/>