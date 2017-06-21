---
layout: post
title: ASP.NET vs ASP.NET Core
---

ASP.NET Core is the next generation of ASP.NET. Many people are learning ASP.NET Core, and most of them have already know ASP.NET. So I think it is good to have a summary of the different between them.

|ASP.NET | ASP.NET Core | 
|---- | ------------ | 
|IIS, Windows only    | Kestrel, Windows, Mac, Linux      |
|One version per machine | Multiple versions per machine  |
|System.Web. Everything is included by default   | No System.Web. Everything is Nuget packages. Excluded all by default  | 
|HTTP Modules, HTTP Handlers, Global.asax   | Middlewares        |
|MVC + Web API + Web Pages | ASP.NET MVC Core |
|Web.config | .json, .ini, environment variables, etc. |
|Child Actions (Html.Render) | View Components |
|Request validation | N/A |
|N/A | Tag Helpers|
|N/A | Build-in Dependency Injection |
|N/A | Build-in Logging API and Providers |
|N/A | Application Part |
|N/A | Dependency injection into views |
|N/A | A new way of localization with IStringLocalizer, IViewLocalizer
|N/A | File Providers |
|N/A | WebSockets| 

Please let's me know if you find out more.