# 第47章 授权端点(Authorize Endpoint)
授权端点可用于通过浏览器请求令牌或授权码。此过程通常涉及最终用户的身份验证和可选的同意。

> **注意**
IdentityServer支持OpenID Connect和OAuth 2.0授权请求参数的子集。有关完整列表，请参见[此处](https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest)。

* **`client_id`**  
客户的标识符（必填）。

* **`scope`**  
一个或多个注册范围（必填）

* **`redirect_uri`**  
必须与该客户端允许的重定向URI之一完全匹配（必需）

* **`response_type`**  
    * `id_token` 请求身份令牌（仅允许身份范围）
    * `token` 请求访问令牌（仅允许资源范围）
    * `id_token token` 请求身份令牌和访问令牌
    * `code` 请求授权码
    * `code id_token` 请求授权代码和身份令牌
    * `code id_token token` 请求授权代码，身份令牌和访问令牌

* **`response_mode`**  
    * `form_post` 将令牌响应作为表单发送而不是片段编码重定向（可选）

* **`state`**  
identityserver将回显令牌响应的状态值，这是针对客户端和提供者之间的往返状态，关联请求和响应以及CSRF /重放保护。（推荐的）

* **`nonce`**  
identityserver将回显身份令牌中的nonce值，这是为了重放保护）
通过隐式授权对身份令牌是必需的。

* **`prompt`**  
    * `none`请求期间不会显示任何UI。如果这是不可能的（例如，因为用户必须登录或同意），则返回错误
    * `login` 即使用户已登录并具有有效会话，也会显示登录UI

* **`code_challenge`**  
发送PKCE的代码质询

* **`code_challenge_method`**  
plain表示挑战是使用纯文本（不推荐） S256表示使用SHA256对挑战进行哈希处理

* **`login_hint`**  
可用于预先填写登录页面上的用户名字段

* **`ui_locales`**  
提供有关登录UI所需显示语言的提示

* **`max_age`**  
如果用户的登录会话超过最大年龄（以秒为单位），将显示登录UI

* **`acr_values`**  
允许传递其他身份验证相关信息 - 身份服务器特殊情况下面的专有acr_values：
    * `idp:name_of_idp` 绕过login / home领域屏幕并将用户直接转发到选定的身份提供者（如果允许每个客户端配置）
    * `tenant:name_of_tenant` 可用于将租户名称传递给登录UI

例

```
GET /connect/authorize?
    client_id=client1&
    scope=openid email api1&
    response_type=id_token token&
    redirect_uri=https://myapp/callback&
    state=abc&
    nonce=xyz
```   

（删除了URL编码，并添加了换行符以提高可读性）

> **注意**
您可以使用[IdentityModel](https://github.com/IdentityModel/IdentityModel2)客户端库以编程方式创建授权请求.NET代码。有关更多信息，请查看IdentityModel[文档](https://github.com/thinksjay/IdentityModel/blob/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86%20%E5%8D%8F%E8%AE%AE%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%BA%93/%E7%AC%AC2%E7%AB%A0%20%E6%8E%88%E6%9D%83%E7%AB%AF%E7%82%B9(Authorize%20Endpoint).md)。