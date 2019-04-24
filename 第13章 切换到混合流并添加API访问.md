# 第13章 切换到混合流并添加API访问

在之前的快速入门中，我们探讨了API访问和用户身份验证。现在我们想把这两个部分放在一起。   

OpenID Connect和OAuth 2.0组合的优点在于，您可以使用单个协议和使用令牌服务进行单次交换来实现这两者。   

在之前的快速入门中，我们使用了OpenID Connect隐式流程。在隐式流程中，所有令牌都通过浏览器传输，这对于身份令牌来说是完全正确的。现在我们还想要一个访问令牌。   

访问令牌比身份令牌更敏感，如果不需要，我们不希望将它们暴露给“外部”世界。OpenID Connect包含一个名为“混合流”的流程，它为我们提供了两全其美的优势，身份令牌通过浏览器渠道传输，因此客户端可以在进行任何更多工作之前对其进行验证。如果验证成功，客户端会打开令牌服务的反向通道以检索访问令牌。   

## 13.1 修改客户端配置
没有太多必要的修改。首先，我们希望允许客户端使用混合流，此外我们还希望客户端允许执行不在用户上下文中的服务器到服务器API调用（这与我们的客户端凭证快速启动非常相似）。这是使用该`AllowedGrantTypes`属性表示的。   

接下来我们需要添加一个客户端密钥。这将用于检索反向通道上的访问令牌。   

最后，我们还让客户端访问`offline_access`范围 - 这允许请求刷新令牌以实现长期存在的API访问：   

``` C#
new Client
{
    ClientId = "mvc",
    ClientName = "MVC Client",
    AllowedGrantTypes = GrantTypes.Hybrid,

    ClientSecrets =
    {
        new Secret("secret".Sha256())
    },

    RedirectUris           = { "http://localhost:5002/signin-oidc" },
    PostLogoutRedirectUris = { "http://localhost:5002/signout-callback-oidc" },

    AllowedScopes =
    {
        IdentityServerConstants.StandardScopes.OpenId,
        IdentityServerConstants.StandardScopes.Profile,
        "api1"
    },
    AllowOfflineAccess = true
};
```   

## 13.2 修改MVC客户端
MVC客户端的修改也很少 - ASP\.NET Core OpenID Connect处理程序内置了对混合流的支持，因此我们只需要更改一些配置值。   

我们配置 `ClientSecret` 以让它跟 IdentityServer 上的信息相匹配。添加 `offline_access` scopes，然后设置`ResponseType`为`code id_token`（基本的意思就是“使用混合流”）。要在我们的MVC客户端标识中保留网站声明，我们需要使用声明显式映射声明。   

``` C#
.AddOpenIdConnect("oidc", options =>
{
    options.SignInScheme = "Cookies";

    options.Authority = "http://localhost:5000";
    options.RequireHttpsMetadata = false;

    options.ClientId = "mvc";
    options.ClientSecret = "secret";
    options.ResponseType = "code id_token";

    options.SaveTokens = true;
    options.GetClaimsFromUserInfoEndpoint = true;

    options.Scope.Add("api1");
    options.Scope.Add("offline_access");
    options.ClaimActions.MapJsonKey("website", "website");
});
```   

当你运行 MVC 客户端的时候，不会有太大的区别。除此之外，授权确认页现在还会向你请求访问 额外的 API 和 离线访问（offline access） scope。   

## 13.3 使用访问令牌
OpenID Connect处理程序会自动为您保存令牌（在我们的案例中为身份，访问和刷新）。这就是`SaveTokens`设置的作用。   

cookie检查视图迭代这些值并在屏幕上显示它们。   

从技术上讲，令牌存储在cookie的属性部分中。访问它们的最简单方法是使用`Microsoft.AspNetCore.Authentication`命名空间中的扩展方法。   

例如：   

``` C#
var accessToken = await HttpContext.GetTokenAsync("access_token")
var refreshToken = await HttpContext.GetTokenAsync("refresh_token");
```   

要使用访问令牌访问API，您需要做的就是检索令牌，并在HttpClient上设置它：   

``` C#
public async Task<IActionResult> CallApi()
{
    var accessToken = await HttpContext.GetTokenAsync("access_token");

    var client = new HttpClient();
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
    var content = await client.GetStringAsync("http://localhost:5001/identity");

    ViewBag.Json = JArray.Parse(content).ToString();
    return View("json");
}
```   

创建一个名为*json.cshtml* 的视图，如下所示：   

``` xml
<pre>@ViewBag.Json</pre>
```   

确保API正在运行，启动MVC客户端并/home/CallApi在身份验证后调用。