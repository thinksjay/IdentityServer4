# 第28章 确认(Consent)
在授权请求期间，如果IdentityServer需要用户同意，则浏览器将被重定向到同意页面。

同意用于允许最终用户授予客户端对资源（身份或API）的访问权限。这通常仅对第三方客户端是必需的，并且可以在客户端设置上按客户端启用/禁用。

## 28.1 确认页
为了让用户同意，托管应用程序必须提供同意页面。该快速入门UI有一个批准页面的基本实现。

同意页面通常呈现当前用户的显示名称，请求访问的客户端的显示名称，客户端的徽标，有关客户端的更多信息的链接以及客户端请求访问的资源列表。允许用户表明他们的同意应该被“记住”也是很常见的，因此将来不会再次提示同一客户。

一旦用户提供了同意，同意页面必须通知IdentityServer同意，然后必须将浏览器重定向回授权端点。

## 28.2 授权上下文
IdentityServer将*returnUrl*参数（可在用户交互选项上配置）传递到包含授权请求参数的同意页面。这些参数提供了同意页面的上下文，可以在交互服务的帮助下阅读。该`GetAuthorizationContextAsync`API将返回的实例`AuthorizationRequest`。

可以使用`IClientStore`和`IResourceStore`接口获取有关客户端或资源的其他详细信息。

## 28.3 通知IdentityServer同意结果
该API允许'grantconsentasync'在[交互服务](https://github.com/thinksjay/IdentityServer4/blob/master/%E7%AC%AC%E4%BA%94%E9%83%A8%E5%88%86%20%E5%8F%82%E8%80%83/%E7%AC%AC59%E7%AB%A0%20IdentityServer%E4%BA%A4%E4%BA%92%E6%9C%8D%E5%8A%A1.md)页面通知identityserver同意的结果（这也可能是在客户端访问等）。

IdentityServer将暂时保留同意的结果。这种持久性默认使用cookie，因为它只需要持续足够长的时间来将结果传回给授权端点。这种临时持久性与用于“记住我的同意”功能的持久性不同（并且授权端点持续“记住我对用户的同意”）。如果您希望在同意页面和授权重定向之间使用其他一些持久性，那么您可以`IMessageStore<ConsentResponse>`在DI中实现并注册实现。

## 28.4 将用户返回到授权端点
一旦同意页面通知IdentityServer结果，就可以将用户重定向回*returnUrl*。您的同意页面应通过验证*returnUrl*是否有效来防止打开重定向。这可以通过调用[交互服务](https://github.com/thinksjay/IdentityServer4/blob/master/%E7%AC%AC%E4%BA%94%E9%83%A8%E5%88%86%20%E5%8F%82%E8%80%83/%E7%AC%AC59%E7%AB%A0%20IdentityServer%E4%BA%A4%E4%BA%92%E6%9C%8D%E5%8A%A1.md)的`IsValidReturnUrl`来完成。此外，如果`GetAuthorizationContextAsync`返回非null结果，那么您还可以信任*returnUrl*有效。