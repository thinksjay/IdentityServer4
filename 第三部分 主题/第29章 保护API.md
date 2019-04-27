# 第29章 保护API
IdentityServer 默认以[JWT](https://tools.ietf.org/html/rfc7519)（JSON Web令牌）格式发出访问令牌。

今天的每个相关平台都支持验证JWT令牌，[这里](https://jwt.io/)可以找到一个很好的JWT库列表。热门库例如：

* ASP.NET Core的[JWT bearer authentication handler](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.JwtBearer/)
* Katana的[JWT bearer authentication middleware](https://www.nuget.org/packages/Microsoft.Owin.Security.Jwt)
* Katana的[IdentityServer authentication middleware](https://identityserver.github.io/Documentation/docsv2/consuming/overview.html)
* NodeJS的[jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken) 

保护基于ASP.NET Core的API只需在DI中配置JWT承载认证处理程序，并将认证中间件添加到管道：

``` C#
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMvc();

        services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
            .AddJwtBearer(options =>
            {
                // base-address of your identityserver
                options.Authority = "https://demo.identityserver.io";

                // name of the API resource
                options.Audience = "api1";
            });
    }

    public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
    {
        app.UseAuthentication();
        app.UseMvc();
    }
}
```  

## 29.1 IdentityServer身份验证处理程序
我们的身份验证处理程序与上述处理程序的用途相同（实际上它在内部使用Microsoft JWT库），但添加了一些其他功能：

* 支持JWT和参考令牌
* 用于引用标记的可扩展缓存
* 统一配置模型
* 范围验证  


对于最简单的情况，我们的处理程序配置看起来非常类似于上面的代码段：

``` C#
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMvc();

        services.AddAuthentication(IdentityServerAuthenticationDefaults.AuthenticationScheme)
            .AddIdentityServerAuthentication(options =>
            {
                // base-address of your identityserver
                options.Authority = "https://demo.identityserver.io";

                // name of the API resource
                options.ApiName = "api1";
            });
    }

    public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
    {
        app.UseAuthentication();
        app.UseMvc();
    }
}
```   

你可以从[nuget](https://www.nuget.org/packages/IdentityServer4.AccessTokenValidation/)或[github](https://github.com/IdentityServer/IdentityServer4.AccessTokenValidation)获得包。

## 29.2 支持引用标记
如果传入令牌不是JWT，我们的中间件将联系发现文档中的内省端点以验证令牌。由于内省端点需要身份验证，因此您需要提供已配置的API密钥，例如：

``` C#
.AddIdentityServerAuthentication(options =>
{
    // base-address of your identityserver
    options.Authority = "https://demo.identityserver.io";

    // name of the API resource
    options.ApiName = "api1";
    options.ApiSecret = "secret";
})
```   

通常，您不希望为每个传入请求执行到内省端点的往返。中间件有一个内置缓存，您可以像这样启用：

``` C#
.AddIdentityServerAuthentication(options =>
{
    // base-address of your identityserver
    options.Authority = "https://demo.identityserver.io";

    // name of the API resource
    options.ApiName = "api1";
    options.ApiSecret = "secret";

    options.EnableCaching = true;
    options.CacheDuration = TimeSpan.FromMinutes(10); // that's the default
})
```

处理程序将使用在DI容器中注册的任何*IDistributedCache*实现（例如标准的*MemoryDistributedCache*）。

## 29.3 验证范围
所述*ApiName*属性检查该令牌具有匹配观众（或短`aud`），如权利要求。

在IdentityServer中，您还可以将API细分为多个范围。如果需要该粒度，可以使用ASP.NET Core授权策略系统来检查范围。

### 29.3.1 制定全球政策： 

``` C#
services
    .AddMvcCore(options =>
    {
        // require scope1 or scope2
        var policy = ScopePolicy.Create("scope1", "scope2");
        options.Filters.Add(new AuthorizeFilter(policy));
    })
    .AddJsonFormatters()
    .AddAuthorization();
```  

### 29.3.2 制定范围政策：
``` C#
services.AddAuthorization(options =>
{
    options.AddPolicy("myPolicy", builder =>
    {
        // require scope1
        builder.RequireScope("scope1");
        // and require scope2 or scope3
        builder.RequireScope("scope2", "scope3");
    });
});
```