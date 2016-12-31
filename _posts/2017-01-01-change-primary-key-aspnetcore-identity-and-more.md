---
layout: post
title: Change primary key for ASP.NET Core Identity and more
---

By default ASP.NET Core Identity use a string value for the primary keys. In [SimplCommerce](https://github.com/simplcommerce/SimplCommerce) we decided to use a long value instead.
We also changed the name of identity tables and run some custom code every time a user login into the system. In this blog post I am going to show you how we did it.

First, when creating new ASP.NET Core Web Application, in the ASP.NET Core template selecting dialog, choose "Web Application" then click on the "Change Authentication" button, then choose "Individual User Accounts"

![Create ASP.NET Core Project](/images/creating-aspnetcore-project.png "Create ASP.NET Core Project")

## The model
After the project has been created. Open the ApplicationUser.cs in the "Model" folder, you will see that it inherit from IdentityUser which then inherit from IdentityUser<string>. So we need to change that 

```cs
public class ApplicationUser : IdentityUser<long, IdentityUserClaim<long>, ApplicationUserRole, IdentityUserLogin<long>>
{
}
```

We also need create our new ApplicationUserRole class

```cs
public class ApplicationUserRole : IdentityUserRole<long>
{
}
```
And the the ApplicationRole class

```cs
public class ApplicationUserRole : IdentityRole<long, ApplicationUserRole, IdentityRoleClaim<long>>
{
}
```
Actually you can name these classes whatever name you like. It doesn't matter

## The DbContext
We need to modify a little bit the inheritance of ApplicationDbContext. Instead of inherit from IdentityDbContext<ApplicationUser>, it should be

```cs
 public class ApplicationDbContext : IdentityDbContext<ApplicationUser, ApplicationRole, long, IdentityUserClaim<long>, ApplicationUserRole, IdentityUserLogin<long>, IdentityRoleClaim<long>, IdentityUserToken<long>>
 
```

## UserStore and RoleStore
Create our custom UserStore and RoleStore

```cs
public class ApplicationUserStore : UserStore<ApplicationUser, ApplicationRole, ApplicationDbContext, long, IdentityUserClaim<long>, ApplicationUserRole,
    IdentityUserLogin<long>, IdentityUserToken<long>>
{
    public ApplicationUserStore(ApplicationDbContext context) : base(context)
    {
    }

    protected override IdentityUserClaim<long> CreateUserClaim(ApplicationUser user, Claim claim)
    {
        var userClaim = new IdentityUserClaim<long> { UserId = user.Id };
        userClaim.InitializeFromClaim(claim);
        return userClaim;
    }

    protected override IdentityUserLogin<long> CreateUserLogin(ApplicationUser user, UserLoginInfo login)
    {
        return new IdentityUserLogin<long>
        {
            UserId = user.Id,
            ProviderKey = login.ProviderKey,
            LoginProvider = login.LoginProvider,
            ProviderDisplayName = login.ProviderDisplayName
        };
    }

    protected override ApplicationUserRole CreateUserRole(ApplicationUser user, ApplicationRole role)
    {
        return new ApplicationUserRole()
        {
            UserId = user.Id,
            RoleId = role.Id
        };
    }

    protected override IdentityUserToken<long> CreateUserToken(ApplicationUser user, string loginProvider, string name, string value)
    {
        return new IdentityUserToken<long>
        {
            UserId = user.Id,
            LoginProvider = loginProvider,
            Name = name,
            Value = value
        };
    }
}
```

```cs
public class ApplicationRoleStore : RoleStore<ApplicationRole, ApplicationDbContext, long, ApplicationUserRole, IdentityRoleClaim<long>>
{
    public ApplicationRoleStore(ApplicationDbContext context) : base(context)
    {
    }

    protected override IdentityRoleClaim<long> CreateRoleClaim(ApplicationRole role, Claim claim)
    {
        return new IdentityRoleClaim<long> { RoleId = role.Id, ClaimType = claim.Type, ClaimValue = claim.Value };
    }
}
```

## The Startup.cs

Change adding the EntityFrameworkStores to our ApplicationUserStore and ApplicationRoleStore that we just have created

```cs
services.AddIdentity<ApplicationUser, ApplicationRole>()
    .AddRoleStore<ApplicationRoleStore>()
    .AddUserStore<ApplicationUserStore>()
    .AddDefaultTokenProviders();
```
## Migrations
We will need to re-create entity framework migration classes. Delete add the file in folder "Data\Migrations" then go to the Package Manager Console type `Add-Migration CreateIdentitySchema`

## Bonus

If you like to change the name of the Indentity tables, go to the OnModelCreating method of ApplicationDbContext and add custom mapping like

```cs
modelBuilder.Entity<ApplicationUser>()
    .ToTable("Core_User");
```

In some business cases, you might also need to run some code right after the users successfully login . For example my case in [SimplCommerce](https://github.com/simplcommerce/SimplCommerce). We need to migrate the shopping cart of the not login user (guest) to their real user when they loginned. We extended the SignInManager and broadcast a message every time a user login successfully

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

## The source code

The source code of this post can be found here [https://github.com/thiennn/AspnetCoreIdentityLong](https://github.com/thiennn/AspnetCoreIdentityLong)

Thanks for reading
