---
layout: post
title: Integration tests .Net Core
categories: [testing, integration]
tags: [.NET]
comments: true
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
  - It also seems could be an option for the .NET 6 issue regarding the multiple appsettings during the publish.


Questions that I've:

- How the test host or mvc.testing host the application?
- In my current projects/executions I see that in sometimes the Startup are loaded 2x
  - Is this something expected or it is a bad implementation from my side?
- Does this way influence somehow the API behavior ?
- There are any project using it on a CI/CD environment?
- About the AppSettings.Json, today it is necessary to mirror the API settings in order to have the API working properly
  - Again is this the expected behavior or only a bad implementation from my side?

## Scenarios
## Tools