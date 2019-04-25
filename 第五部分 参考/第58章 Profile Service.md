# 第58章 Profile Service
IdentityServer通常在创建令牌或处理对userinfo或内省端点的请求时需要有关用户的身份信息。默认情况下，IdentityServer仅具有身份验证cookie中的声明，以便为此身份数据进行绘制。

将用户所需的所有可能声明放入cookie中是不切实际的，因此IdentityServer定义了一个可扩展点，允许根据用户需要动态加载声明。这个可扩展点是`IProfileService`开发人员通常可以实现此接口来访问包含用户身份数据的自定义数据库或API。

## 58.1 IProfileService APIs 
* **`GetProfileDataAsync`**  
期望为用户加载声明的API。它传递了一个实例`ProfileDataRequestContext`。

* **`IsActiveAsync`**  
预期用于指示当前是否允许用户获取令牌的API。它传递了一个实例`IsActiveContext`。  

## 58.2 ProfileDataRequestContext  
模拟用户声明的请求，并且是返回这些声明的工具。它包含以下属性：

* **`Subject`**  
该`ClaimsPrincipal`模型的用户。

* **`Client`**  
`Client`用于正被请求的声明。

* **`RequestedClaimTypes`**  
请求的声明类型集合。

* **`Caller`**  
正在请求声明的上下文的标识符（例如，身份令牌，访问令牌或用户信息端点）。常量`IdentityServerConstants.ProfileDataCallers`包含不同的常量值。

* **`IssuedClaims`**  
将返回`Claim`的列表。预计这将由自定义`IProfileService`实现填充。

* **`AddRequestedClaims`**  
扩展方法对`ProfileDataRequestContext`填充`IssuedClaims`，但首先根据索引过滤`RequestedClaimTypes`。

## 58.3 请求的范围和声明映射
客户端请求的范围控制用户声明在令牌中返回给客户端的内容。该`GetProfileDataAsync`方法负责根据`ProfileDataRequestContext`上的`RequestedClaimTypes`集合动态获取这些声明。

该`RequestedClaimTypes`集合是基于该定义的用户声明[资源](https://github.com/thinksjay/IdentityServer4/blob/master/%E4%B8%BB%E9%A2%98/%E7%AC%AC19%E7%AB%A0%20%E5%AE%9A%E4%B9%89%E8%B5%84%E6%BA%90.md)的范围进行建模。如果请求的范围是[身份资源](https://github.com/thinksjay/IdentityServer4/blob/master/%E5%8F%82%E8%80%83/%E7%AC%AC54%E7%AB%A0%20%E8%BA%AB%E4%BB%BD%E8%B5%84%E6%BA%90.md)，则将`RequestedClaimTypes`根据在`IdentityResource`中定义的用户声明类型填充声明中的声明。如果请求的范围是[API资源](https://github.com/thinksjay/IdentityServer4/blob/master/%E5%8F%82%E8%80%83/%E7%AC%AC55%E7%AB%A0%20%20API%E8%B5%84%E6%BA%90.md)，则将`RequestedClaimTypes`根据`ApiResource`中定义的用户声明类型填充声明中的声明`Scope`。

## 58.4 IsActiveContext 
对要确定的请求建模是当前允许用户获取令牌。它包含以下属性：

* **`Subject`**  
该`ClaimsPrincipal`模型的用户。

* **`Client`**  
`Client`用于正被请求的声明。

* **`Caller`**  
正在请求声明的上下文的标识符（例如，身份令牌，访问令牌或用户信息端点）。常量`IdentityServerConstants.ProfileDataCallers`包含不同的常量值。

* **`IsActive`**  
指示是否允许用户获取令牌的标志。预计这将由自定义`IProfileService`实现分配。