# 第49章 令牌端点(Token Endpoint)
令牌端点可用于以编程方式请求令牌。它支持`password`，`authorization_code`，`client_credentials`，`refresh_token`和`urn:ietf:params:oauth:grant-type:device_code`的类型。此外，可以扩展令牌端点以支持扩展授权类型。

> **注意**
IdentityServer支持OpenID Connect和OAuth 2.0令牌请​​求参数的子集。有关完整列表，请参见[此处](http://openid.net/specs/openid-connect-core-1_0.html#TokenRequest)。

* **`client_id`**  
客户标识符（必填）

* **`client_secret`**  
客户端密钥可以在帖子正文中，也可以作为基本身份验证标头。可选的。

* **`grant_type`**  
`authorization_code`，`client_credentials`，`password`，`refresh_token`，`urn:ietf:params:oauth:grant-type:device_code`或自定义

* **`scope`**  
一个或多个注册范围。如果未指定，将发出所有明确允许范围的标记。

* **`redirect_uri`**  
`authorization_code`授权类型所需

* **`code`**  
授权代码（`authorization_code`授权类型需要）

* **`code_verifier`**  
PKCE证明密钥

* **`username`**  
资源所有者用户名（`password`授权类型所需）

* **`password`**  
资源所有者密码（`password`授予类型所需）

* **`acr_values`**  
允许传递`password`授权类型的其他身份验证相关信息- identityserver特殊情况下面的专有acr_values：
    * `idp:name_of_idp` 绕过login/home领域页面并将用户直接转发到选定的身份提供者（如果允许每个客户端配置）
    * `tenant:name_of_tenant` 可用于将租户名称传递给令牌端点

* **`refresh_token`**  
刷新令牌（`refresh_token`授予类型所需）

* **`device_code`**  
设备代码（`urn:ietf:params:oauth:grant-type:device_code`授权类型所需）

## 48.1 示例

```
POST /connect/token

    client_id=client1&
    client_secret=secret&
    grant_type=authorization_code&
    code=hdh922&
    redirect_uri=https://myapp.com/callback
```

（为了便于阅读，删除了表格编码并添加了换行符）

> **注意**
您可以使用[IdentityModel](https://github.com/IdentityModel/IdentityModel2)客户端库以编程方式从.NET代码访问令牌端点。有关更多信息，请查看IdentityModel[文档](https://github.com/thinksjay/IdentityModel/blob/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86%20%E5%8D%8F%E8%AE%AE%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%BA%93/%E7%AC%AC4%E7%AB%A0%20%E4%BB%A4%E7%89%8C%E7%AB%AF%E7%82%B9(Token%20Endpoint).md)。