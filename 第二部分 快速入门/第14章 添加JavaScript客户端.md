# 第14章 添加JavaScript客户端
本快速入门将展示如何构建基于浏览器的JavaScript客户端应用程序（有时称为“ SPA ”）。   

用户将登录IdentityServer，使用IdentityServer发出的访问令牌调用We​​b API，并注销IdentityServer。所有这些都将来自浏览器中运行的JavaScript。   

## 14.1 JavaScript客户端的新项目
为JavaScript应用程序创建一个新项目。它可以只是一个空的Web项目，一个空的ASP.NET Core应用程序，或者像Node.js应用程序。本快速入门将使用ASP.NET Core应用程序。   

在~/src目录中创建一个新的“空”ASP.NET Core Web应用程序。您可以使用Visual Studio或从命令行执行此操作：   
``` shell
md JavaScriptClient
cd JavaScriptClient
dotnet new web
```   

## 14.2 修改宿主
修改JavaScriptClient项目以在端口5003上运行。   

## 14.3 添加静态文件中间件
鉴于该项目旨在运行客户端，我们所需要的只是ASP.NET Core提供构成我们应用程序的静态HTML和JavaScript文件。静态文件中间件旨在实现此目的。   

在`Startup.cs`中的`Configure`方法注册静态文件中间件（同时删除其他所有内容）：   

``` C#
public void Configure(IApplicationBuilder app)
{
    app.UseDefaultFiles();
    app.UseStaticFiles();
}
```   

此中间件现在将从应用程序的~/wwwroot文件夹中提供静态文件。这是我们将放置HTML和JavaScript文件的地方。如果项目中不存在该文件夹，请立即创建。

## 14.4 引用OIDC客户端
在基于ASP\.NET Core MVC的客户端项目的前一个快速入门中，我们使用库来处理OpenID Connect协议。在JavaScriptClient项目的快速入门中，我们需要一个类似的库，除了一个在JavaScript中运行并且设计为在浏览器中运行的库。[oidc-client库](https://github.com/IdentityModel/oidc-client-js)就是这样一个库，可以通过[NPM](https://github.com/IdentityModel/oidc-client-js)，[Bower](https://bower.io/search/?q=oidc-client)等获取到该库，还可以直接从[github](https://github.com/IdentityModel/oidc-client-js/tree/release/dist)下载该库。   

### 14.4.1 NPM
如果要使用NPM下载oidc-client，请从JavaScriptClient项目目录运行以下命令：  

``` shell
npm i oidc-client
copy node_modules\oidc-client\dist\* wwwroot
```   

这将在本地下载最新的oidc-client软件包，然后将相关的JavaScript文件复制到~/wwwroot中，以便它们可以由您的应用程序提供。   

### 14.4.2 手动下载
如果您只想手动下载oidc-client JavaScript文件，请浏览到[GitHub](https://github.com/IdentityModel/oidc-client-js/tree/master/dist)存储库 并下载JavaScript文件。下载后，将它们复制到~/wwwroot中，以便它们可以提供给您的应用程序。   

## 14.5 添加HTML和JavaScript文件
接下来是将您的HTML和JavaScript文件添加到~/ wwwroot。我们将有两个HTML文件和一个特定于应用程序的JavaScript文件（除了*oidc-client.js*库）。在~/wwwroot中，添加一个名为*index.html*和*callback.html*的HTML文件，并添加一个名为*app.js*的JavaScript文件。   

### 14.5.1 index.html
这将是我们应用程序中的主页面。它将只包含用于登录，注销和调用Web API的按钮的HTML。它还将包含`<script>`标记以包含我们的两个JavaScript文件。它还将包含`<pre>`用于向用户显示消息的用途。   

它应该如下所示：   

``` html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title></title>
</head>
<body>
    <button id="login">Login</button>
    <button id="api">Call API</button>
    <button id="logout">Logout</button>

    <pre id="results"></pre>

    <script src="oidc-client.js"></script>
    <script src="app.js"></script>
</body>
</html>
```   

### 14.5.2 app.js
这将包含我们的应用程序的主要代码。第一件事是添加一个帮助函数来将消息记录到`<pre>`：   

``` javascript
function log() {
    document.getElementById('results').innerText = '';

    Array.prototype.forEach.call(arguments, function (msg) {
        if (msg instanceof Error) {
            msg = "Error: " + msg.message;
        }
        else if (typeof msg !== 'string') {
            msg = JSON.stringify(msg, null, 2);
        }
        document.getElementById('results').innerHTML += msg + '\r\n';
    });
}
```   

接下来，添加代码以将`click`事件处理程序注册到三个按钮：

``` javascript
document.getElementById("login").addEventListener("click", login, false);
document.getElementById("api").addEventListener("click", api, false);
document.getElementById("logout").addEventListener("click", logout, false);
```   

接下来，我们可以使用`UserManager`类从OIDC客户端库来管理OpenID Connect协议。它需要MVC Client中必需的类似配置（尽管具有不同的值）。添加此代码以配置和实例化UserManager：  

``` javascript
var config = {
    authority: "http://localhost:5000",
    client_id: "js",
    redirect_uri: "http://localhost:5003/callback.html",
    response_type: "code",
    scope:"openid profile api1",
    post_logout_redirect_uri : "http://localhost:5003/index.html",
};
var mgr = new Oidc.UserManager(config);
```   

接下来，`UserManager`提供`getUserAPI`以了解用户是否登录到JavaScript应用程序。它使用JavaScript `Promise`以异步方式返回结果。返回的`User`对象具有`profile`包含用户声明的属性。添加此代码以检测用户是否已登录JavaScript应用程序：   

``` javascript
mgr.getUser().then(function (user) {
    if (user) {
        log("User logged in", user.profile);
    }
    else {
        log("User not logged in");
    }
});
```   

接下来，我们要实现的`login`，`api`和`logout`功能。在`UserManager`提供了`signinRedirect`登录用户，并且`signoutRedirect`以注销用户。`User`我们在上面的代码中获得的对象还具有`access_token`可用于向Web API进行身份验证的属性。在`access_token`将被传递给通过网络API 授权与头承载方案。添加此代码以在我们的应用程序中实现这三个功能：   

``` javascript
function login() {
    mgr.signinRedirect();
}

function api() {
    mgr.getUser().then(function (user) {
        var url = "http://localhost:5001/identity";

        var xhr = new XMLHttpRequest();
        xhr.open("GET", url);
        xhr.onload = function () {
            log(xhr.status, JSON.parse(xhr.responseText));
        }
        xhr.setRequestHeader("Authorization", "Bearer " + user.access_token);
        xhr.send();
    });
}

function logout() {
    mgr.signoutRedirect();
}
```   

> **注意**
有关如何创建上述代码中使用的api的信息，请参阅客户端凭据快速入门。   

### 14.5.3 callback.html

`redirect_uri`用户登录IdentityServer后，此HTML文件是指定的页面。它将完成与IdentityServer的OpenID Connect协议登录握手。这个代码全部由`UserManager`我们之前使用的类提供。登录完成后，我们可以将用户重定向回主index.html页面。添加此代码以完成登录过程：   

``` html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title></title>
</head>
<body>
    <script src="oidc-client.js"></script>
    <script>
        new Oidc.UserManager({response_mode:"query"}).signinRedirectCallback().then(function() {
            window.location = "index.html";
        }).catch(function(e) {
            console.error(e);
        });
    </script>
</body>
</html>
```   

## 14.6 客户注册加入IdentityServer的JavaScript客户端
既然客户端应用程序已经准备就绪，我们需要在IdentityServer中为这个新的JavaScript客户端定义一个配置条目。在IdentityServer项目中找到客户端配置（在*Config.cs*中）。将新客户端添加到我们的新JavaScript应用程序的列表中。它应该具有下面列出的配置：

``` C#
// JavaScript Client
new Client
{
    ClientId = "js",
    ClientName = "JavaScript Client",
    AllowedGrantTypes = GrantTypes.Code,
    RequirePkce = true,
    RequireClientSecret = false,

    RedirectUris =           { "http://localhost:5003/callback.html" },
    PostLogoutRedirectUris = { "http://localhost:5003/index.html" },
    AllowedCorsOrigins =     { "http://localhost:5003" },

    AllowedScopes =
    {
        IdentityServerConstants.StandardScopes.OpenId,
        IdentityServerConstants.StandardScopes.Profile,
        "api1"
    }
}
```   

## 14.7 允许Ajax以CORS的方式调用WebAPI
最后一点配置是在Web API项目中配置CORS。这将允许从http://localhost:5003到http://localhost:5001进行Ajax调用。

### 14.7.1 配置CORS

`ConfigureServices`在*Startup.cs*中将CORS服务添加到依赖注入系统：   

``` C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvcCore()
        .AddAuthorization()
        .AddJsonFormatters();

    services.AddAuthentication("Bearer")
        .AddIdentityServerAuthentication(options =>
        {
            options.Authority = "http://localhost:5000";
            options.RequireHttpsMetadata = false;

            options.ApiName = "api1";
        });

    services.AddCors(options =>
    {
        // this defines a CORS policy called "default"
        options.AddPolicy("default", policy =>
        {
            policy.WithOrigins("http://localhost:5003")
                .AllowAnyHeader()
                .AllowAnyMethod();
        });
    });
}
```   

将CORS中间件添加到管道中Configure：   

``` C#
public void Configure(IApplicationBuilder app)
{
    app.UseCors("default");

    app.UseAuthentication();

    app.UseMvc();
}
```   

## 14.8 运行JavaScript应用程序
现在您应该能够运行JavaScript客户端应用程序：   
<div align="center">
<image src="https://identityserver4.readthedocs.io/en/latest/_images/6_not_logged_in.png">
</div>   

单击“登录”按钮以对用户进行签名。一旦用户返回到JavaScript应用程序，您应该看到他们的个人资料信息：   

<div align="center">
<image src="https://identityserver4.readthedocs.io/en/latest/_images/6_logged_in.png">
</div>   

然后单击“API”按钮以调用Web API：   

<div align="center">
<image src="https://identityserver4.readthedocs.io/en/latest/_images/6_api_results.png">
</div>   

最后点击“退出”以签署用户。   

<div align="center">
<image src="https://identityserver4.readthedocs.io/en/latest/_images/6_signed_out.png">   

您现在可以开始使用IdentityServer进行登录，注销和验证对Web API的调用的JavaScript客户端应用程序。

