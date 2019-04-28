# 第63章 ASP.NET Identity 支持
提供了基于ASP.NET身份的实现，用于管理IdentityServer用户的身份数据库。此实现是IdentityServer中的扩展点，以便为用户加载身份数据以将声明发送到令牌。

这个支持的仓储位于[此处](https://github.com/IdentityServer/IdentityServer4.AspNetIdentity/)，NuGet包就在[这里](https://www.nuget.org/packages/IdentityServer4.AspNetIdentity)。

要使用此库，请正常配置ASP.NET Identity。然后在调用`AddIdentityServer`后使用`AddAspNetIdentity`扩展方法：

``` C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

    services.AddIdentityServer()
        .AddAspNetIdentity<ApplicationUser>();
}
```  

`AddAspNetIdentity`需要作为通用参数，为您的用户建模ASP.NET Identity（以及传递给AddIdentityASP.NET Identity 的同一个用户）。这将配置IdentityServer使用实现`IUserClaimsPrincipalFactory`，`IResourceOwnerPasswordValidator`和`IProfileService`的ASP.NET Identity。它还配置了一些用于IdentityServer的ASP.NET Identity选项（例如要使用的声明类型和身份验证cookie设置）。