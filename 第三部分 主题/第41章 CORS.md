# 第41章 CORS
IdentityServer中的许多端点将通过基于JavaScript的客户端的Ajax调用进行访问。鉴于IdentityServer最有可能托管在与这些客户端不同的源上，这意味着需要配置[跨源资源共享](http://www.html5rocks.com/en/tutorials/cors/)（CORS）。

## 41.1 基于客户端的CORS配置
配置CORS的一种方法是在[客户端配置](https://github.com/thinksjay/IdentityServer4/blob/master/%E7%AC%AC%E4%BA%94%E9%83%A8%E5%88%86%20%E5%8F%82%E8%80%83/%E7%AC%AC56%E7%AB%A0%20Client.md)上使用`AllowedCorsOrigins`该集合。只需将客户端的原点添加到集合中，IdentityServer中的默认配置将查询这些值以允许来自源的跨源调用。

> **注意**
配置CORS时，请务必使用原点（不是URL）。例如：`https://foo:123/`是一个URL，而是`https://foo:123`一个原点。

如果您使用我们提供的“内存中”或基于EF的客户端配置，则将使用此默认CORS实现。如果您定义自己的`IClientStore`，那么您将需要实现自己的自定义CORS策略服务（见下文）。

## 41.2 自定义Cors策略服务
IdentityServer允许托管应用程序实现`ICorsPolicyService`完全控制CORS策略。

要实现单一的方法是：`Task<bool> IsOriginAllowedAsync(string origin)`。如果允许原点则返回true，否则返回false

实现后，只需在DI中注册实现，然后IdentityServer将使用您的自定义实现。

### 41.2.1 DefaultCorsPolicyService

如果您只是希望对一组允许的原点进行硬编码，那么您可以使用一个预先构建`ICorsPolicyService`的实现调用`DefaultCorsPolicyService`。这将被配置为DI单例，并以其硬编码的`AllowedOrigins`收集，或设置标志`AllowAll`为true允许所有的源点。例如，在`ConfigureServices`：

``` C#
var cors = new DefaultCorsPolicyService(_loggerFactory.CreateLogger<DefaultCorsPolicyService>())
{
    AllowedOrigins = { "https://foo", "https://bar" }
};
services.AddSingleton<ICorsPolicyService>(cors);
```   

> **注意**
AllowAll谨慎使用。

## 41.3 将IdentityServer的CORS策略与ASP.NET Core的CORS策略混合
IdentityServer使用ASP.NET Core的CORS中间件来提供其CORS实现。托管IdentityServer的应用程序可能还需要CORS用于自己的自定义端点。通常，两者应该在同一个应用程序中一起工作。

您的代码应使用ASP.NET Core中记录的CORS功能，而不考虑IdentityServer。这意味着您应该定义策略并正常注册中间件。如果您的应用程序在`ConfigureServices`中定义了策略，那么这些策略应继续在您使用它们的相同位置（在您配置CORS中间件的地方或在`EnableCors`控制器代码中使用MVC 属性的位置）。相反，如果您使用CORS中间件（通过策略构建器回调）定义内联策略，那么它也应该继续正常工作。

您使用ASP.NET Core CORS服务与IdentityServer之间可能存在冲突的一种情况是您决定创建自定义`ICorsPolicyProvider`。鉴于ASP.NET Core的CORS服务和中间件的设计，IdentityServer实现了自己的自定义`ICorsPolicyProvider`并将其注册到DI系统中。幸运的是，IdentityServer实现旨在使用装饰器模式来包装`ICorsPolicyProvider`已在DI中注册的任何现有模式 。这意味着你也可以实现`ICorsPolicyProvider`，但它只需要在DI中的IdentityServer之前注册（例如，在`ConfigureServices`）。

