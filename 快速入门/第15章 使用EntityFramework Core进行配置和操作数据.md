# 第15章 使用EntityFramework Core进行配置和操作数据
IdentityServer旨在实现可扩展性，其中一个可扩展点是用于IdentityServer所需数据的存储机制。本快速入门展示了如何配置IdentityServer以使用EntityFramework Core（EF）作为此数据的存储机制（而不是使用我们迄今为止使用的内存中实现）。   

> **注意**
除了手动配置EF支持外，还有一个IdentityServer模板可用于创建具有EF支持的新项目。使用创建它。有关更多信息，请参见[此处](https://www.github.com/IdentityServer/IdentityServer4.Templates)`dotnet new is4ef`  

## 15.1 IdentityServer4.EntityFramework 
我们要移动到数据库有两种类型的数据：   

* 配置数据
* 运营数据   

配置数据是有关资源和客户端的配置信息。操作数据是IdentityServer在使用时生成的信息，例如令牌，代码和同意。这些存储使用接口建模，我们在`IdentityServer4.EntityFramework.Storage` Nuget包中提供这些接口的EF实现。   

注册我们的EF实现的扩展方法包含在IdentityServer4.EntityFramework Nuget包中（引用`IdentityServer4.EntityFramework.Storage`）。立即从IdentityServer项目添加对`IdentityServer4.EntityFramework` Nuget包的引用：   

``` shell
cd quickstart/src/IdentityServer
dotnet add package IdentityServer4.EntityFramework
```   

## 15.2 使用的SqlServer 
鉴于EF的灵活性，您可以使用任何EF支持的数据库。对于本快速入门，我们将使用Visual Studio附带的SQLD的LocalDb版本。   

## 15.3 数据库架构更改和使用EF迁移
该`IdentityServer4.EntityFramework.Storage`包中包含从IdentityServer的模型映射实体类。作为IdentityServer模型变化，所以会在实体类`IdentityServer4.EntityFramework.Storage`。当您使用`IdentityServer4.EntityFramework.Storage`并随着时间的推移升级时，您将负责自己的数据库架构以及实体类更改时该架构所需的更改。管理这些更改的一种方法是使用[EF迁移](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/index)，此快速入门将显示如何完成此操作。如果迁移不是您的首选项，那么您可以以任何您认为合适的方式管理架构更改。   

> **注意**
为`IdentityServer4.EntityFramework.Storage`中的实体维护SqlServer的最新SQL脚本。他们就在[这里](https://github.com/IdentityServer/IdentityServer4.EntityFramework.Storage/tree/master/migrations/SqlServer/Migrations)。

## 15.4 配置存储
接下来的步骤是，以取代当前`AddInMemoryClients`，`AddInMemoryIdentityResources`和`AddInMemoryApiResources`在在*Startup.cs*方法`ConfigureServices`。我们将使用以下代码替换它们：   

``` C#
const string connectionString = @"Data Source=(LocalDb)\MSSQLLocalDB;database=IdentityServer4.Quickstart.EntityFramework-2.0.0;trusted_connection=yes;";
var migrationsAssembly = typeof(Startup).GetTypeInfo().Assembly.GetName().Name;

// configure identity server with in-memory stores, keys, clients and scopes
services.AddIdentityServer()
    .AddTestUsers(Config.GetUsers())
    // this adds the config data from DB (clients, resources)
    .AddConfigurationStore(options =>
    {
        options.ConfigureDbContext = b =>
            b.UseSqlServer(connectionString,
                sql => sql.MigrationsAssembly(migrationsAssembly));
    })
    // this adds the operational data from DB (codes, tokens, consents)
    .AddOperationalStore(options =>
    {
        options.ConfigureDbContext = b =>
            b.UseSqlServer(connectionString,
                sql => sql.MigrationsAssembly(migrationsAssembly));

        // this enables automatic token cleanup. this is optional.
        options.EnableTokenCleanup = true;
    });
```   

您可能需要将这些命名空间添加到文件中：   

``` C#
using Microsoft.EntityFrameworkCore;
using System.Reflection;
```   

上面的代码是对连接字符串进行硬编码，如果您愿意，您可以随意更改。   

`AddConfigurationStore`和`AddOperationalStore`注册EF支持的存储实现。   

在添加存储的调用内部，`ConfigureDbContext`属性的赋值注册委托以配置数据库提供程序`DbContextOptionsBuilder`。在这种情况下，我们调用`UseSqlServer`注册SQLServer。您也可以看出，这是使用连接字符串的位置。   

最后，假设将使用EF迁移（至少对于此快速入门），调用将`MigrationsAssembly`用于通知EF将包含迁移代码的主机项目（这是必需的，因为它与包含`DbContext`类的程序集不同）。   

我们接下来会添加迁移。   

## 15.5 添加迁移
要创建迁移，请在IdentityServer项目目录中打开命令提示符。在命令提示符下运行以下两个命令：   

``` shell
dotnet ef migrations add InitialIdentityServerPersistedGrantDbMigration -c PersistedGrantDbContext -o Data/Migrations/IdentityServer/PersistedGrantDb
dotnet ef migrations add InitialIdentityServerConfigurationDbMigration -c ConfigurationDbContext -o Data/Migrations/IdentityServer/ConfigurationDb
```   

您现在应该在项目中看到~/Data/Migrations/IdentityServer文件夹。其中包含新创建的迁移的代码。   

## 15.6 初始化数据库
现在我们已经进行了迁移，我们可以编写代码来从迁移中创建数据库。我们还将使用我们在之前的快速入门中定义的内存配置数据来为数据库设定种子。   

> **注意**
本快速入门中使用的方法用于简化IdentityServer的启动和运行。您应该设计适合您的体系结构的数据库创建和维护策略。   

在*Startup.cs*中添加此方法以帮助初始化数据库：   

``` C#
private void InitializeDatabase(IApplicationBuilder app)
{
    using (var serviceScope = app.ApplicationServices.GetService<IServiceScopeFactory>().CreateScope())
    {
        serviceScope.ServiceProvider.GetRequiredService<PersistedGrantDbContext>().Database.Migrate();

        var context = serviceScope.ServiceProvider.GetRequiredService<ConfigurationDbContext>();
        context.Database.Migrate();
        if (!context.Clients.Any())
        {
            foreach (var client in Config.GetClients())
            {
                context.Clients.Add(client.ToEntity());
            }
            context.SaveChanges();
        }

        if (!context.IdentityResources.Any())
        {
            foreach (var resource in Config.GetIdentityResources())
            {
                context.IdentityResources.Add(resource.ToEntity());
            }
            context.SaveChanges();
        }

        if (!context.ApiResources.Any())
        {
            foreach (var resource in Config.GetApis())
            {
                context.ApiResources.Add(resource.ToEntity());
            }
            context.SaveChanges();
        }
    }
}
```   

上面的代码可能需要将这些命名空间添加到您的文件中：   
``` C# 
using System.Linq;
using IdentityServer4.EntityFramework.DbContexts;
using IdentityServer4.EntityFramework.Mappers;
```   

然后我们可以从`Configure`方法中调用它：  

``` C#
public void Configure(IApplicationBuilder app)
{
    // this will do the initial DB population
    InitializeDatabase(app);

    // the rest of the code that was already here
    // ...
}
```   

现在，如果运行IdentityServer项目，则应创建数据库并使用快速入门配置数据进行种子设定。您应该能够使用SQL Server Management Studio或Visual Studio来连接和检查数据。   

<div align="center">
<image src="https://identityserver4.readthedocs.io/en/latest/_images/7_database.png">
</div>   

> **注意**
上面的`InitializeDatabase`辅助API可以方便地为数据库设定种子，但是这种方法并不适合每次运行应用程序时执行。填充数据库后，请考虑删除对API的调用。   

## 15.7 运行客户端应用程序
您现在应该能够运行任何现有的客户端应用程序并登录，获取令牌并调用API - 所有这些都基于数据库配置。   

> **注意**
本节中的代码仍然依赖于`Config.cs`及其虚构用户Alice和Bob。如果您的用户列表很简短且静态，则调整后的`Config.cs`版本可能就足够了，但您可能希望在数据库中动态管理更大且更流畅的用户列表。ASP.NET Identity是一个需要考虑的选项，下一节的快速入门列出了此解决方案的示例实现。