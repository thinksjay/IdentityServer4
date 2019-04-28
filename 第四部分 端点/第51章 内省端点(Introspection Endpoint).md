# 第51章 内省端点(Introspection Endpoint)
内省端点是[RFC 7662](https://tools.ietf.org/html/rfc7662)的实现。

它可用于验证引用令牌（如果消费者不支持适当的JWT或加密库，则可以使用JWT）。内省端点需要身份验证 - 因为内省端点的客户端是API，您可以在其上配置秘密`ApiResource`。

示例

```
POST /connect/introspect
Authorization: Basic xxxyyy

token=<token>
```   

成功的响应将返回状态代码200以及活动或非活动令牌：

``` json
{
    "active": true,
    "sub": "123"
}
```   

未知或过期的令牌将被标记为无效：

``` json
{
    "active": false,
}
```  

无效请求将返回400，未授权请求401。

> **注意**
您可以使用[IdentityModel](https://github.com/IdentityModel/IdentityModel2)客户端库以编程方式从.NET代码访问内省端点。有关更多信息，请查看IdentityModel[文档](https://github.com/thinksjay/IdentityModel/blob/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86%20%E5%8D%8F%E8%AE%AE%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%BA%93/%E7%AC%AC5%E7%AB%A0%20%E4%BB%A4%E7%89%8C%E8%87%AA%E7%9C%81%E7%AB%AF%E7%82%B9(Token%20Introspection%20Endpoint).md)。