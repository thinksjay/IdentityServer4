# 第55章 API资源
此类建模API资源。   

* **`Enabled`**  
指示此资源是否已启用且可以请求。默认为true。

* **`Name`**  
API的唯一名称。此值用于内省身份验证，并将添加到传出访问令牌的受众。

* **`DisplayName`**  
该值可以在例如同意屏幕上使用。

* **`Description`**  
该值可以在例如同意屏幕上使用。

* **`ApiSecrets`**  
API密钥用于内省端点。API可以使用API​​名称和密钥进行内省验证。

* **`UserClaims`**  
应包含在访问令牌中的关联用户声明类型的列表。

* **`Scopes`**  
API必须至少有一个范围。每个范围可以有不同的设置。   

## 55.1 范围
在简单的情况下，API只有一个范围。但是在某些情况下，您可能希望细分API的功能，并让不同的客户端访问不同的部分。  

* **`Name`**  
范围的唯一名称。这是客户端将用于授权/令牌请求中的scope参数的值。

* **`DisplayName`**  
该值可以在例如同意屏幕上使用。

* **`Description`**  
该值可以在例如同意屏幕上使用。

* **`Required`**  
指定用户是否可以在同意屏幕上取消选择范围（如果同意屏幕要实现此类功能）。默认为false。

* **`Emphasize`**  
指定同意屏幕是否会强调此范围（如果同意屏幕要实现此功能）。将此设置用于敏感或重要范围。默认为false。

* **`ShowInDiscoveryDocument`**  
指定此范围是否显示在发现文档中。默认为true。

* **`UserClaims`**  
应包含在访问令牌中的关联用户声明类型的列表。此处指定的声明将添加到为API指定的声明列表中。

## 55.2 便利构造函数行为
只是关于为`ApiResource`类提供的构造函数的注释。  

要完全控制数据`ApiResource`，请使用不带参数的默认构造函数。如果要为每个API配置多个范围，可以使用此方法。例如：   

``` C#
new ApiResource
{
    Name = "api2",

    Scopes =
    {
        new Scope()
        {
            Name = "api2.full_access",
            DisplayName = "Full access to API 2"
        },
        new Scope
        {
            Name = "api2.read_only",
            DisplayName = "Read only access to API 2"
        }
    }
}
```   

对于每个API只需要一个范围的简单方案，则提供了几个接受a的便捷构造函数`name`。例如：

``` C#
new ApiResource("api1", "Some API 1")
```   

使用便利构造函数等同于：  

``` C#
new ApiResource
{
    Name = "api1",
    DisplayName = "Some API 1",

    Scopes =
    {
        new Scope()
        {
            Name = "api1",
            DisplayName = "Some API 1"
        }
    }
}
```