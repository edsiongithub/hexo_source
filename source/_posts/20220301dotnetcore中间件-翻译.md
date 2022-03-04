---
title: asp.net core 中间件(翻译)
date: 2022-03-01 09:23:13
layout: 2022-03-01 09:23:13
id: 0034
tags:
 - dotnet core
 - Middleware
 - 翻译
categories:
 - ASP.NET Core
 - 翻译
---
 
 准备翻译dotnet tutorial网站上一些dotnet方面的知识文章。先从中间件开始，原文地址：
 [ASP.NET Core Middleware with Examples](https://dotnettutorials.net/lesson/asp-net-core-middleware-components/)

<!--more-->

本文主要讨论以下与`ASP.NET Core`中间件相关概念
1. 什么是`ASP.NET Core`中间件
2. 在`ASP.NET Core`应用的什么地方使用中间件
3. 如何在`ASP.NET Core`应用中配置中间件
4. 使用中间件的例子
5. `ASP.NET Core`中间件的执行顺序
6. `ASP.NET Core`中的请求代理是什么
7. `ASP.NET Core`中的Use,Run,Map方法的作用
8. 什么是`UseDeveloperExceptionPage`中间件
9. 如何使用`Run()`和`Use()`扩展方法配置中间件
10. MapGet和Map方法的区别

### 什么是`ASP.NET Core`中间件

`ASP.NET Core`中间件是一种软件组件（技术上仅仅是一些C#的类），聚合到应用管线用来处理Http请求和响应。每个中间件执行以下任务：
1. 选择是将Http请求传递到管线中的下一个中间件。通过调用中间件的next()方法实现。
2. 能够执行一些工作在管线的下一个中间件之前和之后。

`ASP.NET Core`中有许多内置中间件已经可以直接使用。你也可以在你的`ASP.NET Core`应用中根据需要创建自己的中间件。最重要的是，一个给定的`ASP.NET Core`中间件只专注一个目的：即完成一个职责。


### `ASP.NET Core`应用中哪里使用中间件
 `ASP.NET Core`应用中使用中间件的一些例子：

1. 使用一个中间件来做用户认证
2. 使用日志记录请求和响应
3. 使用一个中间件来处理错误（异常）
4. 用来处理静态文件，如图片、javascript脚本或css样式文件等
5. 使用中间件来给用户授权，指定特定访问资源

中间件通常是我们用来处理 `ASP.NET Core`应用请求管线。如果在先前版本的`.NET Framework`工作过，你应该知道使用HTTP Handlers 和 HTTP Modules来创建请求处理管线。请求管线将决定HTTP请求和响应将如何执行。

### 如何在`ASP.NET Core`应用中配置中间件

在`ASP.NET Core`应用中，我们需要在Startup.cs文件中的Startup类中使用Configure()方法来配置中间件（`.net5`以上版本没有该类了）。Startup这个类是在应用开始时执行的。当我们使用空的`ASP.NET Core`应用模板创建项目，将会生成如下带有Configuration()方法的代码：

![asp.net core中配置中间件](https://dotnettutorials.net/wp-content/uploads/2019/01/word-image-102.png)
当你需要配置中间件时，只需要在startup类中使用Configuration()方法中使用 IApplicationBuilder对象的Use方法。如上图所示，Configuration()方法仅使用了3个中间件来创建请求处理管线。这3个中间件分别是：
1. `UseDeveloperExceptionPage()`中间件
2. `UseRouting()`中间件
3. `UseEndpoints()`中间件
在理解以上三个内置中间件之前，先了解什么是中间件以及这些中间件如何在`ASP.NET Core`应用中工作的。

#### 理解中间件
在`ASP.NET Core`应用中，中间件可以访问到流入的HTTP请求和流出的HTTP响应，所以可以用来：
1. 通过生成一个HTTP响应来处理流入的HTTP请求
2. 处理并可以修改流入的HTTP请求，然后将该HTTP请求传给管线上的下一个中间件
3. 处理并可以修改流出的HTTP响应，并将HTTP响应传递给管线上的下一个中间件或`ASP.NET Core`应用（如果管线上已经没有下一个中间件的话）
为了更好理解，下面的图展示了`ASP.NET Core`应用中处理管线如何执行。

![中间件执行顺序](https://dotnettutorials.net/wp-content/uploads/2019/01/asp-net-core-middleware-components.png)

如上图所示，有一个日志中间件。这个中间件简单的记录下请求时间，然后将请求传递给下一个中间件-静态文件处理中间件，用来做后续处理。

`ASP.NET Core`应用中的中间件也可能通过创建HTTP响应来处理HTTP请求，也可能不调用请求管线中的下一个中间件。这个概念称为：请求管线短路。

例如，有一个静态文件中间件，如果流入HTTP请求是某些静态文件，如图片、CSS样式文件、JavaScript脚本文件等，静态文件中间件就可以处理该请求了，它将不再调用请求管线中下一个中间件：MVC中间件，以此来短路请求管线。

`ASP.NET Core`应用中的中间件不仅能够访问HTTP请求，也可以访问管线中的响应。例如，在本例中的日志中间件可能记录下响应发送给客户端的时间。

### `ASP.NET Core`中的中间件执行顺序
理解中间件执行顺序非常重要。`ASP.NET Core`应用中间件执行顺序是按照他们加入管线的顺序。所以，当往请求处理管线中加入中间件时需要非常注意顺序。

按照你的应用需求，你可能会添加任意数量的中间件。例如，如果你开发的仅仅是静态网站来处理一些静态的HTML页面和图片，那么你仅需要在请求管线中添加“静态文件”中间件。

如果你开发的是安全的动态数据驱动web应用，那么你将需要多个中间件来完成任务，例如日志中间件，认证中间件，授权中间件，MVC中间件等。

### `ASP.NET Core`中的请求代理是什么
在`ASP.NET Core`中，请求代理用来创建请求管线，即请求代理是用来处理每个流入HTTP请求的。在 `ASP.NET Core`中，你可以使用Run，Map，Use扩展方法来配置请求代理。你可以使用一个匿名方法（中间件in-line方式）或者创建一个可复用的类来实现请求代理。这些匿名方法或可复用的类即成为中间件或中间件组件。请求管线中的每个中间件都可以触发或短路下一个中间件。


### `ASP.NET Core`中的`Run()`和`Use()`扩展方法配置中间件

在 `ASP.NET Core`中，使用Use和Run扩展方法来向请求管线中注册行内中间件。Run扩展方法允许我们添加终点中间件（即该中间件后面不会再有中间件了）。另一方面，Use扩展方法允许我们项请求管线中添加可以继续调用下一个中间件的中间件。

如果你观察Configure方法，你会看到它获取了一个IApplicationBuilder接口的实例，然后使用该实例的扩展方法如Use、Run来配置中间件组件。

在上面的示例，3个中间件通过使用IApplicationBuilder的示例注册到请求管线中。他们分别是：
1. UseDeveloperExceptionPage()中间件
2. UseRouting()
3. UseEndpoints() 中间件

#### `UseDeveloperExceptionPage`中间件
在Configure方法中，`UseDeveloperExceptionPage()`中间件注册到请求管线中，该中间件仅在服务器环境被设置为“development”时才会显示一个图片。该中间件仅当应用中出现一个未处理的异常发生且时在开发模式下，会显示出现问题的代码。你可以考虑使用黄色页面来代替。稍后下一篇文章将会示例如何使用中间件。

#### `UseRouting()`中间件
该中间件组件用于添加端点路由中间件到请求管线，该中间件将URL（或者流入HTTP请求）映射到一个特定的资源。将会在后面路由相关的文章讨论细节。

#### `UseEndpoints()`中间件
在这个中间件，使用Map扩展方法路由决定。下面是UseEndpoints中间件的默认实现。MapGet扩展方法，我们将路由设置为"/"，说明仅匹配域名。即任何仅包含域名的请求URL均由该中间件处理

![](https://dotnettutorials.net/wp-content/uploads/2019/01/word-image-103.png)
除了使用MapGet方法，你话可以使用Map方法来映射。
![](https://dotnettutorials.net/wp-content/uploads/2019/01/word-image-104.png)
此时运行应用，你将获得预期的输出。

#### MapGet和Map方法的区别
MapGet方法仅处理GET方法的HTTP请求，而Map方法则可以处理GET、POST、PUT以及DELETE的HTTP请求。

### 如何使用Run扩展方法配置中间件

接下来我们使用Run扩展方法创建并配置自定义的中间件。首先，Configure方法中有所有的代码注释。你可以注释掉所有已有代码，然后复制黏贴下面的代码到Configure方法中。下面的代码向请求管线中注册了一个简单的中间件，该中间件仅打印一些消息。

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.Run(async (context) =>
    {
        await context.Response.WriteAsync("Getting Response from First Middleware");
    });
}
```
输出：
![](https://dotnettutorials.net/wp-content/uploads/2019/01/word-image-105.png)
我们触发IApplicationBuilder实例（app变量)Run扩展方法，注册该中间件至请求管线。下面是Run方法的定义。

![](https://dotnettutorials.net/wp-content/uploads/2019/01/configure-middleware-components-in-asp-net-core.png)

从上面Run方法定义可以看出，该方法实现IApplicationBuilder接口的扩展方法。这是为什么可以使用IApplicationBuilder的实例app调用Run扩展方法。如果你对扩展方法不熟悉，可以先阅读下面文章。

https://dotnettutorials.net/lesson/extension-methods-csharp/

从上面图片也能看出，Run方法接收一个输入参数，类型为RequestDelegate。下面是RequestDelegate的定义。

![](https://dotnettutorials.net/wp-content/uploads/2019/01/request-delegate-definition.png)

从上面图片看出，RequestDelegate代理一类方法，该类方法接收一个类型为HttpContext的参数。如果你对代理不熟悉，可以阅读下面文章。

https://dotnettutorials.net/lesson/delegates-csharp/

前面一直讨论到的`ASP.NET Core`中间件既能够访问HTTP Request，也能访问HTTP Response，就是因为上面的那个HttpContext对象。

在我们的例子中，我们以行内匿名方法传递一个请求代理，并且传递HTTPContext对象作为输入参数给请求代理。如下图所示：

![](https://dotnettutorials.net/wp-content/uploads/2019/01/run-method-in-asp-net-core.png)
注意：除了使用行内匿名方法传递请求代理外，你也可以定义一个独立类来传递请求代理。后面文章将讨论。

#### 再添加一个中间件
修改Configure方法内的代码如下，用来再添加一个中间件：

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.Run(async (context) =>
    {
        await context.Response.WriteAsync("Getting Response from First Middleware");
    });
    app.Run(async (context) =>
    {
        await context.Response.WriteAsync("Getting Response from Second Middleware");
    });
}
```
现在用Run扩展方法注册了2个中间件。如果你运行应用，将获得如下输出
```
Getting Response from 1st Middleware
```
仅打印出第一个中间件的输出。原因是我们使用Run扩展方法来注册的中间件，使用该方法注册的中间件为端点中间件，即他们不会再调用请求管线后面的中间件了。

#### 使用Use扩展方法配置中间件
那么问题来了，如何调用请求管线后面的中间件呢？答案是使用Use扩展方法来注册中间件。如下代码所示。
```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.Use(async (context, next) =>
    {
        await context.Response.WriteAsync("Getting Response from 1st Middleware");
        await next();
    });
    app.Run(async (context) =>
    {
        await context.Response.WriteAsync("Getting Response from 2nd Middleware");
    });
}
```
现在运行应用，将会得打预期的输出，即两个中间件的输出均显示出来了。

![Use扩展方法配置](https://dotnettutorials.net/wp-content/uploads/2019/01/word-image-106.png)
#### 理解Use扩展方法
Use扩展方法以行内方式添加一个中间件代理到请求管线中。下面是Use扩展方法的定义：

![Use扩展方法](https://dotnettutorials.net/wp-content/uploads/2019/01/use-extension-method-in-asp-net-core.png)

该方法也是作为IApplicationBuilder接口的一个扩展方法来实现。这也是我们能够使用IApplicationBuilder实例触发该方法的原因。从该方法定义中可以看出，方法使用两个输入参数，第一个参数是HttpContext对象，通过它可以访问HTTP请求和响应。第二个参数是Func类型，一个泛型代理，可以处理请求或调用请求管线中的下一个中间件。

注意：如果你想向下一个中间件传递请求，那么你需要调用next方法。

本篇至此结束，下一篇将举例使用自定义中间件。本篇尝试解释中间件如何在`ASP.NET Core`应用中处理请求管线。


---
### 译者添加

这是第一篇尝试翻译的文章，尽量理解原作者意思。以后我都会在后面加上译者，添加一些自己的想法。

由于本篇原作者写的时候，使用的应该是`ASP.NET Core 5`版本以下，所有示例代码均在低版本下。5.0以上已经没有StartUp.cs文件了，因此实现上略有区别。会在下一篇翻译完作者举例使用自定义中间件的时候，同时给出高版本的中间件实现方式。