# 第42章 发现(discovery)
可以在*https://baseaddress/.well-known/openid-configuration*找到发现文档。它包含有关IdentityServer的端点，密钥材料和功能的信息。

默认情况下，所有信息都包含在发现文档中，但通过使用配置选项，您可以隐藏各个部分，例如：

``` C#
services.AddIdentityServer(options =>
{
    options.Discovery.ShowIdentityScopes = false;
    options.Discovery.ShowApiScopes = false;
    options.Discovery.ShowClaims = false;
    options.Discovery.ShowExtensionGrantTypes = false;
});
```  

## 42.1扩展发现
您可以向发现文档添加自定义条目，例如：

``` C#
services.AddIdentityServer(options =>
{
    options.Discovery.CustomEntries.Add("my_setting", "foo");
    options.Discovery.CustomEntries.Add("my_complex_setting",
        new
        {
            foo = "foo",
            bar = "bar"
        });
});
```   

当您添加以~开头的自定义值时，它将扩展到IdentityServer基址以下的绝对路径，例如：

``` C#
options.Discovery.CustomEntries.Add("my_custom_endpoint", "~/custom");
```   

如果要完全控制发现（和jwks）文档的呈现，可以实现`IDiscoveryResponseGenerator 接口（或从我们的默认实现派生）。