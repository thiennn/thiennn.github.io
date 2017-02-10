---
layout: post
title: Upgraded from project.json to csproj for SimplCommerce
---

Today, we have upgraded from project.json to csproj for [SimplCommerce](https://github.com/simplcommerce/SimplCommerce). It's simple than what I have thought. In this blog post I will share you this journey.

While working with ASP.NET Core in SimplCommerce, I felt in love with project.json. It's an innovation, modern and simple. However, in order to make the tooling compatible with other .NET app models (WinForms, WPF, UWP, ASP.NET, iOS, Android, etc.), Microsoft has decided to move .NET Core projects to .csproj/MSBuild. More detail about this can be found [here](https://blogs.msdn.microsoft.com/dotnet/2016/05/23/changes-to-project-json/).

Struggling for a while, we finally decided to say goodbye to project.json.

![Goodbye project.json](/images/rip_projectjson.png "Goodbye project.json")

Microsoft doesn't support .NET Core csproj on VS 2015. So, to work with csproj, we need to install VS 2017 RC or go with command line. I chose to install VS 2017 RC :). I use the lasted version of  VS 2017 RC which included .NET Core 1.0 SDK – RC4.

After installed VS 2017 RC. I opened the SimplCommerce.sln. VS 2017 show a dialog and ask for an upgrade. Note it's an one way upgrade. Once upgraded, you cannot go back.

![Goodbye project.json](/images/rip_projectjson_migration.png "Goodbye project.json")

Took a deep breath and click OK. Whoo! sucessful, the migration report show no error, there was a warning on the solution file but it not important. All the projects are loaded.

Wait *"Error occurred while restoring NuGet packages: The operation failed as details for project SimplCommerce.WebHost could not be loaded."* appeared in the Output panel. None of the project can build because all the dependencies could not be loaded. 

Right click on the solution and then click "Restore Nuget Package" didn't work aslo the same error appeared. Tried to workaroud in VS 2017, no luck.

Then I opened command line and typed "dotnet restore", enter. wow 

*"C:\Program Files\dotnet\sdk\1.0.0-rc4-004771\NuGet.targets(97,5): error : '1.1.0-preview4-final;1.0.0-msbuild3-final' is not a valid version string. [D:\Projects\SimplCommerce\SimplCommerce.sln]"*

Let find this string. It is in the SimplCommerce.WebHost.csproj 

`<DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="1.1.0-preview4-final;1.0.0-msbuild3-final" />`

It seem not correct, let remove the "1.1.0-preview4-final;"

All the projects build sucessful. Control + F5 the application run just fine.

### Let take a look at what have changed.

 - All the file project.json, **.xproj was removed. The global.json is removed also.
 - The files project.lock.json are renamed and moved here obj\project.assets.json
 - The SimplCommerce.sln just change a little but and seem not important
 - The **csproj look simple and nice
 
So [SimplCommerce](https://github.com/simplcommerce/SimplCommerce) has been upgraded to use csproj. There are a couple of things I want to note here:

 > AppVeyor hasn't offically support .NET Core csproj. But you can request to join their beta [here](https://github.com/appveyor/ci/issues/1179)
 > Travis hasn't support also. I have submited the request [https://github.com/travis-ci/travis-ci/issues/7301](https://github.com/travis-ci/travis-ci/issues/7301)
 
### And  below are some useful references:

 - [Changes to Project.json](https://blogs.msdn.microsoft.com/dotnet/2016/05/23/changes-to-project-json/) by Scott Hunter - .NET blog
 - [Announcing .NET Core Tools Updates in VS 2017 RC](https://blogs.msdn.microsoft.com/dotnet/2017/02/07/announcing-net-core-tools-updates-in-vs-2017-rc/) by Rich Lander - .NET blog
 - [Project.json to MSBuild conversion guide](http://www.natemcmaster.com/blog/2017/01/19/project-json-to-csproj/) by Nate McMaster
 - [Part 2 - Caveats of project.json to MSBuild conversion](http://www.natemcmaster.com/blog/2017/02/01/project-json-to-csproj-part2/) by Nate McMaster
