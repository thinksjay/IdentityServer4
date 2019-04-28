# 第62章 EntityFramework支持
为IdentityServer中的配置和操作数据扩展点提供了基于EntityFramework的实现。EntityFramework的使用允许任何EF支持的数据库与此库一起使用。

这个库的仓库位于[这里](https://github.com/IdentityServer/IdentityServer4.EntityFramework/)，NuGet包就在[这里](https://www.nuget.org/packages/IdentityServer4.EntityFramework)。

此库提供的功能分为两个主要区域：配置存储和操作存储支持。根据托管应用程序的需要，这两个不同的区域可以独立使用或一起使用。

## 62.1 配置存储支持客户端，资源和CORS设置
如果希望从EF支持的数据库加载客户端，标识资源，API资源或CORS数据（而不是使用内存配置），则可以使用配置存储。此支持提供了`IClientStore`和`IResourceStore`，以及`ICorsPolicyService`可扩展性点的实现。这些实现使用一个`DbContext`被调用的类`ConfigurationDbContext`来为数据库中的表建模。

要使用配置存储支持，请在调用`AddIdentityServer`后使用`AddConfigurationStore`扩展方法：

``` C#
public IServiceProvider ConfigureServices(IServiceCollection services)
{
    const string connectionString = @"Data Source=(LocalDb)\MSSQLLocalDB;database=IdentityServer4.EntityFramework-2.0.0;trusted_connection=yes;";
    var migrationsAssembly = typeof(Startup).GetTypeInfo().Assembly.GetName().Name;

    services.AddIdentityServer()
        // this adds the config data from DB (clients, resources, CORS)
        .AddConfigurationStore(options =>
        {
            options.ConfigureDbContext = builder =>
                builder.UseSqlServer(connectionString,
                    sql => sql.MigrationsAssembly(migrationsAssembly));
        });
}
```  

要配置配置存储，请使用`ConfigurationStoreOptions`的options对象传递给配置回调。

## 62.2 ConfigurationStoreOptions 
此选项类包含用于控制配置存储的属性和`ConfigurationDbContext`。

* **`ConfigureDbContext`**  
`Action<DbContextOptionsBuilder>`用作回调的类型委托来配置底层`ConfigurationDbContext`。`ConfigurationDbContext`如果直接使用EF，代理可以以相同的方式配置`AddDbContext`，这允许使用任何EF支持的数据库。

* **`DefaultSchema`**  
允许为`ConfigurationDbContext`中的所有表设置默认数据库模式名称。

``` C#
options.DefaultSchema = "myConfigurationSchema";
```  

如果需要更改迁移历史记录表的架构，可以将另一个操作链接到`UserSqlServer`：

``` C#
options.ConfigureDbContext = b =>
    b.UseSqlServer(connectionString,
        sql => sql.MigrationsAssembly(migrationsAssembly).MigrationsHistoryTable("MyConfigurationMigrationTable", "myConfigurationSchema"));
```  

## 62.3 操作存储支持授权授予，同意和令牌（刷新和引用）
如果希望从EF支持的数据库（而不是默认的内存数据库）加载授权授予，同意和令牌（刷新和引用），则可以使用操作存储。此支持提供了`IPersistedGrantStore`可扩展性点的实现。该实现使用一个`DbContext`被调用的类`PersistedGrantDbContext`来为数据库中的表建模。

要使用操作性存储支持，请在调用`AddIdentityServer`后使用`AddOperationalStore`扩展方法：

``` C#
public IServiceProvider ConfigureServices(IServiceCollection services)
{
    const string connectionString = @"Data Source=(LocalDb)\MSSQLLocalDB;database=IdentityServer4.EntityFramework-2.0.0;trusted_connection=yes;";
    var migrationsAssembly = typeof(Startup).GetTypeInfo().Assembly.GetName().Name;

    services.AddIdentityServer()
        // this adds the operational data from DB (codes, tokens, consents)
        .AddOperationalStore(options =>
        {
            options.ConfigureDbContext = builder =>
                builder.UseSqlServer(connectionString,
                    sql => sql.MigrationsAssembly(migrationsAssembly));

            // this enables automatic token cleanup. this is optional.
            options.EnableTokenCleanup = true;
            options.TokenCleanupInterval = 30; // interval in seconds
        });
}
```   

要配置操作存储，请使用`OperationalStoreOptions`的`options`对象传递给配置回调。

## 62.4 OperationalStoreOptions 
此选项类包含用于控制操作`PersistedGrantDbContext`存储的属性。

* **`ConfigureDbContext`**  
`Action<DbContextOptionsBuilder>`用作回调的类型委托来配置底层`PersistedGrantDbContext`。`PersistedGrantDbContext`如果直接使用EF，代理可以以相同的方式配置`AddDbContext`，这允许使用任何EF支持的数据库。

* **`DefaultSchema`**  
允许为`PersistedGrantDbContext`中的所有表设置默认数据库模式名称。

* **`EnableTokenCleanup`**  
指示是否将从数据库中自动清除过时条目。默认是false。

* **`TokenCleanupInterval`**  
令牌清理间隔（以秒为单位）。默认值为3600（1小时）。

## 62.5 跨不同版本的IdentityServer的数据库创建和模式更改
跨不同版本的IdentityServer（以及EF支持）很可能会更改数据库架构以适应新的和不断变化的功能。

我们不为创建数据库或将数据从一个版本迁移到另一个版本提供任何支持。您需要以组织认为合适的任何方式管理数据库创建，架构更改和数据迁移。

使用EF迁移是一种可行的方法。如果您确实希望使用迁移，请参阅[EF快速入门](https://github.com/thinksjay/IdentityServer4/blob/master/%E7%AC%AC%E4%BA%8C%E9%83%A8%E5%88%86%20%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/%E7%AC%AC15%E7%AB%A0%20%E4%BD%BF%E7%94%A8EntityFramework%20Core%E8%BF%9B%E8%A1%8C%E9%85%8D%E7%BD%AE%E5%92%8C%E6%93%8D%E4%BD%9C%E6%95%B0%E6%8D%AE.md)以获取有关如何入门的示例，或参阅有关EF迁移的Microsoft[文档](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/index)。

我们还为当前版本的数据库模式发布了示例[SQL脚本](https://github.com/IdentityServer/IdentityServer4.EntityFramework.Storage/tree/dev/migrations/SqlServer/Migrations)。