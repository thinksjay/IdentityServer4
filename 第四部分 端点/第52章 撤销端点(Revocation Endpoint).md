# 第52章 撤销端点(Revocation Endpoint)
此端点允许撤消访问令牌（仅限引用令牌）和刷新令牌。它实现了令牌撤销规范（[RFC 7009](https://tools.ietf.org/html/rfc7009)）。

* **`token`**  
要撤销的令牌（必填）

* **`token_type_hint`**  
`access_token`或`refresh_token`（可选）

示例

```
POST /connect/revocation HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW

token=45ghiukldjahdnhzdauz&token_type_hint=refresh_token
```

> **注意**
您可以使用[IdentityModel](https://github.com/IdentityModel/IdentityModel2)客户端库以编程方式从.NET代码访问吊销端点。有关更多信息，请查看IdentityModel[文档](https://github.com/thinksjay/IdentityModel/blob/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86%20%E5%8D%8F%E8%AE%AE%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%BA%93/%E7%AC%AC6%E7%AB%A0%20%E4%BB%A4%E7%89%8C%E6%92%A4%E9%94%80%E7%AB%AF%E7%82%B9(Token%20Revocation%20Endpoint).md)。