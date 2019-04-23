# 第10章 使用密码保护API

**OAuth 2.0资源所有者密码授权**允许客户端向令牌服务发送用户名和密码，并获取代表该用户的访问令牌。   

除了无法承载浏览器的旧应用程序之外，规范通常建议不要使用资源所有者密码授予。一般来说，当您要对用户进行身份验证并请求访问令牌时，使用其中一个交互式**OpenID Connect**流程通常要好得多。   

然而，这个授权类型允许我们在 IdentityServer 快速入门中引入 用户 的概念，这是我们要展示它的原因。    

## 10.1 添加用户
就像基于内存存储的资源（即 范围 Scopes）和客户端一样，对于用户也可以这样做。      

> 注意
有关如何正确存储和管理用户帐户的详细信息，请查看基于`ASP.NET Identity`的快速入门。   

`TestUser`类代表测试用户及其声明。让我们通过在`config`类中添加以下代码来创建几个用户：   

首先将以下 `using`语句添加到`Config.cs`文件中：   

``` dotnet
using IdentityServer4.Test;

public static List<TestUser> GetUsers()
{
    return new List<TestUser>
    {
        new TestUser
        {
            SubjectId = "1",
            Username = "alice",
            Password = "password"
        },
        new TestUser
        {
            SubjectId = "2",
            Username = "bob",
            Password = "password"
        }
    };
}
```   

然后使用IdentityServer注册测试用户：   

``` dotnet
public void ConfigureServices(IServiceCollection services)
{
    // configure identity server with in-memory stores, keys, clients and scopes
    services.AddIdentityServer()
        .AddInMemoryApiResources(Config.GetApiResources())
        .AddInMemoryClients(Config.GetClients())
        .AddTestUsers(Config.GetUsers());
}
```   

该`AddTestUsers`扩展方法在背后做了以下几件事：   

* 添加对资源所有者密码授予的支持
* 添加对登录UI通常使用的用户相关服务的支持（我们将在下一个快速入门中使用它）
* 添加对基于测试用户的配置文件服务的支持（您将在下一个快速入门中了解更多信息）   

## 10.2 为资源所有者密码授权添加客户端
您可以通过更改`AllowedGrantTypes`属性，将授权类型的支持添加到现有客户端。 如果您需要您的客户端能够使用绝对支持的两种授权类型。

我们正在为资源所有者用例创建一个单独的客户端，将以下内容添加到客户端配置中：   

``` dotnet
public static IEnumerable<Client> GetClients()
{
    return new List<Client>
    {
        // other clients omitted...

        // resource owner password grant client
        new Client
        {
            ClientId = "ro.client",
            AllowedGrantTypes = GrantTypes.ResourceOwnerPassword,

            ClientSecrets =
            {
                new Secret("secret".Sha256())
            },
            AllowedScopes = { "api1" }
        }
    };
}
```   

## 10.3 使用密码授权请求令牌
在解决方案中添加新的控制台客户端。   

新客户端看起来与我们为**客户端凭据授权**所做的非常相似。主要区别在于客户端会以某种方式收集用户的密码，并在令牌请求期间将其发送到令牌服务。   

`IdentityModel`再次可以在这里提供帮助：   

``` dotnet
// request token
var tokenResponse = await client.RequestPasswordTokenAsync(new PasswordTokenRequest
{
    Address = disco.TokenEndpoint,
    ClientId = "ro.client",
    ClientSecret = "secret",

    UserName = "alice",
    Password = "password",
    Scope = "api1"
});

if (tokenResponse.IsError)
{
    Console.WriteLine(tokenResponse.Error);
    return;
}

Console.WriteLine(tokenResponse.Json);
```   

当您将令牌发送到身份API端点时，您会注意到与客户端凭据授权相比有一个小但重要的区别。访问令牌现在将包含sub唯一标识用户的声明。通过在调用API之后检查内容变量可以看到这个`sub`声明，并且控制器应用程序也会在屏幕上显示该声明。   

`sub`信息的存在（或缺失）使得 API 能够区分代表客户端的调用和代表用户的调用。