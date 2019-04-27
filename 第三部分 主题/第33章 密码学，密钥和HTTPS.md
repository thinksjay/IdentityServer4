# 第33章 密码学(Cryptography)，密钥(Keys)和HTTPS
IdentityServer依赖于几个加密机制来完成它的工作。

## 33.1 令牌签名和验证
IdentityServer需要非对称密钥对来签署和验证JWT。此密钥对可以是证书/私钥组合或原始RSA密钥。无论如何，它必须支持带有SHA256的RSA。

加载签名密钥和相应的验证部分是通过实现`ISigningCredentialStore`和`IValidationKeysStore`来完成的。如果要自定义加载密钥，可以实现这些接口并将其注册到DI。

DI构建器扩展有几种方便的方法来设置签名和验证密钥 - 请参阅[此处](https://github.com/thinksjay/IdentityServer4/blob/master/%E7%AC%AC%E4%B8%89%E9%83%A8%E5%88%86%20%E4%B8%BB%E9%A2%98/%E7%AC%AC18%20%E7%AB%A0%20%E5%90%AF%E5%8A%A8.md#182-%E5%AF%86%E9%92%A5%E6%9D%90%E6%96%99)。

## 33.2 签名密钥翻转
虽然一次只能使用一个签名密钥，但您可以将多个验证密钥发布到发现文档。这对于密钥翻转很有用。

翻转通常如下所示：

1. 您请求/创建新的密钥材料
2. 除了当前的验证密钥之外，还要发布新的验证密钥。您可以使用`AddValidationKeys`构建器扩展方法。
3. 所有客户端和API现在都有机会在下次更新发现文档的本地副本时了解新密钥
4. 在一定时间（例如24小时）之后，所有客户端和API现在都应接受旧密钥材料和新密钥材料
5. 只要你愿意，就可以保留旧的密钥材料，也许你有需要验证的长寿命代币
6. 当旧密钥材料不再使用时，将其退出
7. 所有客户端和API将在下次更新发现文档的本地副本时“忘记”旧密钥  

这要求客户端和API使用发现文档，并且还具有定期刷新其配置的功能。

## 33.3 数据保护
ASP.NET Core中的Cookie身份验证（或MVC中的防伪）使用ASP.NET Core数据保护功能。根据您的部署方案，这可能需要其他配置。有关更多信息，请参阅Microsoft[文档](https://docs.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview)。

## 33.4 HTTPS 
我们不强制使用HTTPS，但对于生产，它必须与IdentityServer进行每次交互。

