# 第9章 使用客户端凭据保护API

快速入门介绍了使用IdentityServer保护API的最基本方案。 我们将定义一个API和一个想要访问它的客户端。 客户端将通过提供`ClientCredentials`在IdentityServer请求访问令牌，`ClientCredentials`充当客户端和IdentityServer都知道的秘密，并且它将使用该令牌来访问API。   

## 9.1设置ASP.NET核心应用程序
首先为应用程序创建一个目录 - 然后使用我们的模板创建一个包含基本IdentityServer设置的ASP\.NET Core应用程序，例如：   

``` dotnet
md quickstart
cd quickstart

md src
cd src

dotnet new is4empty -n IdentityServer
```   

这将创建以下文件：   

* `IdentityServer.csproj`- 项目文件和`Properties\launchSettings.json`文件
* `Program.cs`和`Startup.cs`- 主要的应用程序入口点
* `Config.cs` - IdentityServer资源和客户端配置文件   

您现在可以使用自己喜欢的文本编辑器来编辑或查看文件。如果您希望获得Visual Studio支持，可以添加如下解决方案文件：   

``` dotnet
cd ..
dotnet new sln -n Quickstart
```   

然后让它添加你的IdentityServer项目（记住这个命令，因为我们将在下面创建其他项目）：   

``` dotnet
dotnet sln add .\src\IdentityServer\IdentityServer.csproj
```

> **注意**
 此模板中使用的协议是`http`，当在`Kestrel`上运行时，端口设置为`5000`或`IISExpress`上的随机端口。您可以在`Properties\launchSettings.json`文件中更改它。但是，所有快速入门指令都假定您使用`Kestrel`上的默认端口以及`http`协议，该协议足以进行本地开发。   

## 9.2 定义API资源
API是您要保护的系统中的资源。   

资源定义可以通过多种方式加载，模板使用“代码作为配置”appproach。在[Config.cs](https://github.com/IdentityServer/IdentityServer4.Samples/blob/master/Quickstarts/1_ClientCredentials/src/IdentityServer/Config.cs)文件中，您可以找到一个名为GetApisAPI 的方法，如下所示：
``` dotnet
public static IEnumerable<ApiResource> GetApis()
{
    return new List<ApiResource>
    {
        new ApiResource("api1", "My API")
    };
}
```
## 9.3 定义客户端
下一步是定义可以访问此API的客户端。   

对于此方案，客户端将不具有交互式用户，并将使用IdentityServer的所谓客户端密钥进行身份验证。将以下代码添加到[Config.cs](https://github.com/IdentityServer/IdentityServer4.Samples/blob/master/Quickstarts/1_ClientCredentials/src/IdentityServer/Config.cs)文件中：   

``` dotnet
public static IEnumerable<Client> GetClients()
{
    return new List<Client>
    {
        new Client
        {
            ClientId = "client",

            // no interactive user, use the clientid/secret for authentication
            AllowedGrantTypes = GrantTypes.ClientCredentials,

            // secret for authentication
            ClientSecrets =
            {
                new Secret("secret".Sha256())
            },

            // scopes that client has access to
            AllowedScopes = { "api1" }
        }
    };
}
```   

## 9.4 配置IdentityServer 
在`Startup.cs`加载资源和客户端定义 - 模板已经为您执行此操作：   

``` dotnet
public void ConfigureServices(IServiceCollection services)
{
    var builder = services.AddIdentityServer()
        .AddInMemoryIdentityResources(Config.GetIdentityResources())
        .AddInMemoryApiResources(Config.GetApis())
        .AddInMemoryClients(Config.GetClients());

    // rest omitted
}
```   

就是这样 - 如果您运行服务器并浏览浏览器 `http://localhost:5000/.well-known/openid-configuration`，您应该会看到所谓的**发现文档**。客户端和API将使用它来下载必要的配置数据。  
<div align="center">
<image src="https://identityserver4.readthedocs.io/en/latest/_images/1_discovery.png">
</div>   

首次启动时，IdentityServer将为您创建一个开发人员签名密钥，它是一个名为的文件`tempkey.rsa`。您不必将该文件检入源代码管理中，如果该文件不存在，将重新创建该文件。   

## 9.5 添加API
接下来，为您的解决方案添加API。

您可以使用Visual Studio中的ASP.NET Core Web API（或空）模板，也可以使用.NET CLI来创建API项目。从`src`文件夹中运行以下命令：   

``` dotnet
dotnet new web -n Api
```   

然后通过运行以下命令将其添加到解决方案中：   

``` dotnet
cd ..
dotnet sln add .\src\Api\Api.csproj
```   

将API应用程序配置为`http://localhost:5001`仅运行。您可以通过编辑Properties文件夹中的`launchSettings.json`文件来完成此操作。将应用程序URL设置更改为：   

```
"applicationUrl": "http://localhost:5001"
```   

## 9.6 控制器
在API项目中添加一个新文件夹Controllers和一个新控制器`IdentityController`：   

``` dotnet
[Route("identity")]
[Authorize]
public class IdentityController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        return new JsonResult(from c in User.Claims select new { c.Type, c.Value });
    }
}
```   

这个控制器将在后面被用于测试授权需求，同时通过API的眼睛（浏览工具）来可视化身份信息。

## 9.7 配置
最后一步是将身份验证服务添加到DI和身份验证中间件到管道。这些将：  

* 验证输入的令牌以确保它来自可信任的发布者（IdentityServer）
* 验证令牌是否可用于该 api（也就是 Scope）。   

将`Startup`更新为如下所示：

``` dotnet
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMvcCore()
            .AddAuthorization()
            .AddJsonFormatters();

        services.AddAuthentication("Bearer")
            .AddJwtBearer("Bearer", options =>
            {
                options.Authority = "http://localhost:5000";
                options.RequireHttpsMetadata = false;

                options.Audience = "api1";
            });
    }

    public void Configure(IApplicationBuilder app)
    {
        app.UseAuthentication();

        app.UseMvc();
    }
}
```   

`AddAuthentication`将身份验证服务添加到DI并配置`"Bearer"`为默认方案。 `UseAuthentication`将身份验证中间件添加到管道中，以便在每次调用主机时自动执行身份验证。   

`http://localhost:5001/identity`在浏览器上导航到控制器应返回401状态代码。这意味着您的API需要凭证，现在受IdentityServer保护。   

## 9.8 创建客户端
最后一步是编写请求访问令牌的客户端，然后使用此令牌访问API。为此，在您的解决方案中添加一个控制台项目，请记住在以下位置创建它src：   

``` dotnet
dotnet new console -n Client
```   

然后和以前一样，使用以下方法将其添加到您的解   

``` dotnet
cd ..
dotnet sln add .\src\Client\Client.csproj
```   

打开`Program.cs`并将内容从这里复制到它。   

客户端程序异步调用`Main`方法以运行异步`http`调用。 从`C#7.1`开始，此功能可用，一旦您编辑`Client.csproj`以将以下行添加为`PropertyGroup`，它就可用：   

``` xml
<LangVersion>latest</LangVersion>
```   

IdentityServer的令牌端点实现**OAuth 2.0**协议，您可以使用原始HTTP来访问它。但是，我们有一个名为`IdentityModel`的客户端库，它将协议交互封装在易于使用的API中。   

将`IdentityModel` NuGet包添加到您的客户端。这可以通过Visual Studio的nuget对话框，手动添加到`Client.csproj`文件，或使用CLI来完成：   

``` dotnet
dotnet add package IdentityModel
```   

IdentityModel包括用于发现端点的客户端库。这样您只需要知道IdentityServer的基地址 - 可以从元数据中读取实际的端点地址：   

``` dotnet
// discover endpoints from metadata
var client = new HttpClient();
var disco = await client.GetDiscoveryDocumentAsync("http://localhost:5000");
if (disco.IsError)
{
    Console.WriteLine(disco.Error);
    return;
}
```   

接下来，您可以使用**发现文档**中的信息向IdentityServer请求令牌以访问api1：   

``` dotnet
// request token
var tokenResponse = await client.RequestClientCredentialsTokenAsync(new ClientCredentialsTokenRequest
{
    Address = disco.TokenEndpoint,

    ClientId = "client",
    ClientSecret = "secret",
    Scope = "api1"
});

if (tokenResponse.IsError)
{
    Console.WriteLine(tokenResponse.Error);
    return;
}

Console.WriteLine(tokenResponse.Json);
```   

> **注意**
将访问令牌从控制台复制并粘贴到<https://jwt.io/>以检查原始令牌。

## 9.9 调用
要将访问令牌发送到API，通常使用HTTP `Authorization`标头。这是使用`SetBearerToken`扩展方法完成的：   

``` dotnet
// call api
var client = new HttpClient();
client.SetBearerToken(tokenResponse.AccessToken);

var response = await client.GetAsync("http://localhost:5001/identity");
if (!response.IsSuccessStatusCode)
{
    Console.WriteLine(response.StatusCode);
}
else
{
    var content = await response.Content.ReadAsStringAsync();
    Console.WriteLine(JArray.Parse(content));
}
```   

输出应如下所示：

<div align="center">
<image src="https://identityserver4.readthedocs.io/en/latest/_images/1_client_screenshot.png">
</div>   

> **注意**
默认情况下，访问令牌将包含有关范围(scope)，生命周期（nbf和exp），客户端ID（client_id）和颁发者名称（iss）的声明。   

## 9.10 进一步的实验
本演练重点介绍了迄今为止的成功之路   

* 客户端能够请求令牌
* 客户端可以使用令牌来访问API   

你现在可以尝试引发一些错误来学习系统的相关行为，比如：   

* 尝试在未运行时连接到IdentityServer（不可用）
* 尝试使用无效的客户端ID或密码来请求令牌
* 尝试在令牌请求期间请求无效范围
* 尝试在API未运行时调用API（不可用）
* 不要将令牌发送到API
* 将API配置为需要与令牌中的范围不同的范围
