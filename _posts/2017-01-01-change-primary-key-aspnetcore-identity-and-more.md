---
layout: post
title: Change primary key for ASP.NET Core Identity and more
---

<div class="alert alert-warning" role="alert">
 Updated on 4-Jan-2017 to much more simpler
</div>

By default ASP.NET Core Identity use a string value for the primary keys. In [SimplCommerce](https://github.com/simplcommerce/SimplCommerce) we decided to use a long value instead.
We also changed the name of identity tables and run some custom code every time a user login into the system. In this blog post I am going to show you how we did it.

First, when creating new ASP.NET Core Web Application, in the ASP.NET Core template selecting dialog, choose "Web Application", then click on the "Change Authentication" button, and choose "Individual User Accounts"

![Create ASP.NET Core Project](/images/creating-aspnetcore-project.png "Create ASP.NET Core Project")

## The model
After the project has been created. Open the ApplicationUser.cs in the "Model" folder, you will see that it inherit from IdentityUser which then inherit from IdentityUser\<string>. So we need to change that 

```cs
public class ApplicationUser : IdentityUser<long>
{
}
```

## The DbContext
We need to modify a little bit the inheritance of ApplicationDbContext. Instead of inheriting from IdentityDbContext\<ApplicationUser>, it should be

```cs
 public class ApplicationDbContext : IdentityDbContext<ApplicationUser, IdentityRole<long>, long>
 
```

## The Startup.cs

Telling ASP.NET Core Identity that we are using long, not string

```cs
services.AddIdentity<ApplicationUser, IdentityRole<long>>()
    .AddEntityFrameworkStores<ApplicationDbContext, long>()
    .AddDefaultTokenProviders();
```

## The Migrations

We will need to re-create entity framework migration classes. Delete add the file in folder "Data\Migrations" then go to the Package Manager Console type `Add-Migration CreateIdentitySchema`

## Bonus

If you would like to change the name of the Indentity tables, go to the OnModelCreating method of ApplicationDbContext and add custom mapping like

```cs
modelBuilder.Entity<ApplicationUser>()
    .ToTable("Core_User");
```

In some business cases, you might also need to run some code right after users successfully login . For example, my case in [SimplCommerce](https://github.com/simplcommerce/SimplCommerce). We need to migrate the shopping cart of the not login user (guest) to their real user when they loginned. We extended the SignInManager and broadcast a message every time a user login successfully

```cs
namespace SimplCommerce.Module.Core.Extensions
{
    public class SimplSignInManager<TUser> : SignInManager<TUser> where TUser : class
    {
        private readonly IMediator _mediator;

        public SimplSignInManager(UserManager<TUser> userManager,
            IHttpContextAccessor contextAccessor,
            IUserClaimsPrincipalFactory<TUser> claimsFactory,
            IOptions<IdentityOptions> optionsAccessor,
            ILogger<SignInManager<TUser>> logger,
            IMediator mediator)
            : base(userManager, contextAccessor, claimsFactory, optionsAccessor, logger)
        {
            _mediator = mediator;
        }

        public override async Task SignInAsync(TUser user, bool isPersistent, string authenticationMethod = null)
        {
            var userId = await UserManager.GetUserIdAsync(user);
            _mediator.Publish(new UserSignedIn {UserId = long.Parse(userId)});
            await base.SignInAsync(user, isPersistent, authenticationMethod);
        }
    }
}
```

Then register the SimplSignInManager in ConfigureServices method of Startup.cs

```cs
services.AddScoped<SignInManager<User>, SimplSignInManager<User>>();
```

## The source code

The source code of this post can be found here [https://github.com/thiennn/AspnetCoreIdentityLong](https://github.com/thiennn/AspnetCoreIdentityLong)

Thanks for reading
