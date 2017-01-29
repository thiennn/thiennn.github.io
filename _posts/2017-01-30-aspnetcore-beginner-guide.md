---
layout: post
title: ASP.NET Core - Beginner guide
---

In this post I assume that you are .NET developers. You have already known ASP.NET Webform or MVC and you want to learn ASP.NET Core.

Although you might encounter some issues during the development because the tooling is not stable, .NET Core and ASP.NET Core definitely are the future of .NET. I believe that after the tooling reach RTM and with the release of .NET Core 2.0 and .NET Standard 2.0 (expected in Spring 2017), the adoption rate of .NET Core will increase significantly.

# ASP.NET Core in nutshell

![ASP.NET Core](/images/aspnetcore.png "ASP.NET Core")

ASP.NET Core is very different from the ASP.NET that you have already known. For example: there is no System.Web.dll, HTTP Modules, HTTP Handlers, Global.asax or Web.config. ASP.NET Core is a significant redesign of ASP.NET.

The most important ASP.NET Core principle I think you have to remember is the modularity. ASP.NET Core is a composition of granular NuGet packages and everything is excluded by default. That means you might need to add some packages when you need some features. And by this way you only take the minimal set of packages you need.

# 1. What I need to install on my PC

An IDE. In theory, you can use any editor to code .NET Core. However, I would recommend you to use Visual Studio. It's the best IDE to code .NET. My second recommendation is Visual Studio Code. Some of my friends use Visual Studio for Mac, but I haven't. 

.NET Core SDK or NET Core tools for Visual Studio if you use Visual Studio [Download here](https://www.microsoft.com/net/download/core). 

# 2. Create an ASP.NET Core project

Create an asp.net core project and take a quick look at the project structure and generated code.

- [By Visual Studio](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/start-mvc)
- [By Command Line](https://docs.microsoft.com/en-us/aspnet/core/getting-started)

# 3. Learn core concepts

- [.NET Core fundamentals](https://docs.microsoft.com/en-us/dotnet/articles/core/index)

- [ASP.NET Core fundamentals](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/index)

- [Middleware](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware)

- [Configuration](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration)

- [Dependency Injection In ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection)

- [TagHelper](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro)

# 4. Follow tutorials

 It is also a good idea to getting started by follow tutorials
 
- [Getting started with ASP.NET Core and Entity Framework Core using Visual Studio](https://docs.microsoft.com/en-us/aspnet/core/data/ef-mvc/)

- [Some other tutorials from Microsoft](https://docs.microsoft.com/en-us/aspnet/core/tutorials/)

- [Free video courses](https://www.asp.net/freecourses)

# 5. More important concepts

- [.NET Standard](https://docs.microsoft.com/en-us/dotnet/articles/standard/library)

- [.NET Standard series of Immo Landwerth on youtube](https://www.youtube.com/watch?v=YI4MurjfMn8&list=PLRAdsfhKI4OWx321A_pr-7HhRNk7wOLLY)

- [NET Core command-line interface tools](https://docs.microsoft.com/en-us/dotnet/articles/core/tools/)

# 6. Reading code samples

- [Entropy](https://github.com/aspnet/Entropy/tree/dev/samples)

- [Music store](https://github.com/aspnet/MusicStore)

- [SimplCommerce, of course ;)](https://github.com/simplcommerce/SimplCommerce)

# Further reading

- [.NET Homepage](http://dot.net)

- [.NET Core Doc](https://docs.microsoft.com/en-us/dotnet/articles/core/)

- [ASP.NET Core Doc](https://docs.microsoft.com/en-us/aspnet/core/)

- [Channel 9](https://channel9.msdn.com) and search for ASP.NET Core

- [Awesome .NET Core - A collection of awesome .NET core libraries, tools, frameworks and software](https://github.com/thangchung/awesome-dotnet-core/)

- And googling, of course.