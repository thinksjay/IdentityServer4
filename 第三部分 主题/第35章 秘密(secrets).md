# 第35章 秘密(secrets)
在某些情况下，客户端需要使用身份服务器进行身份验证，例如

* 在令牌端点请求令牌的机密应用程序（也称为客户端）
* API在内省端点验证引用令牌

为此，您可以将秘密列表分配给客户端或API资源。

秘密解析和验证是身份服务器中的可扩展点，开箱即用它支持共享机密以及通过基本身份验证头或POST主体传输共享机密。

## 35.1 创建共享密钥
以下代码设置散列共享密钥：

``` C#
var secret = new Secret("secret".Sha256());
```  

现在可以将此秘密分配给`Client`或`ApiResource`。请注意，它们不仅支持单个秘密，还支持多个秘密。这对于秘密翻转和轮换非常有用：

``` C#
var client = new Client
{
    ClientId = "client",
    ClientSecrets = new List<Secret> { secret },

    AllowedGrantTypes = GrantTypes.ClientCredentials,
    AllowedScopes = new List<string>
    {
        "api1", "api2"
    }
};
```  

实际上，您还可以为秘密分配说明和到期日期。该描述将用于记录，以及用于实施秘密生存期的到期日期：

``` C#
var secret = new Secret(
    "secret".Sha256(),
    "2016 secret",
    new DateTime(2016, 12, 31));
```  

## 35.2 使用共享密钥进行身份验证
您可以将Client id/secret组合作为POST正文的一部分发送：

```
POST /connect/token

client_id=client1&
client_secret=secret&
...
```  

..或作为基本认证头：

```
POST /connect/token

Authorization: Basic xxxxx

...
```

您可以使用以下C＃代码手动创建基本身份验证标头：

``` C#
var credentials = string.Format("{0}:{1}", clientId, clientSecret);
var headerValue = Convert.ToBase64String(Encoding.UTF8.GetBytes(credentials));

var client = new HttpClient();
client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", headerValue);
```  

该[IdentityModel](https://github.com/IdentityModel/IdentityModel2)库有一个叫做辅助类`TokenClient`和`IntrospectionClient`封装认证和协议消息。

## 35.3 超越共享秘密
还有其他技术来验证客户端，例如基于公钥/私钥密码术。IdentityServer包括对私钥JWT客户端机密的支持（请参阅[RFC 7523](https://tools.ietf.org/html/rfc7523)）。

秘密可扩展性通常由三件事组成：

* 一个秘密的定义
* 一个知道如何从传入请求中提取秘密的秘密解析器
* 一个秘密验证器，知道如何根据定义验证解析的秘密  

秘密解析器和验证器是`ISecretParser`和`ISecretValidator`接口的实现。要使它们可用于IdentityServer，您需要将它们注册到DI容器，例如：

``` C#
builder.AddSecretParser<JwtBearerClientAssertionSecretParser>()
builder.AddSecretValidator<PrivateKeyJwtSecretValidator>()
```  

我们的默认私钥JWT秘密验证器期望完整（叶）证书作为秘密定义的base64。然后，此证书将用于验证自签名JWT上的签名，例如：

``` C#
var client = new Client
{
    ClientId = "client.jwt",
    ClientSecrets =
    {
        new Secret
        {
            Type = IdentityServerConstants.SecretTypes.X509CertificateBase64,
            Value = "MIIDATCCAe2gAwIBAgIQoHUYAquk9rBJcq8W+F0FAzAJBgUrDgMCHQUAMBIxEDAOBgNVBAMTB0RldlJvb3QwHhcNMTAwMTIwMjMwMDAwWhcNMjAwMTIwMjMwMDAwWjARMQ8wDQYDVQQDEwZDbGllbnQwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDSaY4x1eXqjHF1iXQcF3pbFrIbmNw19w/IdOQxbavmuPbhY7jX0IORu/GQiHjmhqWt8F4G7KGLhXLC1j7rXdDmxXRyVJBZBTEaSYukuX7zGeUXscdpgODLQVay/0hUGz54aDZPAhtBHaYbog+yH10sCXgV1Mxtzx3dGelA6pPwiAmXwFxjJ1HGsS/hdbt+vgXhdlzud3ZSfyI/TJAnFeKxsmbJUyqMfoBl1zFKG4MOvgHhBjekp+r8gYNGknMYu9JDFr1ue0wylaw9UwG8ZXAkYmYbn2wN/CpJl3gJgX42/9g87uLvtVAmz5L+rZQTlS1ibv54ScR2lcRpGQiQav/LAgMBAAGjXDBaMBMGA1UdJQQMMAoGCCsGAQUFBwMCMEMGA1UdAQQ8MDqAENIWANpX5DZ3bX3WvoDfy0GhFDASMRAwDgYDVQQDEwdEZXZSb290ghAsWTt7E82DjU1E1p427Qj2MAkGBSsOAwIdBQADggEBADLje0qbqGVPaZHINLn+WSM2czZk0b5NG80btp7arjgDYoWBIe2TSOkkApTRhLPfmZTsaiI3Ro/64q+Dk3z3Kt7w+grHqu5nYhsn7xQFAQUf3y2KcJnRdIEk0jrLM4vgIzYdXsoC6YO+9QnlkNqcN36Y8IpSVSTda6gRKvGXiAhu42e2Qey/WNMFOL+YzMXGt/nDHL/qRKsuXBOarIb++43DV3YnxGTx22llhOnPpuZ9/gnNY7KLjODaiEciKhaKqt/b57mTEz4jTF4kIg6BP03MUfDXeVlM1Qf1jB43G2QQ19n5lUiqTpmQkcfLfyci2uBZ8BkOhXr3Vk9HIk/xBXQ="
        }
    },

    AllowedGrantTypes = GrantTypes.ClientCredentials,
    AllowedScopes = { "api1", "api2" }
};
``` 

您可以实现自己的秘密验证器（或扩展我们的秘密验证器）来实现例如链信任验证。