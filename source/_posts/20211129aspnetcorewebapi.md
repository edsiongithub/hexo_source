---
title: ASP.Net Core WebApi开发
date: 2021-11-29 15:19:13
layout: 2021-11-29 15:19:13
id: 0029
tags:
 - asp.net core
 - web api
categories:
 - ASP.NET Core
---

本文包含：
* 创建`asp.net core` 项目
* 搭建vs code开发环境
* 使用httprepl工具调试api
* 使用openapi（swagger）
* 版本控制

<!--more-->

### vs code搭建webapi项目开发环境
首先需要安装dotnet sdk（6.0长期支持版），可以在微软官方下载并安装，一路下一步即可。打开powershell，输入`dotnet --version`，显示dotnet版本说明环境安装成功。
* 接下来在VS Code中，新建目录用于存放项目。
* 在终端中输入dotnet命令，创建api项目
```
dotnet new webapi
```
* 安装C#扩展插件
* 新建终端，输入命令编译并运行项目
```
dotnet build
dotnet run
```

### 使用openapi（swagger）

如果在VisualStudio中创建Api项目，会有选项询问是否使用OpenApi。如果选中，则会在创建项目是，自动帮助添加swager引用。~~使用VS Code创建的项目并不包含，可以通过手动安装Nuget包来将Swagger引用到项目中。~~  使用`dotnect cli`命令创建的项目，也是包含swagger的。
关于Swagger的介绍，可以在<a href="https://swagger.io/">官网</a>中查看。它提供了生成、描述、调用和可视化 RESTful 风格的 Web 服务。

在此之前，先安装一个插件，用于管理Nuget包。插件市场中搜索Nuget，选择Nuget Package Manager GUI。安装后，在VS Code 中按下 `ctrl + shift + p`键，选择Nuget Package Manager GUI，会弹出类似Visual Studio中的Nuget包管理器界面。
![Nuget Manager Package GUI](https://raw.githubusercontent.com/edsiongithub/blogimages/master/202111/NugetManager.png)

此时，可以像在Visual Studio中一样来管理Nuget包了。

使用VS Code 打开我们的项目，双击`Program.cs`文件，会发现和`.net core 3.X`以前的版本有些许不同。该文件中没有那么多的`using`语句，没有类，没有方法。看上去更像一个python创建Flask应用时的代码。这是采用了C# 10的<a href="https://docs.microsoft.com/zh-cn/dotnet/csharp/whats-new/tutorials/top-level-statements">顶级语句</a>功能。
``` csharp
var builder = WebApplication.CreateBuilder(args);
// Add services to the container.
builder.Services.AddControllers();
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
var app = builder.Build();
// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

在上面的代码中，可以找到注册Swagger的相关信息。运行项目后，控制台中会输出项目运行的地址（如：`http://localhost:5039`），此时只需要在浏览器中输入`http://localhost:5039/swagger/index.html`即可以查看swagger生成的api文档，也可以在这里对api进行调试。
![swagger ui](https://raw.githubusercontent.com/edsiongithub/blogimages/master/202111/swaggerui.png)
### 使用httprepl调试
安装httprepl工具
```
dotnet tool install -g Microsoft.dotnet-httprepl
```
VS Code中新建控制台，输入如下命令(假设你的webapi应用运行在5000端口中，且有一个WeatherForcast控制器提供天气服务）：
```
httprepl http://localhost:5000      #链接到web应用
cd WeatherForcast                   #切换到指定的controller
ls                                  #列出该Controller下的所有http服务（Get、Post、Put、Delete)
get [id]                            #调用get服务
post -h [headers] -c [contents]     #调用post方法，-h提供header，-c提供body参数
```

### 版本控制
Nuget安装包`Microsoft.AspNetCore.Mvc.Versioning`包，`Program.cs`文件中添加以下代码：
```csharp
builder.Services.AddApiVersioning(options =>
{
    options.ReportApiVersions = true;
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.DefaultApiVersion = new ApiVersion(1, 0);
});
```
将原来的Controller分别放置到V1目录和V2目录中，修改V1、V2版本中的Controller上的路由信息，将版本信息添加在路由信息中
```csharp
    [ApiController]
    [Route("api/v1/[controller]")]
    public class PizzaController : ControllerBase
    {
        ...
    }
```