# 第16章 使用ASP\.NET Core Identity

> **注意**
对于任何先决条件（例如模板），首先要查看概述。

IdentityServer旨在提供灵活性，其中一部分允许您为用户及其数据（包括账户密码）使用所需的任何数据库。如果您从新的用户数据库开始，那么ASP.NET Identity是您可以选择的一个选项。本快速入门显示了如何在IdentityServer中使用ASP.NET Identity。

本快速入门使用ASP.NET Identity的方法是为IdentityServer Host创建一个新项目。这个新项目将取代我们在之前的快速入门中构建的先前IdentityServer项目。这个新项目的原因是由于使用ASP.NET身份时UI资产的差异（主要是登录和注销的差异）。此解决方案中的所有其他项目（针对客户端和API）将保持不变。

> **注意**
本快速入门假设您熟悉ASP\.NET Identity的工作原理。如果不是，建议您先了解它。

## 16.1 ASP\.NET身份的新项目
第一步是为您的解决方案添加ASP.NET Identity的新项目。我们提供了一个模板，其中包含ASP.NET Identity与IdentityServer所需的最小UI资产。您最终将删除IdentityServer的旧项目，但是您需要迁移一些项目。   

首先创建一个将使用ASP.NET Identity的新IdentityServer项目：   
``` shell
cd quickstart/src
dotnet new is4aspid -n IdentityServerAspNetIdentity
```   

当提示“seed”用户数据库时，选择“Y”表示“是”。这将使用我们的“alice”和“bob”用户填充用户数据库。他们的密码是“Pass123$”。   

> **注意**
该模板使用Sqlite作为用户的数据库，并在模板中预先创建EF迁移。如果您希望使用其他数据库提供程序，则需要更改代码中使用的提供程序并重新创建EF迁移。   

## 16.2 检查新项目
在您选择的编辑器中打开新项目，并检查生成的代码。一定要看看：

### 16.2.1 IdentityServerAspNetIdentity.csproj 
请注意对IdentityServer4.AspNetIdentity的引用。此NuGet包中包含IdentityServer的ASP.NET Identity集成组件。

### 16.2.2 Startup.cs 
在`ConfigureServices`中，注意必要的`AddDbContext<ApplicationDbContext>`和调用以配置ASP\.NET Identity。`AddIdentity<ApplicationUser, IdentityRole>`   

另请注意，您在之前的快速入门中完成的大部分相同IdentityServer配置已经完成。模板使用内存中的样式用于客户端和资源，这些样式来自`Config.cs`。    

最后，注意添加新的`AddAspNetIdentity<ApplicationUser>`。 `AddAspNetIdentity`添加集成层以允许IdentityServer访问ASP.NET Identity用户数据库的用户数据。当IdentityServer必须将用户的声明添加到令牌时，这是必需的。   

### 16.2.3 Config.cs 
*Config.cs*包含硬编码的内存客户端和资源定义。为了使相同的客户端和API与之前的快速入门保持一致，我们需要将旧IdentityServer项目中的配置数据复制到此项目中。现在就这样做，然后*Config.cs*看起来像这样：   

``` C#
public static class Config
{
    public static IEnumerable<IdentityResource> GetIdentityResources()
    {
        return new List<IdentityResource>
        {
            new IdentityResources.OpenId(),
            new IdentityResources.Profile(),
        };
    }

    public static IEnumerable<ApiResource> GetApis()
    {
        return new List<ApiResource>
        {
            new ApiResource("api1", "My API")
        };
    }

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
            },
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
            },
            // OpenID Connect hybrid flow client (MVC)
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
            },
            // JavaScript Client
            new Client
            {
                ClientId = "js",
                ClientName = "JavaScript Client",
                AllowedGrantTypes = GrantTypes.Code,
                RequirePkce = true,
                RequireClientSecret = false,

                RedirectUris =           { "http://localhost:5003/callback.html" },
                PostLogoutRedirectUris = { "http://localhost:5003/index.html" },
                AllowedCorsOrigins =     { "http://localhost:5003" },

                AllowedScopes =
                {
                    IdentityServerConstants.StandardScopes.OpenId,
                    IdentityServerConstants.StandardScopes.Profile,
                    "api1"
                }
            }
        };
    }
}
```   

此时，您不再需要旧的IdentityServer项目。   

### 16.2.4 Program.cs文件和SeedData.cs 
*Program.cs*中的`Main`比大多数ASP.NET Core项目略有不同。请注意这是如何查找名为/ seed的命令行参数，该参数用作为ASP.NET Identity数据库中的用户设定种子的标志。

查看`SeedData`类的代码，了解如何创建数据库以及创建第一个用户。

### 16.2.5 AccountController 
要在此模板中检查的最后一个代码是`AccountController`。这包含与之前的快速入门和模板略有不同的登录和注销代码。请注意使用ASP.NET Identity `SignInManager<ApplicationUser>`和`UserManager<ApplicationUser>`来自ASP.NET Identity来验证凭据和管理身份验证会话。   

其余大部分代码与之前的快速入门和模板相同。   

## 16.3 使用MVC客户端登录
此时，您应该运行所有现有客户端和模板。一个例外是*ResourceOwnerClient* -密码将需要更新，更新`password`为`Pass123$`。   

启动MVC客户端应用程序，您应该能够单击“安全”链接以登录。   

<div align="center">
<image src="https://identityserver4.readthedocs.io/en/latest/_images/8_mvc_client.png">
</div>   

您应该被重定向到ASP\.NET Identity登录页面。使用新创建的用户登录：   

<div align="center">
<image src="https://identityserver4.readthedocs.io/en/latest/_images/8_login.png">
</div>   

登录后，您会看到正常的同意页面。获得同意后，您将被重定向回MVC客户端应用程序，其中应列出您的用户声明。   

<div align="center">
<image src="https://identityserver4.readthedocs.io/en/latest/_images/8_claims.png">
</div>   

您还应该能够单击“使用应用程序标识调用API”来代表用户调用API：   

<div align="center">
<image src="https://identityserver4.readthedocs.io/en/latest/_images/8_api_claims.png">
</div>    

现在，您正在使用IdentityServer中的ASP.NET身份用户。

## 16.4 少了什么东西？
此模板中的大部分代码与我们提供的其他快速入门和模板类似。您将注意到此模板中缺少的一件事是用户注册，密码重置以及您可能期望从Visual Studio ASP.NET Identity模板中获得的其他内容的UI代码。   

鉴于需求的多样性和使用ASP.NET Identity的不同方法，我们的模板明确地不提供这些功能。您应该知道ASP.NET Identity如何充分地将这些功能添加到项目中。或者，您可以基于Visual Studio ASP.NET Identity模板创建新项目，并将您在这些快速入门中了解到的IdentityServer功能添加到该项目中。

