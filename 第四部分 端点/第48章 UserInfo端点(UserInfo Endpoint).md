# 第48章 UserInfo端点(UserInfo Endpoint)
UserInfo端点可用于检索有关用户的身份信息（请参阅[规范](http://openid.net/specs/openid-connect-core-1_0.html#UserInfo)）。

调用者需要发送代表用户的有效访问令牌。根据授予的范围，UserInfo端点将返回映射的声明（至少需要*openid*作用域）。

示例

```
GET /connect/userinfo
Authorization: Bearer <access_token>
HTTP/1.1 200 OK
Content-Type: application/json

{
    "sub": "248289761001",
    "name": "Bob Smith",
    "given_name": "Bob",
    "family_name": "Smith",
    "role": [
        "user",
        "admin"
    ]
}
```  

> **注意**
您可以使用[IdentityModel](https://github.com/IdentityModel/IdentityModel2)客户端库以编程方式从.NET代码访问userinfo端点。有关更多信息，请查看IdentityModel[文档](https://github.com/thinksjay/IdentityModel/blob/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86%20%E5%8D%8F%E8%AE%AE%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%BA%93/%E7%AC%AC7%E7%AB%A0%20UserInfo%E7%AB%AF%E7%82%B9(UserInfo%20Endpoint).md)。