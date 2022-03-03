---
layout: post
title: Integration tests
categories: [testing, integration]
tags: [.NET]
comments: true
mermaid: true
---

- [Before it starts](#before-it-starts)
- [.NET world for integration tests](#net-world-for-integration-tests)
- [WebAPI Example](#webapi-example)
- [Console Example](#console-example)
- [Microsoft Tools/Framework](#microsoft-toolsframework)
- [BDD](#bdd)
- [CI/CD](#cicd)
- [Collecting metrics](#collecting-metrics)

---

# Before it starts

First a quick explanation regarding the Integration tests from my point of view.

It is possible to expect different things from integrated tests, it is ok to, for example, set up End to End tests that cover one path and also set up a service test that runs over the same path, but it doesn’t mean that both tests are doing the same thing, or those tests share the same goal.

E2E tests could have, for example, the goal to mimic the end-user usage in different environments using different tools to perform this. And on the other hand, an integrated service test could have the goal to test the application behavior that is owned. So it is ok to expect different outcomes from different types of tests, and the general idea is to try to clean the corners as much as possible to make sure that no bugs live there. another point on this discussion is that tests that live in a different level as a Unit, tend to have a higher cost, either to build, debug, troubleshoot, or execution time. One really good description of test types and the most known Test Pyramid can be found on Martin Fowler’s blog about [practical test pyramid](https://martinfowler.com/articles/practical-test-pyramid.html) and from there it is possible to have a picture on this matter.

So based on this assumption integration tests tend to be difficult to maintain, and I would also put on this soup the mind map level of abstraction that a developer person needs to build on the mind to understand and follow the path that are been performed during execution of an integration test. And perhaps sometimes also it’s possible to see a “Behavior abstraction” on top of that. (a.k.a: BDD).

If different types of tests could bring different results, and integration are expensive why do we still keep doing it?. As an enthusiast of Integration tests, I would try to dig into this question, showing examples using .NET and the frameworks/tools that are on the market nowadays, to not give a conclusion but to propose a discussion around the cost X benefits

I will show some possibilities to test WebAPI, Console Applications using integrated tests in order to extract as much as possible, the value from it, also trying to keep it as simple as possible for beginners but with some context regarding what happens behind the scenes once we are doing this type of tests.

Upcoming:

- .NET world for integration tests
- WebAPI Example
- Console Example
- Microsoft Tools/frameworks
- BDD

# .NET world for integration tests

_Note: On this topic I will only talk about .NET Core projects_

There's available a page on Microsoft documentation that touch this topic [Integration tests in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-6.0) and it's come with a suggestion about how to create a integration test project and how to play with it on local environment. With covers the basic steps but it is possible that an real world application can requires more than that, for example mocking solution, Startup and Tier down events, metrics. And I will try to tie those other possible scenarios on this post.

And to help me on this journey I will take advantage of a bunch of tools that I've on my bucket.


- Tool: _[FLuentAssertion](https://fluentassertions.com/)_  
  Propose: Help to make test assertions in a fluent way, and can help to catch exceptions, build assertion context to validate more than one field, and also facilitate once verifying the message error once an assertion fails

- Tool: _[Moq](https://github.com/moq/moq4)_  
  Propose: Provide a way to mock services and also verify if something happen during the test execution

- Tool: _[WireMock.Net](https://github.com/WireMock-Net/WireMock.Net)_  
  Propose: It can create an local server to mock an pre-defined API behavior, it can help to deal with external dependencies

- Tool: _[FluentDocker](https://github.com/mariotoffia/FluentDocker)_  
  Propose: It enable docker and docker-compose integration using a fluent API

- Tool: _[xUnit](https://xunit.net/)_  
  Propose: Test framework, all examples here will be using this framework here.

- Tool: _[SpecFlow](https://specflow.org/)_  
  Propose: A BDD Framework for .NET, I also will provide some insights about this

# WebAPI Example
# Console Example
# Microsoft Tools/Framework
# BDD
# CI/CD
# Collecting metrics

<!-- ## Microsoft documentation



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

-->