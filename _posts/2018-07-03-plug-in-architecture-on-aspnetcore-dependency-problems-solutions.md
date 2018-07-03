---
layout: post
title: Plug-in architecture on ASP.NET Core - Dependency problems and solutions
---

Plug-in architecture here I mean that the application’s functionality can be extended or removed by install or uninstall plugins at runtime without the need to re-compile or re-deploy the application. Plugins are stored in a marketplace, users can find them install or uninstall them on the fly, directly in the app. This type of architecture is usually found in CMS, or ecommerce systems. We are also apply it in [SimplCommerce](https://github.com/simplcommerce/SimplCommerce)

![Plugin](/images/plugin.png "Plugin")
> Note: This picture is from the internet


Plugins can have dependencies. For example, a plugin to do payment via Stripe depends on Stripe SDK.  With normal ASP.NET Core application it is not a problem, but with this plug-in architecture, it is the big challenge

First, in order to make sure that it is safe to remove a plugin, avoiding DLL hell problem, we put all the dependencies of a plugin and the plugin itself into a separate folder. When a plugin is no longer need, we can simply delete its folder. Checkout the article [Modular Web Application with ASP.NET Core](https://www.codeproject.com/Articles/1109475/Modular-Web-Application-with-ASP-NET-Core)  to see how we archive this. You can think a module is a plugin.

Second, because we want to offer installing plugins at runtime, plugins cannot be added as a project reference to the main application, so they aren't loaded natively by the .NET Core runtime. We have to manually load them along with their dependencies.

In .NET Core dependencies are managed by adding package references, a referenced package then can depend on other packages and so on. They are all nuget packages. These nuget packages are not stored in our solution but in a global packageFolders. In Windows, it is C:\Users\<username>\.nuget\packages. During the complication assemblies of referenced packages are not copied to the output folder, they are tracked in the deps.json instead.

There is a flag property called 'CopyLocalLockFileAssemblies' in the project file (.csproj), when you turn it on, it will copy all the dependencies of the project during the compile time, including all the System, Microsoft.AspNetCore assemblies and they are too much.

So how to include only assemblies we need for a plugin? I have found 3 workarounds to archive this.

The first workaround: turn CopyLocalLockFileAssemblies on and write a custom MSBuild task to delete redundant assemblies such as System.* or Microsoft.AspNetCore.* assemblies. The downside of this workaround is that it is hard to track the list of assemblies need to delete, and the build time is slow.

The second workaround: make some treats in the project file (.csproj) to copy some particular assemblies we need. For example, the case of Stripe. In the project file, in the PropertyGroup section we add a property to store the Stripe version 

```xml

<StripePackageVersion>11.10.0</StripePackageVersion>
```

Then in the ItemGroup we add the following 2 lines

```xml
<PackageReference Include="Stripe.net" Version="$(StripePackageVersion)" />
<Content CopyToOutputDirectory="PreserveNewest" Include="$(NuGetPackageRoot)stripe.net\$(StripePackageVersion)\lib\netstandard1.2\*.dll" />
```

The downside of this workaround is that we must know exactly what assemblies we want to copy and this might difficult if the dependency has other dependencies.

The third workaround is adding project reference during the development time as normal ASP.NET Core application, when development complete, we do dotnet publish on the plugin project. During the publish .NET Core will travel all dependency graph and copies all the dependencies to the output folder. Fortunately, all the assemblies referenced by the Microsoft.AspNetCore.App are not included, because they are installed when you install the ASP.NET Core runtime.

The last problem we have to resolve is the dependency versions conflicts. Because plugins don’t know each other, they can be developed at different time by different teams. Therefore, some different version of common assemblies can be used. For example, a plugin A use the version 9.3.0 of WindowsAzure.Storage, another plugin B use 9.2.0 of WindowsAzure.Storage. When we install both 2 plugins will cause version conflict. The workaround for this is that before installing a plugin, all the dependencies of that plugin must be scanned to check for the potential version conflict. If there is a conflict then the system will not allow to install that plugin.

```csharp
Assembly assembly;
try
{
	assembly = AssemblyLoadContext.Default.LoadFromAssemblyPath(file.FullName);
}
catch (FileLoadException)
{
	// Get loaded assembly
	assembly = Assembly.Load(new AssemblyName(Path.GetFileNameWithoutExtension(file.Name)));

	if (assembly == null)
	{
		throw;
	}

	string loadedAssemblyVersion = FileVersionInfo.GetVersionInfo(assembly.Location).FileVersion;
	string tryToLoadAssemblyVersion = FileVersionInfo.GetVersionInfo(file.FullName).FileVersion;

	if (tryToLoadAssemblyVersion != loadedAssemblyVersion)
	{
		throw new Exception($"Cannot load {file.FullName} {tryToLoadAssemblyVersion} because {assembly.Location} {loadedAssemblyVersion} has been loaded");
	}
}
```

Plug-in architecture on ASP.NET Core is hard. But it is a critical feature of CMS, Ecommerce or SaaS applications. I do hope that Microsoft will have more support on this.
