# 第61章 IdentityServer Options
* **`IssuerUri`**  
设置将在发现文档和已颁发的JWT令牌中显示的颁发者名称。建议不要设置此属性，该属性从客户端使用的主机名中推断颁发者名称。
* **`PublicOrigin`**  
此服务器实例的来源，例如<https://myorigin.com>。如果未设置，则从请求推断出原始名称。  

## 61.1 端点(Endpoints)
允许启用/禁用各个端点，例如令牌，授权，用户信息等。   

默认情况下，所有端点都已启用，但您可以通过禁用不需要的端点来锁定服务器。   

## 61.2 发现(Discovery)
允许启用/禁用发现文档的各个部分，例如端点，范围，声明，授权类型等。   

该`CustomEntries`字典允许将自定义元素添加到发现文档中。  

## 61.3 认证(Authentication)
* **`CookieLifetime`**  
身份验证cookie生存期（仅在使用IdentityServer提供的cookie处理程序时有效）。   

* **`CookieSlidingExpiration`**  
指定cookie是否应该滑动（仅在使用IdentityServer提供的cookie处理程序时有效）。  

* **`RequireAuthenticatedUserForSignOutMessage`**  
指示是否必须对用户进行身份验证以接受参数以结束会话端点。默认为false。   

* **`CheckSessionCookieName`**  
用于检查会话端点的cookie的名称。  

* **`RequireCspFrameSrcForSignout`**  
如果设置，将要求frame-src CSP标头在结束会话回调端点上发出，该端点向客户端呈现iframe以进行前端通道注销通知。默认为true。   

## 61.4 事件(Events)
允许配置是否应将哪些事件提交到已注册的事件接收器。有关活动的更多信息，请参见[此处](https://github.com/thinksjay/IdentityServer4/blob/master/%E7%AC%AC%E4%B8%89%E9%83%A8%E5%88%86%20%E4%B8%BB%E9%A2%98/%E7%AC%AC32%E7%AB%A0%20%E4%BA%8B%E4%BB%B6.md)。   

## 61.5 InputLengthRestrictions
允许设置各种协议参数的长度限制，如客户端ID，范围，重定向URI等。

## 61.6 用户交互(UserInteraction)
* **`LoginUrl`**，**`LogoutUrl`**，**`ConsentUrl`**，**`ErrorUrl`**，**`DeviceVerificationUrl`**  
设置登录，注销，同意，错误和设备验证页面的URL。  

* **`LoginReturnUrlParameter`**    
设置传递给登录页面的返回URL参数的名称。默认为returnUrl。  

* **`LogoutIdParameter`**  
设置传递给注销页面的注销消息id参数的名称。默认为logoutId。  

* **`ConsentReturnUrlParameter`**  
设置传递给同意页面的返回URL参数的名称。默认为returnUrl。  

* **`ErrorIdParameter`**  
设置传递给错误页面的错误消息id参数的名称。默认为errorId。  

* **`CustomRedirectReturnUrlParameter`**  
设置从授权端点传递给自定义重定向的返回URL参数的名称。默认为returnUrl。  

* **`DeviceVerificationUserCodeParameter`**  
设置传递给设备验证页面的用户代码参数的名称。默认为userCode。  

* **`CookieMessageThreshold`**  
IdentityServer和某些UI页面之间的某些交互需要cookie来传递状态和上下文（上面具有可配置“message id”参数的任何页面）。由于浏览器对cookie的数量及其大小有限制，因此该设置用于防止创建过多的cookie。该值设置将创建的任何类型的消息cookie的最大数量。一旦达到限制，将清除最早的消息cookie。这有效地表示用户在使用IdentityServer时可以打开多少个选项卡。   

## 61.7 缓存(Caching)
这些设置仅在启动时在服务配置中启用了相应的缓存时才适用。  

* **`ClientStoreExpiration`**  
从客户端存储加载的客户端配置的缓存持续时间。

* **`ResourceStoreExpiration`**  
缓存从资源存储加载的标识和API资源配置的持续时间。  

## 61.8 CORS 
IdentityServer支持某些端点的CORS。底层CORS实现由ASP.NET Core提供，因此它在依赖注入系统中自动注册。

* **`CorsPolicyName`**  
将为IdentityServer中的CORS请求评估的CORS策略的名称（默认为`"IdentityServer4"`）。处理此问题的策略提供程序是根据`ICorsPolicyService`依赖注入系统中注册的方式实现的。如果您希望自定义允许连接的CORS源集，那么建议您提供自定义实现`ICorsPolicyService`。   

* **`CorsPaths`**  
IdentityServer中支持CORS的端点。默认为发现，用户信息，令牌和吊销端点。  

* **`PreflightCacheDuration`**  
*Nullable\<TimeSpan\>*指示在预检*Access-Control-Max-Age*响应头中使用的值。默认为null，表示未在响应上设置缓存标头。  

## 61.9 CSP（内容安全策略）
在适当的情况下，IdentityServer会为某些响应发出CSP标头。  

* **`Level`**  
要使用的CSP级别。默认情况下使用CSP级别2，但如果必须支持旧浏览器，则将其更改`CspLevel.One`为兼容它们。  

* **`AddDeprecatedHeader`**  
指示是否`X-Content-Security-Policy`还应发出较旧的CSP标头（除了基于标准的标头值）。默认为true。  

 ## 61.10 设备流程
* **`DefaultUserCodeType`**  
要使用的用户代码类型，除非在客户端级别设置。默认为数字，9位数代码。  

* **`Interval`**  
定义令牌端点上允许的最小轮询间隔。默认为5。