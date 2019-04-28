# 第46章 发现端点(Discovery Endpoint)
发现端点可用于检索有关IdentityServer的元数据 - 它返回发布者名称，密钥材料，支持的范围等信息。有关详细信息，请参阅[规范](https://openid.net/specs/openid-connect-discovery-1_0.html)。

发现端点可通过*/.well-known/openid-configuration*相对于基地址获得，例如：

```
https://demo.identityserver.io/.well-known/openid-configuration
```  

> **注意**
您可以使用[IdentityModel](https://github.com/IdentityModel/IdentityModel2)客户端库以编程方式从.NET代码访问发现端点。有关更多信息，请查看IdentityModel[文档](https://github.com/thinksjay/IdentityModel/blob/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86%20%E5%8D%8F%E8%AE%AE%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%BA%93/%E7%AC%AC1%E7%AB%A0%20%E5%8F%91%E7%8E%B0%E7%AB%AF%E7%82%B9(Discovery%20Endpoint).md)。