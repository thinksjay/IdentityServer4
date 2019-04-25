# 第23章 Windows身份验证
在支持的平台上，您可以使用IdentityServer使用Windows身份验证对用户进行身份验证（例如，针对Active Directory）。当前使用以下命令托管IdentityServer时，Windows身份验证可用：   

* [Kestrel](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel)在Windows上使用IIS和IIS集成包
* Windows上的[HTTP.sys](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/httpsys)服务器  

在这两种情况下，使用方案`“Windows”`在`HttpContext`上使用`ChallengeAsync` API来触发Windows身份验证。我们的[快速入门UI](https://github.com/IdentityServer/IdentityServer4.Quickstart.UI)中的帐户控制器实现了必要的逻辑。   

## 23.1 使用 Kestrel
使用Kestrel时，必须运行“behind”IIS并使用IIS集成：   
``` C#
var host = new WebHostBuilder()
    .UseKestrel()
    .UseUrls("http://localhost:5000")
    .UseContentRoot(Directory.GetCurrentDirectory())
    .UseIISIntegration()
    .UseStartup<Startup>()
    .Build();
```   

使用该`WebHost.CreateDefaultBuilder`方法设置时，会自动配置`WebHostBuilder`。   

IIS（或IIS Express）中的虚拟目录也必须启用Windows并启用匿名身份验证。   

IIS集成层将Windows身份验证处理程序配置为DI，可以通过身份验证服务调用。通常在IdentityServer中，建议禁用此自动行为。这是在`ConfigureServices`：   

``` C#
services.Configure<IISOptions>(iis =>
{
    iis.AuthenticationDisplayName = "Windows";
    iis.AutomaticAuthentication = false;
});
```  

> **注意**
默认情况下，显示名称为空，Windows身份验证按钮不会显示在快速入门UI中。如果依赖于自动发现外部提供程序，则需要设置显示名称。