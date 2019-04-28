# 第50章 设备授权端点(Device Authorization Endpoint)
设备授权端点可用于请求设备和用户代码。此端点用于启动设备流授权过程。

> **注意**
终端会话端点的URL可通过[发现端点](https://github.com/thinksjay/IdentityServer4/blob/master/%E7%AC%AC%E4%B8%89%E9%83%A8%E5%88%86%20%E4%B8%BB%E9%A2%98/%E7%AC%AC42%E7%AB%A0%20%E5%8F%91%E7%8E%B0(discovery).md)获得。

* **`client_id`**  
客户标识符（必填）

* **`client_secret`**  
客户端密钥可以在帖子正文中，也可以作为基本身份验证标头。可选的。

* **`scope`**  
一个或多个注册范围。如果未指定，将发出所有明确允许范围的标记。

示例

```
POST /connect/deviceauthorization

    client_id=client1&
    client_secret=secret&
    scope=openid api1
```

（为了便于阅读，删除了表格编码并添加了换行符）

> **注意**
您可以使用[IdentityModel](https://github.com/IdentityModel/IdentityModel2)客户端库以编程方式从.NET代码访问设备授权终结点。有关更多信息，请查看IdentityModel[文档](https://github.com/thinksjay/IdentityModel/blob/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86%20%E5%8D%8F%E8%AE%AE%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%BA%93/%E7%AC%AC9%E7%AB%A0%20%E8%AE%BE%E5%A4%87%E6%8E%88%E6%9D%83%E7%AB%AF%E7%82%B9(Device%20Authorization%20Endpoint).md)。