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
The Dependency Injection a process of injecting the object of a class into a class that depends on it. The Dependency Injection is the most commonly used design pattern nowadays to remove the dependencies between the objects that allow us to develop loosely coupled software components.

Let us discuss the step by step procedure to implement dependency injection in ASP.NET Core MVC application.

#### `ASP.NET Core`中的依赖注入
The ASP.NET Core Framework is designed from scratch to support inbuilt support for Dependency Injection. The ASP.NET Core Framework injects objects of dependency classes through constructor or method by using a built-in IoC (Inversion of Control) container.

![](https://dotnettutorials.net/wp-content/uploads/2019/03/word-image-19.png)
ASP.NET Core framework contains simple out-of-the-box IoC containers which do not have as many features as other third party IoC containers such as Unity, StructureMap, Castle Windsor, Ninject, etc. If you want more features such as auto-registration, scanning, interceptors, or decorators then you may replace the built-in IoC container with a third-party container.

The built-in container is represented by IServiceProvider implementation that supports constructor injection by default. The types (classes) managed by built-in IoC containers are called services.

### `ASP.NET Core`中服务的类型
There are two types of services in ASP.NET Core. They are as follows:

Framework Services: Services that are a part of the ASP.NET Core framework such as IApplicationBuilder, IHostingEnvironment, ILoggerFactory, etc.
Application Services: The services (custom types or classes) which you as a programmer create for your application.
In order to let the IoC container automatically inject our application services, we first need to register them with the IoC container.

### 如何向`ASP.NET Core`依赖注入容器中注册服务
We need to register a service with ASP.NET Core Dependency Injection Container within the ConfigureServices() method of the Startup class.

Before we discuss how to register a service with the Dependency Injection Container, it is important to understand the lifetime of service. When a class receives the dependency object through dependency injection, then whether the instance it receives is unique to that instance of the class or not depends on the lifetime of the service. Setting the lifetime of the dependency object determines how many times the dependency object needs to be created.

### `ASP.NET Core`提供哪些方法来注册服务

The ASP.NET core provides 3 methods to register a service with the ASP.NET Core Dependency Injection container as follows. The method that we use to register a service will determine the lifetime of that service.

Singleton
Transient
Scoped
Singleton: In this case, the IoC container will create and share a single instance of a service object throughout the application’s lifetime.

Transient: In this case, the IoC container will create a new instance of the specified service type every time you ask for it.

Scoped: In this case, the IoC container will create an instance of the specified service type once per request and will be shared in a single request.

Note: The Built-in IoC container manages the lifetime of a registered service. It automatically disposes of a service instance based on the specified lifetime.

### 使用`ASP.NET Core`依赖注入注册TestStudentRepository
We need to configure the service instance within the ConfigureServices() method of the Startup class. The following code shows how to register a service with different lifetimes:
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
ASP.NET Core framework includes extension methods for each types of lifetime; AddSingleton(), AddTransient() and AddScoped() methods for singleton, transient and scoped lifetime respectively. The following example shows the ways of registering types (service) using extension methods.
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
So, let us use the Single Instance of the service in this example. So, modify the ConfigureService method of the Startup class as shown below. Which method you want to use to register your application service to the built-in IoC Container is your personal preference. I am going to use the following.
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
Once we register the service, the IoC container automatically performs constructor injection if a service type is included as a parameter in a constructor. Let us modify the HomeController as shown below to use the Constructor dependency injection.
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
Code Explanation:
In the above example, the IoC container will automatically pass an instance of the TestStudentRepository to the constructor of HomeController. We don’t need to do anything else. An IoC container will create and dispose of an instance of the IStudentRepository based on the registered lifetime. As we are injecting the dependency object through a constructor, it is called as constructor dependency injection.

We created the _ repository variable as read-only which will ensure that once we injected the dependency object then that value can never be changed.

At this point, run the application and you should get the output as expected as shown in the below image.
![](https://dotnettutorials.net/wp-content/uploads/2019/03/word-image-21.png)

#### `ASP.NET Core MVC`应用中Action方法注入
Sometimes we may only need a dependency service type in a single action method. For this, use the [FromServices] attribute with the service type parameter in the method.
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
Run the application and you will get the expected output as shown below.
![](https://dotnettutorials.net/wp-content/uploads/2019/03/word-image-22.png)
#### 属性注入
内置IoC容器不支持属性注入，如果需要的话，得使用第三方IoC容器。

### 手动获取服务
It is not required to include dependency services in the constructor. We can access dependent services configured with built-in IoC containers manually using the RequestServices property of HttpContext as shown below.
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
With the above changes in place, run the application and you should get the output as expected as shown in the below image.
![](https://dotnettutorials.net/wp-content/uploads/2019/03/word-image-23.png)
Note: It is recommended to use constructor injection instead of getting it using RequestServices.

### 什么时候该用谁
In real-time applications, you need to register the components such as application-wide configuration as Singleton. The Database access classes like Entity Framework contexts are recommended to be registered as Scoped so that the connection can be re-used. If you want to run anything in parallel then it is better to register the component as Transient.

So, in short:
AddSingleton(): When we use the AddSingleton() method to register a service, then it will create a singleton service. It means a single instance of that service is created and that singleton instance is shared among all the components of the application that require it. That singleton service is created when we requested for the first time.

AddScoped(): Scoped means instance per request. When we use the AddScoped() method to register a service, then it will create a Scoped service. It means, an instance of the service is created once per each HTTP request and uses that instance in other calls of the same request.

AddTransient(): When we use the AddTransient() method to register a service, then it will create a Transient service. It means a new instance of the specified service is created each time when it is requested and they are never shared.

What are the advantages of using ASP.NET Core Dependency Injection?
The ASP.NET Core Dependency Injection allows us to develop loosely coupled software components. Using the ASP.NET Core Dependency Injection, it is very easy to swap with a different implementation of a component.

In the next article, I am going to discuss the Controllers in ASP.NET Core MVC application. Here, in this article, I try to explain the ASP.NET Core Dependency Injection with an example. I hope this article will help you to understand the concept of Dependency Injection in ASP.NET Core Application. 

### 参考
<a href="https://dotnettutorials.net/course/dot-net-design-patterns/">设计模式</a>
<a href="https://dotnettutorials.net/lesson/dependency-injection-design-pattern-csharp/">依赖注入</a>
<a href="https://dotnettutorials.net/lesson/introduction-to-inversion-of-control/">IoC容器</a>


