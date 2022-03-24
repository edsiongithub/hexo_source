---
title: ASP.NET Core 依赖注入
date: 2022-03-03 10:17:46
layout: 2022-03-03 10:17:46
id: 0036
tags:
 - dotnet core
 - 依赖注入
 - 翻译
categories:
 - ASP.NET Core
 - 翻译
---
 原文连接 https://dotnettutorials.net/lesson/asp-net-core-dependency-injection/

 本文将通过例子讨论`ASP.NET Core`依赖注入。请阅读先前关于《`ASP.NET Core MVC`模型》的文章。依赖注入设计模式是实时应用中最常用的设计模式。`ASP.NET COre`提供了内置依赖注入。本文将讨论一下细节：
 1. 理解`ASP.NET Core`依赖注入的需求
 2. 什么是依赖注入
 3. 如何在`ASP.NET Core`依赖注入容器中注册一个服务
 4. `ASP.NET Core`依赖注入容器中提供哪些不同方法来注册服务
 5. 理解Singleton，Scoped， Transient方法
 6. 使用依赖注入的优势

<!--more-->

### 理解`ASP.NET Core`依赖注入的需求
我们通过举个例子理解为什么`ASP.NET Core`需要依赖注入。首先，使用空项目模板创建一个`ASP.NET Core`应用，命名为“FirstCoreMVCWebApplication”。

#### 添加模型
创建好空模板项目后，向项目中添加模型。首先创建一个Models目录，在Models目录中添加一个类文件，命名为Student。该Student类就是我们这个应用中的模型。复制下面的代码到Student.cs文件中。
```csharp
namespace FirstCoreMVCWebApplication.Models
{
    public class Student
    {
        public int StudentId { get; set; }
        public string Name { get; set; }
        public string Branch { get; set; }
        public string Section { get; set; }
        public string Gender { get; set; }
    }
}
```
#### 创建服务接口
在Models目录下创建一个IStudentRepository.cs接口文件。这个接口声明一系列我们用来操作学生数据的方法。复制下面代码到IStudentRepositry.cs文件中。

```csharp
using System.Collections.Generic;
namespace FirstCoreMVCWebApplication.Models
{
    public interface IStudentRepository
    {
        Student GetStudentById(int StudentId);
        List<Student> GetAllStudent();
    }
}
```

### 创建服务实现
在Models目录下创建一个名为TestStudentRepository.cs的文件。将下面代码复制到该文件内。这个类实现了IStudentRepository接口定义的两个方法。

```csharp
using System.Collections.Generic;
using System.Linq;
namespace FirstCoreMVCWebApplication.Models
{
    public class TestStudentRepository : IStudentRepository
    {
        public List<Student> DataSource()
        {
          return  new List<Student>()
            {
                new Student() { StudentId = 101, Name = "James", Branch = "CSE", Section = "A", Gender = "Male" },
                new Student() { StudentId = 102, Name = "Smith", Branch = "ETC", Section = "B", Gender = "Male" },
                new Student() { StudentId = 103, Name = "David", Branch = "CSE", Section = "A", Gender = "Male" },
                new Student() { StudentId = 104, Name = "Sara", Branch = "CSE", Section = "A", Gender = "Female" },
                new Student() { StudentId = 105, Name = "Pam", Branch = "ETC", Section = "B", Gender = "Female" }
            };
        }
        public Student GetStudentById(int StudentId)
        {
            return DataSource().FirstOrDefault(e => e.StudentId == StudentId);
        }
        public List<Student> GetAllStudent()
        {
            return DataSource();
        }
    }
}
```
#### Startup类
在Startup类中，我们需要做两件事。一，向IoC容器中配置MVC。 二，将MVC中间件添加到请求管线中。将下面代码复制到Startup.cs文件中

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
namespace FirstCoreMVCWebApplication
{
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllersWithViews();
        }
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            app.UseRouting();
            app.UseEndpoints(endpoints =>
            {
                endpoints.MapDefaultControllerRoute();
            });
        }
    }
}
```

### 不使用依赖注入
创建一个Controllers目录，添加一个HomeController.cs文件。将下面代码复制到该控制器文件中。
```csharp
using FirstCoreMVCWebApplication.Models;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
namespace FirstCoreMVCWebApplication.Controllers
{
    public class HomeController : Controller
    {
        public JsonResult Index()
        {
            TestStudentRepository repository = new TestStudentRepository();
            List<Student> allStudentDetails = repository.GetAllStudent();
            return Json(allStudentDetails);
        }
        public JsonResult GetStudentDetails(int Id)
        {
            TestStudentRepository repository = new TestStudentRepository();
            Student studentDetails = repository.GetStudentById(Id);
            return Json(studentDetails);
        }
    }
}
```
以上代码修改完成后，运行应用，控制器中的两个方法应该能够按预期执行。我们来看看上面的实现有什么问题以及如何在`ASP.NET Core`应用中解决这个问题。

#### 以上实现的问题在哪
在上面的HomeController中，为了获取学生数据，HomeController类依赖TestStudentRepository类。在HomeController类中，我们创建了TestStudentRepository类示例，然后调用GetStudentById()和GetAllStudent方法。这样HomeController类和TestStudentRepository类就紧耦合了。

当修改了IStudentRepository接口的实现类时，我们同时需要修改HomeController类。在`ASP.NET Core`应用中，我们可以使用依赖注入设计模式来解决。

### 什么是依赖注入设计模式
依赖注入是将一个类的对象注入到另一个依赖于该类的类中。依赖注入是目前最流行的一种设计模式，它能去除对象间的依赖从而开发松耦合的软件模块。

下面我们来一步一步在`ASP.NET Core MVC`应用中实现依赖注入。

#### `ASP.NET Core`中的依赖注入
`ASP.NET Core`框架支持内置依赖注入，`ASP.NET Core`框架中类依赖的对象可以通过内置IoC（Inverse of Control 控制反转）容器使用构造函数或者方法的方式注入。

![](https://dotnettutorials.net/wp-content/uploads/2019/03/word-image-19.png)
`ASP.NET Core`框架包含一个简单的开箱即用的IoC容器，该容器与其他第三方IoC容器，如Unity、StructureMap、Castle Windsor、Ninject相比没有那么丰富的功能。如果你想使用更多的特性，如自动注册、扫描、拦截器、装饰器等，那么你需要使用第三方IoC容器来替换内置的IoC容器。

内置的容器通过`IServiceProvider`接口的实现来表示，默认支持构造函数方式注入。内置IoC容器管理的类被成为服务。

### `ASP.NET Core`中服务的类型
`ASP.NET Core`中的服务类型有两种：

1. 框架服务：`ASP.NET Core`框架中包含的一部分服务，如`IApplicationBuilder`、`IHostingEnvironment`、`ILoggerFactory`等。
2. 应用服务：有程序员为自己的应用创建的服务（自定义类型或者类）。

为了容IoC容器自动注入我们的应用服务，我们需要首先向容器中注册这些服务。

### 如何向`ASP.NET Core`依赖注入容器中注册服务
我们需要在Startup类中使用`ConfigureServices()`方法向`ASP.NET Core`依赖注入容器中注册一个服务。

在我们讨论如何向依赖注入容器中注册服务前，先了解服务的生命周期非常重要。当一个类通过依赖注入接收到依赖对象，无论它接收到的实例对该类是唯一的或者不依赖于服务的生命周期。（这一句没翻译明白，原话： When a class receives the dependency object through dependency injection, then whether the instance it receives is unique to that instance of the class or not depends on the lifetime of the service.）设置依赖对象的生命周期决定了依赖对象需要被创建多少次。

### `ASP.NET Core`提供哪些方法来注册服务
`ASP.NET Core`提供以下3中方法向依赖注入容器注册服务。选择不同的方法注册服务决定了服务的生命周期。

<strong>Singleton</strong>
 这种情况下，IoC容器在整个应用生命周期中，只创建一次对象实例并共享使用。
<strong>Transient</strong>
这种情况下，IoC容器在你每次请求该服务时，会创建一个新的服务实例。
<strong>Scoped</strong>
这种情况下，IoC容器在每次请求时创建一个新的服务实例并在每个请求中共享该实例。

注意：IoC内置容器管理注册服务的生命周期，它将根据服务实例的生命周期来自动释放服务实例。

### 使用`ASP.NET Core`依赖注入注册TestStudentRepository
我们需要在Startup类中使用`ConfigureServices()`方法中配置服务实例，下面代码展示如何注册一个不同生命周期的服务:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    //Adding MVC Service. Framework Service
    services.AddControllersWithViews();
    //Application Service
    services.Add(new ServiceDescriptor(typeof(IStudentRepository), new TestStudentRepository())); // by default singleton
    services.Add(new ServiceDescriptor(typeof(IStudentRepository), typeof(TestStudentRepository), ServiceLifetime.Singleton)); // singleton
    services.Add(new ServiceDescriptor(typeof(IStudentRepository), typeof(TestStudentRepository), ServiceLifetime.Transient)); // Transient
    services.Add(new ServiceDescriptor(typeof(IStudentRepository), typeof(TestStudentRepository), ServiceLifetime.Scoped));    // Scoped
}
```

### 扩展方法来注册
`ASP.NET Core`框架为每种生命周期注册方法提供扩展方法：`AddSingleton()`,`AddTransient()`,` AddScoped()`方法分别对应`singleton`, `transient` , `scoped`生命周期。下面代码展示使用扩展方法来注册不同生命周期的服务对象：
```csharp
public void ConfigureServices(IServiceCollection services)
{
    //Adding MVC Service. Framework Service
    services.AddControllersWithViews();
    //Application Service
    services.AddSingleton<IStudentRepository, TestStudentRepository>();
    services.AddSingleton(typeof(IStudentRepository), typeof(TestStudentRepository));
    services.AddTransient<IStudentRepository, TestStudentRepository>();
    services.AddTransient(typeof(IStudentRepository), typeof(TestStudentRepository));
    services.AddScoped<IStudentRepository, TestStudentRepository>();
    services.AddScoped(typeof(IStudentRepository), typeof(TestStudentRepository));
}
```
我们使用唯一实例服务类型来举例。修改`Startup`类中的`ConfigureService`方法，选择哪种方法向IoC容器中注册你的服务是你的个人选择，我将使用下面这种（扩展方法的方式）：
```csharp
public void ConfigureServices(IServiceCollection services)
{
    //Adding MVC Service. Framework Service
    services.AddControllersWithViews();
    //Application Service
    services.AddSingleton<IStudentRepository, TestStudentRepository>();
}
```

#### `ASP.NET Core MVC`应用中构造函数注入
一旦我们注册服务，如果该服务类作为构造函数的一个参数，则IoC容器自动使用构造韩式方式注入服务。我们修改HomeController控制器代码如下，来使用构造函数方式实现注入：

```csharp
using FirstCoreMVCWebApplication.Models;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
namespace FirstCoreMVCWebApplication.Controllers
{
    public class HomeController : Controller
    {
        //Create a reference variable of IStudentRepository
        private readonly IStudentRepository _repository = null;
        //Initialize the variable through constructor
        public HomeController(IStudentRepository repository)
        {
            _repository = repository;
        }
        public JsonResult Index()
        {
            List<Student> allStudentDetails = _repository.GetAllStudent();
            return Json(allStudentDetails);
        }
        public JsonResult GetStudentDetails(int Id)
        {
            Student studentDetails = _repository.GetStudentById(Id);
            return Json(studentDetails);
        }
    }
}
```
代码解释：
在上面的例子中，IoC容器自动向HomeController传递一个TestStudentRepository的对象实例，我们无需做任何更多的操作。IoC容器会根据注册服务的生命周期来创建并销毁IStudentRepository对象实例。因为我们使用构造函数方式注入对象，这就叫做构造函数依赖注入。

我们创建一个只读的`_repository`变量，这样一旦我们注入了依赖的对象，这个对象将不会再改变。

运行我们的应用，你将获得如下图所示的输出：

![](https://dotnettutorials.net/wp-content/uploads/2019/03/word-image-21.png)

#### `ASP.NET Core MVC`应用中Action方法注入
有时我们可能只需要向某个具体的action方法中注入依赖对象。我们可以使用`[FromServices]`特性来标注，使用服务类型作为方法的参数。如下例所示：
```csharp
using FirstCoreMVCWebApplication.Models;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
namespace FirstCoreMVCWebApplication.Controllers
{
    public class HomeController : Controller
    {
        public HomeController()
        {
        }
        public JsonResult Index([FromServices] IStudentRepository repository)
        {
            List<Student> allStudentDetails = repository.GetAllStudent();
            return Json(allStudentDetails);
        }
    }
}
```
运行应用，你会获得下面的输出：
![](https://dotnettutorials.net/wp-content/uploads/2019/03/word-image-22.png)
#### 属性注入
内置IoC容器不支持属性注入，如果需要的话，得使用第三方IoC容器。

### 手动获取服务
依赖服务不是必须要包含在构造函数中，我们可以通过内置IoC容器配置，使用`HttpContext`的属性`RequestSerices`来手动获取依赖服务，如下面例子所示：

```csharp
using FirstCoreMVCWebApplication.Models;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;

namespace FirstCoreMVCWebApplication.Controllers
{
    public class HomeController : Controller
    {
        public JsonResult Index()
        {
            var services = this.HttpContext.RequestServices;
            var _repository = (IStudentRepository)services.GetService(typeof(IStudentRepository));
            List<Student> allStudentDetails = _repository.GetAllStudent();
            return Json(allStudentDetails);
        }

        public JsonResult GetStudentDetails(int Id)
        {
            var services = this.HttpContext.RequestServices;
            var _repository = (IStudentRepository)services.GetService(typeof(IStudentRepository));
            Student studentDetails = _repository.GetStudentById(Id);
            return Json(studentDetails);
        }
    }
}
```
将HomeController代码修改成如上，运行应用后将获得如下所示的输出。
![](https://dotnettutorials.net/wp-content/uploads/2019/03/word-image-23.png)
注意：推荐使用构造函数注入而不是使用`RequestServices`。

### 什么时候该用谁
在实时应用中，你需要将一些组件如应用范围内的配置注册成Sigleton。数据库访问类如EntityFramework上下文推荐注册为Scoped，这样数据库连接可以被复用。如果你想并行执行任何内容，最好将该组件注册为Transient。
简而言之：
<strong>AddSingleton():</strong>
当我们使用AddSingleton()方法注册服务，将创建一个singleton服务。该实例将在应用中的所有组件中共享使用。第一次请求该服务时创建singleton服务。

<strong>AddScoped():</strong>
Scoped意味着每个请求一个实例。当我们使用AddScoped()方法注册服务，创建一个Scoped服务。意味着每个HTTP请求到达时创建该服务，并在该HTTP请求中的其他调用中共性使用该服务实例。

<strong>AddTransient():</strong>
当我们使用AddTransient()方法注册服务时，创建Transient类型服务。意味着对特定服务的每次请求（调用）创建一个新的实例，这些实例不会共享使用。

使用`ASP.NET Core`依赖注入有什么有点？
`ASP.NET Core`依赖注入允许我们开发松耦合的软件组件。使用`ASP.NET Core`依赖注入，可以很容易将一个组件的实现替换成另外一个不同的实现。
在下一篇文章，我将讨论`ASP.NET Core MVC`应用中的控制器。本篇我视图通过例子解释`ASP.NET Core`依赖注入。希望这篇文章有助于你理解`ASP.NET Core`依赖注入的概念。

### 参考
<a href="https://dotnettutorials.net/course/dot-net-design-patterns/">设计模式</a>
<a href="https://dotnettutorials.net/lesson/dependency-injection-design-pattern-csharp/">依赖注入</a>
<a href="https://dotnettutorials.net/lesson/introduction-to-inversion-of-control/">IoC容器</a>


### 译者注
在翻译这篇文章前，我写过一篇<a href="https://edsiongithub.github.io/2021/12/14/24/">关于`ASP.NET Core`依赖注入</a>的文章。通过一个完整的MVC例子演示了三种扩展方法注册服务，获得服务实例的生命周期。以及选择使用哪种情况选择哪个方法来注册服务。


