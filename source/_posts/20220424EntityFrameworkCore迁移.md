---
title: EntityFrameworkCore迁移（Migration)支持多数据库
date: 2022-04-24 15:08:44
layout: 2022-04-24 15:08:44
id: 0044
tags:
 - EntityFrameworkCore
categories:
- AspNetCore
- EntityFrameworkCore

---
 
 在博客园上跟着一位博主（<a href="https://www.cnblogs.com/code4nothing/">CODE4NOTHING</a>)的文章学习使用<a href="https://www.cnblogs.com/code4nothing/p/build-todolist-index.html">使用.NET6开发WebApi项目</a>，最近想来将博主放在Github上的源码拷贝到本地来学习。但本地没有装SqlServer，于是想将数据库切换到Sqlite上。于是想到修改源码，让原来的项目支持多数据库。所以有了这篇踩坑记。

<!--more-->

## 认识迁移（Migration）
以我对Migration浅显的认识，它类似于一个版本管理，管理的内容则是对数据库的修改操作。在没有ORM框架时，我们通常会把创建数据库的脚本文件随项目一起分发。当项目修改时，我们就要更新数据库的脚本文件。有了Migration，每次对数据库的修改都会生成Migration的文件（.cs类型文件），打开这些文件会发现，它将原本的sql脚本进行了对象化。

以上为个人理解，接下来看官方定义：
>在实际项目中，数据模型随着功能的实现而变化：添加和删除新的实体或属性，并且需要相应地更改数据库架构，使其与应用程序保持同步。 EF Core 中的迁移功能能够以递增方式更新数据库架构，使其与应用程序的数据模型保持同步，同时保留数据库中的现有数据。

>简要地说，迁移的方式如下：

> 1. 当引入数据模型更改时，开发人员使用 EF Core 工具添加相应的迁移，以描述使数据库架构保持同步所需的更新。EF Core 将当前模型与旧模型的快照进行比较，以确定差异，并生成迁移源文件；文件可在项目的源代码管理中进行跟踪，如任何其他源文件。
> 2. 生成新的迁移后，可通过多种方式将其应用于数据库。 EF Core 在一个特殊的历史记录表中记录所有应用的迁移，使其知道哪些迁移已应用，哪些迁移尚未应用。

使用下面的命令来生成迁移，如果在powershell中使用dotnet-ef的话，需要安装efcore工具
```bash
dotnet tool install --global dotnet-ef --version 6.0    --安装dotnet-ef工具

dotnet-ef migrations add First --project YourProjectName --添加迁移文件（First为迁移名称）

dotnet-ef database update --project YourProjectName --将迁移提交至数据库（此时，对数据库的修改才生效）

```
如果有多个项目，且采用领域设计模式，将数据库上下文放在Infrastructure项目中，则使用下面的命令管理Migration
```bash
#--context指定DbContext -o 指定输出目录
dotnet-ef migrations add SqliteInit --context TodoListSqliteDbContext -o Migrations/Sqlites -p TodoList.Infrastructure.csproj -s TodoList.Api.csproj
```

## 认识DbContext

> DbContext 实例表示与数据库的会话，可用于查询和保存实体的实例。 DbContext 是工作单元和存储库模式的组合。

DbContext的生存周期从创建实例开始到释放实例结束。EF Core中的典型工作单元包括：
* 创建 DbContext 实例
* 根据上下文跟踪实体实例。 实体将在以下情况下被跟踪
    * 正在从查询返回
    * 正在添加或附加到上下文
* 根据需要对所跟踪的实体进行更改以实现业务规则
* 调用 SaveChanges 或 SaveChangesAsync。 EF Core 检测所做的更改，并将这些更改写入数据库。
* 释放 DbContext 实例

DbContext是实体类与数据库之间的桥梁，负责与数据库交互。主要功能有以下几点：
1. 包含所有实体映射到数据库表的实体集`DbSet<TEntity>`
2. 将LINQ-to-Entities查询转换为SQL查询并发送给数据库
3. 更改跟踪：它会跟踪每个实体从数据库中查询出来后发生的修改变化
4. 持久化数据：基于实体状态执行插入、更新、删除操作到数据库中


## TodoList项目支持多数据库
有了上面的知识做铺垫，我们来修改这个TodoList项目就要水到渠成了。首先，我们在TodoList.Api项目中的appsettings.json中增加一个配置项`DbType`，用于指定数据库类型。然后在ConnectionStrings中，增加Sqlite的链接字符串。修改后json文件如下:
```javascript
//省略以上
"DbType": "Sqlite", //增加此项，指定使用哪种数据库
  "ConnectionStrings": {
    "SqliteConStr":  "Data Source = TodoListDb.db",//增加此项为Sqlite数据库连接
    "SqlServerConnection": "Server=localhost,1433;Database=TodoListDb;User Id=sa;Password=StrongPwd123;"
  },
```

接下来，在与数据库打交道的TodoList.Infrastructure项目中，来修改我们的`DbContext`文件。
根据前面的介绍，为了支持多种数据库，我们需要建立多个`DbContext`。根据需要，我们新建TodoListSqliteDbContext.cs和TodoListSqlserverDbContext.cs两个文件。代码分别如下：
`TodoListSqliteDbContext.cs `
```csharp
/// 
  private readonly IConfiguration configuration;
  private readonly IDomainEventService _domainEventService;
  private readonly ICurrentUserService _currentUserService;

  public TodoListSqliteDbContext(IConfiguration configuration,
  IDomainEventService domainEventService,
  ICurrentUserService currentUserService)
  {
      this.configuration = configuration;
      _domainEventService = domainEventService;
      _currentUserService = currentUserService;
  }

  public override async Task<int> SaveChangesAsync(CancellationToken cancellationToken = new())
  {
      // 在我们重写的SaveChangesAsync方法中，去设置审计相关的字段，目前对于修改人这个字段暂时先给个定值，等到后面讲到认证鉴权的时候再回过头来看这里
      foreach (var entry in ChangeTracker.Entries<AuditableEntity>())
      {
          switch (entry.State)
          {
              case EntityState.Added:
                  entry.Entity.CreatedBy = _currentUserService.UserName;
                  entry.Entity.Created = DateTime.UtcNow;
                  break;

              case EntityState.Modified:
                  entry.Entity.LastModifiedBy = _currentUserService.UserName;
                  entry.Entity.LastModified = DateTime.UtcNow;
                  break;
          }
      }

      // 在写数据库的时候同时发送领域事件，这里要注意一定要保证写入数据库成功后再发送领域事件，否则会导致领域对象状态的不一致问题。
      var events = ChangeTracker.Entries<IHasDomainEvent>()
              .Select(x => x.Entity.DomainEvents)
              .SelectMany(x => x)
              .Where(domainEvent => !domainEvent.IsPublished)
              .ToArray();
      var result = await base.SaveChangesAsync(cancellationToken);
      await DispatchEvents(events);
      return result;
  }

  protected override void OnConfiguring(DbContextOptionsBuilder options)
  {
      options.UseSqlite(configuration.GetConnectionString("SqliteConStr"),
          b => b.MigrationsAssembly(typeof(TodoListSqliteDbContext).Assembly.FullName));
  }

  private async Task DispatchEvents(DomainEvent[] events)
  {
      foreach (var @event in events)
      {
          @event.IsPublished = true;
          await _domainEventService.Publish(@event);
      }
  }
```
`TodoListSqlserverDbContext.cs`
```csharp
  private readonly IConfiguration configuration;

  public TodoListSqlServerDbContext(IConfiguration configuration)
  {
      this.configuration = configuration;
  }

  protected override void OnConfiguring(DbContextOptionsBuilder options)
  {
      options.UseSqlite(configuration.GetConnectionString("SqliteConStr"),
          b => b.MigrationsAssembly(typeof(TodoListDbContext).Assembly.FullName));
  }
```
解释一下为什么Sqlite和SqlServer的DbContext中代码有些差距。主要是Sqlite中多出了SaveChangeAsync和DispatchEvents这两个方法。本来这两个方法是在TodoListDbContext这个父类去实现的，但我修改完代码后，运行项目，SaveChangeAsync这个方法并不能正确运行，其内部的_currentUserService对象为空。也就是说并没有调用TodoListDbContext这个类的构造函数来注入domainEventService和currentUserService。所以我只能将他们放到具体的子类中去实现。

然后修改依赖注入中`AddInfrastructure`扩展方法的代码，根据DbType来选择实际的DbContext对象。
```csharp
  public static IServiceCollection AddInfrastructure(this IServiceCollection services, IConfiguration configuration)
    {
        var dbType = configuration.GetValue<string>("DbType");
        if (dbType == "SqlServer")
            services.AddDbContext<TodoListDbContext, TodoListSqlServerDbContext>();
        else if (dbType == "Sqlite")
            services.AddDbContext<TodoListDbContext, TodoListSqliteDbContext>();

        //services.AddDbContext<TodoListDbContext>(options =>
        //    options.UseSqlServer(
        //        configuration.GetConnectionString("SqlServerConnection"),
        //        b => b.MigrationsAssembly(typeof(TodoListDbContext).Assembly.FullName)));
      //以下代码为原代码，无需修改，省略...
    }
```

修改完成，使用dotnet-ef工具重新构建数据库，使用下面命令
``` bash
dotnet-ef migrations add SqliteInit --context TodoListSqliteDbContext -o Migrations/Sqlites -p TodoList.Infrastructure.csproj -s TodoList.Api.csproj

dotnet-ef database update
```
执行完成后，将在Migrations/Sqlite下生成迁移文件。并在TodoList.Api项目下生成TodoListDb.db文件，其中包含TodoList、TodoItem表，以及Identity相关的几张表。

## 参考博客
1. `DbContext概述` https://docs.microsoft.com/zh-cn/ef/core/dbcontext-configuration/
2. `迁移概述` https://docs.microsoft.com/zh-cn/ef/core/managing-schemas/migrations/?tabs=dotnet-core-cli
3. `使用多个提供程序进行迁移` https://docs.microsoft.com/zh-cn/ef/core/managing-schemas/migrations/providers?tabs=dotnet-core-cli
4. `asp.net core + entity framework core 多数据库类型支持实战`  https://www.cnblogs.com/wjsgzcn/p/12819003.html



