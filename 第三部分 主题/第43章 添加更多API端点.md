# 第43章 添加更多API端点
您可以向托管IdentityServer4的应用程序添加更多API端点。

您通常希望通过它们所托管的IdentityServer实例来保护这些API。这不是问题。只需将令牌验证处理程序添加到主机（请参阅[此处](https://github.com/thinksjay/IdentityServer4/blob/master/%E7%AC%AC%E4%B8%89%E9%83%A8%E5%88%86%20%E4%B8%BB%E9%A2%98/%E7%AC%AC29%E7%AB%A0%20%E4%BF%9D%E6%8A%A4API.md)）：

``` C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();

    // details omitted
    services.AddIdentityServer();

    services.AddAuthentication()
        .AddIdentityServerAuthentication("token", isAuth =>
        {
            isAuth.Authority = "base_address_of_identityserver";
            isAuth.ApiName = "name_of_api";
        });
}
```   

在您的API上，您需要添加`[Authorize]`属性并显式引用您要使用的身份验证方案（在此示例中`token`，您可以选择您喜欢的任何名称）：

``` C#
public class TestController : ControllerBase
{
    [Route("test")]
    [Authorize(AuthenticationSchemes = "token")]
    public IActionResult Get()
    {
        var claims = User.Claims.Select(c => new { c.Type, c.Value }).ToArray();
        return Ok(new { message = "Hello API", claims });
    }
}
```  

如果要从浏览器调用该API，则还需要配置CORS（请参阅[此处](https://github.com/thinksjay/IdentityServer4/blob/master/%E7%AC%AC%E4%B8%89%E9%83%A8%E5%88%86%20%E4%B8%BB%E9%A2%98/%E7%AC%AC41%E7%AB%A0%20CORS.md)）。

## 43.1 发现
如果需要，您还可以将端点添加到发现文档中，例如：

``` C#
services.AddIdentityServer(options =>
{
    options.Discovery.CustomEntries.Add("custom_endpoint", "~/api/custom");
})
```