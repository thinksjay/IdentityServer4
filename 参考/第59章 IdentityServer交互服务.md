# 第59章 IdentityServer交互服务
`IIdentityServerInteractionService`接口旨在提供用户界面用于与IdentityServer通信的服务，主要与用户交互有关。它可以从依赖注入系统获得，通常作为构造函数参数注入到IdentityServer的用户界面的MVC控制器中。

## 59.1 IIdentityServerInteractionService APIs
* **`GetAuthorizationContextAsync`**  
基于传递给登录或同意页面的`returnUrl`返回`AuthorizationRequest`。

* **`IsValidReturnUrl`**  
指示在登录或同意后`returnUrl`是否为重定向的有效URL。

* **`GetErrorContextAsync`**  
根据传递给错误页面的`errorId`返回`ErrorMessage`。

* **`GetLogoutContextAsync`**  
根据传递给注销页面的`logoutId`返回`LogoutRequest`。

* **`CreateLogoutContextAsync`**  
如果当前没有`logoutId`，则用于创建`logoutId`。这将创建一个cookie，捕获注销所需的所有当前状态，`logoutId`标识该cookie。这通常在没有当前`logoutId`时使用，并且注销页面必须捕获当前用户在重定向到外部身份提供程序以进行注销之前注销所需的状态。新创建的`logoutId`需要在注销时往返外部身份提供商，然后在注销回调页面上使用，就像在普通注销页面上一样。

* **`GrantConsentAsync`**  
接受`ConsentResponse`以通知IdentityServer用户同意特定的`AuthorizationRequest`。

* **`GetAllUserConsentsAsync`**  
返回用户的`Consent`集合。

* **`RevokeUserConsentAsync`**  
撤消用户对客户端的所有同意和授权。

* **`RevokeTokensForCurrentSessionAsync`**  
撤消用户在当前会话期间签署的客户的所有同意和授权。

## 59.2 AuthorizationRequest
* **`ClientId`**  
发起请求的客户端标识符。

* **`RedirectUri`**  
成功授权后将用户重定向到的URI。

* **`DisplayMode`**  
显示模式从授权请求传递。

* **`UiLocales`**  
从授权请求传递的UI语言环境。

* **`IdP`**  
外部身份提供者请求。这用于绕过家庭领域发现（HRD）。这是通过“idp：”前缀提供给授权请求的`acr_values`参数。

* **`Tenant`**  
租户请求。这是通过“tenant：”前缀提供给授权请求上的`acr_values`参数。

* **`LoginHint`**  
用户将用于登录的预期用户名。这是通过授权请求上的`login_hint`参数从客户端请求的。

* **`PromptMode`**  
授权请求中请求的提示模式。

* **`AcrValues`**  
从授权请求传递的acr值。

* **`ScopesRequested`**  
授权请求中请求的范围。

* **`Parameters`**  
整个参数集合传递给授权请求。

## 59.3 ErrorMessage
* **`DisplayMode`**  
显示模式从授权请求传递。

* **`UiLocales`**  
从授权请求传递的UI语言环境。

* **`Error`**  
错误代码。

* **`RequestId`**  
每请求标识符。这可用于向最终用户显示，并可用于诊断。

## 59.4 LogoutRequest
* **`ClientId`**  
发起请求的客户端标识符。

* **`PostLogoutRedirectUri`**  
用户在注销后将其重定向到的URL。

* **`SessionId`**  
用户当前的会话ID。

* **`SignOutIFrameUrl`**  
要在注销页面上的`<iframe>`中呈现以启用单点注销的URL。

* **`Parameters`**  
整个参数集合传递给结束会话端点。

* **`ShowSignoutPrompt`**  
指示是否应根据传递到结束会话端点的参数提示用户注销。

## 59.5 ConsentResponse
* **`ScopesConsented`**  
用户同意的范围集合。

* **`RememberConsent`**  
指示是否持久保留用户同意的标志。

## 59.6 Consent
* **`SubjectId`**  
授予同意的主题ID。

* **`ClientId`**  
同意的客户端标识符。

* **`Scopes`**  
范围的集合同意。

* **`CreationTime`**  
获得同意的日期和时间。

* **`Expiration`**  
同意过期的日期和时间。