# 第37章 资源所有者密码验证(Resource Owner Password Validation)
如果要使用OAuth 2.0资源所有者密码凭据授权（aka `password`），则需要实现并注册`IResourceOwnerPasswordValidator`接口：

``` C#
public interface IResourceOwnerPasswordValidator
{
    /// <summary>
    /// Validates the resource owner password credential
    /// </summary>
    /// <param name="context">The context.</param>
    Task ValidateAsync(ResourceOwnerPasswordValidationContext context);
}
```  

在上下文中，您将找到已解析的协议参数，如UserName和Password，以及原始请求，如果您想查看其他输入数据。

然后，您的工作是实施密码验证并相应地设置`Result`上下文。请参阅[GrantValidationResult](https://github.com/thinksjay/IdentityServer4/blob/master/%E7%AC%AC%E4%BA%94%E9%83%A8%E5%88%86%20%E5%8F%82%E8%80%83/%E7%AC%AC57%E7%AB%A0%20GrantValidationResult.md)文档。