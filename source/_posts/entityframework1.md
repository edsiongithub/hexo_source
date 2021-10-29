---
layout: 2021-10-25 20:34:00
title: entityframework基础
date: 2021-10-26 09:57:41
tags:
 - C#
 - EntityFramework
---

使用EntityFrameWork基本步骤，Db First模式
1. Nuget上安装EntityFrameWork
2. 配置文件中写连接字符串
3. 创建DbContext
4. 创建Entity，对应到数据库表字段
5. 实例化DbContext，并创建增删改查
下面以Console类型项目，简单展示EntityFrameWork使用过程。事先创建数据库，并建立表及测试数据。本文先介绍纯手工方式使用EntityFrameWork，如果习惯于早期的```Ado.Net```方式访问数据库，那么这样来使用EntityFrameWork就不会太陌生。最后还会介绍使用配置向导来生成所需要的各种类。
<!--more-->

### App.Confi中写连接字符串

首先需要在项目中使用Nuget安装EntityFramework.
![安装EntityFramework](https://raw.githubusercontent.com/edsiongithub/blogimages/master/20210831/install%20ef.png)
```xml
<connectionStrings>
    <add name="BlogContext" providerName="System.Data.SqlClient" connectionString="data source=ServerIP;initial catalog=MyBlogs;user id=yoursqlid;password=yourpwd;" />
  </connectionStrings>
```

### 创建EntityClass及DbContext
``` CSharp
 public class BlogDbContext : DbContext
    {
        public BlogDbContext():
            base("name=BlogContext")
        { }

        public DbSet<Blog_BlogContent> BlogContents { get; set; }

    }

  public class Blog_BlogContent
    {
        public string ID { get; set; }
        public string CatagoryID { get; set; }
        public string Title { get; set; }
        public string Content { get; set; }
        public string Author { get; set; }
        public DateTime CreateTime { get; set; }
        public Boolean IsDisplay { get; set; }
    }

```


### 创建DbContext实例及查询
```Csharp
  using (BlogDbContext db = new BlogDbContext())
    {
        Console.WriteLine("connection status:" + db.Database.Connection.State);

        var blogs = (from b in db.BlogContents
                        select b).ToList();
        foreach (var blog in blogs)
        {
            Console.WriteLine(String.Format("ID:{0}, Title: {1}", blog.ID, blog.Title));
        }
        Console.WriteLine("after select command, \n");
        Console.WriteLine("connection status:" + db.Database.Connection.State);
    }
```


### 有几点需要注意：
* EntityClass的类名，需要跟数据库表名一致（即Blog_BlogContent类名一定对应Blog_BlogContent表）
* EntityClass中的字段和数据库表列对应，数据类型也要对应一致
*  自己的BlogDbContext继承DbContext，且需要暴露一个DbSet
*  DbSet一定需要```get;set;```访问器，否则会报一个参数为空的异常



### 向导模式
使用Db First模式的话，还可以使用VisualStudio的向导模式来配置连接字符串，并且会通过T4模板来生成EntityClass，以及DbContext
* 右键项目，添加，添加新项，选择```ADO.NET``` 实体数据模型
* 选择来自数据库的EF实体设计
![选择来自数据库的EF设计器](https://raw.githubusercontent.com/edsiongithub/blogimages/master/20210831/select1.png)
* 配置数据库连接。
这个地方和早期的SqlDataAdapter + DataSet 访问数据库时非常相似。
![](https://raw.githubusercontent.com/edsiongithub/blogimages/master/20210831/configdb.png)

![](https://raw.githubusercontent.com/edsiongithub/blogimages/master/20210831/configdb1.png)
* 数据库连接配置成功后，选择表来创建实体
配置完成后，选择是否将数据库连接中的敏感数据（用户名、密码）显示在配置文件中。
![配置完成](https://raw.githubusercontent.com/edsiongithub/blogimages/master/20210831/finish.png)

选择数据表来创建实体。
![选择表](https://raw.githubusercontent.com/edsiongithub/blogimages/master/20210831/selecttables.png)
* 最后看下生成的实体图（和早期DataSet中的Table也和像）
![](https://raw.githubusercontent.com/edsiongithub/blogimages/master/20210831/entities.png)
看下设计器上创建好的表对应的实体。同时，项目中多了一个.edmx文件，展开后会看到我们需要的实体类（数据库表名对应的类名）已经自动生成好了，接下去直接写Main方法中的查询那些代码就可以了。


---

纯手工搭建EntityFrame访问数据库完成测试代码：
```Csharp
using System;
using System.Data.Entity;
using System.Linq;

namespace ConsoleDemo.Main
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("from db, get blogs.....");
            using (BlogDbContext db = new BlogDbContext())
            {
                Console.WriteLine("connection status:" + db.Database.Connection.State);

                var blogs = (from b in db.BlogContents
                             select b).ToList();
                foreach (var blog in blogs)
                {
                    Console.WriteLine(String.Format("ID:{0}, Title: {1}", blog.ID, blog.Title));
                }
                Console.WriteLine("after select command, \n");
                Console.WriteLine("connection status:" + db.Database.Connection.State);
            }

            Console.ReadLine();
        }
    }


    public class BlogDbContext : DbContext
    {
        public BlogDbContext():
            base("name=BlogContext")
        { }

        public DbSet<Blog_BlogContent> BlogContents { get; set; }

    }


    public class Blog_BlogContent
    {
        public string ID { get; set; }
        public string CatagoryID { get; set; }
        public string Title { get; set; }
        public string Content { get; set; }
        public string Author { get; set; }
        public DateTime CreateTime { get; set; }
        public Boolean IsDisplay { get; set; }
    }
}

```

