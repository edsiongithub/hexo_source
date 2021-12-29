---
title: asp.net core中AddTransient、AddScoped、AddSingleton区别
date: 2021-12-14 13:40:51
layout: 2021-12-14 13:40:51
id: 0030
tags:
- asp.net core
- DI/IoC
---

* 三者基本概念
* 示例演示三者区别
* 什么时候该用谁

<!--more-->

## 三者基本概念
这三个方法是`asp.net core`提供的，用来将服务注册到依赖注入容器中。而不同的注册方法决定了注入对象的生命周期。从单词字面意思理解，瞬态、范围、单例。
* <strong>AddSingleton:</strong> 创建一个单例服务，即在整个应用程序生命周期内只有这一个服务的实例。对于所有的http请求，都使用这一个实例。首次请求时创建该实例。

* <strong>AddScoped:</strong> 在范围内的每个请求中创建一个新的Scoped服务实例。会对每个http请求进行创建一个实例，但在同一Web请求中的其他服务在调用这个请求，也会使用相同实例。对于同一个客户端请求中相同，不同客户端请求是不同的。

* <strong>AddTransient:</strong> 每次请求都会创建新的实例。

接下来通过示例演示三者区别。创建一个`asp.net core mvc`项目，通过不同注册方式来演示。


## 示例演示

### 创建项目
VS Code中新建目录，用于存放项目。在菜单中打开终端，输入以下命令创建mvc项目。
```powershell
dotnet new -au None -n coremvcdemo
```

### 创建Model、Repository

创建Model目录，新建学生实体类。
```csharp
    public class Student
    {
        public int Id {get;set;}

        [Required(ErrorMessage = "请输入姓名")]
        [Display(Name="名字")]
        public string Name {get; set;}

        [Required]
        [Display(Name="主修科目")]
        public MajorEnum? Major {get;set;}

        [Display(Name = "电子邮件")]
        public string Email{get;set;}
    }

    public enum MajorEnum
    {
        语文,
        英语,
        数学,
        化学 
    }

    public class HomeDetailsViewModel
    {
       public Student student {get;set;}

        public string PageTitle {get;set;}
    
    }
```

```csharp
    public interface IStudentRepository
    {
        Student GetStudent(int id);

        IEnumerable<Student> GetAllStudents();

        Student Add(Student student);
    }
```

```csharp
 public class MockStudentRepositry:IStudentRepository
    {
        private List<Student> _studentList ;

        public MockStudentRepositry()
        {
            _studentList = new List<Student>()
            {
                new Student() { Id = 1, Name = "张三", Major = MajorEnum.化学, Email = "Tony-zhang@52abp.com" },
                new Student() { Id = 2, Name = "李四", Major = MajorEnum.英语, Email = "lisi@52abp.com" },
                new Student() { Id = 3, Name = "王二麻子", Major = MajorEnum.数学, Email = "wang@52abp.com" },
            };
        }


        public Student Add(Student student)
        {
            student.Id = _studentList.Max(s => s.Id) + 1;
            _studentList.Add(student);
            return student;
        }


        public IEnumerable<Student> GetAllStudents()
        {
            return _studentList;
        }

        public Student GetStudent(int id)
        {
            return _studentList.FirstOrDefault(a => a.Id == id);
        } 
    }
```

### 准备Controller、Views

修改HomeController代码如下
<details>
<summary> 点击查看 HomeController 代码</summary>

```csharp
public class HomeController : Controller
{
    private readonly ILogger<HomeController> _logger;
    private readonly IStudentRepository _studentRepository;

    public HomeController(ILogger<HomeController> logger, IStudentRepository studentRepository)
    {
        _studentRepository = studentRepository;
        _logger = logger;
    }

    public IActionResult Index()
    {
        IEnumerable<Student> students = _studentRepository.GetAllStudents();
        return View(students);
    }

    public IActionResult Details(int? id)
    {
        HomeDetailsViewModel homeDetailsViewModel = new HomeDetailsViewModel()
        {
            student = _studentRepository.GetStudent(id ?? 1),
            PageTitle = "学生详细信息"
        };
        return View(homeDetailsViewModel);
    }

   [HttpGet]
   public ViewResult Create()
   {
       return View();
   }

   [HttpPost]
   public IActionResult Create(Student student)
   {
       if(ModelState.IsValid)
       {
           Student newStudent = _studentRepository.Add(student);
           return View();//RedirectToAction("Details", new {id = newStudent.Id});
       }
       return View();
   }
}
```
</details>


Views目录下创建Index.cshtml、Details.cshtml、Create.cshtml
Index.cshtml代码如下，用来展示学生列表。
```html
@model List<Student>
@{ ViewBag.Title ="学生列表"; }

<div  class="col-sm-10">
    <a href="/Home/Create">添加学生</a>
</div>
@foreach (var item in Model)
{
    <div  class="col-sm-10">
        @item.Id @item.Name @item.Major @item.Email 
    </div>
}

  <div class="form-group row">
    <div class="col-sm-10">
      学生总人数 = @Model.Count()
    </div>
  </div>
```

Create.cshtml代码如下，提供学生信息新增。

<details>
<summary>
 查看代码
</summary>

```html
@model Student 
@inject IStudentRepository _studentRepository 
@{ ViewBag.Title ="创建学生信息"; }

<form asp-controller="home" asp-action="create" method="post" class="mt-3">
  <div asp-validation-summary="All" class="text-danger"></div>

  <div class="form-group row">
    <label asp-for="Name" class="col-sm-2 col-form-label"></label>
    <div class="col-sm-10">
      <input asp-for="Name" class="form-control" placeholder="请输入名字"/>
      <span asp-validation-for="Name" class="text-danger"></span>
    </div>
  </div>

  <div class="form-group row">
    <label asp-for="Email" class="col-sm-2 col-form-label"></label>
    <div class="col-sm-10">
      <input
        asp-for="Email"
        class="form-control"
        placeholder="请输入邮箱地址"
     />
      <span asp-validation-for="Email" class="text-danger"></span>
    </div>
  </div>

  <div class="form-group row">
    <label asp-for="Major" class="col-sm-2 col-form-label"></label>
    <div class="col-sm-10">
      <select
        asp-for="Major"
        class="custom-select mr-sm-2"
        asp-items="Html.GetEnumSelectList<MajorEnum>()"
      >
        <option value=""> 请选择</option>
      </select>
      <span asp-validation-for="Major" class="text-danger"></span>
    </div>
  </div>

  <div class="form-group row">
    <div class="col-sm-10">
      <button type="submit" class="btn btn-primary">创建</button>
    </div>
  </div>

  <div class="form-group row">
    <div class="col-sm-10">
      学生总人数 = @_studentRepository.GetAllStudents().Count().ToString()
    </div>
  </div>
</form>
```


</details>

### Program.cs中使用不同方式注册及运行结果

```csharp
using coremvcdemo;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddMvc();//.AddControllersWithViews();
//使用不同方式注册服务
builder.Services.AddScoped<IStudentRepository, MockStudentRepositry>();
//builder.Services.AddSingleton<IStudentRepository, MockStudentRepositry>();
//builder.Services.AddTransient<IStudentRepository, MockStudentRepositry>();
```

![Singleton](https://raw.githubusercontent.com/edsiongithub/blogimages/master/202112/Singleton.gif)
<center>图一、AddSingleton方式注入</center>
<strong>解释：</strong>

可以看到每一次提交，学生总数都在增加。同时，我们点击Home，回到首页，学生列表中也可以看到新增加的两个学生信息。说明MockStudentRepository没有重新实例化，否则学生列表就被初始为最初的三个了。

---

![Scoped](https://raw.githubusercontent.com/edsiongithub/blogimages/master/202112/Scoped.gif)
<center>图二、AddScoped方式注入</center>
<strong>解释：</strong>

第一次点击创建按钮是，学生总人数增加到4了。但再次提交时，学生总数依然是4。
对于一次请求，到Home控制器，该控制器实例化studentRepository，对于同一个http请求，控制器和视图等多个地方需要用到服务，则在该http请求域中使用同一个服务实例。

---
![Transient](https://raw.githubusercontent.com/edsiongithub/blogimages/master/202112/Transient.gif)
<center>图三、AddTransient方式注入</center>
<strong>解释：</strong>

无论点击几次创建按钮，学生总数一直是3。说明MockStudentRepository一直在被实例化，总是显示新实例化的对象中创建的三个学生信息。

## 什么时候该用谁

一般较常用AddScoped和AddSingleton。但在多线程时，需要注意，AddScoped无法在新线程里面共享主线程中创建的实例。因此，需要使用AddSingleton方式注入。


| 服务类型 | 同一个HTTP请求的范围 | 横跨多个不同 HTTP 请求 | 使用时参考意见 |
| --- | --- | --- | --- |
| Singleton Service | 同一个实例 | 同一个实例 | 每次创建都会占用更多内存资源，对性能产生负面影响。适用于状态很少或无状态的轻量级服务 |
| Scoped Service | 同一个实例 | 新实例 | 需要维护请求中的状态时 |
| Transient Service | 新实例 | 新实例 | 可以提高内存效率，但随着时间累积将造成内存泄漏 |