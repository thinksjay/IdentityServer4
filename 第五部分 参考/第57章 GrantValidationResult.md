# 第57章 GrantValidationResult
该`GrantValidationResult`类模型补助确认为扩展授权和资源所有者密码授权的结果。

最常见的用法是使用身份验证（成功用例）：

``` C#
context.Result = new GrantValidationResult(
    subject: "818727",
    authenticationMethod: "custom",
    claims: optionalClaims);
``` 

...或使用错误和描述（失败用例）：

``` C#
context.Result = new GrantValidationResult(
    TokenRequestErrors.InvalidGrant,
    "invalid custom credential");
```  

在这两种情况下，您都可以传递将包含在令牌响应中的其他自定义值。