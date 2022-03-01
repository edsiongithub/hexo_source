---
title: ASP.NET Core中间件处理请求管线
date: 2022-03-03 15:26:53
layout: 2022-03-03 15:26:53
id: 0034
tags:
 - dotnet core
 - Middleware
 - 翻译
---
 这是第二篇关于`ASP.NET Core`中间件的翻译文章， 原文地址：
 https://dotnettutorials.net/lesson/asp-net-core-request-processing-pipeline/

本片文章将通过一个例子讨论`ASP.NET Core`请求处理管线。请阅读先前关于`ASP.NET Core`应用中间件的文章。本章将详细讨论以下几点：
1. 理解`ASP.NET Core`请求处理管线
2. 如何在`ASP.NET Core`应用中创建并注册多个中间件
3. 请求处理管线中的中间件执行顺序是怎样的

<!--more-->

### 理解`ASP.NET Core`请求处理管线
为了理解`ASP.NET Core`请求处理管线概念，我们修改Startup类中的Configure()方法代码如下。这里我们注册了3个中间件到请求管线中。前2个组件我们是使用Use()扩展方法来注册的，所以他们是可以调用请求管线中下一个中间件。最后一个使用Run()扩展方法来注册，它就是终点组件，即它将不在调用下一个组件了。

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
namespace FirstCoreWebApplication
{
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
        }
        public void Configure(IApplicationBuilder app)
        {
            app.Use(async (context, next) =>
            {
                await context.Response.WriteAsync("Middleware1: Incoming Request\n");
                await next();
                await context.Response.WriteAsync("Middleware1: Outgoing Response\n");
            });
            app.Use(async (context, next) =>
            {
                await context.Response.WriteAsync("Middleware2: Incoming Request\n");
                await next();
                await context.Response.WriteAsync("Middleware2: Outgoing Response\n");
            });
            app.Run(async (context) =>
            {
                await context.Response.WriteAsync("Middleware3: Incoming Request handled and response generated\n");
            });
        }
    }
}
```
现在执行应用看到浏览器上输出如下信息：

![](https://dotnettutorials.net/wp-content/uploads/2019/02/word-image-5.png)

### 理解`ASP.NET Core` 请求处理管线执行顺序

为了理解这一点，我们使用下面这张图来比较上面的输出。这样更容易理解`ASP.NET Core`应用请求处理管线。

![](https://dotnettutorials.net/wp-content/uploads/2019/02/word-image-6.png)

当流入的HTTP请求道道，第一个中间件接收到，即中间件1在响应流中日志记录下<font color="blue">“Middleware1: Incoming Request”</font>。所以我们在浏览器中看到第一行输出该信息。

当第一个中间件记录完该信息，它开始调用next()方法，该方法会触发请求处理管线中的第二个中间件，即中间件2.

第二个中间件日志记录下<font color="blue">“Middleware2: Incoming Request”</font>，结果我们看到浏览器上输出该消息紧跟在第一条日志消息。然后第二个中间件调用next()方法，该方法将触发请求管线中第3个中间件，即中间件3。

第3个中间件处理请求并产生响应。所以我们看到浏览器输出<font color="blue">“Middleware3: Incoming Request handled and response generated”</font>信息。

中间件3使用Run()扩展方法注册，所以这是一个终点组件。所以从这里开始，请求管线开始反转。意味着从这个中间件返回给第二个中间件，第2个中间件记录下消息： <font color="blue">“Middleware2: Outgoing Response”</font>然后传递给第一个中间件，最后第1个中间件记录下如下消息： <font color="blue">“Middleware1: Outgoing Response”</font>。
### 需要记住的关键点

1. `ASP.NET Core`请求处理管线包含一系列中间件，按顺序一个接一个调用。
2. 每个中间件能够在触发下一个中间件函数next()之前或之后做一些操作。中间件也可以决定不调用下一个中间件，这被称为请求管线短路。
3. 中间件能够访问到流入的请求和流出的响应。
4. 最关键点在于中间件的执行顺序，是按照他们在Startup类中的Configure方法中加入的顺序执行。这些中间件按顺序触发请求管线中的下一个，响应时按反向顺序传递。为了实现应用的安全，效率及功能，需要严格按照顺序定义中间件。

下一篇文章，将讨论`ASP.NET Core`中的`wwwroot`这个目录。本篇尝试解释`ASP.NET Core`请求处理管线，希望对你理解有所帮助。

### 译者添加
本片例子中，注册中间件都是在Startup类中的Configure方法中。但使用最新版本`ASP.NET Core 6.0`模板创建的应用，会发现默认生成的项目文件中并没有Startup.cs这个文件，也找不到Startup类和Configure方法。只有一个Program.cs，相关配置代码都在这里。下面是新版本中，实现一个自定义中间件并注册该中间件的方法。
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

var app = builder.Build();

app.Use(async (HttpContext context, Func<Task> task) =>
{
    await context.Response.WriteAsync("Middleware1: request arrive \n");
    await task.Invoke();
    await context.Response.WriteAsync("Middleare1: Outcomming response \n ");
});

app.Use(async (HttpContext context, RequestDelegate next) =>
{
    await context.Response.WriteAsync("Middleware2: request arrive \n");
    await next.Invoke(context);
    await context.Response.WriteAsync("Middleare2: Outcomming response \n ");
});

(app as IApplicationBuilder).Use(request => async context =>
{
    await context.Response.WriteAsync("Middleware3: request arrive \n");
    await context.Response.WriteAsync("Middleare3: Outcomming response \n ");
});

app.Run();
```

可以使用上面三种方式使用Use扩展方法来注册中间件（行内匿名方法的方式）。也可以创建一个可复用的类来实现中间件。

```csharp
    //创建一个类实现自己的中间件，并提供一个扩展方法UseMyMiddleWareHandler
    public static class MyMiddleWareExtensions
    {
        public static IApplicationBuilder UseMyMiddleWareHandler(this IApplicationBuilder app)
        {
            app.UseMiddleware<MyMiddleWare>();
            return app;
        }
    }

    public class MyMiddleWare
    {
        private readonly RequestDelegate _next;

        public MyMiddleWare(RequestDelegate next)
        {
            _next = next;
        }

        public async Task InvokeAsync(HttpContext context)
        {
            await context.Response.WriteAsync("Middleware start \n");
            await context.Response.WriteAsync("Middleware end \n");
        }
    }

    //Program中使用上面的扩展方法注册中间件
    var builder = WebApplication.CreateBuilder(args);
    // Add services to the container.
    var app = builder.Build();
    app.UseMyMiddleWareHandler();
    app.Run();
```
