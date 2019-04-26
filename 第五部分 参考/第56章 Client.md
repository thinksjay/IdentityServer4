# 第56章 Client
该`Client`模型的OpenID Connect或OAuth 2.0 客户端-例如，本地应用，Web应用程序或基于JS的应用程序。

## 56.1 Basics
* **`Enabled`**  
指定是否启用客户端。默认为true。

* **`ClientId`**  
客户端的唯一ID

* **`ClientSecrets`**  
客户端机密列表 - 访问令牌端点的凭据。

* **`RequireClientSecret`**  
指定此客户端是否需要密钥才能从令牌端点请求令牌（默认为`true`）

* **`AllowedGrantTypes`**  
指定允许客户端使用的授权类型。使用`GrantTypes`该类进行常见组合。

* **`RequirePkce`**  
指定使用基于授权代码的授权类型的客户端是否必须发送校验密钥

* **`AllowPlainTextPkce`**  
指定使用PKCE的客户端是否可以使用纯文本代码质询（不推荐 - 默认为`false`）

* **`RedirectUris`**  
指定允许的URI以返回令牌或授权码

* **`AllowedScopes`**  
默认情况下，客户端无权访问任何资源 - 通过添加相应的范围名称来指定允许的资源

* **`AllowOfflineAccess`**  
指定此客户端是否可以请求刷新令牌（请求`offline_access`范围）

* **`AllowAccessTokensViaBrowser`**  
指定是否允许此客户端通过浏览器接收访问令牌。这对于强化允许多种响应类型的流是有用的（例如，通过禁止应该使用*code id_token*来添加令牌响应类型并因此将令牌泄漏到浏览器的混合流客户端。

* **`Properties`**  
字典根据需要保存任何自定义客户端特定值。

## 56.2 认证/注销
* **`PostLogoutRedirectUris`**  
指定在注销后重定向到的允许URI。有关更多详细信息，请参阅[OIDC Connect会话管理规范](https://openid.net/specs/openid-connect-session-1_0.html)。

* **`FrontChannelLogoutUri`**  
指定客户端的注销URI，以用于基于HTTP的前端通道注销。有关详细信息，请参阅[OIDC Front-Channel规范](https://openid.net/specs/openid-connect-frontchannel-1_0.html)。

* **`FrontChannelLogoutSessionRequired`**  
指定是否应将用户的会话ID发送到`FrontChannelLogoutUri`。默认为`true`。

* **`BackChannelLogoutUri`**  
指定客户端的注销URI，以用于基于HTTP的反向通道注销。有关更多详细信息，请参阅[OIDC Back-Channel规范](https://openid.net/specs/openid-connect-backchannel-1_0.html)。

* **`BackChannelLogoutSessionRequired`**  
指定是否应在请求中将用户的会话ID发送到`BackChannelLogoutUri`。默认为`true`。

* **`EnableLocalLogin`**  
指定此客户端是否可以仅使用本地帐户或外部IdP。默认为`true`。

* **`IdentityProviderRestrictions`**  
指定可以与此客户端一起使用的外部IdP（如果列表为空，则允许所有IdP）。默认为空。

* **`UserSsoLifetime`*添加在2.3中***  
自上次用户进行身份验证以来的最长持续时间（以秒为单位）。默认为`null`。您可以调整会话令牌的生命周期，以控制在使用Web应用程序时，用户需要重新输入凭据的时间和频率，而不是进行静默身份验证。

## 56.3 Token
* **`IdentityTokenLifetime`**  
身份令牌的生命周期，以秒为单位（默认为300秒/ 5分钟）

* **`AccessTokenLifetime`**  
访问令牌的生命周期，以秒为单位（默认为3600秒/ 1小时）

* **`AuthorizationCodeLifetime`**  
授权代码的生命周期，以秒为单位（默认为300秒/ 5分钟）

* **`AbsoluteRefreshTokenLifetime`**  
刷新令牌的最长生命周期（秒）。默认为2592000秒/ 30天

* **`SlidingRefreshTokenLifetime`**  
滑动刷新令牌的生命周期，以秒为单位。默认为1296000秒/ 15天

* **`RefreshTokenUsage`**  
    `ReUse` 刷新令牌时，刷新令牌句柄将保持不变

    `OneTime`刷新令牌时将更新刷新令牌句柄。这是默认值。

* **`RefreshTokenExpiration`**  
    `Absolute` 刷新令牌将在固定时间点到期（由AbsoluteRefreshTokenLifetime指定）  

    `Sliding`刷新令牌时，将刷新刷新令牌的生命周期（按SlidingRefreshTokenLifetime中指定的数量）。生命周期不会超过*AbsoluteRefreshTokenLifetime*。

* **`UpdateAccessTokenClaimsOnRefresh`**  
获取或设置一个值，该值指示是否应在刷新令牌请求上更新访问令牌（及其声明）。

* **`AccessTokenType`**  
指定访问令牌是引用令牌还是自包含JWT令牌（默认为Jwt）。

* **`IncludeJwtId`**  
指定JWT访问令牌是否应具有嵌入的唯一ID（通过jti声明）。

* **`AllowedCorsOrigins`**  
如果指定，将由默认CORS策略服务实现（内存和EF）用于为JavaScript客户端构建CORS策略。

* **`Claims`**  
允许客户端的设置声明（将包含在访问令牌中）。

* **`AlwaysSendClientClaims`**  
如果设置，将为每个流发送客户端声明。如果不是，仅用于客户端凭证流（默认为`false`）

* **`AlwaysIncludeUserClaimsInIdToken`**  
在请求id token和access token时，如果用户声明始终将其添加到id token而不是请求客户端使用userinfo endpoint。默认值为*false*。

* **`ClientClaimsPrefix`**  
如果设置，将以前缀为前缀客户端声明类型。默认为*client_*。目的是确保它们不会意外地与用户声明冲突。

* **`PairWiseSubjectSalt`**  
对于此客户端的用户，在成对的subjectId生成中使用的salt值。

## 56.4 确认页面
* **`RequireConsent`**  
指定是否需要同意屏幕。默认为*true*。

* **`AllowRememberConsent`**  
指定用户是否可以选择存储同意决策。默认为*true*。

* **`ConsentLifetime`**  
用户同意的生命周期，以秒为单位。默认为null（无到期）。

* **`ClientName`**  
客户端显示名称（用于记录和同意屏幕）

* **`ClientUri`**  
有关客户端的更多信息的URI（在同意屏幕上使用）

* **`LogoUri`**  
URI到客户端徽标（在同意屏幕上使用）

## 56.5 设备流程
* **`UserCodeType`**  
指定用于客户端的用户代码的类型。否则使用默认值。

* **`DeviceCodeLifetime`**  
设备代码的生命周期，以秒为单位（默认为300秒/ 5分钟）